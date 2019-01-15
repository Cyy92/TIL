# Kubernetes 연동

다음은 __Serverless__ 환경에서 Kubernetes container 안에 prometheus 및 grafana를 배포하기 위한 방법에 관련된 가이드이다. 여러 절차를 거쳐서 배포가 되는데 그 절차는 다음과 같다.

> Note

Serverless = Micro service



- Namespace 생성 

  Namespace란 동일한 물리적 클러스터에 의해 지원되는 가상 클러스터인데, 많은 사용자가 여러 팀 또는 프로젝트에 분산되어있는 환경에서 사용하기 위한 것으로 리소스 할당량을 통해 여러 사용자 간 클러스터 리소스를 나누는 방법이다. 

  이를 생성하기 위해 다음과 같이 설정 파일을 작성한다. 

  ```bash
  $ vim namespaces.yml
  
  >>
  apiVersion: v1
  kind: Namespace
  metadata:
    name: serverless
  ---
  apiVersion: v1
  kind: Namespace
  metadata:
    name: serverless-fn
  ```

  설정 파일 작성을 완료하면 다음의 명령어를 통해 namespace를 생성한다. 

  ```bash
  $ kubectl apply -f ./namespaces.yml
  
  >>
  root@ubuntu:~# kubectl apply -f ./namespaces.yml
  namespace/serverless created
  namespace/serverless-fn created
  ```

  Namespace가 제대로 생성되었는지 다음의 명령어를 통해 확인한다. 

  ```bash
  $ kubectl get namespaces
  
  >> 
  root@ubuntu:~# kubectl get namespaces
  NAME             STATUS   AGE
  default          Active   13s
  kube-public      Active   13s
  kube-system      Active   13s
  serverless       Active   13s
  serverless-fn    Active   13s
  ```

  Kubernetes는 기본적으로 default, kube-public, kube-system이라는 namespace를 지원하는데 각각에 대한 설명은 다음과 같다. 

  | Namespace   | Description                                                  |
  | ----------- | ------------------------------------------------------------ |
  | default     | namespace가 지정되지 않은 개체의 기본 namespace              |
  | kube-public | Kubernetes 시스템에 의해 생성된 객체의 namespace             |
  | kube-system | 자동으로 만들어지는 namespace, <br/>일부 자원을 전체 클러스터에서 공개적으로 표시하고 <br/>읽을 수 있어야 할 경우를 대비하여 예약되어 있다. |



