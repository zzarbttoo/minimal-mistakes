# System Structure & Program Execution

  

  

  

* 이 글은 KOCW에 공개되어있는 '반효경 교수님'의 운영체제 강의 및 강의 교재 Operation System Concepts(a.k.a 공룡책🦕)의 내용을 기반으로 작성했다.

  

  

* 이번 챕터에서는 운영체제의 동작을 이해하기 위한 하드웨어의 동작, 프로그램의 동작을 설명을 할 것이다

  

  

  

## 컴퓨터 구조

  

  

  

### 기본적 컴퓨터 구조는 1) CPU +메모리(컴퓨터) &nbsp; + &nbsp; 2) I/O device 를 말한다

  

  

#### 1) cpu 와 cpu의 작업 공간 메모리

  

cpu는 매 clock cycle 마다 메모리에서 instruction(기계어)를 읽어서 실행하게 된다
modebit을 통해 지금 실행되는 것이 운영체제인지 사용자 프로그래밍인지 구분하게 되고,
interrupt line을 통해 interrupt 신호를 감지한다.

  

  

#### 2) io device

  

  

io device 는 controller 가 붙어서 제어를 해준다(마치 작은 CPU)
io device도 각각의 작업 공간이 있는데 그것을 local buffer 이라고 한다(cf CPU의 작업 공간인 memory)
cpu에 비해 io device 는 처리 속도가 훨씬 느리다!

  

  

##### ex) 키보드/마우스(input device), 프린터/모니터(output device) 하드 디스크 (input, output device)

  

  

## interrupt Request(IRQ)

  

  

  

#### interrupt : 주변기기가 CPU에게 어떤 사실을 알리는 일

  

  

#### interrupt line(IRQ) : interrupt의 line

  

  

CPU가 interrupt 를 당하게 된다면, interrupt 당한 시점의 레지스터와 program counter을 save 한 후 CPU의 제어를 interrupt 처리 루틴에 넘긴다 . 즉 CPU는 어떤 일을 하다가도 주변 기기에서 인터럽트 요청이 들어오면 우선순위에 따라 하던 일을 중단하고 그 요청을 받아 다른 일을 하게 된다.


interrupt의 종류에는 1) software interrupt와 2) hardware interrupt(좁은 의미의 interrupt) 두 가지가 있다
인터럽트에 따라 해야하는 일이 다르며, 그에 대한 코드는 운영체제 코드에 정의가 되어있다


### 1. 소프트웨어 interrupt(Trap)

소프트웨어 interrupt(trap)은 사용자 프로그램이 운영체제를 호출하는 것으로 1)System call, 2)Exception 이 있다


### 1) System Call


System Call의 설명을 진행하기에 앞서 Mode bit과 I/O 입출력 과정에 대한 이해가 필요하다.


### Mode bit

Mode bit 은 사용자 프로그램의 잘못된 수행으로 다른 프로그램 및 운영체제에 피해가 가지 않도록 하는 보호 장치이다 
Mode bit을 통해 사용자 모드와 모니터 모드 하드웨어적으로 총 두가지 operation을 제공하게 된다 

- 사용자 모드 (1) : 사용자 프로그램 수행(제한된 instruction만을 수행할 수 있다)
- 모니터 모드 (0) : OS 코드 수행 (메모리 접근/ io 접근 instruction을 모두 할 수 있다)
  

