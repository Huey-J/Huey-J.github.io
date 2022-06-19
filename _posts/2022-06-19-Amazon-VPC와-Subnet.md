---
title: Amazon VPC와 Subnet
author: Huey-J
date: 2022-06-19 12:00:00 +0800
categories: [데브옵스]
tags: [AWS]
---


# 들어가기 전에

- CIDR 블록: 
- NAT 게이트웨이: private 서브넷의 인스턴스가 인터넷이나 다른 VPC 등에 연결되도록 허용하는 퍼블릭 서브넷의 EC2 인스턴스를 말한다.


# Virtual Private Cloud

VPC(Virtual Private Cloud)는 말 그대로 사용자가 정의하는 가상의 네트워크입니다. VPC를 통해 인스턴스가 속한 네트워크를 구분하여 각 네트워크에 맞는 설정을 부여할 수 있습니다. 네트워크는 공인 IP 주소와 사설 IP 주소를 갖고 있습니다. AWS는 VPC의 중요성을 강조하여 2019년 부터 모든 서비스에 VPC를 적용하도록 강제하였습니다. VPC를 만들지 않고 인스턴스를 생성하더라도 AWS가 제공하는 Default VPC에 인스턴스가 배치되어 있을 겁니다.

- VPC
  - VPC 안에 서브넷을 나눈다.
  - 지역을 서브넷을 기준으로 나눈다.
  - 라우팅 테이블:
  - IKE 버전
    - Ikev2 : 터널 연결이 끊기면 은행에서 연결요청을 해줘야 함
    - Ikev1 : 우리쪽에서 연결 가능


# 서브넷

거대한 네트워크 대역의 VPC 주소를 잘개 쪼갠 네트워크 주소를 이야기 합니다.


## 기대효과

1. IP주소를 효율적으로 사용하기 위해 서비스를 목적별로 분리하여 효율적으로 운영할 수 있습니다.
2. 네트워크를 분리하여 외부에 노출되선 안되는 서비스를 별도로 관리하고 보안성을 강화할 수 있습니다.


## 종류

- Public Subnet: 내부에 Internet Gateway, ELB, Public IP/Elastic IP를 가진 인스턴스를 생성할 수 있으며 외부(인터넷)와 통신할 수 있습니다.
- Private Subnet: 외부와 차단되어 있으며 해당 서브넷에 속한 인스턴스들은 Private IP만을 가지고 있습니다.
- VPN 전용 Subnet:



# References

VPC 및 서브넷 개요 - [https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Subnets.html](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Subnets.html)\
Amazon VPC란 무엇인가? - [https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)\
VPC 개념 이해하기 - [https://jbhs7014.tistory.com/164](https://jbhs7014.tistory.com/164)