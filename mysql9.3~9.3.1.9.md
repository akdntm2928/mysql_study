# 9.3 고급 최적화
mysql 옵티마이저가 실행계획을 수립할때 통계정보와 옵티마이저 옵션을 통해 최적에 실행계획을 수립한다. 
옵티마이저 옵션은 크게 옵티마이저 옵션과 옵티마이저 스위치 두개로 나눌수있다.

옵티마이저 옵션은 초기부터 지원하게 된 기능이지만 많은사람들이 사용을 하지않는데 하지만 조인관련되 쿼리가 많은경우에는 사용을 해서 조인서능을 높힐수 있다.

옵티마이저 스위치는 mysql5.5부터 지원 하는 기능이면 mysql 서버의 고급 최적화 기능들을 활성화할지를 제어하는 용도이다.


## 9.3.1 옵티마이저 스위치 옵션
옵티마이저 스위치 옵션은 optimizer_swich 시스템 변수을 사용하는데 
optimizer_swich 옵션을 셋트로 사용할수 있다.

<table>
<thead>
    <tr>
        <th>
            옵티마이저 스위치 이름
        </th>
        <th>
            기본값
        </th>
        <th>
            설명
        </th>
    </tr>
</thead>
<tbody>
<tr>
    <td>batched_key_access</td>
    <td>off</td>
    <td>BKA 조인 알고리즘을 이용할지 여부 설정</td>
</tr>
<tr>
    <td>block_nested_loop</td>
    <td>on</td>
    <td>Block Nested Loop 조인 알고리즘을 사용할지 여부</td>
</tr>
<tr>
    <td>engine_condition_pushdown</td>
    <td>on</td>
    <td>Engine Condition Pushdown 기능을 사용할지 여부</td>
</tr>
<tr>
    <td>index_condition_pushdown</td>
    <td>on</td>
    <td>index condition pushdown 기능을 이용할지 여부 설정</td>
</tr>
<tr>
    <td>use_index_extenion</td>
    <td>on</td>
    <td>index Extenion 최적화를 이용할지 여부 설정</td>
</tr>
<tr>
    <td>index_merge</td>
    <td>on</td>
    <td>index_merge기능을 이용할지 여부 설정</td>
</tr>
<tr>
    <td>index_merge_intersection</td>
    <td>on</td>
    <td>index merge intersection기능을 이용할지 여부 설정</td>
</tr>
<tr>
    <td>index merge_union</td>
    <td>off</td>
    <td>index merge union을 이용할지 여부 설정</td>
</tr>
<tr>
    <td>mrr</td>
    <td>off</td>
    <td>MRR 최적화를 사용할지 여부</td>
</tr>
<tr>
    <td>mrr_cost_based</td>
    <td>on</td>
    <td>비용 기반의 MRR 최적화를 사용할지 여부</td>
</tr>
<tr>
    <td>semijoin</td>
    <td>on</td>
    <td>세미 조인 최적화를 사용할지 여부</td>
</tr>
<tr>
    <td>firstmatch</td>
    <td>on</td>
    <td>FristMatch 세미조인 최적화를 사용할지 여부 설정</td>
</tr>
<tr>
    <td>loosescan</td>
    <td>on</td>
    <td>LooseScan 세미 조인 최적화를 사용할지 여부</td>
</tr>
<tr>
<td>materialization</td>
<td>on</td>
<td>
materialization최적화를 사용할지 여부 설정<br>
(materialization 세미 조인 포함)
</td>
</tr>

<tr>
<td>subquery_materialization_cost_based</td>
<td>on</td>
<td>
비용기반의  materialization 최적화를 사용할지 여부 설정 <br>

</td>
</tr>
</tbody>
</table>

위와 같이 옵티마이저설정은 on,off,defalut로 설정할수 있는데
on은 옵션의 값으 활성화 하는것이고 off는 활성화를 종료하고 default는 초기값으로 해당 옵션을 사용하는 것이다

옵티마이저 스위치는 다음과 같이 세가지 옵셔으로 글로벌,세션,해당쿼리에만 옵티마이저 옵션을 적용할수 있다.

GLOBAL에서 사용시

    SET GLOBAL optimizar_switch='index_merge=on,index_merge_union=on'

SESSION에서 사용시
    
    SET SESSION optimizar_switch='index_merge=on,index_merge_union=on'

