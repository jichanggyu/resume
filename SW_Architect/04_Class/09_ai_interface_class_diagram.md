# 🧩 ai_interface_node 패키지 – Class Diagram (최종 자세한 설명 버전)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `IImageSynchronizer` (이미지+조인트 데이터 동기화 인터페이스)
- `IInferenceEngine` (AI 추론 엔진 인터페이스)
- `IPredictionPublisher` (예측 결과 퍼블리시 인터페이스)

### 📘 Core Classes
- `AINode` (rclpy.Node, AI 예측 통합 제어)
- `ImageSynchronizer` (이미지+조인트 동기화 구현)
- `InferenceEngine` (AI 추론 수행 구현)
- `PredictionPublisher` (ROS2 퍼블리시 구현)
- `InferenceLogger` (추론 결과 로깅)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 IImageSynchronizer (Interface)
- **책임**: 이미지와 조인트 상태를 동기화하여 반환
- **행동**:
  - `+ sync_inputs(image_msg, joint_msg) -> Tuple[np.ndarray, List[float]]`
- **협력**: AINode
- **에러처리**: 데이터 누락 시 None 반환
- **비고**: 타임스탬프 보정 허용 가능

## 📘 IInferenceEngine (Interface)
- **책임**: 입력 데이터에 대해 AI 추론 수행
- **행동**:
  - `+ predict(image: np.ndarray, joints: List[float]) -> List[float]`
- **협력**: AINode
- **에러처리**: 추론 실패 시 예외 발생
- **비고**: ONNX/TensorRT 백엔드 지원

## 📘 IPredictionPublisher (Interface)
- **책임**: 추론 결과를 ROS2 토픽으로 퍼블리시
- **행동**:
  - `+ publish(prediction: List[float]) -> None`
- **협력**: AINode
- **에러처리**: 퍼블리시 실패 시 로깅
- **비고**: `/predicted_joint` 토픽 사용

---

# ✅ 3. Core Class 상세 설명

## 📘 AINode
- **책임**: 이미지 수신, 추론 수행, 예측 퍼블리시 전체 제어
- **속성**:
  - `- synchronizer: IImageSynchronizer`
  - `- inference_engine: IInferenceEngine`
  - `- publisher: IPredictionPublisher`
  - `- logger: InferenceLogger`
- **행동**:
  - `+ process_cycle() -> None`
- **협력**: ImageSynchronizer, InferenceEngine, PredictionPublisher, InferenceLogger
- **비고**: 주기적으로 추론 사이클 실행 (Timer 기반)

## 📘 ImageSynchronizer
- **책임**: 이미지와 조인트 입력을 동기화
- **행동**:
  - `+ sync_inputs(image_msg, joint_msg) -> Tuple[np.ndarray, List[float]]`
- **협력**: AINode
- **비고**: 가장 최신 데이터 기준 동기화 지원

## 📘 InferenceEngine
- **책임**: AI 모델 로딩 및 추론 수행
- **행동**:
  - `+ predict(image: np.ndarray, joints: List[float]) -> List[float]`
- **협력**: AINode
- **비고**: 다양한 모델 포맷(ONNX, TensorRT) 지원 가능

## 📘 PredictionPublisher
- **책임**: 예측 결과를 ROS2 Topic으로 퍼블리시
- **행동**:
  - `+ publish(prediction: List[float]) -> None`
- **협력**: AINode
- **비고**: `/predicted_joint` 토픽 발행

## 📘 InferenceLogger
- **책임**: 추론 결과 및 성능 로깅
- **행동**:
  - `+ log_inference_result(success: bool, latency_ms: float) -> None`
- **협력**: AINode
- **비고**: latency, FPS, 실패율 기록

---

# ✅ 4. 클래스 간 관계 요약 (UML 관계)

- AINode → (Aggregation) → IImageSynchronizer, IInferenceEngine, IPredictionPublisher, InferenceLogger
- ImageSynchronizer → (Inheritance) → IImageSynchronizer
- InferenceEngine → (Inheritance) → IInferenceEngine
- PredictionPublisher → (Inheritance) → IPredictionPublisher

---

# ✅ 5. ROS2 및 시스템 특성 반영

- AINode는 ROS2 Node (rclpy.Node)로 동작
- `/predicted_joint` 토픽 퍼블리시
- 10~30Hz 주기 추론 및 퍼블리시 사이클 운영
- 동기화, 추론, 퍼블리시를 비동기 파이프라인으로 구성 가능

---

# ✅ 6. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 입력 동기화 실패 | 최근 유효 데이터 재사용 또는 None 반환 |
| 모델 로딩 실패 | fallback 모델 적용 또는 중단 |
| 추론 실패 | 예외 발생 후 기본값 퍼블리시 |
| 퍼블리시 실패 | 로깅 후 다음 주기 지속 |

---

# 📢 최종 요약

- 입력/추론/출력 단계를 명확히 분리한 구조
- 인터페이스 기반으로 유연한 확장성 확보
- ROS2 통신 및 추론 파이프라인 최적화 설계 완료

---
