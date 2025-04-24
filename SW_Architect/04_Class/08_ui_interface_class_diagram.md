# 🧩 ui_interface_node 패키지 – Class Diagram (최종 자세한 설명 버전)

---

# ✅ 1. 클래스 목록 및 구조

### 📘 Interface (추상 클래스)
- `ICommandService` (명령 실행 서비스 인터페이스)
- `IStatusService` (상태 조회 서비스 인터페이스)
- `IScenarioService` (시나리오 선택 서비스 인터페이스)
- `IWebSocketManager` (WebSocket 연결 관리 인터페이스)

### 📘 Core Classes
- `APIServer` (FastAPI 인스턴스)
- `CommandRouter`
- `StatusRouter`
- `ScenarioRouter`
- `CommandService` (implements ICommandService)
- `StatusService` (implements IStatusService)
- `ScenarioService` (implements IScenarioService)
- `WebSocketManager` (implements IWebSocketManager)
- `StatusBroadcaster`

---

# ✅ 2. 인터페이스 상세 설명

## 📘 ICommandService (Interface)
- **책임**: 로봇 명령 전송 처리
- **행동**:
  - `+ send_pick_command(params: dict) -> bool`
  - `+ send_place_command(params: dict) -> bool`
- **협력**: CommandRouter
- **에러처리**: 명령 실패 시 False 반환
- **비고**: ROS2 서비스 호출 기반

## 📘 IStatusService (Interface)
- **책임**: 로봇 상태 조회 처리
- **행동**:
  - `+ get_pose_status() -> dict`
  - `+ get_battery_status() -> dict`
- **협력**: StatusRouter
- **에러처리**: 상태 수신 실패 시 빈 dict 반환
- **비고**: ROS2 Topic 구독 기반

## 📘 IScenarioService (Interface)
- **책임**: 시나리오 선택 요청 처리
- **행동**:
  - `+ select_scenario(name: str) -> bool`
- **협력**: ScenarioRouter
- **에러처리**: 선택 실패 시 False 반환
- **비고**: ROS2 서비스 호출 기반

## 📘 IWebSocketManager (Interface)
- **책임**: WebSocket 연결 및 상태 업데이트 브로드캐스트
- **행동**:
  - `+ connect(websocket) -> None`
  - `+ disconnect(websocket) -> None`
  - `+ broadcast(message: str) -> None`
- **협력**: StatusBroadcaster
- **에러처리**: 연결 실패/해제 실패 로깅
- **비고**: 다중 연결 관리 지원

---

# ✅ 3. Core Class 상세 설명

## 📘 APIServer
- **책임**: FastAPI 서버 인스턴스 관리
- **속성**:
  - `- app: FastAPI`
- **행동**:
  - `+ create_app() -> FastAPI`
- **협력**: CommandRouter, StatusRouter, ScenarioRouter
- **비고**: `/api/robot/`, `/api/status/`, `/api/scenario/` 엔드포인트 등록

## 📘 CommandRouter
- **책임**: 명령 관련 HTTP 엔드포인트 처리
- **행동**:
  - `+ pick_endpoint()`
  - `+ place_endpoint()`
- **협력**: CommandService
- **비고**: POST API 처리

## 📘 StatusRouter
- **책임**: 상태 조회 관련 HTTP 엔드포인트 처리
- **행동**:
  - `+ pose_endpoint()`
  - `+ battery_endpoint()`
- **협력**: StatusService
- **비고**: GET API 처리

## 📘 ScenarioRouter
- **책임**: 시나리오 선택 엔드포인트 처리
- **행동**:
  - `+ select_endpoint()`
- **협력**: ScenarioService
- **비고**: POST API 처리

## 📘 CommandService
- **책임**: 명령을 ROS2 서비스로 전송
- **행동**:
  - `+ send_pick_command(params: dict) -> bool`
  - `+ send_place_command(params: dict) -> bool`
- **협력**: ROS2 Service Client
- **비고**: 실패 시 에러 반환

## 📘 StatusService
- **책임**: ROS2 Topic으로부터 상태 수신
- **행동**:
  - `+ get_pose_status() -> dict`
  - `+ get_battery_status() -> dict`
- **협력**: ROS2 Topic Subscriber
- **비고**: 캐시 기반 조회 최적화

## 📘 ScenarioService
- **책임**: 시나리오 선택 ROS2 서비스 호출
- **행동**:
  - `+ select_scenario(name: str) -> bool`
- **협력**: ROS2 Service Client
- **비고**: 실패 시 에러 반환

## 📘 WebSocketManager
- **책임**: WebSocket 연결 관리 및 메시지 브로드캐스트
- **행동**:
  - `+ connect(websocket) -> None`
  - `+ disconnect(websocket) -> None`
  - `+ broadcast(message: str) -> None`
- **협력**: StatusBroadcaster
- **비고**: 다중 연결 관리 및 상태 동기화 지원

## 📘 StatusBroadcaster
- **책임**: 로봇 상태 업데이트를 WebSocket으로 전송
- **행동**:
  - `+ broadcast_status_update(status: dict) -> None`
- **협력**: WebSocketManager
- **비고**: 실시간 상태 변경 반영

---

# ✅ 4. 클래스 간 관계 요약 (UML 관계)

- APIServer → (Aggregation) → CommandRouter, StatusRouter, ScenarioRouter
- CommandRouter → (Association) → ICommandService
- StatusRouter → (Association) → IStatusService
- ScenarioRouter → (Association) → IScenarioService
- WebSocketManager → (Implements) → IWebSocketManager
- CommandService → (Implements) → ICommandService
- StatusService → (Implements) → IStatusService
- ScenarioService → (Implements) → IScenarioService
- StatusBroadcaster → (Aggregation) → WebSocketManager

---

# ✅ 5. 시스템 특성 반영

- FastAPI 서버 구동
- REST API + WebSocket 통신 동시 지원
- ROS2와의 서비스/토픽 연결 통합
- 상태 변경 실시간 반영 (WebSocket 브로드캐스트)

---

# ✅ 6. 예외/에러 상황 대응

| 상황 | 대응 방법 |
|:--|:--|
| ROS2 서비스 호출 실패 | HTTP 500 반환 |
| WebSocket 연결 실패 | 연결 해제 후 재시도 |
| 요청 파라미터 오류 | HTTP 400 반환 |
| 상태 수신 실패 | 캐시된 이전 데이터 반환 |

---

# 📢 최종 요약

- REST + WebSocket 이중 통신 체계 확립
- 인터페이스 기반으로 서비스 로직 분리
- ROS2 통신과 예외 대응 모두 체계적 설계 완료

---
