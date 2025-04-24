# 📦 ai_interface_node 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`ai_interface_node` 패키지는 AI PC에서 동작하며, Vision Node와 Status Node로부터 수신한 이미지와 조인트 상태를 기반으로  
실시간으로 다음 로봇 조인트 상태를 예측하여 ROS2 토픽으로 퍼블리시한다. 이 패키지는 다음의 책임을 가진다:

- 이미지 및 조인트 상태를 ROS2 토픽을 통해 구독
- 이미지와 조인트 데이터의 동기화 및 전처리
- 동기화된 데이터에 대한 AI 모델 추론 수행
- 추론 결과를 `/predicted_joint` 토픽으로 publish
- 추론 성능 및 결과 로깅을 통한 실시간 모니터링
- 동기화, 지연 관리, QoS 최적화를 포함한 실시간 처리 지원

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `vision_node` | ROS2 Topic 구독 | `/image_raw` 토픽에서 이미지 수신 |
| `status_node` | ROS2 Topic 구독 | `/joint_states` 토픽에서 조인트 상태 수신 |
| `command_executor_node`<br>`status_node` | ROS2 Topic publish | `/predicted_joint` 토픽으로 예측된 다음 조인트 상태 퍼블리시 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 구독 (Subscribe)

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/image_raw` | `sensor_msgs/msg/Image` or `CompressedImage` | Vision Node에서 퍼블리시하는 이미지 |
| `/joint_states` | `sensor_msgs/msg/JointState` | Status Node에서 퍼블리시하는 조인트 상태 |

#### ✅ 퍼블리시 (Publish)

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/predicted_joint` | `sensor_msgs/msg/JointState` | 예측된 다음 조인트 상태 |

---

### 4. ⚙️ 내부 구성 요소

#### ✅ 인터페이스 (Interfaces)

| 인터페이스명 | 책임 | 주요 메소드 |
|------------|------|------|
| `IImageSynchronizer` | 이미지와 조인트 데이터 동기화 | `sync_inputs(image_msg, joint_msg)` |
| `IInferenceEngine` | AI 추론 엔진 | `predict(image, joints)` |
| `IPredictionPublisher` | 예측 결과 퍼블리시 | `publish(prediction)` |

#### ✅ 클래스 (Classes)

| 클래스명 | 타입 | 책임 | 
|------------|------|------|
| `AINode` | ROS2 Node | 이미지 수신, 추론 수행, 예측 퍼블리시 전체 제어 |
| `ImageSynchronizer` | `IImageSynchronizer` 구현 | 이미지와 조인트 입력을 동기화 |
| `InferenceEngine` | `IInferenceEngine` 구현 | AI 모델 로딩 및 추론 수행 |
| `PredictionPublisher` | `IPredictionPublisher` 구현 | 예측 결과를 ROS2 토픽으로 퍼블리시 |
| `InferenceLogger` | Class | 추론 결과 및 성능 로깅 |

---

### 5. 🔄 동작 흐름 예시

#### ✅ 추론 사이클 프로세스

1. `AINode`가 `/image_raw`, `/joint_states` 토픽을 구독하여 데이터 수신
2. `ImageSynchronizer`가 수신된 이미지와 조인트 데이터를 타임스탬프로 동기화
3. `InferenceEngine`이 동기화된 이미지+조인트 상태를 입력으로 받아 추론 실행
4. 예측된 조인트 벡터가 `PredictionPublisher`를 통해 `/predicted_joint`로 퍼블리시
5. `InferenceLogger`가 추론 결과, 지연 시간, 성공/실패 여부를 로깅
6. 10~30Hz 주기로 위 사이클이 반복 수행됨

---

### 6. 📎 에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 입력 동기화 실패 | 최근 유효 데이터 재사용 또는 None 반환, 로그 기록 |
| 모델 로딩 실패 | fallback 모델 적용 또는 노드 중단, 오류 상태 보고 |
| 추론 실패 | 예외 발생 후 기본값 퍼블리시, 오류 로그 기록 |
| 퍼블리시 실패 | 로깅 후 다음 주기 지속, 상태 모니터링 |

---

### 7. 📎 추가 고려 사항

- 추론 주기 유지(10~30Hz)를 위한 이미지 버퍼링 및 비동기 스레드 분리
- confidence 기반 filtering 또는 정규화 기능 구현
- 다양한 모델 포맷(ONNX, TensorRT) 지원으로 추론 성능 최적화
- 입력/추론/출력 단계를 명확히 분리한 구조로 유연한 확장성 확보
- 추후 추론 결과를 UI에 전송할 경우, FastAPI 연계 구조 추가 가능

---