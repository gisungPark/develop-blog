# Runtim Data Area

## Runtime Data Area

프로그램을 수행하기 위해 운영체제에게 할당받은 메모리 공간

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

* PC Register
  * 스레드가 시작될 때 생성되며,  스레드마다 하나씩 존재
  * 스레드가 어떤 부분을 어떤 명령으로 실행해야할지에 대한 기록을 저장
* JVM 스택 영역
  * 프로그램 실행과정에서 임시로 할당되었다가 메소드를 빠져나가면 바로 소멸되는 특성의 데이터를 저장하기 위한 영역
  * 각종 변수나 임시 데이터, 스레드나 메소드의 정보를 저장
  * 메소드 호출 시마다 각각의 스택 프레임이 생성
* Native method stack
  * 바이트 코드가 아닌 실제 실행할 수 있는 기계어로 작성된 프로그램을 실행시키는 영역
* Method Area = Class Area = Static area
  * 클래스 정보를 처음 메모리 공간에 올릴 때 초기화되는 대상을 저장하는 메모리 공간
* Runtime Constant Pool
  * 스태틱 영역에 존재하는 별도의 관리영역
  * 상수 자료형을 저장하여 참조하고 중복을 막는 역할
* Heap 영역
  * 객체를 저장하는 가상 메모리 공간
  * new 연산자로 생성되는 객체와 배열을 저장

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

