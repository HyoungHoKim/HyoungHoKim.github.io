---
layout: post
title:  "42_minishell"
date:   2021-05-08 10:45:31 +0900
categories: [STUDY, STUDY/42_SEOUL]
---

나만 볼려고 적는 minishell 정리!

### 환경변수 
이번 과제에서는 환경변수도 처리할 수 있어야한다. 기존의 환경변수를 처리할 수 있어야하고, 새로운 환경변수도 추가할 수 있어야한다. 그러려면 첫번째 환경변수를 받고 초기화할 수 있어야한다. 

1. 환경변수 받기 

환경변수를 받는 법은 쉽다. main문애 매개변수를 하나 추가하면 된다. 원래 써왔던 2개의 매개변수 이외에도, char **envp를 붙이면 문자열 형태로 환경 변수를 받을 수 있다. 받은 환경변수는 아래와 같이 envp에 입력된다.
```
USER=hyoukim
__CFBundleIdentifier=com.microsoft.VSCode
COMMAND_MODE=unix2003
LOGNAME=hyoukim
PATH=/usr/local/opt/llvm/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/opt/llvm/bin
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.xt0y8YA8Qj/Listeners
SHELL=/bin/zsh
HOME=/Users/hyoukim
__CF_USER_TEXT_ENCODING=0x1F5:0x3:0x33
TMPDIR=/var/folders/63/33s2c4t546n_6h4ckyzfrp3h0000gn/T/
XPC_SERVICE_NAME=0
XPC_FLAGS=0x0
ORIGINAL_XDG_CURRENT_DESKTOP=undefined
SHLVL=1
...
```

### signal
`sig_t signal(int sig, sig_t func)`
sig는 시그널 번호, func는 해당 시그널을 처리할 핸들러.
    - signal이란 Software interrupt로, process에 무엇인가 발생했음을 알리는 간단한 메시지를 비동기적으로 보내는 것.
    - 시그널을 받았을 때, 시그널은 고유한 의미를 내포하고 있다. 이러한 시그널을 받은 실행 객체인 프로세스는 그에 맞는 행동을 해야한다.
        1. 그 시그널을 처리할 등록된 함수(handler)를 호출한다.
        2. 시그널을 무시한다. 
        3. 시그널을 무시하지 않지만, 그렇다고 해서 특별히 함수를 호출하지도 않는다. 

우리가 가끔 터미널에서 무한 루프를 돌 때 Ctrl+C를 누르는 것도 키보드 입력으로 발생시킬 수 있는 시그널의 일종이다. 

일반적으로 프로세스의 경우 SIGINT(Ctrl+C) 시그널을 통하여 수행중인 프로세스(터미널)을 종료시킬 수 있지만 minishell에서는 우리가 만든 minishell만 종료되고 터미널은 여전히 살아있도록 해야하기 때문에, 시그널을 받은 프로세스가 취할 행동을 바꿔주는게 handler 함수이다.
    - 하지만 fork()을 통하여 일반 명령을 수행하는 자식 프로세스의 경우엔 작업 도중 수행을 중단시킬 수 있어야하므로 자식 프로세스의 경우에 한해서만 `SIGINT`를 DEFAULT로 설정한다.
    - 자식 프로세스가 백그라운드로 수행중일 때는 쉘의 뒤편에서 암묵적으로 수행하는 프로세스이므로 `SIGINT`시그널을 무시하도록 설정한다.

미니쉘에서 다루는 signal
1. Ctrl+C(2, SIGINT)
    - 키보드 입력으로 오는 인터럽트 시그널
    - 실행을 중지하고 프로세스를 종료
    - 프롬프트를 다시 띄움
2. Ctrl+\\(3, SIGQUIT)
    - 키보드로부터 오는 인터럽트 시그널
    - 실행을 중지하고 프로세스를 종료시킨 뒤 코어덤프
        - 코어덤프는 프로그램이 비정상적으로 종료되었을때 작정 중이던 메모리 상태를 기록한 파일이다. 
