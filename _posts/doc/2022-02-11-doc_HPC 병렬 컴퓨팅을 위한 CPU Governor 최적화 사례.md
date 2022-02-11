---
title: HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례
subtitle: 
layout: post
icon: fa-book
order: 3


---



[HCloud Classic](https://github.com/orgs/hcloud-classic/teams/innogrid) 프로젝트에서는 자체 개발된 Kernel로 단일 가상 환경에서의 병렬 컴퓨팅을 지원 하여 HPC의 성능을 극대화 합니다.
HPC 성능의 극대화를 위한 Hcloud System에서의 가버너 최적화 사례를 공유 드리려 합니다.

## CPU Governor?

CPU는 클럭을 높여 속도를 높이거나 전력을 절감하기 위해 클럭을 낮출 수 있습니다.
이때 고클럭을 사용 할 수록 높은 전압이 들어가고 CPU는 더 많은 전력을 소비하게 됩니다.
반대로, 저클럭에서는 사용 전압이 낮아지고 적은 전력을 소비하게 됩니다.

이러한 특성 때문에 휴대용 기기에서는 배터리의 사용 시간을 증가시키기 위해서 기기가 사용 중일 때는 높은 성능을 내기 위해 클럭을 올리고, 진행 중인 작업이 없거나 화면이 꺼져있거나 하는 등, 기기가 사용 중이지 않을때 전력 소모를 줄이기 위해 클럭을 내리게 됩니다.

또한, 데스크탑 PC나 서버에서 항상 최고의 성능을 내기 위해서는 CPU가 항상 최고 클럭을 유지하도록 해야 합니다.

이때 사용 되는 것이 CPU Governor(가버너) 입니다.

가버너는 CPU 의 부하 정도에 따라서 CPU 의 클럭을 올리거나 낮추는 역할을 수행합니다.
가버너의 종류에 따라 CPU 클럭이 낮은 상태나 높은 상태로 계속 유지되기도 하고 부하 정도에 따라 동적으로 계속 변화하기도 합니다.

사용하는 가버너에 따라서 스마트폰이나 태블릿 같은 휴대용 기기에서는 배터리의 사용시간이 증가할 수도 있고 성능을 우선시하게 하여 배터리가 빨리 감소할 수도 있습니다.
이러한 이유로, 스마트기기 제조사나 Custom Kernel 을 제작하는 사용자들은 새로운 가버너를 개발하여 사용하기도 합니다.
또한 개발된 가버너에 따라, CPU의 부하 정도에 따른 클럭 변화 값을 지정할 수도 있습니다.

## Governor 종류
Vanila Kernel 에 포함된 x86 에서 사용되는 가버너들에는 다음과 같은 것들이 있습니다.

* performance : CPU 클럭을 최대로 유지 (성능 우선) 합니다.
* powersave : CPU 클럭을 최저 상태로 유지 (전력 절감 우선) 합니다.
* ondemand : CPU 사용량에 따라 CPU 클럭을 변경 합니다.
* userspace : CPU를 사용자가 지정한 주파수로 동작하게 합니다.
* conservative : ondemand 와 비슷하지만, CPU 클럭을 바로 올리지 않고 천천히 서서히 올리게 합니다. (전력 절감)
* schedutil : 스케줄러의 로드율에 따라 CPU 클럭을 변경합니다.

그 외에도, Android Kernel 에는 다양한 가버너들이 존재합니다.

ondemand 에서는 CPU 부하가 조금만 걸려도 CPU 클럭이 최대치로 올라가서 배터리 소모량이 많아지게 됩니다.

따라서, 부하율에 따른 CPU 상승률을 세분화하고 터치스크린 작동시에 CPU 상승이 되도록 하는등의 기능을 추가한 interactive 라는 가버너가 추가되어 많이 사용됩니다.

## Governor 변경하기
그럼 이제 실제로 Governor 를 변경해 보도록 하겠습니다.

CPU는 Intel CPU를 기준으로 테스트하여 작성하였습니다.

Intel CPU 의 경우 `intel_pstate` 드라이버가 사용중인 경우에 사용가능한 가버너가 powersave 와 performance 2가지만 가능하여서 해당 드라이버를 비활성화 해주는 작업이 필요합니다.

GRUB 기본 설정파일을 열어 다음과 같이 `GRUB_CMDLINE_LINUX` 부분을 수정해 줍니다.

`vi /etc/default/grub`

``` bash
GRUB_CMDLINE_LINUX="intel_pstate=disable"
```

다음으로 GRUB 설정파일을 업데이트 하기 위해서 아래 명령을 실행합니다.

``` bash
$ sudo update-grub
```

시스템을 재시작합니다.

``` bash
$ sudo init 6
```

재부팅을 하고 난 후 `/sys/devices/system/cpu/cpu0/cpufreq` 디렉터리가 존재하는지 확인합니다. Intel x86 환경에서 cpufreq 드라이버들이 모듈화되어있고, 로드되지 않으면 디렉터리가 보이지 않을 수 있습니다.

이 경우에는 다음 명령들을 실행하여 모듈을 로드해야 합니다.

``` bash
$ sudo modprobe acpi-cpufreq
```

현재 사용가능한 가버너 목록은 다음과 같이 확인할 수 있습니다.

``` bash
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
 conservative ondemand userspace powersave performance schedutil
```

가버너는 다음과 같이 변경할 수 있습니다.

``` bash
$ echo ondemand | sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```

## Governor 에 따른 클럭 변화 및 성능 분석

이번에는 Governor 에 따라서 클럭이 어떻게 변화하고 성능에 어느 정도 영향이 있는지 테스트해 보기로 하겠습니다.

첫 번째 CPU 의 클럭 변화를 통해 클럭 변화를 확인하고, `time` 명령어를 통해서 작업 수행시간이 얼마나 걸리는지 측정해 보도록 하겠습니다.

여기서는 `mpirun` 이라는 명령을 통해 `count_prime` 이라는 소수를 찾는 작업을 하는 프로그램을 실행하도록 하겠습니다.

첫 번째 CPU 의 클럭 변화는 다음 명령을 통해 확인 할 수 있습니다.
1초마다 클럭의 변화를 체크합니다.

``` bash
$ watch -n1 cat /sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_cur_freq
```

![Untitled.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled.png)<br>

`mpirun` 의 작업수행 시간은 다음과 같이 확인하였습니다.

``` bash
# time /opt/openmpi/bin/mpirun --allow-run-as-root --oversubscribe -n 18 count_prime 100
```

가버너의 세부 설정을 조절할 수 있는 `/sys/devices/system/cpu/cpufreq/[가버너명]` 폴더에 있는 값들은 모두 기본값을 사용하였습니다.

* ondemand
CPU 부하율이 올라가면 CPU 클럭을 올리고, CPU 부하율이 낮아지면 CPU 클럭을 내립니다.
현재 작업이 진행 중이지 않을 때의 클럭은 다음과 같습니다.
![Untitled 1.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 1.png)<br>
`mpirun` 명령을 실행하는 순간 CPU 클럭이 최대치로 상승하였습니다.
![Untitled 2.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 2.png)<br>
테스트 결과는 다음과 같습니다.

``` bash
real   0m21.161s
user   12m23.212s
sys    0m0.143s
```

* conservative
CPU 부하율이 오르면 CPU 클럭이 오르고 CPU 부하율이 내려가면 CPU 클럭이 내려갑니다. 단, 클럭의 변화가 바로 상승하거나 하강하지 않고 서서히 변화합니다.
`mpirun` 명령을 실행하는 순간 CPU 클럭이 바로 최대치로 되지 않고 약간 상승한 모습을 보여주고 있습니다.
![Untitled 3.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 3.png)<br>
잠시 후에 CPU 클럭이 최대치로 되는 것을 볼 수 있습니다.
![Untitled 4.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 4.png)<br>
작업이 끝나고 나서도 CPU 클럭이 서서히 내려가는 것을 확인할 수 있습니다.
![Untitled 5.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 5.png)<br>
![Untitled 6.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 6.png)<br>
![Untitled 7.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 7.png)<br>
테스트 결과는 다음과 같습니다.
ondemand 보다 약간 시간이 더 걸린 것을 확인할 수 있습니다.

``` bash
real   0m21.119s
user   12m24.324s
sys    0m0.1127s
```

* powersave
CPU 클럭을 항상 최저 상태로 유지합니다.
`mpirun` 명령을 실행해도 CPU 클럭이 오르지 않고 최저 클럭에 유지되어 있는 모습을 볼 수 있습니다.
![Untitled 8.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 8.png)<br>
테스트 결과는 다음과 같습니다. conservative 보다 시간이 더 걸린 것을 확인할 수 있습니다.

``` bash
real   0m22.489s
user   12m29.3283
sys    0m0.143s
```

* performance
CPU 클럭을 항상 최고 상태로 유지합니다.
`mpirun` 명령을 실행하지 않아도 최고 클럭에 유지되어 있는 모습을 볼 수 있습니다.
![Untitled 9.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 9.png)<br>
역시 `mpirun` 명령을 실행중에도 클럭은 항상 최고 클럭을 유지하고 있습니다.
![Untitled 10.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 10.png)<br>
테스트 결과는 다음과 같습니다. ondemand 와 거의 비슷한 시간을 보여주고 있습니다.

``` bash
real   0m21.119s
user   12m26.378
sys    0m0.118s
```

userspace 은 설정한 클럭에 따라 클럭이 고정적이 때문에 테스트 대상에서 제외하도록 하겠습니다.
테스트 결과를 종합해 보면 다음과 같습니다.

```
|      | ondemand   | conservative | powersave  | performance |
| ---- | ---------- | ------------ | ---------- | ----------- |
| real | 0m21.161s  | 0m21.119s    | 0m22.489s  | 0m21.119s   |
| user | 12m23.212s | 12m24.324s   | 12m29.3283 | 12m26.378   |
| sys  | 0m0.143s   | 0m0.1127s    | 0m0.143s   | 0m0.118s    |
```

결론은 전기세 걱정 없이 언제나 최고 성능만을 원한다면 performance 를 사용하고, 그렇지 않고 사용하지 않을 때 전력 사용량을 줄이기 위해서는 ondemand 를 사용하는 것이 적절해 보입니다.

## CentOS 6 CPU 설정 분석

Hcloud Classic 에서는 단일 가상 서버를 구성을 위한 다양한 OS를 지원하고 있습니다. 그 중, CentOS 6 에서 Governor 설정이 어떻게 진행되는지 알아보도록 하겠습니다.

먼저 Governor 설정은 Kernel 설정을 변경하는 것이므로, CentOS 6 의 Kernel 소스를 다운받아서 Kernel Config 를 살펴보기로 하였습니다.

CentOS 6 Kernel 을 다운받아서 x86\_64 시스템의 Kernel Config 에 해당되는 `arch/x86/configs/x86_64_defconfig` 파일을 확인해 보면 기본값이 userspace 인 것을 확인할 수 있습니다.

![Untitled 11.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 11.png)<br>

하지만 부팅이 완료된 CentOS 6 의 가버너를 확인해 보면 ondemand 로 나오는 것을 확인할 수 있습니다.

``` bash
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
ondemand
```

부팅 과정에서 가버너 설정을 변경하는 것이 아닐지 추측해 봅니다.

실제로 확인하기 위해서, 서비스 관련 스크립트들이 위치한 `/etc/init.d` 에 cpu 관련 스크립트가 있는지 확인해 봅니다.

![Untitled 12.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 12.png)<br>

cpuspeed 파일이 있는 것을 확인할 수 있습니다.

해당 파일을 열어서 확인해보면 서비스를 시작하는 부분에 특정 CPU 에 맞는 CPUFREQ 모듈을 사용하지 않는 경우 기본적으로 `acpi-cpufreq` CPUFREQ 모듈을 로드 하는 것을 확인할 수 있습니다.

![Untitled 13.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 13.png)<br>

이후에 `acpi-cpufreq` 를 사용하는 경우 `cpufreq-(가버너명)` 의 모듈을 로드하는 것을 확인할 수 있습니다.

![Untitled 14.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 14.png)<br>

다음으로 adjust\_cpufreq 함수에서 scaling\_governor 와 governor 를 인자로 받아오는 것을 볼 수 있습니다.

![Untitled 15.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 15.png)<br>

adjust\_cpufreq 함수에서 scaling\_governor 부분을 governor 인자로 변경하는 것을 확인할 수 있습니다.

마지막으로 변경한 governor 가 userspace 가 아니면 governor 의 세부 설정을 적용하는 것을 확인할 수 있습니다.

![Untitled 16.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 16.png)<br>

종합적으로 정리해 보면 CentOS6 에서는 가버너가 변경되는 과정은 다음과 같습니다.

1. 처음에 cpufreq 드라이버가 로드되지 않은 상태로 부팅이 시작됩니다.
2. acpi-cpufreq 모듈을 로드하면서 Kernel 컴파일 시 설정된 가버너의 기본값인 userspace 로 설정됩니다.
3. ondemand 로 가버너를 변경합니다.
4. 이후에 해당 가버너에 맞는 튜닝 값이 설정됩니다.

## Kernel 의 기본 Governor 가 ondemand 가 되도록 CentOS 6 Kernel 컴파일하기

CentOS 6 Kernel 소스를 다운로드 하여 압축을 풀고 해당 디렉터리에 진입합니다.

``` bash
$ tar xvf linux-2.6.32-754.35.1.el6.tar.bz2
$ cd linux-2.6.32-754.35.1.el6.tar.bz2
```

아래 명령어를 실행하여 Kernel 설정 파일을 생성합니다.

``` bash
$ make defconfig
```

생성된 `.config` 파일을 열어 scaling 을 검색합니다.

USERSPACE 가 기본 거버너로 되어 있는 것을 확인할 수 있습니다.

`vi .config`

![Untitled 17.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 17.png)<br>

ondemand 를 기본 가버너로 하고자 할 경우 GOVERNOR 관련 부분과 CPUFREQ 관련 부분을 아래와 같이 수정해 줍니다.

다른 CPUFREQ 드라이버를 비활성화하고 acpi-cpufreq 드라이버만 사용하도록 하였습니다.

가버너들도 모듈이 아닌 built-in 드라이버를 사용하도록 하였습니다.

![Untitled 18.png](/assets/images/docs/HPC 병렬 컴퓨팅을 위한 CPU Governor 최적화 사례/Untitled 18.png)<br>

이제 아래 명령어를 사용하여 변경된 사항을 적용합니다.

``` bash
$ make oldconfig
```

Kernel 을 컴파일합니다. (j 옵션은 CPU 코어수 \* 1.2배 정도(반올림)로 입력합니다.)

``` bash
$ make -j20
```

빌드된 모듈을 설치합니다.
해당 명령은 `/lib/modules` 디렉터리에 `[Kernel 버젼]-[Kernel 이름]` 폴더명으로 Kernel 의 모듈들을 설치합니다.

``` bash
$ sudo make modules_install
```

마지막으로 빌드된 Kernel 을 설치합니다.
해당 명령은 `/boot` 디렉터리에 Kernel 과 램디스크를 설치하고, GRUB 부팅 메뉴에 새로운 Kernel 항목을 등록합니다.

``` bash
$ make install
```

## 결론

지금까지 CPU Governor 가 어떻게 설정되고, CPU Governor 에 따라 CPU 클럭이 어떻게 변화하는지 살펴보았습니다.

사용 용도에 따라서 성능이 먼저냐, 에너지 효율이 먼저냐에 따라서 CPU Governor 를 적절하게 선택하여 사용하시면 되겠습니다.

긴 글 읽어 주셔서 감사합니다.
