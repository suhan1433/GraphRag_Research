# **Overview**

1. **기존 연구 한계**

LLM + GNN 통합 연구는 주로

- 전통적 그래프 과제 (노드/엣지/그래프 분류)
    - 그래프의 구조적 특징을 바탕으로 예측을 하는 과제
    - e.g. 지식 그래프에서 "A는 B의 무엇인가?" 관계 예측
- 소규모·합성 그래프 질의응답
    - 그래프 구조 자체에 대한 질문을 하는 형태
    - e.g. "노드 A와 B 사이에 경로가 있나요?", "이 노드의 이웃은 누구인가요?"

→ 실제 대규모 텍스트 그래프, 복잡한 응용에는 미흡

따라서) 그래프를 활용한 유연한 question-answering 프레임워크를 만들고자 함

2. **제안 방법: GraphQA & G-Retriever**
- GraphQA 벤치마크 구축
    - 다양한 그래프 기반 질의응답 과제에서 데이터 수집
- G-Retriever 제안
    - 최초의 RAG 기반 텍스트 그래프 QA 프레임워크
    - Soft prompting 기반 파인튜닝으로 성능 향상
3. **핵심 기술**
- 환각 방지
<img width="958" height="567" alt="image" src="https://github.com/user-attachments/assets/40b2cb13-e0a2-49da-ac03-74d3244604f3" />

    - baseline의 경우 User의 모든 그래프 정보를 다 넣고 답변을 하도록함 → 할루시네이션 발생
    - G-Retriever의 경우 RAG방식으로 필요한 정보만을 검색해와 답변을 함 → 할루시네이션 감소
- 그래프에 맞춘 새로운 검색 방법을 제안 → 질문에 관련성 높은 subgraph그래프를 반환하여 설명 향상
    - subgraph 검색을 Prize-Collecting Steiner Tree(PCST) 최적화 문제로 공식화하여 이웃 정보를 고려한 검색이 가능하도록 함
1. **실험 결과**
- 다양한 도메인에서 기존 baseline 대비 성능 향상
- 환각 현상 완화

# **사전지식**

## **Textual Graphs**

Textual Graphs란?

노드와 엣지가 텍스트 속성을 가지는 그래프
<img width="588" height="88" alt="image" src="https://github.com/user-attachments/assets/91fc3826-bbf1-4c3c-bb70-ba8289e9fe0d" />


x_n: 노드에 있는 텍스트 속성

x_e: 엣지에 있는 텍스트 속성

<img width="900" height="248" alt="image" src="https://github.com/user-attachments/assets/1ab6a3f3-e14c-4670-b013-da9a1fd14f39" />


D: vocabulary

L_n, L_e : 텍스트의 길이

e.g.Textual Graph

<img width="435" height="158" alt="image" src="https://github.com/user-attachments/assets/77c3a98f-788b-4c08-97d0-a188b01ccf3b" />




## **LMs for Text Encoding**

LMs의 사용:

노드와 엣지 text 속성을 인코딩하기 위한 용도

<img width="410" height="58" alt="image" src="https://github.com/user-attachments/assets/8cf79a2a-841b-4e01-93e8-4c7bc5e1515f" />


x_n: 노드의 텍스트 속성

z_n: LM으로 x_n을 encode 인코딩하여 d 차원의 output vector

## **LLMs and Prompt Tuning**

**SoftPromptTuning이란?**

프롬프트를 “문자열”이 아니라 “학습 가능한 벡터”로 만들어,

LLM을 finetuning 없이 특정 작업에 적응시키는 방법

Prompt(embedding) + Input sequence(embedding) → generated output y

<img width="1314" height="353" alt="image" src="https://github.com/user-attachments/assets/07aecac4-ccf2-41d9-a88b-304c9444e1d7" />

<img width="770" height="142" alt="image" src="https://github.com/user-attachments/assets/742b1519-b5f0-4db9-8d9d-d0f05fc516cc" />


P: Prompt

X: sequence of tokens

p_theta:  LLM모델

### **SoftPromptTuning 방식 예제**

**Step1. 입력 문장 임베딩**

```
X = ["I", "love", "pizza"]
```

- 토큰 개수 3인 경우 가정,
- 각 토큰은 LLM임베딩 레이어를 통해 임베딩 차원 d_l = 4짜리 벡터로 바꿈

<img width="636" height="162" alt="image" src="https://github.com/user-attachments/assets/9c39cfa7-eeaa-4c55-b327-346994686459" />


