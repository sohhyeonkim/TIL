## 도메인 모델 패턴

일반적인 애플리케이션 아키텍쳐는 표현, 응용, 도메인, 인프라스트럭처로 구성된다. 

- 표현 영역은 (사용자 인터페이스 또는 표현) 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여기서 사용자는 소프트웨어를 사용하는 사람 뿐만 아니라 외부 시스템일 수도 있다.
- 응용 영역은 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다. 
- 도메인 영역은 시스템이 제공할 도메인 규칙을 구현한다.
- 인프라스트럭처는 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.


지금부터 살펴볼 도메인 모델은 마틴 파울러가 쓴 <엔터프라이즈 애플리케이션 아키택처 패턴> 책의 도메인 모델 패턴을 의미한다. 도메인 모델은 아키텍처 상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 말한다. 

도메인 계청은 도메인의 핵심 규칙을 구현한다. 주문 도메인의 경우 '출고 전에 배송지를 변경할 수 있다'라는 규칙과 '주문취소는 배송 전에만 할 수 있다'라는 규칙을 구현한 코드가 도메인 계층에 위치하게 된다. 이런 도메인 규칙을 객체 지향 기법으로 구현하는 패턴이 도메인 모델 패턴이다. 

```java
public class Order {
  private OrderState state;
  private ShippingInfo shippingInfo;

  public void changeShippingInfo(ShippingInfo newShippingInfo) {
    if(!state.isShippingChangeable()) {
      throw new IllegalStateException("can't change shipping in "+ state);
    }
    this.shippingInfo = newShippingInfo;
  }
}

public enum OrderState {
  PAYMENT_WAITING {
    public boolean isShippingChangeable() {
      return true
    }
  }, 
  PREPARING {
    public boolean isShippingChangeable() {
      return true;
    }
  },
  SHIPPED, DELIVERING, DELIVERY_COMPLETED;

  public boolean isShippingChangeable() {
    return false;
  }
}
```
위의 코드는 주문 도메인의 일부 기능을 도메인 모델 패턴으로 구현한 것이다. 주문 상태를 표현하는 OrderState는 배송지를 변경할 수 있는지를 검사할 수 있는 isShippingChangeable() 메서드를 제공하고 있다. 코드를 보면 주문대기중 상태와 상품 준비중 상태의 메서드는 true를 리턴한다. 즉, OrderState는 주문 대기중이거나 상품 준비중에는 배송지를 변경할 수 있다는 도메인 규칙을 구현하고 있다.

실제 배송지 정보를 변경하는 Order 클래스의 changeShippingInfo() 메서드는 OrderState의 isShippingChangeable() 메서드를 이용해서 변경 가능한 경우에만 배송지를 변경한다.

큰 틀에서 보면 OrderState는 Order에 속한 데이터이므로 배송지 정보 변경 가능 여부를 판단하는 코드를 Order로 이동할 수도 있다. 

배송지 변경이 가능한지를 판단할 규칙이 주문상태와 다른 정보를 함께 사용한다면 OrderState만으로는 배송지 변경 가능 여부를 판단할 수 없으므로 Order에서 로직을 구현해야한다. 

배송지 변경 가능 여부를 판단하는 기능이 Order에 있든 OrderState에 있든 중요한 점은 주문과 관련된 중요 업무 규칙을 주문 도메인 모델인 Order나 OrderState에서 구현한다는 점이다. 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다. 

## 도메인 모델 도출

도메인에 대한 이해 없이 코딩을 시작할 수 없다. 기획서, 유스케이스, 사용자 스토리와 같은 요구사항과 관련자와의 대화를 통해 도메인을 이해하고 이를 바탕으로 도메인을 이해하고 이를 바탕으로 도메인 모델 초안을 만들어야 비로소 코드를 작성할 수 있다. 화이트보드, 종이와 연필, 모델링 틀 중 무엇을 선택하든 구현을 시작하려면 도메인에 대한 초기 모델이 필요하다. 주문 도메인과 관련된 몇가지 요구사항을 보자.

