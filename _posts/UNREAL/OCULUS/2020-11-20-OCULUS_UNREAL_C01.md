---
layout: post
title:  "Oculus와 언리얼 엔진을 사용한 VR개발_C01"
date:   2020-09-27 18:22:27 +0900
categories: [UNREAL, UNREAL/OCULUS]
---

### 시작하며
- 언리얼 엔진 공식 홈페이지 온라인 러닝 프로젝트 중 \<Oculus와 언리얼 엔진을 사용한 VR개발\>에 대한 요약 및 정리입니다.
- 블루프린트로 진행되며, 프로젝트를 하는데 필요한 에셋은 모두 제공됩니다. 코스 설명대로 따라하면 다운받을 수 있습니다.
- 추후 C++로 바꿔볼 예정입니다.
- 관련 링크 : https://www.unrealengine.com/en-US/onlinelearning-courses/vr-development-with-oculus-and-unreal-engine

### 프로젝트 세팅(Oculus.ver)
- 플러그인에서 Oculus VR, Oculus Audio, Online Subsystem Oculus 활성
- 플러그인->온라인 플랫폼에서 Oculus가 아닌 모든 서브시스템을 비활성화
- 프로젝트 세팅
    - Forward Shading 활성화
        - VR 프로젝트에서 일반적으로 쓰지 않는 기능들을 비활성화해서 속도와 효율성을 높혀줌, 이는 VR에서 필요한 높은 렌더 해상도와 프레임을 유지시키는데 도움을 줌
    - Motion Blur 비활성화
        - Forward Render는 Motion Blur를 지원하지 않음. 또한 VR 환경에서는 좋은 기능이 아닌데, 이는 카메라 또는 사용자의 머리가 움직일 때마다 사용자의 뷰가 블러 처리되기 때문임.
    - Instance Stereo Rendering 활성화
        - VR에는 각 눈에 해당하는 카메라가 두 개 있는데, Instance Stero Rendering을 사용하면, 두 카메라가 렌더 데이터의 특정 영역을 공유하여 draw thread duration를 단축함.
    - Anti Aliasing Method->MSAA
        - MSAA는 Forward Rendering 작업이 가능하며, 대부분의 GPU에서 좋은 퀄리티를 제공함.
        - MSAA는 GPU의 메모리를 활용하여 스무딩 효과를 연출함. 메모리 스펙이 낮은 그래픽 카드의 경우 퍼포먼스에 영향을 줄 수 있음.
    - 지원 플랫폼에서는 사용할 플랫폼만 선택해주는 것이 좋다. 이 강좌에서는 Window만 선택한다. 다만 Oculus Quest용으로 개발 중이라면 Android도 포함해야 합니다.
    - Start in VR 활성화
    - Smooth Frame Rate 비활성화
        - 타깃 하드웨어와 FPS가 변하는 상황에서는 도움이 됨.
        - 고정된 프레임 레이트에서는 유용하지 않음.
    - Use Fixed Frame Rate 비활성
        - 이 옵션은 FPS 한계를 설정함. 하드웨어가 해당 FPS를 얻지 못하면 FPS는 게임의 클럭 속도를 낮춰 프레임 레이트를 맞춤.
    - Custom TimeStamp -> None으로 설정
        - 프레임 단위의 시간이 중요한 애플리케이션에 유용함. VR에서는 이 기능이 필요 없습니다.
    - Quest용으로 개발 중이라면 Mobile HDR이 비활성화 되고 Mobile Multi-View가 활성화 되야됨. 이렇게 해도 Rift용 빌드에는 영향을 미치지 않습니다.

#### 강의 제공 프로젝트를 실행할 때
    - DefaultEngine.ini -> OnlineSubsystem 영역에 OculusAppID와 RiftAppID를 추가해야함. 추가하지 않고 빌드된 것을 실행한다면 오류가 발생할 수 있음.
    - Quest나 Go로 하는 경우에는 Unreal Engine -> Engine -> Config -> Android -> AndroidEngine.ini에도 추가해야함.