3. Ctrl+D(15, SIGTERM)
    - Terminate의 약자로 프로세스를 정상 종료시킴
    - standard input에 EOF를 등록한다. 쉘에 더 이상 명령을 입력하지 않겠다는 뜻
    - 즉, 프로세스 할 일을 다 마치고 터미널을 종료시킴
    - kill 명령의 기본 시그널

### 프로세스 

#### 프로세스 생성
- 부모 프로세스가 자식 프로세스를 생성
- 프로세스 트리(계층구조) 생성
- 프로세스는 자원을 필요로 함
    - 운영체제로부터 받는다.
    - 부모와 공유한다.
- 자원의 공유
    - 부모와 자식이 모든 자원을 고유하는 모델
    - 일부를 공유하는 모델
    - 전혀 공유하지 않는 모델
- 수행 
    - 부모와 자식이 공존하여 수행되는 모델
    - 자식이 종료될 때까지 부모가 기다리는 모델
- 주소공간
    - 자식은 부모의 공간을 복사함
    - 자식은 그 공간에 새로운 프로그램은 올림
- 유닉스의 예
    - fork() 시스템 콜이 새로운 프로세스를 생성
        - 부모를 그대로 복사 
        - 주소 공간 할당
    - fork 다음에 이어지는 exec() 시스템 콜을 통해 새로운 프로그램을 메모리에 올림 

#### 프로세스 종료
- 프로세스가 마지막 명령을 수행한 후 운영체제에게 이를 알려줌 (exit)
    - 자식이 부모에게 output data를 보냄(via wait)
    - 프로세스의 각종 자원들이 운영체제에 반납됨
- 부모 프로세스가 자식의 수행을 종료시킴 (abort)
    - 자식이 할당 자원의 한계치를 넘어섬
    - 자식에게 할당된 테스크가 더 이상 필요하지 않음
    - 부모가 종료(exit)하는 경우
        - 운영체제는 부모프로세스가 종료하는 경우 자식이 더 이상 수행되도록 두지 않는다. 
        - 단계적인 종료

#### fork() 시스템 콜 : 운영체제야!, 새로운 프로세스를 만들어줘라~
```
int main()
{
    int pid;
    pid = fork(); // fork()가 실행된 다음 부터에서 부모 자식 분기(fork()이후 부터)가 나뉨, main함수의 시작부분에서부터 나뉘지 않음
    if (pid == 0) // this is child
        printf("\n Hello, I am child\n");
    else if (pid > 0) // this is parent
        printf("\n Hello, I am parent\n");
}
```
자식과 부모의 구별은 fork()함수의 리턴 값(pid)로 구분, 자식은 0, 부모는 양수값을 가짐

exec() 시스템 콜 : execlp의 매개변수의 새로운 프로그램으로 완전히 새롭게 실행됨, 한번 exec()으로 새로운 프로그램을 실행하면 다시 돌아갈 수 없음, 자식에게 새로운 프로그램을 덮어씌우기 위한 함수
```
int main()
{
    int pid;
    pid = fork();
    if (pid == 0) // this is child
    {
        printf("\n Hello, I am child\n");
        execlp("/bin/data", "/bin/datae", (char*) 0);
    }
    else if (pid > 0) // this is parent
        printf("\n Hello, I am parent\n");
}
```
- 관련 예제
```
int main()
{
    int main()
    {
        printf("1");
        execlp("echo", "echo", "3", (char *)0);
        printf("2");
    }
}
```
위의 프로그램을 실행하면 1, 3이 출력되고 2는 출력되지 않음. exec()콜 이후에는 완전히 echo가 실행되고 뒤의 내용은 잊혀지기 때문에 echo가 실행되고 종료되는 형태 

#### Wait() 시스템 콜

프로세스 A가 wait()시스템 콜을 호출하면 커널은 child가 종료될 때까지 프로세스 A를 sleep시칸다. (block 상태), child process가 종료되면 커널은 프로세스 A를 깨운다.(ready 상태)

fork 함수로 자식 프로세스를 생성하면 부모 프로세스와 자식 프로세스는 순서와 관계없이 실행되고, 먼저 실행을 마친 프로세스는 종료한다. 이 때 좀비 프로세스같은 불안정 상태의 프로세스가 발생하는데 이를 방지하기 위해 동기화 함수를 수행해서 부모 프로세스와 자식 프로세스를 동기화 시켜야한다. 

