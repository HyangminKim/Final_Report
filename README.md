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

####1.판매자가 상품을 등록한다.
<pre><code>http http://localhost:8081/product productName=Noodle productStatus=01
http http://localhost:8081/product productName=Desk productStatus=01
http http://localhost:8081/product productName=Coffee productStatus=01</code></pre>


