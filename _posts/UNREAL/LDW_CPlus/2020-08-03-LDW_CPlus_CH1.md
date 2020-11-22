---
layout: post
title:  "이득우 C++ Chapter01"
date:   2020-08-03 22:11:15 +0900
categories: [UNREAL, UNREAL/LDW_CPLUS]
---

이득우의 언리얼 C++ 게임 개발의 정석이란 책입니다. 전에 봤던 책이지만 복습 겸 빠르게 보고 갈 계획입니다.

### 프로젝트 구성 폴더
- Config : 게임 프로젝트의 설정 값을 보관하는 공간입니다. 이 폴더를 제거하면 게임 프로젝트의 중요한 설정 정보가 날아가므로 항상 보관해야 합니다.

- Content : 게임 프로젝트에 사용하는 에셋을 관리하는 공간입니다. 항상 보관해야 합니다.
- Intermediate : 프로젝트 관리에 필요한 임시 파일을 저장하는 공간입니다. 이 폴더는 제거해도 에디터에 의해 자동으로 재생성됩니다.

- Saved : 에디터 작업 중에 생성된 결과물을 저장하는 공간입이다. 예를 들어 세이브 파일, 스크린 샷은 모두 이곳에 저장됩니다. 이 폴더를 제거해도 게임 프로젝트에는 영향을 주지 않습니다.

- Binaries : C++ 코드가 컴파일된 결과물을 저장하는 공간입니다. 이 폴더는 삭제해도 빌드 할때 마다 새롭게 생성됩니다.

- Source : C++ 소스 코드가 위치한 공간입니다. C++ 소스 외에도 언리얼 엔진은 독특한 빌드 설정을 담은 C# 소스 파일이 있으며, 폴더를 삭제할 때 프로젝트 구성이 망가지므로 주의해야합니다.

- 프로젝트이름.sln : C++ 프로젝트를 관리하기 위한 비주얼 스튜디오의 솔루션 파일입니다. 솔루션이 관리하는 각 프로젝트 파일은 Intermediate 폴더 내 ProjectFiles 폴더에 있습니다. 프로젝트 파일과 솔루션 파일은 삭제하더라도 uproject 파일을 우클릭해 뜨는 Generate Visual project file 메뉴를 선택하면 언제든지 재생성할 수 있습니다.

### 기본 상식
- 언리얼에서는 C++ 코드를 컴파일한 결과물을 모듈이라 한다. 게임 로직을 담은 모듈에는 특별히 게임 모듈이라는 명칭을 사용한다.
- 모빌리티 값을 무버블로 변경하면 라이트맵 시스템을 사용하지 않고 실시간으로 라이트를 계산한다. 대부분의 프로젝트에서 무버블 라이트를 사용하는 경우는 드물다.
- 세팅 -> 프로젝트 세팅 -> 맵&모드 -> Default Maps 섹션을 변경하면 처음 로드 되는 레벨을 바꿀 수 있다.