![](https://lh3.googleusercontent.com/EdedsZj5lbyKE7kSByDAmMKXTjBqZbZAHKsAUBvw3A2pVxFMl0GLnbo91xwg8lB4JcO7milEUkrT8t1OJMBAaOaJ02ABbhXnWf61CZZeZDg7G2pD7Eem0nv4MRZeDEtvrOgXiJ8X)

보안상의 목적이 있음
io 접근이 가능한 것은 운영체제 뿐이기 때문에
사용자 프로그램으로 cpu를 넘겨줄 때 mode bit을 1로 바꾸어서 넘겨주도록 한다
그러면 사용자 프로그램이 instruction을 실행할 때, 운영체제, io 접근 을 하려고 하면 mdoebit이 1인 것을 보고 접근이 안되도록 한다
내 프로그램 안에서 함수호출을 하는 것은 메모리 안에서 하는 것인데, io를 호출해야하는 경우 운영체제 함수를 호출해야한다
그 때 modebit이 1이기 때문에 호출을 할 수 없을 것이다
그러면 프로그램이 직접 interupt line을 세팅하는 instruction을 실행한다
아까처럼 타이머나 device controller가 interruption을 거는 것이 아니라 프로그램이 직접 건다
그럼 modebit이 0으로 바뀌고 cpu 제어권이 운영체제로 넘어간다
그럼 시스템 콜을 이후 device controller 에게 요청을 하게 된다
cpu 안에 있는 (프로그램 카운터 레지스터) 가 가르키고 있는 메모리 주소의 일 실행counter instruction 으로 4byte + 해서 다음 프로그램 실행

  

근데 프로그램에서 if 문으로 바이트를 건너뛰어야할 상황 등이 생기면 또 그에 맞는 레지스터 실행

  

근데 레지스터의 일을 하기 전에 intterupt line을 먼저 확인하게 된다

  

그래서 cpu를 누가 쓰고 있었던 interrupt 가 걸린다면 cpu 제어권이 운영체제로 넘어가게된다

  

그럼 interrupt 마다 interrupt에 따라 처리해야할 상황등이 운영체제 안에 함수로 정의되어있다

  

인터럽트 라인별로 인터럽트 vector가 있다 < - 인터럽트 종류별로 몇번 라인에 인터럽트가 들어왔는지 나타내는 엔트리

  

그리고 그 인터럽트가 들어와있을 때 운영체제 메모리 어디에 있는 instruction을 실행해야하는가

  

인터럽트 번호와 인터럽트 주소의 쌍을 가지고 있는 것이 인터럽트 벡터

  

예를 들어 하드 디스트 컨트롤러가 인터럽트를 걸고, 3번라인이 세팅됐다 하면

  

interrupt vector 에 가면 3번에 해당하는 주소가 있다

  

그래서 그 주소가 함수 위치를알려주고 잇는 것이다

  

해당하는 함수에 가면 그 디스크 컨트롤러가 발생시킨 interrupt에 대해 cpu에서 실행해야하는 함수가 정의되어있다

  

즉 실제 해야할 일이 interrupt handler, (interrupt service routine)

  

그리고 인터럽트 종류별로 그 함수로 점프하게끔 해주는 주소를 가지고 있는 것이 interrupt vector

  

cpu는 매번 프로그램 카운터가 가르키고 있는 곳을 실행하고 있는데, interrupt가 걸리면 제어권이 운영체제로 넘어가게 된다 

다른 사용자 프로그램의 메모리를 본다거나, io 디바이스에 접근하거나 하는 것은 0일때
1일때는 사용자 프로그램을 100% 믿을 수 없기 때문에 자기 메모리 주소 영역만 보고 일을 해야한다
그래서 모든 기계어를 실행하지 못하도록 한다
그래서 사용자 프로그램이 io작업을 하기 위해서는 운영체제로 요청을 해야한다
운영체제한테 요청을 하려면 프로그램 카운터 레지스터가 운영체제로 점프를 해야하는데 모드빗이 1일때는 못하기 때문에
그래서 시스템 콜을 하게 된다
운영체제의 함수를 사용자 프로그램이 요청하는 것
바로 점프는 불가능하기 때문에 사용자 프로그램이 운영체제를 호출할때는 의도적으로 interrupt line을 세팅한다
그럼 하던 일을 멈추고 cpu가 운영체제로 넘어가게 된다
그리고 modebit은 cpu를 운영체제가 가지고 있느냐 아니면 사용자 프로그램이 가지고 있느냐
트랩은 소프트웨어가 거는 interrupt
시스템 콜은 어떻게 하는냐
그러면 키보드, 디스크 등의 요청을 어떻게 아느냐
그것을 전달하기 위해 interrupt line이 붙어있다
그래서 만약에 cpu가 프로그램 a를 실행하고 있다 그러면 cpu에서 실행을 하다 scanf 등의 io를 이용한 후 실행 하는 경우가 있다(io device 접근)
하지만 cpu는 iodevice에 직접 접근하지 않고 메모리를 실행하는 instruction만 접근하도록 되어있다
만약에 cpu가 iodevice가 할 일이 생기면 device controller을 이용해 시킨다
그러한 instruction이 존재함
그것을 읽어오는 작업은 오래 걸릴 것이다 디스크는 그 시킨 일을 하면서 읽어다가자신의 로컬 버퍼에 집어넣게 된다
그러면서 빠른 cpu가 놀면 낭비가 될 것이다
일반적으로 cpu는 메모리 접근만 하다가 io 접근이 필요하면 io controller을 시키고 다시 자신의 일을 한다
원래 보통 읽어온 결과를 이용하는 프로그램을 이용한다
그럼 키보드 에다가 결과가 나오면 알려달라고 해놓고, 자기 작업을 계속 한다
이후 넘어오는 값을 더이상 모르겠다라고 cpu가 판단한다면, 다른 프로그램으로 cpu가 넘어간다
cpu는 빠르면서 쉬지않고 일을 한다
그래서 프로그램 여러개가 실행될 때 cpu는 여기저기 왔다갔다 하면서 다 실행을 하기 때문에 사용자 입장에서는 interactive 하게 느껴진다  

![](https://lh5.googleusercontent.com/1OB6wehF6ONgs9JSnS98gUWDtucgsZdpVsxeimnNv9xeY0_rzpZDxzD_5GJ0ocjFtHBXr777id_uwM7bNyl9gLmPhlwnMlKi3-y2k5OeLtMLInPkT9a3kq73lT3WK2IrrnqQ6Pdf)

결국 cpu는 pc register 안에 있는 instruction 주소값에 해당하는 instruction만 실행하는 것
그리고 그 instruction 중에서 io 장치를 접근해야하는 상황이 되면 device driver 을 통해 읽기/쓰기 등의 명령을 하게 한다
실제로 읽고 쓰는 일은 device controller을 시킴
근데 device driver 은 cpu가 수행하는 장치를 실행하기 위한 코드를 담고 있다
그래서 직접 일을 하지 못하고 메뉴얼대로 일을 한다
메뉴얼은 메모리 몇번지에 있는 일을 하라고 얘기
스스로 일하지 않는다
그리고 이런 전체적인 통제는 os가 하고
io 접근은 특권 명령이기 땜누에 운영체제를 통해서만 io 장치에 접근할 수 있다
그럼 무언가를 읽어와야할 때
운영체제에게 그걸 부탁하게 되고 그것을 시스템 콜이라고 한다
사용자 프로그램이 운영체제의 커널을 호출하는 것(시스템 콜)
io를 해야할 경우 이 프로그램이 자동으로 자진해서 io를 해달라고 cpu를 넘기게 된다
사용자 프로그램은 본인이 직접 io 프로그램으로 접근할 수 없고, 무조건 운영체제를 통해서 접근할 수 있도록 막아놨다
(보안 등등)
그래서 키보드에서 무엇을 읽어오거나, 화면을 출력하거나 해야할 경우 운영체제로 스스로 cpu를 넘겨주고 운영체제는 io controller 에게 요청을 하는 것이다
그리고 io 작업은 오래걸리기 때문에 io를 요청한 사용자 프로그램에게 cpu를 넘기는 것이 아니라 다른 프로그램에게 cpu를 넘기게 된다

  


  

![](https://lh4.googleusercontent.com/2uQFygWgXzv_TnOmRkFxq_S2gOkC3zN6ZTpFwTDuMxO0l7xyA424ldaZwa-XaQrJrBmDFhCNrN9yr8xmWKCsPEADClD7Afy3TTkYlwsVplrlJx5wDrIaBa8JTIHXrY4mO5WknMu7)

  

  

### 2) exception(예외)

  

  

프로그램이 cpu에서 instruction을 실행하다 exception을 발생시키거나, 운영체제 메모리 접근하려는 상황이 발생했을 때(modebit 1의 상태) interrupt line이 자동으로 세팅되고 자동적으로 cpu가 운영체제로 넘어간다

  

  

### 2. 하드웨어 interrupt

  

  

하드웨어 interrupt는 하드웨어 장치가 interrupt를 거는 것으로 1) io controller interrupt와 2)timer interrupt가 있다

  

  

### 1) io controller interrupt

  

io controller interrupt 는 다음과 같은 과정을 거쳐 진행된다 (system call 과정은 생략)

  

  

1. io controller 가 요청한 작업을 끝내서 io buffer에 값이 들어오면 io controller가 cpu에 interrupt 를 걸게 된다

  

2. interrupt line이 세팅되면 cpu 제어권이 당장 실행되고 있던 프로그램에서 운영체제로 넘어오게 된다.

  

3. 입력된 값을 io 요청을 했던 프로그램의 메모리 영역에 카피를 해준다

  

4. interrupt를 당한 그 프로그램을 다시 실행해주게 된다

  

  

이후 순차적으로 프로그램들을 실행한 후 i/o를 요청했던 프로그램의 순서가 되면 그 프로그램을 실행하게 된다

  

  

### 2) Timer interrupt

  

  

io를 사용하지 않아 system call 등의 interrupt 가 발생하지 않는 프로그램이 무한 루프 등일 이용해 cpu를 계속해서 독점하는 경우가 발생하면, cpu의 time sharing을 구현할 수 없다

  

  

#### 그래서 컴퓨터 안에는 타이머라는 하드웨어를 두고 있다

  

타이머 작동 순서는 다음과 같다

  

  

1. 운영체제는 cpu를 가지고 있다가 사용자 프로그램에 cpu를 넘겨줄 때 타이머를 같이 달아 전달한다

  

  

2. 타이머는 1 clock tick당 시간이 1씩 감소하며 시간이 다 되면 운영체제에게 제어권이 넘어가도록 interrupt를 발생시킨다

  

  

3. cpu는 instruction을 하나씩 실행하다 instruction 하나가 끝날때마다 interrupt line을 체크한다

  

  

4. timer가 interrupt를 걸어왔으면, cpu는 하던 일을 잠시 멈추고, cpu의 제어권이 사용자 프로그램으로부터 운영체제로 자동으로 넘어가도록 된다

  

  

운영체제는 cpu 제어권을 자유롭게 줄 수는 있지만, 자유롭게 뺐을 수는 없기 때문에 추가적으로 하드웨어 타이머(timer)을 넣은 것이다

  

> (운영체제 -> A 프로그램-> 운영체제 -> B 프로그램....) 이 순서대로 cpu 제어권이 넘어가게 된다

  

  

이렇게 타이머는 특정 프로그램이 cpu를 독점하는 것을 막으며, 현재 시간을 계산할 때도 사용된다

  

##### cf ) 그냥 메모리에서 instruction을 돌다가 프로그램이 수행을 마치고 종료가 되면 cpu를 자동으로 반납하게 된다

  

  

#### 위와 같은 interrupt들의 작동 때문에 운영체제는 cpu를 사용할 일이 없게 된다 (interrupt 를 사용할 때만 운영체제에 cpu가 넘어오게 됨)

  

  

## 입출력

  

### 1. Device Controller

  

Device Controller 은 해당 I/O 장치 유형을 관리하는 일종의 작은 CPU이며, 제어 정보를 위한 1)control register/status register과 2) local buffer(data register)을 가진다

  

  

