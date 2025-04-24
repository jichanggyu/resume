# ai_interface_node 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### AINodeControlComponent
- 구성 요소: AINode
- 책임: ROS2 노드 전체 실행 제어, 입력 동기화 및 추론 주기 관리

### InputSynchronizationComponent
- 구성 요소: ImageSynchronizer
- 책임: 이미지 및 조인트 상태 동기화

### InferenceExecutionComponent
- 구성 요소: InferenceEngine, ModelLoader
- 책임: AI 모델 로딩 및 예측 실행

### InferenceOutputComponent
- 구성 요소: PredictionPublisher
- 책임: 추론 결과를 ROS2 Topic으로 퍼블리시

### InferenceLoggingComponent
- 구성 요소: InferenceLogger
- 책임: 추론 성공/실패 및 처리 시간 로깅

---

## 2. 컴포넌트 상세 설명

### AINodeControlComponent
- 구성 요소: AINode
- 역할:
  - ROS2 노드 초기화
  - 주기적 추론 루프 실행
- 입력:
  - 이미지, 조인트 상태 (ROS2 Topic 구독)
- 출력:
  - `PredictionPublisher.publish(prediction: List[float])`
- 협력:
  - 모든 하위 컴포넌트

### InputSynchronizationComponent
- 구성 요소: ImageSynchronizer
- 역할:
  - ROS2 토픽으로부터 이미지와 조인트 상태 수신
  - 타임스탬프 기준으로 동기화
- 입력:
  - `+ sync_inputs(image_msg, joint_msg) -> Tuple[image, joints]`
- 출력:
  - 동기화된 입력 데이터
- 협력:
  - AINode

### InferenceExecutionComponent
- 구성 요소: InferenceEngine, ModelLoader
- 역할:
  - 모델을 로딩하고 예측을 실행
- 입력:
  - `+ predict(image, joints) -> List[float]`
  - `+ load_model(path: str) -> Any`
- 출력:
  - 추론 결과 (List[float])
- 협력:
  - ImageSynchronizer, PredictionPublisher

### InferenceOutputComponent
- 구성 요소: PredictionPublisher
- 역할:
  - 추론 결과를 ROS2 토픽으로 퍼블리시
- 입력:
  - `+ publish(prediction: List[float]) -> None`
- 출력:
  - `/predicted_joint` ROS2 토픽
- 협력:
  - AINode

### InferenceLoggingComponent
- 구성 요소: InferenceLogger
- 역할:
  - 추론 성공 여부, latency, 실패율 등을 로그로 기록
- 입력:
  - `+ log_inference_result(success: bool, latency_ms: float) -> None`
- 출력:
  - 로그 시스템 기록
- 협력:
  - AINodeControlComponent

---

## 3. 컴포넌트 간 관계 요약

- AINodeControlComponent → InputSynchronizationComponent
- AINodeControlComponent → InferenceExecutionComponent
- AINodeControlComponent → InferenceOutputComponent
- AINodeControlComponent → InferenceLoggingComponent

---

## 4. 요약

| 컴포넌트 | 책임 | 주요 협력 대상 |
|:--|:--|:--|
| AINodeControlComponent | ROS2 노드 관리 및 추론 흐름 제어 | 전체 하위 컴포넌트 |
| InputSynchronizationComponent | 이미지/조인트 데이터 동기화 | AINode |
| InferenceExecutionComponent | AI 모델 로딩 및 추론 실행 | PredictionPublisher |
| InferenceOutputComponent | 추론 결과 퍼블리시 | ROS2 시스템 |
| InferenceLoggingComponent | 추론 결과 및 성능 기록 | AINode |

---

## 최종 정리

- 실시간 AI 추론 흐름을 입력 동기화 → 추론 실행 → 결과 퍼블리시로 세분화
- 추후 AI 모델 교체 및 입력 변경 시 최소 영향으로 대응 가능
- ROS2 기반 노드 구조와 추상화된 컴포넌트 책임이 조화롭게 구성됨
