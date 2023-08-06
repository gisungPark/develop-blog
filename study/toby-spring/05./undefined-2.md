# 인프라 빈 구성 정보의 분리

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>



<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

`@EnableMyAutoConfiguration` 애노테이션을 추가하고, `@MySpringBootApplication`에는 `@EnableMyAutoConfiguration` 만을 붙여 어떤 클래스를 autoConfiguration 할 지를 위임한다.

`@MySpringBootApplication` 을 단순화했다.

