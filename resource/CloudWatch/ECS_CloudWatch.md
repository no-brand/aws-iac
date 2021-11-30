# ECS CloudWatch 로깅

## Metrics 정의
Metric (지표) 란 모니터링 할 변수로, 시간에 따른 변수의 값을 시계열로 모니터링 합니다.<br>
- 생성: 무료 (EC2 instance, EBS volume, RDB instance), 유료 (세부 모니터링), Custom (원하는 순서, 속도로 추가 가능)
- 정의: 깊이로 구성
    - 네임스페이스 (namespace): `AWS/ECS`, `ECS/ContainerInsights`
    - 0개 이상의 차원 (dimension): `ClusterName, ServiceName`
    - 이름 (metric name): `CPUUtilization`, `MemoryUtilization`
- 구성: timestamp, 측정단위 (unit)
- 유지/삭제: 즉시 삭제가 불가능하고, 15개월간 생성된 리전에 유지됩니다.
    - duration (기간) 설정에 따라 유지기간이 달라집니다.
<br>

## Custom Metric 생성
Custom Metric 을 생성할 수 있습니다. <br>
```bash
# Custom NameSpace (MyService) > Metric with no dimensions > Custom Metric Name (PageViewCount)
$ aws cloudwatch put-metric-data --metric-name PageViewCount --namespace MyService --value 2 --timestamp 1637902401

# 여러번 호출하는 대신, 특정시간의 (3초) Aggregation 정보를 한꺼번에 올립니다. (통계횟수 줄이기)
$ aws cloudwatch put-metric-data --metric-name PageViewCount --namespace MyService \
  --statistic-values Sum=11,Minimum=2,Maximum=5,SampleCount=3 --timestamp 1637902401
```
<br>

## ECS CloudWatch Container Insights
Namespace 가 `ECS/ContainerInsights` 입니다. <br>
다양한 Metric 을 이용해서 원하는 값을 만들 수 있습니다. <br>
- CPU 사용률 : CpuUtilized / CpuReserved * 100
- 메모리 사용률 : MemoryUtilized / MemoryReserved * 100

| metric name    | dimensions                  | description                   |
|----------------|-----------------------------|-------------------------------|
| CpuReserved    | ServiceName, ClusterName 등 | 작업에서 예약된 CPU 유닛      |
| CpuUtilized    | ServiceName, ClusterName 등 | 작업에서 사용하는 CPU 유닛    |
| MemoryReserved | ServiceName, ClusterName 등 | 작업에서 예약된 메모리        |
| MemoryUtilized | ServiceName, ClusterName 등 | 작업에서 사용 중인 메모리     |

[Amazon ECS Container Insights 지표](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Container-Insights-metrics-ECS.html)

그리고 Cloudwatch Container Insight 로 수집된 지표는 사용자 지정 지표로 과금됩니다. <br>
[Amazon CloudWatch 요금](https://aws.amazon.com/ko/cloudwatch/pricing/) <br>
<br>
