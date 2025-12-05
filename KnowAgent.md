- [0. Abstract]
- [1. Introduction]
- [2. Background]
- [3. KnowAgent]
    - [3.1 Definition of Action Knowledge]
    - [3.2 Action Knowledge를 활용한 Planning Path 생성]
    - [3.3 지식 기반 자기학습을 통한 Planning Path 정제]
- [Prompt Format]

# **0. Abstract**

목적:

- Planning을 action knowledge와 통합하여 향상

기존 문제점:

executable actions를 생성하여 상호작용 해야 하는 환경에서 성능 저하

- LLMs의 내장된 action knowledge의 부족
- 테스크를 해결하는 동안 planning trajectores 가이드 실패하고 이로인해 Planning환각 기인함

KnowAgent 제안:

- `action knowledge base` 와 `knowledgeable self-learning strategy` 사용
    
    ← planning 동안 action path를 제약하기 위해
    

실험 결과:

- HoptpotQA, ALFWorld 를 backbone 모델를 기반으로 실험
- KnowAgent가 planning 환각 완화에 효과적이고, 성능도 유사하거나 더 높은 결과 보임

# **1. Introduction**

기존:

planning 능력을 향상 시킨 방법

- task decomposition
- reflection
- collaborative division of labor
- utilization of external tools

문제점:

위 방법으로도 모델의 내제적 이해 능력과 학습한 지식 범위에 의해 planning 능력에 한계가 존재

기존 문제점을 해결 하기 위한 시도들:

- task-specific, trajectories 합성데이터로 fine tuning

시도 방법의 문제점:

executing planning tasks할 때, 특히 open-source모델이 문제가 됨

모델이 knowledge 규칙을 위반하는 Planning 환각이 일어남 → 불필요하거나 충돌나는 계획을 함

KnowAgent 제안:

외부 action knowledge활용을 집중하는 KnowAgent 제안.

이를통해 synthetic trajectories를 개선하고, Planning 환각을 해결을 목표로 함

Figure1:

step1: action planning knowledge을 통합한 광범위한 `action knowledge base`를 구축

action kowledge base는 모델의 외부 저장소 역할을 하며, 모델의 action 생성 프로세스를 가이드한다.

step2: action knowledge를 텍스트로 변환

action trajectories 생성에 활용

step3:knowledgeable self-learning

모델의 반복적 과정 에서 생성된 행동 경로를 사용하여 액션 지식의 이해와 활용 능력을 지속적으로 향상

- HoptpotQA, ALFWorld 를 backbone 모델를 기반으로 실험
- KnowAgent가 planning 환각 완화에 효과적이고, 성능도 유사하거나 더 높은 결과 보임

# **2. Background**

1. Language Agents와 환경 관찰
- 언어 에이전트는 외부 환경을 관찰하기 위해 Thoughts와 실행 가능한 행동Actions을 생성함.
- 본 논문에서는 planning trajectory 형식을 기반으로 KNOWAGENT를 훈련 및 평가.
1. Planning Trajectory 구조

2.1 기본표현

- 하나의 trajectory τ는 다음과 같이 Thought-Action-Observation(T, A, O) 삼중항으로 표현됨:
    - T (Thoughts): 에이전트의 내적 사고
    - A (Actions): 실행 가능한 행동
    - O (Observations): 환경으로부터 받은 피드백
- 시점 t에서의 trajectory history Ht:

H_t = (T_0, A_0, O_0, T_1, ..., T_{t-1}, A_{t-1}, O_{t-1})

2.2 다음 단계 생성

1. Thought(Tt) 생성

에이전트는 이전 히스토리 Ht를 바탕으로 다음 내적 사고 Tt를 생성.

1. Action(At) 생성

생성된 Tt와 히스토리 Ht를 기반으로 행동 At 결정

1. Observation(Ot) 기록
- At의 실행 결과를 Ot로 처리하고 trajectory에 추가
- 새로운 히스토리 Ht+1 생성

# **3. KnowAgent**

Action Knowledge를 정의한 후, 이 지식을 활용해 플래닝 경로를 생성하고,

knowledgeable Self-Learning 메커님을 통해 지속적으로 정제하여 프레임워크를 강화

## **3.1 Definition of Action Knowledge**

**1) Action (행동)**

- Ea = {a₁, ..., aₙ₋₁}
- 특정 작업을 수행하기 위해 LLM이 취해야 하는 action 행동들의 집합
    
    예: *검색하기, 확인하기, 이동하기, 선택하기* 등.
    

---

**2) Action Rules (행동 규칙)**

- R = {r₁, ..., rₙ₋₁}
- 행동들 간 전이 가능 여부와 순서 논리를 정의하는 규칙 집합
- 규칙의 형태:
    
    rₖ: aᵢ → aⱼ
    
    → 특정 행동을 수행한 뒤 어떤 행동이 합법적으로 이어질 수 있는지 결정
    
- 행동 간 관계나 작업-specific 요구사항에 기반.

---

**3) Action Knowledge (행동 지식)**

- (Ea, R) 로 표현
- 구성 요소:
    - 행동 집합(Ea)
    - 행동 간 전이 규칙(R)
- 여러 작업(task)의 행동 지식을 모으면
    
    → Action KB (Action Knowledge Base) 가 됨
    
