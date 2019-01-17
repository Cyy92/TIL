# Install CentOS 7



## Prerequisites

- __설치 부팅 디스크 만들기__

  로컬 PC에 설치하는 경우, 설치 CD 또는 USB를 이용하여 설치 마법사를 실행해야 한다.

  설치 부팅 디스크는 다음의 [링크](https://www.balena.io/etcher/)에서 Etcher 프로그램을 다운로드 받아 USB 또는 SD 카드에 CentOS 7 이미지를 구워 만들 수 있다.

  ![그림1](https://github.com/Cyy92/user-images/blob/master/Install%20OS/Etcher.png?raw=true)

- CentOS 7 이미지 다운로드

  CentOS 7 이미지는 다음의 [링크](http://mirror.kakao.com/centos/7/isos/x86_64/)에서 다운로드 받을 수 있다. 



## Install OS

이미지 굽기를 완료하면, 설치 부팅 디스크로 컴퓨터를 부팅하여 설치를 진행한다. 



### 설치 순서

---

#### 1. Get started

- `Install CentOS 7` 선택한다.

![그림2](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(1).png?raw=true) {:.aligncenter}



#### 2. Select language

- (선택)원하는 언어 선택 후 `Continue` 를 눌러서 설치를 진행한다. 

![그림3](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(2).png?raw=true) {:.aligncenter}



#### 3. Installation summary

- 사용할 S/W, 네트워크, 파티셔닝 등등의 설정을 거쳐 설치가 시작된다. 
- 모든 설정이 끝나면, `Begin Installation`을 눌러 본격적으로 설치를 시작한다. 

![그림4](https://github.com/Cyy92/user-images/blob/master/Install%20OS/new%20setup%20centos7%20(3).PNG?raw=true) {:.aligncenter}



#### 4. Date & Time

- 지역 및 날짜, 시간 설정 

![그림6](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(5).PNG?raw=true) {:.aligncenter}



#### 5. Software selection

- S/W 환경을 선택할 수 있다.  
- (선택)편의성을 위해 윈도우 환경인 `GNOME Desktop` 을 선택한다.
- (선택)추가로 설치할 플러그인 선택 후 `Done` 을 누른다. 

![그림5](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(4).png?raw=true) {:.aligncenter}



#### 6. Installation destination

- 효율적인 디스크 관리를 위한 디스크 파티셔닝
- `I will configure partitioning` 선택 후 `Done`을 누른다. 

![그림8](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(7).png?raw=true) {:.aligncenter}



- `Done`을 누르면 다음과 같은 화면이 표시되고, 임의로 파티션 별 용량을 설정할 수 있다. (서버 기준)
- 파티션닝을 완료하면 마찬가지로 `Done`을 누른다. 

![그림7](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(6).png?raw=true) {:.aligncenter}

| Partition | Description                                                  | Recommended size |
| --------- | ------------------------------------------------------------ | ---------------- |
| /boot     | 부팅에 필요한 커널, 모듈 등이 존재하는 디렉토리              | 최소 500 MB      |
| /         | 특수 목적으로 분할된 파티션을 제외하고 남은 공간이 <br/>할당되는 파티션 | 10 GB            |
| swap      | 시스템에서 메모리 용량이 부족하거나 오래 사용하지 않은 <br/>메모리에 로드된 프로그램을 저장하기 위해 사용 | 1 GB             |
| /home     | 사용자 데이터를 저장하기 위한 공간                           | 1 GB             |
| /backup   | 백업 데이터가 저장되는 공간<br/>별도의 디스크 사용 권장      |                  |



#### 7. User settings

- root 계정으로 로그인 할 경우의 비밀번호 & 사용자 계정 생성

![그림8](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(8).png?raw=true) {:.aligncenter}



![그림9](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(10).png?raw=true) {:.aligncenter}



#### 8. Initial setup

- `LICENSING` 선택해서 CentOS 7 라이센스 계약을 확인한 후, `I accept the license agreement.`를 누른다.

![그림10](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(11).png?raw=true) {:.aligncenter}

#### 9. Start CentOS 7

- 설치 과정 중 설정한 계정으로 로그인 한 뒤, CentOS 7을 시작한다.
- root 계정으로 로그인하고자 하는 경우, `Not listed?`를 선택 후 Username에 root를, Password에 설정한 비밀번호를 입력한다. 

- 사용자 계정으로 로그인하고자 하는 경우, 생성한 사용자 계정을 선택 후 시작한다.

![그림11](https://github.com/Cyy92/user-images/blob/master/Install%20OS/setup%20centos7%20(12).png?raw=true) {:.aligncenter}



#### 10. Setting network configuration

- CentOS 7 설치가 끝나면 터미널을 실행해서 추가로 네트워크 설정이 필요하다. 

- 네트워크 설정은 고정 IP를 사용하기 위함이다.

  #### 10 - 1) 네트워크 정보 확인

  - 다음의 명령어를 통해 네트워크 디바이스 명 등의 정보를 확인한다.

    ```bash
    $ ifconfig
    ```


  #### 10 - 2) 설정 파일 변경

  - 시스템의 IP를 변경하기 위해서 모든 이더넷 설정파일들이 존재하는 디렉토리로 이동한다.

    ```bash
    $ cd /etc/sysconfig/network-scripts/
    ```

  - 위에서 확인한 디바이스의 설정파일을 수정한다.  `IPADDR`, `PREFIX`, `GATEWAY`, `DNS1`, `NETMASK`, `HWADDR`에 값을 추가하면 된다. 

    ```bash
    $ vi ifcfg-ens33
    
    >>
    TYPE=Ethernet
    BOOTPROTO=none
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_FAILURE_FATAL=no
    IPV6_ADDR_GEN_MODE=stable-privacy
    NAME=ens33
    UUID=4cb9f880-a738-4ff2-bc1a-142c41c68516
    DEVICE=ens33
    ONBOOT=yes
    IPADDR=# your ip address
    PREFIX=# 16
    GATEWAY=# your gateway address
    DNS1=# your dns address
    NETMASK=# your netmask address
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_PRIVACY=no
    HWADDR= # 00:0c:29:d3:4b:cc (your mac address)
    ```

    이더넷 MAC 주소는 다음과 같이 확인할 수 있다. 

    ```bash
    $ ip link show '네트워크 디바이스 이름'
    ```

    ![그림12](https://github.com/Cyy92/user-images/blob/master/Install%20OS/mac%20addr.png?raw=true) {:.aligncenter}


  #### 10 - 3) 네트워크 재시작

  - 다음의 명령어를 통해 네트워크를 재시작해서 변경사항이 적용되게끔 한다.

    ```bash
    $ systemctl restart network
    or
    $ service network restart
    ```

  - 마지막으로 변경사항이 적용되었는지 확인한다.

    ```bash
    $ ifconfig
    ```

