---
title: Elastic Stack (ELK)
author: Huey-J
date: 2022-08-08 12:00:00 +0800
categories: [데브옵스, 네트워크]
tags: [AWS, DevOps]
---

# NOW : Elastic Stack : ELK Stack

모니터링 서비스에서 ELK는 가장 많이 사용되는 방법 중 하나이다.
OOOO년에 출시되어 오랜 기간 사랑받은 ELK 방식은 Elastic 홈페이지에 의하면 사실 현재 이름이 바뀌었다.
원래는 ELK란 Elastic, Logstash, Kibana의 머리글자를 따서 만든 약자이다.
하지만 점차 다양한 서비스와 플러그인을 지원하게 되면서 더 이상 머리 글자를 따서 만드는 형식으로는 너무 길어지는 상황이 생겼다.

그렇게 나온 용어가 바로 Elastic Stack 이다. 이번 포스트에서는 바로 이 Elastic Stack 에 대해 알아


ELK 스택 (Elastic Stack) 이란?
ELK 스택은 Elasticsearch, Logstash, Kibana의 세 가지 인기 있는 프로젝트로 구성된 스택을 의미하는 약어입니다.
Elasticsearch라고도 불리는 ELK 스택은 사용자에게 모든 시스템과 애플리케이션에서 로그를 집계하고 이를 분석하며 애플리케이션과 인프라 모니터링 시각화를 생성하고, 빠르게 문제를 해결하며 보안 분석할 수 있는 능력을 제공합니다.

ELK 스택은 로그 분석 공간에서의 필요를 채워주기 때문에 유명합니다.
여러분의 IT 인프라가 점점 더 퍼블릭 클라우드로 이동할수록, 해당 인프라와 서버 로그, 애플리케이션 로그, 클릭스트림 프로세스를 모니터링하기 위해 로그 관리와 분석 솔루션이 필요합니다.
ELK 스택은 개발자와 DevOps 엔지니어가 오류 진단, 애플리케이션 성능, 인프라 모니터링으로부터 값진 인사이트를 얻을 수 있도록 적은 비용으로 단순하면서도 강력한 로그 분석 솔루션을 제공합니다.
https://aws.amazon.com/ko/opensearch-service/the-elk-stack/


"ELK"는 Elasticsearch, Logstash 및 Kibana, 이 오픈 소스 프로젝트 세 개의 머리글자입니다.
Elasticsearch는 검색 및 분석 엔진입니다.
Logstash는 여러 소스에서 동시에 데이터를 수집하여 변환한 후 Elasticsearch 같은 “stash”로 전송하는 서버 사이드 데이터 처리 파이프라인입니다.
Kibana는 사용자가 Elasticsearch에서 차트와 그래프를 이용해 데이터를 시각화할 수 있게 해줍니다.
Elastic Stack은 ELK Stack이 그 다음 단계로 발전한 것입니다.
https://www.elastic.co/kr/what-is/elk-stack


## Elastic Search

Elasticsearch는 Apache Lucene에 구축되어 배포된 검색 및 분석 엔진입니다.
다양한 언어를 지원하고 고성능에 스키마가 없는 JSON 문서로 Elasticsearch는 다양한 로그 분석과 검색 사용 사례에 최고의 선택이 되었습니다.
https://aws.amazon.com/ko/opensearch-service/the-elk-stack/what-is-elasticsearch/

