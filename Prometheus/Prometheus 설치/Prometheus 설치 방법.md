# Prometheus 설치 방법

다음과 같은 절차를 통해 prometheus를 설치할 수 있다. 



## 환경설정

Prometheus를 설치하기 위해 다음과 같은 환경을 설정한다. 



- 로그인 없이 prometheus 및 node exporter를 사용하기 위해 사용자를 생성한다.

  ```bash
  $ useradd --no-create-home --shell /usr/sbin/nologin prometheus
  $ useradd --no-create-home --shell /bin/false node_exporter
  ```



- Prometheus binary와 설정 파일을 저장하기 위한 디렉토리를 생성한다. 

  ```bash
  $ mkdir /etc/prometheus
  $ mkdir /var/lib/prometheus
  ```



- 생성한 디렉토리의 권한을 사용자에게 부여한다. 

  ```bash
  $ chown prometheus:prometheus /etc/prometheus
  $ chown prometheus:prometheus /var/lib/prometheus
  ```



## Prometheus server

Prometheus server는 다음의 [링크](https://prometheus.io/download/)에서 최신 release를 OS 별로 선택해서 다운로드 받을 수 있다. 다운로드 후 다음의 명령을 통해 설치를 진행한다. 

```bash
$ apt-get update && apt-get upgrade
$ wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.5.0.linux-amd64.tar.gz

$ tar xfz prometheus-2.5.0.linux-amd64.tar.gz
$ cd prometheus-2.5.0.linux-amd64
```



압축을 풀고 `prometheus-2.5.0.linux-amd64` 디렉토리 안에 생성된 prometheus, promtool binary 파일과 consoles, console_libraries 폴더를 다음과 같이 적절한 위치로 이동시킨다. 

```bash
$ cp ./prometheus /usr/local/bin/
$ cp ./promtool /usr/local/bin/

$ cp -r ./consoles /etc/prometheus
$ cp -r ./console_libraries /etc/prometheus

$ cd .. && rm -rf prometheus-2.5.0.linux-amd64
```



Prometheus server를 설치하기 위한 파일 및 디렉토리의 설정이 완료되었으면 모니터링을 하기 위한 설정 파일을 다음과 같이 작성한다. 아래의 설정 파일은 하나의 예시이며 사용자가 임의로 설정 항목들을 추가하여 작성할 수 있다. 

```bash
$ nano /etc/prometheus/prometheus.yml

>> 
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```



- 설정 파일의 필드 데이터

  | Field          | Description                                                  |
  | :------------- | :----------------------------------------------------------- |
  | global         | 필드는 metrics 수집을 위한 전역 설정 값 <br/>scrape_interval: Prometheus Server가 클라이언트에서 데이터를 가져오는 간격evaluation_interval: Prometheus Server가 규칙을 평가하는 빈도<br/>                                    이 빈도를 통해 새로운 시계열을 만들고 알림을 생성 |
  | rule_files     | Prometheus server가 로드할 규칙의 위치를 지정할 때 사용되는 필드 |
  | scrape_configs | 실제로 수집 대상을 지정하는 필드<br/>targets: Prometheus server가 접근해서 데이터를 가져올<br/>               exporter의 http 엔드포인트<br/>               Prometheus server는 targets/metrics에 접근해서 데이터를 pull |



위와 같이 설정 파일을 정의한 후, 다음과 같이 prometheus server를 실행할 수 있다.

```bash
$ prometheus --config.file=/etcc/prometheus/prometheus.yml
```



## Exporter

다음은 서버의 CPU, 메모리, 디스크 등의 상태를 모니터링 하는 node exporter에 대한 가이드이다. 먼저 다음의 [링크](https://prometheus.io/download/)에서 OS에 맞게 최신 버전의 node exporter 압축 파일의 링크주소를 복사한 후,  명령어를 통해 다운로드 받는다.

```bash
$ wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0-rc.0/node_exporter-0.17.0-rc.0.linux-amd64.tar.gz

$ tar xvf node_exporter-0.17.0-rc.0.linux-amd64.tar.gz
```



압축을 푼 후, `node_exporter-0.17.0-rc.0.linux-amd64` 디렉토리 안에 생성된 binary 파일을 적절한 위치로 이동시킨다. 

```bash
$ cp node_exporter-0.17.0-rc.0.linux-amd64/node_exporter /usr/local/bin
$ rm –rf node_exporter-0.17.0-rc.0.linux-amd64 node_exporter-0.17.0-rc.0.linux-amd64.tar.gz
```



서버가 시작되었을 때, node exporter를 자동으로 실행하기 위해서 다음과 같이 설정한다.

```bash
$ nano /etc/systemd/system/node_exporter.service

>>
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
$ systemctl daemon-reload
$ systemctl start node_exporter
$ systemctl status node_exporter
$ systemctl enable node_exporter
```



## Grafana

Ubuntu 기준 grafana의 설치 방법은 다음과 같다. 

```bash
$ wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.0.2_amd64.deb

$ apt-get install -y adduser libfontconfig
$ dpkg -i grafana_4.0.2_amd64.deb
$ vim /etc/apt/sources.list

>>
# Add the following line
deb https://packagecloud.io/grafana/stable/debian/ stretch main
deb https://packagecloud.io/grafana/testing/debian/ stretch main 

$ curl https://packagecloud.io/gpg.key | sudo apt-key add –
$ apt-get update
$ apt-get install grafana
$ systemctl daemon-reload
$ systemctl enable grafana-server.service
$ systemctl start grafana-server
```



### Get started

#### 1. Login

설치가 완료되면, `http://서버 IP:3000`으로 접속할 수 있고 다음과 같은 화면을 볼 수 있다. 기본 계정 admin / admin으로 로그인 한 후, 적절한 데이터소스를 지정하여 대시보드를 띄울 수 있다. 

![그림1](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/Grafana%20login%20page.png?raw=true)



#### 2. Create datasources

![그림2](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/create%20datasources%20in%20grafana.png?raw=true)



로그인 후, grafana 메인 페이지의 왼쪽 상단 아이콘을 클릭하면 그림과 같이 하위 메뉴가 보이고 Data Sources 항목을 클릭 후, Add data source를 통해 데이터소스를 생성할 수 있다. 

- Config 

  Name : 데이터소스 이름

  Type : Prometheus

  Url : http://서버 IP:9090



#### 3. Create dashboards

- New dashboards


![그림3](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/create%20new%20dashboards.png?raw=true)



대시보드는 새로 생성할 수도 있고, 기존의 만들어진 대시보드를 import할 수도 있다. 새로 생성하기 위해서는 그림과 같이 Dashboards -> New 항목을 선택 후 생성해 놓았던 데이터소스를 추가한다. 데이터소스를 추가하게 되면 Metric lookup에서 사용할 수 있는 metrics를 검색할 수 있고, prometheus query를 사용할 수도 있다. 또한, 대시보드 그래프에 표시될 값을 Legend format에 {{value}}로 지정할 수도 있다. 



> Note

Prometheus query는 다음의 [링크](https://prometheus.io/docs/prometheus/latest/querying/functions/)에서 확인할 수 있다. 



- Import dashboards



  ![그림4](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/import%20dashboards.png?raw=true)



[Grafana Labs](https://grafana.com/dashboards) 공식 홈페이지에서는 대시보드 templates를 제공하고 있는데, json 파일을 다운로드 받은 후, 그림과 같이 해당 파일을 업로드하여 대시보드를 import할 수도 있다. 



#### 4. Share dashboards



![그림5](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/share%20dashboards.png?raw=true)



반대로 만든 대시보드를 json 파일로 공유할 수도 있다. Share -> Export 항목을 선택하면 해당 대시보드를 json 파일로 저장하고 이를 공유할 수 있다. 



#### 5. Embed dashboards

특정 웹 페이지에 grafana dashboard를 embedding 하는 것 또한 가능하다. 이는 간단하게 링크를 html에 추가하기만 하면 되는데 대시보드의 링크를 추출하기 위한 방법은 다음과 같다. \

![그림6](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/embed%20grafana%20web.png?raw=true)

Share -> Link 항목을 선택하면 대시보드의 링크가 표시되고, html에 다음과 같이 추가하면 된다. 현재 시간까지의  모니터링 된 대시보드를 embedding 하고 싶다면 위 그리의 Current time range를 선택하면 된다. 

```bash
<iframe src = "http://10.0.0.100:3000/dashboard/db/new-dashboard"></iframe>
```



이와 같이 추가하게 되면, 또 다른 웹 페이지에서 grafana 웹 페이지를 실행할 수 있다. 페이지 위에 새로운 페이지가 추가된 개념으로써, 대시보드를 생성하고, 데이터소스를 생성하는 등 유사한 동작을 똑같이 실행할 수 있다. 



혹은, 패널 별로, 즉 대시보드 그래프 별로 embedding 하는 것 또한 가능하다. 

![그림7](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/embed%20dashboards%20panel.png?raw=true)

Panel Title(설정한 패널 이름)을 클릭하고 Share -> Embed 항목을 누르면 위와 비슷한 유형의 링크가 표시된다.  마찬가지로 이 링크를 html에 추가하기만 하면 되는데 웹에 표시될 패널의 크기를 width와 height로 자유롭게 설정할 수도 있다. 