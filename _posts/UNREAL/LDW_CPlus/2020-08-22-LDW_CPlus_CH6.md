---
layout: post
title:  "이득우 C++ Chapter06"
date:   2020-08-22 16:58:14 +0900
categories: [UNREAL, UNREAL/LDW_CPLUS]
---

이득우의 언리얼 C++ 게임 개발의 정석의 Chapter06 요약입니다.

### 캐릭터 모델
- 언리얼 엔진에서는 인간형 좀 더 효과적으로 제작하기 위해 character라는 특수한 모델을 제공한다.
- character는 pawn의 상속을 받고 있다. capsule 컴포넌트와 skeletalMesh 컴포넌트를 사용하고 있으며, characterMovement라는 컴포넌트를 사용해 움직임을 관리한다.
- CharacterMovement
    - 점프와 같은 중력을 반영한 움직임을 제공한다.
    - 다양한 움직임을 설정할 수 있다.
        - 걷기
        - 기어가기
        - 날아가기
        - 수영하기
- 멀티 플레이 네트워크 환경에서 캐릭터들의 움직임을 자동으로 동기화한다.

### 컨트롤 회전의 활용
- 플레이어의 컨트롤러는 주로 게임 세계의 물리적인 요소를 고려하지 않은 플레이어의 의지와 관련된 데이터를 관리한다.
- 폰은 게임 세계에서 물리적 제약을 가지기 때문에 현재 캐릭터가 처한 물리적인 상황을 관리한다.
- 폰이 관리하는 대표적인 속성 중 하나가 속도이다. 속도는 현재 폰의 이동 상태를 알려주는 중요한 데이터 중 하나다.
- 플레이어 컨트롤러는 플레이어의 의지를 나타내는 컨트롤 회전(Control Rotation)이라는 속성을 제공한다.
- 입력에 따라 변화되는 컨트롤 회전 값은 캐릭터의 카메라 설정에서 다양하게 사용된다.
- 언리얼 엔진의 캐릭터 모델은 기본으로 컨트롤 회전의 Yaw 회전 값과 폰의 Yaw 회전이 서로 연동되어 있다. 이를 지정하는 속성이 액터의 Pawn 섹션에 위치한 UseControllerRotationYaw이다. 이 설정으로 인해 마우스를 좌우로 움직이면 캐릭터는 Z축으로 회전하지만, 마우스의 상하 이동은 폰의 회전에 아무런 영향을 주지 않는다.

### 삼인칭 컨트롤 구현 (GTA 방식)
- 캐릭터의 이동 : 현재 보는 시점을 기준으로 상하, 좌우 방향으로 마네킹이 이동하고 카메라는 회전하지 않음
- 캐릭터의 회전 : 캐릭터가 이동하는 방향으로 마네킹이 회전함
- 카메라의 지지대의 길이 : 450cm
- 카메라의 회전 : 마우스 상하좌우 이동에 따라 카메라의 지지대가 상하좌우로 회전
- 카메라 줌 : 카메라 시선과 캐릭터 사이에 장애물이 감지되면 캐릭터가 보이도록 카메라를 장애물 앞으로 줌인
- SpringArm 속성 설정
```
void AABCharacter::SetControlMode(int32 ControlMode) {
	if (ControlMode == 0) {
		SpringArm->TargetArmLength = 450.0f;
		SpringArm->SetRelativeRotation(FRotator::ZeroRotator);
		SpringArm->bUsePawnControlRotation = true;
		SpringArm->bInheritPitch = true;
		SpringArm->bInheritRoll = true;
		SpringArm->bInheritYaw = true;
		SpringArm->bDoCollisionTest = true;
		bUseControllerRotationYaw = false; // true로 바꾸면 카메라 시선에 맞춰 캐릭터가 같이 회전한다.
	}
}
```
- GTA 방식에서는 SpringArm의 회전 값은 옵션에 의해 컨트롤 회전 값과 동일하므로 컨트롤 회전 값이 카메라가 바라보는 방향이라고 할 수 있다. 이 회전 값으로부터 회전 행렬을 생성한 후, 원하는 방향 축을 대입해 캐릭터가 움직일 방향 값을 가져올 수 있다. 언리얼 엔진에서 시선 방향은 X축, 우측 방향은 Y축을 의미한다.
```
void AABCharacter::UpDown(float NewAxisValue) {
	AddMovementInput(FRotationMatrix(GetControlRotation()).GetUnitAxis(EAxis::X), NewAxisValue);
}

void AABCharacter::LeftRight(float NewAxisValue) {
	AddMovementInput(FRotationMatrix(GetControlRotation()).GetUnitAxis(EAxis::Y), NewAxisValue);
}
```
- 위의 방식으로 시선 방향으로 잘 움직이지만 현재 캐릭터의 회전 기능이 없기 때문에 월드의 X축을 향해 바로보며 이동한다. 이는 캐릭터 무브먼트 컴포넌트의 OrientRotationToMovement 기능을 사용하면 손쉽게 해결할 수 있다. 회전 속도를 함께 지정해 캐릭터가 부드럽게 회전하도록 기능을 추가한다.
```
GetCharacterMovement()->bOrientRotationToMovement = true;
GetCharacterMovement()->RotationRate = FRotator(0.0f, 720.0f, 0.0f);
```

