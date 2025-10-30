## **Intent Detection and Entity Recognition with Text-to-SQL**

전통적인 NLU 시스템을 LLM으로 대체하여 Intent와 Entity를 추출한 후, 매핑된 SQL 템플릿을 사용

<img width="993" height="702" alt="image" src="https://github.com/user-attachments/assets/a12d3482-ab04-4b59-990f-97d9ebb954c9" />



```
사용자 질문
    ↓
┌─────────────────────────────┐
│ LLM: Intent Detection       │ → Few-shot prompting
│ 예: RETRIEVE_RESERVATIONS   │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ LLM: Entity Recognition     │ → Few-shot prompting
│ 예: Start Date, End Date    │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ Intent → Tables 매핑        │ → Key-value 검색
│ RETRIEVE_RESERVATIONS       │
│ → [reservations, flights]   │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ Schema Filtering            │
│ 해당 테이블의 스키마만 추출  │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ SQL 생성                    │
│ 모든 정보를 프롬프트로 조합  │
└─────────────────────────────┘
```
```

**Step1: Intent Detection (Few-shot)**

```
Few-shot 예시:
"Need all the bookings from 10th to 15th October 2023" → RETRIEVE_RESERVATIONS
"Who made a reservation last Wednesday?" → IDENTIFY_RECENT_CUSTOMERS
"What was our earning from confirmed bookings?" → CALCULATE_REVENUE
사용자 질문: "Show me reservations from Oct 10 to 15, 2023"
LLM 출력: RETRIEVE_RESERVATIONS
```

**Step2: Entity Recognition (Few-shot)**

```
Few-shot 예시:
"October 10th to October 15th, 2023" → "Start Date:October 10th, 2023|End Date:October 15th, 2023"
사용자 질문: "Show me reservations from Oct 10 to 15, 2023"
LLM 출력: "Start Date:October 10, 2023|End Date:October 15, 2023"
```

**Step3: Intent->Tables 매핑**

```
intent_to_tables = {
    "RETRIEVE_RESERVATIONS": ["reservations", "flights"],
    "IDENTIFY_RECENT_CUSTOMERS": ["reservations", "customers"],
    "CALCULATE_REVENUE": ["reservations", "transactions"]
}
# RETRIEVE_RESERVATIONS → ["reservations", "flights"]
```

**Step 4: Schema Filtering**

```
-- reservations 테이블 스키마만 가져오기
CREATE TABLE reservations (
  reservation_id INT64,
  customer_id INT64,
  flight_id INT64,
  reservation_datetime DATETIME,
  status STRING
)
```
```

**Step 5: SQL 생성 프롬프트**

```
INTENT: RETRIEVE_RESERVATIONS
ENTITIES: Start Date:October 10, 2023|End Date:October 15, 2023
TABLES: reservations, flights
SCHEMA: [위의 스키마 정보]
User Query: "Show me reservations from Oct 10 to 15, 2023"
→ Please construct a SQL query using the above information.
```

**LLM 생성 SQL:**

```
SELECT *
FROM flight_reservations.reservations
WHERE reservation_datetime BETWEEN '2023-10-10' AND '2023-10-15'
```

**장점 (Advantages):**

- 테이블과 컬럼 수가 제한된 소규모 BigQuery 데이터셋에 적합함.
- 기존의 의도 탐지(intent detection) 및 개체명 인식(NER) 시스템을 LLM 기반 솔루션으로 대체하는 데 이상적임.
- 여러 모델을 학습하거나 재학습하는 복잡한 파이프라인이 필요 없으므로 단순화됨.
- 고객 대상 애플리케이션이나 시나리오 범위가 제한된 시스템에 적합함.
- SQL 쿼리 생성의 높은 정확도와 설명 가능성(explainability)을 목표로 함.
- 각 모듈이나 컴포넌트를 디버깅하고 검사하기 쉬움.

**제한점 (Limitations):**

- 새로운 시나리오를 추가하려면 few-shot 프롬프트에 사용되는 데이터를 수동으로 업데이트해야 함.
- 자동으로 새로운 쿼리 시나리오를 처리할 유연성이 부족하며, 수동 개입이 필요함.

## **Pattern II: RAG (Retrieval-Augmented Generation)**

수백~수천 개의 테이블이 있을 때, Intent 매핑이 불가능합니다. 대신 Semantic Search로 관련 테이블/컬럼을 찾음

<img width="989" height="723" alt="image" src="https://github.com/user-attachments/assets/2e120965-e996-45ea-9393-1d2dae3aaf06" />


