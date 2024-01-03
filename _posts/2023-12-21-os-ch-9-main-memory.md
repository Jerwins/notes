---
layout: post
title: 운영체제 9장 - 메인 메모리 (1) - 기본
categories: [스터디-운영체제]
tags: [
    운영체제,
    OS,
    Operating System,
    memory,
    main memory,
    메인 메모리,
    논리 주소,
    물리 주소,
    동적 로딩,
    동적 연결,
    DLL,
    베이스,
    base,
    리밋,
    limit,
    레지스터
    register,
    재배치,
    relocation,
  ]
date: 2023-12-21 23:59:59 +0900
---

# 메인 메모리

## 이 장의 목표

- 논리 주소와 물리 주소의 차이점과 주소를 변환할 때 MMU(메모리 관리 장치)의 역할을 설명한다.
- 메모리를 연속적으로 할당하기 위해 최초, 최적 및 최악 접합 전략을 사용합니다.
- 내부 및 외부 단편화의 차이점을 설명한다.
- TLB (Translation look-aside buffer)가 포함된 페이징 시스템에서 논리 주소를 물리 주소로 변환한다.
- 계층적 페이징, 해시 페이징 및 역 페이지 테이블을 설명한다.
- IA-32, x86-64 및 ARMv8 아키텍처의 주소 변환에 관해 설명한다.

## 9.1 배경

메인 메모리는 현대 컴퓨터 시스템의 운영에 중심적인 역할을 함.

메모리는 각각 주소가 할당된 일련의 바이트들로 구성됨.

CPU는 PC (program counter)가 지시하는 대로 메모리로부터 다음 수행할 명령어를 가져오는데 그 명령어는 필요한 경우 추가적인 데이터를 더 가져올 수 있으며 반대로 데이터를 메모리로 내보낼수도 있음.

일반적으로 명령어의 실행은 먼저 메모리로부터 한 명령어를 가져오는 데서부터 시작된다.

그런 다음 명령어를 해독하고, 메모리에서 피연산자(operand)를 가져와 피연산자에 대해 명령어를 실행한 후에 계산 결과를 메모리에 다시 저장한다.

메모리는 주소에 지시한 대로 읽기/쓰기만 할 뿐 이 주소들이 어떻게 생성되었는지 혹시 그 주소가 가리키는 내용이 무엇인지를 모른다.

따라서 주소가 프로그램에 의해서 어떻게 생성되었는지에 대한 세부 사항은 이 시점에서의 고려 대상이 아니다. 여기서는 프로그램에 의해서 생성되는 일련의 주소만 언급하기로 한다.

### 9.1.1 기본 하드웨어

메인 메모리와 각 처리 코어에 내장된 레지스터들은 CPU가 직접 접근할 수 있는 유일한 범용 저장장치이다.

기계 명령어들은 메모리 주소만을 인수로 취하고, 디스크의 주소를 인수로 취하지 않는다.

따라서 실행되는 명령어와 데이터들은 CPU가 직접적으로 접근할 수 있는 메인 메모리와 레지스터에 있어야 한다. 만약 데이터가 메모리에 없다면 CPU가 그것들을 처리하기 전에 메모리로 이동시켜야 한다.

레지스터들은 CPU 클록의 1사이클 내에 접근이 가능하다. 레지스터에 있는 명령의 해독과 간단한 연산을 클록 틱당 하나 또는 그 이상의 속도로 처리한다.
하지만 메모리 버스를 통해 전송되는 메인 메모리의 경우는 상황이 다르다. 메인 메모리의 접근을 완료하기 위해서는 많은 CPU 클록 틱 사이클이 소요되며, 이 경우 CPU가 필요한 데이터가 없어서 명령어를 수행하지 못하고 지연되는 현상(stall)이 발생하게 된다.
이러한 문제를 해결하기 위해 빠른 속도의 메모리 - 캐시를 추가한다.

물리 메모리의 상대적인 접근 속도의 차이를 고려하면서 올바른 동작을 보장해야 한다.
이를 위해 각 영역을 나누어 사용한다.

![logical-address-spaces](/assets/images/2023-12-21-os-ch-9-main-memory/logical-address-spaces.png)

base register와 limit register를 통해 논리 주소 공간(logical address space)가 정해진다.

> 기준 : base, 상한 : limit  
> 영어로 기억하는게 더 좋을 것 같다.

