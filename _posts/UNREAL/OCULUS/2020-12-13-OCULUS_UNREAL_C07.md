---
layout: post
title:  "Oculus와 언리얼 엔진을 사용한 VR개발_C07"
date:   2020-12-13 01:49:49 +0900
categories: [UNREAL, UNREAL/OCULUS]
---

### 시작하며
- 언리얼 엔진 공식 홈페이지 온라인 러닝 프로젝트 중 \<Oculus와 언리얼 엔진을 사용한 VR개발\>에 대한 요약 및 정리입니다.
- 블루프린트로 진행되며, 프로젝트를 하는데 필요한 에셋은 모두 제공됩니다. 코스 설명대로 따라하면 다운받을 수 있습니다.
- 추후 C++로 바꿔볼 예정입니다.
- 관련 링크 : https://www.unrealengine.com/en-US/onlinelearning-courses/vr-development-with-oculus-and-unreal-engine

### 표준 컨트롤러 이동 구현
- 프로젝트 세팅 -> Input -> Axis Mappings
  - VR 템플릿은 일반적인 모션 컨트롤러에 필요한 모든 축 매핑을 제공합니다.
  - 'MotionControllerThumbLeft_Y'을 보면 이 축 매핑의 이름만 봐도 어떤 입력을 받아들이는 것인지 알 수 있습니다. 전통적으로 플레이어를 앞뒤로 움직이는데 사용됐습니다.
  - 'MotionControllerThumbLeft_X'는 좌우 회전에 사용됐습니다.
  - 'MotionControllerThumbRight_X'는 카메라 회전에 사용됩니다.
  - 이 세가지 이벤트의 역할은 모든 프레임에서 호출되어 각 입력의 'Axis Value'를 확인할 겁니다. 플레이어가 지시하지 않는한 아무것도 하지 않게 만드는 간단한 솔루션이 필요합니다.
- MotionContorollerPawn
  - 이 세가지 이벤트의 역할은 모든 프레임에서 호출되어 각 입력의 'Axis Value'를 확인할 겁니다.
    - 'MotionControllerThumbLeft_Y'
    - 'MotionControllerThumbLeft_X'
    - 'MotionControllerThumbRight_X'
  - 플레이어가 지시하지 않는한 아무것도 하지 않게 만드는 간단한 솔루션이 필요합니다.
  - 복제 가능하게 만들고 싶다면 'LocomotionMethod'라는 새 매크로를 생성합니다.
    - Input에 두개 만듭니다.
      - 'Then'이란 이름의 exec(실)
      - 'Axis'란 이름의 float
    - Output에 exec 3개를 만듭니다.
      - 'Teleport'
      - 'Controller'
      - 'Controller - Doing Nothing'
    - 여기서는 이 매크로에서 나오는 루트선택이 'ELocomotionMethod' 값에 의존하게 됩니다.
    - 메서드가 표준 컨트롤러 이동인 경우 플레이어가 썸스틱을 0이 아닌 위치에 두었는지 여부에 따라 루트를 선택할 겁니다.
    - 이 매크로는 'Axis Value'가 모든 프레임에서 실행되는 것을 방지할 뿐만 아니라 플레이어가 다른 이동을 선택했을 경우 나머지 이동도 막아줍니다.


<img src="https://user-images.githubusercontent.com/49055264/102004668-2a3c2a80-3d56-11eb-8a75-891e7c08c29c.PNG" ><br/>


  - 생성한 매크로를 텔레포트에 대한 Pressed 이벤트 뒤에 추가합니다.
  - 또한 위에 생성한 세 이벤트에도 똑같이 추가합니다.
    - 'AddActorWorldOffset' 노드를 생성하고 'Sweep'을 'true'로 설정합니다.
    - 'Delta Location'에서는 가려는 방향을 정하고 평면을 제한한 다음 속도를 곱할 것입니다.
      - Camera -> Get Right Vector -> Project Vector On To Plane('Plane Normal'을 0, 0, 1으로 설정해서 평평한 바닥으로부터 직선으로 올라가는 평면으로 만들어 움직임을 바닥 평면으로 제한합니다.)
      - 이 결과에 이벤트의 'Axis Value'를 곱합니다.
      - MotionControllerThumbLeft_Y 는 X와 동일하지만 Get Right Vector를 Get Forward Vector로 바꾸면 됩니다.


<center><img src="https://user-images.githubusercontent.com/49055264/102004669-2c9e8480-3d56-11eb-90fb-773480c59e01.PNG"></center>


  - 카메라 회전은 Z값만 사용합니다. Map Range Clamped를 이용해 범위를 지정합니다.
    - In Range A = -1 (가장 왼쪽)
    - In Range B = 1 (가장 오른쪽)
    - Rotation Speed는 0.5에 불과한데 이로 인해 회전이 느려져서 물리적으로 머리를 움직이지 않고 카메라만 회전하여 멀미를 유발하는 일을 줄일 수 있습니다.

  - 카메라 비그넷을 위한 매커니즘
    - 카메라 컴포넌트
    - Vignette Intensity = 2.5
    - Post Process Blend Weight = 0
      - 필요할 때만 활성화됨.


  <center><img src="https://user-images.githubusercontent.com/49055264/102004670-2dcfb180-3d56-11eb-8fca-e10849cde60d.PNG"></center>


### 팁
- 변수를 만들 때 순서는 중요하진 않지만 관련된 것들을 가까이에 두는 것이 좋습니다.
- 스팀 VR 과 오큘러스 앱을 함께 켜면 입력값이 제대로 들어가지 않는다. 오큘러스 앱만 키도록 하자!!
