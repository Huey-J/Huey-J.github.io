---
title: AWS Athena 설정하기
author: Huey-J
date: 2022-08-22 12:00:00 +0800
categories: [데브옵스, AWS]
tags: [AWS, DevOps, Elastic Stack]
---

# 들어가며

지난 포스트에서 [Elastic Stack](/posts/Elastic-Stack(ELK)) 로그 모니터링 환경에 대해 알아보았습니다.

하지만... ELK를 구축하는건 꽤 어렵습니다. 시간도 많이 들고, 서버 리소스도 많이 잡아먹습니다.
시간적으로 보나 경제적으로 보나 비싸다는 얘기죠.

고차원적인 로그 분석이 필요한게 아니라면 사실 오버 스펙이 될 가능성이 있습니다.

그렇다면 설치가 쉽고 빠른, 서버 운영 요금 없는 간단한, 하지만 파워풀한 로그 분석 도구는 없을까요?

# AWS Athena

![aws athena icon](/assets/img/aws_athena_icon.png)
_Image Reference: https://aws.amazon.com/ko/trademark-guidelines/_

> Amazon Athena는 표준 SQL을 사용해 Amazon S3에 저장된 데이터를 간편하게 분석할 수 있는 **대화식 쿼리 서비스**입니다.
> Athena는 **서버리스 서비스**이므로 관리할 인프라가 없으며 실행한 쿼리에 대해서만 비용을 지불하면 됩니다.
{: .prompt-tip }