이러한 보호 기법은 운영체제가 개입하게 되면 성능이 떨어지기 때문에 반드시 하드웨어가 지원해야 한다.

개별적인 메모리 공간을 분리하기 위해서 특정 프로세스만 접근할 수 있는 합법적인(legal) 메모리 주소 영역을 설정하고 프로세스가 합법적인 영역만 접근하도록 한다.

base register는 합법적으로 접근할 수 있는 가장 작은 물리 메모리 주소 값을 저장하고 limit register는 주어진 영역의 크기를 저장한다.

사용자 모드에서 수행되는 프로그램이 운영체제의 메모리 공간에 접근하면 운영체제는 치명적인 오류로 간주하고 트랩을 발생시킨다. 이를 통해 사용자 프로그램이 운영체제나 다른 사용자 프로그램의 코드나 데이터 구조를 수정하는 것을 막는다.

![hardware-address-protection](/assets/images/2023-12-21-os-ch-9-main-memory/hardware-address-protection.png)

base register와 limit register는 운영체제에 의해서만 적재된다. 특권 명령은 오직 커널 모드에서만 수행되고, 운영체제만 커널 모드에서 수행되기 때문이다. 사용자 프로그램이 레지스터의 내용을 변경하는 것을 막는다.

커널 모드에서 수행되는 운영체제는 운영체제 메모리 영역과 사용자 메모리 영역의 접근에 어떠한 제약도 받지 않는다.  
이러한 원칙 하에 운영체제는 사용자 프로그램을 사용자 메모리 영역에 적재, 덤프, 시스템콜 매개변수 변경, 입출력 등 다양한 기능을 제공한다.

### 9.1.2 주소의 할당

프로그램은 원래 이진 실행 파일(binary executable file) 형태로 디스크에 저장되어 있다. 실행하려면 프로그램을 메모리로 가져와서 프로세스 문맥(context of a process) 내에 배치해야한다. 이후 CPU에서 실행할 수 있게 된다.  
프로세스가 실행되면 메모리의 명령과 데이터에 접근한다. 프로세스가 종료되면 메모리가 회수된다.

대부분의 시스템은 사용자 프로세스가 물리 메모리 내 어느 부분에 올라와도 문제가 없도록 지원한다.

![multistep-processing-of-a-user-program](/assets/images/2023-12-21-os-ch-9-main-memory/multistep-processing-of-a-user-program.png)

사용자 프로그램은 여러 단계를 거쳐 실행된다. 단계를 거치는 동안 주소는 여러 가지 다른 방식으로 표현한다.

- 프로그램에서 주소는 숫자가 아닌 심볼 형태로 표현된다. (e.g. `int x`)
- 컴파일러는 이 심볼 주소를 재배치 가능 주소로 바인딩 시킨다.
- 링커와 로더가 재배치 가능 주소를 절대 주소로 바인딩 시킨다.

각각의 바인딩 과정은 한 주소 공간에서 다른 주소 공간으로 맵핑하는 것이다.

바인딩은 시점에 따라 다음과 같이 구분된다.

- 컴파일 타임 바인딩 : 프로세스가 메모리 내에 들어갈 위치를 컴파일 시간에 미리 알 수 있다면 컴퍼일러는 absolute code를 생성할 수 있다.
- 로드 타임 바인딩 : 프로세스가 메모리 내 어디로 올라오게 될지를 컴파일 파임에 알 지 못하면 컴파일러는 일단 바이너리 코드를 재배치 가능 코드(relocatable code)로 만든다. 심볼과 진짜 번지수와의 바인딩은 프로그램이 메인 메모리로 적재되는 시간에 이루어지게 된다.
- 실행 타임 바인딩 : 실행 중간에 메모리 내의 한 세그먼트로부터 다른 세그먼트로 옮겨질 수 있다면 바인딩이 실행 타임 까지 허용되었다고 한다. 대부분의 os에서 이 방식을 사용한다.

### 9.1.3 논리 대 물리 주소 공간

CPU가 생성하는 주소를 **논리 주소(Logical address)** 라고 한다.
메모리가 취급하게 되는 주소는 **물리 주소(Physical address)** 라 한다. (**메모리 주소 레지스터(memory-address register)**에 주어진다.)

컴파일 또는 로드 시에 주소를 바인딩하면 논리 주소와 물리 주소가 같다.
그러나 실행 타임 바인딩 기법에서는 논리, 물리 주소가 다르다.

