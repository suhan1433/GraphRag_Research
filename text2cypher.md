# **Overview**

Text2Cypher agent의 사용이유

1. 사용자 질문에 대해 LLM으로 Cypher 생성 시, 해당 Cypher가 DB에서 작동 안 할 수 있음.
2. 만약 Cypher가 DB에서 작동 하더라도, 사용자 질문과 무관한 내용일수 있음

이 두가지 문제점을 해결하기 위해 Agent를 도입

아래는 Text2Cypher Agnet 구조의 3단계를 설명 하고,

가장 성능이 좋은 반복 및 평가 구조를 심층적으로 분석함.

# **Naive Text2Cypher Flow**

3가지 Step으로 생성됨

1. Cypher query 생성:

입력 받은 사용자 질문을 vectorDB에 저장된 유사한 예시를 가져와 few-shot learning을 진행

1. GraphDB에 Cypher query를 실행
2. DB결과를 LLM으로 처리해서 답변함

# **Naive Text2Cypher With Retry Flow**

self-correction 메커니즘을 추가하여 retry를 할 수 있도록 함.

Cypher query실행이 실패 했을 때, CorrectChpherEventstep에서 에러 정보를 LLM에 주어 Cypher을 고치도록 함

# **Naive Text2Cypher With Retry and Evaluation Flow**

evaluation 단계를 추가하며, 이 과정은 GraphDB 검색 결과가 사용자의 질문 답변에 충분히 답할 수 있는지 확인

만약 충촉되지 않는다면, query를 개선 방향성과 함께 수정 단계로 되돌림.

이 과정들이 충족되면, 마지막 요약 과정으로 넘어감.

## **과정**

### **Step1. generate_cypher**

이 과정은 처음 사용자 질문에 대해 Cypher을 생성하는 과정임.

이는 GraphDB에서 사용자 질문과 유사한 Cypher을 검색하고, 이를 활용하여 Cypher을 생성함

1. Few-Shot 진행
2. Few-Shot 정보를 활용하여 Cypher 생성

| **Input** | **Output** |
| --- | --- |
| 사용자 질문 | 사용자 질문, Cypher |
1. **Few-Shot 진행**

vectorDB에서 임베딩 된 사용자 질문과 유사도가 높은 7개의 few-shot 예시를 반환

```
MATCH (f:Fewshot)
WHERE f.database = $database
WITH f, vector.similarity.cosine(f.embedding, $embedding) AS score
ORDER BY score DESC LIMIT 7
RETURN f.question AS question, f.cypher AS cypher""",
param_map={"embedding": embedding, "database": database},
```

e.g.

```
(:Fewshot {
    id: "How to get user name?gpt-4graph1",
    question: "How to get user name?",
    cypher: "MATCH (u:User) RETURN u.name",
    llm: "gpt-4",
    database: "graph1",
    created: (날짜/시간),
    embedding: [0.123, -0.98, ...]     // 숫자 벡터값
})
```

1. **Cypher 생성**

위에서 **“검색된 few-shot + 사용자 질문 +  스키마 + template”** 를 통해 LLM이 Cypher 생성

e.g.

```
사용자 질문 = "나이가 20살 이상인 사람 이름을 알려줘"
schema = """
Nodes:
- Person(name: string, age: int)
- Movie(title: string, year: int)
Relationships:
- (Person)-[:ACTED_IN]->(Movie)
"""
fewshot_examples = [
    {
        "question": "나이가 30살 이상인 사람 이름은?",
        "cypher": "MATCH (p:Person) WHERE p.age >= 30 RETURN p.name"
    },
    {
        "question": "20살 미만인 사람?",
        "cypher": "MATCH (p:Person) WHERE p.age < 20 RETURN p"
    }
]
```

**e.g. cypher 생성 코드**

스키마의 경우 이미 구축해 놓은 neo4j 저장소를 통해 얻음.