```
[사전 준비]
모든 테이블/컬럼 설명
    ↓
Embedding Model (textembedding-gecko)
    ↓
Vector Index (FAISS/Vertex AI Vector Search)
[쿼리 처리]
사용자 질문 → Embedding
    ↓
┌─────────────────────────────┐
│ Table Semantic Search       │
│ Top K=5 테이블 검색            │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ Column Semantic Search      │
│ Top K=20 컬럼 검색            │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│ 매칭된 스키마로 SQL 생성         │
└─────────────────────────────┘
```

**사전 준비: 임베딩 생성**

```
# 테이블 설명 임베딩
table_descriptions = [
    "reservations: Contains flight booking records",
    "customers: Stores customer information and demographics",
    "flights: Flight schedule and route information"
]
# Vertex AI Embedding Model 사용
embeddings = embedding_model.encode(table_descriptions)
# FAISS 인덱스 생성
table_index.add(embeddings)
```

**Step 1: 사용자 질문 → Table Search**

```
user_query = "Who booked flights to New York last week?"
# 질문을 임베딩으로 변환
query_embedding = embedding_model.encode(user_query)
# 유사도 검색 (Top K=5)
matched_tables = table_index.search(query_embedding, k=5)
# 결과:
# 1. reservations (score: 0.92)
# 2. flights (score: 0.85)
# 3. customers (score: 0.78)
# 4. transactions (score: 0.45)
# 5. airports (score: 0.32)
```

**Step 2: Column Search**

```
# 컬럼 설명 검색
matched_columns = column_index.search(query_embedding, k=20)
# 결과:
# reservations.reservation_datetime (score: 0.91)
# reservations.customer_id (score: 0.88)
# flights.destination (score: 0.86)
# customers.first_name (score: 0.82)
# ...
```

**Step 3: SQL 생성**

```
USER_QUERY: "Who booked flights to New York last week?"
MATCHED_SCHEMA:
- Table: reservations
  Columns: reservation_id, customer_id, reservation_datetime
- Table: flights
  Columns: flight_id, destination, departure_datetime
- Table: customers
  Columns: customer_id, first_name, last_name
→ Construct SQL using ONLY the matched schema above.
```

**LLM 생성 SQL:**

```
SELECT c.first_name, c.last_name, r.reservation_datetime
FROM reservations r
JOIN customers c ON r.customer_id = c.customer_id
JOIN flights f ON r.flight_id = f.flight_id
WHERE f.destination = 'New York'
  AND r.reservation_datetime >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
```

**장점 (Advantages):**

- 대규모 데이터셋이나 열(column)이 많은 BigQuery 테이블을 처리하기에 적합함.
- 의도 탐지(intent detection), 개체명 인식(NER), 수동 매핑(manual mapping) 과정을 시맨틱 검색(semantic search) 한 단계로 통합해 워크플로우를 단순화함.
- 짧고 효율적인 프롬프트를 사용하므로 추론(inference) 비용을 절감할 수 있음.
- 관련 테이블과 컬럼 설명만 선택적으로 사용하여 불필요한 데이터를 제거함.
- 데이터 처리 작업의 확장성(scalability)을 향상시킴.

**제한점 (Limitations):**

- 시맨틱 검색을 도입하면서 파이프라인이 더 복잡해짐.
- 효과적인 검색을 위해 임베딩 전략과 파라미터 설정을 신중히 고려해야 함.
- 도구가 관련 없는 테이블이나 컬럼을 검색하는 경우가 많아,
    
    복잡한 쿼리나 불필요한 조인(join)이 발생하고,
    
    계산 시간 증가 및 불필요하게 긴 응답으로 이어질 수 있음.
    
- 초기 단계가 본질적으로 검색 기반 접근법이므로,
    
    낮은 재현율(recall)과 정밀도(precision) 같은 일반적인 정보검색 문제를 겪을 수 있음.
    
    이를 완화하기 위해 재순위화(re-ranking) 등의 보완 전략이 필요할 수 있음.
    

## **Pattern III: SQL Agents (LangChain)**

<img width="985" height="709" alt="image" src="https://github.com/user-attachments/assets/6efca781-79e7-498d-ae6e-613495938ca8" />


Agent가 자율적으로 데이터베이스 스키마를 탐색하고, SQL을 생성하며, 오류 발생 시 자가 수정합니다.

