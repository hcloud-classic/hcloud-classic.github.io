---
title: 소프트웨어 정의서버 테스트 케이스 수행 메뉴얼
subtitle: test
layout: post
icon: fa-book
order: 3

---






해당 메뉴얼은 DISTANT_FORK 계층을 사용자가 지정하여, 특정 계층에서만 DISTANT_FORK 가 수행되도록 하는 방법과 이를 통해 테스트 케이스를 수행 하는 방법을 안내해 드립니다.

메뉴얼을 참고 하시어 각 작업을 수행해 보시고 하단의 링크를 통해 설문에 응답 부탁드립니다.

[HCloud-Classic 서비스 만족도 평가 설문지](https://docs.google.com/forms/d/1BZV4U_x10NBe5-Q66pat6AfQoxYIWqWJTv7YxyYk5hs/edit?usp=sharing)

# 병렬 프로그래밍

## Distant Fork?

DISTANT_FORK 는 현재 실행중인 프로세스의 자식 프로세스를 다른 노드로 이동시킨 후 작업을 처리하도록 하는 기능입니다.

자식 프로세스가 생성 될때마다 계층이 하나씩 증가하며, HCC 에서는 DISTANT_FORK 가 수행되는 계층을 제한하도록 하는 기능을 제공하고 있습니다.



## DISTANT_FORK 계층 제한 개선

- 계층 제한 개수 및 범위 확장

  DISTANT_FORK 계층 제한은 15개를 지정할 수 있으며, 각 계층 제한은 최대 127 Depth 까지 설정이 가능합니다.

![image-20211202151942007](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/image-20211202151942007.png)



- - DISTANT_FORK, MIGRATE 등을 수행할지 지정하고, 현재 계층 (Current Depth) 과 계층 제한이 설정된 목록을 확인 할 수 있습니다.

  - DISTANT_FORK 는 다음과 같이 활성화 합니다.

    ```shell
    # hccgcapset -e +DISTANT_FORK
    # hccgcapset -d +DISTANT_FORK
    ```
  
  - 현재 계층 (Current Depth) 과 계층 제한이 설정된 목록은 다음과 같이 확인합니다.

    ![image-20211202150122732](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/image-20211202150122732.png)

    ```shell
    # hccgcapset -s
    Permitted Capabilities: 05067
    	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
    	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
    Effective Capabilities: 01
    	CHANGE_KERRIGHED_CAP
    Effective Capabilities Depth: 
    	Current Depth: 9
    	Depth Limit  : 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
    Inheritable Permitted Capabilities: 05067
    	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
    	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
    Inheritable Effective Capabilities: 01
    	CHANGE_KERRIGHED_CAP
    ```
  
  - 계층제한 목록은 다음과 같이 설정합니다. 

    - 15개를 지정할 수 있으며, 각 계층 제한은 최대 127 Depth 까지 설정이 가능합니다.

    - 각 계층은 띄어쓰기로 구분합니다.
    
    - 빈 문자열 `""` 를 입력 할 경우 계층 제한 리스트가 모두 0으로 초기화 됩니다.
  
    - Depth 는 Kernel 부팅 후 모든 프로세스에 적용되기 때문에 local 이나 SSH 로 사용중인 Shell 에서의 Depth 는 0이 아닙니다.
    
    - 계층은 연속된 숫자일 필요가 없으며, 지정 계층에서만 Distant Fork 가 적용됩니다.
    
      ```
      # krgcapset -D "10 11 12 13 14 15"
      ```
    
    - 다음과 같이 최대 Depth 를 지정할 수 있습니다.
    
      ```
      # krgcapset -D "11 12 13 100 127"
      ```

![image-20211202153811632](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/image-20211202153811632.png)



- 사용시 주의 사항

  - 자동화 스크립트를 작성하실 경우 현재 실행중인 Shell 의 Current Depth 를 확인한 후 이보다 큰 숫자로 Depth 를 설정해야 합니다.

    ex) 현재 실행중인 bash Shell 에서 `hccgcapset -s` 로 Current Depth 를 확인한 후 Current Depth 가 10 일 경우 이보다 큰 숫자로 계층 제한 리스트를 지정 해야 합니다. (`hccgcapset -D "11 13"`)

  - DISTANT_FORK 권한이 부여된 프로세스에 한해서만 계층 제한 검사를 수행합니다.

  ![image-20211202150624390](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/image-20211202150624390.png)



