---
title: "mybatis Cause: java.lang.NumberFormatException: For input string:"
tags:
  - MyBatis
categories: Java
date: 2025-06-10
---

OGNL 支持使用单引号和双引号来表达字符串，但是使用单引号且只有一个字符会被识别为字符(Character)。字符串和字符比较时会先转化为 `double` 再比较就导致标题里遇到的异常。
### 错误写法:
```XML
<if test="sessionId != null and sessionId != '' and sessionId!='*'">  
    AND t.session_id = #{sessionId}  
</if>
```
### 正确写法:

1. 使用双引号包裹字符

```XML
<if test='sessionId != null and sessionId != "" and sessionId != "*"'>  
    AND t.session_id = #{sessionId}  
</if>
```

2. 把Character转化为String
```XML
<if test="sessionId != null and sessionId != '' and sessionId!='*'.toString()">  
    AND t.session_id = #{sessionId}  
</if>
```

根据org.apache.ibatis.ognl.OgnlOps#compareWithConversion方法。非数字类型的比较会通过 `Comparable` 接口或者 `Enum` 的 `compareTo` 方法比较,非数字类型尝试和数字类型比较时会先转化为 `double` 类型再比较。`Character` 是数字数字类型(根据 `getNumericType` 方法)。

####  单元测试示例
```Java
@Test  
public void testQuoteChar() throws OgnlException {  
    Map<String, Object> params = Map.of("sessionId", "abc");  
    Assert.assertTrue((boolean) Ognl.getValue("sessionId != null and sessionId != '' and sessionId!='*'.toString()",  
            params));  
    Assert.assertTrue((boolean) Ognl.getValue("sessionId != null and sessionId != \"\" and sessionId!=\"*\"",  
            params));  
}
```