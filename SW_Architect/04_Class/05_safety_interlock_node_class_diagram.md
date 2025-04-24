# 🧩 safety_interlock_node 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `ISafetyCheck` (안전 검사 인터페이스)
- `IEmergencyHandler` (비상 상황 처리 인터페이스)
- `ISafetyNotifier` (안전 알림 인터페이스)
- `IInterlockController` (인터락 제어 인터페이스)
- `IHeartbeatMonitor` (하트비트 모니터링 인터페이스)

### 📘 Core Classes
- `SafetyInterlockNode` (rclpy.Node, 안전 인터락 노드)
- `SafetyManager` (안전 기능 통합 관리)
- `EmergencyManager` (비상 상황 관리)
- `InterlockController` (인터락 제어)
- `SafetyMonitor` (안전 상태 모니터링)
- `SafetyNotificationManager` (안전 알림 관리)
- `HeartbeatMonitorManager` (하트비트 모니터링 관리)

### 📘 Safety Check 구현체
- `JointLimitCheck` (implements ISafetyCheck)
- `SpeedLimitCheck` (implements ISafetyCheck)
- `WorkspaceCheck` (implements ISafetyCheck)
- `CollisionCheck` (implements ISafetyCheck)
- `PowerSystemCheck` (implements ISafetyCheck)
- `ToolStateCheck` (implements ISafetyCheck)

### 📘 Emergency Handler 구현체
- `EmergencyStop` (implements IEmergencyHandler)
- `SafetyPowerCut` (implements IEmergencyHandler)
- `SafePoseRecovery` (implements IEmergencyHandler)
- `AlarmActivation` (implements IEmergencyHandler)

### 📘 Heartbeat Monitor 구현체
- `WaistHeartbeatMonitor` (implements IHeartbeatMonitor)
- `LeftArmHeartbeatMonitor` (implements IHeartbeatMonitor)
- `RightArmHeartbeatMonitor` (implements IHeartbeatMonitor)
- `HeadMotorHeartbeatMonitor` (implements IHeartbeatMonitor)
- `CameraHeartbeatMonitor` (implements IHeartbeatMonitor)
- `AGVHeartbeatMonitor` (implements IHeartbeatMonitor)

---

# ✅ 2. 인터페이스 상세 설명

## 📘 ISafetyCheck
- **책임**: 특정 안전 조건 검사
- **행동**:
  - `+ check_safety() -> bool`
  - `+ get_safety_status() -> dict`
  - `+ get_violation_details() -> str`
- **협력**: SafetyManager

## 📘 IEmergencyHandler
- **책임**: 비상 상황 대응
- **행동**:
  - `+ handle_emergency() -> bool`
  - `+ reset_emergency() -> bool`
  - `+ get_emergency_status() -> dict`
- **협력**: EmergencyManager

## 📘 ISafetyNotifier
- **책임**: 안전 관련 알림 처리
- **행동**:
  - `+ notify_violation(details: dict) -> None`
  - `+ notify_emergency(status: dict) -> None`
- **협력**: SafetyNotificationManager

## 📘 IInterlockController
- **책임**: 하드웨어 인터락 제어
- **행동**:
  - `+ activate_interlock() -> bool`
  - `+ deactivate_interlock() -> bool`
  - `+ get_interlock_status() -> bool`
- **협력**: InterlockController

## 📘 IHeartbeatMonitor
- **책임**: 구성 요소 하트비트 상태 모니터링
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ get_last_heartbeat_time() -> float`
  - `+ get_component_status() -> dict`
- **협력**: HeartbeatMonitorManager
- **비고**: 각 구성 요소의 생존 여부 확인

---

# ✅ 3. Core Class 상세 설명

## 📘 SafetyInterlockNode
- **책임**: 안전 시스템 전체 관리
- **속성**:
  - `- safety_manager: SafetyManager`
  - `- emergency_manager: EmergencyManager`
  - `- interlock_controller: InterlockController`
  - `- heartbeat_manager: HeartbeatMonitorManager`
- **행동**:
  - `+ run_safety_checks() -> None`
  - `+ handle_emergency_event() -> None`
  - `+ reset_safety_system() -> bool`
  - `+ check_system_heartbeats() -> bool`

## 📘 SafetyManager
- **책임**: 안전 검사 관리
- **속성**:
  - `- safety_checks: List[ISafetyCheck]`
  - `- current_status: dict`
- **행동**:
  - `+ perform_all_checks() -> bool`
  - `+ get_violation_report() -> dict`
  - `+ register_safety_check(check: ISafetyCheck) -> None`

## 📘 EmergencyManager
- **책임**: 비상 상황 처리 관리
- **속성**:
  - `- handlers: List[IEmergencyHandler]`
  - `- emergency_state: bool`
- **행동**:
  - `+ trigger_emergency() -> None`
  - `+ reset_emergency() -> bool`
  - `+ get_emergency_status() -> dict`

## 📘 InterlockController
- **책임**: 물리적 인터락 장치 제어
- **속성**:
  - `- interlock_status: bool`
  - `- hardware_interface: dict`
- **행동**:
  - `+ enable_interlock() -> bool`
  - `+ disable_interlock() -> bool`
  - `+ check_interlock_status() -> bool`

## 📘 SafetyMonitor
- **책임**: 안전 상태 실시간 모니터링
- **속성**:
  - `- safety_thresholds: dict`
  - `- current_values: dict`
- **행동**:
  - `+ monitor_safety_status() -> None`
  - `+ update_thresholds(thresholds: dict) -> None`
  - `+ get_monitoring_data() -> dict`

## 📘 SafetyNotificationManager
- **책임**: 안전 관련 알림 관리
- **속성**:
  - `- notification_handlers: List[ISafetyNotifier]`
- **행동**:
  - `+ send_notification(type: str, details: dict) -> None`
  - `+ register_handler(handler: ISafetyNotifier) -> None`

## 📘 HeartbeatMonitorManager
- **책임**: 다양한 하드웨어 구성 요소의 하트비트 모니터링
- **속성**:
  - `- monitors: Dict[str, IHeartbeatMonitor]`
  - `- timeout_thresholds: Dict[str, float]`
  - `- component_status: Dict[str, bool]`
- **행동**:
  - `+ check_all_heartbeats() -> bool`
  - `+ get_component_statuses() -> dict`
  - `+ set_timeout_threshold(component: str, timeout: float) -> None`
- **비고**: 모든 주요 시스템 구성 요소의 생존 여부 통합 관리

---

# ✅ 4. Heartbeat Monitor 구현체 상세 설명

## 📘 WaistHeartbeatMonitor
- **책임**: 3개의 허리축 하트비트 모니터링
- **속성**:
  - `- axis_statuses: Dict[str, bool]`
  - `- last_heartbeat_times: Dict[str, float]`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_individual_axis(axis_id: str) -> bool`

