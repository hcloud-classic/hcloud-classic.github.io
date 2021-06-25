---
title: iptables 를 활용한 방화벽 구성하기
subtitle: 
layout: post
icon: fa-book
order: 3
---

# iptables 를 활용한 방화벽 구성하기



본 글에서는 Linux Kernel 내부에 구현되어 있는 netfilter 라는 패킷 필터링 기능을 사용자 환경에서 제어할 수 있게 해주는 iptables 를 활용하여 방화벽을 구성하는 방법에 대해서 알아보도록 하겠습니다.



## iptables Tables

- iptables 에는 4개의 table이 존재합니다.
  - filter : iptables 의 기본 Table 로써 패킷을 통과시킬지 여부, 즉 패킷 필터링을 담당합니다.
  - nat : Network Address Translation 의 약자로써. IP 주소 변환을 담당한다. Adaptive IP 에서 외부주소와 내부주소간의 변환을 위해 필요합니다.
  - mangle : 패킷의 데이터를 변경하는 특수 규칙을 적용하거나 성능 향상을 위한 TOS(Type Of Service) 를 설정합니다.
  - raw : Kernel 의 Netfilter 의 연결 추적 하위 시스템과 독립적으로 동작해야 하는 규칙을 설정할때 사용합니다.
- 본 글에서는 filter 와 nat 테이블을 활용하여 방화벽을 구성합니다.



## iptables Chains

- iptables 의 filter Table 에는 3개의 기본 Chain 으로 구성되어 있으며 각 Chain 의 역할은 다음과 같습니다.
  - INPUT : 외부에서 Linux Machine 으로 들어오는 패킷들에 대한 정책을 설정합니다.
  - FORWARD : Linux Machine 내부에서 서로 다른 네트워크 인터페이스간 Forwarding 을 하기 위한 정책을 설정하며, NAT 가 이에 해당됩니다.
  - OUTPUT : Linux Machine 으로 부터 외부로 나가는 패킷들에 대한 정책을 설정합니다.



- iptables 의 nat Table 에는 2개의 기본 Chain 으로 구성되어 있으며 각 Chain 의 역할은 다음과 같습니다.
  - PREROUTING : Linux Machine 으로 패킷이 들어오는 시점에 해당됩니다. INPUT 보다 앞선 시점이며, PREROUTING 을 통해 들어온 패킷에 대해 Routing 을 어떻게 처리할지 결정합니다.
  - POSTROUTING : Linux Machine 으로 부터 패킷이 나가는 시점에 해당됩니다. filter Table 을 거친 패킷이 최후에 어떻게 처리될지 결정하는 시점입니다.

<p align="center">
<img src="/assets/images/docs/iptables 를 활용한 방화벽 구성하기/iptables_chain.png"/><br>
[iptables Chain 들의 동작과정]<br>
이미지 출처 : https://url.kr/ds2n3o
</p>



- 각각의 기본 체인에 사용자가 정의한 Chain 을 추가 할 수 있습니다.
  - 사용자 정의 Chain 을 사용하는 이유
    - 특정 어플리케이션에서 iptables 통한 설정을 제어해야 하는데 해당 어플리케이션에서 사용하고자 하는 모든 iptables 정책들을 초기화 하고자 하는 경우 사용자가 정의한 Chain 만 초기화 하면 됩니다.
    - 특정 어플리케이션에 해당되는 정책들을 한눈에 알 수 있으며 해당 어플리케이션에만 적용되는 정책들을 별도로 관리 할 수 있습니다.



## 네트워크 구성

<p align="center">
<img src="/assets/images/docs/iptables 를 활용한 방화벽 구성하기/network_diagram.png"/><br>
[네트워크 구성도]
</p>




위와 같이 네트워크를 구성해 보도록 하겠습니다.

- eth0 은 외부 인터페이스에 해당하며 인터넷과 연결되는 구간에 있는 인터페이스 입니다.
- eth1 과 eth2 는 내부 인터페이스에 해당하며, 각각 별도의 내부아이피 대역을 가지고 구성된 클러스터로 향하는 인터페이스 입니다.



### 외부 아이피를 내부 클러스터의 노드와 매핑하기