### 삼인칭 컨트롤 구현(디아블로 방식)
- 캐릭터의 이동 : 상하좌우 키를 조합해 캐릭터가 이동할 방향을 결정
- 캐릭터의 회전 : 캐릭터는 입력한 방향으로 회전
- 카메라의 길이 : 조금 떨어진 800cm
- 카메라 회전 : 카메라의 회전 없이 항상 고정된 시선으로 45도로 내려다봄
- 카메라 줌 : 없음. 카메라와 캐릭터 사이에 장애물이 있는 경우 외곽선으로 처리
- GTA 방식에서는 상하 키 입력과 좌우 키 입력을 각각 처리했지만, 이번에는 상하 키와 좌우 키를 조합해 캐릭터의 회전과 이동이 이뤄져야 한다.
- 언리얼은 FRotationMatrix는 회전된 좌표계 정보를 정보를 저장하는 행렬이다. 앞서 살펴본 GTA 방식에서는 FRotator 값으로 회전 행렬을 생성하고, 이를 토대로 변환된 좌표계의 X, Y축 방향을 가져왔다. 디아블로 방식에서는 거꾸로 하나의 벡터 값과 이에 직교하는 나머지 두축을 구해 회전 행렬을 생성하고, 이와 일치하는 FRotator 값을 얻어오는 방식을 사용했다. 하나의 벡터로부터 회전 행렬을 구축하는 명령은 MakeFromX, Y, Z가 있는데, 두 축의 입력을 합산한 최종 벡터 방향과 캐릭터의 시선 방향(X축)이 일치해야 하므로 이 중에서 MakeFromX가 사용됐다.
- 현재 캐릭터는 키보드 조합에 따라 최소 45도 단위로 끊어져서 회전한다. UseControllerDesireRotation 속성을 체크하면 컨트롤 회전이 가리키는 방향으로 캐릭터가 부드럽게 회전한다.

### 컨트롤 설정의 변경
- 언리얼 엔진은 액션 매핑 입력 설정과 연동하도록 BindAction이라는 함수를 제공하는데, 이 함수에는 버튼이 눌렸는지(EInputEvent::IE_Pressed), 떼어졌는지(EInputEvent::IE_Released)에 대한 부가 인자를 지정할 수 있다.
```
PlayerInputComponent->BindAction(TEXT("ViewChange"), EInputEvent::IE_Pressed, this, &AABCharacter::ViewChange);
```

- 뷰 전환 시 부드럽게 변경하기 위해서 회전값이 목표값까지 각각의 목표 설정 값으로 서서히 변경되도록 FMath 클래스에서 제공하는 InterpTo 명령어를 사용한다.
    - float형 : FInterpTo
    - vector형 : VInterpTo
    - rotator형 : RInterpTo
- 컨트롤 회전 값은 GTA 방식에서는 SpringArm 회전에 사용하고, 디아블로 방식에서는 캐릭터의 방향에 사용된다.

### 기타 메모
- 게임에 플레이어가 입장하면 플레이어를 대변하는 플레이어 컨트롤러가 주어지고, 플레이어 컨트롤러가 조작하는 가상 세계의 물리적인 물체인 Pawn이 등장한다는 것을 설명드렸습니다. 플레이어 컨트롤러는 우리 눈에도 보이지 않는 무형적인 물체이며, 현실 세계 플레이어의 입력과 출력을 담당한다는 역할을 한다고 설명했습니다. 플레이어 컨트롤러의 추가 용도를 이야기하자면 플레이어의 의지(Will)를 담은 정신적인 물체로도 활용합니다. 이러한 플레이어의 의지는 수치화되어서 어딘가에 저장이 되어야 하는데요, 플레이어의 의지 중 자주 사용하는 것 중 하나는 폰의 물리적인 위치에 상관 없이 플레이어가 보고자하는 방향에 대한 정보입니다. 언리얼 엔진은 이를 컨트롤러의 회전 값인 Control Rotation에 보관하도록 설계되어 있습니다.
컨트롤러의 회전을 잘 활용하면 캐릭터 조작에 있어서 다양한 연출을 보여줄 수 있게 되는데요, 지금 우리가 작업하는 3인칭 게임의 경우 가상 세계의 물리적 공간에서 폰이 바라보는 방향과 현실 세계에 플레이어가 바라보고자 하는 방향이 항상 일치 할 수는 없습니다. 즉 게임의 기획에 따라 우리가 보고자하는 방향과 폰이 보는 방향이 같을 수도, 다를 수도 있습니다.
언리얼 엔진의 캐릭터 모델에서는 일단 우리가 보고자하는 방향과 폰이 보는 방향을 Z축 회전인 Yaw 회전에 한해 일치시키도록 설계해놨습니다. 이 것이 액터 설정의 Pawn 섹션에 있는 Use Controller Yaw Rotation 설정입니다.
    - [출처](https://m.blog.naver.com/PostView.nhn?blogId=destiny9720&logNo=220903956593&proxyReferer=https:%2F%2Fwww.google.com%2F) : 저자 블로그