```
from llama_index.core import ChatPromptTemplate
GENERATE_SYSTEM_TEMPLATE = """Given an input question, convert it to a Cypher query. No pre-amble.
Do not wrap the response in any backticks or anything else. Respond with a Cypher statement only!"""
GENERATE_USER_TEMPLATE = """You are a Neo4j expert. Given an input question, create a syntactically correct Cypher query to run.
Do not wrap the response in any backticks or anything else. Respond with a Cypher statement only!
Here is the schema information
{schema}
Below are a number of examples of questions and their corresponding Cypher queries.
{fewshot_examples}
User input: {question}
Cypher query:"""
async def generate_cypher_step(llm, graph_store, subquery, fewshot_examples):
    schema = graph_store.get_schema_str(exclude_types=["Actor", "Director"])
    generate_cypher_msgs = [
        ("system", GENERATE_SYSTEM_TEMPLATE),
        ("user", GENERATE_USER_TEMPLATE),
    ]
    text2cypher_prompt = ChatPromptTemplate.from_messages(generate_cypher_msgs)
    response = await llm.achat(
        text2cypher_prompt.format_messages(
            question=subquery,
            schema=schema,
            fewshot_examples=fewshot_examples,
        )
    )
    return response.message.content

```

### **Step2. excute_query**

이 과정은 생성된 Cypher를 GraphDB에 실행하는 과정임.

Cypher 실행이 실패한다면, 실행이 되도록 설정한 반복 횟수 만큼 Cypher을 수정하는 이벤트(`CorrectCypherEvent`)를 실행

1. GraphDB에서 Cypher을 실행
2. 만약 실행이 된다면 최대 100개까지 저장(만약 최종 실패일 경우 질문 가이드로 사용 가능, 아니라면 적은 양이 나옴)
3. 만약 실행이 안 된다면 “DB error정보 + 사용자 질문 + Cypher”을 입력으로 `CorrectCypherEvent`실행
4. `CorrectCypherEvent` 이벤트를 최대 반복 횟수까지 진행하며, 올바른 Cypher를 얻음

| **Input** | **Output** |
| --- | --- |
| 사용자 질문, Cypher | 사용자 질문, Cypher, GraphDB검색 결과 |

**e.g. CorrectCypherEvent**

LLM이 “그래프DB 스키마 + template + DB error정보 + 사용자 질문 + Cypher” 을 통해 수정한 Cypher 생성

```
from llama_index.core import ChatPromptTemplate
CORRECT_CYPHER_SYSTEM_TEMPLATE = """You are a Cypher expert reviewing a statement written by a junior developer.
You need to correct the Cypher statement based on the provided errors. No pre-amble."
Do not wrap the response in any backticks or anything else. Respond with a Cypher statement only!"""
CORRECT_CYPHER_USER_TEMPLATE = """Check for invalid syntax or semantics and return a corrected Cypher statement.
Schema:
{schema}
Note: Do not include any explanations or apologies in your responses.
Do not wrap the response in any backticks or anything else.
Respond with a Cypher statement only!
Do not respond to any questions that might ask anything else than for you to construct a Cypher statement.
The question is:
{question}
The Cypher statement is:
{cypher}
The errors are:
{errors}
Corrected Cypher statement: """
async def correct_cypher_step(llm, graph_store, subquery, cypher, errors):
    schema = graph_store.get_schema_str(exclude_types=["Actor", "Director"])
# Correct cypher
    correct_cypher_messages = [
        ("system", CORRECT_CYPHER_SYSTEM_TEMPLATE),
        ("user", CORRECT_CYPHER_USER_TEMPLATE),
    ]
    correct_cypher_prompt = ChatPromptTemplate.from_messages(correct_cypher_messages)
    response = await llm.achat(
        correct_cypher_prompt.format_messages(
            question=subquery, schema=schema, errors=errors, cypher=cypher
        )
    )
    return response.message.content

```