### VR에서 데모 레벨 탐색
- 블루 프린트 노드 상단에 작은 시계 모양이 있으면 비동기 노드라는 뜻이다.
    - 비동기 노드 : 즉각적인 함수가 아니라 완료하는데 시간이 걸린다는 뜻.
- Vertify Entitlement Node : DefaultEngine.ini에서 앱 ID를 찾음.
- 원하는 영역에 노드에 드래그 후 c키를 누르면 comment 기능이 활성화 된다.
- If Entitlement Fails
    - 조직의 관리자 또는 개발자인 사용자로 Oculus 소프트웨어 로그인 했는지 확인함.
    - Engine.ini 파일에서 App ID가 올바른지도 확인해야함.

### 핵심 에셋 생성
- GameInstance
    - 세션 처리, 맵 이동 또는 리더보드의 데이터 검색과 같은 함수를 수행하는 것 뿐만 아니라 변수를 보유하는 것이 가장 중요함.
    - 게임 전반적으로 영구적인 특수 프로퍼티가 있음. 따라서 레벨 변경 시에도 해당 게임 인스턴스에 따라 이전에 구성했던 변수 값을 유지할 수 있음.
- GameMode()
    - 사용할 맵의 로우 레벨 블루프린트 전체를 지정하며, 게임의 다양한 규칙을 포함하고 있음.
    - 컨트롤러가 소유해야 할 폰도 식별
- GameState
    - 대부분 게임플레이 중 변경되는 다양한 프로퍼티를 추적하는데 사용되며 모든 것들과 관련이 있음.
    - 게임을 시작한 플레이어의 Oculus ID를 추적하는데도 사용됨.
- PlayerState
    - 관련된 데이터 유지를 감독하며 누구나 액세스할 수 있음.
    - 플레이어 점수, 손 색깔, 이름 등이 포함되어 있음
- 열거형 에셋(Enumeration)
    - 이 열거형 에셋을 사용하면 선택한 아이템을 기반으로 로직을 변경할 수 있음.
- Structure
    - 하나 이상의 다른 변수를 포함하고 있는 변수
    - 개발 중 필요한 모든 다양한 변수를 포함함.
- Transform Variable
    - 3개의 변수 : 위치, 회전, 스케일

### 폰 생성
- 고급 복사 : 복사하는 오브젝트와 관련된 모든 레퍼런스 및 오브젝트들이 함께 복사됨.
- Inheritance(상속)
    - 모든 블루프린트 클래스는 부모 클래스로부터 모두 상속함
    - 간단하게 말하면, 생성되는 모든 블루프린트에는 몇 가지 코드나 블루프린트로 이미 생성된 부모가 있어 상속받아 작업을 수행할 수 있습니다.
- Object
    - 가장 하위 기본 클래스
- Actor
    - 기본적인 블루프린트로, 게임에 필요한 오브젝트를 생성하는데 사용함.
    - 맵으로 배치하고 일반적인 작업을 할 수 있게 하는 코드가 포함되어 있음.
- 부모 함수를 오버라이드하려고 할 때 부모 클래스에서 원래 로직도 호출하려면 부모 호출을 호출해야 함. (Add Call Parent Function)

### 핵심 에셋의 작동
- GameMode_Darts
    - 폰을 스폰할 위치
    - 스폰할 폰
- Arrow Component
    - 게임에서는 숨겨져 있지만 위치와 액터가 향하는 방향을 간편하게 볼 수 있게 함.
    - 원하는 맵에 폰을 스폰하고 원하는 방향을 향하도록 함.
- 변수 옆에 눈모양을 누르면 인스턴스 편집이 가능함.
    - 인스턴스 편집 기능을 활용하면 개별 인스턴스 각각 서로 다른 세팅을 적용할 수 있음.
- Ctrl을 누르고 변수를 드래그하면 get으로
- Alt를 누르고 변수를 드래그하면 set으로