**Step2. SoftPrompting**

프롬프트 길이를 q=2인 경우 가정,

프롬프트 파라미터는 GNN인코더를 통해 임베딩으로 변환

<img width="192" height="50" alt="image" src="https://github.com/user-attachments/assets/971a5ccf-8c21-4a50-9bd5-f87677bc4349" />

<img width="480" height="122" alt="image" src="https://github.com/user-attachments/assets/38185a85-a576-4e1a-8eb1-d394d54b2b14" />


**Step3. Concat**

이제 두 행렬을 이어 붙임:

<img width="696" height="242" alt="image" src="https://github.com/user-attachments/assets/c17fc728-e1f0-4f2f-aebc-47f39f8cd3ae" />


즉, 원래 입력 `"I love pizza"` 앞에 두 개의 가짜 토큰(소프트 프롬프트가 붙음)

**Step4. 모델 학습**

1. 정답 문장을 맞추도록 likelihood최대화
2. LLM 모델 파라미터 theta는 고정
3. 오직 P_e만 업데이트

# **GraphQA bechmark**

3개의 데이터셋이 존재 ( Explanation Graph, Scene Graph, Knowledge Grpah )

Question : 그래프의 특정 요소 또는 관계를 탐구하도록 설계

Answer : 답변은 노드 또는 엣지의 속성안에 존재, 이를 추론 하기위해 multi-hop추론이 필요

<img width="1231" height="442" alt="image" src="https://github.com/user-attachments/assets/cf2bf29e-b662-40e3-a38b-7c3ab24ee7bc" />


1. **Explanation Graph**
- 토론에서 입장을 예측하기 위해 expantion graph를 생성하는 데이터셋
- 주장이 supportive, contradictory인지 평가하는게 주 목적이며, 정확도를 지표로 사용

<img width="1633" height="161" alt="image" src="https://github.com/user-attachments/assets/3b0f2a9c-6a6a-4d28-bed4-e6f486feb560" />


2. **Scene Graph**
- 100,000개의 시각적 질의응답 데이터셋(vqa)
- a scene graph의 텍스트 설명에 기반해 질문을 하는것. 정확도를 지표로 사용

<img width="1650" height="926" alt="image" src="https://github.com/user-attachments/assets/3ddcd9f4-548c-4ce1-860c-5f8523e59f15" />

3. **Knowledge Grpah**
- 4.737개의 질문으로 구성된 multi hop 지식 그래프 QA데이터셋
- 질문에 언급된 entity를 중심으로 2-hop이내에 사실을 답할 수 있도록 구성

<img width="1657" height="531" alt="image" src="https://github.com/user-attachments/assets/769e080d-ee20-4d43-8827-bfea821d9745" />


# **G-retriever**

<img width="1334" height="470" alt="image" src="https://github.com/user-attachments/assets/87e86bf7-62c8-480a-9a3f-f841597c87a5" />


## **Step1. Indexing**

1. 노드와 엣지의 텍스트 속성을 LM을 활용하여 임베딩

<img width="291" height="48" alt="image" src="https://github.com/user-attachments/assets/5c2915fe-b8e3-4921-b47a-29b5154082ee" />


x_n : 노드의 텍스트 속성 정보

| **I/O** | **type** | **E.g(Explanation Graph)** |
| --- | --- | --- |
| Input | str | node_attr: cannabis,
node_attr: marijuana,
edge_attr: causes,
edge_attr: capable of |
| Output | float32 | node, edge 속성 임베딩 |

## **Step2. Retrieval**

1. Query를 LM으로 임베딩
2. KNN 방식으로 Query와 관련된 노드를 topk 검색
3. KNN 방식으로 Query와 관련된 엣지를 topk 검색

<img width="419" height="105" alt="image" src="https://github.com/user-attachments/assets/0d6dbefe-5a5f-4c3b-96c7-a9ea233c790c" />


z_n : 노드 속성 임베딩

z_e : 엣지 속성 임베딩

| **I/O** | **type** | **E.g** |
| --- | --- | --- |
| Input | float32 | node, edge 속성 임베딩 |
| Output | float32 | 검색된 topk개의 
node, edge임베딩 |

## **Step3. Subgraph Construction**

**목표:**

관리 가능한 그래프 사이즈를 유지하면서,

가능한 많은 관련 노드와 엣지를 연결된 Subgraph를 추출하는 것

