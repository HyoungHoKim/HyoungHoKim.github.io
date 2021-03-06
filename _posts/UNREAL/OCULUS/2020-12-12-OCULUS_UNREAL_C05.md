---
layout: post
title:  "Oculus와 언리얼 엔진을 사용한 VR개발_C05"
date:   2020-12-12 19:18:39 +0900
categories: [UNREAL, UNREAL/OCULUS]
---

### 시작하며
- 언리얼 엔진 공식 홈페이지 온라인 러닝 프로젝트 중 \<Oculus와 언리얼 엔진을 사용한 VR개발\>에 대한 요약 및 정리입니다.
- 블루프린트로 진행되며, 프로젝트를 하는데 필요한 에셋은 모두 제공됩니다. 코스 설명대로 따라하면 다운받을 수 있습니다.
- 추후 C++로 바꿔볼 예정입니다.
- 관련 링크 : https://www.unrealengine.com/en-US/onlinelearning-courses/vr-development-with-oculus-and-unreal-engine

### VR 속 이동 디자인
- VR 이동에 겪는 일반적인 문제는 모션 멀미입니다.
- Motion Sickness
  - 모션 멀미 또는 시뮬레이션 멀미는 기존 게임보다 VR에서 훨씬 흔하게 나타납니다. 게임 카메라가 플레이어의 머리에 있고 플레이어는 움직임을 보면서 자기 자신이 움직인다고 느끼기 때문입니다.
  - VR 환경에서는 카메라가 곧 사용자의 뷰임을 기억해야 합니다. 따라서 시네마틱 카메라 또는 시점의 위치나 방향을 바꾸는 것은 무엇이든 지양해야 합니다. 이런 요소가 VR에서 사용자의 모션 멀미를 유발하는 주된 원인입니다.
  - 헤드셋과 렌즈의 피지컬 지오메트리에 일치해야하는 필드 오브 뷰(FOV, Field Of View) 값을 오버라이드하거나 조작하지 마세요. 이 값들은 디바이스 SDK 또는 내부 환경설정을 통해 자동 설정되어야 합니다. 불일치가 있는 경우 사용자가 고개를 돌리면 월드가 뒤틀리는 것처럼 보이고 불편함이나 현기증을 유발합니다.
  - 확대경이나 시야각이 아주 넓은 렌즈로 주변을 보면 1인칭 게임에서 얼마나 혼란스러울지 생각해 보세요.
  - Walking Bob
    - Walking Bob 카메라 이펙트는 VR에서 걷기를 흉내내는 일반적인 방법이지만 이 효과가 동작을 과장하면 멀미를 유발할 수 있습니다.
  - 사용자가 이벤트를 전달 받았을 때 카메라를 흔들면 안됩니다. 예를 들어 사용자 옆에서 폭발이 발생한 경우 VR 게임이 아니라면 카메라 세이크를 사용해서 그 움직임을 전달할 수 있겠지만 VR 게임에서는 이런 인위적인 움직임이 금방 멀미를 유발할 수 있습니다.
  - 최종 사용자가 모션 멀미를 느낄 확률은 최대한 줄이면서 경험을 만드는 팁
    1. VR 경험에서 월드와 레빌을 디자인할 때는 기존 게임에서 사용하던 것보다 흐릿한 라이트와 색을 사용하세요. VR 경험에서 강하고 화사한 라이팅은 시뮬레이션 멀미를 더 빠르게 유발할 수 있습니다. 평상시 쓰던 것보다 차가운 색과 흐릿한 빛으로 이런 현상을 방지하세요. 프로젝트가 실제로 어떻게 보이는 종종 테스트하면서 확인해야합니다.
    2. 너무 밝은 씬 또는 섬광 이펙트는 빛에 민감한 사용자에게 해로울 수 있으므로 항상 염두에 둬야 합니다. 이런 특정 이펙트가 프로젝트에서 꼭 필요하다면 사용자에게 그 효과의 강도를 최소화하는 옵션을 부여하는 것이 좋습니다. 불편함 없이 그 경험을 즐길 수 있도록 말입니다.
    3. 계단 대신 엘리베이터를 사용하세요. 높낮이가 있는 공간을 사용자가 빠르게 움직일 때 특히 올라가거나 내려가는 계단을 놓는다면 실제 사용자의 몸은 계단을 오르내리고 있지 않기 때문에 큰 혼란을 줄 수 있습니다. 계단 끝에 다다르지 않았는데 그렇다고 착각하는 경우를 생각해보세요. 발을 헛디디게 됩니다. VR 게임에서 계단을 사용하면 그와 유사한 위험을 일으키게 됩니다.
    4. 또 한가지 주의할 점은 가속입니다. 사용자의 가속은 서서히 일어나선 안되고 즉시 이뤄져야 합니다. 사람의 움직임이 실제로 그렇기 때문입니다. 발을 내딛기로 마음먹으면 그 즉시 그 속도로 움직입니다. 우리 뇌는 몸이 나아가기 시작할 때 다리를 서서히 가속하지 않습니다. VR에서의 모든 움직임은 최대한 실제 인간의 움직임과 일치해야 합니다.
    5. 속도는 사용자의 입력에 따라 달라질 수 있습니다. 사용자가 속도와 관련해 세밀하게 제어할 수 있어야 합니다.
    6. Depth of Field나 Motion Blur 포스트 프로세싱 이펙트는 사용자가 보는 것에 큰 영향을 미치고 우리의 눈 동작을 방해하기 때문에 시뮬레이션 멀미를 유발하므로 사용하지 마세요. 이런 이펙트는 퍼포먼스 비용도 비쌉니다.

- 플레이하는 최종 사용자의 문제를 최소화하기 위해서는 몇가지 중요한 요소를 염두해야 합니다.
  - 개발자는 VR 환경에 익숙하기 때문에 VR 경험을 더 편하게 느낍니다. 종종 테스트를 실시하여 헤드셋에 익숙하지 않은 사용자가 시뮬레이션 멀미를 겪는 등 생각하지 못한 문제가 발생하는지 확인해야 합니다. 다양한 사용자를 대상으로 애플리케이션을 테스트하여 가능한 다양한 사람이 모션 멀리를 겼지 않도록 하는 것이 권장됩니다. 또한 개발팀 외부 사용자의 비평은 최종 프로젝트의 전반적인 성공에 매우 중요합니다.
