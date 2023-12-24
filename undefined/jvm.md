# JVM

> 학습 목표
>
> * JVM의 특징
> * JVM 구조
> * JVM 메모리 구조



## Java Virtual Machine

* 자바를 실행하기 위한 가상 컴퓨터
* 운영체제에 종속받지 않고 CPU가 Java를 인식, 실행할 수 있게 하는 가상 컴퓨터

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

java 소스코드, 즉 원시코드(.java)는 cpu가 인식하지 못하므로 기계어로 컴파일이 필요하다. 하지만 java는 JVM이라는  가상 머신을 거쳐 OS에 도달하기 때문에 OS가 인식할 수 있는 기계어로 변경되는게 아니라 JVM이 인식할 수 있는 byte code(.class)로 변환된다.

변환된 bytecode는 기계어가 아니기 때문에 OS에서 바로 실행되지 않는다. JVM이 OS가 bytecode를 이해할 수 있도록 해석해주며, 이 때문에 bytecode는 OS상관없이 JVM위에서 실행될 수 있다.





## 자바 프로그램 실행과정

1.  프로그램 실행 시, jvm은 운영체제로부터 해당 프로그램이 필요로하는 메모리를 할당받는다.

    ㄴ jvm은 이 메모리를 용도에 따라 여러 영역으로 나눠 관리한다.
2. 자바 컴파일러가(javac)가 자바 소스코드(.java)를 읽어 자바 바이트코드(.class)로 변환한다.
3. Class Loader를 통해 class파일을 JVM으로 로딩한다.
4. 로딩된 class 파일들은 Excution Engine을 통해 해석된다.
5. 해석된 바이트코드는 Runtime Data Area에 배치되어 실질적인 수행이 이뤄진다.

위 과정에서 JVM은 필요에 따라 Thred Synchronization과 GC 같은 관리 작업을 수행한다.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## JVM  구성

jvm 구조는 크게 클래스 로더, Runtime data area, Excution Engine, GC로 나눠진다.



*   클래스 로더 (Class Loader)

    * JVM에 클래스(.class)를 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈이다. (runtime data area에 로딩)
    * 클래스를 처음 참조할 때, 해당 클래스를 로드하고 링크하는 역할


*   실행 엔진 (Excution Engine)

    * 클래스를 실행시키는 역할
    * 클래스 로더가 런타임 데이터 영역(runtime data area)에 배치시킨 자바 바이트 코드(.class)를 실행하는 역할
    * 자바 바이트 코드(.class)를 기계가 실행할 수 있는 형태로 변경


*   인터프리터 (Interpreter)

    * 자바 바이트 코드를 명령어 단위로 읽어서 실행
    * 한 줄씩 실행하기 때문에 느리다는 단점 존재


* JIT (just-in-time)
  * 인터프리터 방식의 단점을 보완하는 역할
  *
* 가비지 콜렉터 (Garbage Collector)
  * 더 이상 사용되지 않는 인스턴스를 찾아 메모리에서 삭제

