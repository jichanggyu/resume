# status_reporter 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### StatusPublisherNodeComponent
- StatusPublisherNode
- 책임: 주기적 상태 수집 및 전체 퍼블리시 흐름 관리 (Timer 기반)

### StatusMonitoringComponent
- IStatusMonitor 인터페이스
- 다양한 모니터 구현체들:
  - PoseMonitor
  - BatteryMonitor
  - CommMonitor
  - TaskStatusMonitor
  - VisionStatusMonitor
  - RealTimeJointMonitor
  - GripperStatusMonitor
- 책임: 각 상태 항목에 대한 개별 수집 및 변화 감지

### SafetyMonitoringComponent
- ISafetyMonitor 인터페이스
- SafetyManager
- 안전 모니터 구현체들:
  - SafetyStatusMonitor
  - CollisionDetectionMonitor
- 책임: 안전 관련 상태 모니터링 및 비상 대응 관리

### StatusAggregationComponent
- StatusAggregator
- 책임: 수집된 상태를 통합하여 하나의 summary 상태로 구성

### StatusPublishingComponent
- IStatusPublisher 인터페이스
- StatusPublisher
- 책임: 통합 상태를 ROS2 Topic으로 퍼블리시

### StatusCachingComponent
- StatusCache
- 책임: 최신 상태를 메모리에 저장하고 요청 시 제공

### StatusAlertComponent
- StatusAnomalyDetector
- 책임: 비정상 상태(배터리 저전압, 통신 끊김 등) 감지 및 알림 발행

### VisualizationComponent
- IVisualizationHandler 인터페이스
- VisualizationManager
- 다양한 시각화 핸들러 구현체들:
  - RobotStateVisualizer
  - TaskFlowVisualizer
  - CameraImageVisualizer
  - FeaturePointVisualizer
  - GraspPointVisualizer
- 책임: 상태 데이터를 시각적으로 표현하고 관리

### WebUISyncComponent
- WebUISyncInterface
- 책임: FastAPI 기반 WebSocket 및 REST 상태 전달

### ACSReportComponent
- ACSReportInterface
- 책임: 통합 상태를 ACS에 TCP/HTTP 방식으로 보고

---

## 2. 컴포넌트 상세 설명

### StatusPublisherNodeComponent
- **구성 요소**: StatusPublisherNode
- **역할**:
  - 주기적 상태 수집 및 퍼블리시 루프 실행
  - 전체 상태 관리 시스템 조정
- **입력**:
  - `+ run_status_loop()`
- **출력**:
  - 통합 상태 관리 흐름 제어
- **협력**:
  - StatusAggregationComponent
  - StatusPublishingComponent
  - VisualizationComponent
  - SafetyMonitoringComponent

### StatusMonitoringComponent
- **구성 요소**: IStatusMonitor 인터페이스 및 구현체들
- **역할**:
  - 센서, 드라이버, 노드로부터 개별 상태 수집
  - 상태 변화 감지 및 데이터 전처리
- **입력**:
  - `+ fetch_from_source()`
- **출력**:
  - `+ get_value() -> dict`
- **협력**:
  - StatusAggregationComponent
  - 다양한 데이터 소스 (하드웨어, 센서, 노드)

### SafetyMonitoringComponent
- **구성 요소**: ISafetyMonitor 인터페이스, SafetyManager, 안전 모니터 구현체들
- **역할**:
  - 안전 관련 상태 모니터링
  - 비상 상황 감지 및 대응
  - 안전 기능 통합 관리
- **입력**:
  - `+ check_safety() -> bool`
  - `+ get_safety_status() -> dict`
- **출력**:
  - 안전 상태 보고
  - 비상 대응 명령
- **협력**:
  - StatusPublisherNodeComponent
  - StatusAggregationComponent
  - StatusAlertComponent

### StatusAggregationComponent
- **구성 요소**: StatusAggregator
- **역할**:
  - 모든 모니터로부터 상태 수집 후 하나의 summary dict 생성
  - 상태 데이터 구조화 및 정규화
- **입력**:
  - `+ collect_summary() -> dict`
- **출력**:
  - `+ get_summary() -> dict`
- **협력**:
  - StatusMonitoringComponent
  - SafetyMonitoringComponent
  - StatusCachingComponent
  - StatusAlertComponent

### StatusPublishingComponent
- **구성 요소**: IStatusPublisher 인터페이스, StatusPublisher
- **역할**:
  - summary 상태를 ROS2 토픽으로 퍼블리시
  - 다양한 포맷으로 상태 발행
- **입력**:
  - `+ publish_all(summary: dict)`
- **출력**:
  - `/status/pose`, `/status/battery`, `/status/task` 등 토픽
- **협력**:
  - StatusAggregationComponent
  - ROS2 퍼블리셔

### StatusCachingComponent
- **구성 요소**: StatusCache
- **역할**:
  - 최신 상태를 메모리에 보관
  - 요청에 따라 상태 제공
  - 상태 히스토리 관리
- **입력**:
  - `+ update(summary: dict)`