이제 eth0 에 외부 IP 를 할당하고 eth1 내부에 구성된 클러스터의 노드에 매핑시키도록 해보겠습니다.

 

먼저 eth0 에 외부 IP 를 할당합니다.

```shell
# ip address add 123.123.123.1/24 dev eth0
```



다음으로 NAT 구성을 위한 Chain 들을 생성해 줍니다. (-N 옵션 다음에 오는 이름을 임의로 지정하실 수 있습니다.)

```shell
# iptables -t filter -N FIREWALL_FORWARD
# iptables -t nat -N FIREWALL_POSTROUTING
# iptables -t nat -N FIREWALL_PREROUTING
```



생성한 Chain 들을 iptables 의 기본 Chain 들에 등록해 줍니다.

iptables 는 등록된 순서대로 규칙이 실행됩니다. 최 상단에 등록을 해주기 위해 `-I 1` 옵션을 사용해 줍니다.

규칙을 1번째 라인에 삽입한다는 의미 입니다.

```shell
# iptables -t filter -I FOWARD -I 1 -j FIREWALL_FORWARD
# iptables -t nat -I POSTROUTING -I 1 -j FIREWALL_POSTROUTING
# iptables -t nat -I PREROUTING -I 1 -j FIREWALL_PREROUTING
```



이제 생성한 Chain 에 NAT 를 등록해 주고, Forwarding 설정을 해줍니다.

- 외부아이피 : 123.123.123.1
- 외부 인터페이스 : eth0
- 내부아이피 : 172.31.10.1
- 내부 인터페이스 : eth1

```shell
# iptables -t nat -A FIREWALL_POSTROUTING -o eth1 -s 172.31.10.1 -j SNAT --to-source 123.123.123.1
# iptables -t nat -A FIREWALL_PREROUTING -i eth0 -d 123.123.123.1 -j DNAT --to-destination 172.31.10.1
# iptables -t filter -A FIREWALL_FORWARD -s 123.123.123.1 -j ACCEPT
# iptables -t filter -A FIREWALL_FORWARD -d 172.31.10.1 -j ACCEPT
```



각 옵션들의 의미는 다음과 같습니다.

- -t : Table 명
- -A : Chain 명
- -o : 나가는 Interface 명
- -i : 들어오는 Interface 명
- -s : Source
- -d : Destination

- -j : SNAT (Source NAT) / DNAT (DestinationNAT) 여부를 지정합니다.
- --to-source : POSTROUTING (패킷이 나가는 시점) 에 대해 내부 IP를 어떤 외부 IP로 변환 할지 지정합니다.
- --to-destination : PREROUTING (패킷이 들어오는 시점) 에 대해 외부 IP를 어떤 내부 IP로 변환 할지 지정합니다.



마지막으로, 위해서 설정한 NAT 규칙들을 사용하기 위해서 Kernel 의 IP Forwarding 옵션을 활성화 해줍니다.

```shell
# echo 1 > /proc/sys/net/ipv4/ip_forward
```



여기까지 설정하셨으면 이제 `123.123.123.1` 이란 외부 IP를 통해서 내부에 구성된 `172.31.10.1` 내부 IP 를 가진 노드에 접속 할 수 있습니다.

이제 필요한 포트들을 포워딩 하고 외부 IP 에 대해 내부에 있는 노드가 ping 에 대한 응답을 할 수 있도록 하는 방법에 대해서 알아보겠습니다.



### 포트 포워딩 하기

위에서 매핑한 `172.31.10.1` 노드의 SSH 를 `123.123.123.1`의 외부 주소와 `2222` 번 포트를 통해 연결 가능하도록 포트포워딩을 하는 방법에 대해서 알아보겠습니다.



- 외부아이피 : 123.123.123.1
- 외부 인터페이스 : eth0
- 외부 포트 : 2222/tcp
- 내부아이피 : 172.31.10.1
- 내부 인터페이스 : eth1
- 내부 포트 : 22/tcp

```shell
# iptables -t nat -A FIREWALL_PREROUTING -i eth0 -p tcp --dport 2222 -d 123.123.123.1 -j DNAT --to-destination 172.31.10.1:22
```



여기서 위에서 설명한 옵션들을 제외한 각 의미는 다음과 같습니다.

