---
title: HCloud-Classic 관리자 포탈 사용 설명서
subtitle: 
layout: post
icon: fa-book
order: 3

---

* TOC
{:toc}




## 1. 로그인 화면

![1](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/1.png)



HCloud-Classic 포탈에 최초 접속시 보이는 화면 입니다.

사용자의 아이디와 비밀번호를 입력하시면 포탈 이용이 가능합니다.



## 2. 리소스 화면

![2](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/2.png)



HCloud-Classic 포탈에 로그인하면 처음에 보이는 화면입니다.

HCloud-Classic 의 전체 사용 가능한 자원중 얼만큼 사용 중인지 한눈에 알 수 있습니다.

CPU, Memory, Storage, Node 사용량을 각각 확인 하실 수 있습니다.



## 3. 노드 목록

![3](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/3-16377308280931.png)



리소스 화면 하단에는 현재 이용중인 그룹에 할당된 노드들의 목록을 확인 하실 수 있습니다.

각 노드의 전원 상태와 스펙을 확인 하실 수 있습니다.

노드 목록에서 하나의 노드를 선택하시면 하단의 Info 탭에서 해당 노드의 상세 스펙을 확인 하실 수 있습니다.



## 4. 관리자 전용 관리 메뉴

### 1. 사용자 관리

![image-20211124142347649](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124142347649.png)



사용자 관리 화면입니다.

HCloud-Classic 을 이용하기 위한 사용자를 생성, 편집 삭제 할 수 있습니다.

하나의 사용자를 선택하면 하단의 Info 탭에서 선택된 사용자의 상세 정보를 확인 하실 수 있습니다.



#### 1. 사용자 생성

![image-20211124143229610](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124143229610.png)

사용자 관리 화면에서 상단의 CREATE 버튼을 클릭하면 사용자를 생성할 수 있는 팝업이 보여집니다.

- ID : 로그인시 사용할 ID 를 입력합니다.
- Auth : 사용자의 권한을 선택합니다. 권한에 따라 접근할 수 있는 메뉴가 달라집니다.
  - Admin : HCloud-Classic 의 모든 메뉴에 접근이 가능합니다.
    - RESOURCE
    - SERVER
    - NETWORK
      - SUBNET
      - ADAPTIVE IP
    - MANAGEMENT
      - USER
      - QUOTA
      - BILLING
      - TIMPANI
  - User : SERVER 와 NETWORK 메뉴에만 접근이 가능합니다.

- E-MAIL : 사용자의 이메일 주소를 입력합니다.
- Password : 로그인시 사용할 암호를 입력합니다. 비밀번호는 7자이상 특수문자, 알파벳 대소문자, 숫자가 포함되어야합니다.
- Password Confirm : 암호를 제대로 입력하였는지 확인하기 위해 한번더 입력해 줍니다.



#### 2. 사용자 편집

![image-20211124144501601](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124144501601.png)

사용자 관리 화면에서 상단의 EDIT 버튼을 클릭하면 사용자를 편집할 수 있는 팝업이 보여집니다.

변경할 항목을 수정하여 입력하고, Password 항목에 기존에 사용하던 암호 또는 변경할 암호를 입력합니다. 

Password Confirm 에 Password 항목에서 입력한 암호를 제대로 입력하였는지 확인하기 위해 한번 더 입력해 주고 하단의 Edit 버튼을 눌러 사용자 편집을 완료합니다.



#### 3. 사용자 삭제

![image-20211124144833664](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124144833664.png)

사용자 관리 화면에 나타난 사용자 목록에서 하나의 사용자를 선택한 후 상단의 DELETE 버튼을 누르면 단일 사용자를 삭제 할 수 있습니다.



![image-20211124144901710](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124144901710.png)

사용자 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 사용자를 삭제 하실 수 있습니다.



![image-20211124145019229](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124145019229.png)

좌측의 체크박스를 이용하여 다중 사용자를 삭제 하실 수 도 있습니다.



