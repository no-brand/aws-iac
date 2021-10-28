# ELB: Elastic Load Balancer
로드밸런서는 서버들의 상태를 파악해서, 데이터/요청을 분산해서 전달하는 접점의 역할을 수행합니다. <br>
클라이언트는 서버의 타겟을 일일이 지정하는 것이 아닌 로드밸런서를 타겟으로 합니다. 그리고 로드밸런서가 정상적인 서버로
부하를 분산해서 처리합니다. <br>

## ELB 의 구성요소
로드밸런서는 Lisener 및 Target Group 으로 구성됩니다.

|type                    |description                                                                                                                                                                    |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Listener (리스너)       |ELB 가 서비스하는 대상을 정의 <br> - 제공하는 서비스의 Protocol 및 Port 를 이용해서 트레픽 전달 룰을 정의                                                                      | 
|Target Group (대상그룹) |부하 분산 대상을 정의 <br> - 하나 이상의 대상을 라우팅하여 부하 분산을 하는데 사용 <br> - 상태확인 (Health Check) 를 수행하고, 정상적인 상태의 대상에게만 데이터/요청을 전달함 |

## ELB 의 종류
로드밸런서의 종류는 Application Load Balancer (ALB), Network Load Balancer (NLB), Class Load Balancer (CLB) 로 나뉩니다.
 - ALB : Application 기반의 (HTTP, HTTPS) 분산 처리를 제공
   - 상대적으로 처리속도가 느릴 수 있지만, HTTPS 에 대해 다양한 라우팅 정책 제공
     > URL 경로 기반, 호스트 기반, HTTP 헤더 기반 라우팅 등 다양한 규칙 정의 가능
   - Forward, Re-direction 지원 및 HTTPS 응답 정의 가능
 - NLB : 네트워크 기반의 (TCP, UDP) 분산 처리를 제공
   - 빠른 처리속도를 제공
   - 고정 IP를 이용할 수 있고, VPC endpoint 서비스를 연결해서 Private Link 구성할 수 있음
 - CLB : VPC 의 예전 버전인 EC2-Classic 지원

|feature                          |ALB         |NLB           |CLB                   |
|---------------------------------|------------|--------------|----------------------|
|프로토콜                         |HTTP, HTTPS |TCP, UDP, TLS |TCP, TLS, HTTP, HTTPS |
|처리 속도                        |느림        |빠름          |중간                  |
|플랫폼                           |VPC         |VPC           |VPC, EC2-Classic      |
|OSI 계층                         |7           |4             |-                     |
|동일 인스턴스로 다수의 포트 전달 |O           |O             |X                     |
|IP를 이용한 관리                 |X           |O             |X                     | 
|Private Link 지원                |X           |O             |X                     |
|URL 경로 기반 라우팅             |O           |X             |X                     |
|호스트 기반 라우팅               |O           |X             |X                     |

## ELB 특징
### 통신 방식
1. Internet Facing Load Balancer
   Public Address 를 가집니다. 인터넷을 통해서 들어오는 요청을 ELB 에 등록된 서버들로 라우팅 합니다.
2. Internal Load Balancer
   Private Address 를 가집니다. ELB 가 위치한 VPC 에 접근해서 등록된 서버들로 라우팅 합니다.
   
### 그 외 특징들
1. 고가용성 (High Availability) : 외부의 트래픽 (데이터/요청) 을 다수의 대상으로 분산 처리 합니다.
2. 상태 확인 (Health Check) : Target Group 에 대한 Keep-alive 를 주기적으로 확인합니다.
3. 보안그룹 (Security Group) : ALB 에 적용가능 (단, NLB 는 적용 불가)
4. OSI 4/7 계층 지원 : TCP/UDP (4계층), HTTP/HTTPS (7계층) 
5. 실시간 모니터링 지원