- 중첩 사용

  - DISTANT_FORK 가 된 프로세스에서 `hccgcapset -D` 명령으로 계층제한을 추가로 지정할 수 있습니다.

  ![image-20211202153104233](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/image-20211202153104233.png)





# 테스트 케이스

하단에 안내 되는 모든 테스트 케이스는 자동으로 수행 되어 분산 되어 있는 각 클러스터에 Fork 되어 멀티코어를 사용 할 수 있도록 자동화 스크립트가 준비되어 있습니다.

## Kernel Compile 을 위한 make

Kernel을 Compile할때에 주로 Makefile에 선언된 절차 대로 실행 하여 빌드를 합니다.

빌드 시에 `make -j xx` 와 같이 `-j` 옵션을 설정 하게 되면, 각 프로세서에 병렬로 할당 되어 수행되는 Job의 수를 설정 할 수 있습니다.

### 수행 작업

기존의 단일 머신에서는 호스트 머신내에서만 병령 프로세싱이 가능하지만, 소프트웨어 정의서버는 각 머신을 클러스터링 하여 통합된 환경에서 동작 가능하게 해줍니다.

### 테스트 수행

- 2개의 Node 로 단일서버를 구성한 후 Leader Node 에서 아래 명령어 들을 차례로 실행하여 make 를 테스트 했을때의 예시입니다.

- Current Depth 는 10 이고, make 명령어 바로 하위 계층(11 Depth) 에서 생성된 프로세스에 대해서만 DISTANT_FORK 를 수행하도록 하였습니다.

  ```shell
  # hccadm nodes add -n2
  # hccadm nodes
  1:online
  2:online
  # hccgcapset -s
  Permitted Capabilities: 01067
  	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
  	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT
  Effective Capabilities: 01
  	CHANGE_KERRIGHED_CAP
  Effective Capabilities Depth: 
  	Current Depth: 10
  	Depth Limit  : 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
  Inheritable Permitted Capabilities: 01067
  	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
  	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT
  Inheritable Effective Capabilities: 01
  	CHANGE_KERRIGHED_CAP
  # hccgcapset -d +DISTANT_FORK
  # hccgcapset -D "11"
  # hccgcapset -s
  Permitted Capabilities: 01067
  	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
  	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT
  Effective Capabilities: 01
  	CHANGE_KERRIGHED_CAP
  Effective Capabilities Depth: 
  	Current Depth: 10
  	Depth Limit  : 11 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
  Inheritable Permitted Capabilities: 01067
  	CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
  	USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT
  Inheritable Effective Capabilities: 05
  	CHANGE_KERRIGHED_CAP, DISTANT_FORK
  # cd /usr/src/centos_hcc_kernel/
  # make -j100
  ```


![devuan_make_test](/assets/images/manual/2021-12-02-manual_DISTANT_FORK 계층 제한 방법/devuan_make_test.png)



## Openmpi 기반의 소수 찾기 프로그램

HCC 시스템에서 DISTANT_FORK 의 동작을 테스트 해보시기 위한 스크립트가 준비 되어 있습니다.

`/root/bench_test` 디렉토리에 해당 테스트 파일들이 준비 되어 있습니다.

openmpi 테스트 파일이 있는 디렉토리로 이동합니다.

```
cd /root/bench_test/openmpi_test
```

### 수행 작업

총 2번의 소수찾기 테스트 케이스를 수행 합니다.

1. 1~131072 까지의 소수를 2의 배수로 구간을 분할하여 소수를 찾습니다.
2. 5~50000 까지의 소수를 10의 배수로 구간을 분할하여 소수를 찾습니다.

```c
  int n_factor;
  int n_hi;
  int n_lo;

  printf ( "\n" );
  printf ( "PRIME_OPENMP\n" );
  printf ( "  C/OpenMP version\n" );

  printf ( "\n" );
  printf ( "  Number of processors available = %d\n", omp_get_num_procs ( ) );
  printf ( "  Number of threads =              %d\n", omp_get_max_threads ( ) );

  n_lo = 1;
  n_hi = 131072;
  n_factor = 2;

  prime_number_sweep ( n_lo, n_hi, n_factor );

  n_lo = 5;
  n_hi = 500000;
  n_factor = 10;

  prime_number_sweep ( n_lo, n_hi, n_factor );
/*
  Terminate.
*/
  printf ( "\n" );
  printf ( "PRIME_OPENMP\n" );
  printf ( "  Normal end of execution.\n" );

  return 0;
```