해당 쿼리에서 사용시

    SELECT /*+ SET_VAR(optimizar_switch='condition_fanout_fillter=off'/*
        .... 쿼리내용

이와 같이 표현하여 해당 옵티마이저 스위치 기능을 사용할수 있다.

## 9.3.1.1 mmr과 배치키 액세스 
mrr은 'multi-Range Read'를 줄여서 부르는 이름인데 메뉼얼에느 DS-MRR이라고도 한다  mysql서버에서 지금까지 지원하던 조인방식은 드라이빙 테이블에 한건에 레코드을 값을 읽어 드리븐 테이블과 일치하는 레코드 값을 찾아서 조인과정을 수행한다. 이를 네스티브 조인이라고 한다
이때 조인처리는 mysql엔진에서 처리하고 레코드값을 읽고/검색하는것은 스트리지 엔지에서 처리하는데 이와같이 네스티브 조인을 사용하면 드라이빙 테이블에 레코드 한건당 드리븐테이블을 계속 읽어 나가야 해서 스토리지엔지는 최적을 할 수 가 없다.

이같은 단점을 보안하기위해서 조인 버퍼에 레코드값을 버퍼링해서 조인버퍼가 가득차면 스토리지엔진으로부터 레코드값을 읽는 과정을 수행한다.
이와같은 방법을 사용하면 순서대로 정렬된 데이터페이지을 스토리지엔진은 그대로 읽어서 반환하면 되는 것이기 때문에 디스크 데이터 페이지 읽기을 최소화 할 수 있다.
물론 데이터 페이지가 innodb 버퍼풀이 있어도 동일하게 동작한다.

이러한 읽기방식을 MRR이라고하면 MRR을 이용해서 실행되는 조인방식을 BKA(bactched key Access) 조인이라고 한다 BKA조인 최적화는 기본적으로 비활성화 되있는데 쿼리특성에 따라 큰도움이 될수도 있지만 부가적인 정렬작업이 필요해지면 오히려 성능에 더 안좋은 영향을 줄수 있다.

------------------------------

## 9.3.1.2 블록 네스티브 루프 조인
기본적으로 조인을 할떄 네스티브 조인을 사용하는데 이방식은 조인에 연결조건에 칼럼들이 모든 index에 있는경우 사용되는방식이다 

블록 네스티브 루프 조인 join buffer 사용한 조인인데 드라이빙 테이블과 드리븐 테이블에서 조인 칼럼에 index을 사용하지 못할 경우에는 드라이빙 테이블에 모든 검색 결과을 조인버퍼에 담아서 드라이븐 테이블에 매칭이되는 칼럼끼로 조인후 그결과 값을 반환 한다 이때 블록 네스티브 루프 조인을 사용하면  Extra에서 Using join Buffer가 표시가 된다.주의상황으로는 이 방식을 사용하면 정렬순서가 흐트러질수 있다

## 인덱스 컨디션 푸시다운

인덱스 컨디션 푸시다운은 index로 설정한 칼럼이 있는 경우 그 해당 조건은 범위검색 조건을 하실 만약 범위검색 조건이 인덱스을 타지 않는경우 스토리지엔진에 그해당 index 컬럼에 대한 내용을 전달할수있도록 핸들러 Api가 개선이 되었다. 이떄 extra에 표시되는 값은 index_condition_pushdown 표시가 된다.(쿼리기능을 몇십배 항상시킬수 있는 중요한 기능이다 기본값은 on이다)

## 인덱스 확장(use_index_extensions)
use_index_extensions 옵티마이저 옵션은 innodb 스토리지 엔지을 사용하는 테이블에서 세컨더리 인덱스를 프라이머리 키와 함께 사용할수 있게 하는 index이다 만약에 이와같이 프라이머리키을 생성하고 세컨더리 인덱스를 생성 할떄 primary ket index을 같이 활용할수가 있다
    
    PRIMARY KEY (dept_no,emp_no)

    key ix_fromdate(from_date)

이와 같이 생성이 되었을때 클러스터 index로 dept_no,emp_no으로 생성되어서 innodb 스토리지엔진으 두개 칼럼으로 정렬이되는데 이때 세컨더리 키가 같이 확장되어서 생성 한것처럼 동작을 한다

    explain select count(*) from dept_emp where from_date='1987-07-25' and dept_no='d001'

    explain select count(*) from dept_emp where from_date='1987-07-25' order by dept_no;

이와같이 쿼리을 실행 했을때 using temporary와 using filesort가 안뜨는것을 확인 할수있는데 이와같은 방식으로 사용하게 되면 레코드을 있는 그대로 순서대로 활용을해서 최적화된 쿼리을 수행할 수 있다.

## 인덱스 머지(index_marge)

인덱스를 이용해서 쿼리를 실행하는경우 일반적으로 옵티마이저는 테이블별로 하나에 index에 칼럼에 조건만 사용되도록 실행계획이 수립된다.
하지만 서로 다른 index을 조건으로 사용하고 있고 하나에 index롤 검색시 레코드 수가 많으면 옵티마이저는 index merge 계획을 수립한다.</br>
index merge 방법은 다음과 같이 3가지방법이 있다

index_merge_intersection : 교집합 
index_merge_sort_union : 절렬후 합집합
index_merge_union: 합집합

------------------------------------- 

## 인덱스 머지 - 교집합(index_merge_intersection)
만약에 primary key로 emp_no 세컨더리키로 first_name이 있을떄 경우을 살펴 보자
    
        EXPLIAN SELECT * FROM employees WHERE frist_name='Georgi' NAD emp_no BETWEEN 10000 AND 20000
    
이와 같은 쿼리을 샐행할떄 frist_name='Georgi' 조건으로만 검색시 253개가 나오고 emp_no BETWEEN 10000 AND 20000 검색시 10000개 나온다 두개에 검색 조건을 합치 결과는 14개만 나와서 옵티마이저는 어떤게 효율적인지 판단해서 14개만 읽는 방법을 선택하여 inster_merge_intersection 방법을 선택을 한다 이때 extra에서는 Using intersect(ix_firtstname,PRIMARY); Using where이 노출이 된다. 

하지만  전에 말했던거와같이 primary키와 세컨더리키는 use_index_extensions 옵션으로 인하여 (emp_no, first_name)과 같은 형태로 인덱스가 생성이 된것과 같다
이와같이 수행하는게 더 좋다고 판단될시에는

    SELECT  /*+ SET_VAR(optimizar_switch='index_marge_intersection=off') */
        .. 쿼리 내용

