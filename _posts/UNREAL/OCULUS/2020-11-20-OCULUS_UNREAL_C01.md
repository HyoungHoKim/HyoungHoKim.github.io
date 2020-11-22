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
