---
title: Elastic Stack (ELK)
author: Huey-J
date: 2022-08-15 12:00:00 +0800
categories: [데브옵스, 네트워크]
tags: [AWS, DevOps, Elastic Stack]
---

# 로그 모니터링

운영환경에 애플리케이션을 배포하고 서비스를 운영하다 보면 경험해보지 못한 오류를 마주치곤 합니다.
우리는 이러한 상황에서 좀 더 효율적으로 오류를 분석하고 대응하기 위해 '로그'를 남겨 놓습니다.

로그를 남기는 방법 또한 다양합니다. 시스템 로컬에 파일로 남기거나 특정 로그 서버를 설정하여 여러 대의 서버 로그를 한곳에서 볼 수도 있죠.

텍스트 그 자체로 분석할 수 있겠지만 로그가 쌓이고 종류가 다양해지면서 이 상태로 분석하는 것은 거의 불가능에 가깝습니다.
따라서 우리는 로그 파일을 모니터링할 수 있는 솔루션이 필요하였고 그중 가장 흔하게 사용되는 것이 바로 오늘 알아볼 **Elastic Stack**입니다.


# ELK

![elk stack](/assets/img/elk_stack.png)
_Image Reference: https://www.elastic.co/brand_

ELK란 Elasticsearch, Logstash, Kibana의 세 가지 인기 있는 프로젝트로 구성된 스택을 의미하는 약자입니다.
모니터링 서비스에서 가장 많이 사용되는 방법 중 하나이며 사용자에게 모든 시스템과 애플리케이션에서 로그를 집계하고 이를 분석하며 애플리케이션과 인프라 모니터링 시각화를 생성하고, 빠르게 문제를 해결하며 보안 분석할 수 있는 능력을 제공합니다.
심지어 오픈소스로, 무료로 사용할 수 있습니다.

> ELK와 비슷하지만 유료 서비스인 [Splunk](https://www.splunk.com/en_us/about-splunk.html)가 있습니다.


## Elastic Search

![elastic search](/assets/img/elastic_search.png)
_Image Reference: https://www.elastic.co/brand_

Elastic search는 [아파치 루씬](#apache-lucene) 기반의 검색 및 분석 엔진입니다.
자바, 닷넷(C#), PHP, 파이썬 등 수많은 언어로 이용할 수 있으며 오픈 소스입니다.
또한 스키마에서 자유로운 JSON 문서를 지원하고 HTTP 웹 인터페이스 역시 지원합니다.

Elastic Search는 사실 로그 분석용 도구는 아닙니다. 성능 짱짱인 랭킹 1위 **검색엔진**이죠.
하지만 그만큼 성능과 확장성이 검증되었기 때문에 로그 모니터링에서의 검색엔진으로써도 활용되고 있습니다.

로그 분석 외에도 Elastic Search는 아래와 같이 다양한 사례에 이용될 수 있습니다.

- 애플리케이션 검색
- 웹사이트 검색
- 엔터프라이즈 검색
- **로깅과 로그 분석**
- 인프라 메트릭과 컨테이너 모니터링
- 애플리케이션 성능 모니터링
- 위치 기반 정보 데이터 분석 및 시각화
- 보안 분석
- 비즈니스 분석

> ### Apache Lucene
> 자바 언어로 이루어진 정보 검색 라이브러리 오픈 소스 소프트웨어.
> 전문 검색 색인 및 검색 기능을 필요로 하는 모든 응용 프로그램에 적합하지만 웹 검색 엔진 및 로컬 단일 사이트 검색 구현에서의 유용성으로 널리 알려져 있다.
> 추천 시스템을 구현하는데 사용되곤 한다.\
> [출처 - 위키피디아](https://ko.wikipedia.org/wiki/아파치_루씬)

## Logstash

![logstash](/assets/img/logstash.png)
_Image Reference: https://www.elastic.co/brand_

Logstash는 JRuby로 개발된 데이터 수집 엔진입니다.
좀 더 길게 표현하면 다양한 소스로부터 동시에
데이터를 수집, 전환하여 전송할 수 있도록 하는 오픈 소스 데이터 처리 파이프라인이라고 말할 수 있겠네요.

시스템 로그, 웹 사이트 로그, 애플리케이션 서버 로그 등 다양한 형식의 로그 데이터 원본에서 비정형 데이터를 쉽게 수집할 수 있습니다.

Elastic Stack에서는 검색 엔진인 Elasticsearch의 데이터 파이프라인으로써 로그 데이터를 인덱싱하여 전송해주는 역할을 담당합니다.

## Kibana

![kibana](/assets/img/kibana.png){: style="width: auto; height:180px;" }
_Image Reference: https://www.elastic.co/brand_

Kibana는 Elasticsearch 데이터를 시각화하고 Elastic Stack을 탐색하게 해주는 데이터 시각화 플러그인입니다.

수집한 로그를 더욱 쉽게 분석할 수 있도록 도와줍니다.
간단하게 말하자면 HTML + Javascript GUI 엔진이라고 볼 수 있습니다.

키바나(Kibana)의 경우 대체할 수 있는 소프트웨어로 그라파나(Grafana)가 있습니다.

![kibana](/assets/img/kibana_example.png)
_Image Reference: https://www.elastic.co/brand_

## ELK 동작 원리

자, 그럼 ELK의 구성 요소를 살펴 보았으니 전체적인 동작 원리를 알아봅시다.

1. Logstash - Data Processing
- 서버 내의 로그 등을 수집하여 데이터 변환, 인덱싱 이후 송신
2. Elastic Search - Storage
- 데이터 저장, 분석, 관리
3. Kibana - Visualize
- 대시보드, 그래프 등의 인터페이스 제공 및 엑세스 제어와 같은 추가 기능 제공


# Elastic Stack

지금까지 ELK에 대해 알아보았습니다. 그럼 제가 글의 첫부분에서 언급한 Elastic Stack은 뭘까요?

Elastic Stack은 ELK의 새로운, 개선된 이름입니다.
시간이 지나면서 Elasticsearch, Logstash, Kibana의 앞글자를 딴 ELK에 Beat 등의 서비스가 추가되었고,
더 이상 머리 글자를 따서 만드는 형식으로는 너무 길어지는 상황이 생겼죠.

그래서 Elastic Stack이라는 이름이 나온 것입니다.
이름이 변경된 이후 ELK 외의 여러 서비스가 출시되었고, 또 많은 회사들이 실제 서비스에 적용하여 활용하고 있습니다.

> ### Beats
> 서버에 에이전트로 설치하여 다양한 유형의 데이터를 ElasticSearch 또는 Logstash에 전송하는 오픈 소스 데이터 수집기\
> [출처 - Elastic/beats](https://www.elastic.co/kr/beats)


# References

[https://www.elastic.co/kr/what-is/elk-stack](https://www.elastic.co/kr/what-is/elk-stack)\
[https://www.elastic.co/kr/kibana](https://www.elastic.co/kr/kibana)\
[https://aws.amazon.com/ko/opensearch-service/the-elk-stack](https://aws.amazon.com/ko/opensearch-service/the-elk-stack)