두가지 주요 장점 제공

1. 쿼리와 무관한 노드와 엣지를 걸러낼 수 있음
2. 효율성을 높임
    - 최적의 그래프 크기와 query와의 관련성을 만족하는 Subgraph를 찾기위해
        
        → Prize-Collecting Steiner Tree(PCST)를 사용
        
    - 아래는 전체 그래프를 넣는 경우와 PCST를 통한 Subgraph를 넣는 경우 비교 결과
  
<img width="820" height="179" alt="image" src="https://github.com/user-attachments/assets/691b2232-58c2-45e8-ab89-ad3982a9ebb0" />


| **I/O** | **type** | **E.g** |
| --- | --- | --- |
| Input | float32 | 검색된 topk개의 
node, edge임베딩 |
| Output | torch_geometric.data.Data | Subgraph로
GNN인코더에 넣을 수 있는 Data |

### **PCST(Prize-Collecting Steiner Tree)**

**정의:**

각 노드가 가진 “보상(prize)”과 엣지의 “비용(cost)”을 고려하여,

전체 이익(보상 − 비용) 이 최대가 되는 트리를 찾는 문제

- Query와 더 관련되어 있는 노드와 엣지에 높은 보상을 부여
- topk 노드,엣지를 내림차순으로 k에서 1 순으로 보상을 할당 나머지는 0

아래 방식으로 노드와 엣지의 보상을 할당

<img width="788" height="102" alt="image" src="https://github.com/user-attachments/assets/65fa20e3-85bf-49ce-a677-d814c6b7ac24" />


최종적으로 찾고자 하는 Subgraph: S^* = (V^*, E^*)는 아래를 따름

<img width="1088" height="195" alt="image" src="https://github.com/user-attachments/assets/6435948c-f804-4d67-aec0-a7879984c62f" />


C_e: 엣지 하나당 사전에 정의된 비용, Subgraph의 크기를 조정하는데 사용

**논문에서 PCST 알고리즘 수정한 부분**

**기존** :

노드 보상만 고려함. 하지만 특정 상황에서 엣지의 의미도 중요함.

**수정** :

엣지의 보상도 추가 함.

**문제점**:

엣지의 비용 C_e보다 엣지의 보상이 큰 경우 문제가 생김.

**문제상황**:

C_e: 1.5, prize(e):2

1. C_e > P_e → C_e - P_e라는 축소된 비용으로 처리
    
    알고리즘상에 엣지를 계산하는 곳이 없기에 엣지 비용에서 prize(e)를 빼야함.
    
2. C_e < P_e → 원래 알고리즘은 음수 비용을 허용하지 않아 직접 적용할 수 없음
    
    1.5-2 = -0.5, 이 경우 PCST알고리즘에서는 C_e>0을 가정으로 하기 때문에 에러가 나옴
    

**해결방법:**

엣지 e를 virtual node v_e로 대체

1. v_e는 엣지 양 끝점과 연결되며,
2. v_e에 P_e - C_e라는 보상을 부여하고,
3. v_e로 이어지는 두 새 엣지의 비용은 0으로 설정한다.

```
# origin: 엣지 e가 노드 u와 v를 직접 연결
u ──(C_e, P_e)── v
# add virtual node
        v_e (reward = P_e - C_e)
       /   \
  (0) /     \ (0)
     u       v

```

### **PCST 예시**

Query : “Einstein은 어떤 상을 받았나요?”

Knowledge Graph:

```
노드들:
- Einstein (물리학자)
- Nobel Prize (상)
- Physics (학문)
- Germany (국가)
- Relativity Theory (이론)
- Marie Curie (물리학자)
- Chemistry (학문)
엣지들:
- Einstein --[won]--> Nobel Prize
- Einstein --[developed]--> Relativity Theory
- Einstein --[born_in]--> Germany
- Nobel Prize --[category]--> Physics
- Marie Curie --[won]--> Nobel Prize
- Nobel Prize --[category]--> Chemistry
```

**Step1.** 관련성 점수 계산 (cos sim)

1. 질문과 관련된 노드와 엣지 top-k개 선택
2. 내림차순으로 k부터1까지 보상, 나머지는 0

