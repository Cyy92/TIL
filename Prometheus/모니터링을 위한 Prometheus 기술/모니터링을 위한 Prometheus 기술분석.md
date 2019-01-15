# 모니터링을 위한 Prometheus 기술분석

Prometheus는 soundcloud에서 만든 open source 모니터링 도구로, 지표 수집을 통한 모니터링을 주요 기능으로 하고 있다. 

Kubernetes 모니터링 뿐만 아니라 애플리케이션이나 서버, OS 등 다양한 대상으로부터 지표를 수집하여 모니터링 할 수 있는 범용 솔루션으로, 아래와 같은 구조를 가지고 있다.

![그림1](https://github.com/Cyy92/TIL/blob/master/Prometheus/assets/Prometheus%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90.png?raw=true)

대부분의 모니터링 도구는 Push 방식, 즉 각 서버에 클라이언트를 설치하고 이 클라이언트가 __metrics__ 데이터를 수집해서 서버로 보내면 서버가 모니터링 상태를 보여주는 방식이다. 

> metrics : 측정 기준에 의해 발생하는 총 카운트로써, 숫자로 표기되는 단순 지표이며 대표적으로 하드웨어 및   	소프트웨어 사용량 등이 있다.



이에 반해 Prometheus는 pull 방식으로 서버가 각 클라이언트를 알고 있어야 하는게 아니라 서버에 클라이언트가 떠 있으면 서버가 주기적으로 클라이언트에 접속해서 데이터를 가져오는 방식이다. Prometheus는 크게 Prometheus server, Exporter, Grafana, Alertmanager, Pushgateway로 구성된다. 

- Exporter

  Exporter는 모니터링 대상의 metrics 데이터를 수집하고 prometheus가 접속했을 때 정보를 알려주는 역할을 한다. 호스트 서버의 CPU, Memory 등을 수집하는 node-exporter도 있고, nginx의 데이터를 수집하는 nginx-exporter도 있다. Exporter를 실행하면 데이터를 수집하는 동시에 http 엔드포인트를 열어서(default: 9100) prometheus 서버가 데이터를 수집해갈 수 있도록 한다. 즉, 웹 브라우저를 통해 해당 http 엔드포인트에 접속하면 metrics 데이터 정보를 확인할 수 있다. 

- Prometheus server

  Prometheus server는 exporter가 열어놓은 http 엔드포인트에 접속해서 metrics를 수집하고, grafana를 연동해서 대시보드 등의 시각화를 한다. 



- Grafana

  수집한 데이터 지표를 그래프로 나타내기 위한 시각화 도구이다. 



- Alertmanager

  알림을 받을 규칙을 만들어서 alertmanager로 보내면 alertmanager가 규칙에 따라 알림을 보낸다. 이를 통해  자원 할당량이 특정 기준을 넘었을 때에 대한 auto-scaling이 가능하다. 



- Pushgateway

  배치나 스케쥴 작업 같은 서비스는 필요한 경우에만 떠 있다가 작업이 끝나면 사라지는 경우가 있다. 이러한 서비스들에 대한 지표를 얻기 위해 push 방식으로 pushgateway에 지표를 전달하면 pushgateway가 지표를 저장하고 있다가 prometheus server가 pull 할 때, 저장된 지표를 반환한다. 