# SNS: Simple Notification Service
알람을 설정/운영/전송 할 수 있도록 지원하는 서비스입니다. <br>
마이크로서비스의 서비스간의 연결을 지원하는 리소스 중 하나이며, 약결합 (Loose Coupling) 으로 통합됩니다. <br>
 - Pub-Sub (Publish, Subscribe) 메시징 패러다임 
 - 푸싱 (Push) 를 기반
 - 실시간 전달 (시간이 관건인 정보의 업데이트에 용이)
<br><br>

### SNS 설정방법
1. SNS Topic (주제) 생성: 이벤트 유형을 식별하는 지점 (메세지를 개시, 알람 구독이 가능한 곳)
2. 구독을 설정: 다양한 알람 프로토콜 지원 (HTTP/HTTPS, 이메일, 문자 등)
<br><br>
   
### 다른 서비스와의 차이점
| resource | 전달 매커니즘                                                                             |
|----------|-------------------------------------------------------------------------------------------|
| SNS      | 푸시 매커니즘 (다수의 구독자에게 시간이 관건인 메세지를 전달)                             |
| SQS      | 폴링 매커니즘 (분산 Application 에서 메세지를 소비, 하나의 분산 구성요소만 동작해도 가능) |
| MQ       | 업계 표준 API, 프로토콜을 지원 (ActiveMQ, RabbitMQ)                                       |

Q. AWS MQ 는 메세지 브로커 (Producer, Consumer) 구성이니, SNS 구성과 동일한지 확인필요
<br><br>

### 요금
사용한 만큼만 지불합니다. <br>
 - SNS 요청: 0.5 USD / 100만건
 - HTTP 알람: 0.06 USD / 10만건
 - 이메일 알람: 2.00 USD / 10만건
<br><br>

### 특징과 기능
1. SNS 주제는 고유한 ARN 으로 나타낼 수 있습니다. (예. arn:aws:sns:us-east-1:1234567890123456:topic)
   - 주제의 이름은 문자, 숫자, 하이픈(-), 밑줄(_) 로 구성
2. 알람 전송, 수신에 사용되는 프로토콜 
   - HTTP/HTTPS: 지정된 URL 로 POST 콜을 수행
   - 이메일/이메일-JSON: 이메일 전달 (JSON 은 이메일 본문 포맷을 의미)
   - SMS  
   - SQS: SQS 로 큐잉할 수 있음
     > Q. SNS 를 SQS 앞에 두고 전달하는 이유가 무엇일까요? <br>
       SNS 로 1:N 메세지를 전달하는데, Subscribe 하는 하나의 서비스가 분산 Application 이라면 좋은 방안
3. SNS 로 게시된 메세지는 취소불가

4. Subscription 필터링 가능
5. 구독자는 경우에 따라 한번 이상의 중복메세지가 발생할 수 있고, 순서가 바뀔 수 있습니다. (네트워트 상황에 따라 SNS 분산 기능의 오류로) <br> 
   - Application 설계시 중복 수신으로 인한 오류가 발생하지 않도록 고려해야 합니다.
   
6. 계정당 Topic 10만, Subscription 1,000만, Filter 200개의 제한이 있습니다.
7. 메시지의 최대크기는 256 KB 까지 (64 KB 청크당 1개의 요청으로 요금이 청구됨)
<br><br>

### Access Control Policy
1. 주제에 대한 전송 프로토콜 선택 가능 (정책으로 제어)
<br><br>

### 모니터링
CloudWatch Metric (지표) 로 모니터링이 가능합니다. <br>
 - NumberOfMessagesPublished
 - NumberOfNotificationsDelivered
 - NumberOfNotificationsFailed
 - PublishSize
<br><br>

### 메세지 형태
HTTP/HTTPS, Email-JSON, SQS 전송 프로토콜을 통해 전달되는 경우, 아래와 같은 구조를 가지고 있습니다. <br>
단, Email 의 경우 Message (본문/페이로드) 만 포함됩니다. <br>

| key              | description                                                                                    |
|------------------|------------------------------------------------------------------------------------------------|
| MessageId        | 고유식별자 (각 알림마다 고유함)                                                                |
| Timestamp        | 알람 게시시간 (GMT)                                                                            |
| TopicArn         | Topic (주제) 의 ARN                                                                            |
| Type             | 전송메세지 유형 (예. Notification)                                                             |
| UnsubscribeURL   | 주제의 구독을 취소하는 링크 (앞으로 알림을 받지 않도록 Subscriber 측에서 구독해지)             |
| Message          | 본문 (페이로드)                                                                                |
| Subject          | 제목 (Publish API 호출시 옵션 파라미터로 제목이 함께 전달된 경우에)                            |
| Signature        | Message, MessageId, Subject (있으면), Type, Timestamp, Topic 값을 Base64 & SHA1withRSA 한 서명 |
| SignatureVersion | SNS 서명의 버전정보                                                                            |

```json
{
  "MessageId": "8ac...ba56",
  "Timestamp": "2021-11-26T07:36:22.818Z",
  "TopicArn": "arn:aws:sns:{region}:{account_id}:{topic_name}",
  "Type": "SubscriptionConfirmation",
  "Message": "You have chosen to subscribe to the topic.\nTo confirm the subscription, visit the SubscribeURL included in this message.",
  "Signature": "...",
  "SignatureVersion": "1",
  "SigningCertURL": "...",
  "Token": "...",
  "SubscribeURL": "..."
}
```
<br><br>

### 모바일 푸시 알람
사용자 경험은 SMS 를 받는것과 유사합니다. <br>
Q. 실제 FCM 과 연동한 예제를 함께 기록할 수 있도록 합니다. <br>
 - Amazon Device Messaging(ADM)
 - Apple Push Notification Service(APNS)
 - Firebase Cloud Messaging(FCM)
 - 중국의 Android 디바이스용 Baidu Cloud Push
<br><br>
