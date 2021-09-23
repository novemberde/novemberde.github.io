---
title: MySQL에서 DB스키마 작성시 주의할 점들
tags: [db, mysql, schema, 스키마, 주의, database]
date: "2019-06-10T11:30:03+00:00"
aliases: ["/database/2019/06/10/MySQL-Desgin-cheatsheet.html"]
ShowBreadCrumbs: true
ShowReadingTime: true
ShowPostNavLinks: true
---
신규프로젝트를 진행할 경우 디비 스키마를 설계를 해야한다.
데이터의 타입부터 외래키 설정등 신경써야하는 부분들이 생각보다 자잘하게 있다.
이 경우에 주의해야하는 점들을 정리해보았다.
이 문서는 지속적으로 수정될 예정이다.

## Datatype

MySQL에서 사용하는 데이터 타입은 다음과 같다.

### Numeric Type

- BIT[(M)] - A bit-value type. M indicates the number of bits per value, from 1 to 64. The default is 1 if M is omitted.
- TINYINT[(M)] [UNSIGNED] [ZEROFILL] - A very small integer. The signed range is -128 to 127. The unsigned range is 0 to 255.
- BOOL, BOOLEAN - These types are synonyms for TINYINT(1). A value of zero is considered false. Nonzero values are considered true. However, the values TRUE and FALSE are merely aliases for 1 and 0, respectively, as shown here.
- SMALLINT[(M)] [UNSIGNED] [ZEROFILL] - A small integer. The signed range is -32768 to 32767. The unsigned range is 0 to 65535.
- MEDIUMINT[(M)] [UNSIGNED] [ZEROFILL] - A medium-sized integer. The signed range is -8388608 to 8388607. The unsigned range is 0 to 16777215.
- INT[(M)] [UNSIGNED] [ZEROFILL] - A normal-size integer. The signed range is -2147483648 to 2147483647. The unsigned range is 0 to 4294967295.
- INTEGER[(M)] [UNSIGNED] [ZEROFILL] - This type is a synonym for INT.
- BIGINT[(M)] [UNSIGNED] [ZEROFILL] - A large integer. The signed range is -9223372036854775808 to 9223372036854775807. The unsigned range is 0 to 18446744073709551615. SERIAL is an alias for BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE.

>Some things you should be aware of with respect to BIGINT columns:
>- All arithmetic is done using signed BIGINT or DOUBLE values, so you should not use unsigned big integers larger than 9223372036854775807 (63 bits) except with bit functions! If you do that, some of the last digits in the result may be wrong because of rounding errors when converting a BIGINT value to a DOUBLE.
>
>  MySQL can handle BIGINT in the following cases:
>
>   - When using integers to store large unsigned values in a BIGINT column.
>
>   - In MIN(col_name) or MAX(col_name), where col_name refers to a BIGINT column.
>
>   - When using operators (+, -, *, and so on) where both operands are integers.
>
>- You can always store an exact integer value in a BIGINT column by storing it using a string. In this case, MySQL performs a string-to-number conversion that involves no intermediate double-precision representation.
>
>- The -, +, and * operators use BIGINT arithmetic when both operands are integer values. This means that if you multiply two big integers (or results from functions that return integers), you may get unexpected results when the result is larger than >9223372036854775807.

- DECIMAL[(M[,D])] [UNSIGNED] [ZEROFILL]
>A packed “exact” fixed-point number. M is the total number of digits (the precision) and D is the number of digits after the decimal point (the scale). The decimal point and (for negative numbers) the - sign are not counted in M. If D is 0, values have no decimal point or fractional part. The maximum number of digits (M) for DECIMAL is 65. The maximum number of supported decimals (D) is 30. If D is omitted, the default is 0. If M is omitted, the default is 10.
>
>UNSIGNED, if specified, disallows negative values.
>
>All basic calculations (+, -, *, /) with DECIMAL columns are done with a precision of 65 digits.

- DEC[(M[,D])] [UNSIGNED] [ZEROFILL], NUMERIC[(M[,D])] [UNSIGNED] [ZEROFILL], FIXED[(M[,D])] [UNSIGNED] [ZEROFILL] - These types are synonyms for DECIMAL. The FIXED synonym is available for compatibility with other database systems.

- FLOAT[(M,D)] [UNSIGNED] [ZEROFILL] - A small (single-precision) floating-point number. Permissible values are -3.402823466E+38 to -1.175494351E-38, 0, and 1.175494351E-38 to 3.402823466E+38. These are the theoretical limits, based on the IEEE standard. The actual range might be slightly smaller depending on your hardware or operating system.