![image-20211124145040406](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211124145040406.png)

삭제할 사용자들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.



#### 4. 사용자 상세 정보 조회

![image-20211125174942660](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125174942660.png)

사용자 목록에서 사용자를 선택하면 하단의 Info 탭에서 선택한 사용자에 대한 상세 정보를 확인 할 수 있습니다.



- ID : 로그인시 사용하는 ID 입니다.
- Name : 사용자에 설정된 이름입니다.
- E-Mail : 사용자의 이메일 주소입니다.
- Last Login : 사용자가 마지막으로 로그인한 시간입니다.
- Group ID : 사용자가 속해있는 그룹의 ID 입니다.
- Group Name : 사용자가 속해있는 그룹의 이름입니다.
- Authentication : 사용자의 권한을 나타냅니다. 권한에 따라 접근할 수 있는 메뉴가 달라집니다.
  - admin : HCloud-Classic 의 모든 메뉴에 접근이 가능합니다.
    - RESOURCE
    - SERVER
    - NETWORK
      - SUBNET
      - ADAPTIVE IP
    - MANAGEMENT
      - USER
      - QUOTA
      - BILLING
      - TIMPANI
  - user : SERVER 와 NETWORK 메뉴에만 접근이 가능합니다.
- Register Date : 사용자가 등록된 시간입니다.



### 2. 쿼터 관리

![image-20211125141856537](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125141856537.png)

HCloud-Classic 을 이용하기 위해 이용가능한 자원을 제한할 수 있는 메뉴입니다.

master 유저의 경우 전체 그룹에 대해서 쿼터를 설정 할 수 있으며, Admin 권한을 가진 유저의 경우 Admin 이 속한 그룹의 쿼터에 대해서만 설정이 가능합니다.



#### 1. 쿼터 생성

![image-20211125165918713](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125165918713.png)

쿼터 관리 화면에서 상단의 CREATE 버튼을 클릭하면 쿼터를 생성할 수 있는 팝업이 보여집니다.

- Group : 쿼터를 생성할 그룹을 선택합니다. master 유저인 경우에만 표시되는 항목입니다.
- Storage Pool : 디스크를 할당할 Storage 노드의 Pool 을 선택합니다.
- SSD Size : 그룹에서 이용할 수 있는 총 SSD 용량을 지정해 줍니다.
- HDD Size : 그룹에서 이용할 수 있는 총 HDD 용량을 지정해 줍니다.
- Subnets : 그룹에서 이용할 수 있는 총 서브넷 개수를 지정해 줍니다. 하나의 단일 가상 서버는 하나의 서브넷을 사용합니다.
- AdaptiveIPs : 단일 가상 서버에 부여할 수 있는 외부 IP의 갯수를 지정해 줍니다.
- Nodes : 그룹에서 이용할 수 있는 총 노드의 개수를 지정해 줍니다.



#### 2. 쿼터 편집

![image-20211125171928835](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125171928835.png)



#### 3. 쿼터 삭제

![image-20211125172024442](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125172024442.png)

쿼터 관리 화면에 나타난 쿼터 목록에서 하나의 쿼터를 선택한 후 상단의 DELETE 버튼을 누르면 단일 쿼터를 삭제 할 수 있습니다.



![image-20211125174216256](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125174216256.png)

쿼터 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 쿼터를 삭제 하실 수 있습니다.



#### 4. 쿼터 상세 정보 조회

![image-20211125174256322](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125174256322.png)

쿼터 목록에서 쿼터를 선택하면 하단의 Info 탭에서 선택한 쿼터에 대한 상세 정보를 확인 할 수 있습니다.

선택된 쿼터의 그룹이 총 이용가능한 쿼터와 리소스가 얼만큼 이용중인지 확인 하실 수 있습니다.



### 3. 요금 조회

![image-20211125142225649](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125142225649.png)

HCloud-Classic 을 이용하면서 발생한 요금을 확인하실 수 있습니다.

