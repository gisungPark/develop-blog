# 3장. 모든 객체의 공통 메서드

```java
Object는 객체를 만들 수 있는 구체 클래스지만 기본적으로는 상속해서 사용하도록 설계되었다. 
Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize)는 모두
재정의를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.

이번 장에서는 final이 아닌 Object메서드들을 언제 어떻게 재정의해야 하는지를 다룬다. 
```

##
