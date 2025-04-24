# UC12 – Place 명령 (Place Command)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC12 |
| **유즈케이스 명** | Place 명령 |
| **관련 기능 요구사항** | F5-2. Place 명령 |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇이 픽업한 물체를 지정된 위치(6D pose)에 정확히 내려놓는다. |

---

## 사전 조건

- 로봇이 이미 물체를 파지하고 있어야 한다 (Pick 완료 상태).
- 목표 위치가 시스템에 설정되어 있어야 한다.
- 로봇 동작 가능 상태(대기 또는 정지)여야 한다.

## 후 조건

- 물체가 지정된 위치에 안전하게 놓여진다.
- Place 성공 여부가 UI에 표시된다.

---

## 기본 흐름

1. 사용자가 Place 명령을 UI에서 실행하거나, Schedular Node가 Place 단계에 진입한다.
2. Command Executor Node는 Place 명령을 시작한다.
3. Place Pose가 미리 정의되어 있거나 전달된 좌표를 참조한다.
4. Command Executor Node는 motion_node에 해당 위치로 이동 및 내려놓기 명령을 보낸다.
5. motion_node는 로봇팔을 이동시키고 그리퍼를 열어 물체를 내려놓는다.
6. 결과 상태가 status_node에 전달된다.
7. FastAPI는 상태를 수신하여 Web UI에 결과를 표시한다.

---

## 예외 흐름

- **E1: 파지 상태 아님**  
  로봇이 물체를 잡고 있지 않은 경우  
  → UI에 “Place 실패: 파지 상태 아님” 표시

- **E2: 위치 오류**  
  Place 위치가 유효 범위를 벗어났거나, 접근 불가한 위치인 경우  
  → “지정 위치 오류” 메시지 표시

- **E3: 충돌 위험 감지**  
  이동 중 장애물 또는 충돌 가능성이 감지된 경우  
  → 동작 중지 및 “충돌 위험” 경고 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Schedular Node or UI | Command Executor Node | 내부 명령 or ROS2 Topic | Place 명령 |
| Command Executor Node | motion_node | ROS2 Action or Topic | 이동 + Place 실행 |
| motion_node | status_node | ROS2 Topic | 결과 상태 (성공/실패) |
| status_node | FastAPI | ROS2 Topic | `/robot/status` |
| FastAPI | Web UI | WebSocket or 응답 | Place 결과 표시 |

---

## 비고

- Place 위치는 사전 설정된 좌표 or 사용자 지정 좌표 사용 가능
- 안전을 위해 그리퍼 열기 전 정지 시간 또는 위치 검증 루틴 삽입 가능
- 성공 후 자동 초기화 또는 다음 작업으로 진행할 수 있음