- FLOAT(p) [UNSIGNED] [ZEROFILL] - A floating-point number. p represents the precision in bits, but MySQL uses this value only to determine whether to use FLOAT or DOUBLE for the resulting data type. If p is from 0 to 24, the data type becomes FLOAT with no M or D values. If p is from 25 to 53, the data type becomes DOUBLE with no M or D values. The range of the resulting column is the same as for the single-precision FLOAT or double-precision DOUBLE data types described earlier in this section.

- DOUBLE[(M,D)] [UNSIGNED] [ZEROFILL] - A normal-size (double-precision) floating-point number. Permissible values are -1.7976931348623157E+308 to -2.2250738585072014E-308, 0, and 2.2250738585072014E-308 to 1.7976931348623157E+308. These are the theoretical limits, based on the IEEE standard. The actual range might be slightly smaller depending on your hardware or operating system.

- DOUBLE PRECISION[(M,D)] [UNSIGNED] [ZEROFILL], REAL[(M,D)] [UNSIGNED] [ZEROFILL] - These types are synonyms for DOUBLE. Exception: If the REAL_AS_FLOAT SQL mode is enabled, REAL is a synonym for FLOAT rather than DOUBLE.

### Date and Time Type

- DATE - A date. The supported range is '1000-01-01' to '9999-12-31'. MySQL displays DATE values in 'YYYY-MM-DD' format, but permits assignment of values to DATE columns using either strings or numbers.
- DATETIME[(fsp)]
  - A date and time combination. The supported range is '1000-01-01 00:00:00.000000' to '9999-12-31 23:59:59.999999'. MySQL displays DATETIME values in 'YYYY-MM-DD HH:MM:SS[.fraction]' format, but permits assignment of values to DATETIME columns using either strings or numbers.
  - An optional fsp value in the range from 0 to 6 may be given to specify fractional seconds precision. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.
  - Automatic initialization and updating to the current date and time for DATETIME columns can be specified using DEFAULT and ON UPDATE column definition clauses, as described in Section 11.3.4, “Automatic Initialization and Updating for TIMESTAMP and DATETIME”.
- TIMESTAMP[(fsp)]
  - A timestamp. The range is '1970-01-01 00:00:01.000000' UTC to '2038-01-19 03:14:07.999999' UTC. TIMESTAMP values are stored as the number of seconds since the epoch ('1970-01-01 00:00:00' UTC). A TIMESTAMP cannot represent the value '1970-01-01 00:00:00' because that is equivalent to 0 seconds from the epoch and the value 0 is reserved for representing '0000-00-00 00:00:00', the “zero” TIMESTAMP value.
  - An optional fsp value in the range from 0 to 6 may be given to specify fractional seconds precision. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.
  - The way the server handles TIMESTAMP definitions depends on the value of the explicit_defaults_for_timestamp system variable (see Section 5.1.8, “Server System Variables”).
  - If explicit_defaults_for_timestamp is enabled, there is no automatic assignment of the DEFAULT CURRENT_TIMESTAMP or ON UPDATE CURRENT_TIMESTAMP attributes to any TIMESTAMP column. They must be included explicitly in the column definition. Also, any TIMESTAMP not explicitly declared as NOT NULL permits NULL values.
  - If explicit_defaults_for_timestamp is disabled, the server handles TIMESTAMP as follows:
  - Unless specified otherwise, the first TIMESTAMP column in a table is defined to be automatically set to the date and time of the most recent modification if not explicitly assigned a value. This makes TIMESTAMP useful for recording the timestamp of an INSERT or UPDATE operation. You can also set any TIMESTAMP column to the current date and time by assigning it a NULL value, unless it has been defined with the NULL attribute to permit NULL values.
  - Automatic initialization and updating to the current date and time can be specified using DEFAULT CURRENT_TIMESTAMP and ON UPDATE CURRENT_TIMESTAMP column definition clauses. By default, the first TIMESTAMP column has these properties, as previously noted. However, any TIMESTAMP column in a table can be defined to have these properties.

- TIME[(fsp)]
  - A time. The range is '-838:59:59.000000' to '838:59:59.000000'. MySQL displays TIME values in 'HH:MM:SS[.fraction]' format, but permits assignment of values to TIME columns using either strings or numbers.
  - An optional fsp value in the range from 0 to 6 may be given to specify fractional seconds precision. A value of 0 signifies that there is no fractional part. If omitted, the default precision is 0.
- YEAR[(4)] - A year in four-digit format. MySQL displays YEAR values in YYYY format, but permits assignment of values to YEAR columns using either strings or numbers. Values display as 1901 to 2155, and 0000.

