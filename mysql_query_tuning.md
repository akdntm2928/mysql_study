#  1. 기본 키를 변형하는 나쁜 sql문

    explain select * from 사원 where substring(사원번호,1,4)  =1100 and length(사원번호) =5;
    explain select  * from 사원 where 사원번호 between 11000 and 11009;

#  2 사용하지 않는 함수를 포함하는 나쁜 sql

    explain select substring(성별,1,1) as 성별 , count(1) 건수
        from 사원 group by substring(성별,1,1);                                                                                     
    explain select 성별, count(*) as 건수 from 사원 group by 성별 ;
    group by 칼럼값을 함수로 변형시키면 임시테이블이 생겨난다.

#  3.  형변환으로 인덱스를 활용하지 못하는 나쁜 sql문
    
    explain select count(1) from 급여 where 사용여부 =1; 
    
    explain select count(1) from 급여 where 사용여부 ='1';
 
#   4. 열을 결합하여 사용하는 나쁜 sql 문

    select * from 사원 where concat(성별,' ',성) ='W Radwan';
    select count(*) from 사원;
    show index from 사원;
    select * from 사원 where 성별 ='M' and 성 ='Radwan';


#  5. 습관적으로 중복을 제거하는 나쁜 sql문

    explain select distinct 사원.사원번호, 사원.이름 , 사원.성, 부서관리자.부서번호
        from 사원 join 부서관리자 on(사원.사원번호 =부서관리자.사원번호);

    select count(*) from 부서관리자;
    select count(*) from 사원;

    explain select  사원.사원번호, 사원.이름 , 사원.성, 부서관리자.부서번호
        from 사원 join 부서관리자 on(사원.사원번호 =부서관리자.사원번호);
    
#  6. 다수 쿼리를 union 연산자로만 합치는 나쁜 sql문

    desc 사원;
    select 'M' as 성별, 사원번호 from 사원 where 성별= 'M' and 성='Baba'
    union 
    select 'F', 사원번호 from 사원 where 성별 ='F' and 성='Baba';


    select 'M' as 성별, 사원번호 from 사원 where 성별= 'M' and 성='Baba'
    union ALl
    select 'F', 사원번호 from 사원 where 성별 ='F' and 성='Baba';
    union 임시테이블을 생성하고 정렬과정이 필요해서 임시테이블을 생성한다
    union all 단순히 합치는 작업만순행해서 임시테이블 필요가 없다
    select /*+set_var(optimizer_switch ='skip_scan=on')*/
    성별, 사원번호 from 사원 where 성 ='Baba';

#  7. 인덱스 고려없이 열을 사용하는 나쁜 쿼리
    desc 사원;
    show index from 사원;
    explain select 성,성별,count(1) as 카운트
    from 사원 group by 성,성별;  
    성별_성 index을 사용하고자 임시테이블을 만들어서 성별_성으로 그룹핑후 정렬작업을 시행한다

    explain select 성,성별,count(1) as 카운트
    from 사원 group by 성별, 성;

#  8. 엉뚱한 인덱스를 사용하는 나쁜 sql문

    explain select 사원번호 from 사원 where 입사일자 like "1989%" and 사원번호>100000;
    show index from 사원;
    select count(*) from 사원  where 입사일자 like "1989%";

    select count(*) from 사원  where 사원번호>100000;
    ##210024 
    explain select count(*) from 사원 use index(I_입사일자) where 입사일자 like "1989%" and 사원번호 > 100000;
    Using index for skip scan 인덱스 루스 스캔 쓸때없느 오버헤드가 발생할수 있다.


    explain select count(*) from 사원  where 입사일자 like "1989%";
    like "텍스트%"을 사용시 index full scan을 한다

    explain select count(*) from 사원 where 입사일자 between '1989-01-01' and '1990-01-01' and 사원번호 > 100000;


#  9. 동등 조건으로 인덱스를 사용하는 나쁜 sql문
    select count(*) from 사원출입기록;


    explain select * from 사원출입기록  where 출입문 ='B';
    explain select * from 사원출입기록 ignore index(I_출입문) where 출입문 ='B';


#  10`. 범위 조건으로 인덱스를 사용하는 나쁜 sql문
    select str_to_date('1994-01-01','%Y-%m-%d');
    explain select 이름,성 from 사원 where 입사일자 between str_to_date('1994-01-01','%Y-%m-%d') and str_to_date('2000-12-31','%Y-%m-%d') ;
    explain select 이름,성 from 사원 use index(I_입사일자) where 입사일자 between str_to_date('1994-01-01','%Y-%m-%d') and str_to_date('2000-12-31','%Y-%m-%d') ;
    explain select 이름,성 from 사원 where YEAR(입사일자) between '1994' and '2000';