### **Step3. evaluate_context**

이 과정은 “GraphDB에서 가져온 결과가 사용자 질문에 답하기 충분한가?” 를 판단하는 과정

만약 LLM이 충분하지 않다고 판단 되면,  `CorrectCypherEvent` 이벤트 호출을 반복하여, Cypher수정

1. GraphDB검색 정보가 사용자 질문에 충분히 답변 가능한지 LLM이 판단
2. 충분하다고 판단되면, “OK”가 답변으로 반환되어 다음과정 진행
3. 충분하지 않다면, 에러정보와 함께 `CorrectCypherEvent`을 실행하여 Cypher을 수정 후 다시 `excute_query` 단계 부터 시작

| **Input** | **Output** |
| --- | --- |
| 사용자 질문, Cypher, GraphDB검색 결과 | 사용자 질문, Cypher, GraphDB검색 결과, 
GraphDB검색 결과 평가 |
- GraphDB검색 결과 평가 정보는 위에 1번 과정 충족 된다면 “OK” 저장, 아니라면 에러정보 저장 됨

e.g. 검색정보가 질문에 대해 충분히 답변 가능한지 판단

```
from llama_index.core import ChatPromptTemplate
EVALUATE_ANSWER_SYSTEM_TEMPLATE = """You are a helpful assistant that must determine if the provided database output (from a Cypher query) is sufficient and relevant to answer a given question. You will receive three inputs:
1. question – The user’s question.
2. cypher – The Cypher query used.
3. database_output – The query results.
Your task is:
* Check if the database_output is enough to answer the question meaningfully.
* If sufficient, reply with "Ok".
* If insufficient, explain what’s wrong (e.g., missing data, incorrect query structure, irrelevant results) and how to fix the Cypher query (or approach) so it would produce the necessary answer."""
EVALUATE_ANSWER_USER_TEMPLATE = """You are given the following information:
Question:
{question}
Cypher Query:
{cypher}
Database Output:
{context}
Analyze whether the provided database output is adequate to answer the question.
* If the context is sufficient, return "Ok".
* Otherwise, describe in detail what the error or shortcoming is, and how to correct the Cypher query (or the approach).
"""
async def evaluate_database_output_step(llm, subquery, cypher, context):
# Correct cypher
    correct_cypher_messages = [
        ("system", EVALUATE_ANSWER_SYSTEM_TEMPLATE),
        ("user", EVALUATE_ANSWER_USER_TEMPLATE),
    ]
    correct_cypher_prompt = ChatPromptTemplate.from_messages(correct_cypher_messages)
    response = await llm.achat(
        correct_cypher_prompt.format_messages(
            question=subquery, cypher=cypher, context=context
        )
    )
    return response.message.content

```

### **Step4. summarize_answer**

이 과정은 최종적으로 만들어낸 Cypher을 저장하는 역할과 최종답변을 생성하는 역할을 함.

Cypher을 저장하는 이유는 추후 few-shot으로 재사용하는 용도임

1. 루프(Cypher생성-수정-평가)를 1회라도 거쳤다면 “success”로 저장 (== GraphDB검색 결과 평가가 “OK”)
2. 아니라면 “fail”케이스로 저장
3. 1번과 2번 과정 진행 때 정보들(사용자 질문+Cypher+임베딩)을 DB에 저장
4. “사용자 질문+Cypher+GraphDB검색 결과”를 LLM에 입력하여 답변 생성
    
    이때 정보가 부족하면 추가 정보를 요청하는 답변을 출력
    

| **Input** | **Output** |
| --- | --- |
| 사용자 질문, Cypher, GraphDB검색 결과,
GraphDB검색 결과 평가 | 최종 답변 |

e.g. 4번 LLM으로 답변 생성 과정