- 최소 한 종류 이상의 상품을 주문해야 한다.
- 한 상품을 한 개 이상 주문할 수 있다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
- 각 상품의 구매 가격의 합은 상품 가격에 구매 개수를 곱한 값이다.
- 주문할 때 배송지 정보를 반드시 지정해야 한다.
- 배송지 정보는 받는 사람 이름, 전화번호, 주소로 구성된다.
- 출고를 하면 배송지를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다.
- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

이 요구 사항에서 알 수 있는 것은 주문은 '출고 상태로 변경하기', '배송지 정보 변경하기', '주문 취소하기', '결제 완료하기' 기능을 제공한다는 것이다. 아직 상세 구현까지 할 수 있는 수준은 아니지만 Order 관련 기능을 메서드로 추가할 수 있다. 

```java
public class Order {
  public void changeShipped() {}
  public void changeShippingInfo(ShippingInfo newShipping){}
  public void cancel() {}
  public void completePayment() {}
}
```
다음 요구사항은 주문 항목이 어떤 데이터로 구성되는지 알려준다.

-  한 상품을 한 개 이상 주문할 수 있다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.

두 요구사항에 따르면 주문 항목을 표현하는 OrderLine은 적어도 주문할 상품, 상품의 가격, 구매 개수를 포함해야한다. 추가로 각 구매 항목의 구매 가격도 제공해야 한다. 이를 구현한 OrderLine은 다음과 같다.

```java
public class OrderLine {
  private Product product;
  private int price;
  private int quantity;
  private int amounts;

  public OrderLine(Product product, int pirce, int quantity) {
    this.product = product;
    this.price = price;
    this.quantity = quantity;
    this.amounts = calculateAmounts();
  }

  private int calculateAmounts() {
    return price * quantity;
  }

  private int getAmounts() { /*...*/}
}
```

OrderLine은 한 상품(pruduct 필드)을 얼마(price)에 몇 개(quantity) 살지를 담고 있고 calculateAmounts() 메서드로 구매 가격을 구하는 로직을 구현하고 있다.

한 종류 이상의 상품을 주문할 수 있으므로 Order는 최소 한 개 이상의 OrderLine을 포함해야한다. 또한 총 주문 금액은 OrderLine에서 구할 수 있다. 두 요구사항은 Order에 다음과 같이 반영할 수 있다.

```java
public class Order {
  private List<OrderLine> orderLines;
  private Monty totalAmounts;

  public Order(List<OrderLine> orderLines) {
    setOrderLines(orderLines);
  }

  private void setOrderLines(List<OrderLine> orderLines) {
    verifyAtLeastOneOrMoreOrderLines(orderLines);
    this.orderLines = orderLines;
    calculateTotalAmounts();
  }

  private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
    if(orderLines == null || orderLines.isEmpty()) {
      throw new IllegalArgumentException("no OrderLine");
    }
  }

  private void calculateTotalAmounts() {
    int sum = orderLines.stream()
                        .mapToInt(x => x.getAmounts())
                        .sum();
    this.totalAmounts = new Money(sum);
  }
}
```

Order는 한 개 이상의 OrderLine을 가질 수 있으므로 Order를 생성할 때 OrderLine 목록을 List로 전달한다. 생성자에서 호출하는 setOrderLines() 메서드는 요구사항에 정의된 제약 조건을 검사한다. 요구사항에 따르면 최소 한 종류 이상의 상품을 주문해야 하므로 verifyAtLeastOneOrMoreOrderLines() 메서드를 이용해서 OrderLine이 한 개 이상 존재하는 지 검사한다. 또한 calculateTotalAmounts() 메서드를 이용해서 총 주문 금액을 계산한다.


배송지 정보는 이름, 전화번호, 주소 데이터를 가지므로 ShippingInfo 클래스를 다음과 같이 정의할 수 있다. 

