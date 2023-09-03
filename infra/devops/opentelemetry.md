# OpenTelemetry

### OpenTelemetry란?

Opentelemetry는 MSA 처럼 복잡하게 분산된 환경으로 인해 중요해진 Traces, Metrics, Logs 같이 관찰 가능성을 위해 만들어진 [CNCF](https://www.cncf.io/) 재단의 오픈소스 프로젝트이다.

OpenTelemtry는 기존의 모니터링 기술과 다르게 데이터를 수집하고 전처리하는 기능만을 책임지고, 실제 데이터를 저장하거나 표현 기능은 다른 벤더에게 위임한다.&#x20;

이렇게 중간에서 인터페이스 역할 덕에 다양한 모니터링 서비스를 보다 쉽게 사용할 수 있고 교체할 수 있다.

아래 코드를 통해 어떻게 opentelemetry를 적용했는지 살펴보자.

[git 저장소](https://github.com/spandios/opentelemetry-example)

### OtelCollector

Collector는 클라이언트로부터 데이터를 수집 후 추가 처리 단계를 거쳐 Exporters에게 전달한다.

Collector는 크게 Receivers, Proceecors, Exporters로 역할을 분리하고 있다.

<figure><img src="https://blog.kakaocdn.net/dn/HZPxi/btssBsPsoNS/XgsvCvtvxyKT0PMQ00rlik/img.webp" alt=""><figcaption></figcaption></figure>

* Receivers: 소스로부터 특정한 포맷으로 데이터를 받는 곳으로 내부 콜렉터가 이해할 수 있는 데이터 포맷으로 변환하고 다음 단계로 전달한다.
* Processors: Exporters로 데이터가 가기 전 추가적인 처리를 하는 곳이다. 데이터에 추가적인 작업을 하거나 배치같은 처리를 한다.
* Exporters: 실제 데이터를 어디에 전달할지 정의하는 곳이다.

### 실행 해보기

docer-compose를 통해 간편히 실행 할 수 있다. Trace는 jaeger Metric은 Prometheus를 이용했다.

```yaml
# OTEL-COLLECTOR, config파일을 작성하고 실행한다.
version: "3"
services:
  otel-collector:
    container_name: otel-collector
    image: otel/opentelemetry-collector:latest
    command: [ "--config=/etc/otel-collector-config.yaml" ]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "4317:4317"   # otlp receiver, 
      - "8888:8888"   # Prometheus metrics exposed by the collector
      - "8889:8889"   # Prometheus exporter metrics
```

```yaml
# config파일을 작성. otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc: # 수집 시 grpc protocols를 사용하며 기본 포트는 4317
processors:
  batch:
# export할 기술과 연결주소를 설정합니다.
exporters:
  prometheus:
    endpoint: "0.0.0.0:8889" 
  logging:
  jaeger:
    endpoint: "jaeger:14250" 
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [ otlp ]
      processors: [ batch ]
      exporters: [ logging, jaeger ] # jaeger에 trace 정보 전달 
    metrics:
      receivers: [ otlp ]
      processors: [ batch ]
      exporters: [ logging, prometheus ] # prometheus에 metric 정보 전달
    logs:
      receivers: [ otlp ]
      exporters: [ logging ]
```

```yaml
# 이제 나머지 docker-compose.yml를 추가합니다. Jaeger, Prometheus, Grafana, Application 
# 예거는 분산환경에서 트레이싱할 수 있는 오픈소스 프로젝트이다. 고로 만들어졌는지 고 케릭터가 있다. 
  jaeger: 
  image: jaegertracing/all-in-one:latest
  container_name: jaeger
  ports:
    - "16686:16686" # 웹 UI에 접속하기 위해

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus-config.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
```

```yaml
# prometheus-config.yml prometheus가 주기적으로 metric 정보를 직접 app에서 가져옴
scrape_configs:
  - job_name: 'otel-collector'
    scrape_interval: 2s
    static_configs:
      - targets: [ 'otel-collector:8889' ]
      - targets: [ 'otel-collector:8888' ]
```

\


```yaml
# application 
application:
    build:
      context: .
    environment:
      OTEL_SERVICE_NAME: opentelemetry-example # service name 
      OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector:4317 # otlp collector 주소 설정
    ports:
      - "8080:8080"
    restart: always
    depends_on:
      - otel-collector
```

### Jaeger

<figure><img src="https://blog.kakaocdn.net/dn/xKylj/btssNjcS65U/bgHdppj8FpWO2MEMLdFZh1/img.png" alt=""><figcaption></figcaption></figure>

이제 curl localhost:8080를 호출한 뒤 http://localhost:16686에 접속하면 trace를 볼 수 있다.

### Prometheus

<figure><img src="https://blog.kakaocdn.net/dn/BIXV0/btssCsJhua8/CZkmUWBMK9oyeezvQKq0fk/img.png" alt=""><figcaption></figcaption></figure>

### Grafana

<figure><img src="https://blog.kakaocdn.net/dn/wSLDq/btssGhHugKo/kYjhKmk3zB3WaCthFYsvw0/img.png" alt=""><figcaption></figcaption></figure>

주의: grafana datasource를 추가할 때 prometheus endpoint는 [http://host.docker.internal:9090](http://host.docker.internal:9090/)로 해야함.
