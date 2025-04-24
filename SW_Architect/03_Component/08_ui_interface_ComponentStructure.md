# ui_interface_node 패키지 – Component Structure (최종 수정본)

---

## 1. 컴포넌트 목록

### UIInterfaceNodeComponent
- 구성 요소: ROS2 노드, FastAPI 애플리케이션 관리
- 책임: ROS2 통합 및 API 서버 전체 생명주기 관리

### APIServerComponent
- 구성 요소: FastAPI 인스턴스, 라우터 등록, CORS 설정
- 책임: HTTP 요청 처리 및 라우팅 구성

### CommandServiceComponent
- 구성 요소: CommandRouter, ICommandService 구현체
- 책임: 로봇 Pick/Place/Move 등 명령 요청 처리 및 ROS2 서비스 호출

### StatusServiceComponent
- 구성 요소: StatusRouter, IStatusService 구현체
- 책임: 로봇 상태 조회 요청 처리 및 상태 캐싱

### ScenarioServiceComponent
- 구성 요소: ScenarioRouter, IScenarioService 구현체
- 책임: 시나리오 선택, 시작, 중지 요청 처리

### WebSocketManagerComponent
- 구성 요소: WebSocketHandler, 연결 관리
- 책임: 클라이언트 연결 관리 및 실시간 메시지 브로드캐스트

### AuthenticationComponent
- 구성 요소: AuthMiddleware, TokenValidator
- 책임: API 접근 인증 및 권한 관리

### ConfigurationComponent
- 구성 요소: ConfigManager, 환경 설정
- 책임: 서버 구성 설정 관리 및 로딩

---

## 2. 컴포넌트 상세 설명

### UIInterfaceNodeComponent
- 구성 요소: ROS2 노드, FastAPI 애플리케이션
- 역할:
  - ROS2 노드로서 API 서버 생명주기 관리
  - ROS2 서비스/토픽과 API 레이어 연결
- 입력:
  - ROS2 파라미터 (포트, 호스트 등)
  - ROS2 토픽 구독 (상태 업데이트)
- 출력:
  - ROS2 서비스 클라이언트 요청
  - 로그 메시지
- 협력:
  - APIServerComponent
  - CommandServiceComponent
  - StatusServiceComponent
  - ScenarioServiceComponent

### APIServerComponent
- 구성 요소: FastAPI 객체, Middleware 설정
- 역할:
  - API 서버 설정 및 시작
  - 라우터 등록 및 CORS 설정
  - 예외 처리 미들웨어 관리
- 입력:
  - `+ create_app() -> FastAPI`
  - 설정 파라미터 (포트, 호스트 등)
- 출력:
  - FastAPI 애플리케이션 인스턴스
- 협력:
  - 모든 서비스 컴포넌트
  - AuthenticationComponent
  - WebSocketManagerComponent

### CommandServiceComponent
- 구성 요소: CommandRouter, CommandServiceImpl
- 역할:
  - 로봇 작업 명령 API 엔드포인트 제공
  - 명령 파라미터 검증 및 변환
  - ROS2 서비스 호출 처리
- 입력:
  - `+ POST /api/command/pick`
  - `+ POST /api/command/place`
  - `+ POST /api/command/move`
  - `+ POST /api/command/cancel`
- 출력:
  - `+ execute_command(command_type, params) -> CommandResult`
- 협력:
  - UIInterfaceNodeComponent (ROS2 서비스 클라이언트)
  - 로봇 제어 관련 ROS2 노드들

### StatusServiceComponent
- 구성 요소: StatusRouter, StatusServiceImpl, StatusCache
- 역할:
  - 로봇 상태 조회 API 제공
  - 상태 데이터 캐싱 및 관리
  - 상태 변경 감지 및 WebSocket 알림 트리거
- 입력:
  - `+ GET /api/status/robot`
  - `+ GET /api/status/system`
  - `+ GET /api/status/task`
  - ROS2 토픽 (상태 업데이트)
- 출력:
  - `+ get_status(status_type) -> StatusData`
  - 상태 변경 이벤트 (WebSocket 푸시용)
- 협력:
  - UIInterfaceNodeComponent (ROS2 토픽 구독)
  - WebSocketManagerComponent

### ScenarioServiceComponent
- 구성 요소: ScenarioRouter, ScenarioServiceImpl
- 역할:
  - 시나리오 관리 API 제공
  - 시나리오 목록 조회 및 선택
  - 시나리오 실행/중지 요청 처리
- 입력:
  - `+ GET /api/scenario/list`
  - `+ POST /api/scenario/select`
  - `+ POST /api/scenario/start`
  - `+ POST /api/scenario/stop`
