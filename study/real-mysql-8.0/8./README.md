# 8. 인덱스

## index가 중요한 이유

```sql
SELECT *
FROM CUSTOMER
WHERE FIRST_NAME ='MINSOO';
```

*   first\_name 에 index가 걸려있지 않다면?

    full scan으로 찾아야 한다. bigO(N)
*   index가 걸려 있다면,

    full scan 보다 더 빨리 찾을 수 있다. bigO(log N)  (B-tree 기반일 경우)

따라서, 조건을 만족하는 튜플들을 빠르게 조회하게 하는 것이 인덱스의 목적이다.

빠르게 정렬하거나 그룹빙 하기 위해!



### 인덱스를 쓰는 이유

```sql
SELECT * FROM CUSTOMER WHERE A = ?

DELETE FROM LOGS WHERE BB < ?;

UPDATE EMPLOYEE SET SALARY = SALARY*1.5 WHERE DEP_ID = 10001;

SELECT * FROM EMPLOYEE E JOIN DEPARTMENT D ON E.DEPT_ID = D.DEPT_ID;
```

특정 조건을 만족하는 데이터를 빠르게 찾기 위해 인덱스를 사용한다.





멀티 컬럼 인덱스/컴퍼짓 인덱스; 두개 이상의 컬럼이 합쳐진 인덱스를

## index 동작 방식

```sql
CREATE INDEX(A,B)
```

위 와 같은 멀티 컬럼 인덱스의 경우, A기준으로 우선 정렬 후, B로 정렬하게 된다. 때문에 멀티컬럼 인덱스의 순서가 중요하다.&#x20;



### b-tree 기반 인덱스의 동작방식







### hash index

* hash table을 사용하여 index구현
* 시간 복잡도 O(1)의 성능
* rehashing에 대한 부담
* equality 비교만 가능, range비교 불가능
* multicolumn index의 경우 전체 attributes에 대한 조회만 가능



## index 사용시 참고 사항





full scan이 더 좋은 경우

* table에 데이터가 조금 있을 때 (몇 십, 몇 백건 정도)
* 조회하려는 데이터가 테이블의 상당 부분을 차지할 때