1) 제어정보를 위한 레지스터(control register/ status register)

  

> cpu가 일을 시킬 때 제어정보를 위한 레지스터를 이용해 지시를 내리게 한다

  

2) local buffer(data register) & device controller

  

>어떤 파일을 저장하고 싶으면 데이터 자체는 local buffer에 저장하고, 명령은 device controller에 시킨다

  

용어

device controller(장치제어기) : 각 장치를 통제하는 작은 CPU(하드웨어)

device driver(장치구동기) : 운영체제 코드 중 각 디바이스 접근을 위한 인터페이스에 맞게 붙이는 소프트웨어 모듈(소프트웨어)

  

  

### 2. DMA

  

cpu는 memory와 local buffer 모두에 접근할 수 있어 io device가 처리한 일이 buffer에 쌓이면 cpu가 내용을 읽어 memory에 복사해 이를 처리한다. 하지만 이렇게 일을 처리하면 cpu가 너무 많은 interrupt를 당해 비효율적이라 할 수 있다

  

  

#### 그래서 메모리에 직접 접근할 수 있는 DMA(direct memory access) controller를 하나 더 둔다

  

  

cpu 뿐만 아니라 DMA도 메모리 영역에 접근할 수 있기 때문에 이 둘이 특정 메모리 영역으로 동시에 접근하면 문제가 생길 수 있다.

  