- Action KB는 계획(planning) 중 환각(hallucination)을 줄이는 핵심 가이드 역할 수행.

---

**4) Extracting Action Knowledge (추출 전략)**

문제점

- 다양한 작업마다 요구되는 행동 지식이 다름
- 완전 수동 구성은 비효율적(시간·노동 많이 듦)

해결책: GPT-4 활용한 2단계 방식

---

**Stage 1 — 초기 생성**

1. 도메인 전문가가 LLM(GPT-4)에 작업 지식 제공
2. LLM이 예비(actions + rules) 목록 생성
3. 해당 목록에는 중복·불필요한 항목이 존재
4. 인간 전문가가 이를 필터링 및 정제

---

**Stage 2 — 최종 규칙 생성**

1. 정제된 행동 목록과 제약 사항을 다시 LLM에게 제공
2. LLM이 최종 행동 규칙 세트 생성

Action Knowledge는 Iteration이 많아질 수록 더 효과가 커짐

→ action knowledge와 self-learning 사이의 선순환으로 추측됨

action knowledge의 제약 아래에서는 모델이 반복 학습을 위해 더 고품질의 trajectory를 합성하게 된다.

## **3.2 Action Knowledge를 활용한 Planning Path 생성**

**3.2.1. Action Knowledge → Text 변환**

LLM으로 task에 필요한 행동들을 식별하여 action knowledge base를 구축 후

이후 단계에서 활용하기 위해 이 정보를 텍스트 형식으로 변환

e.g. HotpotQA 행동 규칙 참고

Search: (Search, Retrieve, Lookup, Finish)

이 규칙은 Search 행동을 수행한 후 이어질 수 있는 여러 경로가 있음을 의미함

즉, Search에서:

- 다시 Search로 계속 진행하거나
- Retrieve로 전환하거나
- Lookup으로 전환하거나
- Finish 단계로 넘어갈 수 있다

---

**3.2.2. Path 생성**

Action knowledge를 활용하여 모델은 작업의 planning 프로세스를 조정함

경로 생성을 위해서 Task Description을 넘어 특수한 프롬프트를 개발함

프롬프트는 action knowledge에 철저히 기반하며 총 네 가지 핵심 구성요소로 이루어짐:

1. Action Knowledge Overview
    - 기반 개념과 규칙을 제시하는 개요 단계
2. Definition of Each Action Step
    - 각 액션이 수행하는 기능과 의미를 상세히 서술
3. Principle of Planning Path Generation
    - 출력 생성 시 따라야 할 제약과 원칙을 정의
4. Demonstrations of Planning Paths
    - 다양한 상황에서 활용할 수 있는 실제 계획 경로 예시 제공

이 구성요소들은 action knowledge를 표현하고, 액션을 명확히 정의하며, 이를 활용한 planning path 생성 과정을 구체적으로 설명하는 데 핵심적이다.

**Path vs Trajectory의 차이**

- **Path(경로)**:
    
    에이전트가 수행하는 **액션들의 순서**만 포함.
    
- **Trajectory(궤적)**:
    
    문제 해결 과정에서 모델이 생성하는 **전체 출력**을 포함하며, 그 안에 path도 포함됨
    
    → 즉 path ⊂ trajectory 관계
    

**Trajectory 생성 과정 개요**

Trajectory τ는 여러 계획된 (P, T, A, O) 네 요소로 이루어진 quadruple의 집합이다.

- P: Action path
- T: 내부 사고(Thought)
- A: 실행 가능한 action
- O: 환경으로부터의 feedback

Trajectory의 history는 다음과 같이 정의함:

H_t = (P_0, T_0, A_0, O_0, …, T_{t-1}, A_{t-1}, O_{t-1})

이 기반 위에서 에이전트는 Pₜ, Tₜ, Aₜ을 생성하게 됨

**Action Path 생성 확률**

파라미터 θ를 가진 확률적 언어 에이전트 π에서, 다음 경로 Pₜ의 생성은 다음과 같이 표현됨:

Thought 및 Action 생성

## **3.3 지식 기반 자기학습을 통한 Planning Path 정제**

knowledgeable self-learning 도입하여 파인튜닝을 통해 action knowledge를 더 깊이 이해하도록 하는 것

Filtiering과 Merging을 통해 trajectory 품질을 향상 시킴

**(1) Filtering(필터링)**

결과를 기준으로 올바른 trajectory를 선별한다.

특히 HotpotQA 같은 작업에서는 action knowledge를 적용해 trajectory를 추가적으로 정제한다.

이 정제 과정에서는 다음과 같은 trajectory들을 제거한다:

- 제공된 AKₘ(행동 지식)과 일치하지 않는 trajectory
- 잘못된 action을 포함하는 trajectory
- 잘못된 순서의 action sequence를 가진 trajectory

---

**(2) Merging(병합)**

그 다음, 서로 다른 iteration에서 생성된 trajectory들을 병합한다.

동일한 task에 대한 여러 trajectory가 존재할 경우:

- 더 효율적인 **trajectory(더 짧은 path) 를 우선적으로 남긴다**
- 이를 통해 최적의 문제 해결 효율성을 보장한다

# **Prompt Format**

path를 생성하기 위해 만든 프롬프트

e.g. Hotpot QA

https://arxiv.org/pdf/2403.03101