master 유저는 전체 그룹에 대해서 조회가 가능하고 admin 권한을 가진 사용자는 현재 그룹에 대해서 조회가 가능합니다.

일별, 월별, 연도별로 조회가 가능하고 조회 기간을 설정하여 조회가 가능합니다.



![image-20211125142205686](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125142205686.png)

요금 조회 목록에서 특정 항목을 선택하시면 하단의 Info 탭을 통해 해당 기간에 부관된 요금에 대해서 각 자원들의 이용 요금을 확인 하실 수 있습니다.



## 4. 네트워크 관리

### 1. 서브넷 관리

![image-20211125115440072](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125115440072.png)

HCloud-Classic 에서 이용할 단일 가상 서버를 위해 사용할 서브넷을 생성, 편집, 삭제 할 수 있는 화면입니다.

각 단일 가상 서버는 별개의 서브넷을 사용합니다.

단일 가상 서버내에 구성된 노드들 중 Leader 노드는 x.x.x.1 과 같이 마지막 자리가 1로 된 IP 주소를 부여 받고 Compute 노드는 x.x.x.2 부터 시작하여 차례로 IP를 할당 받게 됩니다.



#### 1. 서브넷 생성

![image-20211125125122816](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125125122816.png)

서브넷 관리 화면에서 상단의 CREATE 버튼을 클릭하면 서브넷을 생성할 수 있는 팝업이 보여집니다.

- Subnet Name : 생성할 서브넷에 부여할 이름을 입력합니다.
- Domain Prefix : DHCP 설정시 해당 서브넷에 부여할 DNS 접미사를 지정합니다. 도메인 접미사가 필요하지 않은 경우 공란으로 두고 Next 를 클릭하시면 됩니다.
  - DNS 서버로 부터 쿼리 요청이 예를 들어 Domain Name 이 innogrid.com 으로 되어있고 노드 내에서 ping xyz 명령을 실행할 경우 xyz.innogrid.com 에 대한 DNS 를 쿼리한 후 ping 요청을 날리게 됩니다. (xyz 와 같이 . (점) 이 포함 되지 않은 DNS 조회에 대해서 뒤에 접미사 (innogrid.com) 를 붙이게 됩니다.)



![image-20211125113953448](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125113953448.png)

- Guest OS : 단일 가상 서버 생성시 사용하게 될 OS 를 선택합니다. 단일 가상 서버를 생성할때 현재 생성한 서브넷을 선택하게 되면 단일 가상 서버는 해당 OS 로 구성된 디스크와 Kernel, 램디스크 을 사용하게 됩니다.
- DNS IP Address : DNS 서버의 IP 주소를 입력합니다.
- Next Server IP : PXE Boot 를 위한 tftp 서버의 IP 주소를 입력합니다.



![image-20211125114511625](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125114511625.png)

- Network Address : 서브넷의 네트워크 주소를 입력합니다.
- Netmask : 서브넷의 넷마스크를 입력합니다.
- Gateway Address : 생성하는 서브넷의 게이트웨이 주소로 사용할 IP 주소를 입력합니다. 생성하는 서브넷의 네트워크 대역에 포함 되어야 하며, Master Node 에 노드들과 통신하기 위해서 내부적으로 부여되는 주소입니다.



#### 2. 서브넷 편집

![image-20211125115929177](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125115929177.png)

서브넷 목록에서 편집할 서브넷을 선택한 후 상단의 EDIT 버튼을 누르면 서브넷을 편집하실 수 있습니다.



![image-20211125120021048](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125120021048.png)

서브넷을 생성할때와 동일하게 진행해 주시면 됩니다.



#### 3. 서브넷 삭제

![image-20211125123544223](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125123544223.png)

서브넷 관리 화면에 나타난 서브넷 목록에서 하나의 서브넷을 선택한 후 상단의 DELETE 버튼을 누르면 단일 서브넷을 삭제 할 수 있습니다.



