# 프로세스
프로세스는 실행중인 프로그램이다.


### 프로세스의 문맥(context)
- 어느 시점에 프로세스의 수행 상태를 나타내는데 필요한 모든 것
- 문맥의 구성
  - CPU 수행 상태를 나타내는 하드웨어 문맥
      - PC, 각종 register
  - 프로세스의 주소 공간
    - code, data, stack
  - 프로세스 관련 커널 자료 구조
    <p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153741035-c524b983-1fca-406e-b9e9-2f5fa5f1e4ba.PNG" width="200"></p>
    - OS는 프로세스를 관리하기 위해 Kernel stack과 PCB를 갖는다.
      - stack 영역의 프로세스의 kernel stack
        - 프로세스가 시스템콜을 통해 OS에 서비스를 요청하면, 커널 코드가 실행되면서  
        함수 호출시 관련 정보가 stack에 쌓인다. 모든 프로세스가 OS에 서비스 요청을 할 수 있기 때문에, 프로세스 별로 kernel stack을 갖는다.
      - data 영역의 프로세스의 PCB
        - OS가 각 프로세스를 관리하기 위해 프로세스당 유지하는 정보(구조체)
          - OS가 관리상 사용하는 정보
            - Process state, Pid, 스케줄링 정보, 우선순위
          - CPU 수행 관련 하드웨어 값
            - PC, registers
          - 메모리 관련
            - code, data, stack 의 위치 정보
          - 파일 같은 리소스 관련
            - open file descriptors
          - PCB를 연결할 수 있는 Pointer

- 문맥이 왜 필요할까?
  - 컴퓨터에서 한개의 프로세스만 실행되면 필요없겠지만, 실제로는 시분할 시스템에 의해 여러 프로세스가 번갈아 실행되기 때문에, CPU가 하나의 프로세스를 실행하다가
  다른 프로세스로 넘어갈 수 있다. 이때 프로세스의 문맥을 저장하지 않으면, CPU가 다시 해당 프로세스로 돌아왔을 때, 
    어느 시점부터 실행해야 할지 알 수 없게 된다.

### 프로세스의 상태
<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153741011-0739ad13-03e9-4a66-b65d-917fe332a720.PNG" width="400"></p>  

- Running
  - CPU를 받아 명령을 수행중인 상태
- Ready
  - 메모리 등 다른 조건을 모두 만족하고, CPU를 기다리는 상태
- Blocked(wait, sleep)
  - 오래 걸리는 작업으로 인해 CPU를 주어도 당장 명령을 수행할 수 없는 상태
  - Process가 요청한 event(ex.I/O 작업)이 오래 걸리기 때문에, 이 작업이 만족될 때까지 기다리는 상태
    - ex. 디스크 I/O 작업, 공유데이터 접근
- New
  - 프로세스가 생성중인 상태
- Terminated
  - 수행이 끝났지만, 아직 정리할게 남아 사라지지는 않은 상태


<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153740993-50554b5e-6477-4325-8828-d28424b80527.PNG" width="400"></p>  

그림의 큐는 커널의 데이터 영역에 자료구조로 만들어놓고, 프로세스 상태를 바꿔가면서 운영하는 것이다.
그림의 방식은 라운드 로빈  
  
### 문맥 교환(Context Switch)
<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153740932-eb0feb9b-62c3-4ddd-a5b6-afd59785c58e.PNG" width="400"></p>  

- CPU는 한 프로세스를 계속 실행하는 것이 아니라 굉장히 빠르게 다른 프로세스 또한 번갈아 실행하게 된다. 그리고, CPU를 할당 받으면, 처음부터 다시 실행되는 것이 아니라 마지막으로 실행했던 지점부터 재개한다. 
- CPU를 한 프로세스에서 다른 프로세스로 넘겨주는 과정을 문맥교환이라 한다.
- CPU가 다른 프로세스에게 넘어갈 때 OS는 다음을 수행한다.
  - 프로세스의 상태를 그 프로세스의 PCB에 저장
  - CPU를 얻는 프로세스의 상태를 PCB에서 읽어와 하드웨어로 복원한다.


<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153740962-90bd75fb-ee4b-4bc1-9fea-db39488bb5be.PNG" width="400"></p>  

- 시스템콜이나 인터럽트 발생시 사용자 프로세스 -> OS로 CPU가 넘어가는 것이나 이후 동일 사용자 프로세스로 다시 CPU가 돌아오는 것을 문맥교환이라 하지는 않는다.
  - 하지만, 커널의 명령을 실행하는 것이기 때문에, 이전에 사용자 프로세스의 CPU 문맥정보는 PCB에 저장한다. 프로세스 자체가 바뀌는 것에 비해 문맥교환 오버헤드가 적다.