### 테스트 수행

현재 사용중인 쉘의 `Current Depth` 를 확인합니다.

현재 예제에서는 `Current Depth` 가 9 입니다.

```
# hccgcapset -s
Permitted Capabilities: 05067
        CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
        USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
Effective Capabilities: 01
        CHANGE_KERRIGHED_CAP
Effective Capabilities Depth:
        Current Depth: 9
        Depth Limit  : 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
Inheritable Permitted Capabilities: 05067
        CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
        USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
Inheritable Effective Capabilities: 01
        CHANGE_KERRIGHED_CAP
```



`openmpi_tester.sh` 파일을 열어서 계층 제한 하는 부분을 수정해줍니다.

`hccgcapset -D "10 11 12 13"` 의 `-D` 옵션 부분을 수정해 줍니다.

현재 예제 에서는 4개의 노드로 단일 가상 서버를 구성했습니다.

Currnet Depth 가 9 이고, 계층 제한을 "10 11 12 13" 으로 했기 때문에 자식 프로세스가 생길 경우 최대 4번까지 DISTANT_FORK 가 각 노드를 통해 수행하게 됩니다.



- `vi openmpi_tester.sh`

```
#!/bin/bash
EXEDIR='/root/openmpi_src'

hccgcapset -d +DISTANT_FORK
hccgcapset -D "10 11 12 13"
mpirun -n 35 $EXEDIR/count_prime 100
```



스크립트 파일을 실행 하면 openmpi 프로세스가 자식 프로세스가 생길때 마다  DISTANT_FORK 를 수행하며 테스트를 진행하실 수 있습니다.

```
# ./openmpi_tester.sh
```



## R

`/root/bench_test` 디렉토리에 해당 테스트 파일들이 준비 되어 있습니다.

R 테스트 파일이 있는 디렉토리로 이동합니다.

```
cd /root/bench_test/R_test
```

### 수행 작업

해당 예제는 특잇값 분해(SVD)를 R을 통해 병렬 처리하여 멀티코어에서 작업을 수행 하게 합니다.

수행되는 R 코드는 하단과 같습니다.

```R
#! /usr/bin/env Rscript
library("doMC")
registerDoMC()
system.time(x <- foreach(i=1:42) %dopar% svd(matrix(rnorm(1000*1000),ncol=1000)))
```



### 테스트 수행

현재 사용중인 쉘의 `Current Depth` 를 확인합니다.

현재 예제에서는 `Current Depth` 가 9 입니다.

```
# hccgcapset -s
Permitted Capabilities: 05067
        CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
        USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
Effective Capabilities: 01
        CHANGE_KERRIGHED_CAP
Effective Capabilities Depth:
        Current Depth: 9
        Depth Limit  : 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
Inheritable Permitted Capabilities: 05067
        CHANGE_KERRIGHED_CAP, CAN_MIGRATE, DISTANT_FORK, CHECKPOINTABLE
        USE_REMOTE_MEMORY, SEE_LOCAL_PROC_STAT, SYSCALL_EXIT_HOOK
Inheritable Effective Capabilities: 01
        CHANGE_KERRIGHED_CAP
```



`hcc_test.sh` 파일을 열어서 계층 제한 하는 부분을 수정해줍니다.

`hccgcapset -D "10 11 12 13"` 의 `-D` 옵션 부분을 수정해 줍니다.

현재 예제 에서는 4개의 노드로 단일 가상 서버를 구성했습니다.

Currnet Depth 가 9 이고, 계층 제한을 "10 11 12 13" 으로 했기 때문에 자식 프로세스가 생길 경우 최대 4번까지 DISTANT_FORK 가 각 노드를 통해 수행하게 됩니다.

- `vi hcc_test.sh`

```
#!/bin/sh
killall R &
hccgcapset -d +DISTANT_FORK
hccgcapset -D "10 11 12 13"
/root/auto/domc.R
```



스크립트 파일을 실행 하면 R 프로세스가 자식 프로세스가 생길때 마다  DISTANT_FORK 를 수행하며 테스트를 진행하실 수 있습니다.

```
# ./hcc_test.sh
```

