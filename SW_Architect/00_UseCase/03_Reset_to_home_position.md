# UC03 – 초기화 위치 이동 (Reset to Home Position)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC03 |
| **유즈케이스 명** | 초기화 위치 이동 |
| **관련 기능 요구사항** | F1-3. 초기화 위치 이동 |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇을 안전한 초기 위치(홈 포지션)로 복귀시켜 다음 작업을 준비한다. |

---

## 사전 조건

- 로봇이 동작 중이 아니거나 정지 상태여야 한다.
- 홈 위치(기본 자세)의 좌표값이 사전에 시스템에 등록되어 있어야 한다.
- 로봇과 UI, 백엔드, ROS2 간 통신이 정상 상태여야 한다.

## 후 조건

- 로봇이 지정된 초기 위치(6D pose: x, y, z, rx, ry, rz)로 이동한다.
- UI에서 로봇이 초기 위치에 도달했음을 확인할 수 있다.
- 이후 작업 명령 수행이 가능해진다.

---

## 기본 흐름

1. 운영자가 Web UI에서 [초기화] 버튼을 클릭한다.
2. Web UI는 FastAPI에 `POST /api/robot/reset_pose` 요청을 보낸다.
3. FastAPI는 ROS2에 `/robot/cmd_reset_pose` 토픽으로 초기화 명령을 발행한다.
4. `Command Executor Node`는 사전 정의된 pose를 참조하여 이동 명령을 생성한다.
5. `H/W Node`는 로봇팔 또는 관련 장치를 해당 위치로 이동시킨다.
6. 이동 완료 후 상태가 `Status Node`로 전달된다.
7. FastAPI는 상태를 수신하고, Web UI에 `"초기화 완료"` 메시지를 표시한다.

---

## 예외 흐름

- **E1: 초기화 좌표 미설정**  
  시스템에 초기 위치 값이 정의되어 있지 않은 경우  
  → UI에 “초기 위치가 설정되지 않았습니다” 오류 표시

- **E2: 경로 상 장애물 존재**  
  초기 위치로 이동하는 경로에 장애물이 존재하는 경우  
  → ROS2가 이동을 중단하고 경고 메시지 전송  
  → UI에 “초기화 이동 실패: 경로 상 장애물 감지” 표시

- **E3: 이동 실패**  
  로봇이 초기 위치로 이동 중 기계적 문제 발생  
  → UI에 오류 메시지 및 현재 위치 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Web UI | FastAPI | HTTP POST | `/api/robot/reset_pose` |
| FastAPI | ROS2 | ROS2 Topic | `/robot/cmd_reset_pose` |
| Command Executor Node | H/W Node | ROS2 Topic or Action | 초기화 위치 이동 명령 |
| H/W Node | Status Node | ROS2 Topic | "초기화 완료" 또는 "실패" |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` = "home_completed" |
| FastAPI | Web UI | WebSocket or 응답 | 결과 상태 표시 |

---

## 비고

- 초기화 위치는 작업 시작 전 안전한 기준 자세로 중요함.
- 필요 시 각 작업 전에 자동 초기화 과정을 포함하도록 설계 가능.
- ROS2에서는 MoveIt, Nav2 등을 활용한 포즈 기반 이동 제어 가능.
