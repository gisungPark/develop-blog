# PostgreSQL

## 메모리 아키텍처

psql의 메모리 영역은 용도와 접근성에 따라 크게 로컬 메모리와 공유 메모리로 분류할 수 있습니다.&#x20;

### 로컬 메모리 영역

로컬 메모리는 백엔드 프로세스가 각각 할당받아 사용하는 공간으로 주로 백엔드 프로세스가 자신이 담당하는 커넥션을 통해 요청된 쿼리를 수행하기 위해 사용됩니다. 로컬 메모리는 프로세스 별로 할당되므로 다른 프로세스에서 참조/접근이 불가합니다.

* work\_mem; 세션 단위 조정
* work\_mem; 트랜잭션 단위 조정

### 공유 메모리 영역

공유 메모리는 psql서버의 모든 프로세스와 공유되는 공간으로 데이터 블록, 트랜잭션 로그 등 여러 프로세스에서 공유가 필요한 정보를 저장하는 공간으로 사용됩니다. 공유 메모리는 poql 서버가 기동될 때 시스템 메모리에서 할당받고 종료될 때 다시 시스템 메모리로 반환됩니다.



## 백엔드 프로세스

백엔드 프로세스는 자신이 담당하는 커넥션으로부터 들어온 쿼리를 처리하는 역할을 수행합니다. 본 문서에는 조금 더 깊게 들어가서 백엔드 프로세스가 쿼리(query processing)를 처리하는 과정을 소개하겠습니다.

### 1. parser

대부분의 데이터베이스와 마찬가지로 psql은 sql 문법으로 작성된 문자열 형태의 쿼리를 입력으로 받습니다. parser는 입력받은 쿼리의 문법을 검증하는 작업을 수행합니다. 또한 문법 구조를 바탕으로 parser tree를 생성하는 역할을 수행합니다.

문법 검증을 수행한다는 것은 select, from, where 등의 구문이 적절한 위치에 쓰여졌는지, 쉼표가 적절히 쓰여졌는지 만을 판단한다는 의미 입니다. 따라서, parser에서는 실제 테이블 컬럼의 존재 여부와 같은  semantic 검증은 수행하지 않습니다.(해당 작업은 analyzer에서 수행)

parser에서 수행하는 단계를 정확히 이해하기 위하여 Parser Tree라는 개념에 대해서 알 필요가 있습니다. Parser Tree는 Parse Node라고 불리는 구조체들로 이루어진 Tree입니다. Parser Node는 해당 구문의 성격(INSERT, SELECT 등)에 따라 상이한 내용을 담고 있습니다.\


<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://oss.tibero.com/6c2ff524-645e-45f2-a05b-0cd409d12a78" %}

### 2. Analyzer

analyzer는 parser에서 생성한 parse tree를 입력받아 query Tree를 생성합니다. 상술하였듯 parse Tree는 문법만 고려되어 있고 실제 테이블, 컬럼과 같은 의미론적인(semantic) 내용은 포함하고 있지 않습니다. Analyzer에서는 parse Tree에서 언급하는 테이블, 컬럼, 연산자의 유효성 등에 대한 Semantic 검증을 System Catalog 조회를 통해 수행하고, 해당 내용을 추가하여 Query Tree를 생성합니다. 즉 Query Tree는 Parse Tree와 다리 테이블 컬럼을 참조하기 위한 정보를 포함하고 있다는 점에서 차이를 보입니다.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### 3. Rewriter

Analyzer에서 생성된 Query Tree는 Planenr에 전달되기 전, Rewrite 과정을 거치게 됩니다. Rewriter는 사용자가 Rulu System에 저장한 규칙을 기반으로 Query Tree를 수정합니다.



### 4. Planner /Optimizer&#x20;

Analyzer와 Rewriter를 거친 Query Tree는 Planner에 전달됩니다. Planner는 쿼리가 요구하는 데이터의 명세인 Query Tree를 기반으로 해당 데이터를 찾고, 가공하기 위한 실행 계획, 즉 Plan Tree를 생성합니다.

하나의 쿼리에서는 수 많은 조합의 실행 계획이 만들어질 수 있습니다. 따라서 대부분의 데이터베이스에서는 각 계획의 비용을 계산하여 가장 효율적인 실행 계획을 선택합니다. PostgreSQL은 이를 위해 Cost-based-optimization방식을 사용합니다.



{% embed url="https://www.postgresql.org/docs/current/planner-optimizer.html" %}





![](<../.gitbook/assets/image (3).png>)

클라이언트부터 질의 요청이 들어오면 구문 분석 과정(1)을 통해 Parser Tree를 생성하고 의미 분석 과정(2)을 통해 새로운 트랜잭션을 시작하고 Query Tree를 생성한다.

이후 서버에 정의된 Rule에 따라 Query Tree가 재 생성(3)되고 실행 가능한 여러 수행 계획 중 최적화된 Plan Tree를 생성(4)한다. 서버는 이를 수행(5)하여 요청된 질의에 대한 결과를 클라이언트로 전달하게 된다.(6)



{% embed url="https://d2.naver.com/helloworld/227936" %}





