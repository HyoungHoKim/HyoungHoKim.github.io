---
layout: post
title:  "이득우 C++ Chapter03"
date:   2020-08-18 01:00:59 +0900
categories: [UNREAL, UNREAL/LDW_CPLUS]
---

이득우의 언리얼 C++ 게임 개발의 정석의 Chapter03 요약입니다.

### 로깅
- UE_LOG(카테고리, 로깅 수준, 형식 문자열, 인자.. )
- 로깅 수준은 크게 메시지(Log), 경고(Warning), 에러(Error)
- 언리얼 엔진은 로그 카테고리를 선언하기 위해 두개의 매크로를 제공, 하나는 선언부, 하나는 구현부에 사용, 보통 게임 모듈명으로 된 헤더 파일과 소스 파일에 선언하는 것이 일반적이다.
    - 헤더 부분 : DECLARE_LOG_CATEGORY_EXTERN(LDW_CPlus, Log, All);
    - 소스 부분 : DEFINE_LOG_CATEGORY(LDW_CPlus);
    - Log를 사용하는 액터 : UE_LOG(LDW_CPlus, Warning, TEXT("Actor Name : %s, ID : %d, Location X : %.3f"), *GetName(), ID, GetActorLocation().X);
- 매크로 정의
```
LDW_CPlus.h

#define ABLOG_CALLINFO (FString(__FUNCTION__) + TEXT("(") + FString::FromInt(__LINE__) + TEXT(")"))
#define ABLOG_S(Verbosity) UE_LOG(LDW_CPlus, Verbosity, TEXT("%s"), *ABLOG_CALLINFO);
#define ABLOG(Verbosity, Format, ...) UE_LOG(LDW_CPlus, Verbosity, TEXT("%s %s"), *ABLOG_CALLINFO, *FString::Printf(Format, ##__VA_ARGS__))
```

### 어셜션(Assertion)
- 반드시 확인하고 넘어가야 하는 점검 코드를 의미한다.
- 치명적인 오류가 발생했을 시 게임 실행을 중지하고 오류를 띄움
- 옵션에서 '디버킹을 위한 편집 기호'를 설치해야한다.
- 대표적으로 check가 있다.
- 가볍게 경고를 내릴 때는 ensure을 사용한다.

### 액터의 주요 이벤트 함수
- 액터를 구성하는 모든 컴포넌트의 세팅이 완료되면 PostInitializeComponents 함수를 호출한다.
- 액터가 게임에 참여할 때는 BeginPlay 함수를 호출하고, 매 프레임마다 액터의 Tick함수를 호출한다.
- 액터가 게임에서 퇴장할 때 EndPlay를 호출한다.
- 언리얼이 제공하는 액터 클래스에는 액터 동작의 뼈대를 이루는 BeginPlay, Tick과 같은 이벤트 함수들이 선언돼 있고, 이들은 자식이 상속 받을 수 있도록 가상 함수로 선언돼 있다.

### 움직이는 액터 설계
- 정보 은닉을 위해 변수를 private로 선언하면 컴파일 과정에서 에러가 발생한다. 향후 언리얼 에디터에서 변수 값을 변경하려면 UPROPERTY 매크로에 AllowPrivateAccess라는 META 키워드를 추가하면 이를 편집함과 동시에 변수 데이터를 은닉할 수 있어 캡슐화가 가능해진다.
```
private:
    UPROPERTY(EditAnywhere, Category=Stat, Meta = (AllowPrivateAccess = true))
    float RotateSpeed;
```
- Pitch : Y축
- Yaw : Z축
- Roll : X축
- 언리얼에서 시간을 관리하는 주체는 월드의 시간 관리자이다.
    - GetWorld()->GetDeltaSeconds() : DeltaSeconds 값을 가져올 수 있다.
    - GetWorld()->GetTimeSeconds() : 게임을 시작한 후 현재까지 경과된 시간
    - GetWorld()->GetUnpausedTimeSeconds() : 사용자가 게임을 중지한 시간을 제외한 경과 시간
    - GetWorld()->GetRealTimeSeconds() : 현실 세계의 경과 시간
    - GetWorld()->GetAudioTimeSeconds() : 사용자가 게임을 중지한 시간을 제외한 현실 세계의 경과 시간

### 무브먼트 컴포넌트 활용
- 액터의 움직임을 위해 Tick과 DeltaSeconds를 활용하는 것은 일반적이다.
- 언리얼 엔진에서는 움직임이라는 요소를 분리해 액터와 별도로 관리하도록 프레임워크를 구성함, 이것이 무브먼트 컴포넌트다.
- 무브먼트 컴포넌트 : 액터의 움직임에 대한 것을 책임, 액터는 무브먼트 컴포넌트가 제공하는 이동 메커니즘에 따라 이동
    - FloatingPawnMovement : 중력의 영향을 받지 않는 액터의 움직임을 제공한다.
    - RotatingMovement : 지정된 속도로 액터를 회전시킨다.
    - InterpMovement : 지정된 위치로 액터를 이동시킨다.
    - ProjectileMovement : 액터에 중력의 영향을 받아 포물선을 그리는 발사체의 움직임을 제공한다. 주로 총알이나 미사일에 사용된다.
