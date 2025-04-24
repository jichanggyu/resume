# UC11 – Pick 명령 (Pick Command)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC11 |
| **유즈케이스 명** | Pick 명령 |
| **관련 기능 요구사항** | F5-1. Pick 명령 |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | 로봇이 지정된 대상 물체를 비전 기반 파지점에서 픽업한다. |

---

## 사전 조건

- Pick 대상 물체가 카메라 시야 내에 존재해야 한다.
- 비전 시스템이 활성화되어 있어야 하며, 파지점 추출이 가능해야 한다.
- 로봇 팔이 대기 상태이거나 다른 작업 수행 중이 아니어야 한다.

## 후 조건

- 로봇이 파지점에 도달하고 물체를 안정적으로 파지한다.
- Pick 성공 여부가 UI에 표시되고 다음 동작으로 이어질 준비가 된다.

---

## 기본 흐름

1. 사용자가 Pick 명령을 UI에서 실행하거나, Schedular Node가 Pick 단계에 진입한다.
2. Command Executor Node는 Pick 명령을 시작한다.
3. Vision Node에 파지점을 요청한다 (특징점 추출 → 후보 생성 → 최적 파지점 선택).
4. Vision Node는 최적 파지점(Pose) 데이터를 반환한다.
5. Command Executor Node는 H/W Node에 해당 파지점으로 이동 및 파지 명령을 보낸다.
6. H/W Node는 로봇팔을 움직여 파지를 수행한다.
7. 결과 상태가 Status Node에 전달된다.
8. FastAPI는 상태를 수신하여 Web UI에 결과를 표시한다.

---

## 예외 흐름

- **E1: 파지 대상 없음**  
  비전 시스템이 유효한 파지점을 찾지 못함  
  → UI에 “파지 대상 없음” 경고 표시

- **E2: 파지 실패**  
  파지 동작이 실패하거나 슬립 발생  
  → “파지 실패” 메시지 및 재시도 옵션 표시

- **E3: 비전 처리 오류**  
  카메라 입력 또는 특징점 추출 실패  
  → “비전 오류” 경고 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Schedular Node or UI | Command Executor Node | 내부 명령 or ROS2 Topic | Pick 명령 |
| Command Executor Node | Vision Node | ROS2 Service or Topic | 파지점 요청 |
| Vision Node | Command Executor Node | 응답 | 최적 파지 좌표 (Pose) |
| Command Executor Node | H/W Node | ROS2 Action or Topic | Pick 수행 명령 |
| H/W Node | Status Node | ROS2 Topic | 결과 상태 (성공/실패) |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` |
| FastAPI | Web UI | WebSocket or 응답 | Pick 완료 상태 표시 |

---

## 비고

- 파지점 좌표는 6D pose (x, y, z, rx, ry, rz) 형식으로 사용
- 파지 성공률이 낮은 경우 신뢰도 기반 파지점 선택 알고리즘 적용 가능
- Pick 실패 시 자동 재시도 또는 대기 모드로 전환 가능