```
from llama_index.core import ChatPromptTemplate
FINAL_ANSWER_SYSTEM_TEMPLATE = """
You are a highly intelligent assistant trained to provide concise and accurate answers.
You will be given a context that has been retrieved from a Neo4j database using a specific Cypher query.
Your task is to analyze the context and answer the user’s question based on the information provided in the context.
If the context lacks sufficient information, inform the user and suggest what additional details are needed.
Focus solely on the context provided from the Neo4j database to form your response.
Avoid making assumptions or using external knowledge unless explicitly stated in the context.
Ensure the final answer is clear, relevant, and directly addresses the user’s question.
If the question is ambiguous, ask clarifying questions to ensure accuracy before proceeding.
"""
FINAL_ANSWER_USER_TEMPLATE = """
Based on this context retrieved from a Neo4j database using the following Cypher query:
`{cypher_query}`
The context is:
{context}
Answer the following question:
<question>
{question}
</question>
Please provide your answer based on the context above, explaining your reasoning.
If clarification or additional information is needed, explain why and specify what is required.
"""
def get_naive_final_answer_prompt():
    final_answer_msgs = [
        ("system", FINAL_ANSWER_SYSTEM_TEMPLATE),
        ("user", FINAL_ANSWER_USER_TEMPLATE),
    ]
    return ChatPromptTemplate.from_messages(final_answer_msgs)

```

# **예시**

## **예시1**

**도메인:** 영화추천

**상황:** evaluate_context가 충족되지 않는 경우

**질문:** “해리포터 주인공이 나오는 로멘스 영화를 알려줘”

1. **“generate_cypher + excute_query” 과정**
2. **“evaluate_context” 과정 실패**
3. **“generate_cypher + excute_query” 과정 재시작**

2번 에러 결과를 반영하여 재생성

1. **“evaluate_context” 과정 실패**
2. **“generate_cypher + excute_query” 과정 재시작**

4번 에러 결과를 반영하여 재생성

1. **“evaluate_context” 과정 성공**

사용자 질문을 GraphDB 검색 결과로 충분하다고 평가하여, LLM이 답변으로 “OK”만 출력된 경우

1. **최종답변**

## **예시2**

**도메인:** 영화추천

**상황:** excute_query가 충족되지 않는 경우

**질문:** 무슨 치킨이 있어?

1. **“generate_cypher + excute_query” 과정 실패**
2. **excute_query**의 **CorrectCypherEvent** 이벤트 과정

올바른 Cypher을 생성 할 수 없다고 답변

1. **반복 횟수 만큼 시도하더라도 Cypher을 생성 불가**

excute_query에서 DB안의 정보들을 메시지로 출력

1. **evaluate_context 과정 건너띔**

반복 횟수가 남지 않아 최종 답변 과정으로 이동

1. **최종 답변**

[여기서 실행가능](https://text2cypher-llama-agent.up.railway.app/)

# **결론**

효율(성능+시간)이 가장 좋은 Naive Text2Cypher With Retry and Evaluation Flow을 활용

이 agnet를 사용하기 위해 선행 되어야 하는 것

1. 스키마 구축
2. 데이터 구축
3. 필요하다면 template를 수정해야함
4. 사용하는 GraphDB에 맞게 명령어들 수정
5. few-shot을 위한 질문과 cypher을 만들 필요 있음
    
    (최종답변에서 생성된 Cypher가 자동 저장 되므로, 여러번 다양한 질문으로 실행 시키면 됨)
    

result

retry와 evaluation이 정답 연관도를 높히는 역할을 함.

# **Reference**

https://neo4j.com/blog/developer/knowledge-graph-agents-llamaindex/

[GitHub - tomasonjo-labs/text2cypher_llama_agent: A collection of LlamaIndex Workflows-powered agents that convert natural language to Cypher queries designed to retrieve information from a Neo4j database to answer the question with included benchmark data.](https://github.com/tomasonjo-labs/text2cypher_llama_agent?tab=readme-ov-file)
