---
layout: post
title:  "Oculus와 언리얼 엔진을 사용한 VR개발_C03"
date:   2020-12-11 21:07:27 +0900
categories: [UNREAL, UNREAL/OCULUS]
---

### 시작하며
- 언리얼 엔진 공식 홈페이지 온라인 러닝 프로젝트 중 \<Oculus와 언리얼 엔진을 사용한 VR개발\>에 대한 요약 및 정리입니다.
- 블루프린트로 진행되며, 프로젝트를 하는데 필요한 에셋은 모두 제공됩니다. 코스 설명대로 따라하면 다운받을 수 있습니다.
- 추후 C++로 바꿔볼 예정입니다.
- 관련 링크 : https://www.unrealengine.com/en-US/onlinelearning-courses/vr-development-with-oculus-and-unreal-engine

### 머티리얼 렌더링
- Opaque Rendering ( 불투명 렌더링 )
  - 가장 퍼포먼스가 높은 입체 오브젝트에는 Opaque Rendering을 사용하며 전후 분류를 통해 오큘루전된 픽셀의 픽셀 셰이더 계산은 하지 않습니다.
- Masked Rendering
  - 투명 오브젝트에는 Masked Rendering을 쓰는데 이 마스크는 100% 불투명이 아니면 100% 투명한 픽셀을 생성하며 중간이란게 없습니다.
- MSAA가 활성화되어 있다면 언리얼 엔진4는 픽셀 셰이더에서 단순한 텍스처 제거나 인스트럭션 클립만 하는게 아니라 픽셀 디시전별 바이너리 결과가 나옵니다.
- MSAA 수준에 따라 셰이드가 더 많이 보일 텐데 이게 알파값을 기반으로 MSAA 샘플을 비활성화해서 살짝 더 부드러운 트랜지션이 이루어집니다.
- 4X MSAA는 쎄이드를 세개만 제공해서 가장자리가 큰 화면 영역을 커버할 때 벤딩이 일어납니다.
- Translucent Rendering ( 반투명 렌더링 )
  - Translucent Rendering은 완전 불투명과 완전 투명 사이의 오브젝트라면 무엇이든 쓰일 수 있습니다. 이건 퍼포먼스에 가장 큰 영향을 미치기 때문에 게임 최저화에는 가장 추천하지 않는 방법입니다.
- Alpha Transparency, Additive and Modulate
  - 알파 투명이나 애디티브, 또는 모듈레이트의 경우 반투명의 대체재 중에 퍼포먼스가 최고입니다.
  - 중요한 포인트는 에디티브는 머티리얼을 밝히기만 하고 모듈레이트는 어둡게만 함.
-  장점과 단점
  - 이런 방식은 벤딩과 노이즈를 줄여주지만 픽셀 보정 오클루전이나 그림자 드리우기 같은 기능을 망치기도 합니다. 언리얼 엔진에는 오브젝트별 렌더링과 부드러운 섀도를 제공하여 이런 프로퍼티를 계산하는 솔루션이 있습니다. 이 방법이 퍼포먼스에 미치는 영향은 그 오브드로에 따라 달라지며 단순히 콘텐츠 기반일 뿐만 아니라 뷰 기반이기도 합니다.
- Things To Consider
  - Resolution
  - MSAA quality level
  - Content
- 디더링은 다수의 프레임에 따라 바뀌면서 이 패턴이 덜 표시되게 하고 그 대신 시머링을 둘 수 있습니다. 하지만 고정 포비티드 렌더링을 쓸 때 최적의 수단은 아니며 고주파 패턴이 있다면 해상도의 급변이 훨씬 잘 보이기 때문입니다.
- 언리얼 엔진은 안티 에일리어싱 디스턴스 필드와 마스크트 렌더링도 지원합니다.
- Distance Field fonts
  - 디스턴스 필드 폰트로는 3D 텍스트를 훨씬 더 높은 퀄리티와 속도로 렌더링할 수 있습니다.
  - 언리얼 엔진으로 텍스트 렌더 액터로 쓸 새 폰트를 임포트할 때는 반드시 오프라인 캐시 폰트로 설정되어야 합니다. 폰트가 오프라인으로 설정되어 있으면 엔진은 사용할 폰트 스타일과 크기의 임포트를 요청하고 해당 파일 내의 모든 글자가 표시된 텍스처를 생성합니다. 그런 다음 레벨의 텍스트 렌더 액터로 글자를 빌드한 다음 이 텍스처를 샘플링할 수 있습니다. 모든 폰트가 출력 가능한건 아니니 Create Printable Only에 꼭 체크하세요. Use Distance Field Alpha를 체크하면 가까이 있거나 큰 사이즈의 텍스트 렌더가 개선되면서 멋진 모습을 하게 됩니다.
  - 머티리얼 블렌드 모드는 UI 위젯에도 적용이 됩니다.