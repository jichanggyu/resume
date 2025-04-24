# 🧩 status_reporter 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `IStatusMonitor` (개별 상태 모니터 인터페이스)
- `IStatusPublisher` (상태 퍼블리시 인터페이스)
- `IVisualizationHandler` (시각화 처리 인터페이스)
- `ISafetyMonitor` (안전 모니터링 인터페이스)

### 📘 Core Classes
- `StatusPublisherNode` (rclpy.Node, 상태 제어 루프 노드)
- `StatusAggregator` (여러 상태 통합 및 캐시 관리)
- `StatusPublisher` (ROS2 토픽 퍼블리시 수행)
- `StatusCache` (현재 상태 저장 및 외부 요청 대응)
- `StatusAnomalyDetector` (이상 상태 감지 및 알림)
- `WebUISyncInterface` (Web UI 연동)
- `ACSReportInterface` (ACS 양방향 통신)
- `VisualizationManager` (3D 시각화 관리)
- `SafetyManager` (안전 기능 통합 관리)

### 📘 상태 모니터 구현체
- `PoseMonitor` (implements IStatusMonitor)
- `BatteryMonitor` (implements IStatusMonitor)
- `CommMonitor` (implements IStatusMonitor)
- `TaskStatusMonitor` (implements IStatusMonitor)
- `SafetyStatusMonitor` (implements ISafetyMonitor)
- `VisionStatusMonitor` (implements IStatusMonitor)
- `CollisionDetectionMonitor` (implements ISafetyMonitor)
- `RealTimeJointMonitor` (implements IStatusMonitor)
- `GripperStatusMonitor` (implements IStatusMonitor)

### 📘 시각화 핸들러 구현체
- `RobotStateVisualizer` (implements IVisualizationHandler)
- `TaskFlowVisualizer` (implements IVisualizationHandler)
- `CameraImageVisualizer` (implements IVisualizationHandler)
- `FeaturePointVisualizer` (implements IVisualizationHandler)
- `GraspPointVisualizer` (implements IVisualizationHandler)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 IStatusMonitor
- **책임**: 개별 상태(Pose, Battery 등)를 수집하고 dict 형태로 반환
- **행동**:
  - `+ get_value() -> dict`
- **협력**: StatusAggregator
- **비고**: 다양한 상태 항목 추가 시 확장 가능

## 📘 IStatusPublisher
- **책임**: 통합 상태를 외부 시스템에 퍼블리시
- **행동**:
  - `+ publish(status: dict) -> None`
- **협력**: ROS2 or Web/ACS 인터페이스

## 📘 IVisualizationHandler
- **책임**: 다양한 상태 데이터의 시각화 처리
- **행동**:
  - `+ visualize(data: dict) -> None`
  - `+ update_view() -> None`
  - `+ clear() -> None`
- **협력**: VisualizationManager
- **비고**: 실시간 3D 시각화 지원

## 📘 ISafetyMonitor
- **책임**: 안전 관련 상태 모니터링
- **행동**:
  - `+ check_safety() -> bool`
  - `+ get_safety_status() -> dict`
  - `+ trigger_emergency() -> None`
- **협력**: SafetyManager
- **비고**: 충돌 감지, 비상 정지 등 안전 기능 통합

---

# ✅ 3. Core Class 상세 설명

## 📘 StatusPublisherNode
- **책임**: Timer 기반 루프 실행, 전체 수집/퍼블리시 흐름 제어
- **행동**:
  - `+ run_status_loop()`
- **협력**: StatusAggregator, StatusPublisher

## 📘 StatusAggregator
- **책임**: 각 Monitor로부터 상태 수집 후 요약 상태 구성
- **속성**:
  - `- monitors: List[IStatusMonitor]`
- **행동**:
  - `+ collect_summary() -> dict`
- **협력**: StatusCache, StatusAnomalyDetector

## 📘 StatusPublisher
- **책임**: ROS2 Topic 퍼블리시
- **행동**:
  - `+ publish_all(summary: dict)`
- **협력**: ROS2 퍼블리셔

## 📘 StatusCache
- **책임**: 최신 상태 캐싱, 외부 요청 대응
- **행동**:
  - `+ update(summary: dict)`
  - `+ get_current_status() -> dict`
- **협력**: WebUI, ACS

## 📘 StatusAnomalyDetector
- **책임**: 상태 값 분석 → 이상 상태 감지 (예: 저전압, 통신 끊김)
- **행동**:
  - `+ analyze(summary: dict)`
  - `+ publish_alert()`
- **협력**: UI/ACS 알림 인터페이스

## 📘 WebUISyncInterface
- **책임**: Web UI에 상태 전송 (WebSocket 또는 REST)
- **행동**:
  - `+ push_status_to_ui(summary: dict)`

## 📘 ACSReportInterface
- **책임**: ACS에 상태 보고
- **행동**:
  - `+ send_to_acs(summary: dict)`

## 📘 VisualizationManager
- **책임**: 다양한 시각화 핸들러 관리 및 통합
- **속성**:
  - `- visualizers: Dict[str, IVisualizationHandler]`