```
노드 유사도 (상위 k=4개만 선택):
Einstein: 0.95 → prize = 4
Nobel Prize: 0.88 → prize = 3
Physics: 0.65 → prize = 2
Relativity Theory: 0.60 → prize = 1
Germany: 0.30 → prize = 0
Marie Curie: 0.25 → prize = 0
Chemistry: 0.20 → prize = 0
엣지 유사도 (상위 k=3개만 선택):
Einstein --[won]--> Nobel Prize: 0.92 → prize = 3
Nobel Prize --[category]--> Physics: 0.70 → prize = 2
Einstein --[developed]--> Relativity Theory: 0.55 → prize = 1
나머지 엣지들: prize = 0
```

**Step2.** PCST 최적화

```
최대화: (노드 prize 합) + (엣지 prize 합) - (엣지 개수 × Ce)
Ce (엣지 당 비용) = 1.5 로 설정했다고 가정
```

후보 Subgraph비교:

- Einstein + Nobel Prize만 연결

```
노드: Einstein(4) + Nobel Prize(3) = 7
엣지: won(3) = 3
비용: 1개 엣지 × 1.5 = -1.5
총점: 7 + 3 - 1.5 = 8.5 ✓
```

- Einstein + Nobel Prize + Physics 연결

```
노드: Einstein(4) + Nobel Prize(3) + Physics(2) = 9
엣지: won(3) + category(2) = 5
비용: 2개 엣지 × 1.5 = -3
총점: 9 + 5 - 3 = 11 ✓✓ (최고!)
```

- 모든 노드 포함

```
노드: 4+3+2+1+0+0+0 = 10
엣지: 3+2+1+0+0+0 = 6
비용: 6개 엣지 × 1.5 = -9
총점: 10 + 6 - 9 = 7 (비효율적, 크기만 커짐)
```

**실제로 PCST 알고리즘 사용 시 트릭**

엣지 prize가 Cost보다 클 때, Virtual Node 트릭 사용

문제:

```
Einstein --[won]--> Nobel Prize
엣지 비용(Ce) = 1.5
엣지 prize(Pe) = 5
Pe > Ce 이므로 음수 비용이 됨 → 알고리즘이 처리 못함
```

해결:

```
원래:
Einstein --[won(cost=1.5, prize=5)]--> Nobel Prize
변환 후:
Einstein --[cost=0]--> [Virtual_won(prize=3.5)] --[cost=0]--> Nobel Prize
```

## **Step4. Generation**

4가지 과정을 거쳐 Generaton이 됨

1. Subgraph Graph Encdoer 인코딩
2. 인코딩 된 임베딩을 Projection Layer를 거쳐 LLM 차원에 맞춤
3. Text Embedder(LLM의 임베딩 레이어)을 통해 Subgraph text형식을 임베딩
4. 2번과 3번 결과 임베딩을 concat 하여 LLM이 답변 생성

| **I/O** | **type** | **E.g** |
| --- | --- | --- |
| Input | torch_geometric.data.Data | Subgraph로
GNN인코더에 넣을 수 있는 Data |
| Output | str | 최종 답변 |

### **과정**

1. **Graph Encoder**

Subgraph S∗=(V∗,E∗)를  Graph Encoder(GNN)을 사용함

이때, GNN으로 mean pooling을 진행

<img width="469" height="54" alt="image" src="https://github.com/user-attachments/assets/36f74321-4d81-47c4-bc09-719d59f501c1" />


d_g : GNN Dim

2. **Projection Layer**

MLP로 그래프 토큰을 LLM vector space로 정렬하기 위해 사용

<img width="346" height="62" alt="image" src="https://github.com/user-attachments/assets/9cee14b1-09dd-4ae0-a572-54a252312c17" />


d_l: LLM Dim

3. **Text Embedder**

**Step1.** Subgraph textualize

LLM의 텍스트 추론 능력을 향상 시키기 위해 Subgraph의 텍스트 형식 변환

이 과정은 노드와 엣지의 텍스트 속성을 flatten하는 과정이며, textualize라고 부름

**Step2.** Query와 결함

Query x_q와 textualize Subgraph를 합침

**Step3.** TextEmbedder

frozen LLM의 첫번째 레이어인 text embedding layer을 통해 h_t로 변환

<img width="703" height="58" alt="image" src="https://github.com/user-attachments/assets/b1f5676d-5f09-4aa2-87b3-c5eea332d73e" />


L : 토큰의 수

; : concat

**Step4**. LLM Generation with Graph Prompt Tuning

Softprompt 역할하는 그래프 토큰 h_g^hat과 h_t를 concat

이 때, LLM 파라미터 theata는 고정이며,

