# @ComponentScan



@ComponentScan 어노테이션이 붙어있는&#x20;

applicationContext 가 등록된 클래스에 @ComponentScan 어노테이션이 붙어있으면, 이 클래스가 포함된 패키지부터 그 하위 패키지를 뒤져서 @Component 어노테이션이 붙은 모든 클래스를 빈으로 등록한다.&#x20;