```
사용자 질문
    ↓
┌──────────────────────────────┐
│ SQL Agent (LangChain)        │
│ - ODBC 연결 (BigQuery)        │
│ - 스키마 자동 추론               │
└──────────────────────────────┘
    ↓
Agent의 사고 과정 (ReAct)
    ↓
Action: sql_db_list_tables → 테이블 목록 확인
    ↓
Action: sql_db_schema → 스키마 확인
    ↓
Action: sql_db_query → SQL 실행
    ↓
실패? → Traceback 분석 → 수정 → 재실행
    ↓
성공 → 최종 답변
```

**예시 1: 성공 케이스**

```
사용자: "Show all reservations from Oct 10-15, 2023"
> Entering new AgentExecutor chain...
[Agent 사고 과정]
Thought: I need to find the reservations table and check its schema
Action: sql_db_list_tables
Observation: customers, flights, reservations, transactions
Action: sql_db_schema
Action Input: reservations
Observation:
CREATE TABLE reservations (
  reservation_id INT64,
  customer_id INT64,
  flight_id INT64,
  reservation_datetime DATETIME,
  status STRING
)
Thought: Now I can construct the query
Action: sql_db_query
Action Input:
SELECT * FROM reservations
WHERE reservation_datetime BETWEEN '2023-10-10' AND '2023-10-15'
Observation: [(6, 6, 6, datetime.datetime(2023, 10, 10, 10, 0), 'Confirmed'), ...]
Final Answer: 2 flight reservations were found.
> Finished chain.
```

**예시 2: 자가 수정 (Self-Correction)**

```
사용자: "Customers who booked in the last 7 days"
> Entering new AgentExecutor chain...
[1차 시도]
Action: sql_db_query
Action Input:
SELECT customer_id, first_name, last_name, reservation_datetime
FROM customers c
JOIN reservations r ON c.customer_id = r.customer_id
WHERE r.reservation_datetime >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
Observation: Error: Column name customer_id is ambiguous
             ^^^^^^^^^ 어느 테이블의 customer_id인지 모호함!
[Agent 분석]
Thought: The error says customer_id is ambiguous.
         I need to specify which table it belongs to.
[2차 시도 - 수정]
Action: sql_db_query
Action Input:
SELECT c.customer_id, c.first_name, c.last_name, r.reservation_datetime
FROM customers c
JOIN reservations r ON c.customer_id = r.customer_id
WHERE r.reservation_datetime >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
Observation: [(20, 'Ryan', 'Reynolds', ...), (19, 'Queen', 'Elizabeth', ...)]
Final Answer: Found 5 customers who booked in the last 7 days.
> Finished chain.
```

**장점 (Advantages):**

- 사용자의 작업을 크게 단순화할 수 있음.
- 다양한 시나리오에 신속하게 적용할 수 있음.
- 시스템이 실패나 오류로부터 학습하며 성능을 개선함.
- 스키마가 일관된 데이터베이스나 표준화된 쿼리 연산을 사용하는 환경에서 특히 효과적임.

**제한점 (Limitations):**

- 일반적인 에이전트가 복잡한 데이터베이스 구조를 탐색하기 어려움.
- 적응력(adaptability)이 낮고, 맞춤화(customization)가 충분하지 않음.
- LLM은 한정된 컨텍스트 윈도우를 가지므로, 규모가 큰 데이터베이스 처리에 제약이 있음.
- 특정 데이터베이스나 쿼리 요구사항에 맞추려면 프롬프트나 파라미터를 세밀하게 조정(fine-tuning)해야 하며,
    
    이로 인해 시스템의 불투명성과 유연성 부족이 심화될 수 있음.
    
- 에이전트는 종종 존재하지 않는 테이블이나 필드를 “지어내는(hallucination)” 문제를 겪을 수 있음.

## **Pattern IV: Self-Correction (자가 수정)**

Seed 프롬프트로 SQL을 생성하고, 실패 시 에러 메시지를 프롬프트에 추가하여 반복 수정합니다.

<img width="988" height="612" alt="image" src="https://github.com/user-attachments/assets/b6dbe30a-ce37-4532-b4e5-a81be0717b37" />


```
사용자 질문 + Seed Template
    ↓
┌──────────────────────────────┐
│ LLM: SQL 생성 (1차 시도)        │
└──────────────────────────────┘
    ↓
실행 성공?
    ↓ No
┌──────────────────────────────┐
│ Evolution Prompt             │
│ 원본 + 에러 메시지 + 실패 SQL     │
└──────────────────────────────┘
    ↓
LLM: SQL 수정 (2차 시도)
    ↓
실행 성공?
    ↓ Yes
   종료
```

**Seed Template**

