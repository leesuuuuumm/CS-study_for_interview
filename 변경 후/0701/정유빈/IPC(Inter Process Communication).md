<h1> IPC(Inter Process Communication) </h1>

- 프로세스 간에 통신하여 상호 커뮤니케이션 하는 기법
- 프로세스끼리 직접적으로 "대화"하는 방법은 없다.
- 왜냐하면, 프로세스들이 서로 공간을 쉽게 접근하면 프로세스의 데이터나 코드가 다른 프로세스에 의해 쉽게 바뀔 수 있기 때문이다.

<h3> 통신(커뮤니케이션)이 필요한 이유?</h3>

- 성능을 높이기 위해
- 로직을 수행하는 과정에서 필요한 데이터들을 서로 공유하기 위해서

<h3> IPC 종류 </h3>

<h2> 1. File 사용 </h2>

- 텍스트 파일 txt(혹은 다른 포멧의 파일)을 통해 데이터를 주고 받는 것도 IPC 기법 중 하나
- 그러나 이 방법은 문제가 많다.
- 이 방식은 실시간으로 직접 원하는 프로세스에 데이터를 전달하는게 어렵다.
- 디스크에서 데이터 파일을 읽고 프로세스에 적재(load) 되는 과정에서 Context Switcing(컨텍스트 스위칭), Interrupt(인터럽트) 등 여러 일을 처리해야 하기 때문이다.

<h2> 2. 파이프(Pipe) </h2>


