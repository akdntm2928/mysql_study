# group by 처리    
group by이도 order by와 동일하게 스트리밍 처리을 할수 없게하는 처리중
하나이다
이떄 group having절이 있는데 having절을 통한 인덱스는 사용할 수 없다
group 3가지 방식이 있는데 이방식에 대한서 알아가보자

<br>

## 인덱스 스캔 방식: 

- 해당 테이블에 칼럼에 index을 이용해서 사용하는 방식으로 순서대로 정렬된 데이터에 대해서 그룹핑 작업을 수행하고 join 하는 과정으로 처리하는 방식이다 이떄 orderby에 마찬가지로 join과정에서도 드라이빙 테이블에 index칼럼으로 group시에도 사용할수 있다. extra에 별도 using temporary, using index for group by가 표시 되지 않는다.

<br>

## 인덱스 루스 방식
- 루스(Loose) 인덱스 스캔 방식은 인덱스 레코드를 건너 뛰면서 필요한 부분만 읽어서 사용하는 방식으로 사용시 using index for group by가 표시가 난다 
예시로 

        explain select emp_no 
            from salaries
            where from_date = '1985-03-01' -- 
            group by emp_no -- 
    
        ex) emp_no에 대한 그룹에서 1985-03-01에 대한 값으로 된 row값을 가져와서 테이블 생성 이때 범위 검색이 발생한다
            
<br>

## 임시테이블 방식

- group by에서 사용한 값이 드라이빙이든 드리븐 테이블이든 index을 사용하지 못할떄 정렬을 해야되므로 멀티머지가 발생해 임시테이블이 생성된다
mysql 8.0 이하버전은 임시테이블에서 그룹화가 된값에 대해서 정렬을 하지않는데    ex (using filesort X) 8.0 이하버전은 filesort가 발생되므로 굳이 정렬이 필요없다면 order by null을 통해서 속도을 증가 시킬수 있다.

-----------------------------------