```
USER_QUERY: {query}
SCHEMA: {schema}
Please construct a SQL query for BigQuery.
IMPORTANT: Use ONLY DATETIME (not TIMESTAMP).
```

**시나리오: 18세 이상, Confirmed 예약, 현재 월**

```
사용자: "Customers aged 18+ with Confirmed reservations this month, ordered by age"
```

**Attempt 1: 실패 (DATEDIFF 오류)**

```
WITH current_month AS (
  SELECT DATE_TRUNC(CURRENT_DATE(), MONTH) AS start_of_month
)
SELECT c.customer_id, c.first_name, c.last_name,
       DATEDIFF(current_month.end_of_month, c.date_of_birth) / 365 AS age
FROM current_month
CROSS JOIN customers c
WHERE DATEDIFF(current_month.end_of_month, c.date_of_birth) / 365 >= 18
-- 오류: DATEDIFF 함수가 BigQuery에 없음!
```
**Evolution Prompt (자동 생성)**
```
Original Prompt: [위의 Seed Template]
Encountered Error: Function DATEDIFF not found
Generated SQL (Failed):
[위의 실패한 SQL]
→ Please modify the SQL to avoid this error.
```

**Attempt 2: 실패 (구문 오류)**

```
SELECT DATE_TRUNC(CURRENT_DATE(), MONTH) AS start_of_month
       -- WITH 키워드 누락!
)
SELECT c.customer_id, ...
       DATE_DIFF(current_month.end_of_month, c.date_of_birth, YEAR) AS age
...
-- 오류: Syntax error near SELECT
```
**Evolution Prompt 2**
```
[이전 프롬프트]
Encountered Error: Syntax error - missing WITH keyword
Generated SQL (Failed):
[위의 SQL]
→ Please add the WITH keyword.
```

**Attempt 3: 성공**

```
WITH current_month AS (
  SELECT DATE_TRUNC(CURRENT_DATE(), MONTH) AS start_of_month,
         DATE_ADD(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 1 MONTH) AS end_of_month
)
SELECT c.customer_id, c.first_name, c.last_name,
       DATE_DIFF(current_month.end_of_month, c.date_of_birth, YEAR) AS age,
       r.reservation_id, r.status
FROM current_month
CROSS JOIN customers c
JOIN reservations r ON c.customer_id = r.customer_id
WHERE r.status = 'Confirmed'
  AND r.reservation_datetime BETWEEN current_month.start_of_month
                                 AND current_month.end_of_month
  AND DATE_DIFF(current_month.end_of_month, c.date_of_birth, YEAR) >= 18
ORDER BY age;
-- ✅ 성공!
```

**장점 (Advantages):**

- 직접적인 스키마 추론(direct schema inference)에 기반하므로, 구현이 간단하고 직관적임.
- 테이블 및 컬럼 설명이 선택사항(optional)이기 때문에 최소한의 메타데이터로도 작동 가능, 유연성(flexibility)이 높음.
- 높은 설명 가능성(explainability)이 큰 장점으로, 프롬프트의 변화 과정(prompt evolution)과 에러 코드(error codes)를 추적하여
    
    LLM의 응답을 이해하고 문제를 진단(troubleshoot)할 수 있음.
    
- 이전 모델들보다 맞춤화(customizability)가 향상되어,
    
    프롬프트 구성, 진화 기준(evolution criteria), 재시도 횟수(retry count) 등을 필요에 따라 조정 가능함.
    

**제한점 (Limitations):**

- 여러 차례의 반복(iteration)이 필요할 수 있어, 실행 오버헤드(execution overhead)가 발생하고 성능에 영향을 줄 수 있음.
- 스키마 추론을 위한 테이블을 수동으로 선택해야 하는 경우가 있어,
    
    자동화된 접근 방식보다 노동집약적이고 비효율적일 수 있음 (특히 새로운 시나리오나 미지의 데이터베이스를 다룰 때.)
    
- 시드 프롬프트(seed prompt)에 전체 스키마를 포함해야 하므로,
    
    LLM의 컨텍스트 윈도우 한계(context window limitation) 때문에 리소스 소모가 큼.
    
    특히 테이블/컬럼 설명과 샘플 데이터(특히 범주형(categorical)이나 문자열(string) 데이터)가 포함될 경우 문제가 심화됨.
    
- 프롬프트가 반복 과정에서 에러 메시지나 이전 실패 시도의 내용을 추가하면서 점점 커지면,
    
    컨텍스트 윈도우를 초과할 위험이 커지고,
    
    그 결과 모델이 증가하는 입력량을 효율적으로 처리하기 어려워질 수 있음**.**
    