![image](https://github.com/youbeen2798/CS-study_for_interview/assets/62228401/815fca05-dae0-4bbe-af89-1a237eaaf551)

- 단방향 통신, 즉 부모 프로세스 -> 자식 프로세스에게 일방적으로 통신하는 기법
- fork()를 통해 자식 프로세스를 만들고 나서 부모의 데이터를 자식에게 보낸다.

```
char* msg = "Hello Child Process";
int main()
{
	char buf[255];
	int fd[2], pid, nbytes;
	if(pipe(fd) < 0) 						// pipe(fd)로 파이프 생성
		exit(1);
	pid = fork(); 							// 이 함수 실행 다음 코드부터 부모/자식 프로세스로 나눠짐
	if(pid > 0){ 							// 부모 프로세스는 pid에 실제 프로세스 ID가 들어감
		write(fd[1], msg, MSGSIZE); 		// fd[1]에 씀(write 파일 디스크립터)
	} else{ 								// 자식 프로세스는 pid가 0이 들어감
		nbytes = read(fd[0], buf, MSGSIZE); // fd[0]으로 읽음(read 파일 디스크립터)
		printf("%d %s\n", nbytes, buf);
		exit(0);
	}
	return 0;
}
```


<h2> 3. 메시지큐(Message Queue) </h2>

- FIFO 정책으로 데이터가 전송되는 기법이다.

```
// A 프로세스
// key는 1234(예), msgflg는 옵션
msqid = msgget(key, msgflg);
msgsnd(msqid, &buf, buf_length, IPC_NOWAIT);

// B 프로세스
// key는 A에서 지정한 것과 동일하게 해야 해당 큐의 msqid를 얻을 수 있어요.
msqid = msgget(key, msgflg);
msgrcv(msqid, &rbuf, MSGSZ, 1, 0);
```

- msgget(key, msgflg) : key 값의 메시지 큐를 가져온다.
- msgsnd(msqid, &buf, buf_length, IPC_NOWAIT) : msgget의 ID를 통해 얻은 메시지 큐 ID에 버퍼를 push한다.
- msgrcv(msqid, &rbuf, MSGSZ, 1, 0) : msgget의 ID를 통해 얻은 메시지 큐 ID에 버퍼를 push한다.

<b> 파이프 VS 메시지 큐 </b>

- 파이프는 부모 - 자식 간 통신만 가능하지만 메시지 큐는 어떤 프로세스 간이라도 통신이 가능하다.
- 즉, 단방향과 양방향의 차이가 있다.

<h2>  4. 공유 메모리 </h2>

- 그전에 리눅스의 메모리 공간구조에 대해 알아보자.
- 리눅스 Linux는 프로세스 공간이 완전히 분리되어 있다.
- 그리고 어떤 프로세스는 하나 당 4GB 공간을 분리하고 있고, 사용자 공간을 0~3GB 주소까지 저장하고 커널 공간을 3~4GB 주소까지 지정한다.
- 물론 이는 실제 물리 메모리 RAM이 아닌 가상 메모리이다.
- 여기서 중요한 것은, 모든 프로세스가 커널 공간은 공유한다는 것이다.
- 그래서 리눅스는 커널 공간에서 프로세스 간 공유가 가능하다.

![image](https://github.com/youbeen2798/CS-study_for_interview/assets/62228401/6d0850ba-fbbe-4c14-b9ee-f889990ee463)

- 공유 메모리 기법은 커널 공간에 메모리 공간을 만들고, 해당 공간을 변수처럼 쓰는 방식이다.
- 데이터를 버퍼에 넣어 처리하는 메시지 큐와 달리, 해당 메모리 주소를 변수처럼 접근해서 사용한다.
- 공유 메모리 key를 통해 여러 프로세스가 접근을 가능하게 한다.

```
// 1. 공유 메모리 생성 및 메모리 주소 얻기
shmid = shmget((key_t)1234, SIZE, IPC_CREAT|0666/* 접근권한*/));
shmaddr = shmat(shmid, (void*)0, 0);

// 2. 공유 메모리에 쓰기
strcpy((char*)shmaddr, "Linux Programming");

// 3. 공유 메모리 읽기
printf("%s\n", (char*)shmaddr);
```

<h2> 5. 시그널(Signal) </h2>

- 가장 많이 사용하는 기법 중 하나
- 커널 혹은 프로세스에서 다른 프로세스에 이벤트가 발생했는지 알려주는 기법
- 프로세스 관련 코드에 관련 시그널 핸들러를 등록하여, 해당 시그널 처리를 실행하는 방식으로 공유한다.

 ```
  작동 매커니즘
1. 시그널 무시
2. 시그널 Block(블록을 풀면 프로세스에 해당 시그널을 전달해요)
3. 등록된 시그널 핸들러로 특정 동작 수행
4. 등록된 시그널 핸들러가 없으면 커널에서 기본 동작 수행
```

<b> 주요 시그널 </b>

- 운영체제에서 정의된다.(보통 64개)
- CLI에게 kill -l 명령어로 확인 가능하다.(리눅스, Mac OS 기준)

몇 가지 주요 시그널
```
SIGKILL : 프로세스를 죽여요. 슈퍼 관리자가 사용하는 시그널로, 어떤 경우든 프로세스를 죽여요.
SIGALRAM : 알람을 발생시켜요.
SIGSTP : 프로세스를 멈춰요 → Ctrl+Z
SIGCONT : 멈춘 프로세스를 실행시켜요.
SIGSEGV : 프로세스가 다른 메모리 영역을 침범함을 알려요.
SIGUSR1 & 2: 기본 동작이 정의 안된 시그널이에요. 프로그램으로 프로세스를 만들 때, 프로그램에서 특정 시그널은 기본 동작 대신 특별한 동작을 하도록 정의를 할 수 있어요. 위 그림에 맨 마지막 두개를 보시면 아시듯이, Mac OS에선 USR1, USR2로 정의되어 있어요.
```

예제
```
static void signal_handler(int signo){
	printf("Catch SIGINT!\n");
	exit(EXIT_SUCCESS);
}

// 시그널 핸들러 함수 호출
int main(){
	// SIGINT를 받으면 signal_handler 함수 호출
	if(signal(SIGINT, signal_handler) == SIG_ERR){
		printf("Can't catch SIGINT!\n");
		exit(EXIT_FAILURE);
	}
	for(;;){
		pause();
	}
	return 0;
}

// 시그널 핸들러 무시
int main(){
	// SIGINT를 받으면 무시
	if(signal(SIGINT, SIG_IGN) == SIG_ERR){
		printf("Can't catch SIGINT!\n");
		exit(EXIT_FAILURE);
	}
	for(;;){
		pause();
	}
	return 0;
}
```

<b> 시그널과 프로세스 </b>

![image](https://github.com/youbeen2798/CS-study_for_interview/assets/62228401/0f64ff9d-88f3-473b-abb8-340b43c36578)

- PCB에 해당 프로세스가 블록 또는 처리해야 하는 시그널도 관리한다.

```
pending : 처리를 대기 중인 시그널
sigpending : 받은 시그널 유무
blocked : 블록된 시그널 → 비트마스크 형식이에요.
sig : 각 시그널에 대한 동작 처리에 대한 핸들러
```

<h2> 6. 소켓 </h2>

![image](https://github.com/youbeen2798/CS-study_for_interview/assets/62228401/ffcf8720-15fe-44cf-b392-f8f277f87587)

- 마찬가지로 많이 쓰이는 기법이자, 본래 목적이 IPC로 활용하기 위한 것은 아니지만 충분히 활용 가능하다.
- 본래 소켓은 네트워크 통신을 위한 기법이다.
- 기본적으로 클라이언트와 서버 등 2개의 다른 컴퓨터 간의 네트워크 기반 통신을 위한 기술이다.
- 네트워크 디바이스를 사용할 수 있는 시스템 콜이기도 하다.
- 소켓은 이렇게 클라이언트와 서버 뿐 아니라, 하나의 컴퓨터 안에서 2개의 프로세스 간의 통신 기법으로 활용되는 경우도 더러 있다.

![image](https://github.com/youbeen2798/CS-study_for_interview/assets/62228401/291eb98a-56af-4975-86ea-80dcd46153e2)
- 소켓을 사용하면 로컬 컴퓨터 간의 통신 시 이렇게 계층을 타고 내려가면서 송신을 하고, 아래 계층으로부터 위로 올라가서 대상 프로세스가 수신을 하는 방식을 취한다.