이와 같이 옵션을 비활성화 하여 사용 할 수 있다.

-----------------------------------

## 인덱스 머지 - 합집합(index_merge_union)
 index_merge_intersection과 동일한 방식으로 두개 컬럼 조건을 OR로 사용 하는방법이다. 각 테이블에 하나식 레코드을 가져와서 중복체크 작업을한다
 두집합은 프라이머리 키 기준으로 정렬을 한다
 이떄 사용되는 중복제거 알고리즘은 우선 큐라고 한다.
 extra에서는 using union(ix_firstname,ix_hiredate)로 표시가 된다

-------------------------------------

## 인덱스 머지 - 합집합-정력(index_merge_sort_union)

인덱스 합집합과 동일하게 동작 하지만 각인덱스 정렬기준이 각 칼럼에 index에 조건을 수행후 정렬후에 우선큐 작업을 진행한다 
extra에서는 using sort_union(ix_firstname,ix_hiredate)로 표시가 된다
<br>

--------------------------------

## 세미 조인(semijoin)
다른 테이블과 실제로 조인을 하지않고 다른 테이블에서 조건 일치하느 레코드가 있는지 체크하는 쿼리를 세미 조인(semijoin) 이라고 한다.


다음 세미쿼리에 최적화 기능이 없을경우 employees에 테이블을 풀스캔을 한다

        SELECT * 
            FROM employees e 
            WHERE e.emp_no IN
                (SELECT de.emp_no FROM demp_emp de de.FROM='1995-01-01')

세미 조인 형태는 세미 조인 형태와 안티 세미 조인 형태 있다.

세미 조인 형태는 "=(subquery)" 형태와 in (subquery) 형태의 방식은

-  세미 조인 최적화
-  IN-to-EXISTS 최적화
-  MATERIALIZATION 최적화

안티 세미 조인형태는 "<> (subquery)" 형태와 "NOT IN (subquery)" 형태로 방식은
- In-to-EXISTS 최적화
-  MATERIALIZATION 최적화

















