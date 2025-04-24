## 🧩 vision_node 패키지 고급 설계 문서 (최종 수정본)

---

### 1. 📌 책임 (Responsibility)

`vision_node` 패키지는 카메라 영상 수신부터 이미지 처리, 특징점 추출, 파지점 후보 생성 및 최적 파지점 계산까지 비전 기반 로봇 인식 기능을 담당한다.  
주요 책임은 다음과 같다:

- 외부 카메라로부터 실시간 이미지 프레임 수신
- 이미지 처리 파이프라인을 통한 전처리 및 분석
- HALCON 알고리즘을 기반으로 이미지 내 특징점 검출
- 파지 가능한 후보 좌표 리스트 생성
- 가장 적합한 grasp point 계산 및 응답
- 결과를 service 형태로 제공하여 command_executor, task_manager에서 호출 가능
- 처리 결과 시각화 데이터 발행

---

### 2. 🔗 패키지 의존 관계 (Dependencies)

| 연결 대상 패키지 | 연결 방식 | 설명 |
|------------------|-----------|------|
| `h_w_node` | 카메라 장치 드라이버 통합 | 실시간 이미지 스트림 제공 |
| `command_executor` | ROS2 Service 요청 처리 | Pick 시 grasp point 요청 |
| `task_manager` | 선택적 사전 조건 판단에 사용 | 파지 준비 상태 판단 |
| `status_reporter` | ROS2 Topic Publish | 비전 시스템 상태 정보 발행 |

---

### 3. 📬 외부 인터페이스 정의 (Interfaces)

#### ✅ 제공하는 ROS2 서비스

| 서비스명 | 타입 | 설명 |
|----------|------|------|
| `/vision/process_image` | `custom_interfaces/srv/ProcessImage` | 이미지 처리 및 결과 반환 |
| `/vision/extract_features` | `custom_interfaces/srv/ExtractFeatures` | 특징점 추출 및 반환 |
| `/vision/generate_grasp_points` | `custom_interfaces/srv/GenerateGraspPoints` | 파지점 생성 및 반환 |

#### ✅ 발행하는 ROS2 토픽

| 토픽명 | 타입 | 설명 |
|--------|------|------|
| `/vision/processed_image` | `sensor_msgs/msg/Image` | 처리된 이미지 |
| `/vision/features` | `custom_msgs/msg/FeatureList` | 추출된 특징점 리스트 |
| `/vision/grasp_points` | `custom_msgs/msg/GraspPointList` | 생성된 파지점 리스트 |
| `/vision/status` | `custom_msgs/msg/VisionStatus` | 비전 시스템 상태 정보 |

#### ✅ 사용하는 내부 인터페이스

| 대상 | 방식 | 설명 |
|-------|------|------|
| `h_w_node` | 이미지 수신 스트림 | 카메라 드라이버 또는 센서 SDK 연동 |
| `halcon` 라이브러리 | 알고리즘 실행 | 특징점 검출 및 분석 수행 |

---

### 4. ⚙️ 내부 구성 요소 (컴포넌트 구조)

#### 인터페이스 (추상 클래스)
- `IImageProcessor` (이미지 처리 인터페이스)
- `IFeatureExtractor` (특징점 추출 인터페이스)
- `IGraspPointGenerator` (그래스프 포인트 생성 인터페이스)

#### 핵심 클래스
| 컴포넌트명 | 타입 | 역할 |
|------------|------|------|
| `VisionNode` | ROS2 Node | 비전 시스템 전체 관리 및 ROS2 서비스/토픽 제공 |
| `ImageProcessingPipeline` | Class | 이미지 처리 파이프라인 관리 |
| `FeatureExtractionManager` | Class | 특징점 추출 관리 |
| `GraspPointManager` | Class | 그래스프 포인트 생성 및 관리 |
| `VisionDataPublisher` | Class | 비전 처리 결과 발행 |
| `HalconInterface` | Class | Halcon 소프트웨어와의 통합 인터페이스 |

#### 구현체 클래스
- `EdgeDetector` (implements IImageProcessor)
- `ColorSegmentation` (implements IImageProcessor)
- `HalconFeatureExtractor` (implements IFeatureExtractor)
- `HalconGraspGenerator` (implements IGraspPointGenerator)

---

### 5. 🔄 동작 흐름 예시 (Grasp Point 요청 시)

1. command_executor가 `/vision/generate_grasp_points` 서비스 호출
2. `VisionNode`가 호출 수신
3. `ImageProcessingPipeline`이 이미지 전처리 수행 (EdgeDetector, ColorSegmentation 등)
4. `FeatureExtractionManager`가 `HalconFeatureExtractor`를 통해 특징점 추출
5. `GraspPointManager`가 `HalconGraspGenerator`를 통해 파지점 생성
6. 생성된 파지점 랭킹 및 최적 파지점 선택
7. 결과를 서비스 응답으로 반환
8. `VisionDataPublisher`가 처리 결과를 토픽으로 발행

---

### 6. 📊 클래스 간 관계 및 상호작용

- VisionNode는 ImageProcessingPipeline, FeatureExtractionManager, GraspPointManager, HalconInterface를 포함(Aggregation)
- ImageProcessingPipeline은 여러 IImageProcessor 구현체들과 연결(Association)
- FeatureExtractionManager는 IFeatureExtractor 구현체와 연결
- GraspPointManager는 IGraspPointGenerator 구현체와 연결
- HalconFeatureExtractor와 HalconGraspGenerator는 HalconInterface를 참조하여 Halcon 기능 활용

---

### 7. ⚠️ 예외/에러 상황 대응

| 상황 | 대응 방법 |
|------|----------|
| 이미지 수신 실패 | 에러 로그 및 상태 알림 |
| 이미지 처리 실패 | 재시도 또는 에러 보고 |
| 특징점 추출 실패 | Halcon 대체 알고리즘 시도 |
| 그래스프 생성 실패 | 이전 성공한 결과 사용 |
| Halcon 연결 실패 | 내부 백업 알고리즘으로 전환 |

---

### 8. 📢 최종 요약

- 인터페이스 기반 설계로 유연한 비전 처리 구조 제공
- 확장 가능한 이미지 처리 파이프라인 구현
- Halcon 소프트웨어 통합으로 고급 비전 기능 활용
- 특징점 추출 및 그래스프 포인트 생성 기능 강화
- 처리 결과 시각화를 위한 데이터 발행
- 다양한 예외 상황 대응 메커니즘 구현
- UseCase 요구사항 완벽 반영