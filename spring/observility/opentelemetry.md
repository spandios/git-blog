# OpenTelemetry

## OpenTelemetry란?

Opentelemetry는 MSA 처럼 복잡하게 분산된 환경으로 인해 중요해진 Traces, Metrics, Logs 처럼 관찰 가능성을 위해 만들어진 오픈소스이다.

OpenTelemtry는 기존의 모니터링과 다르게 데이터를 수집하고 전처리하는 책임만 가지고 실제 데이터를 저장하거나 표현은 다른 벤더에게 위임한다.

이런 중간에서 인터페이스 역할 덕에 우리는 다양한 모니터링 서비스를 사이드 이펙트 없이 교체하고 사용할 수 있다.

아래 샘플코드([git 저장소](https://github.com/spandios/opentelemetry-example))를 통해 어떻게 opentelemetry를 적용했는지 살펴보자.





### OtelCollector&#x20;

먼저 핵심인 Collector를 알아보고 docker-compose로 띄어보자. Collector는 말 그대로 소스(클라이언트)로부터 데이터를 수집 후 Collector는 크게 Receivers, Proceecors, Exporters로 기능을 나누고 있다.&#x20;

<figure><img src="../../.gitbook/assets/028_OTELGraphic_v1-01-2048x772.png" alt=""><figcaption><p>Collector</p></figcaption></figure>

#### Receivers

소스로부터 특정한 포맷으로 데이터를 받는 곳으로 내부 콜렉터가 이해할 수 있는 데이터 포맷으로 변환하고 다음 단계로 전달한다.

#### Processors <a href="#h-processors" id="h-processors"></a>

Exporters로 데이터가 가기 전 추가적인 처리를 하는 곳이다. 데이터에 추가적인 작업을 하거나 배치같은 처리를 할 수 잇다.

#### Exporters <a href="#h-exporters" id="h-exporters"></a>

실제 모니터링 벤더사에 데이터를 전달하는 곳이다.&#x20;





```yaml
version: "3"
services:

  otel-collector:
    container_name: otel-collector
    image: otel/opentelemetry-collector:latest
    command: [ "--config=/etc/otel-collector-config.yaml" ]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml


```