이 때 사용하는 것이 wait()함수!
- pid_t wait(int *stat_loc); : 임의 자식 프로세스의 상태값 구하기
- pid_t waitpid(pid_t pid, int *stat_loc, int options) : 특정 프로세스의 상태값 구하기
	- -1 : 임의의 자식 프로세스를 기다린다. 이것은 wait(2)와 동일하다.
	- 0 : 프로세스 그룹 ID가 호출 프로세스의 ID와 같은 자식 프로세스를 기다린다. 
	- WNOHANG
		- 이것은 어떤 자식도 종료되지 않았더라도, 즉시 리턴하라는 뜻이다. 
		- 멈추거나 상태가 보고되지않은 자식들을 위해 리턴, status는 프로세스의 상태를 가져오기 위해서 사용된다. status가 NULL이 아닐 경우 status가 가르키는 위치에 프로세스의 상태정보를 저장한다.
	- 반환값 : 종료된 자식 프로세스는 ID에러 일때 -1을 반환하고, 만일 WNOHANG이 사용되고 어떤 자식도 이용할 수 없다면 0을 반환한다. 
		- 자식 프로세스가 정상 종료 : 프로세스 ID를 반환
			- WIFEXITED(statloc) 매크로가 true를 반환
			- 하위 8비트를 참조하여 자식 프로세스가 exit, _exit, _EXIT에 넘겨준 인자값을 얻을 수 있음, WEXISTSTATUS(statloc)
		- 자식 프로세스가 비정상적 종료 : 프로세스 ID를 반환 
			- WIFSIGNALED(statloc) 매크로가 true를 반환
			- 비정상 종료 이유를 WTERMSIG(statloc) 매크로를 사용하여 구할 수 있음
		- wait함수 오류 : -1 
			- ECHILD : 호출자의 자식 프로세스가 없는 경우
			- EINTR : 시스템 콜이 인터럽트 되었을 때 

exit() 시스템 콜
- 프로세스의 종료
    - 자발적 종료
        - 마지막 statement 수행 후 exit() 시스템 콜을 통해
        - 프로그램에 명시적으로 적어주지 않아도 main함수가 리턴되는 위치에 컴파일러를 넣어줌
    - 비자발적 종료
        - 부모 프로세스가 자식 프로세스를 강제 종료시킴
            - 자식 프로세스가 한계치를 넘어서는 자원 요청
            - 자식에게 할당된 태스크가 더 이상 핋요하지 않음
        - 키보드로 kill, break 등을 친 경우
        - 부모가 종료하는 경우 
            - 부모 프로세스가 종료하기 전에 자식들이 먼저 종료됨 

프로세스와 관련한 시스템 콜
1. fork() : create a child(copy)
2. exec() : overlay new image
3. wait() : sleep until child is done
4. exit() : frees all resources, notify parent

프로세스 간 협력
- 독립적 프로세스 : 프로세스는 각자의 주소 공간을 가지고 수행되므로 원칙적으로 하나의 프로세스는 다른 프로세스의 수행에 영향을 미치지 못함
- 협력 프로세스 : 프로세스 협력 메커니즘을 통해 하나의 프로세스가 다른 프로세스의 수행에 영향을 미칠 수 있음
- 프로세스 간 협력 메커니즘 (IPC : Interprocess Communication)
    - 메세지를 전달하는 방법 : message passing, 커널을 통해 메시지 전달
    - 주소 공간을 공유하는 방법 : shared memory, 서로 다른 프로세스 간에도 일부 주소 공간을 공유하게 하는 shared memory 메커니즘이 있음
    - thread : thread는 사실상 하나의 프로세스이므로 프로세스 간 협력으로 보기는 어렵지만 동일한 process를 구성하는 thread들 간에 주소 공간을 공유하므로 협력이 가능 

Message Passing 
- Message system : 프로세스 사이에 공유 변수(shared variable)를 일체 사용하지 않고 통신하는 시스템
- Direct Communication : 통신하려는 프로세스의 이름을 명시적으로 표시
- Indirect Communication : mailbox(또는 port)를 통해 메시지를 간접 전달