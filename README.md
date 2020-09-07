기능적 요구사항 
============ 
Markdown 
- 
판매자가 상품을 등록한다 
고객은 등록된 상품을 예약한다
예약 후 상품 상태가 변경 된다
예약 후 결제 정보가 생성 된다
구매자가 예약한 상품을 결제한다
결제 후 상품 정보를 변경한다
구매자가 예약을 취소한다
취소 후 상품 정보를 상태를 변경한다
구매자는 예약 및 결제 상태를 중간중간 조회한다
예약 및 결제 상태가 바뀔 때 마다 알림을 보낸다

### 비기능적 요구사항 
>Blackquotes1 
>>Sub Blackquotes2 

>###header + blackquotes 
트랜잭션
예약된 상품은 상품 정보 상태가 바로 변경되어 중복 예약되어서는 안된다 Sync 호출
장애격리
예약 기능이 되지 않더라도 결제는 365일 24시간 받을 수 있어야 한다 Async (event-driven), Eventual Consistency
예약이 과중되면 사용자를 잠시동안 받지 않고 예약을 잠시후에 하도록 유도한다 Circuit breaker, fallback
성능
고객이 에약내역 및 대여상태를 my-page(프론트엔드)에서 확인할 수 있어야 한다 CQRS
예약 및 결제 상태가 바뀔때마다 카톡 등으로 알림을 줄 수 있어야 한다 Event driven


###Code representation method 
----- 
“`var x = 0“` 

var foo = “bar” 
<html> </html> 

###List representation method 
+ plus 
* star 
- hyphen 

###Numeric list representation method 
1. num1 
1. num2 

### Hyperlink 
[blog](blog.naver.com/tpgns8488) 

First Header | Second Header 
------------ | ------------- 
Content from cell 1 | Content from cell 2 
Content in the first column | Content in the second column 

@tpgns8488 :+1: How to use markdown? :tophat: 

