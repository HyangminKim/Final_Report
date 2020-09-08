서비스시나리오
============ 
### 기능적 요구사항 
1. 판매자가 상품을 등록한다 </br>
2. 고객은 등록된 상품을 예약한다 </br>
3. 예약 후 상품 상태가 변경 된다 </br>
4. 예약 후 결제 정보가 생성 된다 </br>
5. 구매자가 예약한 상품을 결제한다 </br>
6. 결제 후 상품 정보를 변경한다 </br>
7. 구매자가 예약을 취소한다 </br>
8. 취소 후 상품 정보를 상태를 변경한다 </br>
9. 구매자는 예약 및 결제 상태를 중간중간 조회한다 </br>
10. 예약 및 결제 상태가 바뀔 때 마다 알림을 보낸다 </br>

### 비기능적 요구사항 
트랜잭션 </br>
예약된 상품은 상품 정보 상태가 바로 변경되어 중복 예약되어서는 안된다 Sync 호출 </br>
장애격리 </br>
예약 기능이 되지 않더라도 결제는 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency </br>
예약이 과중되면 사용자를 잠시동안 받지 않고 예약을 잠시후에 하도록 유도한다 Circuit breaker, fallback </br>
성능 </br>
고객이 에약내역 및 대여상태를 my-page(프론트엔드)에서 확인할 수 있어야 한다 CQRS </br>
예약 및 결제 상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다 Event driven </br>


분석설계
============ 
MSAEz 로 모델링한 이벤트스토밍 결과: <br>
http://msaez.io/#/storming/cVNuZs0oJidKR8D7ZWW4anjQhQA2/mine/83e2fd722bb1a93822e7e29d47db0227/-M5Tg016oQ03UwFRn94Q

![image](https://user-images.githubusercontent.com/61259464/92362165-8d506c80-f12a-11ea-8079-50401ef08a83.png)



구현
============ 
분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8080 ~ 8085 이다)
분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

cd Reservation<br>
mvn spring-boot:run</br>
예약 시스템: https://github.com/HyangminKim/Final_Reservation</br>

cd Payment<br>
mvn spring-boot:run</br>
결제 시스템: https://github.com/HyangminKim/Final_Payment</br>

cd Product<br>
mvn spring-boot:run</br>
등록 시스템: https://github.com/HyangminKim/Final_Product</br>

cd Notice</br>
mvn spring-boot:run</br>
알림 시스템: https://github.com/HyangminKim/Final_Notice</br>

cd MyPage </br>
mvn spring-boot:run </br>
마이 페이지 : https://github.com/HyangminKim/Final_MyPage</br>   


<pre><code>Product Status: 01(available) 02(pending) soldout
Reservation Status: 01(reserved) 02(cancle)
Payment Status: 01(unpaid) 02(paid)</code></pre>