- **행동**:
  - `+ update_visualization(data: dict)`
  - `+ register_visualizer(name: str, visualizer: IVisualizationHandler)`
- **협력**: 모든 IVisualizationHandler 구현체

## 📘 SafetyManager
- **책임**: 안전 관련 기능 통합 관리
- **속성**:
  - `- safety_monitors: List[ISafetyMonitor]`
  - `- emergency_handlers: List[callable]`
- **행동**:
  - `+ check_all_safety() -> bool`
  - `+ handle_emergency() -> None`
- **협력**: 모든 ISafetyMonitor 구현체

---

# ✅ 4. 상태 모니터 구현체 상세 설명

## 📘 PoseMonitor
- **책임**: 현재 위치(Pose) 수집
- **행동**: `+ get_value() -> dict`

## 📘 BatteryMonitor
- **책임**: 배터리 퍼센트 수집 및 저전압 감지
- **행동**: `+ get_value() -> dict`

## 📘 CommMonitor
- **책임**: 네트워크 연결 상태 수집 및 끊김 감지
- **행동**: `+ get_value() -> dict`

## 📘 TaskStatusMonitor
- **책임**: 현재 작업(Task) 상태 수집
- **행동**: `+ get_value() -> dict`

## 📘 SafetyStatusMonitor
- **책임**: 충돌/비상 정지 상태 수신
- **행동**: `+ get_value() -> dict`

## 📘 VisionStatusMonitor
- **책임**: 카메라 및 비전 처리 상태 모니터링
- **행동**: 
  - `+ get_value() -> dict`
  - `+ check_camera_status() -> bool`

## 📘 CollisionDetectionMonitor
- **책임**: 실시간 충돌 감지 모니터링
- **행동**:
  - `+ get_value() -> dict`
  - `+ check_collision() -> bool`

## 📘 RealTimeJointMonitor
- **책임**: 실시간 조인트 상태 모니터링
- **행동**:
  - `+ get_value() -> dict`
  - `+ predict_joint_status() -> dict`

## 📘 GripperStatusMonitor
- **책임**: 그립터 상태 모니터링
- **행동**: `+ get_value() -> dict`

---

# ✅ 5. 시각화 핸들러 구현체 상세 설명

## 📘 RobotStateVisualizer
- **책임**: 로봇의 실시간 상태 3D 시각화
- **행동**:
  - `+ visualize(state: dict) -> None`
  - `+ update_joint_states(joints: dict) -> None`

## 📘 TaskFlowVisualizer
- **책임**: 현재 작업 흐름 시각화
- **행동**:
  - `+ visualize(flow: dict) -> None`
  - `+ highlight_current_task() -> None`

## 📘 CameraImageVisualizer
- **책임**: 카메라 이미지 실시간 표시
- **행동**:
  - `+ visualize(image: np.ndarray) -> None`
  - `+ overlay_info(info: dict) -> None`

## 📘 FeaturePointVisualizer
- **책임**: 추출된 특징점 시각화
- **행동**:
  - `+ visualize(features: List[dict]) -> None`
  - `+ highlight_feature(feature_id: int) -> None`

## 📘 GraspPointVisualizer
- **책임**: 생성된 그래스프 포인트 시각화
- **행동**:
  - `+ visualize(grasp_points: List[dict]) -> None`
  - `+ show_best_grasp() -> None`

---

# ✅ 6. 클래스 간 관계 요약 (UML 관계)

- StatusPublisherNode → (Aggregation) → StatusAggregator, StatusPublisher
- StatusAggregator → (Aggregation) → List of IStatusMonitor
- StatusAggregator → StatusCache, StatusAnomalyDetector
- StatusCache → WebUISyncInterface, ACSReportInterface
- All Monitors → (Implements) → IStatusMonitor
- VisualizationManager → (Aggregation) → IVisualizationHandler 구현체들
- SafetyManager → (Aggregation) → ISafetyMonitor 구현체들
- StatusPublisherNode → (Association) → VisualizationManager, SafetyManager

---

# ✅ 7. ROS2 및 시스템 특성 반영

- 모든 상태 데이터는 ROS2 토픽으로 발행
- 시각화 데이터는 Web UI로 실시간 전송
- 안전 관련 데이터는 우선순위 큐로 처리
- ACS와 양방향 통신 지원 (상태 보고 및 명령 수신)

---

# ✅ 8. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 상태 수집 실패 | 기본값 반환, 로그 경고 |
| 통신 끊김 | Alert 발행, UI/ACS로 전파 |
| 시각화 실패 | 대체 뷰 표시 |
| 안전 위험 감지 | 즉시 정지 및 알림 |
| 카메라 오류 | 대체 이미지 표시 |

---

# 📢 최종 요약

- UseCase 기반으로 확장된 모니터링 구조
- 실시간 3D 시각화 시스템 통합
- 안전 관련 기능 강화
- 비전 처리 상태 모니터링 추가
- ACS 양방향 통신 지원
