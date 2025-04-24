## 🧩 status_reporter 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`status_reporter` 패키지는 로봇의 내부 상태 정보를 수집하고 이를 외부(FastAPI, Web UI, ACS 등)에 제공하는 역할을 수행한다. 주요 책임은 다음과 같다:

- 하드웨어 및 소프트웨어로부터 상태 정보를 주기적으로 수집
- 위치, 배터리, 통신 상태 등 다양한 상태를 토픽 형태로 publish
- ACS나 UI에서 요청 시 최신 상태를 빠르게 제공할 수 있도록 상태 캐싱
- 긴급 정지나 충돌 등의 특이 상태를 알림 형태로 전달
- 실시간 3D 시각화 시스템 제공
- 안전 관련 상태 모니터링 및 관리

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `h_w_node` | 토픽 구독 | 배터리, 위치 정보 등 하드웨어 기반 상태 수신 |
| `ui_interface` | WebSocket | 상태 업데이트와 시각화 데이터를 UI에 실시간 전송 |
| `acs_interface` | 양방향 통신 | 로봇 상태 보고 및 명령 수신 |
| `safety_interlock_node` | 토픽 구독 | 안전 관련 상태 및 알림 수신 |
| `task_manager` | 토픽 구독 | 현재 진행 중인 시나리오, 작업 정보 공유 |
| `vision_node` | 토픽 구독 | 비전 처리 상태 및 이미지 수신 |
| `robot_collision_detection` | 토픽 구독 | 충돌 감지 상태 수신 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 제공하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/status/pose` | `geometry_msgs/msg/PoseStamped` | 현재 로봇 위치/자세 |
| `/status/battery` | `std_msgs/msg/Float32` | 배터리 잔량 (%) |
| `/status/comm` | `std_msgs/msg/String` | 네트워크 연결 상태 |
| `/status/task` | `custom_msgs/msg/TaskStatus` | 현재 진행 중인 작업 단계 |
| `/status/safety` | `custom_msgs/msg/SafetyStatus` | 안전 관련 상태 정보 |
| `/status/joints` | `sensor_msgs/msg/JointState` | 실시간 조인트 상태 |
| `/status/gripper` | `custom_msgs/msg/GripperStatus` | 그립퍼 상태 정보 |
| `/status/vision` | `custom_msgs/msg/VisionStatus` | 비전 처리 상태 정보 |
| `/status/alerts` | `custom_msgs/msg/AlertMessage` | 이상 상태 및 경고 메시지 |
| `/status/visualization` | `custom_msgs/msg/VisualizationData` | 시각화 데이터 |

#### ✅ 내부/사용 인터페이스

| 대상 | 방식 | 설명 |
|-------|------|------|
| `WebUI` | WebSocket | 실시간 상태 및 시각화 데이터 전송 |
| `ACS` | 양방향 통신 | 상태 보고 및 명령 수신 |
| `안전 시스템` | 토픽 구독/발행 | 안전 관련 상태 모니터링 및 경고 발행 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

#### 인터페이스 (추상 클래스)
- `IStatusMonitor` (개별 상태 모니터 인터페이스)
- `IStatusPublisher` (상태 퍼블리시 인터페이스)
- `IVisualizationHandler` (시각화 처리 인터페이스)
- `ISafetyMonitor` (안전 모니터링 인터페이스)

#### 핵심 클래스
| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `StatusPublisherNode` | ROS2 Node | 모든 상태 정보 수집 및 발행 제어 |
| `StatusAggregator` | Class | 여러 상태 정보 통합 및 관리 |
| `StatusPublisher` | Class | ROS2 토픽으로 상태 발행 |
| `StatusCache` | Class | 최신 상태 캐싱 및 빠른 응답 지원 |
| `StatusAnomalyDetector` | Class | 이상 상태 감지 및 알림 생성 |
| `WebUISyncInterface` | Class | Web UI와의 실시간 데이터 동기화 |
| `ACSReportInterface` | Class | ACS와의 양방향 통신 처리 |
| `VisualizationManager` | Class | 3D 시각화 시스템 관리 |
| `SafetyManager` | Class | 안전 관련 기능 통합 관리 |

#### 상태 모니터 구현체
- `PoseMonitor` (implements IStatusMonitor)
- `BatteryMonitor` (implements IStatusMonitor)
- `CommMonitor` (implements IStatusMonitor)
- `TaskStatusMonitor` (implements IStatusMonitor)
- `SafetyStatusMonitor` (implements ISafetyMonitor)
- `VisionStatusMonitor` (implements IStatusMonitor)
- `CollisionDetectionMonitor` (implements ISafetyMonitor)
- `RealTimeJointMonitor` (implements IStatusMonitor)
- `GripperStatusMonitor` (implements IStatusMonitor)

#### 시각화 핸들러 구현체
- `RobotStateVisualizer` (implements IVisualizationHandler)
- `TaskFlowVisualizer` (implements IVisualizationHandler)
- `CameraImageVisualizer` (implements IVisualizationHandler)
- `FeaturePointVisualizer` (implements IVisualizationHandler)
- `GraspPointVisualizer` (implements IVisualizationHandler)

---

### 5. 🔄 동작 흐름 예시 (상태 수집 및 전파)

1. 각 IStatusMonitor 구현체들이 담당 상태 정보 수집
2. `StatusAggregator`가 모든 모니터로부터 상태 통합
3. `StatusAnomalyDetector`가 수집된 상태 분석하여 이상 여부 판단
4. `StatusCache`에 최신 상태 저장
5. `StatusPublisher`가 ROS2 토픽으로 상태 발행
6. `WebUISyncInterface`가 WebSocket을 통해 UI에 실시간 데이터 전송
7. `ACSReportInterface`가 ACS에 상태 보고
8. `VisualizationManager`가 필요한 시각화 데이터 생성 및 UI 전송
9. `SafetyManager`가 안전 관련 상태 모니터링 및 필요시 경고 발행

---

### 6. 📊 클래스 간 관계 및 상호작용

- StatusPublisherNode는 StatusAggregator, StatusPublisher, VisualizationManager, SafetyManager를 포함(Aggregation)
- StatusAggregator는 여러 IStatusMonitor 구현체들을 포함
- 각 모니터 구현체들은 IStatusMonitor 또는 ISafetyMonitor 인터페이스를 구현
- StatusCache는 WebUISyncInterface, ACSReportInterface와 협력
- VisualizationManager는 여러 IVisualizationHandler 구현체를 관리
- SafetyManager는 여러 ISafetyMonitor 구현체를 포함하고 안전 관련 기능 통합

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 상태 수집 실패 | 기본값 반환, 로그 경고 |
| 통신 끊김 | Alert 발행, UI/ACS로 전파 |
| 시각화 실패 | 대체 뷰 표시 |
| 안전 위험 감지 | 즉시 정지 및 알림 |
| 카메라 오류 | 대체 이미지 표시 |
| 상태 발행 실패 | 재시도 및 캐시 활용 |
| ACS 연결 끊김 | 재연결 시도 및 로컬 캐싱 |

---

### 8. 📢 최종 요약

- 인터페이스 기반 설계로 유연한 상태 모니터링 구조 제공
- 다양한 상태 모니터링 기능 통합 (하드웨어, 안전, 비전 등)
- 실시간 3D 시각화 시스템 구현으로 직관적인 상태 확인 지원
- 이상 상태 감지 및 알림 시스템 강화
- ACS 및 Web UI와의 양방향 통신 지원
- 안전 관련 모니터링 기능 강화
- 모든 UseCase 요구사항 지원