#### memory controller은 cpu와 DMA 중 어떤 것이 먼저 접근할 지 중재하는 역할을 해준다

  

  

DMA의 작동 순서는 다음과 같다

  

  

1. i/o 작업이 끝나면 local buffer에 내용이 담기게 된다.

  

2. DMA가 이 때 메모리로 내용을 복사한다

  

3. 여러번의 복사 작업이 끝나 기준치(byte 가 아닌 block 단위)까지의 일을 하게 되면 DMA는 cpu에 interrupt를 한번만 걸어 내용이 메모리에 다 올라왔다 보고를 한다

  

  

이를 통해 interrupt의 횟수를 줄여 overhead를 줄이고, cpu가 더 효율적으로 일할 수 있도록 한다 (빠른 입출력 장치를 메모리에 가까운 속도로 처리하기 위해 사용)

  

  

### 3. 동기식 입출력/비동기식 입출력(Synchronuous I/O)

  

  

입출력 방식에는 1) 동기식 입출력, 2)비동기식 입출력 두가지가 있다

  

  

#### 1. 동기식 입출력(synchronous I/O)

  

동기식 입출력은 I/O 요청 후 입출력 작업이 완료된 후에야 제어가 사용자 프로그램에 넘어가는 것을 말한다(미리 시간을 맞춰서 조율 해 놓는 것이 동기식)

  

