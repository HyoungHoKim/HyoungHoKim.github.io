---
layout: post
title:  "이득우 C++ Chapter02"
date:   2020-08-17 18:29:15 +0900
categories: [UNREAL]
---

이득우의 언리얼 C++ 게임 개발의 정석의 Chapter02 요약입니다.

### 언리얼 엔진의 구성요소
- 월드 : 공간, 시간, 물리, 렌더링을 제공
- 액터 : 콘텐츠를 구성하는 최소단위의 물체, 이름, 유형(액터의 클래스 유형), 트랜스폼, 프로퍼티(속성 값), 게임 로직(블루프린트, C++)
- 레벨 : 월드에 배치된 액터들의 집합
- 컴포넌트 : 시각적, 물리적, 움직임 크게 3가지로 나눌 수 있다.

### 주요 컴포넌트
- 스태틱 메시
- 스켈레탈 메시
- 콜리전
- 카메라
- 오디오
- 파티클 시스템
- 라이트
- 무브먼트

### 요약
- 이 책에서는 언리얼 프로젝트가 동작할 수 있는 최소 기능만 선언된 CoreMinimal.h 대신 다양한 엔진 기능이 지원되는 EngineMinimal.h를 사용한다. (4.15 버전부터는 헤더 파일의 참조로 인한 컴파일 시간과 인텔리센스 부하를 최소화하기 위해 CoreMinimal.h 공용 헤더 파일만 참조하도록 C++ 템플릿이 변경되었다.)
- C++ 프로그래밍에서는 포인터를 선언하면 명시적으로 객체를 소멸시켜야되지만 언리얼 엔진은 언리얼 실행 환경(런타임)을 통해 객체가 사용되지 않으면 할당된 메모리를 자동으로 소멸시키는 기능을 제공한다.
- 언리얼 실행 환경이 우리가 선언한 객체를 자동으로 관리하게 만들려면 코드에서 UPROPERTY라는 매크로를 사용해 객체를 지정해줘야한다. (모든 객체가 UPROPERTY 매크로를 써서 자동으로 메모리 관리할 수 있는 것은 아님, 언리얼 오브젝트만 가능하다.)
- C++클래스가 언리얼 오브젝트 클래스가 되는 방법
    - 클래스 선언 메크로 : 언리얼 오브젝트임을 명시하기 위해 클래스 선언 윗줄에 UCLASS라는 매크로 선언, 클래스 내부에는 GENERATED_BODY 매크로를 선언
    - 클래스 이름 접두사 : 규칙에 맞게 접두사가 붙어야 한다. 접두사는 크게 U와 A가 제공됨, A는 액터 클래스, U는 액터가 아닌 클래스에 사용된다.
    - generated.h 헤더파일 : 마지막 #include 구문에 이 헤더파일을 반드시 선언해야한다.
    - 외부 모듈에 공개 여부 : 윈도우 DLL 시스템은 DLL 내 클래스 정보를 외부에 공개할 지 결정하는 _declspec(dllexpert)라는 키워드를 제공한다. 언리얼 엔진에서 이 키워드를 사용하려면 '모듈명_API'라는 키워드를 선언앞에 추가해야한다. 이 키워드가 없으면 다른 모듈에서 해당 객체를 접근할 수 없다.
- 액터의 구축은 클래스 생성자 코드에서 진행, 언리얼은 new가 아닌 CreateDefaultSubobject API라는 함수를 사용한다.
    - Body = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("BODY"));
    - CreateDefaultSubobject에서 사용하는 문자열 값은 액터에 속한 컴포넌트를 구별하기 위한 Hash값 생성에 사용된다. 유일한 값을 지정해야한다.
- 루트 컴포넌트 지정 :
- 디테일 윈도우에서 컴포넌트 속성을 편집할려면 특별한 키워드를 등록해줘야한다.
    - UPROPERTY(VisibleAnywhere)
- 액터를 클릭 후 End키를 누르면 바닥에 붙는다.
- 생성자 코드에서 설정한 값은 언리얼 오브젝트의 기본값이 된다.
- 클래스 이름에 붙은 F접두사는 언리얼 오브젝트와 관련 없는 일반 C++ 클래스 혹은 구조체를 말한다.
- 언리얼 제공하는 대표적인 값 유형
    - 바이트 : uint8
    - 정수 : int32
    - 실수 : float
    - 문자열 : FString, FName
    - 구조체 : FVector, FRotator, FTransform
- 속성의 데이터를 변경하려면 EditAnywhere 키워드를 사용한다.
- 'Category=분류명' 규칙으로 키워드를 추가하면 지정한 분류에서 속성 값을 관리할 수 있다.
- 코드로 에셋 직접 연결하기
    - 언리얼 엔진은 에셋의 키값으로 경로값을 사용한다.
    - {오브젝트 타입}'{폴더명}/{파일명}.(에셋명)' : StaticMesh'/Game/InfinityBladeGrassLands/Environments/Plains/Env_Plains_Ruins/StaticMesh/SM_Plains_Castle_Fountain_01.SM_Plains_Castle_Fountain_01'
    - ConstructorHelpers라는 클래스의 FObjectFinder를 사용해 변수를 선언한다.
     ```
    ConstructorHelpers::FObjectFinder<UStaticMesh>SM_BODY(TEXT("/Game/InfinityBladeGrassLands/Environments/Plains/Env_Plains_Ruins/StaticMesh/SM_Plains_Castle_Fountain_01.SM_Plains_Castle_Fountain_01"));

    if (SM_BODY.Succeeded()) {
		Body->SetStaticMesh(SM_BODY.Object);
	}
    ```
    - 에셋의 경로 정보는 게임 실행 중에 변경될 일이 없기 때문에 static으로 선언해 한번만 초기화 해주는 것이 좋다.