- 다른 프로세스로 CPU가 넘어가는 것만을 문맥교환이라 한다.
  - 문맥교환시 프로세스가 사용하던 캐시 메모리까지 다 지워야 하기 때문에 오버헤드가 크다.
  
### 프로세스를 스케줄링하기 위한 큐
- Job queue
  - 현재 시스템 내에 있는 모든 프로세스의 집합(ready/device queue에 있는 프로세스도 포함)
- Ready Queue
  - 현재 메모리 내에 있으면서 CPU를 잡아서 실행되기를 기다리는 프로세스의 집합
- Device Queues
  - I/O Device의 처리를 기다리는 프로세스의 집합
  
<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153741104-9518e26a-f66a-4407-a413-83599a960c0d.PNG" width="400"></p>
<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153741151-fa10aef0-1968-4bee-9a78-5b6273a7154f.PNG" width="400"></p>

### 스케줄러
- 장기 스케줄러(job 스케줄러)
  - new 상태에서 메모리에 올라가는 것을 admit 하면 ready 상태로 바뀌게 되는데, 장기 스케줄러가 프로세스에게 메모리를 줄지 안줄지 결정
  - 시작 프로세스 중 어떤 것들을 ready queue로 보낼지 결정
  - 프로세스에 memory(및 각종 자원)를 주는 문제
  - degree of multiprogramming(메모리 위 프로세스 수)을 제어
    - 메모리 위에 너무 많은 프로세스가 올라가도 성능이 좋지 않고, 너무 적어도 안좋다.
    - 메모리에 프로세스가 1개라면, 그 프로세스가 CPU를 쓰다가 I/O를 하러가면 CPU가 낭비
    - 메모리에 프로세스가 여러개면, 어떤 프로세스가 CPU를 쓰다가 I/O를 하러가더라도 다른 프로세스에 CPU를 할당하므로 자원 활용도가 낫다.
    - 하지만, 너무 많이 올라가 있으면, CPU가 실행할 때마다 프로세스의 당장 필요한 부분조차 메모리에 없는 경우가 잦아져, 그럴 때마다 I/O를 해야하므로 이또한 성능이 안좋다. 따라서 degree of multiprogramming을 제어하는 것은 중요하다.
  - time sharing system에는 보통 장기 스케줄러가 없다. 프로그램이 실행하면 곧바로 메모리에 올라가고 ready 상태가 된다.
    - 그러면 너무 많은 프로그램이 올라가지 않도록 degree of multiprogramming를 어떻게 조절할까? -> 중기 스케줄러를 이용
- 단기 스케줄러(CPU 스케줄러)
  - 다음번에 어떤 프로세스를 running시킬지 결정
  - 즉, 프로세스에 CPU를 주는 문제
- 중기 스케줄러(Swapper)
  - 메모리에 너무 많은 프로세스가 있으면, 시스템 성능 향상을 위해 여유 공간 마련을 위해 일부 프로세스를 통째로 디스크로 쫓아낸다.
  - 프로세스에게서 memory를 뺏는 문제
  - degree of multiprogramming을 제어
  - 위에서 언급한 running, ready, blocked 상태 외에 중기 스케줄러로 인해 메모리를 빼앗긴 상태가 필요하다. -> suspended (stopped)
    - Suspended
      - 외부적인 이유로 프로세스의 수행이 정지된 상태(메모리에서 내려감) 
        - ex) 시스템이 프로세스를 swap out, 사용자가 ctrl + z
      - 외부에서 재개를 해줘야 active한 상태가 될 수 있음
    - blocked : 자신이 요청한 오래걸리는 일을 기다리는 상태. 끝나면 ready
  
### 좀 더 자세한 프로세스 상태도
<p align="center"><img src = "https://user-images.githubusercontent.com/67847920/153741196-865f27df-fdcd-4448-a7a2-07c806ac7c3e.PNG" width="400"></p>

- Running(user mode) : 프로세스가 CPU를 받아 자신의 명령을 실행하고 있는 상태
- Running(kernel mode) : 프로세스가 자신이 할 수 없는 일을 시스템 콜을 통해 요청하여 OS의 코드를 실행중인 상태
  - 유의할 점 : Kernel이 Running 중인 것은 아니라 사용자 프로세스가 Kernel mode에서 Running인 것이다. 이러한 상태값들은 OS가 사용자 프로세스를 관리하기 위해 사용하는 것이지 OS 본인의 상태는 논하지 않는다.
- active : 실행 중, CPU 기다리거나 I/O 등 작업을 수행중인 상태
- inactive: 프로세스가 정지된 상태  
- Suspended Blocked -> Suspended Ready
  - 메모리에 프로세스가 없기 때문에, CPU 관점에서는 얼어붙은 상태이지만, I/O같은 작업이 진행중이라면 그것을 마쳤을 때 Suspended Ready로 넘어가기도 한다.
  