- **출력**:
  - `+ get_current_status() -> dict`
  - `+ get_history() -> List[dict]`
- **협력**:
  - WebUISyncComponent
  - ACSReportComponent
  - VisualizationComponent

### StatusAlertComponent
- **구성 요소**: StatusAnomalyDetector
- **역할**:
  - 배터리 저전압, 통신 오류 등 비정상 상태 감지
  - 경고 토픽 발행 및 이벤트 전파
- **입력**:
  - `+ analyze(summary: dict)`
- **출력**:
  - `/status/alert` 토픽 또는 이벤트
- **협력**:
  - StatusAggregationComponent
  - WebUISyncComponent
  - ACSReportComponent

### VisualizationComponent
- **구성 요소**: IVisualizationHandler 인터페이스, VisualizationManager, 시각화 핸들러 구현체들
- **역할**:
  - 상태 데이터의 시각적 표현 관리
  - 다양한 시각화 방식 제공 (3D 모델, 그래프, 이미지 등)
- **입력**:
  - `+ update_visualization(data: dict)`
  - `+ register_visualizer(name: str, visualizer: IVisualizationHandler)`
- **출력**:
  - 시각화 데이터 및 변환된 표현
- **협력**:
  - StatusPublisherNodeComponent
  - StatusCachingComponent
  - WebUISyncComponent

### WebUISyncComponent
- **구성 요소**: WebUISyncInterface
- **역할**:
  - 상태를 FastAPI 서버로 전송
  - WebSocket 또는 REST 응답 제공
  - 시각화 데이터 변환 및 전달
- **입력**:
  - `+ push_status_to_ui(summary: dict)`
- **출력**:
  - WebSocket 메시지 또는 REST 응답
- **협력**:
  - StatusCachingComponent
  - VisualizationComponent
  - 웹 클라이언트

### ACSReportComponent
- **구성 요소**: ACSReportInterface
- **역할**:
  - 통합 상태를 ACS 시스템으로 전송
  - ACS와의 양방향 통신 관리
- **입력**:
  - `+ send_to_acs(summary: dict)`
- **출력**:
  - ACS 프로토콜에 맞는 데이터 전송
- **협력**:
  - StatusCachingComponent
  - StatusAlertComponent
  - 외부 ACS 시스템

---

## 3. 컴포넌트 간 관계 요약

- StatusPublisherNodeComponent → StatusMonitoringComponent (상태 수집 요청)
- StatusPublisherNodeComponent → SafetyMonitoringComponent (안전 상태 체크)
- StatusPublisherNodeComponent → VisualizationComponent (시각화 관리)
- StatusMonitoringComponent → StatusAggregationComponent (개별 상태 전달)
- SafetyMonitoringComponent → StatusAggregationComponent (안전 상태 전달)
- StatusAggregationComponent → StatusPublishingComponent (통합 상태 발행)
- StatusAggregationComponent → StatusCachingComponent (상태 캐싱)
- StatusAggregationComponent → StatusAlertComponent (이상 상태 분석)
- StatusCachingComponent → WebUISyncComponent (UI 상태 제공)
- StatusCachingComponent → ACSReportComponent (ACS 상태 제공)
- StatusCachingComponent → VisualizationComponent (시각화 데이터 제공)
- VisualizationComponent → WebUISyncComponent (시각화 결과 전달)

---

## 4. 요약

| 컴포넌트 | 책임 | 협력 대상 |
|----------|------|------------|
| StatusPublisherNodeComponent | 전체 상태 관리 시스템 조정 | 모든 컴포넌트 |
| StatusMonitoringComponent | 개별 상태 수집 | 다양한 소스, Aggregator |
| SafetyMonitoringComponent | 안전 상태 모니터링 및 대응 | 안전 관련 소스, Aggregator |
| StatusAggregationComponent | 상태 통합 및 구조화 | Monitors, Cache, Alert |
| StatusPublishingComponent | ROS2 토픽 발행 | Aggregator, ROS2 |
| StatusCachingComponent | 상태 캐싱 및 히스토리 관리 | UI, ACS, Visualization |
| StatusAlertComponent | 이상 상태 감지 및 경고 | Aggregator, UI, ACS |
| VisualizationComponent | 상태 데이터 시각화 관리 | Cache, WebUI |
| WebUISyncComponent | UI 연동 및 데이터 전달 | Cache, 웹 클라이언트 |
| ACSReportComponent | ACS 연동 및 보고 | Cache, 외부 시스템 |

---

## 5. 최종 정리

- 인터페이스 기반 설계로 다양한 상태 모니터 및 시각화 확장 가능
- 안전 모니터링과 일반 상태 모니터링의 명확한 분리
- 상태 수집, 통합, 퍼블리시, 캐싱, 시각화, 외부 연계의 모듈화된 구조
- Web UI 및 3D 시각화 기능 강화
- 알림 및 이상 감지 기능 통합
- ACS와의 양방향 통신 지원
- 실시간 상태 모니터링 및 대응 능력 향상

---
