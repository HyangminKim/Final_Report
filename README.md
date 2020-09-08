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

### 비기능적 요구사항 
#### 트랜잭션 
예약된 상품은 상품 정보 상태가 바로 변경되어 중복 예약되어서는 안된다 Sync 호출 </br>
#### 장애격리 
예약 기능이 되지 않더라도 결제는 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency </br>
예약이 과중되면 사용자를 잠시동안 받지 않고 예약을 잠시후에 하도록 유도한다 Circuit breaker, fallback </br>
#### 성능 
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

### 구현
<pre><code>상품상태: 01(available) 02(pending) soldOut
예약상태: 01(reserved) 02(cancle)
결제상태: 01(Unpaid) 02(Paid)</code></pre>

#### 1.판매자가 상품을 등록한다.
<pre><code>http http://product:8080/product productName=Phone productStatus=01
http http://product:8080/product  productName=Tv productStatus=01
http http://product:8080/product  productName=Coffee productStatus=01</code></pre>
결과(product)</br>
![image](https://user-images.githubusercontent.com/61259464/92441979-1f6b7a00-f1ea-11ea-8dba-b8090b0a441f.png)

#### 2.고객은 등록된 상품을 예약한다
<pre><code>http http://reservation:8080/reservation productId=3 reservationStatus=01
http http://reservation:8080/reservation productId=2 reservationStatus=01</code></pre>
결과(Reservation)</br>
![image](https://user-images.githubusercontent.com/61259464/92440584-b1be4e80-f1e7-11ea-95e0-2f13eb430330.png)

#### 3. 예약 후 상품 상태가 변경 된다
결과(Product)</br>
![image](https://user-images.githubusercontent.com/61259464/92440819-0b267d80-f1e8-11ea-9b4f-00e2a858413f.png)

#### 4. 예약 후 결제 정보가 생성 된다
결과(Payment)</br>

![image](https://user-images.githubusercontent.com/61259464/92441104-8e47d380-f1e8-11ea-8f4e-27f4221a190f.png)

### 5. 구매자가 예약한 상품을 결제한다 
<pre><code>http PATCH http://payment:8080/paid id=1 reservationId=1 productId=3 paymentStatus=02</code></pre>
결과(Payment)</br>
![image](https://user-images.githubusercontent.com/61259464/92441249-d666f600-f1e8-11ea-9744-f9e0650b34f8.png)

### 6. 결제 후 상품 정보를 변경한다 
결과( Product)</br>
![image](https://user-images.githubusercontent.com/61259464/92441416-25149000-f1e9-11ea-9258-dd75515370ba.png)

### 7. 구매자가 예약을 취소한다, 8. 취소 후 상품 정보를 상태를 변경한다
<pre><code>http PATCH http://reservation:8080/reservationupdate id=2 productId=1 reservationStatus=02</code></pre>
결과(Reservation) type=Cancled</br>
![image](https://user-images.githubusercontent.com/61259464/92441639-876d9080-f1e9-11ea-90d0-1dd373b5739d.png)

### 9. 구매자는 예약 및 결제 상태를 중간중간 조회한다
결과(MyPage)</br>
![image](https://user-images.githubusercontent.com/61259464/92441867-ec28eb00-f1e9-11ea-9337-15e77c39a93b.png)

운영
============
### 파이프라인
![image](https://user-images.githubusercontent.com/61259464/92431177-c8f24180-f1d1-11ea-9274-d6cbc806b72b.png)

### POD 실행 화면
![image](https://user-images.githubusercontent.com/61259464/92466876-b72d9000-f20b-11ea-8e49-5e6cfd6883fe.png)

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
<pre><code>$ siege -c100 -t60S -r10 --content-type "application/json" 'http://reservation:8080/reservation POST {"productId": "2",, "reservationStatus" : "01" }'</code></pre>
![image](https://user-images.githubusercontent.com/61259464/92451904-534d9c00-f1f8-11ea-9055-7729509478cf.png)


운영시스템은 죽지 않고 지속적으로 CB 에 의하여 적절히 회로가 열림과 닫힘이 벌어지면서 자원을 보호하고 있음을 보여줌. 

### 오토스케일 아웃
상품 서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 15프로를 넘어서면 replica 를 10개까지 늘려준다:
<pre><code>kubectl autoscale deploy product --min=1 --max=10 --cpu-percent=15</code></pre>
CB 에서 했던 방식대로 워크로드를 2분 동안 걸어준다.
<pre><code>$ siege -c100 -t60S -r10 --content-type "application/json" 'http://reservation:8082/reservation POST {"productId": "2", "reservationStatus" : "01" }'</code></pre>

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