![image-20211125123619227](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125123619227.png)

서브넷 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 서브넷을 삭제 하실 수 있습니다.



![image-20211125123752992](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125123752992.png)

좌측의 체크박스를 이용하여 다중 서브넷을 삭제 하실 수 도 있습니다.



![image-20211125123815322](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125123815322.png)

삭제할 서브넷들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.



#### 4. 서브넷 상세 정보 조회

![image-20211125175237945](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125175237945.png)

서브넷 목록에서 서브넷을 선택하면 하단의 Info 탭에서 선택한 서브넷에 대한 상세 정보를 확인 할 수 있습니다.



- UUID : 생성한 서브넷에 부여된 UUID 입니다.
- Subnet Name :  생성한 서브넷에 지정한 이름입니다.
- State : 단일 가상 서버에서 사용중인 서브넷인지 표시합니다. 단일 가상 서버에서 사용중인 경우 `in use`, 사용중이지 않은 경우 `not in use`로 표시됩니다.
- Network IP : 생성한 서브넷의 네트워크 주소입니다.
- PXE Boot IP : 단일 가상 서버의 Leader 노드 IP 주소입니다.
- Server
  - UUID : 해당 서브넷을 사용하고 있는 단일 가상 서버에 부여된 UUID 입니다.
  - Name : 해당 서브넷을 사용하고 있는 단일 가상 서버에 지정한 이름 입니다.
- Group : 해당 서브넷이 생성된 그룹의 이름입니다.
- OS : 서브넷을 생성시 선택한 운영체제의 종류입니다.
- Created At : 서브넷을 생성한 시간입니다.
- DNS : 해당 서브넷에 구성된 노드들이 사용할 DNS 주소 입니다.
- Gateway : 해당 서브넷에 구성된 노드들이 사용할 Master 노드에 할당되는 Gateway 주소입니다.
- Leader
  - UUID : 단일 가상 서버에서 해당 서브넷을 사용중인 경우, 단일 가상 서버의 Leader 노드의 UUID 가 표시됩니다.
  - Name : 단일 가상 서버에서 해당 서브넷을 사용중인 경우, 단일 가상 서버의 Leader 노드에 지정한 이름이 표시됩니다.



### 2. Adaptive IP 관리

![image-20211125175201117](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125175201117.png)

HCloud-Classic 에서 이용중인 단일 가상 서버의 내부 IP에 외부 IP를 연결하여 외부에서 단일 가상 서버에 접속이 가능하도록 해주는 기능입니다.

포트포워딩을 통해 단일 가상 서버에서 사용중인 포트를 외부 포트에 연결하여 이용하실 수 있습니다.

Adaptive IP 를 할당하면 기본적으로 외부로 부터 단일 가상 서버로 향하는 Ping 은 허용되고 그외 모든 포트들의 INPUT 에 대해서는 차단 하도록 되어있습니다.



#### 1. Adaptive IP 생성

![image-20211125124546428](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125124546428.png)

Adaptive IP 관리 화면에서 상단의 CREATE 버튼을 누르면 단일 가상 서버에 외부 IP를 할당할 수 있습니다.

- External Interface IP : 단일 가상 서버에 부여할 외부 IP를 선택해 줍니다. 선택한 외부 IP는 Master 노드의 External Interface에 할당이 됩니다. 현재 다른 단일 가상 서버에서 이용중이거나 IP 충돌이 감지된 외부 IP들에 대해서는 목록에 나타나지 않습니다.

- Server UUID : 생성한 단일 가상 서버의 목록 중에서 외부 IP를 부여할 단일 가상 서버를 선택합니다.



#### 2. Adaptive IP 삭제

![image-20211125143823596](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125143823596.png)

Adaptive IP 관리 화면에 나타난 Adaptive IP 목록에서 하나의 Adaptive IP를 선택한 후 상단의 DELETE 버튼을 누르면 단일 Adaptive IP를 삭제 할 수 있습니다.