가장 중요한 포인트는 '**서버리스 서비스**'라는 것입니다.
서버가 따로 돌아가지 않기 때문에 운영 비용이 들지 않고, 조회할 때 마다 요금이 발생하는 방식입니다.
심지어 그 금액도 스캔한 데이터의 TB당 5.00 USD(2022년 8월 기준)로 굉장히 저렴합니다.
[요금 링크](https://aws.amazon.com/ko/athena/pricing/#example)

아래는 AWS에서 설명하는 Athena의 이점입니다. 요약한 내용이며 자세한 내용은 [여기](https://aws.amazon.com/athena)를 눌러주세요.

- Start querying instantly
: Athena는 서버리스 서비스입니다.
Amazon S3에 저장된 데이터를 지정하고 스키마를 정의한 다음 기본적으로 제공되는 쿼리 편집기를 사용해 쿼리를 시작하면 됩니다.

- Pay per query
: Amazon Athena에서는 실행한 쿼리에 대한 비용만 지불합니다.
데이터를 압축 및 파티셔닝하고 컬럼 형식으로 변환함으로써 쿼리당 비용을 30%에서 90% 절감하고 성능을 더 향상할 수 있습니다.

- Open, powerful, standard
: Amazon Athena는 ANSI SQL을 지원하는 Presto를 사용하며, CSV, JSON 등 다양한 표준 데이터 형식과 호환됩니다.
Athena는 대화형 쿼리에 적합하며 대용량 조인, 윈도 함수, 어레이 등 복잡한 분석을 처리하는 데도 손색이 없습니다.

- Fast, really fast
: Amazon Athena에서는 빠른 속도의 대화식 쿼리 성능을 구현하기 위해 컴퓨팅 리소스가 충분한지 걱정할 필요가 없습니다.
Amazon Athena는 병렬 방식으로 쿼리를 자동 실행하기 때문에 대부분 결과가 수 초 만에 반환됩니다.

그럼 AWS Athena 환경 구축을 직접 해보며 얼마나 간단한지 확인해 봅시다.

# Athena 설정하기

> 해당 포스트에서는 S3 버킷에 ELB의 Access Log가 수집되고 있음을 가정하고 AWS Athena 설정 방법을 설명합니다.
{: .prompt-info }

## 1. IAM 권한 설정

AWS Athena 콘솔을 사용하기 위해서는 추가적인 IAM 권한 설정이 필요합니다. 다음은 관련된 두개의 정책입니다.

자세한 내용은 [AWS Athena IAM 공식문서](https://docs.aws.amazon.com/ko_kr/athena/latest/ug/managed-policies.html)를 참고해 주세요.

### AWSQuicksightAthenaAccess

Amazon QuickSight와 Athena의 통합에 필요한 작업에 대한 액세스 권한을 부여

- athena : Athena 리소스에 대한 액세스를 허용.
- glue : AWS Glue 데이터베이스, 테이블 및 파티션에 대한 액세스를 허용.
- s3 : Amazon S3에서 쿼리 결과를 읽고 쓰거나, Amazon S3에 상주하는 공개적으로 사용 가능한 Athena 데이터 예제를 읽거나, 버킷을 나열할 수 있도록 허용.
- lakeformation : 보안 주체가 Lake Formation에 등록된 데이터 레이크 위치의 데이터에 액세스하기 위해 임시 자격 증명을 요청하도록 허용.

### AmazonAthenaFullAccess

Athena에 대한 완전한 액세스 권한 부여

- sns : Amazon SNS 주제를 나열하고 주제 속성을 가져올 수 있도록 허용. 이를 통해 모니터링 및 알림 목적으로 Athena에 Amazon SNS 주제를 사용할 수 있음.
- cloudwatch : 보안 주체에게 CloudWatch 경보의 생성, 읽기 및 삭제를 허용.
- +AWSQuicksightAthenaAccess 정책 전부

## 2. S3 설정

Athena의 대상이 될 S3를 지정합니다. 해당 S3의 prefix를 입력하고 저장 버튼을 눌러줍니다.\
여기서 Athena와 S3의 리전은 같아야 합니다.

`쿼리편집기 → 설정 → 관리 → 쿼리 대상 S3 버킷 저장`


![athena s3 setting1](/assets/img/aws_athena_s3_setting1.png)
![athena s3 setting2](/assets/img/aws_athena_s3_setting2.png)

## 3. 데이터베이스, 테이블 생성

편집기에서 아래의 쿼리문을 통해 데이터베이스와 테이블을 생성합니다.

![athena create database](/assets/img/aws_athena_createdatabase.png)

쿼리문 제일 마지막에 있는 location 부분에 S3 환경에 맞추어 수정하고 실행합니다.

``` sql
CREATE EXTERNAL TABLE IF NOT EXISTS elb_log (
        type string,
        time string,
        elb string,
        client_ip string,
        client_port int,
        target_ip string,
        target_port int,
        request_processing_time double,
        target_processing_time double,
        response_processing_time double,
        elb_status_code string,
        target_status_code string,
        received_bytes bigint,
        sent_bytes bigint,
        request_verb string,
        request_url string,
        request_proto string,
        user_agent string,
        ssl_cipher string,
        ssl_protocol string,
        target_group_arn string,
        trace_id string,
        domain_name string,
        chosen_cert_arn string,
        matched_rule_priority string,
        request_creation_time string,
        actions_executed string,
        redirect_url string,
        lambda_error_reason string,
        target_port_list string,
        target_status_code_list string,
        classification string,
        classification_reason string
        )
        ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
        WITH SERDEPROPERTIES (
        'serialization.format' = '1',
        'input.regex' =
        '([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*)[:-]([0-9]*) ([-.0-9]*) ([-.0-9]*) ([-.0-9]*) (|[-0-9]*) (-|[-0-9]*) ([-0-9]*) ([-0-9]*) \"([^ ]*) ([^ ]*) (- |[^ ]*)\" \"([^\"]*)\" ([A-Z0-9-]+) ([A-Za-z0-9.-]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^\"]*)\" ([-.0-9]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^ ]*)\" \"([^\s]+?)\" \"([^\s]+)\" \"([^ ]*)\" \"([^ ]*)\"')
        LOCATION 's3://<s3 위치>/AWSLogs/<계정 ID>/elasticloadbalancing/<region 위치>/';  -- 설정 필요
```

## 4. 데이터 조회

자! 이렇게 AWS Athena 환경 구축이 완료되었습니다.

간단하게 최근 10개 데이터만 조회하는 쿼리문을 실행하여 Athena 환경 구축이 잘 되었는지 확인해 봅시다.

```sql
-- 최근 10개 로그 조회
SELECT *
    FROM elb_log
    ORDER by time DESC
    LIMIT 10;
```

![athena select test](/assets/img/aws_athena_selecttest.png)


# +a 파티션 설정을 통한 최적화


위 사진에서의 제 환경은 약 2년 정도의 로그가 있었습니다.
그리고 사진 우측 아래를 보면 실행시간이 21초나 되고 스캔한 데이터도 2.6GB나 됩니다.

생각보다 성능이 아주아주 많이 느립니다. 겨우 10개의 데이터를 요청한건데 말이죠.

이는 Athena가 모든 로그데이터를 스캔하고 그 중 10개의 데이터를 반환해서 이러한 현상이 발생한 것입니다.

이를 해결하기 위해 AWS는 파티션이라는 설정을 통해 최적화 할 수 있도록 하였습니다.

앞서 말했듯 AWS Athena의 과금 정책은 스캔한 데이터의 양 만큼 청구됩니다.
또한 해당 데이터 양은 응답 속도에도 영향을 주기 때문에 스캔해야하는 데이터가 줄이는 것이 필수적입니다.
데이터를 줄임으로써 성능도 향상시키고 비용도 절감할 수 있는 아주 고마운 친구죠.

그럼 거의 필수라고 할 수 있는 이 파티션 설정을 해봅시다.

## 파티션 프로젝션

- 아래 쿼리문을 통해 테이블을 생성하면 year, month, day 컬럼이 추가되어 데이터가 생성된다.
- 아래의 코드는 AWS에서 제공하는 ELB 파티션 테이블 코드를 사용하였다. [Amazon Athena를 사용하여 Application Load Balancer 액세스 로그를 분석하려면 어떻게 해야 하나요?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs)

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS elb_log_partition_projection (
        type string,
        time string,
        elb string,
        client_ip string,
        client_port int,
        target_ip string,
        target_port int,
        request_processing_time double,
        target_processing_time double,
        response_processing_time double,
        elb_status_code string,
        target_status_code string,
        received_bytes bigint,
        sent_bytes bigint,
        request_verb string,
        request_url string,
        request_proto string,
        user_agent string,
        ssl_cipher string,
        ssl_protocol string,
        target_group_arn string,
        trace_id string,
        domain_name string,
        chosen_cert_arn string,
        matched_rule_priority string,
        request_creation_time string,
        actions_executed string,
        redirect_url string,
        lambda_error_reason string,
        target_port_list string,
        target_status_code_list string,
        classification string,
        classification_reason string
        )
        PARTITIONED BY(year int, month int, day int)
        ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
        WITH SERDEPROPERTIES (
        'serialization.format' = '1',
        'input.regex' =
        '([^ ]*) ([^ ]*) ([^ ]*) ([^ ]*):([0-9]*) ([^ ]*)[:-]([0-9]*) ([-.0-9]*) ([-.0-9]*) ([-.0-9]*) (|[-0-9]*) (-|[-0-9]*) ([-0-9]*) ([-0-9]*) \"([^ ]*) ([^ ]*) (- |[^ ]*)\" \"([^\"]*)\" ([A-Z0-9-]+) ([A-Za-z0-9.-]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^\"]*)\" ([-.0-9]*) ([^ ]*) \"([^\"]*)\" \"([^\"]*)\" \"([^ ]*)\" \"([^\s]+?)\" \"([^\s]+)\" \"([^ ]*)\" \"([^ ]*)\"')
        LOCATION 's3://<s3 위치>/AWSLogs/<계정 ID>/elasticloadbalancing/<region 위치>/';
        TBLPROPERTIES (
            'has_encrypted_data'='false',
            'projection.day.digits'='2',
            'projection.day.range'='01,31',
            'projection.day.type'='integer',
            'projection.enabled'='true',
            'projection.month.digits'='2',
            'projection.month.range'='01,12',
            'projection.month.type'='integer',
            'projection.year.digits'='4',
            'projection.year.range'='2020,2100',
            'projection.year.type'='integer',
            "storage.location.template" = "s3://<s3 위치>/AWSLogs/<계정 ID>/elasticloadbalancing/<region 위치>/${year}/${month}/${day}"
        );
```

아래 사진을 보면 파티션을 설정함으로써 쿼리 실행 시간과 스캔한 데이터의 양이 확연히 줄어든 것을 확인할 수 있다.

또한 사진의 우측에 3개의 컬럼이 추가된 것을 볼 수 있다.

![athena query projection1](/assets/img/aws_athena_queryprojection1.png)
![athena query projection2](/assets/img/aws_athena_queryprojection2.png)
![athena query projection3](/assets/img/aws_athena_queryprojection3.png)

> 추가로 파티션 프로젝션 설정값을 조정하여 테이블 생성 시 해당 테이블에 포함될 데이터의 조건을 걸어서 전체 데이터가 아닌 조건에 맞는 데이터만 추가되도록 설정할 수 있다.\
> ex) 2020년 로그 테이블 설정 : 'projection.year.range'='2020,2020'
{: .prompt-info }

# 마무리

AWS Athena는 구축도 간편하고 쿼리문을 통해 조회를 하여 조건을 걸어 검색하기 편하다는 점에 있어서 장점을 가지는 것 같습니다.

하지만 이름 그대로 AWS에서 제공하는 서비스이기 때문에 AWS에서만 사용이 가능하다는 한계가 있다는 점과
디자인이 이쁘지 않다(?)는 점이 확실한 단점이 될 것 같네요.

포스트 초반에 언급하였던 것처럼 용도와 상황에 맞는 기술스택을 가져가는게 중요한 것 같습니다.


# References

[https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs](https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs)\
[https://docs.aws.amazon.com/athena/latest/ug/partitions.html](https://docs.aws.amazon.com/athena/latest/ug/partitions.html)\
[https://jojoldu.tistory.com/537](https://jojoldu.tistory.com/537)\
[https://aws.amazon.com/ko/blogs/korea/top-10-performance-tuning-tips-for-amazon-athena/](https://aws.amazon.com/ko/blogs/korea/top-10-performance-tuning-tips-for-amazon-athena/)\
[https://medium.com/bunjang-tech-blog/aws-athena를-사용해서-s3-데이터-조회해보자-bb250f1c2b1f](https://medium.com/bunjang-tech-blog/aws-athena를-사용해서-s3-데이터-조회해보자-bb250f1c2b1f)\
[https://stackoverflow.com/questions/51761067/aws-athena-sql-query-error-with-timestamp](https://stackoverflow.com/questions/51761067/aws-athena-sql-query-error-with-timestamp)
