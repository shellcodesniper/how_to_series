---
title: 지갑 없이 접속하기
category: oracle-cloud
sub_category: AUTONOMOUS DB
order: 1
id: 3
# permalink: /NestJs/docker
# description: 도커를 이용한 Nestjs 배포 가이드
# parent: NESTJS
# parentUrl: /NestJs/docker/
# subparent: Configuration
# subparentUrl: /Prometheus/config/
# priority: 1
---



![Oracle DB 11g Errors Guide:Amazon.ca:Appstore for Android](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129144423uGRqE9_41QodfboFdL.png)



## 서론

맥과 DBEaver를 이용하여 암호 없이 Autonomous DB 에 접속하려면 어떠한 요구조건이 필요하고, 어떻게 접속하는지 확실히 설명해주는 한글 문서를 찾지 못하여 추가하였습니다.







## 사전 작업



### 1. 데이터베이스 생성

![KUUWANGE_-9701788](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/2022112915030914ncif_KUUWANGE_-9701788.png)

AUTONOMOUS DB를 생성했음을 전제로,

mTLS ( 전자 지갑 인증 ) 없이 접속 하고자 하시면, 한가지 더 설정을 해주셔야합니다



## 2. 네트워크 액세스 제어 목록 추가 ( IP 제한 )

![KUUWANGE_-9701844](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129150404UEyyC8_KUUWANGE_-9701844.png)







## 3.  DB 접속 문자열 복사

 <img style="height: 40px" src="https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129150500LKiaQ5_KUUWANGE_-9701900.png">

버튼을 클릭하여, 상호TLS -> TLS 로 변경해주세요

 	

![KUUWANGE_-9701971](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129150611g4vrMK_KUUWANGE_-9701971.png)





### 4. 접속 문자열 복사

![KUUWANGE_-9702002](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129150642UarUXR_KUUWANGE_-9702002.png)

TLS 이름 _high, _low, _medium 등 선택할 내용을 아직 모르시겠다 하면 일단 첫번째를 복사해보죠..?



### 5. 접속 문자열 만들기 ( 핵심.. ? )

```apl
jdbc:oracle:thin:@(description= (retry_count=20)(retry_delay=3)(address=(protocol=tcps)(port=1521)(host=adb.ap-seoul-1.oraclecloud.com))(connect_data=(service_name=asdasdasdas.adb.oraclecloud.com))(security=(ssl_server_dn_match=yes)))
```

사실 이게 핵심이라 할 수 있습니다.

`jdbc:oracle:thin:@` 을 앞에 붙여주시고, 그 뒤에 복사하신 접속 문자열을 붙여넣어주면 뚝딱하고 만들어지죠





### 6.DBEAVER 설정

차례로 누르시고 붙여넣기만 하시면..!



![KUUWANGE_-9702315](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129151155JUwo5A_KUUWANGE_-9702315.png)



**Connection Type : Custom** 으로 설정해주시고, JDBC URL Template에 새로 만든 jdbc connect string을 넣어줍니다.

![KUUWANGE_-9702349](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129151230UrHJQF_KUUWANGE_-9702349.png)



![KUUWANGE_-9702409](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129151329osJpm5_KUUWANGE_-9702409.png)

적절하게 username, password 채워주시고?

Test Connection을 눌러보시면 완성.



![KUUWANGE_-9702424](https://bdev-s3.s3.ap-northeast-2.amazonaws.com/pics/20221129151344TgM1yk_KUUWANGE_-9702424.png)