![image-20211125143927249](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125143927249.png)

Adaptive IP 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 Adaptive IP를 삭제 하실 수 있습니다.



![image-20211125144042684](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125144042684.png)

좌측의 체크박스를 이용하여 다중 Adaptive IP를 삭제 하실 수 도 있습니다.



![image-20211125144106572](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125144106572.png)

삭제할 Adaptive IP들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.



#### 3. Port Forwarding 추가

![image-20211125145142989](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125145142989.png)

상단의 Adaptive IP 목록에서 Port Forwarding 을 할당할 Adaptive IP 를 선택하고 하단의 Port Forwarding 탭의 CREATE 버튼을 클릭합니다.



![image-20211125145227924](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125145227924.png)

Port Forwarding 을 생성하는 팝업이 나타나면 필요한 정보들을 입력한 후 하단의 CREATE 을 버튼을 클릭합니다.

- Protocol
  - TCP : TCP 포트에 대해서 Port Forwarding 을 설정합니다.
  - UDP : UDP 포트에 대해서 Port Forwarding 을 설정합니다.
  - TCP + UDP : TCP 와 UDP 포트 모두에 대해서 Port Forwarding 을 설정합니다.

- External Port : Port Forwarding 에 사용할 외부 포트 번호를 입력합니다. 외부에서 접속시 외부 IP 와 외부 포트 를 통해 접속이 가능합니다.
- Internal Port : 단일 가상 서버 내부에서 사용되는 포트 번호를 입력합니다.
- Description : 생성할 Port Forwarding에 대한 설명을 입력합니다.



#### 4. Port Forwarding 삭제

![image-20211125145442795](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125145442795.png)

Adaptive IP 관리 화면에 나타난 Adaptive IP 목록에서 하나의 Adaptive IP를 선택합니다. 하단의 Port Forwarding 목록에서 삭제하고자 하는 포트를 선택한 후 Port Forwarding 란의 DELETE 버튼을 누르면 단일  Port Forwarding 을 삭제 할 수 있습니다.



![image-20211125160137356](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160137356.png)

Port Forwarding 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 Port Forwarding 를 삭제 하실 수 있습니다.



![image-20211125160252994](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160252994.png)

좌측의 체크박스를 이용하여 다중 Port Forwarding을 삭제 하실 수 도 있습니다.



![image-20211125160316427](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160316427.png)

삭제할 Port Forwarding들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.



## 5. 서버 관리

### 1. 서버 생성

![1](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/1-16378251184812.png)

서버 관리 화면에서 상단의 CREATE 버튼을 클릭하면 서버를 생성할 수 있는 팝업이 보여집니다.

- Server Name : 생성할 단일 가상 서버에 부여할 이름을 입력합니다.
- Description : 생성할 단일 가상 서버에 대한 설명을 입력합니다.



![2](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/2-16378251184813.png)

- Guest OS : 생성할 단일 가상 서버에서 사용할 운영체제를 선택합니다. 지원되는 운영체제는 다음과 같습니다.
  - CentOS 6
  - Devuan ASCII
  - Debian 9
  - LMDE 3
  - antiX 17.4.1
-  CPU : 단일 가상 서버에서 사용할 CPU 코어 수를 입력합니다.
- Memory : 단일 가상 서버에서 사용할 메모리의 크기를 입력합니다.



![3](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/3-16378251184811.png)

- Node Number : 단일 가상 서버를 구성하는 노드의 개수를 입력합니다.
- OS Volume : 단일 가상 서버를 생성할때 운영체제를 설치할 디스크와 사용자의 데이터를 저장할 디스크가 별도로 구성됩니다. 운영체제가 설치될 디스크의 용량을 지정합니다.
- Data Volume : 사용자의 데이터를 저장할 디스크의 용량을 지정합니다.



![4](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/4.png)

- Subnet : 단일 가상 서버에서 사용할 서브넷을 선택합니다. 서브넷 관리 메뉴에서 생성한 서브넷의 목록중 서버에 할당되지 않은 서브넷들의 목록이 보여집니다.



