---
layout: post
title:  "42_minishell"
date:   2021-05-10 15:20:21 +0900
categories: [STUDY, STUDY/42_SEOUL]
---

나만 볼려고 적는 minishell 정리!

### Parsing
처음에는 그냥 순서대로 명령어, 문자열 부분을 잘라서 저장하려고 했는데, 그게 아니라 한 라인 즉 ';', '|'를 기준으로 한 라인을 자르고 그 라인 내에서 다시 나누는 것이 토큰화라고 한다. 즉 리스트를 두개나 3개를 이용해서 나눠줘야한다. 

### 일단 중지
교육 과정이 바뀌면서 push_swap이라는 과정이 밑으로 내려갔다. 즉, push_swap을 안하면 윗 서클 과정을 할 수 없다는 애기... minishell 등록이 안된다. minishell은 일단 스톱하고 push_swap부터 해야겠다. 
