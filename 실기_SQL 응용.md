# SQL 응용



## 프로시저

- 절차형 SQL을 활용하여 특정 기능을 수행할 수 있는 트랜잭션 언어

- EXCUTE 프로시저명 (파라미터,파라미터);

```
**선언부**
CREATE PROCEDURE SALES_CLOSING
(V_CLOSING_DATE IN CHAR(8))
IS
V_SALES_TOT_AMT NUMBER := 0;

**시작/종료부**
BEGIN

**제어부**
IF V_CLOSING_DATE < "20000101" THEN
SET
   V_CLOSING_DATE = "20200101"
END IF

**SQL**
SELECT SUM(SLAES_AMT)
 INTO V_SALES_TOT_AMT //처리 결과를 해당 변수에 저장
 FROM SALES_LIST_T
WHERE SALES_DATE = V_CLOSING_DATE;

**예외부**
EXCEPTION
WHEN NO_DATE_FOUND THEN
SET V_SALES_TOT_AMT = 0;

INSERT INTO
SALES_CLOSED_T(SALES_DATE,SALES_TOT_AMT)
VALUES(V_CLOSING_DATE,V_SALES_TOT_AMT);

**실행부**
COMMIT;
END;

```

- 제어부 세부사항

  - IF문

  - ```
    IF 조건 THEN
    문장;
    ELSEIF 조건 THEN
    문장;
    
    ELSE
    문장
    END IF;
    ```

  - CASE문

  - ```
    CASE 변수
     WHEN 값1 THEN
      SET 명령어;
     WHEN 값2 THEN
      SET 명령어;
     ELSE
      SET 명령어;
    END CASE;
    ```

  - 검색된 CASE 문

  - ```
    CASE 변수
     WHEN 조건1 THEN
      SET 명령어;
     WHEN 조건2 THEN
      SET 명령어;
     ELSE
      SET 명령어;
    END CASE;
    ```

  - LOOP문

  - ```
    LOOP
     문장;
     EXIT WHEN 탈출조건;
    END LOOP;
    ```

  - WHILE문

  - ```
    WHILE 반복 조건 LOOP
     문장;
     EXIT WHEN 탈출조건;
    END LOOP;
    ```

  - FOR문

  - ```
    FOR 인덱스 IN 시작 값 .. 종료값
     LOOP
     문장;
    END LOOP;
    ```





## 사용자 정의함수

- 절차형 SQL을 활용하여 일련의 SQL 처리 수행
- 수행 결과를 단일 값으로 반환할 수 있는 절차형 SQL
-  프로시저와 거의 동일하지만 반환부분이 다르다.



- ```
  CREATE FUNCTION GET_AGE(V_BIRTH_DATE IN CHAR(8))
  IS
  
  BEGIN
   V_CURRENT_YEAR CHAR(4);
   V_BIRTH_YEAR CHAR(4);
   V_AGE NUMBER;
   
  IF V_BIRTH_DATE > "300000" THEN
  SET
   V_CLOSING_DATE = "20200101"
  END IF;
  
  SELECT TO_CHAR(SYSDATE,'YYYY'),SUBSTR(V_BIRTH_DATE,1,4)
   INTO V_CURRENT_YEAR,V_BIRTH_YEAR
   FROM DUAL;
   
  SET AGE = TO_NUMBER(V_CURRNET_YEAR) - TO_NUMBER(V_BIRTH_YEAR) + 1;
  
  **반환부**
  RETURN AGE;
  END;
  ```



- ```
  SELECT GET_AGE('19900101') FROM DUAL
  ```



## 트리거

- 특정 테이블에 삽입,수정,삭제 등의 데이터 변경 이벤트가 발생하면 DBMS에서 자동적으로 실행되도록 구현된 프로그램
- 특정 테이블에 대한 데이터변경을 시작점으로 설정하고, 그와 관련된 작업을 자동적으로 수행하기 위해 트리거 사용
- 무결성 유지,로그 메시지 출력 등 별도처리를 위해 사용



- 트리거 종류
  - 행 트리거
    - 데이터 변화가 생길 때마다 실행
  - 문장 트리거
    - 트리거에 의해 단 한번 실행
- 반환값이 존재하지 않음 , DML 을 주목적으로 사용

- TCL ( ROLLBACK,COMMIT 사용 불가)

- ```
  CREATE TRIGGER PUT_EMPLOYEE_HIST
  
  AFTER UPDATE OR DELETE
  ON EMPLOYEE
  FOR EACH ROW
  
  BEGIN
  
  IF UPDATING
  THEN
  **SQL문**
  ~~~
  ELSEIF DELETING
  THEN
   INSERT INTO EMPLOYEE~
   ~
  END IF;
  END;
  ```



## 집계성 함수

- 집계함수, 그룹함수, 윈도 함수

- 집계함수

  - ```
    SELECT COUNT(*)
    FROM STUDENNT
    WHERE 국어 >=80
    
    SELECT SUM(국어),AVG(영어)
    FROM STUDENT
    
    SELECT MAX(국어),MIN(국어)
    FROM STUDENT
    
    SELECT STDDEV(국어)*표준편차*,VARIANCE(국어)
    FROM STUDENT
    ```

- 그룹 함수

  - 중간 집계 값을 산출

  - ```
    SELECT DEPT,JOB,SUM(SALARY)
    FROM DEPT_SALARY
    GROUP BY ROLLUP(DEPT,JOB)
    
    SELECT DEPT,JOB,SUM(SALARY)
    FROM EMP
    GROUP BY CUBE(DEPT,JOB)
    
    SELECT DEPT,NAME,SUM(SALARY)
    FROM DEPT_SALARY
    GROUP BY GROUPING SETS(DEPT,JOB)
    ```

- 윈도 함수

  - 순위 함수

  - ```
    SELECE NAME,SALARY ,
    RANK() OVER (ORDER BY SALARY DESC) A,
    DENSE_RANK() OVER (ORDER BY SALARY DESC) B ,
    ROW_NUMBER() OVER (ORDER BY SALARY DESC) C
    FROM EMPLOYEE;
    ```

    

  - 행순서 함수

  - ```
    SELECE NAME,SALARY ,
    RANK(NAME) OVER (ORDER BY SALARY DESC) A,
    FIRST_VALUE(NAME) OVER (ORDER BY SALARY DESC) B ,
    LAG(NAME) OVER (ORDER BY SALARY DESC) C ,
    LEAD(NAME) OVER (ORDER BY SALARY DESC) D
    FROM EMPLOYEE;
    ```

  - 

  - 그룹 내 비율 함수

  - ```
    SELECT NAME,SALARY,
    RATIO_TO_PERCENT(NAME) OVER (ORDER BY SALARY DESC) A,
    PERCENT_RANK(NAME) OVER (ORDER BY SALARY DESC) B
    FROM EMPLOYEE;
    ```

  - 