#### 1.판매자가 상품을 등록한다.
<pre><code>http http://localhost:8081/product productName=Noodle productStatus=01
http http://localhost:8081/product productName=Desk productStatus=01
http http://localhost:8081/product productName=Coffee productStatus=01</code></pre>
결과(product)</br>
![image](https://user-images.githubusercontent.com/61259464/92427678-136ec080-f1c8-11ea-97ac-109b52be95d7.png)

#### 2.고객은 등록된 상품을 예약한다
<pre><code>http http://localhost:8082/reservation productId=1 reservationStatus=01
http http://localhost:8082/reservation productId=2 reservationStatus=01</code></pre>
결과(product)</br>


운영
============
### 파이프라인
![image](https://user-images.githubusercontent.com/61259464/92431177-c8f24180-f1d1-11ea-9274-d6cbc806b72b.png)

### POD 실행 화면
![image](https://user-images.githubusercontent.com/61259464/92436869-aa477700-f1e0-11ea-8fa9-2ab6736ecdd5.png)

### 구현
<pre><code>http http://localhost:8081/product productName=Noodle productStatus=01
http http://localhost:8081/product productName=Desk productStatus=01
http http://localhost:8081/product productName=Coffee productStatus=01</code></pre>
결과(product)</br>

### CI/CD 설정
각 구현체들은 각자의 Git을 통해 빌드되며, Git Master에 트리거 되어 있다. pipeline build script 는 각 프로젝트 폴더 이하에 azure_pipeline.yml 에 포함되었다.
azure_pipeline.yml 참고

### 동기식 호출 / 서킷 브레이킹 / 장애격리
서킷 브레이킹 프레임워크의 선택: Spring FeignClient + Hystrix 옵션을 사용하여 구현함</br>
시나리오는 예약(Reservation)-->상품상태 변경(Product) 시의 연결을 RESTful Request/Response 로 연동하여 구현이 되어있고, 결제 요청이 과도할 경우 CB 를 통하여 장애격리</br>
Hystrix 를 설정: 요청처리 쓰레드에서 처리시간이 610 밀리가 넘어서기 시작하여 어느정도 유지되면 CB 회로가 닫히도록 (요청을 빠르게 실패처리, 차단) 설정
<pre><code># application.yml

hystrix:
  command:
    # 전역설정
    default:
      execution.isolation.thread.timeoutInMilliseconds: 610</code></pre>

피호출 서비스(Product) 의 임의 부하 처리 - 400 밀리에서 증감 220 밀리 정도 왔다갔다 하게
<pre><code># (Product) Product.java (Entity)

    @PostUpdate
    public void onPostUpdate(){  //상품 상태를 변경한 후 적당한 시간 끌기

        ...
        
        try {
            Thread.currentThread().sleep((long) (400 + Math.random() * 220));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }</code></pre>
    
부하테스터 siege 툴을 통한 서킷 브레이커 동작 확인 </br>
동시사용자 100명</br>
60초 동안 실시</br>
<pre><code>$ siege -c100 -t60S -r10 --content-type "application/json" 'http://reservation:8080/reservation POST {"productId": "2",, "reservationStatus" : "01" }'

Lifting the server siege...
Transactions:                   1067 hits
Availability:                  74.91 %
Elapsed time:                  59.46 secs
Data transferred:               0.37 MB
Response time:                  5.36 secs
Transaction rate:              17.94 trans/sec
Throughput:                     0.01 MB/sec
Concurrency:                   96.13
Successful transactions:        1067
Failed transactions:             285
Longest transaction:            7.01
Shortest transaction:           0.02</code></pre>

운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌. 하지만, 74.21% 가 성공.
### 오토스케일 아웃
앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다.

상품 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 15프로를 넘어서면 replica 를 10개까지 늘려준다:
<pre><code>kubectl autoscale deploy product --min=1 --max=10 --cpu-percent=15</code></pre>
CB 에서 했던 방식대로 워크로드를 2분 동안 걸어준다.
<pre><code>$ siege -c100 -t60S -r10 --content-type "application/json" 'http://reservation:8082/reservation POST {"productId": "2", "reservationStatus" : "01" }'</code></pre>
오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다:
<pre><code>kubectl get deploy pay -w</code></pre>
어느정도 시간이 흐른 후 (약 30초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:
<pre><code>NAME    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
pay     1         1         1            1           17s
pay     1         2         1            1           45s
pay     1         4         1            1           1m
:</code></pre>
siege 의 로그를 보아도 전체적인 성공률이 높아진 것을 확인 할 수 있다.
<pre><code>Transactions:		        5078 hits
Availability:		       92.45 %
Elapsed time:		       120 secs
Data transferred:	        0.34 MB
Response time:		        5.60 secs
Transaction rate:	       17.15 trans/sec
Throughput:		        0.01 MB/sec
Concurrency:		       96.02</code></pre>

### 무정지 재배포
모든 프로젝트의 readiness probe 및 liveness probe 설정 추가
<pre><code>readinessProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  initialDelaySeconds: 10
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 10
livenessProbe:
  httpGet:
     path: /actuator/health
     port: 8080
  initialDelaySeconds: 120
  timeoutSeconds: 2
  periodSeconds: 5
  failureThreshold: 5
  </code></pre>
