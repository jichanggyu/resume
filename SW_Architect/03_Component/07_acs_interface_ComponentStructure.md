# ACSInterfaceNode 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### ACSInterfaceNodeComponent
- 구성 요소: ACSInterfaceNode
- 책임: ROS2 노드로서 ACS 인터페이스 전체 관리 및 조정

### ACSWebSocketCommunicationComponent
- 구성 요소: WebSocketClient
- 책임: ACS 시스템과 WebSocket 연결 유지 및 메시지 수신/전송

### ACSCommandHandlingComponent
- 구성 요소: ACSCommandReceiver
- 책임: 수신된 명령 메시지를 파싱하고 SchedulerNode로 전달

### ACSStatusReportingComponent
- 구성 요소: ACSStatusReporter
- 책임: 현재 로봇 상태를 주기적으로 수집하여 ACS로 전송

---

## 2. 컴포넌트 상세 설명

### ACSInterfaceNodeComponent
- **구성 요소**: ACSInterfaceNode
- **역할**:
  - ACS 인터페이스 시스템 전체 제어 및 관리
  - ROS2 노드로서 다른 노드들과 통신
  - WebSocket 통신 및 명령/상태 흐름 조정
- **입력**:
  - `+ start_communication() -> None`
  - `+ handle_incoming_message(message: str) -> None`
- **출력**:
  - 명령 처리 및 상태 보고 흐름 제어
- **협력**:
  - ACSWebSocketCommunicationComponent
  - ACSCommandHandlingComponent
  - ACSStatusReportingComponent

### ACSWebSocketCommunicationComponent
- **구성 요소**: WebSocketClient
- **역할**:
  - ACS 서버와 WebSocket 연결 유지
  - 메시지 송수신 처리
  - 연결 복구 및 예외 처리
- **입력**:
  - `+ connect(uri: str) -> None`
  - `+ receive_message() -> str`
  - `+ send_message(message: str) -> bool`
- **출력**:
  - 수신된 메시지
  - 연결 상태 정보
- **협력**:
  - ACSInterfaceNodeComponent
  - ACSCommandHandlingComponent
  - ACSStatusReportingComponent

### ACSCommandHandlingComponent
- **구성 요소**: ACSCommandReceiver
- **역할**:
  - WebSocket 메시지를 JSON 형태로 파싱
  - 메시지 유효성 검증
  - SchedulerNode에 시나리오 명령 전달
- **입력**:
  - `+ parse_and_dispatch(message: str) -> None`
- **출력**:
  - SchedulerNode 명령 호출
- **협력**:
  - ACSInterfaceNodeComponent
  - ACSWebSocketCommunicationComponent
  - SchedulerNode (외부)

### ACSStatusReportingComponent
- **구성 요소**: ACSStatusReporter
- **역할**:
  - StatusNode로부터 로봇 상태 수집
  - 상태 데이터 포맷팅 및 처리
  - WebSocket을 통해 상태 전송
- **입력**:
  - `+ gather_status() -> dict`
  - `+ send_status() -> None`
- **출력**:
  - ACS로 전송될 포맷팅된 상태 데이터
- **협력**:
  - ACSInterfaceNodeComponent
  - ACSWebSocketCommunicationComponent
  - StatusNode (외부)

---

## 3. 컴포넌트 간 관계 요약

- ACSInterfaceNodeComponent → ACSWebSocketCommunicationComponent (통신 요청)
- ACSInterfaceNodeComponent → ACSCommandHandlingComponent (명령 처리 요청)
- ACSInterfaceNodeComponent → ACSStatusReportingComponent (상태 보고 요청)
- ACSWebSocketCommunicationComponent → ACSCommandHandlingComponent (수신 메시지 전달)
- ACSStatusReportingComponent → ACSWebSocketCommunicationComponent (상태 전송 요청)
- ACSCommandHandlingComponent → SchedulerNode (외부 명령 전달)
- ACSStatusReportingComponent → StatusNode (외부 상태 요청)

---

## 4. 요약

| 컴포넌트 | 책임 | 주요 협력 대상 |
|:--|:--|:--|
| ACSInterfaceNodeComponent | ACS 인터페이스 전체 관리 | 모든 ACS 컴포넌트 |
| ACSWebSocketCommunicationComponent | WebSocket 연결 및 메시지 송수신 | ACSInterface, Command, Status |
| ACSCommandHandlingComponent | 명령 파싱 및 실행 요청 전달 | ACSInterface, WebSocket, Scheduler |
| ACSStatusReportingComponent | 로봇 상태 수집 및 전송 | ACSInterface, WebSocket, Status |

---

## 5. 최종 정리

- ROS2 노드로서의 ACSInterfaceNode를 중심으로 명확한 책임 분리
- WebSocket 기반 명령 수신 및 상태 전송 흐름을 독립 컴포넌트로 분리
- 연결 관리, 명령 처리, 상태 보고의 명확한 분리로 유지보수성 향상
- 예외 상황(연결 끊김, 메시지 오류 등)에 대한 체계적 대응 구조
- 향후 추가될 수 있는 새로운 명령 유형이나 상태 형식에 유연하게 대응 가능

---