마지막으로 CREATE 버튼을 클릭하면 단일 가상 서버의 생성이 진행됩니다.



![image-20211125163840877](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125163840877.png)

생성 과정은 서버 목록에서 생성 중인 서버를 선택 한 후 하단의 Log 탭을 통하여 확인 하실 수 있습니다.



### 2. 서버 접속

#### 1. VNC 를 통한 접속

![image-20211125152455106](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125152455106.png)

서버 목록에서 접속하고자 하는 Running 상태의 서버 우측의 >_ 아이콘을 클릭하시면 VNC 접속화면이 팝업으로 나타납니다.



![17](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/17.png)

GUI 를 통해서 단일 가상 서버를 제어하실 수 있습니다.



#### 2. SSH 를 통한 접속

![image-20211125153018004](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125153018004.png)

Adaptive IP 관리 메뉴에서 생성한 단일 가상 서버에 Adaptive IP 를 생성하여, 외부 IP를 할당해 주고 Port Forwarding 항목에 SSH (TCP/22) 를 추가하시면 외부 IP(Public IP) 와 외부 포트(External Port)를 통해 단일 가상 서버에 접속하실 수 있습니다.



### 3. 서버 삭제

![image-20211125160642720](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160642720.png)

서버 관리 화면에 나타난 서버 목록에서 하나의 서버를 선택한 후 상단의 DELETE 버튼을 누르면 단일 서버를 삭제 할 수 있습니다.



![image-20211125180437509](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125180437509.png)

서버 삭제 팝업이 나타나면 하단의 DELETE 버튼을 눌러서 서버를 삭제 하실 수 있습니다.



![image-20211125180553526](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125180553526.png)

좌측의 체크박스를 이용하여 다중 서브넷을 삭제 하실 수 도 있습니다.



![image-20211125180609347](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125180609347.png)

삭제할 서브넷들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.



### 4. 서버 상세 정보 조회

서버 목록에서 서버를 선택하여 서버의 상세 정보를 확인 하실 수 있습니다.



![image-20211125160824312](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160824312.png)

Info 탭에서는 생성한 서버에 대한 기본적인 정보와 단일 가상 서버를 구성하고 있는 자원의 정보를 확인 하실 수 있습니다.



#### 1. 로그 조회

![image-20211125160918141](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125160918141.png)

Log 탭에서는 단일 가상 서버가 생성중일 때의 로그, 단일 가상 서버에 대한 추가 적인 작업 (Adaptive IP, Volume 관련 작업들)에 대한 로그를 확인 하실 수 있습니다.

실패한 작업에 대해서는 Result 에 Failed 라고 표시되며, 실패한 작업을 선택하시면 상세 로그를 확인 하실 수 있습니다.



#### 2. 서버의 노드 조회

![image-20211125161030780](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161030780.png)

Nodes 탭에서는 생성한 단일 가상 서버를 구성하고 있는 노드들에 정보를 확인 하실 수 있습니다.



##### 2.1 서버의 노드 편집

![image-20211125161122760](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161122760.png)

Nodes 탭에서 Edit 버튼을 클릭하시면 단일 가상 서버를 구성하고 있는 노드들을 수정 할 수 있습니다.



![image-20211125161210498](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161210498.png)

Node 를 선택할 수 있는 팝업이 위와 같이 나타납니다.

죄측에는 사용가능한 노드들이 표시되고, 우측에는 선택된 단일 가상 서버를 구성 하고 있는 노드들이 표시됩니다.

현재 예시는 로그인한 사용자가 총 4개의 노드를 사용할 수 있으며, 선택된 단일 가상 서버가 4개의 노드로 구성된 예시입니다.



![image-20211125161228682](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161228682.png)

현재 예시는 단일 가상 서버를 구성하고 있는 4개의 노드 중에서 Node1, Node3 2개의 노드를 제외시키기 위해 선택한 모습입니다.