```java 
public class ShippingInfo {
  private String receiverName;
  private String receiverPhoneNumber;
  private String shippingAddress1;
  private String shippingAddrerss2;
  private String shippingZipcode;

  // ...생성자, getter
}
```

앞서 요구사항 중에 '주문할 때 배송지 정보를 반드시 저장해야한다'라는 내용이 있다. 이는 Order를 생성할 때 OrderLine의 목록 뿐만 아니라 ShippingInfo도 함께 전달해야 함을 의미한다. 이를 생성자에 반영한다.

```java 
public class Order {
  private List<OrderLine> orderLines;
  private Monty totalAmounts;
  private ShippingInfo shippingInfo;

  public Order(List<OrderLine> orderLines) {
    setOrderLines(orderLines);
    setShippingInfo(shippingInfo);
  }

  private void setOrderLines(List<OrderLine> orderLines, ShippingInfo shippingInfo) {
    verifyAtLeastOneOrMoreOrderLines(orderLines);
    this.orderLines = orderLines;
    calculateTotalAmounts();
  }

  private void setShippingInfo(ShippingInfo shippingInfo) {
    if(shippingInfo == null) {
      throw new IllegalArgumentException("no shippingInfp");
    }
    this.shippingInfo = shippingInfo;
  }

  private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
    if(orderLines == null || orderLines.isEmpty()) {
      throw new IllegalArgumentException("no OrderLine");
    }
  }

  private void calculateTotalAmounts() {
    int sum = orderLines.stream()
                        .mapToInt(x => x.getAmounts())
                        .sum();
    this.totalAmounts = new Money(sum);
  }
}
```

생성자에서 호출하는 setShippingInfo() 메서드는 ShippingInfo가 null이면 익셉션이 발생하는데, 이렇게 함으로써 '배송지 정보 필수'라는 도메인 규칙을 구현한다.

도메인을 구현하다 보면 특정 조건이나 상태에 따라 제약이나 규칙이 달리 적용되는 경우가 많다. 주문 요구 사항에서는 다음 내용이 제약과 규칙에 해당된다. 

- 출고를 하면 배송지 정보를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 없다. 
- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다. 

위의 세가지 요구사항에 따르면, 출고 상태에 따라 배송지 정보 변경기능이 필요하며 결제 여부 또한 알 수 있어야 한다. 따라서, 이 요구 사항들은 결제 완료 전을 의미하는 상태와 결제 완료 내지 상품 준비중, 배송중 등의 상태를 표현할 수 있어야 한다. 다른 요구 사항을 확인해서 추가로 존재할 수 있는 상태를 분석한 뒤, 다음과 같이 열거 타입을 이용해서 상태 정보를 표현할 수 있다. 

```java
public enum OrderState {
  PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED,CANCELED;
}
```

배송지 변경이나 주문 취소 기능은 출고 전에만 가능하다는 제약 규칙이 있으므로 이 규칙을 적용하기 위해 changeShippingInfo()와 cancel()은 verifyNotYetShipped() 메서드를 먼저 실행한다. 

```java
public class Order {
  private OrderState state;

  public void changeShippingInfo(ShippingInfo shippingInfo) {
    verifyNotYetShipped();
    setShippingInfo(newShippingInfo);
  }

  public void cancel() {
    verifyNotYetShipped();
    this.state = OrderState.CANCELED;
  }

  private void verifyNotYetShipped() {
    if(state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
      throw new IllegalStateException("already shipped");
    }
  }
}
```

지금까지 주문과 관련된 요구사항에서 도메인 모델을 점진적으로 만들어 나갔다. 일부는 구현 수준까지 만들었고 일부는 이름 정도만 결정했다. 이렇게 만든 모델은 요구사항 정련을 위해 도메인 전문가나 다른 개발자와 논의하는 과정에서 공유하기도 한다. 

## 엔티티와 벨류

도출한 모델은 크게 엔티티와 벨류로 구분할 수 있다. 앞서 요구사항 분석 과정에서 만든 모델도 엔티티와 벨류로 구분할 수 있다. 

