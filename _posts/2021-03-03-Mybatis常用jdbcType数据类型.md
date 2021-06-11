---
layout: post
title:  "Mybatis常用jdbcType数据类型"
imges: 
date:   2021-03-03 10:02:00 +0800
description: 
tags: java
---


###  MyBatis 通过包含的jdbcType类型 



| BIT      | FLOAT   | CHAR        | TIMESTAMP     | OTHER   | UNDEFINED |
| TINYINT  | REAL    | VARCHAR     | BINARY        | BLOB    | NVARCHAR  |
| SMALLINT | DOUBLE  | LONGVARCHAR | VARBINARY     | CLOB    | NCHAR     |
| INTEGER  | NUMERIC | DATE        | LONGVARBINARY | BOOLEAN | NCLOB     |
| BIGINT   | DECIMAL | TIME        | NULL          | CURSOR  |           |



### Mybatis  中  javaType  和  jdbcType  对应和CRUD例子

**Xml代码** 

```xml
<resultMap type="java.util.Map" id="resultMap">
  <result property="FLD_NUMBER" column="FLD_NUMBER"  javaType="double" jdbcType="NUMERIC"/>
  <result property="FLD_VARCHAR" column="FLD_VARCHAR" javaType="string" jdbcType="VARCHAR"/>
  <result property="FLD_DATE" column="FLD_DATE" javaType="java.sql.Date" jdbcType="DATE"/>
  <result property="FLD_INTEGER" column="FLD_INTEGER"  javaType="int" jdbcType="INTEGER"/>
  <result property="FLD_DOUBLE" column="FLD_DOUBLE"  javaType="double" jdbcType="DOUBLE"/>
  <result property="FLD_LONG" column="FLD_LONG"  javaType="long" jdbcType="INTEGER"/>
  <result property="FLD_CHAR" column="FLD_CHAR"  javaType="string" jdbcType="CHAR"/>
  <result property="FLD_BLOB" column="FLD_BLOB"  javaType="[B" jdbcType="BLOB" />
  <result property="FLD_CLOB" column="FLD_CLOB"  javaType="string" jdbcType="CLOB"/>
  <result property="FLD_FLOAT" column="FLD_FLOAT"  javaType="float" jdbcType="FLOAT"/>
  <result property="FLD_TIMESTAMP" column="FLD_TIMESTAMP"  javaType="java.sql.Timestamp" jdbcType="TIMESTAMP"/>
 </resultMap> 
```



###   Mybatis  中  javaType  和  jdbcType  对应关系

 ```
Notepad++代码

1.  JDBC Type      		Java Type 
2.  CHAR        		String 
3.  VARCHAR       		String 
4.  LONGVARCHAR     	        String 
5.  NUMERIC       		java.math.BigDecimal 
6.  DECIMAL       		java.math.BigDecimal 
7.  BIT    			   boolean 
8.  BOOLEAN       		boolean 
9.  TINYINT       		byte 
10. SMALLINT      		short 
11. INTEGER       		int 
12. BIGINT       		long 
13. REAL        		float 
14. FLOAT        		double 
15. DOUBLE       		double 
16. BINARY       		byte[] 
17. VARBINARY      		byte[] 
18. LONGVARBINARY               byte[] 
19. DATE        		java.sql.Date 
20. TIME        		java.sql.Time 
21. TIMESTAMP      		java.sql.Timestamp 
22. CLOB        		Clob 
23. BLOB        		Blob 
24. ARRAY        		Array 
25. DISTINCT      		mapping of underlying type 
26. STRUCT       		Struct 
27. REF             	        Ref 
28. DATALINK      		java.net.URL[color=red][/color] 
 ```





