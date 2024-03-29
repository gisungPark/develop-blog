# 디미터 법칙(Law of Demeter)

디미터 라는 이름의 프로젝트를 진행하던 도중 다른 객체들과의 협력을 통해 프로그램을 완성해나가는 객체지향 프로그래밍에서 객체들의 협력 경로를 제한하면 결합도를 효과적으로 낮출 수 있다는 사실을 발견했고 디미터 법칙을 만들었다.

현재 디미터 법칙은 객체 간 관계를 설정할 때 객체 간의 결합도를 효과적으로 낮출 수 있는 유용한 지침 중 하나로 꼽히며 객체 지향 생활 체조 원칙 중 **한 줄에 점을 하나만 찍는다.**로 요약되기도 한다.



### Don’t Talk to Strangers

디미터 법칙의 핵심은 객체 구조의 경로를 따라 멀리 떨어져 있는 낯선 객체에 메시지를 보내는 설계를 피하라는 것이다.

바꿔 말해서 객체는 내부적으로 보유하고 있거나 메시지를 통해 확보한 정보만 가지고 의사 결정을 내려야 하고, 다른 객체를 탐색해 뭔가를 일어나게 해서는 안된다.

이러한 핵심적인 내용 때문에 디미터 법칙은 '낯선 이에게 말하지 마라'라고도 불리며,\
한 객체가 알아야 하는 다른 객체를 최소한으로 유지하라는 의미로 principle of least knowledge(최소 지식 원칙)라고도 불린다.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

Board 객체의 addComment 메서드를 살펴보자.\
boar객체의 인스턴수 변수 posts에서 getter를 거듭해 멀리 떨어져 있는 낯선 객체 Comment를 추가하는 코드이다.

이처럼 getter가 줄줄이 이어지는 코드 형태가 디미터 법칙을 위반한 전형적인 코드 형태이다.

왜 낯선 객체에 메시지를 보내는 설계를 피해야 할까?\
즉, 디미터 법칙을 위반했을 때의 문제점은 무엇일까?

앞서 살펴봤던  Post 객체에서 만약 인스턴스 변수가 Lis\<Comment> comments에서 Comments라는 일급컬렉션 객체로 수정된다면 어떻게 될까?

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

getter를 통해 `Post` 객체의 `List<Comment> comments`를 사용하던 `Board` 객체의 `addComment`메서드가 깨지게 된다. 이처럼 `Board` 객체의  `addComment`메서드 내에서 `Post`객체도 알고 `Comment` 객체도 알고 있다면 `Board` 객체는 `Post`객체의 변화에도 영향을 받고 `Comment`객체의 변화에도 영향을 받는다.

이러한 설계가 프로젝트 내에 즐비하다면 하나의 변화에 수많은 클래스들이 무너질 가능성이 있다.\
즉, 객체 간 결합도가 높아지고 객체 구조의 변화에 쉽게 무너진다. 변화에 유연히 대처하지 못하는 것이다.



## 규칙화

디미터 법칙은 "노출 범위를 제한하기 위해 객체의 모든 메서드는 다음에 해당하는 메서드만을 호출해야 한다"라고 말한다.

* 객체 자신의 메서드들
* 메서드의 파라미터롤 넘어온 객체들의 메서드들
* 메서드 내부에서 생성, 초기화된 객체의 메서드들
* 인스턴스 변수를 가지고 있는 객체가 소유한 메서드들

<figure><img src="../../../.gitbook/assets/image (2) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



위 규칙을 지켜서 최대한 노출 범위를 제한하면 좀 더 에러가 적고, 변화에 유연히 대처할 수 있는 클래스를 만들 수 있다.

## 주의 사항

*   자료구조라면 디미터 법칙을 거론할 필요가 없다.

    객체라면 내부 구조를 숨겨야 하므로 디미터 법칙을 지켜야 한다. 하지만 자료구조라면 당연히 내부 구조를 노출해야 하므로 디미터 법칙이 적용되지 않는다. 객체와 자료구조의 차이가 궁금하다면 아래 링크를 참고하자.

{% embed url="https://namget.tistory.com/entry/%ED%81%B4%EB%A6%B0%EC%BD%94%EB%93%9C-6%EC%9E%A5-%EA%B0%9D%EC%B2%B4%EC%99%80-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0" %}







{% embed url="https://tecoble.techcourse.co.kr/post/2020-06-02-law-of-demeter/" %}











