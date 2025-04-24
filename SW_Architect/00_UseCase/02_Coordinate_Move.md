# UC02 – 좌표 기반 이동 (Coordinate Move)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC02 |
| **유즈케이스 명** | 좌표 기반 이동 |
| **관련 기능 요구사항** | F1-2. 좌표 기반 이동 |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇이 사용자가 입력한 특정 위치(x, y, θ)로 이동할 수 있어야 한다. |

---

## 사전 조건

- 로봇이 정지 또는 대기 상태여야 한다.
- 이동 명령에 사용되는 좌표가 작업 공간 내 유효 범위여야 한다.
- 로봇과 웹 UI, FastAPI, ROS2 간 통신이 활성화되어 있어야 한다.

## 후 조건

- 로봇이 요청된 좌표로 이동을 완료한다.
- 웹 UI에 도착 위치, 이동 결과, 이동 성공 여부가 표시된다.

---

## 기본 흐름

1. 운영자가 Web UI에서 좌표를 입력하고 [이동] 버튼을 클릭한다.
2. Web UI는 FastAPI에 `POST /api/robot/move` 요청을 보낸다 (좌표 포함).
3. FastAPI는 해당 pose 정보를 포함하여 ROS2에 `/robot/cmd_move_pose` 토픽을 publish한다.
4. `Command Executor Node`는 이동 명령을 수신하고, 목표 좌표로의 동작 계획을 수립한다.
5. `H/W Node`는 해당 좌표로 로봇팔을 이동시킨다 (Rainbow Robotics PGM 사용).
6. 이동이 완료되면 상태 메시지가 `Status Node`에 전달된다.
7. `Status Node`는 "이동 완료" 상태를 FastAPI에 전달한다.
8. FastAPI는 상태를 Web UI로 전달하고, UI에 `"이동 완료"` 메시지를 표시한다.

---

## 예외 흐름

- **E1: 좌표 유효성 오류**
  - 입력한 좌표가 작업 가능 범위를 벗어난 경우
  - → UI에 “유효하지 않은 좌표입니다” 알림 표시

- **E2: 이동 중 충돌 가능 경로**
  - 이동 경로에 장애물이 있어 충돌 위험이 예상될 경우
  - → ROS2가 이동을 중단하고 경고 메시지를 전송  
  - → UI에 “이동 불가: 장애물 감지” 표시

- **E3: 이동 실패 (기계적 문제)**
  - 휠, 모터, 센서 오류 등으로 인해 이동이 완료되지 못한 경우
  - → UI에 “이동 실패” 메시지 + 상세 오류 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Web UI | FastAPI | HTTP POST | `/api/robot/move` (pose 포함) |
| FastAPI | ROS2 | ROS2 Topic | `/robot/cmd_move_pose` |
| Command Executor Node | H/W Node | ROS2 Action or Topic | 지정 pose로 이동 요청 |
| H/W Node | Status Node | ROS2 Topic | 이동 완료/실패 상태 전송 |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` = "completed" or "failed" |
| FastAPI | Web UI | WebSocket or 응답 | 이동 결과 표시 |

---

## 비고

- 좌표 이동은 작업 공간 내 안전한 경로 계산을 포함해야 한다.
- 향후 경로 시각화, 경로 미리보기 기능과도 연계될 수 있음.
- ROS2 이동 노드는 경로 계획 + 이동 제어 + 장애물 회피 기능을 포함해야 함.
