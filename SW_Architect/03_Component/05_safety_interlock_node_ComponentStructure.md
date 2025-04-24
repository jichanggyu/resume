# 🧩 safety_interlock_node 패키지 – 컴포넌트 구조 설계

---

## 1. 컴포넌트 목록

### SafetyNodeComponent
- SafetyInterlockNode (rclpy.Node)
- 책임: 안전 시스템 전체 관리 및 ROS2 인터페이스 제공

### SafetyCheckComponent
- SafetyManager
- ISafetyCheck 구현체들 (JointLimitCheck, SpeedLimitCheck, WorkspaceCheck 등)
- 책임: 다양한 안전 조건 검사 및 관리

### EmergencyHandlingComponent
- EmergencyManager
- IEmergencyHandler 구현체들 (EmergencyStop, SafetyPowerCut 등)
- 책임: 비상 상황 감지 및 대응 관리

### HeartbeatMonitoringComponent
- HeartbeatMonitorManager
- IHeartbeatMonitor 구현체들 (WaistHeartbeatMonitor, LeftArmHeartbeatMonitor 등)
- 책임: 주요 구성 요소의 하트비트 모니터링

### InterlockControlComponent
- InterlockController
- 책임: 물리적 인터락 장치 제어

### SafetyMonitoringComponent
- SafetyMonitor
- 책임: 안전 상태 실시간 모니터링

### NotificationComponent
- SafetyNotificationManager
- ISafetyNotifier 구현체들
- 책임: 안전 관련 알림 처리 및 관리

---

## 2. 컴포넌트 상세 설명

### SafetyNodeComponent
- **구성 요소**: SafetyInterlockNode
- **역할**:
  - 안전 시스템 전체 조정 및 관리
  - ROS2 서비스 및 토픽 인터페이스 제공
  - 주기적인 안전 검사 및 하트비트 모니터링 실행
- **입력**:
  - ROS2 서비스 요청
  - 하트비트 신호
  - 안전 상태 데이터
- **출력**:
  - 안전 상태 토픽
  - 비상 정지 명령
  - 인터락 제어 신호
- **협력**:
  - 모든 하위 컴포넌트
  - 외부 하드웨어 노드들

### SafetyCheckComponent
- **구성 요소**: SafetyManager, ISafetyCheck 구현체들
- **역할**:
  - 다양한 안전 조건 검사 실행
  - 안전 위반 상태 감지 및 보고
- **입력**:
  - `+ perform_all_checks() -> bool`
  - `+ register_safety_check(check: ISafetyCheck) -> None`
- **출력**:
  - 안전 검사 결과
  - 위반 상태 보고서
- **협력**:
  - SafetyNodeComponent
  - EmergencyHandlingComponent

### EmergencyHandlingComponent
- **구성 요소**: EmergencyManager, IEmergencyHandler 구현체들
- **역할**:
  - 비상 상황 감지 및 적절한 대응 실행
  - 비상 정지 및 복구 절차 관리
- **입력**:
  - `+ trigger_emergency() -> None`
  - `+ reset_emergency() -> bool`
- **출력**:
  - 비상 대응 명령
  - 상태 보고서
- **협력**:
  - SafetyNodeComponent
  - InterlockControlComponent

### HeartbeatMonitoringComponent
- **구성 요소**: HeartbeatMonitorManager, IHeartbeatMonitor 구현체들
- **역할**:
  - 주요 하드웨어 구성 요소의 하트비트 신호 모니터링
  - 생존 상태 감지 및 관리
- **입력**:
  - 하트비트 신호 (각 하드웨어로부터)
  - `+ check_all_heartbeats() -> bool`
- **출력**:
  - 구성 요소 상태 보고서
  - 하트비트 손실 알림
- **협력**:
  - SafetyNodeComponent
  - EmergencyHandlingComponent

### InterlockControlComponent
- **구성 요소**: InterlockController
- **역할**:
  - 물리적 인터락 장치 제어
  - 하드웨어 안전 인터락 관리
- **입력**:
  - `+ enable_interlock() -> bool`
  - `+ disable_interlock() -> bool`
- **출력**:
  - 인터락 제어 신호
  - 인터락 상태 보고서
- **협력**:
  - SafetyNodeComponent
  - EmergencyHandlingComponent

### SafetyMonitoringComponent
- **구성 요소**: SafetyMonitor
- **역할**:
  - 실시간 안전 상태 모니터링
  - 안전 임계값 관리
- **입력**:
  - `+ monitor_safety_status() -> None`
  - `+ update_thresholds(thresholds: dict) -> None`
- **출력**:
  - 모니터링 데이터
  - 임계값 초과 알림
- **협력**:
  - SafetyNodeComponent
  - SafetyCheckComponent

### NotificationComponent
- **구성 요소**: SafetyNotificationManager, ISafetyNotifier 구현체들
- **역할**:
  - 안전 관련 알림 생성 및 발송
  - 다양한 알림 채널 관리
- **입력**:
  - `+ send_notification(type: str, details: dict) -> None`
- **출력**:
  - 알림 메시지 (다양한 채널로)
- **협력**:
  - SafetyNodeComponent
  - 모든 안전 관련 컴포넌트

---

## 3. 컴포넌트 간 관계 요약

- SafetyNodeComponent → SafetyCheckComponent (안전 검사 실행)
- SafetyNodeComponent → HeartbeatMonitoringComponent (하트비트 모니터링 실행)
- SafetyNodeComponent → EmergencyHandlingComponent (비상 상황 대응 요청)
- SafetyNodeComponent → InterlockControlComponent (인터락 제어 요청)
- SafetyCheckComponent → EmergencyHandlingComponent (안전 위반 시 비상 상황 알림)
- HeartbeatMonitoringComponent → EmergencyHandlingComponent (하트비트 손실 시 비상 상황 알림)
- EmergencyHandlingComponent → InterlockControlComponent (비상 상황 시 인터락 활성화 요청)
- 모든 컴포넌트 → NotificationComponent (알림 발송 요청)

---

## 4. 요약

| 컴포넌트 | 책임 | 협력 |
|:--|:--|:--|
| SafetyNodeComponent | 전체 시스템 조정 및 ROS2 인터페이스 | 모든 컴포넌트 |
| SafetyCheckComponent | 안전 조건 검사 및 위반 감지 | SafetyNode, Emergency |
| EmergencyHandlingComponent | 비상 상황 대응 및 관리 | SafetyNode, Interlock |
| HeartbeatMonitoringComponent | 구성 요소 생존 모니터링 | SafetyNode, Emergency |
| InterlockControlComponent | 물리적 인터락 제어 | SafetyNode, Emergency |
| SafetyMonitoringComponent | 실시간 안전 상태 모니터링 | SafetyNode, SafetyCheck |
| NotificationComponent | 안전 관련 알림 관리 | 모든 컴포넌트 |

---

## 5. 최종 정리

- 종합적인 안전 시스템 구축을 위한 모듈화된 컴포넌트 구조
- 안전 검사, 하트비트 모니터링, 비상 대응의 명확한 분리를 통한 확장성 확보
- 다양한 하드웨어 구성 요소에 대한 하트비트 모니터링 지원
- 물리적 인터락 및 소프트웨어 안전 시스템의 통합
- 실시간 모니터링 및 즉각적 대응 메커니즘
- 다중 안전 장치 및 백업 대응 전략으로 안전성 극대화
- 확장 가능한 알림 시스템으로 다양한 상황에 대응

---