- Prometheus 배포

  Kubernetes container 내부에 배포하는 작업은 대부분 설정 파일을 통해 이뤄지는데 prometheus 역시 설정 파일을 통해 배포된다. 설정 파일 관련 포맷은 추후에 업로드할 예정이다.  설정 파일은 다음과 같이 작성할 수 있다. 

  ```bash
  $ vim prometheus.yml
  
  >>
  apiVersion: apps/v1beta1
  kind: Deployment
  metadata:
    name: prometheus
    namespace: serverless
  spec:
    replicas: 1
    template:
      metadata:
        labels:
          app: prometheus
      spec:
        containers:
        - name: prometheus
          image: prom/prometheus:v2.3.1
          command:
            - "prometheus"
            - "--config.file=/etc/prometheus/prometheus.yml"
          imagePullPolicy: Always
          ports:
          - containerPort: 9090
            protocol: TCP
          resources:
            requests:
              memory: 512Mi
            limits:
              memory: 512Mi
          volumeMounts:
          - mountPath: /etc/prometheus/prometheus.yml
            name: prometheus-config
            subPath: prometheus.yml
          - mountPath: /etc/prometheus/alert.rules.yml
            name: prometheus-config
            subPath: alert.rules.yml
  
        volumes:
          - name: prometheus-config
            configMap:
              name: prometheus-config
              items:
                - key: prometheus.yml
                  path: prometheus.yml
                  mode: 0644
                - key: alert.rules.yml
                  path: alert.rules.yml
                  mode: 0644
  
  ---
  
  apiVersion: v1
  kind: Service
  metadata:
    name: prometheus
    namespace: serverless
    labels:
      app: prometheus
  spec:
    type: NodePort
    ports:
      - port: 9090
        protocol: TCP
        targetPort: 9090
        nodePort: 31119
    selector:
      app: prometheus
  
  
  ---
  
  kind: ConfigMap
  apiVersion: v1
  metadata:
    labels:
      app: prometheus
    name: prometheus-config
    namespace: serverless
  data:
    prometheus.yml: |
      global:
        scrape_interval:     15s
        evaluation_interval: 15s
        external_labels:
            monitor: 'serverless-monitor'
      rule_files:
          - 'alert.rules.yml'
      scrape_configs:
        - job_name: 'prometheus'
          scrape_interval: 5s
          static_configs:
            - targets: ['localhost:9090']
        - job_name: "gateway"
          scrape_interval: 5s
          dns_sd_configs:
            - names: ['gateway.serverless']
              port: 10000
              type: A
              refresh_interval: 5s
        - job_name: 'node_exporter'
          scrape_interval: 5s
          static_configs:
            - targets: ['10.0.0.100:9100']
      alerting:
        alertmanagers:
        - static_configs:
          - targets:
            - "10.0.0.100:9093"              
  
    alert.rules.yml: |
      groups:
      - name: serverless
        rules:
        - alert: service_down
          expr: up == 0
        - alert: APIHighInvocationRate
          expr: sum(rate(gateway_function_invocation_total{code="200"}[10s])) BY (function_name)
            > 5
          for: 5s
          labels:
            service: gateway
            severity: major
            value: '{{$value}}'
          annotations:
            description: High invocation total on {{ $labels.instance }}
            summary: High invocation total on {{ $labels.instance }}
  ```

  위와 같이 설정 파일을 작성하였으면 다음의 명령어를 통해 container에 배포한다.

  ```bash
  $ kubectl apply -f prometheus.yml
  ```