이러면 논리 주소를 **가상 주소(virtual address)**라 한다. (이 책에서는 논리 주소와 가상 주소를 같은 뜻으로 사용한다.)

프로그램에 의해 생성된 모든 논리 주소의 집합을 **논리 주소 공간(logical address space)** 라고 하며 이 논리 주소와 연결되는 모든 물리 주소 집합을 **물리 주소 공간(physical address space)** 이라 한다.

프로그램의 실행중에는 가상 주소를 물리 주소로 바꿔줘야 하는데 이 mapping 작업은 **메모리 관리 장치 (memory management unit, MMU)** 에 의해 처리된다.

![mmu-relocation-register](/assets/images/2023-12-21-os-ch-9-main-memory/mmu-relocation-register.png)

base register를 **재배치 레지스터(relocation register)** 라고도 부른다.
재배치 레지스터 속에 들어있는 값은 주소가 메모리로 보내질 때마다 그 모든 주소에 더해진다. (논리주소 346 + 재배치 레지스터 값 14,000 = 물리 주소 14,346)

사용자 프로그램은 실제적인 물리 주소에 접근하지 못한다. 포인터를 생성해서 그것에 대해 저장, 연산, 다른 주소들과 비교하는 등 다양한 일을 처리한다.
간접 로드 및 저장 시에는 기준 레지스터에 의해 상대적인 주소로 접근한다. 사용자 프로그램은 논리 주소를 사용하고, MMU가 물리 주소로 바꾸어준다.

### 9.1.4 동적 로딩 (dynamic loading)

지금까지의 설명에서는 프로세스가 실행되기 위해 그 프로세스 전체가 미리 메모리에 올라와 있는 것을 전제로 설명하였다.
이 경우 프로세스의 크기는 메모리의 크기보다 커질 수 없다.

메모리 공간의 더 효율적으로 이용하기 위해서는 동적 적재를 사용해야 한다.

동적 적재에서 각 루틴은 실제 호출되기 전까지는 메모리에 올라오지 않고 재배치 가능한 상태로 디스크에서 대기하고 있다.

> 루틴 : 특정 작업을 수행하는 소프트웨어 어플리케이션의 컴포넌트

먼저는 main 프로그램이 메모리에 올라와 실행되고, 이 루틴이 다른 루틴을 호출하면 호출된 루팅이 메모리에 로드 됐는지를 확인한다. 로드 되어 있지 않다면 재배치 가능 연결 로더(relocatable linking loader)를 통해 호출된 루틴을 메모리로 가져오고 변경된 부분을 테이블에 기록해둔다. 그 후 CPU 제어는 중단되었던 루틴으로 보내진다.

동적 로딩의 장점은 루틴이 필요한 경우에만 적재된다는 것이다.

### 9.1.5 동적 연결 및 공유 라이브러리

동적 연결 라이브러리(DLL)은 사용자 프로그램이 실행될 때 사용자 프로그램에 연결되는 시스템 라이브러리이다.

동적 연결 개념은 동적 적재의 개념과 유사하다. 동적 로딩에서는 로딩이 실행 시까지 미루어졌었지만 동적 연결은 연결이 실행 시까지 미루어진다.
DLL을 통해 크기를 줄이고 메인 메모리의 낭비를 줄인다. DLL을 사용하면 여러 프로세스 간에 라이브러리를 공유할 수 있다. (이러한 이유로 DLL은 공유 라이브러리 라고도 함.)

프로그램이 동적 라이브러리에 있는 루틴을 참조하면 로더는 DLL을 찾아 필요한 경우 메모리에 적재한다. 그런 다음 동적 라이브러리의 함수를 참조하는 주소를 DLL이 저장된 메모리의 위치로 조정한다.

이렇게 하면 추가 라이브러리는 어느 때나 새로운 버전으로 교체될 수 있고, 이 라이브러리를 사용하는 모든 프로그램은 추가적인 링크 작업이 없이 자동으로 새로운 버전을 사용할 수 있게 된다.

동적 로딩과는 달리 동적 연결과 공유 라이브러리는 운영체제의 도움이 필요하다. 운영체제를 통해 같은 메모리 주소를 함께 접근할 수 있도록 해준다.

> 동적 로딩은 실행 중에 필요한 코드나 리소스를 메모리에 로드하는 것에 중점을 두고,  
> 동적 연결은 실행 중에 외부 라이브러리와의 연결을 동적으로 처리하는 것에 중점을 둠.