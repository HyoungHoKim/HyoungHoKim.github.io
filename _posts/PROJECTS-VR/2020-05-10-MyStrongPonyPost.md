---
layout: post
title:  "My Strong Horse"
date:   2020-05-10 22:15:31 +0900
categories: [PROJECTS, PROJECTS/VR]
---

My-Strong-Pony는 중세 기사를 컨셉으로한 VR 콘텐츠 입니다.
VR과 Sym4D를 이용해 좀 더 현장감 있는 말타기와 전투에 초점을 두었고,
추가적으로 Photon2를 이용해 다른 플레이어와 1:1 멀티 PVP 전투를 할 수 있도록 했습니다.
### 트레일러 영상
[![Video Label](http://img.youtube.com/vi/kox2td_puWE/0.jpg)](https://youtu.be/kox2td_puWE?t=0s)

### 개요
- [Github Link](https://github.com/HyoungHoKim/MyStringPony_OnlyScript) : 프로젝트에 여러 유료 에셋를 사용했기 때문에, 저작권 문제로 직접 짠 코드만을 공개
- [스팀 출시 링크](https://store.steampowered.com/app/1295670/My_Strong_Horse)
- 메디치 어트랙션 기반 VR 개발자 양성 과정 어트랙션 미니 프로젝트
- 개발 기간 : 2020.03.30 ~ 2020.04.16
- 장비 : VIVE PRO, Sym4D
- 팀명 : Horslic
- 팀원 : 박철우(팀장, AI, 맵 디자인, UI), 연인석(플레이어 이동 및 공격, Sym4D), 김형호(플레이어 IK 연동, 네트워크)
- 자세한 사항은 아래 발표 문서 참조

### 조작법
컨트롤러 위치에 따라 전진하고 방향을 전환하는 방법을 사용했습니다.
* 전진 : 양손의 컨트롤러를 무릎 높이로 일정 속도 이상으로 내리면 앞으로 이동
* 회전 : 원하는 방향의 컨트롤러를 HMD 근방으로 올리면 회전
* 정지 : 양손의 컨트롤러를 HMD쪽으로 모두 올리면 정지
* 공격 : 오른쪽 컨트롤러의 트리거를 당기면 창이 생성, 적을 찌르면 공격이 된다.

### 게임 구성
##### 싱글모드
* 블루팀과 레드팀으로 나눠 적 진영의 대장 기수(5명)를 잡으면 승리
* 플레이어 아래 부분에 UI를 통해 적팀과 플레이어 팀의 현황을 알 수 있음
* 플레이어가 공격 받아 죽으면 다시 리스폰, 승리 조건 달성까지 반복

[![Video Label](http://img.youtube.com/vi/TqClSBWs4i8/0.jpg)](https://youtu.be/TqClSBWs4i8?t=0s)

##### 멀티모드
* 양 플레이어 중 먼저 죽이는 사람이 승리
* 한 쪽 승리 시 두 플레이어 모두 로비로 이동, 재대결 및 싱글 모드로 플레이 가능

[![Video Label](http://img.youtube.com/vi/Lt9TsjLDIJY/0.jpg)](https://youtu.be/Lt9TsjLDIJY?t=0s)

### 핵심 로직
- 제가 개발한 로직만 정리했습니다.

#### 에너미 이동 로직

```
IEnumerator CheckAll()
   {
       while (!isDead)
       {
           yield return new WaitForSeconds(0.2f);

           float closestDistSqr = Mathf.Infinity;
           Transform closestAvoid = null;

           //태그 가지고있는애들 전부 검색
           foreach (string tag in avoidTags)
           {
               //자기자신 제외
               avoidObjs.AddRange(GameObject.FindGameObjectsWithTag(tag));
               if (avoidObjs.IndexOf(this.gameObject) > -1)
               {
                   avoidObjs.RemoveAt(avoidObjs.IndexOf(this.gameObject));
               }

               //집단 안에 제일 가까운애 검색
               foreach (GameObject avoidObj in avoidObjs)
               {

                   Vector3 objectPos = avoidObj.transform.position;
                   avoidTargetDist = (objectPos - transform.position).sqrMagnitude;

                   if (avoidTargetDist < closestDistSqr)
                   {
                       closestDistSqr = avoidTargetDist;
                       closestAvoid = avoidObj.transform;
                   }
               }
               targetAvoid = closestAvoid;
           }
           Debug.Log(targetAvoid.gameObject.name);
           if (targetAvoid != null)
           {
               Debug.DrawLine(transform.position, targetAvoid.position, Color.yellow);
           }
       }

   }

   //스테이트 지정
   IEnumerator CheckState()
   {
       while (!isDead)
       {
           yield return Y.Variables.waitForSeconds_200ms;

           if (Physics.Raycast(rayF, out targetHit, avoidMinRange, avoidMask))//회피 우선
           {
               currentState = CurrentState.avoid;
               yield return new WaitForSeconds(2.0f);

           }        
           else if (tag == "RED" || tag == "BLUE")//대장을 제외한 나머지는 기본상태 follow
           {
               currentState = CurrentState.follow;
           }
           else
               currentState = CurrentState.trace;//대장은 기본 trace
       }

   }

   //스테이트별 코루틴 실행
   IEnumerator CheckForAction()
   {
       while (!isDead)
       {
           //레이캐스트 실행
           float trX = transform.position.x;
           float trY = transform.position.y;
           float trZ = transform.position.z;

           Vector3 trRay = new Vector3(trX, trY + 1.0f, trZ);

           rayF = new Ray(trRay, transform.forward);
           Debug.DrawRay(rayF.origin, rayF.direction * avoidMinRange, Color.white);

           tr = GetComponent<Transform>();

           //속도 보간 계산 하면서 이동
           moveSpeedCal = Mathf.Lerp(moveSpeedMin, moveSpeedMax, Time.time);
           tr.Translate(Vector3.forward.normalized * moveSpeedCal * Time.deltaTime);

           switch (currentState)
           {
               case CurrentState.idle:
                   AnimationIdle();
                   break;
               case CurrentState.follow:
                   Following();
                   break;
               case CurrentState.avoid:
                   Avoiding();
                   break;
               case CurrentState.trace:
                   Tracing();
                   break;
           }
           yield return null;
       }
   }

   //대장 따라가기
  void Following()
  {
      AlphaSearch();

      AnimationF();


      //타겟 위치로 방향 돌림
      Vector3 dir = targetAlpha.position - tr.position;
      Quaternion ro = Quaternion.LookRotation(dir);
      tr.rotation = Quaternion.Lerp(tr.rotation, ro, Time.deltaTime * 1.0f);
  }


  //적 추적
  void Tracing()
  {
      EnemySearch();

      AnimationF();

      //타겟 위치로 방향 돌림
      agent.destination = targetEnemy.position;
  }

  //적 가까이에서 통과, 크게 회전
  void Avoiding()
  {
      //targetAvoid = targetHit.transform;
      //Vector3 avoidDir = tr.InverseTransformPoint(targetAvoid.position);

      AnimationL();

      //회피는 왼쪽으로만
      tr.Rotate(Vector3.up * (moveSpeedCal * -10.0f) * Time.deltaTime);
  }

  void AlphaSearch()
  {
      float closestDistSqr = Mathf.Infinity;
      Transform closestAlpha = null;

      //집단 안에 제일 가까운애 검색
      if (alphaObjs != null)
      {
          foreach (GameObject alPhaObj in alphaObjs)
          {
              Vector3 objectPos = alPhaObj.transform.position;
              alphaTargetDist = (objectPos - transform.position).sqrMagnitude;

              if (alphaTargetDist < closestDistSqr)
              {
                  closestDistSqr = alphaTargetDist;
                  closestAlpha = alPhaObj.transform;
              }
          }
          targetAlpha = closestAlpha;
      }
  }

  void EnemySearch()
  {
      float closestDistSqr = Mathf.Infinity;
      Transform closestEnemy = null;

      //태그 가지고있는애들 전부 검색
      foreach (string tag in enemyTags)
      {
          enemyObjs = GameObject.FindGameObjectsWithTag(tag);

          //집단 안에 제일 가까운애 검색
          foreach (GameObject enemyObj in enemyObjs)
          {
              Vector3 objectPos = enemyObj.transform.position;
              enemyTargetDist = (objectPos - transform.position).sqrMagnitude;

              if (enemyTargetDist < closestDistSqr)
              {
                  closestDistSqr = enemyTargetDist;
                  closestEnemy = enemyObj.transform;
              }
          }
          targetEnemy = closestEnemy.transform;
      }
  }

 ```


### 스크린 샷

<img src="https://user-images.githubusercontent.com/49055264/83052635-8f665200-a08a-11ea-88d3-2d87a9d64120.PNG" style="float: left;  margin-right: 20px; margin-bottom: 10px" width="450px" height="300px"><br/>

<img src="https://user-images.githubusercontent.com/49055264/83052648-942b0600-a08a-11ea-8c03-2e30a07a5da9.PNG" style="float: left;  margin-right: 20px; margin-bottom: 10px" width="450px" height="300px"><br/>

<img src="https://user-images.githubusercontent.com/49055264/83052661-97be8d00-a08a-11ea-928f-e67f03b993f9.PNG" style="float: left;  margin-right: 20px; margin-bottom: 10px" width="450px" height="300px"><br/>

<img src="https://user-images.githubusercontent.com/49055264/83052667-9ab97d80-a08a-11ea-9e87-3a557c9779e6.PNG" style="float: left;  margin-right: 20px; margin-bottom: 10px" width="450px" height="300px"><br/>

<img src="https://user-images.githubusercontent.com/49055264/83052680-9e4d0480-a08a-11ea-92af-21d53f6e39f4.PNG" style="margin-right: 20px; margin-bottom: 10px" width="450px" height="300px"><br/>

### 세부문서
* [참조 링크](https://drive.google.com/file/d/1ptYsvg2h5BnRkOHgK3rtUCUDyxq-LADO/view?usp=sharing)
