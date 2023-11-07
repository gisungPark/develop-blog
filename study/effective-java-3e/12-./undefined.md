# 자바 직렬화

## 직렬화, 역직렬화

* 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트 형태의 데이터로 변환하는 기술과&#x20;
* 바이트로 변환된 객체를 다시 변환하는 기술(역직렬화)를 아울러서 이야기한다.
* 시스템 적으로 이야기 하자면 JVM의 메모리에 상주하는 객체 데이터를 바이트 형태로 변환하는 기술과
* 직렬화된 바이트 형태의 데이터를 객체로 변환해서 JVM으로 상주시키는 형태를 같이 이야기합니다.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

\*바이트 스트림; 스트림은 클라이언트와 서버 사이 출발지, 목적지로 입출력하기 위한 **데이터가 흐르는 통로를 말한다**. 자바는 스트림의 기본 단위를 바이트로 두고 있기 때문에 네트워크 또는 데이터베이스로 전송하기 위한 최소 단위인 바이트 스트림으로 변환하여 처리한다.

### 직렬화 예제 코드

`Serializable` 인터페이스를 상속받은 객체는 직렬화 할 수 있는 기본 조건을 가집니다.

```java
Member member = new Member("김배민", "deliverykim@baemin.com", 25);
byte[] serializedMember;
try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(member);
        // serializedMember -> 직렬화된 member 객체 
        serializedMember = baos.toByteArray();
    }
}
// 바이트 배열로 생성된 직렬화 데이터를 base64로 변환
System.out.println(Base64.getEncoder().encodeToString(serializedMember));
}
```

### 역직렬화 예제 코드

* 직렬화 대상이 된 객체의 클래스가 클래스 패스에 존재해야 하며 import 되어 있어야 합니다.
  * 중요한 점은 직렬화와 역직렬화를 진행하는 시스템이 서로 다를 수 있다는 것을 반드시 고려해야 합니다.
* 자바 직렬화 대상 객체는 동일한 serialVersionUID를 가지고 있어야 합니다.

```
  private static final long serialVersionUID = 1L;
```

```java
 // 직렬화 예제에서 생성된 base64 데이터 
String base64Member = "...생략";
byte[] serializedMember = Base64.getDecoder().decode(base64Member);
try (ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember)) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
        // 역직렬화된 Member 객체를 읽어온다.
        Object objectMember = ois.readObject();
        Member member = (Member) objectMember;
        System.out.println(member);
    }
}
```



## 자바의 직렬화가 사용되는 이유

데이터 직렬화의 종류 3가지

### 문자열 형태의 직렬화 방법

직접 데이터를 문자열 형태로 확인 가능한 직렬화 방법입니다. 범용적인 API나 데이터를 변환하여 추출할때 많이 사용합니다. 표 형태의 다량의 데이터를 직렬화시 CSV가 많이 쓰이고 구조적인 데이터는 이전에는 xml을 많이 사용했으며 최근에는 JSON 형태를 많이 사용하고 있습니다.

* CSV
  * 데이터를 표현하는 데 가장 많이 사용되는 방법 중 하나로 콤마(,) 기준으로 데이터를 구분하는 방법입니다.
* JSON
  * 최근에 가장 많이 사용하는 포맷으로 쉽게 사용가능하고, 다른 데이터 포맷 방식에 비해 오버헤드가 적습니다.

### 이진 직렬화 방법









그럼 본론으로 돌아와 왜 자바 직렬화를 사용할까? CSV, JSON, 프로토콜 버퍼등은 시스템의 고유 특성과 상관없는 대부분의 시스템에서의 데이터 교화시 많이 사용됩니다. 하지만 "자바 직렬화 형태의 데이터 교환은 <mark style="background-color:red;">**"자바 시스템 간의 데이터 교환을 위해서 존재한다"**</mark>고 생각하시면 됩니다.

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt="" width="298"><figcaption></figcaption></figure>

### 그럼 자바에서도 CSV, JSON을 사용하면 되지 자바 직렬화를 써야 되는 이유가 있나요?

"목적에 따라 적절하게 써야 한다"

* 자바 직렬화의 장점
  * 자바 직렬화는 자바 시스템에서 최적화 되어 있다.\
    복잡한 데이터 구조의 클래스 객체라도 직렬화 조건만 지키면 큰 작업 없이 직렬화 가능하다. 또한 데이터 타입이 자동으로 맞춰지기 때문에 관련 부분을 신경쓰지 않아도 된다.







## 출처

{% embed url="https://techblog.woowahan.com/2550/" %}

{% embed url="https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0" %}

{% embed url="https://m.blog.naver.com/kkson50/220564174258" %}