- Grafana 배포 

  Prometheus를 통해 수집된 데이터를 시각화하기 위해 grafana도 마찬가지로 배포하여야 하는데 prometheus와 동일하게 설정 파일을 통해 배포한다. 

  ```bash
  $ vim grafana.yml
  
  >>
  apiVersion: apps/v1beta1
  kind: Deployment
  metadata:
    name: grafana-core
    namespace: serverless
    labels:
      app: grafana
      component: core
  spec:
    replicas: 1
    template:
      metadata:
        labels:
          app: grafana
          component: core
      spec:
        containers:
        - image: grafana/grafana:4.0.2
          name: grafana-core
          imagePullPolicy: IfNotPresent
          resources:
             limits:
              cpu: 100m
              memory: 100Mi
            requests:
              cpu: 100m
              memory: 100Mi
          env:
            - name: GF_AUTH_BASIC_ENABLED
              value: "true"
            - name: GF_SECURITY_ADMIN_USER
              valueFrom:
                secretKeyRef:
                  name: grafana
                  key: admin-username
            - name: GF_SECURITY_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: grafana
                  key: admin-password
            - name: GF_AUTH_ANONYMOUS_ENABLED
              value: "true"
          readinessProbe:
            httpGet:
              path: /login
              port: 3000
          volumeMounts:
          - name: grafana-persistent-storage
            mountPath: /var/lib/grafana
        volumes:
        - name: grafana-persistent-storage
          emptyDir: {}
  
  ---
  
  apiVersion: v1
  kind: Secret
  data:
    admin-password: YWRtaW4=
    admin-username: YWRtaW4=
  metadata:
    name: grafana
    namespace: serverless
  type: Opaque
  
  ---
  
  apiVersion: v1
  kind: Service
  metadata:
    name: grafana
    namespace: serverless
    labels:
      app: grafana
      component: core
  spec:
    type: NodePort
    ports:
      - port: 3000
        targetPort: 3000
        nodePort: 30949
    selector:
      app: grafana
      component: core
  
  ---
  
  apiVersion: v1
  data:
    dashboard.json: |
      {
        "__inputs": [
          {
            "name": "DS",
            "label": "PWD",
            "description": "",
            "type": "datasource",
            "pluginId": "prometheus",
            "pluginName": "Prometheus"
          }
        ],
        "__requires": [
          {
            "type": "panel",
            "id": "graph",
            "name": "Graph",
            "version": ""
          },
          {
            "type": "panel",
            "id": "singlestat",
            "name": "Singlestat",
            "version": ""
          },
          {
            "type": "grafana",
            "id": "grafana",
            "name": "Grafana",
            "version": "4.0.2"
          },
          {
            "type": "datasource",
            "id": "prometheus",
            "name": "Prometheus",
            "version": "1.0.0"
          }
        ],
        "id": null,
        "title": "New Dashboard",
        "tags": [],
        "style": "dark",
        "timezone": "browser",
        "editable": true,
        "sharedCrosshair": true,
        "hideControls": false,
        "time": {
          "from": "now-1h",
          "to": "now"
        },
        "timepicker": {
          "refresh_intervals": [
            "5s",
            "10s",
            "30s",
            "1m",
            "5m",
            "15m",
            "30m",
            "1h",
            "2h",
            "1d"
          ],
          "time_options": [
            "5m",
            "15m",
            "1h",
            "6h",
            "12h",
            "24h",
            "2d",
            "7d",
            "30d"
          ]
        },
        "templating": {
          "list": [
            {
              "allValue": null,
              "current": {},
              "datasource": "DS",
              "hide": 0,
              "includeAll": false,
              "label": null,
              "multi": false,
              "name": "server",
              "options": [],
              "query": "label_values(node_boot_time, instance)",
              "refresh": 1,
              "regex": "",
              "sort": 0,
              "tagValuesQuery": null,
              "tagsQuery": null,
              "type": "query"
            }
          ]
        },
        "annotations": {
          "list": []
        },
        "refresh": "30s",
        "schemaVersion": 13,
        "version": 4,
        "links": [],
        "gnetId": 854,
        "rows": [
          {
            "title": "New row",
            "panels": [
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 8,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [
                  {
                    "alias": "Load",
                    "color": "#DEDAF7",
                    "fill": 0,
                    "stack": false,
                    "yaxis": 2
                  }
                ],
                "span": 6,
                "stack": true,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "sum(irate(node_cpu_seconds_total{mode!=\"idle\"}[5m])) by (mode)",
                    "intervalFactor": 2,
                    "legendFormat": "{{mode}}",
                    "metric": "node_cpu_seconds_total",
                    "refId": "A",
                    "step": 10
                  },
                  {
                    "expr": "node_load1",
                    "intervalFactor": 2,
                    "legendFormat": "Load",
                    "refId": "B",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "CPU Usage",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "percentunit",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "none",
                    "label": "Load",
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              },
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 2,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "rightSide": false,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [
                  {
                    "alias": "/.+ receive/",
                    "transform": "negative-Y"
                  }
                ],
                "span": 6,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "irate(node_network_receive_bytes_total[1m])",
                    "intervalFactor": 2,
                    "legendFormat": "{{device}} receive",
                    "metric": "node_network_receive_bytes",
                    "refId": "A",
                    "step": 10
                  },
                  {
                    "expr": "irate(node_network_transmit_bytes_total[1m])",
                    "intervalFactor": 2,
                    "legendFormat": "{{device}} transmit",
                    "metric": "node_network_transmit_bytes",
                    "refId": "B",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Network",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "Bps",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": "250px",
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          },
          {
            "title": "New row",
            "panels": [
              {
                "aliasColors": {
                  "Free": "#70DBED"
                },
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 4,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [
                  {
                    "alias": "Cached",
                    "color": "#64B0C8"
                  }
                ],
                "span": 6,
                "stack": true,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "node_memory_MemTotal_bytes - node_memory_MemFree_bytes - node_memory_Buffers_bytes - node_memory_Cached_bytes",
                    "hide": false,
                    "intervalFactor": 2,
                    "legendFormat": "Used",
                    "metric": "",
                    "refId": "A",
                    "step": 10
                  },
                  {
                    "expr": "node_memory_Buffers",
                    "intervalFactor": 2,
                    "legendFormat": "Buffered",
                    "metric": "node_memory_Buffers",
                    "refId": "D",
                    "step": 10
                  },
                  {
                    "expr": "node_memory_Cached",
                    "intervalFactor": 2,
                    "legendFormat": "Cached",
                    "metric": "node_memory_SwapFree",
                    "refId": "B",
                    "step": 10
                  },
                  {
                    "expr": "node_memory_MemFree",
                    "intervalFactor": 2,
                    "legendFormat": "Free",
                    "metric": "node_memory_MemTotal",
                    "refId": "C",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Memory Usage",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "bytes",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              },
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 5,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "rightSide": false,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [
                  {
                    "alias": "/.* written/",
                    "transform": "negative-Y"
                  }
                ],
                "span": 6,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "irate(node_disk_read_bytes_total[5m])",
                    "hide": false,
                    "intervalFactor": 2,
                    "legendFormat": "{{device}} read",
                    "metric": "node_disk_bytes_read",
                    "refId": "B",
                    "step": 10
                  },
                  {
                    "expr": "irate(node_disk_written_bytes_total[5m])",
                    "hide": false,
                    "intervalFactor": 2,
                    "legendFormat": "{{device}} written",
                    "metric": "node_disk_",
                    "refId": "C",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Disk thoughput",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "Bps",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "ms",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": "250px",
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          },
          {
            "title": "New row",
            "panels": [
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 6,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "rightSide": false,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [
                  {
                    "alias": "/.* io time/",
                    "yaxis": 2
                  }
                ],
                "span": 6,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "node_disk_io_now",
                    "intervalFactor": 2,
                    "legendFormat": "{{device}}",
                    "metric": "node_disk_io",
                    "refId": "A",
                    "step": 10
                  },
                  {
                    "expr": "irate(node_disk_io_time_seconds_total[5m])",
                    "intervalFactor": 2,
                    "legendFormat": "{{device}} io time",
                    "metric": "node_disk_io_time_ms",
                    "refId": "B",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Disk IO",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "iops",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "ms",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              },
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "grid": {},
                "id": 7,
                "legend": {
                  "alignAsTable": true,
                  "avg": true,
                  "current": true,
                  "max": true,
                  "min": true,
                  "show": true,
                  "total": true,
                  "values": true
                },
                "lines": true,
                "linewidth": 2,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [],
                "span": 6,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "(node_filesystem_size_bytes - node_filesystem_avail_bytes) / node_filesystem_size_bytes",
                    "intervalFactor": 2,
                    "legendFormat": "{{mountpoint}}",
                    "metric": "node_filesystem_size",
                    "refId": "A",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Disk usage",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "cumulative"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "percentunit",
                    "label": "",
                    "logBase": 1,
                    "max": "1",
                    "min": "0",
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": "250px",
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          },
          {
            "title": "Dashboard Row",
            "panels": [
              {
                "cacheTimeout": null,
                "colorBackground": true,
                "colorValue": false,
                "colors": [
                  "rgba(50, 172, 45, 0.97)",
                  "rgba(237, 129, 40, 0.89)",
                  "rgba(245, 54, 54, 0.9)"
                ],
                "datasource": "DS",
                "editable": true,
                "error": false,
                "format": "none",
                "gauge": {
                  "maxValue": 100,
                  "minValue": 0,
                  "show": false,
                  "thresholdLabels": false,
                  "thresholdMarkers": true
                },
                "id": 9,
                "interval": null,
                "links": [],
                "mappingType": 1,
                "mappingTypes": [
                  {
                    "name": "value to text",
                    "value": 1
                  },
                  {
                    "name": "range to text",
                    "value": 2
                  }
                ],
                "maxDataPoints": 100,
                "nullPointMode": "connected",
                "nullText": null,
                "postfix": "",
                "postfixFontSize": "50%",
                "prefix": "",
                "prefixFontSize": "50%",
                "rangeMaps": [
                  {
                    "from": "null",
                    "text": "N/A",
                    "to": "null"
                  }
                ],
                "span": 4,
                "sparkline": {
                  "fillColor": "rgba(31, 118, 189, 0.18)",
                  "full": false,
                  "lineColor": "rgb(31, 120, 193)",
                  "show": false
                },
                "targets": [
                  {
                    "expr": "up{job=\"gateway\"}",
                    "intervalFactor": 2,
                    "refId": "A",
                    "step": 60
                  }
                ],
                "thresholds": "0.1",
                "title": "Gateway Health",
                "type": "singlestat",
                "valueFontSize": "80%",
                "valueMaps": [
                  {
                    "op": "=",
                    "text": "I'm Sick",
                    "value": "0"
                  },
                  {
                    "op": "=",
                    "text": "Healthy",
                    "value": "1"
                  }
                ],
                "valueName": "avg"
              },
              {
                "cacheTimeout": null,
                "colorBackground": false,
                "colorValue": false,
                "colors": [
                  "rgba(50, 172, 45, 0.97)",
                  "rgba(237, 129, 40, 0.89)",
                  "rgba(245, 54, 54, 0.9)"
                ],
                "datasource": "DS",
                "editable": true,
                "error": false,
                "format": "none",
                "gauge": {
                  "maxValue": 50,
                  "minValue": 0,
                  "show": true,
                  "thresholdLabels": false,
                  "thresholdMarkers": true
                },
                "id": 10,
                "interval": null,
                "links": [],
                "mappingType": 1,
                "mappingTypes": [
                  {
                    "name": "value to text",
                    "value": 1
                  },
                  {
                    "name": "range to text",
                    "value": 2
                  }
                ],
                "maxDataPoints": 100,
                "nullPointMode": "connected",
                "nullText": null,
                "postfix": "",
                "postfixFontSize": "50%",
                "prefix": "",
                "prefixFontSize": "50%",
                "rangeMaps": [
                  {
                    "from": "null",
                    "text": "N/A",
                    "to": "null"
                  }
                ],
                "span": 4,
                "sparkline": {
                  "fillColor": "rgba(31, 118, 189, 0.18)",
                  "full": false,
                  "lineColor": "rgb(31, 120, 193)",
                  "show": false
                },
                "targets": [
                  {
                    "expr": "sum(gateway_service_count)",
                    "intervalFactor": 2,
                    "refId": "A",
                    "step": 60
                  }
                ],
                "thresholds": "20,40,50",
                "title": "Gateway Service Count",
                "type": "singlestat",
                "valueFontSize": "80%",
                "valueMaps": [
                  {
                    "op": "=",
                    "text": "N/A",
                    "value": "null"
                  }
                ],
                "valueName": "avg"
              },
              {
                "cacheTimeout": null,
                "colorBackground": false,
                "colorValue": true,
                "colors": [
                  "rgba(245, 54, 54, 0.9)",
                  "rgba(237, 129, 40, 0.89)",
                  "rgba(50, 172, 45, 0.97)"
                ],
                "datasource": "DS",
                "editable": true,
                "error": false,
                "format": "none",
                "gauge": {
                  "maxValue": 100,
                  "minValue": 0,
                  "show": false,
                  "thresholdLabels": false,
                  "thresholdMarkers": true
                },
                "id": 11,
                "interval": null,
                "links": [],
                "mappingType": 1,
                "mappingTypes": [
                  {
                    "name": "value to text",
                    "value": 1
                  },
                  {
                    "name": "range to text",
                    "value": 2
                  }
                ],
                "maxDataPoints": 100,
                "nullPointMode": "connected",
                "nullText": null,
                "postfix": "",
                "postfixFontSize": "50%",
                "prefix": "",
                "prefixFontSize": "50%",
                "rangeMaps": [
                  {
                    "from": "null",
                    "text": "N/A",
                    "to": "null"
                  }
                ],
                "span": 4,
                "sparkline": {
                  "fillColor": "rgba(31, 118, 189, 0.18)",
                  "full": true,
                  "lineColor": "rgb(31, 120, 193)",
                  "show": true
                },
                "targets": [
                  {
                    "expr": "sum(gateway_function_invocation_total)",
                    "intervalFactor": 2,
                    "refId": "A",
                    "step": 60
                  }
                ],
                "thresholds": "",
                "title": "Total Invocation",
                "type": "singlestat",
                "valueFontSize": "80%",
                "valueMaps": [
                  {
                    "op": "=",
                    "text": "N/A",
                    "value": "null"
                  }
                ],
                "valueName": "current"
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": 250,
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          },
          {
            "title": "Dashboard Row",
            "panels": [
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 4,
                "id": 12,
                "legend": {
                  "avg": false,
                  "current": false,
                  "max": false,
                  "min": false,
                  "show": true,
                  "total": false,
                  "values": false
                },
                "lines": true,
                "linewidth": 3,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [],
                "span": 6,
                "stack": false,
                "steppedLine": true,
                "targets": [
                  {
                    "expr": "gateway_service_count",
                    "intervalFactor": 2,
                    "legendFormat": "{{function_name}} ",
                    "refId": "A",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Replica scaling",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "individual"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              },
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 4,
                "id": 13,
                "legend": {
                  "avg": false,
                  "current": false,
                  "max": false,
                  "min": false,
                  "show": true,
                  "total": false,
                  "values": false
                },
                "lines": true,
                "linewidth": 1,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [],
                "span": 6,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "rate (gateway_function_invocation_total [20s])",
                    "intervalFactor": 2,
                    "legendFormat": "{{function_name}} {{code}}",
                    "refId": "A",
                    "step": 10
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Function rate",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "individual"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": 250,
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          },
          {
            "title": "Dashboard Row",
            "panels": [
              {
                "aliasColors": {},
                "bars": false,
                "datasource": "DS",
                "editable": true,
                "error": false,
                "fill": 1,
                "id": 14,
                "legend": {
                  "avg": false,
                  "current": false,
                  "max": false,
                  "min": false,
                  "show": true,
                  "total": false,
                  "values": false
                },
                "lines": true,
                "linewidth": 1,
                "links": [],
                "nullPointMode": "connected",
                "percentage": false,
                "pointradius": 5,
                "points": false,
                "renderer": "flot",
                "seriesOverrides": [],
                "span": 12,
                "stack": false,
                "steppedLine": false,
                "targets": [
                  {
                    "expr": "(rate(gateway_functions_seconds_sum[20s]) / rate(gateway_functions_seconds_count[20s]))",
                    "intervalFactor": 2,
                    "legendFormat": "{{function_name}} ",
                    "refId": "A",
                    "step": 4
                  }
                ],
                "thresholds": [],
                "timeFrom": null,
                "timeShift": null,
                "title": "Execution duration (s)",
                "tooltip": {
                  "msResolution": false,
                  "shared": true,
                  "sort": 0,
                  "value_type": "individual"
                },
                "type": "graph",
                "xaxis": {
                  "mode": "time",
                  "name": null,
                  "show": true,
                  "values": []
                },
                "yaxes": [
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  },
                  {
                    "format": "short",
                    "label": null,
                    "logBase": 1,
                    "max": null,
                    "min": null,
                    "show": true
                  }
                ]
              }
            ],
            "showTitle": false,
            "titleSize": "h6",
            "height": 250,
            "repeat": null,
            "repeatRowId": null,
            "repeatIteration": null,
            "collapse": false
          }
        ]
      }
    prometheus-datasource.json: | 
      {
        "name": "DS",
        "type": "prometheus",
        "access": "proxy",
        "url": "http://10.0.0.100:31119",
        "basicAuth": false
      }
  kind: ConfigMap
  metadata:
    creationTimestamp: null
    name: grafana-import-dashboards
    namespace: serverless
  
  ---
  
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: grafana-import-dashboards
    namespace: serverless
    labels:
      app: grafana
      component: import-dashboards
  spec:
    template:
      metadata:
        name: grafana-import-dashboards
        labels:
          app: grafana
          component: import-dashboards
      spec:
        containers:
        - name: grafana-import-dashboards
          image: giantswarm/tiny-tools
          command: ["/bin/sh", "-c"]
          workingDir: /opt/grafana-import-dashboards
          args:
            - >
              for file in *-datasource.json ; do
                if [ -e "$file" ] ; then
                  echo "importing $file" &&
                  curl --silent --fail --show-error \
                    --request POST http://${GF_ADMIN_USER}:${GF_ADMIN_PASSWORD}@10.0.0.100:30949/api/datasources \
                    --header "Content-Type: application/json" \
                    --data-binary "@$file" ;
                  echo "" ;
                fi
              done ;
              for file in *-dashboard.json ; do
                if [ -e "$file" ] ; then
                  echo "importing $file" &&
                  ( echo '{"dashboard":'; \
                    cat "$file"; \
                    echo ',"overwrite":true,"inputs":[{"name":"DS_PROMETHEUS","type":"datasource","pluginId":"prometheus","value":"prometheus"}]}' ) \
                  | jq -c '.' \
                  | curl --silent --fail --show-error \
                    --request POST http://${GF_ADMIN_USER}:${GF_ADMIN_PASSWORD}@10.0.0.100:30949/api/dashboards/db \
                    --header "Content-Type: application/json" \
                    --data-binary "@-" ;
                  echo "" ;
                fi
              done
          env:
          - name: GF_ADMIN_USER
            valueFrom:
              secretKeyRef:
                name: grafana
                key: admin-username
          - name: GF_ADMIN_PASSWORD
            valueFrom:
              secretKeyRef:
                name: grafana
                key: admin-password
          volumeMounts:
          - name: config-volume
            mountPath: /opt/grafana-import-dashboards
        restartPolicy: Never
        volumes:
        - name: config-volume
          configMap:
            name: grafana-import-dashboards
  ```

  Grafana 역시 다음의 명령어를 통해 배포한다.

  ```bash
  $ kubectl apply -f grafana.yml
  ```



- 실행

  모든 구성요소의 배포가 완료되면 container 안에서 실행하기 전 pod의 상태가 `Running`인지를 확인해야 한다. 

  ```bash
  $ kubectl get all -n serverless
  ```

  위의 명령어를 통해 pod의 상태를 확인한 후, prometheus와 grafana를 실행하면 된다. 설정파일에 두 개의 서비스는 NodePort로 정의되었는데, 이를 통해 클러스터 IP로만 접근 가능한 것이 아닌 모든 노드(서버)의 IP와 포트로도 접근이 가능하다. 때문에 kubernetes container 안에서 실행된 각각의 서비스들은 다음과 같이 웹에서 접근할 수 있다.

  - Prometheus

    ```bash
    http://10.0.0.100:31119
    ```

  - Grafana

    ```bash
    http://10.0.0.100:30949
    ```