제외할 노드들을 선택하고 ◀ 버튼을 클릭하면 좌측으로 옮겨지고 우측의 목록에서 제외됩니다.



![image-20211125161244826](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161244826.png)

Node1, Node3 2개의 노드를 제외한 모습입니다.

하단의 Edit 버튼을 클릭하면 단일 가상 서버의 전원이 종료되고 2개의 노드로 구성된 후 다시 부팅하여 클러스터링을 진행합니다.



#### 3. 서버 모니터링

Monitoring 탭에서는 생성한 단일 가상 서버의 상태를 모니터링 하실 수 있습니다.

서버 목록에서 모니터링 하고자 하는 단일 가상 서버를 선택하고 하단의 Monitoring 탭을 클릭하여 확인할 수 있습니다.

각 항목들은 1초 간격으로 갱신됩니다.



![monitoring](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/monitoring.png)

- CPU : 현재 CPU 사용량이 표시됩니다.
- Memory : 현재 Memory 사용량이 표시됩니다.
- Disk : 마운트된 디스크들의 총 용량과 사용용량, 사용가능한 용량이 표시됩니다.



![image-20211125162011852](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125162011852.png)

- Disk I/O : 디스크 읽기/쓰기 사용량이 표시됩니다.
- Network : 외부 네트워크 사용량이 표시됩니다.



![image-20211125161939017](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125161939017.png)

- Latency : 단일 가상 서버를 구성하고 있는 노드들 간의 지연 시간이 표시됩니다.



![image01](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image01.png)

- Process : 단일 가상 서버에서 실행되고 있는 프로세스 목록을 확인 할 수 있습니다. 각 프로세스가 어떤 노드에서 동작중인지 확인 할 수 있습니다.



#### 4. 서버 볼륨 조회/생성/삭제

![image-20211125180043576](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125180043576.png)

단일 가상 서버에서 사용되는 디스크의 목록이 표시됩니다.

상단의 CREATE 와 DELETE 버튼을 통해 추가적인 데이터 디스크를 생성하거나 삭제할 수 있습니다.



#### 5. 백업/복구

![image-20211125162116791](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125162116791.png)

단일 가상 서버의 디스크들을 백업하거나 복구 할 수 있습니다.

상단의 CREATE 와 DELETE 버튼을 통해 백업을 생성하거나 삭제 할 수 있습니다.



## 6. 알람 조회

### 1. 실시간 알람 표시

![image-20211125142746159](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125142746159.png)

단일 가상 서버의 상태에 대한 알람을 실시간으로 표시해 주는 기능입니다.

새로운 알림이 발생한 경우 HCloud-Classic 좌측의 로그인한 사용자의 ID 표시되는 영역의 우측에 빨간색 원으로 표시가 됩니다.

로그인한 사용자의 ID 표시되는 영역에 커서를 올리면 하위 메뉴중 Alarm 이라는 항목 우측에 새로운 알람의 개수가 표시됩니다.



Alarm 메뉴를 누르시면 해당 사용자의 ID에 할당된 단일 가상 서버들에 대한 알람 목록을 조회하실 수 있습니다.



![image-20211125150105221](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125150105221.png)



### 2. 알람 상세 정보 조회

알람 목록에서 Detail 의 전체 내용을 확인하고자 하는 경우 해당 알람을 클릭하여 확인 하실 수 있습니다.

![image-20211125150619514](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125150619514.png)



### 3. 알람 삭제

![image-20211125152147539](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125152147539.png)

좌측의 체크박스를 이용하여 알람을 삭제 하실 수 있습니다.



![image-20211125152208081](/assets/images/manual/2021-11-25-manual_HCloud-Classic 관리자 포탈 사용 설명서/image-20211125152208081.png)

삭제할 알람들이 맞는 지 확인 후, 하단의 DELETE 버튼을 눌러 삭제하실 수 있습니다.

