- 1. Overview
- 2. Agent Methodology
    - 2.1 Construction
    - 2.1.1 Profile Definition
2.1.2 Memory Mechanism
2.1.3 Planning Capability
2.1.4 Action Excution
2.2 Agent Collaboration
2.2.1 Centralized Control
2.2.2 Decentralized Collaboration
2.2.3 Hybrid Architecture
2.3 Agent Evolution
2.3.1 Autonomous optimization and self-learning
2.3.2 Multi-Agent Co-Evolution
2.3.3 Evolution via External Resources
---

Awsome-Agent-Paper의 Agent Methodolgy 개념을 소개하고자 작성 했습니다.

필요한 개념의 구현 방법은  Fig2의 관련 논문들을 참고 바랍니다.

Survey논문자체가 개념들을 추상적으로 작성되어 있어, 이런게 있구나 정도로 보시면 될 것 같습니다.

문서 작성할 때 왜곡을 없애려고 최대한 논문에서 사용한 표현 그대로 사용했습니다.

이 링크로 [github](https://github.com/luo-junyu/Awesome-Agent-Papers) , [paper](https://arxiv.org/pdf/2503.21460)를 확인 가능합니다.

# **1. Overview**

1. **Agent Methodology**(Agent 방법론)는 구성, 협업, 진화의 기초적 측면을 다루며
2. **Evaluation and Tools**(평가 및 도구)는 벤치마크, 평가 프레임워크, 개발 도구를 제시하고
3. **Real-World Issues**(현실 세계 문제)는 보안, 개인정보 보호, 사회적 영향과 관련된 핵심 이슈를 다루며
4. **Applications**(응용)는 LLM 에이전트가 적용되고 있는 다양한 영역을 강조한다.

# **2. Agent Methodology**

Construction, Collaboration, Evolution 3가지 방법에 대해 소개한다.

아래그림은 이 3가지 방법들에 대해 살펴볼 수 있으며, 각 방법에 대한 논문을 확인할 수 있다.

Construction(2.1)에서는 Profile Definition, Memory Mechanism, Planning Capability, Action Excution을 포함한 구성요소를 설정한다.

Collaboration(2.2)에서는 다수의 Agent의 Centralized Control, Decentralized Collaboration, Hybrid Achitecture 방법을 다룬다.

Evolution(2.3)에서는 Autonomous Optimization and Self-Learning, Multi-Agent-Co-Evolution, Evolution via Extenal Resources를 통해 시간에 따라 Agent 성능을 향상 시킨 방법을 다룬다.

## **2.1 Construction**

Agent의 Construction은 LLM기반 자율 시스템 개발의 기초 단계로,

목표 지향적인 행동을 가능하도록 하는 핵심 구성 요소를 체계적으로 설계하는 과정을 포함한다.

아래 4가지는 핵심 요소들이며, Agent Construction에서 가장 중요한 부분이다

- Profile definition
- Memory mechanism
- Planning capability
- Action execution

이 구성 요소들은 recursive optimization loop을 형성한다

즉, 메모리가 계획 수립에 정보를 제공하고, 실행 결과가 메모리를 업데이트하며, 컨텍스트 피드백이 Agent 프로필을 정교화하는 구조이다.

---

### **2.1.1 Profile Definition**

Profile Definition이란?

Agent의 내재적 속성과 행동 패턴을 설정하여 운영상의 정체성(operational identity)을 확립하는 과정이다.

두가지 방법론을 소개한다.

- Human-Curated Static Profiles
- Batch-Generated Dynamic Profiles

**Human-Curated Static Profiles**

개념: 도메인 전문가가 에이전트의 역할, 행동 규칙, 책임을 명시적으로 설계하고 고정합니다.

- 사전 정의된 행동 지침과 업무 요구 사항을 엄격히 준수
- 에이전트 간 표준화된 통신 프로토콜 보장
- 높은 해석 가능성(interpretability)과 규제 준수(regulatory compliance)가 요구되는 상황에서 특히 효과적

이러한 프레임워크는 일반적으로 사전 정의된 에이전트 구성 요소 간 조정된 상호작용(coordinated interactions)을 통해, 구조화된 통신 패턴(structured communication patterns)으로 복잡한 기능을 수행한다.

e.g.

- Camel, AutoGen , OpenAgents: 사전 정의된 대화 역할(예: 사용자 프록시, 어시스턴트)을 통해 인간-에이전트 협업을 조율, 구조화된 대화를 통해 작업 수행
- MetaGPT, ChatDev, AFlow: 역할 기반(role-based) 조정 패턴 시연
    - ChatDev: 정적 기술 역할(예: 제품 관리자, 프로그래머)을 협력하여 코드 개발 수행, 결정적(interaction protocols) 상호작용
    - MetaGPT, AFlow: 구조화된 역할 조정을 통해 일반적인 작업 해결로 확장

**Batch-Generated Dynamic Profiles**

개념: 파라미터 조정을 통해 다양한 특성을 가진 Agent 집단을 자동 생성합니다.

- 에이전트 생성 과정에서 성격 특성, 지식 배경, 가치 체계에 통제된 변화를 주입
    
    (e.g. 템플릿 기반 프롬프트, 잠재 공간 샘플링)
    
- 이질적 집단(heterogeneous populations) 생성
    
    → 복잡한 사회적 역학(complex social dynamics) 표현 가능
    
- 인간-에이전트 상호작용을 현실적으로 시뮬레이션할 때 필수
    
    → 사회적 행동 연구, 집단 지능 시뮬레이션 등
    

e.g.제품 테스트 시뮬레이션

```
# 다양한 고객 페르소나 생성
templates = [
    "당신은 {age}세 {occupation}입니다. {income_level} 소득이며,
     제품 선택 시 {priority}를 가장 중요하게 생각합니다.",
]
→ 에이전트 A: 28세 디자이너, 심미성 중시
→ 에이전트 B: 55세 회계사, 가성비 중시
→ 에이전트 C: 35세 개발자, 기능성 중시
```

---

### **2.1.2 Memory Mechanism**

Memory Mechanism이란?

Agent가 시간적 차원에 걸쳐 정보를 저장, 구성, 검색할 수 있는 능력을 주는 역할을 한다.

- Short-term memory
- Long-term memory
- Knowledge Retrieval

**Short-Term Memory**

역할: dialog histories와 environmental feedback을 유지하여, context-sensitive task execution을 지원한다.

상호작용적 교환(interactive exchanges)을 통한 세부적 추론(detailed reasoning) 가능

- 한계:
    - intermediate reasoning trace는 과제 완료 후 소멸하며, 새 시나리오에 직접 전이 불가
    - LLM의 context window 제한 때문에, 실제 구현 시 정보 압축(e.g.: 요약, 선택적 보존) 필요
    - Multiturn interaction의 깊이를 제한하지 않으면 성능 저하 발생

e.g.

- ReAct : 반영적 사고(reflection)를 통한 사고
- ChatDev: 소프트웨어 개발
- Graph of Thoughts: 정교한 문제 해결
- AFlow: 워크플로우 자동화

**Long-term memory**

역할: intermediate reasoning trajectories을 미래에 재사용 가능한 도구로 체계적으로 보관한다.

**Knowledge Retrieval**

역할: Agent가 외부 지식 저장소를 생성과정에 통합을 통해 Agent가 접근 가능한 정보 경계를 확장

---

### **2.1.3 Planning Capability**

Planning Capability이란?

실제 응용 환경에서 Agent가 다양하고 복잡한 작업과 상황을 처리할 수 있도록 효과적인 계획을 세우는 능력이다.

계획 능력은 두가지 관점에서 볼 수 있다.

- task decomposition
- feedback-driven iteration

---

**Task Decomposition Strategies**

개념: 복잡한 문제를 관리 가능한 하위 과업으로 나누어 LLM의 계획 능력을 향상시키는 기본적인 접근 방식

작업 분해 전략은 두가지로 나눈다.

- single-path chaining
- multi-path tree expansion

**single-path chaining**

개념: Agent에게 하위 과업들로 구성된 계획을 세우도록 요청 후, Agent는 주어진 순서대로 하위 과업을 수행

이런 plan-and-solve 직관적이지만 아래와 같은 문제점이 존재한다.

- 유연성 부족
- chaining과정에서 오류 누적
- 문제 해결 과정 동안 사전 정의된 계획을 벗어날 수 없음

→ 따라서 이 문제점을 해결 하기 위해 일부 연구는

1. Dynamic planning: Agent가 현재 상황을 기반으로 다음 하위 과업만 동적으로 생성

2. Multiple Chain of thought: 여러개의 CoT결과를 사용하기에 앙상블 효과 있음

**multi-path tree expansion**

개념: Agent가 계획을 Tree구조로 사용한다.

장점:

Agent가 여러 reasoing 경로를 탐색할 수 있고,

피드백을 기반으로 backtracking도 가능하다.

→ 이 방식은 LLM이 이전 상태로 되돌아갈 수 있게 하며, 잘못된 결정을 수정하고 trial → error → correct 과정이 필요한 복잡한 작업에 적용할 수 있다.

---

**Feedback-driven iteration**

Agent가 feedback을 개선에 활용하여 시간이 지날수록 성능을 향상시킨다.

feedback의 종류

- environmental feedback: Agent가 작동하는 환경으로부터 직접 받는 신호

```
generated_code = agent.write_code("리스트 정렬 함수")
test_results = run_tests(generated_code)
→ Environmental Feedback:
  Test 1: PASS (정확도 100%)
  Test 2: FAIL (시간 초과)
  Test 3: FAIL (메모리 오버플로우)
```

- human feedback: 사용자 상호작용 또는 사전에 준비된 라벨링 데이터에서의 정보
- model introspection: Agent 자체가 생성하는 피드백
- multi-agent collaboration:여러 Agent들과 함께 문제를 해결하며 서로 피드백을 제공한다

→ 이러한 피드백들은 Agent의 성능을 평가하고 계획을 갱신, 추론 경로 조정, 또는 목표 수정 등에 활용한다.

---

### **2.1.4 Action Excution**

LLM이 계획 능력이 있다면, 실제 세계에서 게획된 행동으로 실행하는 것도 Agent의 핵심 능력 중 하나다.

Action Excution은 두가지 측면으로 구성된다

- Tool utilization
- Physical Interaction

**Tool utilization**

tool-use decision과 tool selection 측면이 존재한다.

- tool-use decision
    
    문제를 해결하기 위해 도구를 사용할지 판단하는 과정이다.
    
    Agent가 자신이 생성한 내용에 대한 확신이 낮거나 특정 tool이 필요한 문제를 마주쳤을 때 도구 사용 여부 결정한다.
    
- tool selection
    
    tool 선택과정에서는 사용 가능한 도구에 대한 이해와 현재 Agent의 상황 이해가 필요하다.
    
    e.g. Yuan et al는 tool 문서를 단순화하여 Agent가 도구를 더 잘 이해하도록 하여 정확한 tool 선택 가능하게 했다.
    

---

## **2.2 Agent Collaboration**

효과적인 협업은 Agent들이 distributed intelligence을 활용하고, 행동을 조율하며, 다중 Agent 상호작용을 통해 의사결정을 정교화할 수 있다.

기존의 협업 패러다임을 3가지 아키텍처로 구분하였다.

- Centralized Control
- Decentralized Collaboration
- Hybrid Architecture

---

### **2.2.1 Centralized Control**

개념:

중앙 컨트롤러가 작업 할당과 의사결정 통합을 통해 Agent 활동을 조직하는 계층적 조정 구조를 사용한다.

이때 다른 Sub Agent들은 오직 중앙 컨트롤러와만 소통할 수 있다.

두 가지 구형 방식으로 나뉜다.

- Explicit Controller Systems
- Differentiation-based Systems

**Explicit Controller Systems**

별도의 조정 모듈(종종 독립된 LLM Agent 구현됨)을 사용해 작업을 분해하고 하위 목표를 할당한다.

```
┌─────────────────────────┐
│   중앙 컨트롤러            │ ← 독립적으로 존재
│  (별도 LLM/사람/모듈)    │
└────────┬────────────────┘
         │ 명령/할당
    ┌────┼────┬────┐
    ↓    ↓    ↓    ↓
  Agent Agent Agent Agent
   A     B     C     D
```

**Differentiation-based Systems**

프롬프트를 이용해 메타 에이전트가 서로 다른 하위 역할을 맡도록 유도함으로써 중앙 집중 제어를 달성한다.

```
┌───────────────────────────────┐
│   메타 에이전트 (단일 LLM)         │
│                               │
│  내부적으로 역할 전환:             │
│  "나는 지금 Plan Agent"         │
│  "나는 지금 Tool Agent"         │
│  "나는 지금 Reflect Agent"      │
└───────────────────────────────┘
```

| 구분 | 시스템/논문 | 중앙 컨트롤러 역할 | 특징 / 구현 방식 | 활용 사례 / 효과 |
| --- | --- | --- | --- | --- |
| **Explicit Controller Systems** | **Coscientist** | 인간 운영자(human operator)가 중앙 컨트롤러 | 실험 단계별로 전문화된 에이전트/도구 할당, 최종 실행 계획 직접 통제 | 과학 실험 워크플로우 표준화, 단계별 관리 |
|  | **LLM-Blender** | 별도 컨트롤러 LLM | Cross-attention encoder로 응답 쌍 비교 → 상위 응답 통합 | 응답의 장점 강화, 단점 완화, 최적 출력 생성 |
|  | **MetaGPT** | 전문화된 매니저를 중앙에서 직접 배치 | 실제 소프트웨어 개발 워크플로우 시뮬레이션, 기능별 역할/단계 관리 | 소프트웨어 개발 단계별 통합 및 관리 |
| **Differentiation-based Systems** | **AutoAct** | 하나의 메타 에이전트가 3개의 서브 에이전트(plan/tool/reflect)로 분화 | 복잡한 ScienceQA 과제 분해 | 복합 과제 수행 시 단계별 전문화 및 효율적 처리 |
|  | **Meta-Prompting** | 단일 모델이 코디네이터 역할 | 메타 프롬프트를 통해 과제를 도메인별 하위 과제로 분해, 하위 에이전트에 동적 할당 | 중간 출력 통합 → 최종 솔루션 생성 |

---

### **2.2.2 Decentralized Collaboration**

Centralized Control는 모든 Agent 간 통신, 작업 스케줄링, 충돌 해결을 단일 제어 노드가 처리해야 하므로 병목이 발생 가능하다.

이와 달리, decentralized collaboration은 self-organizing 프로토콜을 통해 노드 간 직접적인 상호작용을 가능하게한 방식이다

```
# 중안집중식
    ┌─────┐
    │중앙  │ ← 병목 발생
    └──┬──┘
  ┌────┼────┐
  ↓    ↓    ↓
  A    B    C
# 분산식
     A ←→ B
     ↕ ⤫ ↕
     C ←→ D
모든 에이전트가 직접 소통
```

두 가지 접근 방식으로 구성된다.

- Revision-based systems
- Communication-based systems

**Revision-based systems**

각 Agent 서로가 생성한 최종 결론만을 관찰하고,

구조화된 수정 프로토콜을 통해 공유 출력물을 반복적으로 정제한다

e.g. 미적분 문제 해결

```
문제: ∫₀^π sin²(x)dx = ?
【Round 1: 초기 답변】
Agent A:
"답: π/2
방법: sin²(x) = (1-cos(2x))/2 사용"
신뢰도: 90%
Agent B:
"답: π
방법: sin²(x) 직접 적분"
신뢰도: 70%
Agent C:
"답: π/2
방법: 수치 적분"
신뢰도: 85%
【Round 2: 상호 분석】
Agent A가 B 분석:
┌────────────────────────────┐
│ Agent B의 오류 발견:          │
│ "sin²(x) 직접 적분은 불가능     │
│ 삼각함수 항등식 필요"           │
│ → B의 신뢰도 하향: 70% → 30%  │
└────────────────────────────┘
Agent C가 A 검증:
┌────────────────────────────┐
│ Agent A의 방법 확인:          │
│ "변환 공식 정확함.             │
│ 내 수치 결과와도 일치"          │
│ → A의 신뢰도 상향: 90% → 95%  │
└────────────────────────────┘
【Round 3: 인간 예시 참조】
시스템이 유사 문제 예시 제공:
"∫₀^π cos²(x)dx = π/2"
Agent B (재계산):
"오류 인정. 올바른 방법:
sin²(x) = (1-cos(2x))/2
적분하면 π/2"
신뢰도: 30% → 90%
【Round 4: 합의】
최종 답: π/2
합의 방법:
- 변환 공식: (1-cos(2x))/2
- 검증: 수치 적분 일치
- 신뢰도: 95% (3명 합의)
```

| 시스템 | 특징 / 역할 |
| --- | --- |
| MedAgents | 도메인별 전문 에이전트를 순차적으로 배치 → 독립적 제안/수정 → 최종 투표로 합의 |
| ReConcile | 상호 응답 분석, 신뢰도 평가, 인간 검수 예시 기반 반복적 답변 개선 |
| METAL | 차트 생성용 텍스트/비주얼 전문 수정 에이전트 도입 → 도메인 특화 정제 향상 |

**Communication-based systems**

Agent들이 서로 직접 대화하고 서로의 추론 과정을 관찰할 수 있게 한다.

→ 이 특성 덕분에 인간 사회적 상호작용과 같은 동적 시나리오를 모델링하는 데 적합하다

e.g. 미국 독립 전쟁이 프랑스 혁명에 미친 영향은?

```
【Round 1: 초기 주장】
Agent A (역사학자 1):
"프랑스 혁명에 직접적 영향 컸음.
이유:
1. 라파예트가 미국 독립 전쟁 참전
2. 자유·평등 이념 전파
3. 군주제에 대한 회의 확산"
Agent B (역사학자 2):
"영향은 제한적이었음.
이유:
1. 프랑스 내부 경제 위기가 주원인
2. 계몽사상이 더 중요한 배경
3. 미국과 프랑스의 상황 차이 큼"
Agent C (역사학자 3):
"간접적이지만 중요한 영향.
이유:
1. 프랑스 재정 파탄 (미국 지원 비용)
2. 혁명 정당성의 선례 제공
3. 공화정 모델 제시"
【Round 2: 토론 (추론 과정 공유)】
Agent B → Agent A:
"라파예트 영향은 과대평가됐습니다.
근거를 보세요:
→ [검색] '프랑스 재정 위기 1789'
→ 국가 부채가 GDP의 150%
→ 미국 지원 비용: 전체 부채의 20%
이게 더 직접적 원인 아닌가요?"
Agent A (추론 과정 공개):
"좋은 지적입니다. 제 논리를 수정하겠습니다:
1단계: 미국 독립 성공 확인 ✅
2단계: 프랑스 지원 확인 ✅
3단계: 재정 영향 분석 중...
   → 아, B의 데이터가 설득력 있네요
4단계: 결론 수정 필요"
Agent C → Agent B:
"경제 위기만으로는 부족합니다.
제 추론:
IF 경제 위기 THEN 혁명?
역사적 반례:
- 1720년 미시시피 거품 → 혁명 없음
- 1847년 경제 위기 → 소규모 저항
THEREFORE 이념적 정당성도 필수
미국 독립이 바로 그 정당성 제공"
【Round 3: 합의 형성】
Agent A (수정된 입장):
"B와 C의 주장을 통합하면:
미국 독립 전쟁의 영향:
- 직접: 재정 파탄 (B 주장)
- 간접: 혁명 정당성 (C 주장)
두 요소가 결합하여 영향"
Agent B (양보):
"A와 C의 지적 수용.
순전히 경제만은 아니었음 인정"
Agent C:
"합의안:
'미국 독립 전쟁은 프랑스 혁명에
경제적 촉발(trigger)과
이념적 정당성을 동시에 제공했다'"
전원 합의: ✅
```

| 시스템 | 특징 / 역할 |
| --- | --- |
| MAD | 구조화된 통신 프로토콜 → “degeneration-of-thought” 문제 해결 (초기 솔루션 고착 방지) |
| MADR | 비현실적 주장 비판, 논거 개선, 검증 가능한 설명 생성 → 사실 확인 강화 |
| MDebate | 합의 구축 최적화 → 유효 주장 고집과 협력적 개선 번갈아 수행 |
| AutoGen | 그룹 채팅 프레임워크 구현 → iterative debate 통한 결정 정제 |

---

### **2.2.3 Hybrid Architecture**

개념:

centralized coordination과 decentralized collaboration을 전략적으로 결합한 방식

두 가지 경우로 구성된다.

- static systems
- dynamic systems

**static systems**

사전에 고정된 패턴으로 협력 방식을 결합하며, 실행 중에는 구조가 변하지 않습니다.

e.g. CAMEL-대학교 행정 시스템

구조

```
┌─────────────────────────────────────┐
│   총장실 (중앙 컨트롤러)                 │ ← 그룹 간 조정
└────────┬────────────────────────────┘
         │
    ┌────┼─────────┬──────────┐
    ↓    ↓         ↓          ↓
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│학사팀    │ │학생지원  │ │시설팀    │ │홍보팀   │
│(그룹1)  │ │(그룹2)  │ │(그룹3)   │ │(그룹4)  │
└───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘
    │          │          │          │
  [분산]     [분산]        [분산]     [분산]
    ↓         ↓          ↓          ↓
  팀원들     팀원들        팀원들       팀원들
  자율협력   자율협력      자율협력      자율협력
```

실행과정

```
【그룹 간 조정: 중앙집중식】
총장실 (중앙 컨트롤러):
┌────────────────────────────────┐
│ 전체 목표 설정:                   │
│ - 일정: 2025년 2월 20일           │
│ - 예산: 5000만원                 │
│ - 참가 인원: 신입생 1000명         │
│                                │
│ 그룹별 할당:                      │
│ 학사팀 → 프로그램 설계              │
│ 학생지원팀 → 멘토링 조직             │
│ 시설팀 → 장소 준비                 │
│ 홍보팀 → 안내 자료 제작             │
└────────────────────────────────┘
【그룹 1: 학사팀 (내부 분산 협력)】
팀원 A:
"커리큘럼 소개 세션 어떻게 할까?"
팀원 B:
"학과별로 나누는 게 좋겠어.
각 학과 교수님 초청하자."
팀원 C:
"시간 배분은 이렇게:
- 전체 소개: 30분
- 학과별: 각 20분"
팀원 A:
"좋아! 그럼 내가 교수님들 섭외할게."
팀원 B:
"나는 발표 자료 준비."
팀원 C:
"나는 시간표 작성."
→ 팀 내부 결정: 자율적 역할 분담 ✅
【그룹 2: 학생지원팀 (내부 분산 협력)】
팀원 X:
"선배 멘토 몇 명 필요할까?"
팀원 Y:
"1000명이니까... 50명?
신입생 20명당 멘토 1명."
팀원 Z:
"멘토 교육은 언제 해?"
팀원 X:
"오리엔테이션 1주일 전에 하자."
→ 팀 내부 결정: 멘토 50명, 사전 교육 ✅
【그룹 간 조정: 충돌 해결】
시설팀 → 총장실:
"⚠️ 문제 보고:
대강당 수용 인원: 800명
신입생: 1000명
→ 공간 부족!"
총장실 (중재):
"학사팀과 시설팀 조정 필요.
해결 방안 논의하라."
총장실 → 학사팀:
"프로그램을 2회로 나눌 수 있나?"
학사팀:
"가능합니다. 오전/오후 분리."
총장실 → 시설팀:
"2회 진행으로 변경. 문제 해결?"
시설팀:
"✅ 해결됨. 각 회 500명."
【최종 통합: 중앙 컨트롤러】
총장실:
┌────────────────────────────────┐
│ 최종 계획 승인:                   │
│ ✅ 학사팀: 프로그램 확정            │
│ ✅ 학생지원팀: 멘토 50명 배정        │
│ ✅ 시설팀: 2회 진행으로 해결         │
│ ✅ 홍보팀: 안내문 발송 준비          │
│                                │
│ → 모든 그룹 승인, 실행 시작          │
└────────────────────────────────┘
```

| 시스템 | 구현 방식 | 특징 |
| --- | --- | --- |
| CAMEL | 에이전트를 그룹 내 분산 팀으로 나눔 (role-playing simulation) | 그룹 간 조정은 중앙집권적 관리 |
| AFlow | 3단계 계층: 중앙 전략 계획 → 분산 전술 협상 → 시장 기반 운영 자원 할당 | 중앙-분산 혼합 계층 구조 |
| EoT | 4가지 협업 패턴(BUS, STAR, TREE, RING) 공식화 | 작업 특성에 맞춰 네트워크 토폴로지 정렬 |

**dynamic systems**

상황에 따라 에이전트 간 협력 구조를 실시간으로 변경하는 방식입니다

| 시스템 | 구현 방식 | 특징 |
| --- | --- | --- |
| DiscoGraph | Teacher-Student 구조, holistic-view 입력, feature map distillation | edge weights을 활용한 에이전트 간 적응적 공간 attention |
| DyLAN | Agent Importance Score 기반 | 가장 기여도가 높은 에이전트를 식별 후, 
협업 구조 동적 조정 → 작업 완료 최적화 |
| MDAgents | 작업 복잡도에 따라 협업 구조 동적 할당 | 단순 작업: 단일 에이전트 
처리복잡 작업: 계층적 협업 수행 |

---

## **2.3 Agent Evolution**

LLM Agent는 자율적개선, 멀티 에이전트 상호작용, 외부 자원 통합을 통해 계속 진화하고 있다.

진화의 3가지 핵심 방법을 다룬다.

- autonomous optimization and self-learning
- multi-agent co-evolution
- evolution via external resources

### **2.3.1 Autonomous optimization and self-learning**

LLM이 광범위한 supervision 없이도 스스로 능력을 향상할 수 있게 한다.

여기에는 아래와 같은 메커니즘을 포함한다.

- self-supervised learning
- self-reflection
- self-correction
- self-rewarding

이를 통해 모델은 스스로 탐색하고, 출력을 동적으로 정제한다.

**Self-Supervised Learning and Adaptive Adjustment.**

Self-supervised Learning은 비라벨 데이터 또는 모델이 스스로 생성한 데이터를 활용해 LLM을 향상시킨다.

e.g.

- Self-evolution learning (SE):
    
    사전학습(pretraining) 과정에서 토큰 마스킹과 학습 전략을 동적으로 조정해 성능 향상.
    
- evolutionary optimization 기법들:
    
    모델 병합 및 적응을 효율적으로 수행하여 추가 자원 없이 성능을 향상.
    
- DiverseEvol:
    
    데이터 다양성과 선정 효율성을 개선해 instruction tuning을 정교화.
    

**Self-Reflection and Self-Correction**

LLM이 스스로 오류를 식별하고 수정하면서 출력을 반복적으로 개선하도록 한다.

e.g.

- SELF-REFINE:
    
    모델이 외부 감독 없이 자체 피드백을 반복 적용해 응답을 향상.
    
- STaR, V-STaR:
    
    모델이 자신의 문제 해결 과정을 검증하고 정제하도록 훈련하여 라벨 데이터 의존도를 줄임.
    
- Self-verification:
    
    생성한 출력에 대해 사후적으로 평가 및 검증해 더 안정적이고 신뢰성 있는 결정이 가능하도록 함.
    

**Self-Rewarding and Reinforcement Learning**

LLM이 내부적으로 보상 신호를 생성하여 성능을 향상시킬 수 있게 한다

e.g.

- Contrastive distillation:
    
    자기 보상 기반 정렬을 통해 모델 간(혹은 모델 내부의 다양한 표현 간) 정렬을 촉진.
    
- RLC:
    
    evaluation-generation gap을 활용하는 강화학습 전략을 적용해 자기 개선 촉진.
    

---

### **2.3.2 Multi-Agent Co-Evolution**

LLM이 다른 Agent의 상호작용을 통해 성능을 향상하도록 한다.

여기에는 두가지 방법이 포함된다.

- cooperative learning
- competitive co-evolution

**cooperative learning**

멀티 에이전트 협업은 지식 공유, 공동 의사결정, 조정된 문제 해결을 가능하게 해 LLM의 성능을 향상시킨다.

e.g.

- ProAgent:
    
    LLM 기반 에이전트가 협력 작업에서 팀원의 의도를 추론하고 신념을 갱신하여 동적으로 적응하도록 지원
    
    → 제로샷 협업 능력 강화.
    
- CORY:
    
    RL 미세조정을 협력적 멀티 에이전트 프레임워크로 확장해,
    
    에이전트들이 역할 교환(role-exchange)을 통해 정책의 최적성과 안정성을 향상.
    
- CAMEL:
    
    에이전트들이 Inception Prompting을 활용해 자율적으로 소통하며 역할극 기반 협업을 수행
    
    → 멀티 에이전트 환경에서 조율·문제 해결 효율 향상.
    

**competitive co-evolution**

LLM이 적대적 상호작용, 토론, 전략 경쟁을 통해 강화되도록 한다.

e.g.

- Red-team LLMs:
    
    적대적 상호작용 중 지속적으로 진화하면서
    
    LLM의 취약점을 드러내고 모드 붕괴를 방지하여 보다 견고한 안전성 정렬(safety alignment) 을 달성.
    
- Du et al.:
    
    여러 LLM이 서로의 주장을 비판·정제하는 다중 라운드 토론 프레임워크를 제안
    
    → 사실성 향상, 환각 감소.
    
- MAD framework:
    
    에이전트들 간의 토론을 티키타카 방식으로 구조화하여
    
    divergent thinking를 촉진하고 복잡한 작업에서 논리적 추론을 강화
    

---

### **2.3.3 Evolution via External Resources**

외부 자원은 구조화된 정보와 피드백을 제공함으로써 에이전트의 진화를 촉진한다.

- Knowledge-enhanced evolution
- External feedback-driven evolution

**Knowledge-Enhanced Evolution**

LLM은 외부의 구조화된 지식을 통합함으로써 추론, 의사결정, 작업 수행 능력을 향상시킬 수 있다.

e.g.

- KnowAgent :
    
    LLM 기반 계획(planning)에 행동 지식(action knowledge)을 통합하고,
    
    의사결정 경로를 제약하며 환각(hallucination)을 줄임 → 보다 신뢰할 수 있는 작업 수행 가능.
    
- World Knowledge Model (WKM):
    
    전문가 지식과 경험적 지식을 종합하여 계획을 개선,
    
    글로벌 사전 정보(global priors)와 동적 지역 지식(dynamic local knowledge)을 제공해 의사결정 안내.
    

**External Feedback-Driven Evolution**

LLM은 도구, 평가자, 인간으로부터 외부 피드백을 활용하여 행동을 반복적으로 정교화하고 성능을 향상시킬 수 있다.

e.g.

- CRITIC :
    
    LLM이 도구 기반 피드백을 통해 출력물을 검증하고 수정 → 정확도 향상, 일관성 개선.
    
- STE :
    
    시행착오, 상상력, 기억 시뮬레이션을 통해 도구 학습(tool learning)을 강화 → 효과적 도구 사용 및 장기 적응 가능.
    
- SelfEvolve:
    
    LLM이 실행 결과 피드백을 활용해 코드를 생성·디버깅하는 2단계 프레임워크를 적용 → 인간 개입 없이 성능 향상.