## 📘 LeftArmHeartbeatMonitor
- **책임**: 왼쪽 협동로봇 하트비트 모니터링
- **속성**:
  - `- controller_status: bool`
  - `- last_heartbeat_time: float`
  - `- safety_controller_status: bool`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_safety_controller() -> bool`

## 📘 RightArmHeartbeatMonitor
- **책임**: 오른쪽 협동로봇 하트비트 모니터링
- **속성**:
  - `- controller_status: bool`
  - `- last_heartbeat_time: float`
  - `- safety_controller_status: bool`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_safety_controller() -> bool`

## 📘 HeadMotorHeartbeatMonitor
- **책임**: 머리 모터 하트비트 모니터링
- **속성**:
  - `- motor_status: bool`
  - `- last_heartbeat_time: float`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_position_feedback() -> bool`

## 📘 CameraHeartbeatMonitor
- **책임**: 카메라 시스템 하트비트 모니터링
- **속성**:
  - `- camera_status: bool`
  - `- last_heartbeat_time: float`
  - `- stream_active: bool`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_stream_status() -> bool`

## 📘 AGVHeartbeatMonitor
- **책임**: AGV 하트비트 모니터링
- **속성**:
  - `- agv_status: bool`
  - `- last_heartbeat_time: float`
  - `- controller_status: bool`
- **행동**:
  - `+ monitor_heartbeat() -> bool`
  - `+ check_connection_status() -> bool`

---

# ✅ 5. 클래스 간 관계 요약 (UML 관계)

- SafetyInterlockNode → (Aggregation) → SafetyManager, EmergencyManager, InterlockController, HeartbeatMonitorManager
- SafetyManager → (Association) → ISafetyCheck 구현체들
- EmergencyManager → (Association) → IEmergencyHandler 구현체들
- SafetyNotificationManager → (Association) → ISafetyNotifier 구현체들
- InterlockController → (Implements) → IInterlockController
- HeartbeatMonitorManager → (Association) → IHeartbeatMonitor 구현체들

---

# ✅ 6. ROS2 및 시스템 특성 반영

- SafetyInterlockNode는 다음 서비스/토픽 제공:
  - `/safety/check_status`
  - `/safety/emergency_stop`
  - `/safety/reset_system`
  - `/safety/interlock_control`
  - `/safety/heartbeat_status`
- 안전 상태 실시간 모니터링 및 발행
- 비상 정지 명령 우선 처리
- 하드웨어 인터락 직접 제어
- 주요 구성 요소 하트비트 모니터링

---

# ✅ 7. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| 안전 검사 실패 | 즉시 비상 정지 |
| 하드웨어 인터락 실패 | 백업 시스템 활성화 |
| 통신 두절 | 자동 안전 모드 전환 |
| 센서 오류 | 보수적 안전 판단 |
| 전원 이상 | 안전한 종료 수행 |
| 허리축 하트비트 손실 | 허리 모터 전원 차단 및 인터락 활성화 |
| 협동로봇 하트비트 손실 | 해당 로봇 비상정지 및 인터락 활성화 |
| 머리 모터 하트비트 손실 | 머리 모터 전원 차단 |
| 카메라 하트비트 손실 | 경고 알림 및 로깅 |
| AGV 하트비트 손실 | AGV 비상정지 명령 송신 |
| 다중 구성요소 하트비트 손실 | 전체 시스템 인터락 활성화 |

---

# 📢 최종 요약

- 종합적인 안전 시스템 구축
- 실시간 모니터링 및 즉각 대응
- 다중 안전 장치 지원
- 안전한 복구 절차 구현
- 모든 주요 구성 요소 하트비트 모니터링 통합
- 구성 요소별 이상 상태 감지 및 대응
- 시스템 생존성 확보를 위한 선제적 안전 관리
- UseCase 요구사항 완벽 반영

---