#### &nbsp; 1) 구현방법 1

  

> I/O 가 끝날 때 까지 해당 프로그램에 cpu를 할당시킨다

  

> -> cpu를 낭비시키며, 매 시점 I/O를 하나만 할 수 있어 I/O를 낭비시킨다

  

  

#### &nbsp; 2) 구현방법 2

  

> I/O 처리가 끝날 때까지 해당 프로그램에서 CPU를 빼았고, I/O 처리를 기다리는 줄에 그 프로그램을 줄세운다. 그 때까지 다른 프로그램에 CPU를 할당한다

  

> -> cpu와 I/O 를 낭비하지 않을 수 있다

  

  

#### 2. 비동기식 입출력

  

I/O가 시작된 후 입출력 작업이 끝나기를 기다리지 않고 제어가 사용자 프로그램으로 즉시 넘어가는 것

  

>asynchronous로 짜여진 시스템 프로그램은 운영체제에 i/o요청만 해놓고 바로 cpu 제어권을 얻어서 다른 작업을 하는 것이다

  

>(io와 관련 없는 작업을 먼저 진행-> io 결과를 안 후에 io 작업을 진행)

  

  

보통 read 는 synchronous, writing 은 asynchronous 하게 하며, 비동기 & 동기 모두 I/O의 완료는 모두 interrupt가 알려준다

  

  

### 4. 서로 다른 입출력 명령어

  

  

#### 1. 일반적 io 방식

  

memory instruction(로드 스토어) 와 io 디바이스의 주소를 따로 사용한다

  

#### 2. Memory Mapped

  

io device에 메모리 주소를 매겨서 memory instruction을 통해 io 디바이스 접근

  

> io 장치에도 메모리 주소의 연장 주소를 붙인 후 메모리 접근 가능하게 함(사실 메모리 접근이 아니라 io 접근인 것이다)

  

  

## 저장장치

  

### Storage 구조

저장장치의 구조는 다음과 같이 1) Primary Storage, 2) Secondary Storage 로 구성되어있다

기본적으로 위로 올라올수록 속도가 빠르고 비싼 매체를 사용하게 된다.

  