>The SUM() and AVG() aggregate functions do not work with temporal values. (They convert the values to numbers, losing everything after the first nonnumeric character.) To work around this problem, convert to numeric units, perform the aggregate operation, and convert back to a temporal value. 

### String Type

>- CHARACTER SET specifies the character set. If desired, a collation for the character set can be specified with the COLLATE attribute, along with any other attributes.(latin1, utf-8...etc). CHARSET is a synonym for CHARACTER SET.
>- Specifying the CHARACTER SET binary attribute for a character string data type causes the column to be created as the corresponding binary string data type: CHAR becomes BINARY, VARCHAR becomes VARBINARY, and TEXT becomes BLOB. For the ENUM and SET data types, this does not occur; they are created as declared.
>- The BINARY attribute is shorthand for specifying the table default character set and the binary (_bin) collation of that character set. In this case, comparison and sorting are based on numeric character code values.
>- The ASCII attribute is shorthand for CHARACTER SET latin1.
>- The UNICODE attribute is shorthand for CHARACTER SET ucs2.

- [NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name] - A fixed-length string that is always right-padded with spaces to the specified length when stored. M represents the column length in characters. The range of M is 0 to 255. If M is omitted, the length is 1.
- [NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name] - A variable-length string. M represents the maximum column length in characters. The range of M is 0 to 65,535. The effective maximum length of a VARCHAR is subject to the maximum row size (65,535 bytes, which is shared among all columns) and the character set used. For example, utf8 characters can require up to three bytes per character, so a VARCHAR column that uses the utf8 character set can be declared to be a maximum of 21,844 characters.
- BINARY[(M)] - The BINARY type is similar to the CHAR type, but stores binary byte strings rather than nonbinary character strings. An optional length M represents the column length in bytes. If omitted, M defaults to 1.
- VARBINARY(M) - The VARBINARY type is similar to the VARCHAR type, but stores binary byte strings rather than nonbinary character strings. M represents the maximum column length in bytes.
- TINYBLOB - A BLOB column with a maximum length of 255 (28 − 1) bytes. Each TINYBLOB value is stored using a 1-byte length prefix that indicates the number of bytes in the value.
- TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name] - A TEXT column with a maximum length of 255 (28 − 1) characters. The effective maximum length is less if the value contains multibyte characters. Each TINYTEXT value is stored using a 1-byte length prefix that indicates the number of bytes in the value.
- BLOB[(M)] - A BLOB column with a maximum length of 65,535 (216 − 1) bytes. Each BLOB value is stored using a 2-byte length prefix that indicates the number of bytes in the value. An optional length M can be given for this type. If this is done, MySQL creates the column as the smallest BLOB type large enough to hold values M bytes long.
- TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name] - A TEXT column with a maximum length of 65,535 (216 − 1) characters. The effective maximum length is less if the value contains multibyte characters. Each TEXT value is stored using a 2-byte length prefix that indicates the number of bytes in the value. An optional length M can be given for this type. If this is done, MySQL creates the column as the smallest TEXT type large enough to hold values M characters long.
- MEDIUMBLOB - A BLOB column with a maximum length of 16,777,215 (224 − 1) bytes. Each MEDIUMBLOB value is stored using a 3-byte length prefix that indicates the number of bytes in the value.
- MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name] - A TEXT column with a maximum length of 16,777,215 (224 − 1) characters. The effective maximum length is less if the value contains multibyte characters. Each MEDIUMTEXT value is stored using a 3-byte length prefix that indicates the number of bytes in the value.
- LONGBLOB - A BLOB column with a maximum length of 4,294,967,295 or 4GB (232 − 1) bytes. The effective maximum length of LONGBLOB columns depends on the configured maximum packet size in the client/server protocol and available memory. Each LONGBLOB value is stored using a 4-byte length prefix that indicates the number of bytes in the value.
- LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name] - A TEXT column with a maximum length of 4,294,967,295 or 4GB (232 − 1) characters. The effective maximum length is less if the value contains multibyte characters. The effective maximum length of LONGTEXT columns also depends on the configured maximum packet size in the client/server protocol and available memory. Each LONGTEXT value is stored using a 4-byte length prefix that indicates the number of bytes in the value.
- ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]
  - An enumeration. A string object that can have only one value, chosen from the list of values 'value1', 'value2', ..., NULL or the special '' error value. ENUM values are represented internally as integers.
  - An ENUM column can have a maximum of 65,535 distinct elements.
  - The maximum supported length of an individual ENUM element is M <= 255 and (M x w) <= 1020, where M is the element literal length and w is the number of bytes required for the maximum-length character in the character set.
- SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]
  - A set. A string object that can have zero or more values, each of which must be chosen from the list of values 'value1', 'value2', ... SET values are represented internally as integers.
  - A SET column can have a maximum of 64 distinct members.
  - The maximum supported length of an individual SET element is M <= 255 and (M x w) <= 1020, where M is the element literal length and w is the number of bytes required for the maximum-length character in the character set.

## Which is faster: char(1) or tinyint(1) ? Why?

[I think you should create column with ENUM('n','y'). Mysql stores this type in optimal way. It also will help you to store only allowed values in the field.](https://stackoverflow.com/questions/2023476/which-is-faster-char1-or-tinyint1-why)

## 고려할 점들

- 현재 운영하는 데이터베이스의 기본 Character Collation을 알아두어야한다. 생성할 테이블이 데이터베이스의 기본 Charset을 사용하는 것으로 옵션이 설정되어 있을 수 있기 때문이다. 
    - 예를들어, Amazon Aurora MySQL의 Default collation은 latin1으로 되어있다. 하지만 보통 MySQL Workbench에서는 utf-8을 default로 옵션이 설정되어있기 때문에 나중에 프로덕션에서 한글로 입력이 안되는 경우가 있다.
- ID값은 BIGINT를 추천한다. 나중에 서비스가 커져 INT를 BIGINT로 바꾸는 경우도 생각보다 만만치 않다. 고려할 점이 생긴다. 마음 편히 BIGINT로 ID를 지정하여 추후에 있을지도 모를 업무를 줄여야 한다.
- ENUM을 적극 활용한다. yes 또는 no를 'y'와 'n'으로 CHAR(1)로 선언하여 사용하는 경우가 있다. 이는 스키마의 콘텍스트를 이해하는데 명확하지 않을 뿐만 아니라
'a', '1' 등과 같은 다른 문자도 허용한다. 반면에 ENUM('yes', 'no')로 표현하면 명확하게 의미를 포함할 뿐만 아니라 코멘트가 없이 쉽게 컬럼이 가진 속성에 대해서 빠르게 이해할 수 있다.
- 테이블 명을 지을때는 복수형을 사용하지 않늗다. 가끔 orm에서 저절로 복수형으로 테이블을 생성해주는 경우가 있는데, 마지막 글자가 y이거나 s일 경우에 -ies나 -ses로 표현되어 상당히 골치아픈 경우가 있다.
명확하게 단수형으로 한다.
- 이모지를 지원하는 컬럼은 utf8-mb4로 charset을 설정하도록 한다.
- deleted_at과 같은 컬럼은 인덱스를 걸어줄 필요가 없다. 오히려 인덱스를 컬어주게되면 퍼포먼스에 영향을 끼친다.
- 인덱스를 걸어주는 컬럼을 선택하는 경우는 조회시 전체 데이터의 1%내로 WHERE절에서 조건을 걸어줄 때이다.
- 어지간한 데이터는 히스토리를 남기도록한다. UPDATE문 사용은 지양한다. 대신 INSERT를 하고 나중의 값을 불러올 때는 최신의 데이터만 가져온다.
- 테이블간의 외래키를 직접적으로 넣는 것에 대해서 고민을 해야한다. 1:N관계가 나은지 아니면 사이에 테이블을 하나 더 두어 N:M 관계의 테이블을 둘지 잘 생각한다.
1:N 관계의 테이블은 강한 결합이 되어 나중에 서비스 확장에 많은 난관에 봉착하게 된다. N:M 관계의 테이블을 하나 더 두게 된다면 후임자가 쉽게 이해못하는 경우도 있다.
쉬운 설계로 할 것인가 잘만들 것인가는 다른 문제이다.
- 한국식 영어를 쓰지 않도록 한다. 찾아보면 좋은 컬럼명들이 있다. 글로벌 서비스가 목표라면 대충짓지 않도록한다.
- Push service를 위한 토큰 저장테이블은 ad_id값과 함께 따로 두도록 한다. 여러 디바이스를 사용하는 사용자일 경우에 모든 디바이스에 알림을 던질 수 있다.
가끔 유저테이블에 두는 FCM 토큰값을 두는 경우가 있는데 이 때문에 추후에 코드를 수정하는게 굉장히 귀찮다.


## References

- [https://dev.mysql.com/doc/refman/8.0/en/data-types.html](https://dev.mysql.com/doc/refman/8.0/en/data-types.html)
- [https://stackoverflow.com/questions/2023476/which-is-faster-char1-or-tinyint1-why](https://stackoverflow.com/questions/2023476/which-is-faster-char1-or-tinyint1-why)

> 이 문서는 지속적으로 업데이트 될 예정입니다.