#  11 작은 테이블이 먼저 조인에 참여하는 나쁜 sql문

    select straight_join 매핑.부서번호,부서.부서번호
        from 부서사원_매핑 매핑,부서 where 매핑.부서번호 = 부서.부서번호 and 매핑.시작일자 >= '2002-03-01';

    select 매핑.부서번호,부서.부서번호
        from 부서사원_매핑 매핑,부서 where 매핑.부서번호 = 부서.부서번호 and 매핑.시작일자 >= '2002-03-01';

    alter table 부서사원_매핑 add index ix_시작일자(시작일자);
    alter table 부서사원_매핑 drop index ix_시작일자;

    explain select  매핑.부서번호 from 부서사원_매핑 매핑 where 매핑.시작일자 >= '2002-03-01';


#  12. 메인 테이블에 계속 의존하는 나쁜 SQL 문

    explain select 사원.사원번호, 사원.이름, 사원.성 
        from 사원 where 사원번호 >450000
        and (select max(연봉)
                from 급여
            where 사원번호 = 사원.사원번호)>100000;
            
    explain select 사원.사원번호, 사원.이름, 사원.성 
        from 사원 where 사원번호 >450000
        and (select max(연봉)
                from 급여
                where 연봉 >100000
                group by 사원번호);
                
    select 사원.사원번호, 사원.이름, 사원.성 
        from 사원 left join 급여 on 급여.사원번호 = 사원.사원번호
        where 사원.사원번호 >450000  and 
        급여.연봉>100000 group by 사원.사원번호;
        
    explain select 사원.사원번호, 사원.이름, 사원.성 
        from 사원 left join 급여 on 급여.사원번호 = 사원.사원번호
        where 사원.사원번호 >450000  group by 사원.사원번호 having max(급여.연봉) >100000;


#  13.  불필요한 조인을 수행하는 나쁜 조인

    select count(distinct 사원.사원번호) as 데이터건수
        from 사원,
        (SELECT 사원번호 FROM 사원출입기록 AS 기록 WHERE 출입문 ='A') 기록  WHERE 사원.사원번호 = 기록.사원번호;
        
        DESC 사원;
        SELECT COUNT(*) FROM 사원출입기록;
        


    SELECT COUNT(DISTINCT 사원.사원번호) FROM 사원,사원출입기록 WHERE 사원.사원번호=사원출입기록.사원번호 AND 사원출입기록.출입문 ='A';
    EXPLAIN SELECT COUNT(*) FROM 사원 WHERE EXISTS(SELECT 1 FROM 사원출입기록 WHERE 출입문 ='A' AND 사원.사원번호=사원출입기록.사원번호 );

    SHOW WARNINGS;
#  14. 처음부터 모든 데이터를 가져오는 나쁜 sql 문

    select 사원.사원번호,급여.평균연봉,급여.최고연봉,급여.최저연봉
    from 사원,
        (select 
            사원번호,
            round(avg(연봉),0) 평균연봉 ,
            round(max(연봉),0) 최고연봉, 
            round(min(연봉),0) 최저연봉 
            from 급여 group by 사원번호) 급여
    where 사원.사원번호 = 급여.사원번호
        and 사원.사원번호 between 10001 and 10100;
        
    explain select 사원.사원번호,
            round(avg(연봉),0) 평균연봉 ,
            round(max(연봉),0) 최고연봉, 
            round(min(연봉),0) 최저연봉 
        from 사원,급여
    where 사원.사원번호 = 급여.사원번호 
        and 사원.사원번호 between 10001 and 10100
        group by 사원.사원번호;
        

#  15.  비효율적인 페이징을 수행하는 나쁜 sql문

    select 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자
        from 사원,급여
    where 사원.사원번호 = 급여.사원번호
        and 사원.사원번호 between 10001 and 50000
    group by 사원.사원번호
    order by sum(급여.연봉) desc
    limit 150,10;

    select 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자 from 
        (select 사원번호 
            from 급여 where 사원번호 between 10001 and 50000
            group by 사원번호
            order by sum(연봉)
            limit 150,10
        ) as 급여,사원 
        where 급여.사원번호 = 사원.사원번호;
    쿼리 실행시 조인에 방식 네스티드 루프조인은 중첩반복문이기 때문에 드라이빙 테이블에 갯수가 더적을수록 효율적이다.
    
