# UC15 – 허리 정렬 (자세 보정) 명령 (Waist Alignment Command)

## 개요

| 항목 | 내용 |
|------|------|
| **유즈케이스 ID** | UC15 |
| **유즈케이스 명** | 허리 정렬 (자세 보정) 명령 |
| **관련 기능 요구사항** | F5-4. 허리 정렬 (자세 보정) |
| **주 액터** | 운영자 (Web UI 사용자) |
| **목적** | AGV 또는 로봇의 기초 위치 오차를 보정하여, 로봇 상체(팔)가 정밀한 작업 위치에 맞춰 정렬되도록 한다. |

---

## 사전 조건

- 현재 로봇의 위치 오차를 측정할 수 있어야 한다 (ex. 비전 기반 마커, 기준물, 위치센서).
- 허리 보정용 모션이 정의되어 있어야 한다.
- 로봇이 정지 상태이거나 대기 상태여야 한다.

## 후 조건

- 로봇 상체가 기준 위치에 대해 정렬된다.
- 이후 Pick/Place 등의 동작 정확도가 향상된다.

---

## 기본 흐름

1. 운영자가 Web UI에서 허리 정렬 명령을 실행하거나, 시나리오 내에서 명령이 트리거된다.
2. Command Executor Node는 허리 정렬 명령을 수신하고 실행을 시작한다.
3. Command Executor Node는 Vision Node에 보정 오차 요청을 보낸다.
4. Vision Node는 현재 이미지 기반으로 허리 오차를 추출하여 오차값(예: Δθ)을 반환한다.
5. Command Executor Node는 기준 위치 + 보정 오차를 계산하여 목표 각도를 생성한다.
6. H/W Node에 허리 회전 명령을 보낸다 (예: 0° + Δθ).
7. H/W Node는 해당 각도로 허리를 회전시키고 완료 여부를 판단한다.
8. 결과는 status_node를 통해 FastAPI로 전달되며, UI에 표시된다.

---

## 예외 흐름

- **E1: 기준 위치 인식 실패**  
  기준 마커 또는 위치센서 인식 실패 시  
  → “보정 실패: 기준 위치 인식 불가” 경고 표시

- **E2: 허리 정렬 동작 중 충돌 위험**  
  회전 중 주변 장애물 감지  
  → 즉시 동작 중지 및 “충돌 위험” 표시

- **E3: 보정 범위 초과**  
  현재 오차가 허용 보정 범위를 초과하는 경우  
  → “정렬 불가: 위치 보정 한계 초과” 경고 표시

---

## 통신 흐름

| 송신자 | 수신자 | 방식 | 메시지 |
|--------|--------|------|--------|
| Schedular Node or UI | Command Executor Node | 내부 명령 or ROS2 Topic | 허리 정렬 명령 |
| Command Executor Node | Vision Node | ROS2 Service or Topic | 오차값 요청 |
| Vision Node | Command Executor Node | 응답 | 오차값(Δθ 등) |
| Command Executor Node | H/W Node | ROS2 Topic or Action | 허리 보정 회전 명령 |
| H/W Node | Status Node | ROS2 Topic | 정렬 완료 or 실패 |
| Status Node | FastAPI | ROS2 Topic | `/robot/status` |
| FastAPI | Web UI | WebSocket or 응답 | 상태 표시 |

---

## 비고

- 허리 보정 동작은 회전, 슬라이드 또는 둘 다 포함될 수 있음
- 보정 기준은 환경에 따라 AR 마커, QR 코드, 기준 위치 센서 등 다양하게 설정 가능
- 보정 후 로봇의 위치 정밀도가 향상되며, 이후 작업의 성공률을 높임