일래스틱서치(Elasticsearch)는 루씬 기반의 검색 엔진이다.
HTTP 웹 인터페이스와 스키마에서 자유로운 JSON 문서와 함께 분산 멀티테넌트 지원 전문 검색 엔진을 제공한다.
일래스틱서치는 자바로 개발되어 있으며 아파치 라이선스 조항에 의거하여 오픈 소스로 출시되어 있다.
공식 클라이언트들은 자바, 닷넷(C#), PHP, 파이썬, 그루비 등 수많은 언어로 이용이 가능하다.
일래스틱서치는 로그스태시(Logstash)라는 이름의 데이터 수집 및 로그 파싱 엔진, 그리고 키바나(Kibana)라는 이름의 분석 및 시각화 플랫폼과 함께 개발되어 있다.
이 3개의 제품들은 연동 솔루션으로 사용할 목적으로 설계되어 있으며 이를 "일래스틱 스택"(Elastic Stack, 과거 이름: ELK 스택)으로 부른다.
https://ko.wikipedia.org/wiki/일래스틱서치

> 아파치 루씬
> 아파치 루씬(Apache Lucene)은 자바 언어로 이루어진 정보 검색 라이브러리 자유-오픈 소스 소프트웨어이다.
> 전문 검색(Full text) 색인 및 검색 기능을 필요로 하는 모든 응용 프로그램에 적합하지만 루씬은 웹 검색 엔진 및 로컬 단일 사이트 검색 구현에서의 유용성으로 널리 알려져 있다.
> 루씬은 또한 추천 시스템을 구현하는데 사용되고 있다. 예를 들어, 루씬의 'MoreLikeThis' 클래스는 유사한 문서에 대한 추천을 생성 할 수 있다.
>
> 루씬 그 자체는 색인 및 검색을 제공하는 라이브러리이며, 웹 크롤러나 HTML 구문 분석 등의 기능은 포함하지 않는다.
> 참고로 확장기능이 포함되지 않은 루씬 사용자. 예를 들어 트위터는 실시간 검색을 위해서 루씬을 사용하고 있다.
> 하지만 아래의 다양한 프로젝트가 루씬의 기능을 확장한다.
> - 아파치 너치 - 웹 크롤러 및 HTML 구문 분석 제공
> - 아파치 솔라 - 엔터프라이즈 검색 서버
> - Compass - 엘라스틱서치의 전신
> - CrateDB - 오픈소스, 루씬을 기반으로 하는 분산 SQL 데이터베이스
> - DocFetcher - 크로스 플랫폼 데스크톱 환경 검색 애플리케이션
> - 엘라스틱서치 - 2010년에 만들어진 엔터프라이즈 서버
> - Kinosearch - 약간의 루씬 포팅 함께 펄과 C로 작성한 검색 엔진. Socialtext의 위키, 모조모조 위키 엔진, Human Metabolome Database(HMDB)와 Toxin and Toxin-Target Database(T3DB)에서 사용한다.
> - Swiftype - 루씬 기반의 엔터프라이즈 서버 스타트업
>
> https://ko.wikipedia.org/wiki/아파치_루씬

## Logstash

Logstash는 다양한 소스로부터 데이터를 수집하고 곧바로 전환하여 원하는 대상에 전송할 수 있도록 하는 경량의 오픈 소스 서버측 데이터 처리 파이프라인입니다.
오픈소스 분석 및 검색 엔진인 Elasticsearch의 데이터 파이프라인으로 자주 사용됩니다.
Elasticsearch와의 긴밀한 통합, 강력한 로그 처리 능력, 사전 구축된 200개 이상의 오픈 소스 플러그인을 통해 데이터 인덱싱을 돕는 Logstash는 Elasticsearch에 데이터를 로드할 때 가장 많이 사용됩니다.
https://aws.amazon.com/ko/opensearch-service/the-elk-stack/logstash/

## Kibana

오픈소스이며 Elastic Search용 데이터 시각화 플러그인

Kibana는 로그와 시계열 분석, 애플리케이션 모니터링, 운영 인텔리전스사용 사례에 사용되는 데이터 시각화 및 탐색 도구입니다.
히스토그램, 선형 그래프, 원형 차트, 열 지도, 내장 지리 공간적 지원과 같은 강력하고 사용하기 쉬운 기능을 제공합니다.
또한 유명 분석 및 검색 엔진인 Elasticsearch와 긴밀하게 통합되어 Kibana는 Elasticsearch에 저장된 데이터를 시각화할 때 기본 선택 사항이 되었습니다.
https://aws.amazon.com/ko/opensearch-service/the-elk-stack/kibana/

Kibana는 Elasticsearch 데이터를 시각화하고 Elastic Stack을 탐색하게 해주는 무료 오픈 소스 인터페이스입니다.
쿼리 작업량 추적부터 앱을 통한 요청 흐름 방식에 대한 이해에 이르기까지 무엇이든 해보세요.
https://www.elastic.co/kr/kibana/

## APM

Elastic Search가 지원하는 모니터링 시스템.
Spring 기반 웹서버에서 편리하게 구성할 수 있게 Elastic APM Agent를 제공하고 있습니다.
스프링 기반의 웹서버를 Elasticsearch, Kibana를 이용해서 APM 연결하는 내용입니다.