Graph Encoder phi_1 와 Projection Layer phi_2의 파라미터는 역전파를 통해 최적화할 수 있도록 함

<img width="696" height="108" alt="image" src="https://github.com/user-attachments/assets/4dea58f4-3c74-4113-bad4-7c75d72421d8" />


## **Inference Flow**

<img width="2057" height="913" alt="image" src="https://github.com/user-attachments/assets/d529ab2a-0af9-42be-99c8-2f3d94b5f5ab" />


SoftPrompting 과정(파란색)

1. 사용자 Query로 node/edge Topk개 검색
2. 검색된 Topk개의 node/edge을 PCST 활용하여 Subgraph생성
3. Subgraph를 GNN으로 인코딩
4. GNN 임베딩을 MLP를 거쳐 한번더 임베딩

Text embedding 과정(초록색)

1. 검색된 Subgraph를 textualize한 설명과 사용자 질문을 합친 후
2. LLM의 텍스트 임베딩 레이어를 통해 임베딩

LLM 응답 과정(검정)

1. concat(GNN임베딩, Text임베딩)
2. LLM을 통해 답변 생성

# **실험 결과**

## **성능 비교**

<img width="896" height="357" alt="image" src="https://github.com/user-attachments/assets/a51bc87e-3cc9-4346-8b56-d33a5c581427" />


- **Inference-only:** frozen된 LLM을 사용하여 직접 질의응답을 수행하는 방식

<img width="762" height="333" alt="image" src="https://github.com/user-attachments/assets/3ff1544d-22dd-4889-bb38-494093b344ab" />


- **Frozen LLM w/ Prompt Tuning (PT):** LLM의 파라미터는 고정한 채, 프롬프트만 조정하는 방식

<img width="1297" height="327" alt="image" src="https://github.com/user-attachments/assets/2df09ff0-7514-4355-a69e-5d3541a2bba1" />


- **Tuned LLM:** LoRA 를 이용해 LLM 자체를 fine-tuning하는 방식

<img width="1313" height="334" alt="image" src="https://github.com/user-attachments/assets/eee5f606-fc43-44e6-b935-4bea9f43ef3b" />


최고 성능은 Bold, 두번 째 성은 밑줄로 표시

1. Inference-only 설정에서는, G-Retriever가 모든 베이스라인을 능가함.
    
    graph knowledge이 제공되지 않은 경우(즉, 질문만 주어진 경우)
    
    오히려 LLM이 더 나은 성능을 보이기도 하는데,
    
    그래프 지식이 오히려 복잡성이나 잡음(noise) 을 유발했기 때문으로 보임

<img width="458" height="178" alt="image" src="https://github.com/user-attachments/assets/ddd84218-89be-4437-b313-17fa9bb4d65e" />

    

| 방식 | 핵심 아이디어 | 프롬프트 예시 | 특징 / 목적 |
| --- | --- | --- | --- |
| **Zero-shot** | 텍스트로 된 그래프 설명 + 작업 설명만 주고, 추가 예시 없이 즉시 출력하도록 함 | “Given the following graph, answer the question.” | 전혀 학습 없이 즉각적 추론. 그래프를 단순한 텍스트 정보로만 이용 |
| **Zero-CoT** | Zero-shot에 “Let’s think step by step.” 문장을 추가하여 단계적 사고 유도 | “Question: … Let’s think step by step.” | LLM이 연쇄적 추론(Chain of Thought)을 수행하도록 유도 |
| **CoT-BAG** (Build-a-Graph Prompting) | 그래프를 먼저 구성하도록 지시한 뒤 문제를 풀게 함 | “Let’s construct a graph with the nodes and edges first.” | 그래프 구조를 먼저 인식하게 하여 관계 기반 추론 강화 |
| **KAPING** | 그래프 지식베이스에서 관련 트리플(triple)을 검색해 질문 앞에 붙임 | “(triples) + Question: …” | 질문과 관련된 지식을 사전 검색 후, 컨텍스트로 활용하여 정확도 향상 |
1. Frozen LLM + Prompt Tuning 설정에서는,
    
    G-Retriever가 기존의 일반적인 Prompt Tuning 및
    
    GraphToken (그래프 기반 프롬프트 튜닝 기법)보다 각각
    
    평균 40.6% 및 30.8% 더 높은 성능을 달성
    
2. LoRA를 이용해 LLM을 미세조정한 경우,
    
    G-Retriever는 모든 설정 중 가장 높은 성능을 기록
    

