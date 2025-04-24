# 🧩 ACSInterfaceNode 패키지 – Class Diagram (최종 수정본)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Core Classes
- `ACSInterfaceNode` (rclpy.Node, ACS와 WebSocket 통신)
- `WebSocketClient` (WebSocket 연결 및 메시지 수신)
- `ACSCommandReceiver` (수신 명령 해석 및 분배)
- `ACSStatusReporter` (로봇 상태 ACS로 주기 전송)

---

# ✅ 2. Core Class 상세 설명

## 📘 ACSInterfaceNode
- **책임**: ACS 시스템과 WebSocket을 통한 명령 수신 및 상태 전송 제어
- **속성**:
  - `- websocket_client: WebSocketClient`
  - `- command_receiver: ACSCommandReceiver`
  - `- status_reporter: ACSStatusReporter`
- **행동**:
  - `+ start_communication() -> None`
  - `+ handle_incoming_message(message: str) -> None`
- **협력**: WebSocketClient, ACSCommandReceiver, ACSStatusReporter
- **비고**: 연결 복구, 메시지 포맷 검증 포함

## 📘 WebSocketClient
- **책임**: ACS 서버와의 WebSocket 연결 관리
- **행동**:
  - `+ connect(uri: str) -> None`
  - `+ receive_message() -> str`
- **협력**: ACSInterfaceNode
- **비고**: Keep-alive, 재연결 기능 내장 가능

## 📘 ACSCommandReceiver
- **책임**: 수신한 명령 메시지를 파싱 및 분배
- **행동**:
  - `+ parse_and_dispatch(message: str) -> None`
- **협력**: TaskManagerNode (시나리오 선택, 작업 명령 실행)
- **비고**: 명령 포맷(JSON 등) 유효성 검증

## 📘 ACSStatusReporter
- **책임**: 로봇 상태를 ACS로 주기적으로 전송
- **행동**:
  - `+ gather_status() -> dict`
  - `+ send_status() -> None`
- **협력**: StatusPublisherNode
- **비고**: Pose, Battery, Communication 상태 포함

---

# ✅ 3. 클래스 간 관계 요약 (UML 관계)

- ACSInterfaceNode → (Aggregation) → WebSocketClient, ACSCommandReceiver, ACSStatusReporter
- ACSCommandReceiver → (Association) → TaskManagerNode
- ACSStatusReporter → (Association) → StatusPublisherNode

---

# ✅ 4. ROS2 및 시스템 특성 반영

- ACSInterfaceNode는 ROS2 Node (rclpy.Node)로 동작
- WebSocket을 통한 명령 수신/상태 전송
- ROS2 내부 서비스 호출 통해 TaskManagerNode 제어
- StatusPublisherNode를 참조해 상태 데이터 수집

---

# ✅ 5. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| WebSocket 연결 실패 | 재시도 및 에러 로깅 |
| 메시지 파싱 실패 | 무시 및 경고 로그 출력 |
| 상태 전송 실패 | 다음 주기까지 대기 후 재전송 시도 |

---

# 📢 최종 요약

- ACS와의 통신을 WebSocket 단위로 완전 분리
- 명령 수신 / 상태 전송을 각각 전문 클래스로 분리
- ROS2 노드 간 연계 흐름 (TaskManagerNode, StatusPublisherNode)까지 체계적으로 반영

---