- 출력:
  - `+ list_scenarios() -> List[ScenarioInfo]`
  - `+ select_scenario(scenario_id) -> bool`
  - `+ execute_scenario() -> bool`
  - `+ stop_scenario() -> bool`
- 협력:
  - UIInterfaceNodeComponent (ROS2 서비스 클라이언트)
  - 스케줄러 노드

### WebSocketManagerComponent
- 구성 요소: WebSocketHandler, ConnectionManager
- 역할:
  - WebSocket 연결 수립 및 관리
  - 클라이언트 그룹 관리 및 메시지 브로드캐스트
  - 연결 상태 모니터링
- 입력:
  - `+ WebSocket /ws/status`
  - 상태 변경 이벤트 (StatusServiceComponent로부터)
- 출력:
  - `+ broadcast_message(message_type, data) -> None`
  - `+ send_to_client(client_id, message) -> bool`
- 협력:
  - StatusServiceComponent
  - WebSocket 클라이언트 (브라우저/앱)

### AuthenticationComponent
- 구성 요소: AuthMiddleware, TokenValidator
- 역할:
  - API 요청 인증 처리
  - 토큰 검증 및 권한 확인
  - 접근 제한 관리
- 입력:
  - HTTP 요청 헤더 (Authorization)
  - 인증 설정
- 출력:
  - 인증 결과 (성공/실패)
  - 사용자 컨텍스트
- 협력:
  - APIServerComponent
  - 모든 라우터 컴포넌트

### ConfigurationComponent
- 구성 요소: ConfigManager, 환경 변수 처리
- 역할:
  - 서버 설정 파일 로드 및 파싱
  - 환경 변수 기반 설정 오버라이드
  - 설정 값 접근 인터페이스 제공
- 입력:
  - 설정 파일 (YAML/JSON)
  - 환경 변수
  - ROS2 파라미터
- 출력:
  - `+ get_config(key, default=None) -> Any`
  - `+ load_config() -> dict`
- 협력:
  - UIInterfaceNodeComponent
  - APIServerComponent

---

## 3. 컴포넌트 간 관계 요약

- **UIInterfaceNodeComponent**
  - 전체 시스템의 진입점으로 ROS2와 API 서버 연결
  - APIServerComponent 초기화 및 관리
  - ROS2 서비스 클라이언트 및 토픽 구독자 관리

- **APIServerComponent**
  - 모든 서비스 컴포넌트의 라우터 등록
  - 미들웨어 및 예외 처리 설정
  - WebSocket 엔드포인트 설정

- **StatusServiceComponent**
  - WebSocketManagerComponent에 상태 변경 알림
  - UIInterfaceNodeComponent를 통해 ROS2 상태 정보 수신

- **CommandServiceComponent** & **ScenarioServiceComponent**
  - UIInterfaceNodeComponent의 ROS2 서비스 클라이언트 활용
  - 사용자 요청을 ROS2 명령으로 변환

- **WebSocketManagerComponent**
  - 클라이언트 연결 관리 및 실시간 상태 전송
  - StatusServiceComponent로부터 상태 업데이트 수신

- **AuthenticationComponent**
  - 모든 API 요청의 인증 전처리
  - APIServerComponent의 미들웨어로 동작

- **ConfigurationComponent**
  - 모든 컴포넌트에 설정 정보 제공
  - UIInterfaceNodeComponent 초기화 시 로드

---

## 4. 요약

| 컴포넌트 | 책임 | 주요 협력 대상 |
|:--|:--|:--|
| UIInterfaceNodeComponent | ROS2-API 통합 및 생명주기 관리 | APIServerComponent, ROS2 노드들 |
| APIServerComponent | API 서버 구동 및 라우터 관리 | 모든 서비스 컴포넌트 |
| CommandServiceComponent | 로봇 명령 API 처리 | UIInterfaceNodeComponent |
| StatusServiceComponent | 상태 조회 및 캐싱 | WebSocketManagerComponent |
| ScenarioServiceComponent | 시나리오 관리 API 처리 | UIInterfaceNodeComponent |
| WebSocketManagerComponent | 실시간 통신 관리 | StatusServiceComponent, 클라이언트 |
| AuthenticationComponent | API 접근 인증 및 권한 관리 | APIServerComponent |
| ConfigurationComponent | 서버 설정 관리 | 모든 컴포넌트 |

---

## 최종 정리

- REST API와 WebSocket을 통한 이중 통신 체계 구현
- ROS2와 웹 인터페이스 간의 견고한 연결 구조 확립
- 인터페이스를 통한 서비스 계층 분리로 유연성 확보
- 상태 캐싱 및 실시간 알림으로 효율적인 상태 관리
- 보안 및 설정 관리를 위한 전용 컴포넌트 도입