## **Pattern V: Self-Correction + Optimization**

Pattern IV + 성공 후에도 계속 최적화하여 가장 빠른 쿼리를 찾습니다.

<img width="1007" height="622" alt="image" src="https://github.com/user-attachments/assets/06f8d73d-6e88-4758-89b5-7bf39ac10f13" />


```
사용자 질문
    ↓
┌──────────────────────────────┐
│ SQL 생성 (Trial 1)           │
└──────────────────────────────┘
    ↓
실행 성공 → Latency 기록
    ↓
┌──────────────────────────────┐
│ Optimization Loop            │
│ "더 빠른 쿼리 생성" (Trial 2) │
└──────────────────────────────┘
    ↓
실행 성공 → Latency 기록
    ↓
... (설정된 Trial 수만큼 반복)
    ↓
모든 성공 쿼리를 Latency로 정렬
    ↓
가장 빠른 쿼리 선택
```

**사용자: "Reservations from Oct 10-15, 2023"**

**Trial 1: 기본 JOIN**

```
SELECT r.*, f.*
FROM reservations r
JOIN flights f ON CAST(r.flight_id AS STRING) = CAST(f.flight_id AS STRING)
WHERE r.reservation_datetime BETWEEN '2023-10-10' AND '2023-10-15'
-- ✅ 성공
-- Latency: 8.94초
```

**Trial 2: 별칭 + ORDER BY**

```
SELECT r.*, f.*
FROM reservations AS r
JOIN flights AS f ON r.flight_id = f.flight_id
WHERE r.reservation_datetime BETWEEN '2023-10-10' AND '2023-10-15'
ORDER BY r.reservation_datetime
-- ✅ 성공
-- Latency: 12.22초 (느려짐!)
```

**Trial 3: LIMIT 추가**

```
-- [Trial 2와 동일]
ORDER BY r.reservation_datetime
LIMIT 100
-- ✅ 성공
-- Latency: 11.08초
```

**Trial 4: INDEX 힌트 (실패)**

```
-- [Trial 3과 동일]
LIMIT 100
USE INDEX (reservation_datetime)
-- ❌ 실패 (BigQuery는 USE INDEX 미지원)
```

**Trial 5: Optimizer 힌트**

```
SELECT /*+ USE_NL(r) USE_NL(f) */
  r.*, f.*
FROM reservations AS r
JOIN flights AS f ON r.flight_id = f.flight_id
WHERE r.reservation_datetime BETWEEN '2023-10-10' AND '2023-10-15'
ORDER BY r.reservation_datetime
LIMIT 100
-- ✅ 성공
-- Latency: 7.88초 ← 최고 성능!
```
**최종 순위**
```
Rank 1: Trial 5 (Optimizer Hints) - 7.88초 ⭐
Rank 2: Trial 1 (Basic JOIN)      - 8.94초
Rank 3: Trial 3 (with LIMIT)      - 11.08초
Rank 4: Trial 2 (with ORDER BY)   - 12.22초
```

### **중요 설정 변화**

Temperature=1의 효과:

- 같은 프롬프트여도 다른 최적화 전략 시도
- Trial마다 다른 힌트 사용 (INDEX, JOIN 방식 등)
- 더 많은 후보 쿼리 생성

```
# Pattern IV (Self-Correction만)
temperature = 0  # 결정적 출력 (같은 입력 → 같은 출력)
# Pattern V (Optimization 포함)
temperature = 1  # 창의적 출력 (다양한 최적화 시도)
```

**장점 (Advantages):**

- 쿼리 효율성과 속도를 향상시켜, 동적 최적화(dynamic optimization)를 가능하게 함.
- 쿼리 실행 시간이 중요한 고성능 환경(high-performance setting)에 적합함.

**제한점 (Limitations):**

- 여러 번의 LLM 호출과 쿼리 실행을 반복해야 하므로, 호출당 비용이 증가하고 전체 비용이 높아짐.
- 후속 상태 정보(previous state information)가 필요하므로 병렬 처리(parallelization)가 불가능,
    
    모든 후보 쿼리의 실행 및 순위 평가(ranking)를 기다려야 해서 처리 속도가 느림.
    
- 가장 빠른 쿼리가 정확한 결과를 보장하지 않기 때문에, 결과 검증(result evaluation)이 필수적임.

---

논의 필요 내용

1. Flow에 어떤 기능들을 추가할지

2. 스키마를 가져올 때 얼마나 어떻게 가져올지

3. cypher을 생성하는 모델 어떤걸로? 몇개의 모델로?