![](https://lh5.googleusercontent.com/7SVFhlGIPYcOa2dyUKs12gxNAmMS4u1ttqH5P23CdROEj8j6CQhRSronGM9iNWhP3c01BujkuPxCl8nMQVKBT0WY_ntz_7Tt79Oi86f7COtjOLDMchtAk7l7rWzG4HiBVjQXvTFg)

  
  

#### 1. primary(Executable)

  

Primary Storage는 CPU에 직접 접근하기 때문에 "바이트 단위" 로 접근이 가능하다(executable)

Primary Storage는 휘발성 매체로 구성되어있고(하지만 최근에는 비휘발성 매체를 사용하기도 한다)

Secondary Storage에 비해 용량이 작기 때문에 Primary Storage에 모든 것을 올려놓을 수 없다.

- Cache Memory : CPU의 속도(instruction 당 1clock)와 DRAM의 속도(instruction당 100clock)의 속도를 완충하기 위해 사용한다. 메인 메모리보다 용량이 작기 때문에 당장 필요한 것만 올려서 사용한다

>캐싱 : 당장 필요한 것만 Cache Memory에 올려놓고 사용하는 것으로 빠른 매체로 정보를 읽어들이게 된다. 캐싱은 대부분 재사용을 목적으로 하며, 한번 올려놓으면 두번째 요청할 때 바로 읽어들일 수 있어 속도가 빨라지게 된다

- Registers : 메모리보다 빠르며 정보를 저장할 수 있다

- Main Memory  : DRAM으로 구성되어있다

#### 2. Secondary

Secondary Storage는 setter 단위로 진행이 되는 하드디스크이기 때문에 executable하지 않다.

secondary Storage는 비 휘발성 매체로 구성이 되어있다.

- Magnetic Disk

- Optical Disk

- Magnetic Tape

  

### 프로그램 실행(memory load)

 
File System에 저장되어있는 실행 파일이 메모리에 적재되면 프로세스가 되어 프로그램이 실행되게 된다

![](https://lh6.googleusercontent.com/HsKIahHXhEWn4M2J3Y9FwdlkmMnMrFn9jBp8wKH0Z472gvYhfDFVlkS9XS9eCKxcgY1A0oLj57p1fBfbKubKWqt6bjus2bT-0bW0rrgrPpNuWjcSB_x7mnE8D6goEwXv7HQzNrVT)

  
 이 때 물리적인 메모리에 바로 올라가는 것이 아니라 virtual memory 즉 가상 메모리에 적재를 하게 된다 
 
![](https://lh3.googleusercontent.com/8-ZdahcMPWeG9GZpbOraSt2wKH82dNUdQesUL_RHkTr5DNDue_nhUt2thHscuADQr5Jom1DVZjpnyHrLknpveV6NhsZYoNZa_HYsEHHzH9r-lpWQCAV2PaGY572Hqm7bOs-eSeMn)

###  virtual memory (Address Space)

프로그램을 실행시키게 되면 그 프로그램의 address space (메모리 주소 공간)가 생기게 된다
-  0번지부터 시작하는 독자적인 주소 공간이 생기게 된다
- 이러한 주소 공간은 code, data, stack으로 구성되게 된다
	1) code : cpu에서 실행할 기계어 code
	2) data : 전역변수와 프로그램이 실행하는 자료구조 
	3) stack : code에서 호출한 함수가 저장되며 return 할 때 데이터를 꺼내게 된다 

이러한 Virtual memory는 물리적 메모리에 올라가게 된다 

###  Address translation
- virtual memory 주소는  0번지부터 시작하며 physical memory도 0번지부터 시작하기 때문에  물리적 메모리에 맞게 virtual memory 의 주소를 변환해줘야 한다
- 이때 주소 변환을 해주는 하드웨어 장치를 사용하게 된다 (논리적 메모리 주소-> 물리적 메모리 주소)

### Physical memory
- 커널은 물리적 메모리에 상주하며, stack/data/code 형태로 이루어져있다 
- Address Space를 물리적 메모리 공간에 올릴 때 모두 올려놓는 것이 아니라 필요한 부분만 올려놓게 된다(쪼개서 올려놓는 기법을 virtual memory라고도 한다)  
- virtual memroy 기법을 통해 메모리 낭비를 줄이게 된다 
- 그 프로그램이 종료되게 되면 메모리에서 쫓아내게 된다
- 쫓아내면서 경우에 따라서는 프로그램이 종료되기 전까지 가지고 있어야 하는 부분을 Swap Area에 저장하게 된다 

### Swap Area

- 물리적 메모리의 공간이 한정적이기 때문에 주소 공간에서 당장 필요한 것은 물리적인 공간에 올려놓게 되고 그렇지 않은 경우에는 Disk Swap Area(메모리 연장 공간)에 내려놓게 된다
-  전원이 꺼지면 프로세스가 종료되고, 메모리 영역에 잇는 데이터도 사라지기 때문에 swap area에 있는 데이터도 사라지게 된다(휘발성) <-> File System(비휘발성)
 
 
 ### 커널 주소 공간의 내용 
 앞서 보았듯 커널 주소 공간 역시 code, data, stack의 구조로 되어있다.
 