#  16. 필요 이상으로 많은 정보를 가져오는 나쁜 sql문

    explain select count(사원번호) as 카운트 
        from (
            select 사원.사원번호, 부서관리자.부서번호
                from (select * from 사원 where 성별 ='M' and 사원번호 >300000)  사원 left join 부서관리자 on 사원.사원번호 = 부서관리자.사원번호
        ) 서브쿼리;
    select 사원번호 from 사원 where 성별 ='M' and 사원번호 >300000;

    explain select count(1) from 사원 where 성별 ='M' and 사원번호 >300000;

#  17. 대량에 데이터을 가져와서 group by 하는 데이터

    select distinct 매핑.부서번호
        from 부서관리자 관리자, 부서사원_매핑 매핑 
        where 관리자.부서번호 = 매핑.부서번호
        order by 매핑.부서번호;

    explain select distinct 부서관리자.부서번호
        from 부서관리자 where exists(
            select 1 from 부서사원_매핑 매핑 where 매핑.부서번호 = 부서관리자.부서번호
        );
        
    exists 통해서 부서관리자는 24건에 데이터만 있어 부서관리_매핑에 데이터에서 있으면 true반환하고 
    안읽는 exists활용하면 부서매핑_매핑에 대용량 데이터에 접근을 최소화 할수 있다

#   18. 인덱스 조정으로 착한 쿼리 만들기
    show index from 사원;
    select count(*) from 사원 where 이름='Georgi';
    select count(*) from 사원 where 성='Wielonsky';

    alter table 사원 add index I_이름_성(이름,성);
    select * from 사원 where 이름 ='Georgi' and 성='Wielonsky';
    alter table 사원 drop index I_이름_성;
    이름 성에 다중인덱스을 생성해서 속도을 최적화한다.

 # 19. 인덱스를 하나만 사용하는 나쁜 sql문 
    alter table 사원 add index I_이름(이름);
    select * from 사원 where 이름='Matt' OR 입사일자='1987-03-31';
    alter table 사원 drop index I_이름;


#  20. 큰 규모의 데이터 변경으로 인덱스에 영향을 주는 나쁜 쿼리

    set autocommit =1;
    SELECT @@autocommit;  자동으로 트잭잭션을 commit 해줌
    select @@SQL_SAFE_UPDATES ;   safe모드 활성화모드
    set SQL_SAFE_UPDATES  = 1;

    show index from 사원출입기록;
    alter table 사원출입기록 drop index I_출입문;
    rollback;
    select * from 사원출입기록 where 출입문 ='X';
    update 사원출입기록 set 출입문 ='X' WHERE  출입문 ='B';


#  21. 비효율적인 인덱스를 사용하는 나쁜 SQL문

    show index from 사원;
    select 사원번호 , 이름 ,성 from 사원 where 성별="M" and 성="Baba";
    alter table 사원 drop index I_성별_성;
    alter table 사원 add index I_성_성별(성,성별);



#  22. 파티션을 이용한 최적화

    explain select count(1) from 급여 where 시작일자 between str_to_date('2000-01-01','%Y-%m-%d') and  str_to_date('2000-12-31','%Y-%m-%d');
    ##2831628

    파티션 이용
    alter table 급여
    partition by range columns(시작일자)
    (
        partition p85 values less than ('1985-12-31'),
        partition p86 values less than ('1986-12-31'),
        partition p87 values less than ('1987-12-31'),
        partition p88 values less than ('1988-12-31'),    
        partition p89 values less than ('1989-12-31'),    
        partition p90 values less than ('1990-12-31'),    
        partition p91 values less than ('1991-12-31'),
        partition p92 values less than ('1992-12-31'),
        partition p93 values less than ('1993-12-31'),
        partition p94 values less than ('1994-12-31'),
        partition p95 values less than ('1995-12-31'),
        partition p96 values less than ('1996-12-31'),
        partition p97 values less than ('1997-12-31'),
        partition p98 values less than ('1998-12-31'),
        partition p99 values less than ('1999-12-31'),
        partition p00 values less than ('2000-12-31'),    
        partition p01 values less than ('2001-12-31'),            
        partition p02 values less than ('2002-12-31'),                          
        partition p03 values less than (maxvalue)
    );

    explain select count(1) from 급여 where 시작일자 between str_to_date('2000-01-01','%Y-%m-%d') and  str_to_date('2000-12-31','%Y-%m-%d');
    ##503433



