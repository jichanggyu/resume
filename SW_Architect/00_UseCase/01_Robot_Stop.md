# UC01 – 로봇 정지 (Robot Stop)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC01 |
| **유즈케이스 명** | 로봇 정지 (Emergency / Manual Stop) |
| **관련 기능 요구사항** | F1-1. 로봇 정지 |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇이 현재 수행 중인 모든 동작을 즉시 중단하도록 명령한다. |

---

## 사전 조건

- 로봇이 동작 중이거나 대기 중이어야 한다.
- 웹 UI와 로봇이 정상적으로 연결되어 있어야 한다.

## 후 조건

- 로봇이 모든 동작을 중지한 상태로 전환된다.
- 상태 표시창에 “정지됨(Stopped)”이 표시된다.
- 정지 후 재개하려면 별도의 Start 또는 Reset 명령이 필요하다.

---

## 기본 흐름

1. 운영자가 Web UI에서 [정지] 버튼을 클릭한다.
2. Web UI는 FastAPI에 `POST /api/robot/stop` 요청을 보낸다.
3. FastAPI는 ROS2 토픽 `/robot/cmd_stop`에 정지 명령을 발행한다.
4. `Schedular Node`는 현재 실행 중인 시나리오 또는 작업을 중단한다.
5. `Command Executor Node`는 현재 명령을 멈추고 동작을 중단한다.
6. `H/W Node`는 각 장치(팔, 허리 등)를 정지 상태로 전환한다.
7. `Status Node`는 현재 상태를 `"stopped"`로 갱신하여 상태 토픽으로 발행한다.
8. FastAPI는 해당 상태를 전달하고, UI는 이를 표시한다.
---

## 예외 흐름

- **E1: ROS 통신 장애**
  - FastAPI가 ROS2 노드와 통신할 수 없는 경우
  - → UI에 통신 오류 메시지를 출력하고 재시도 옵션 제공

- **E2: 로봇이 이미 정지 상태**
  - 이미 정지 상태인데 정지 명령을 추가로 보낸 경우
  - → UI에 “이미 정지 상태입니다” 메시지를 표시 (에러 아님)

- **E3: 정지 중 충돌 위험**
  - 작업 도중 정지 명령으로 인해 팔이 충돌 가능 위치에 정지할 수 있음
  - → 정지와 함께 “주의: 위험 위치에 정지됨” 경고 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Web UI | FastAPI | HTTP POST | `/api/robot/stop` |
| FastAPI | ROS2 | ROS2 Topic | `/robot/cmd_stop` |
| Schedular Node | Command Executor Node | 내부 제어 요청 | 작업 중단 명령 |
| Command Executor Node | H/W Node | ROS2 Topic or Action | 장치 정지 명령 |
| H/W Node | 내부 hardware_node | 하드웨어 I/O | 각 장치 정지 |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` = "stopped" |
| FastAPI | Web UI | WebSocket or Polling | 정지 완료 상태 전달 |

---

## 비고

- 정지 기능은 안전과 직결되므로 1초 이내 응답을 목표로 한다.
- 필요 시 하드웨어 기반 정지(비상 정지 버튼 등)와 통합될 수 있음.
- ROS2 제어 노드는 정지 명령을 수신했을 때, 현재 동작을 안전하게 취소 또는 중단해야 한다.
