---
title: AWS Athena
author: Huey-J
date: 2022-08-20 12:00:00 +0800
categories: [데브옵스, AWS]
tags: [AWS, DevOps, Elastic Stack]
---


# Athena 설정

이번 포스트에서는 현재 S3 버킷에 ELB의 Access Log가 수집되고 있음을 가정하고 AWS Athena 설정 방법을 설명합니다.


## 1. IAM 권한 설정

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

쿼리편집기 → 설정 → 관리 → 쿼리 대상 S3 버킷 저장

![athena s3 setting1](/assets/img/aws_athena_s3_setting1.png)
![athena s3 setting2](/assets/img/aws_athena_s3_setting2.png)

## 3. 데이터베이스, 테이블 생성

편집기에서 아래의 쿼리문을 통해 데이터베이스와 테이블을 생성한다.

![athena create database](/assets/img/aws_athena_createdatabase.png)

쿼리문 중간에 있는 location을 S3 환경에 맞추어 수정한다.

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

```sql
-- 최근 10개 로그 조회
SELECT *
        FROM elb_log
        ORDER by time DESC
        LIMIT 10;
```

![athena select test](/assets/img/aws_athena_selecttest.png)


# 파티션 설정을 통한 최적화 (필수)

AWS Athena의 과금 정책은 스캔한 데이터의 양 만큼 청구된다. 또한 해당 데이터 양은 응답 속도에도 영향을 주기 때문에 스캔해야하는 데이터가 줄이는 것이 필수적이다. 따라서 Athena에서는 파티션을 통해 해당 데이터를 줄일 수 있도록 하였다.

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


# References

[https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs](https://aws.amazon.com/ko/premiumsupport/knowledge-center/athena-analyze-access-logs)\
[https://docs.aws.amazon.com/athena/latest/ug/partitions.html](https://docs.aws.amazon.com/athena/latest/ug/partitions.html)\
[https://jojoldu.tistory.com/537](https://jojoldu.tistory.com/537)\
[https://aws.amazon.com/ko/blogs/korea/top-10-performance-tuning-tips-for-amazon-athena/](https://aws.amazon.com/ko/blogs/korea/top-10-performance-tuning-tips-for-amazon-athena/)\
[https://medium.com/bunjang-tech-blog/aws-athena를-사용해서-s3-데이터-조회해보자-bb250f1c2b1f](https://medium.com/bunjang-tech-blog/aws-athena를-사용해서-s3-데이터-조회해보자-bb250f1c2b1f)\
[https://stackoverflow.com/questions/51761067/aws-athena-sql-query-error-with-timestamp](https://stackoverflow.com/questions/51761067/aws-athena-sql-query-error-with-timestamp)