## **Ablation study**

<img width="2195" height="505" alt="image" src="https://github.com/user-attachments/assets/7ca575c9-0307-4f0f-b895-96ad2fdd1843" />

<img width="535" height="218" alt="image" src="https://github.com/user-attachments/assets/cda83c56-e68c-4930-839b-56bf9aa3491e" />


어떤 요소를 제거하든 성능 저하가 발생

1. Graph Encoder 제거 : 22.51% 하락
2. Projcetion Layer 제거 : 1.11% 하락
3. Textualized Graph 제거 : 19.19% 하락

# **실행**

## **Step1. Preprocess**

1. **Graph Textualize**

raw data:

```
arg1	arg2	label	graph
Cannabis should be legal.	It's not a bad thing to make marijuana more available.	support	(cannabis; synonym of; marijuana)(legal; causes; more available)(marijuana; capable of; good thing)(good thing; desires; legal)
```

노드와 엣지 속성 텍스트화 진행 후 csv로 저장

```
node_id,node_attr
0,cannabis
1,marijuana
2,legal
3,more available
4,good thing
```

```
src,edge_attr,dst
0,synonym of,1
2,causes,3
1,capable of,4
4,desires,2
```

## **Step2. Indexing**

node_attr, edge_attr 임베딩후 pt로 저장

## **Step3. Retrieval**

1. Question Embedding
2. Node, Edge topk 검색
3. PCST알고리즘으로 Subgraph 형성 후 Description, Graph 반환

e.g. Subgraph Description

```
node_id,node_attr
15,justin bieber
151,pattie mallette
173,m.0gxnp5d
286,english language
294,jaxon bieber
346,yves bole
356,jeremy bieber
452,jazmyn bieber
545,m.0wfn4pm
551,m.0gxnnwp
610,m.0gxnp5x
933,m.0gxnnwc
1032,this is justin bieber
1359,m.0129jzth
1567,m.0gxnp5m
src,edge_attr,dst
346,people.person.languages,286
1032,film.film.language,286
346,influence.influence_node.influenced_by,15
151,people.person.children,15
294,people.person.parents,356
545,people.sibling_relationship.sibling,151
294,people.person.sibling_s,551
15,base.popstra.celebrity.hangout,610
933,people.sibling_relationship.sibling,452
1359,people.sibling_relationship.sibling,346
151,people.person.sibling_s,545
551,people.sibling_relationship.sibling,294
346,people.person.sibling_s,1359
15,people.person.sibling_s,933
15,base.popstra.celebrity.hangout,173
452,people.person.sibling_s,933
933,people.sibling_relationship.sibling,15
15,people.person.sibling_s,551
15,base.popstra.celebrity.hangout,1567
551,people.sibling_relationship.sibling,15

```

e.g. Graph

GNN에 입력 될 수 있는 torch_geometric.data.Data로 저장됨

Data(x=[노드개수, 1024], edge_index=[시작노드, 도착노드], edge_attr=[엣지개수, 1024], num_nodes=노드개수)

## **Step4. Generation**

SoftPrompting

1. GNN에 Subgraph Data를 입력
2. 출력 결과를 MLP로 차원 변환

Text Embedding

1. Descrition과 Question text를 합침
2. 합친 text를 LLM의 임베딩 레이어를 통해 임베딩

Generation

1. Soft Prompt + Text Embedding을 LLM을 통해 답변 생성

# **결론**

GraphDB에 적용해서 사용한다면, PCST알고리즘과 어떻게 연결 시켜서 빠르게 답변 하게 할지 고민해야 할 듯.

[neo4j PCST](https://neo4j.com/docs/graph-data-science/current/algorithms/prize-collecting-steiner-tree/) : 이 경우는 미리 Prize를 주는 경우로 고정적임. 우린 매번 검색 해오고 그거를 토대로 Prize를 줘야함.

**시도할만한 것**

1. 그래프 인코더 종류 바꾸기

GNN에 따라 robust 다름, PyG 모듈에서 지원하기에 바꿔서 해보면 될듯

<img width="581" height="174" alt="image" src="https://github.com/user-attachments/assets/bc475a95-acb0-4f0a-a039-72fafdba560e" />


2. LLM 모델 변경

<img width="443" height="117" alt="image" src="https://github.com/user-attachments/assets/1c34eeb0-249e-4f2d-8078-1f1f7a7ac8f1" />