![](https://lh4.googleusercontent.com/q_eFpwn47h8CkikrufWQo51e06l1TUQLzdpCOe5WyOhAjTpTTOvHdqC1VWCUNlmkelqPXCmOfgPrwS6XuETahbgbE5yGJ2O-smsjLUAmK38DQpX-HUS4CVMDl-_klTdLysShE7DE)

  

운영체제 코드에는 어떤 커널이 있는가

  

운영체제는 자원을 효율적으로 관리하는 역할을 하기 때문에 관련 코드가 있고

  

사용자에게 편리를 줄 수 있는 서비스를 위한 코드가 있다

  

운영체제는 interrupt 를 얻게 되면 cpu를 얻게 되는데, 각각의 interrupt 마다 cpu를 얻게된다

  

각각의 interrupt 마다 무슨일을 처리해야하는지 커널 코드로 함수 형태로 되어있다

  

  

---------

  

커널의 데이터 부분에는 운영체제의 자료구조가 있다

  

운영체제는 하드웨어를 직접 관리한다

  

그럼 이를 관리하는 자료구조를 각각 가지고 잇다

  

또한 운영체제는 프로세스를 관리한다

  

현재 실행중인 프로세스를 관리하기 위한 자료구조를 가지고 있는데 , 이것을 PCB라고 한다(process control block )

  

운영체제는 함수 구조로 짜여있기 때문에 stack 영역을 사용해야한

  

이를 커널 stack이라고 한다

  

운영체제 코드는 여러 프로그램들이 요청에 따라 불러서 사용할 수 있다

  

(시스템콜)

  

사용자 프로그램들이 운영체제 커널코드를 불러서 사용하기 때문에,

  

함수 호출을 할 때 어떤 프로그램이 커널 스택을 호출했냐에 따라서 사용자 프로그램마다 커널 스택을 따로 두고 있다

  

![](https://lh5.googleusercontent.com/qKNPVSBSPMdkO8Eq7uzI7fR4IohP3OECxmOzGOB0hYy3qWV-RXclwsC5XcqAGLT0AbBZr9vv61V1amZe4Sw0sz0YXv9qLln9c1BqrwhT8ah2IC9ZQYCexsf-BlFxFc6WdS7EMoPx)

  

어떤 프로그램이라도 다 함수 형태로 만들어져잇다

  

언어가 컴파일되어 기계어가 만들어지더라고 기계어 구조에서는 함수에 해당하는 부분이 어디부터 어디인지 써있음

  

프로그램을 작성하게 되면 메인 함수 가 있을 것이고, 그것이 다른 함수를 호출할 수 있다

  

그 함수가 사용자 정의 함수일 수도 있고 라이브러리 함수일 수도 있다

  

무슨 함수이든 프로그램 안에 포함되어이있다

  

반면 커널함수는 프로그램에서 호출해서 가져다 쓸 수 있지만, 내 프로그램 안에는 정의가 없다

  

커널함수로 메모리 주소 점프를 할 수는 없다

  

그래서 제어권을 넘겨준다

  

![](https://lh3.googleusercontent.com/cZ4UEox1OjQhDG2HtSgJzj6BSN66SdMaJjq66CwjG-JjGd2U_krktayg3CjtDNgOrj6E0Pvsr0Jmg10Citw4GM0fLpWtg20ooV_orfadwRdW-qbH5XzN9S6X6tZLePcYnmSYduLl)

  

  

A 라는 프로그램이 시작돼서 종료될 때 까지의 것

  

그래서 처음에는 usermode (프로그램이 cpu를 잡은 상태)

  

usermode 안에서 사용자 정의 함수 호출

  

그래서 usermode 안에서 실행, 그러다 시스템 콜을 하게 되면 주소 공간에서 하는 것이 아니라 kernel 함수 안에서 커널 주소 공간 안에서 실행이 되게 된다

  

(커널 모드의 cpu 동작)

  

그러다 시스템 콜이 끝나게 되면 a라는 프로그램에 cpu 제어권이 넘어오게 되고, 본인의 주소 공간 안에서 실행을 하게 된다

  

  

###### ※ 출처 https://www.sony.co.kr/electronics/support/articles/S500082584