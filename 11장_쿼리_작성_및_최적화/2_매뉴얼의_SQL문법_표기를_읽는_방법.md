# 11.2 매뉴얼의 SQL 문법 표기를 읽는 방법

MySQL 매뉴얼의 SQL 문법을 표기하는 방법

> 매뉴얼에 명시된 SQL 문법은 한눈에 이해되지 않는다는 단점이 있지만 해당 버전에 맞는 SQL 문법을 참조하기에 가장 정확한 자료

```SQL
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition [, partition_name] ...)]
    [(coI name [, col_name] ...)]
    {VALUES | VALUE} (value_ list) [, (value_ list)]...
    [ON DUPLICATE KEY UPDATE assignment_list]

value: {expr | DEFAULT}
value_list： value [, value]
assignment: col_name = value
assignment_list: assignment [, assignment] ...
```

- **대문자** : 모두 키워드를 의미

  - 키워드는 대소문자 구분하지 않고 사용 가능

- **이탤릭체** : 사용자가 선택해서 작성하는 토큰으로 대부분 테이블명이나 칼럼명, 표현식을 사용

  - 이 항목이 SQL 키워드나 식별자가 아닌 경우 `"value"` 나 `"value_list"`와 같이 상세한 문법 설명

- **대괄호** : 해당 키워드나 표현식 자체가 선택 사항

  - 대괄호로 묶여있는 부분은 없어도 오류 발생 X

- **파이프** : 앞과 뒤의 키워드나 표현 식 중 하나만 선택해서 사용 가능

  - **예시**

    - 첫번째 라인의 `LOW_PRIORITY`, `DELAYED`, `HIGH_PRIORITY` 중 하나만 선택해서 사용 가능
    - 대괄호로 묶여있기 때문에 `INSERT`와 `INTO` 키워드 사이에 아무것도 사용하지 않거나 셋 중 하나만 사용 가능

- **중괄호** : 괄호 내의 아이템 중 반드시 하나 사용

  - **예시**
    - VALUES나 VALUE 중 하나의 값 반드시 사용해야함

- **...** : 앞에 명시된 키워드나 표현식의 조합이 반복될 수 있음