- -p : 프로토콜을 의미하며, tcp/udp/icmp 등을 지정해 줄 수 있습니다.
- --dport : 목적지 포트. 외부 포트에 해당됩니다.
- --to-destination : 내부 IP 와 내부 포트를 지정해 줍니다. `내부IP:내부포트` 형태로 지정해줍니다.



### 외부 IP로 요청한 PING 포워딩 하기

위에서 매핑한 `123.123.123.1`의 외부 주소로 ping 을 날리면  `172.31.10.1` 노드에서 응답하도록 하는 방법에 대해서 알아보겠습니다.



```shell
# iptables -t nat -A FIREWALL_PREROUTING -i eth0 -p icmp --icmp-type echo-request -d 123.123.123.1 -j DNAT --to-destination 172.31.10.1
```



- ping 요청은 ICMP Request, ping 응답은 ICMP Reply 에 해당됩니다.
- 외부로 부터 온 ping 요청에 대해 포워딩을 해야 하기 때문에 protocol (`-p`) 는 `icmp` type (`--icmp-type`) 은 `echo-request` 로 지정해 줍니다.



이제 필요한 포트와 ping 을 포워딩 하였으니 나머지 응답들을 차단 시키도록 해보겠습니다.



### INPUT 제한하기

위에서 포워딩한 `2222/tcp` 포트와 ping 요청을 제외하고 나머지 응답들을 차단시키는 방법에 대해서 알아보겠습니다.



먼저 외부 IP 에 대해 허용한 규칙들을 제외한 나머지의 INPUT 들에 대해서 차단 하도록 다음과 같이 설정해 줍니다.

- INPUT Chain 생성 및 등록하기

  `FIREWALL_INPUT` 이라는 사용자 지정 Chain 을 생성하고 `INPUT` 기본 체인에 등록해 줍니다.

  ```shell
  # iptables -t filter -N FIREWALL_INPUT
  # iptables -t filter -I INPUT 1 -j FIREWALL_INPUT
  ```

  

- INPUT Drop Chain 생성 및 등록하기

  `FIREWALL_INPUT_DROP` 이라는 사용자 지정 Chain 을 생성하고 위에서 생성한 `FIREWALL_INPUT` Chain 에 등록해 줍니다.

  ```shell
  # iptables -t filter -N FIREWALL_INPUT_DROP# iptables -t filter -A FIREWALL_INPUT -j FIREWALL_INPUT_DROP
  ```

  

- 외부 IP `123.123.123.1` 에 대한 INPUT 요청 차단하기

  외부 IP 로 향하는 모든 Input 에 대해서 Drop 시킵니다.

  ```shell
  # iptables -t filter -A FIREWALL_INPUT_DROP -d 123.123.123.1 -j DROP
  ```

  

- `2222/tcp` 포트 허용하기

  지정한 외부 IP 와 외부 포트로 들어오는 연결에 대해 허용합니다.

  iptables 는 나중에 추가한 rule 이 가장 마지막에 확인됩니다.

  위에서 모든 Input 에 대해서 Drop 시켰기 때문에 허용한 포트를 먼저 확인하고 이외의 포트들을 차단해야 합니다.

  따라서 특정 포트의 Input 을 허용하는 rule 은 위에서 추가한 모든 Input 을 Drop 시키는 rule 보다 위에 있어야 합니다.

  `-I` 옵션으로 최상위 1번에 삽입 시켜 줍니다.

  ```shell
  # iptables -t filter FIREWALL_INPUT -I 1 -p tcp --dport 2222 -d 123.123.123.1 -j ACCEPT
  ```




지금까지 입력한 iptables 명령어들을 그림으로 간단히 표현해 보면 다음과 같습니다.

<p align="center">
<img src="/assets/images/docs/iptables 를 활용한 방화벽 구성하기/iptables_order.png"/><br>
</p>



지금까지 iptables 로 방화벽을 구성하는 방법에 대해서 알아봤습니다.

여기서는 대략적인 사용 방법들만 몇가지 설명 드렸지만 iptables 로 세부적인 정책들을 설정하면 막강한 방화벽을 구축 하실 수 있습니다.

각자의 원하는 방향대로 iptables 를 활용하여 막강한 방화벽을 구축해 보시기 바랍니다!

