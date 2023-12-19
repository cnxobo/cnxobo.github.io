---
title: Dorado、BDF EL表达式
tags:
  - bdf2
  - dorado
  - EL表达式
id: '366'
categories:
  - - dorado
date: 2014-09-06 11:38:02
---

隐式对象

## Dorado隐式对象

Dorado共提供了14个隐式对象,我们不需要预先定义就可以直接使用这些隐式对象。

#### 与线程无关

标识符

描述

null

 表示null

env

 表示获取系统环境变量即System.getenv()对象。${env.HOME}或${env\["HOME"\]}

system

 Java的系统属性,即System.getProperties()对象。${system.property1}或${system\["property1"\]}

configure

 用于获取Dorado7的配置信息。${configure\["core.runMode"\]}

#### 与线程有关

标识符

描述

argument

用于获取当前配置文件中的参数值，仅可用于View.xml

context

当前线程中的DoradoContext。

ctx

用于简化对DoradoContext的getAttribute方法的访问。 ${ctx\["foo"\]}相当于${context.getAttribute("foo")}

util

基本工具类，见com.bstek.dorado.core.el.ExpressionUtilsObject的javadoc,如:${util.getToday()}//获取当前日期${util.getDate('yyyy年MM月dd日 HH时mm分ss秒')}//指定日期格式

request

当前的HttpServletRequest对象,如：${request.getAttribute("foo")}

req

用于简化对request的getAttribute方法的访问。 **${req\["foo"\]}**相当于${request.getAttribute("foo")}

param

用于简化对request的getParameter方法的访问。 **${param\["foo"\]} or ${param.foo}**相当于${request.getParameter("foo")}

session

当前的HttpSession对象,与会话作用域属性的名称和值相关联的 Map 类。范例：${session.getAttribute("foo")}

servletContext

当前的SerlvetContext对象。范例：${servletContext.getAttribute("foo")}web基本工具类，见com.bstek.dorado.web.WebExpressionUtilsObject的javadoc

res

国际化

acomp

自定义控件ID

web

getDataProvider

${dorado.getDataProvider("beanId#methodName").getResult(argument)}

## BDF2隐式对象

标识符

描述

loginUser

当前登录用户 ContextHolder.getLoginUser()

loginUsername

当前登录用户名 ContextHolder.getLoginUser()==null?null:ContextHolder.getLoginUser().getUsername()

authenticationExceptionMessage

异常消息 this.getAuthenticationExceptionMessage()

## 扩展

通过实现 ContextVarsInitializer 接口并注册到 Spring 上下文，我们就可以添加自定义的 隐式对象。

package com.bstek.dorado.core.el;

import java.util.Map;

/\*\*
 \* EL上下文中隐式变量集合的初始化器。
 \* <p>
 \* EL上下文中的隐式变量常常被理解为EL表达式的前缀。<br>
 \* 例如，假设我们将一个{@link java.util.Date}以<code>Date</code>的前缀放入到隐式变量集合中， 即
 \*
 \* <pre>
 \* vars.put(&quot;Date&quot;, new java.util.Date(););
 \* </pre>
 \*
 \* 。<br>
 \* 而后，我们就可以在EL表达式中这样来使用该Date对象了
 \*
 \* <pre>
 \* ${Date.toLocaleString()}
 \* </pre>
 \*
 \* 。
 \* </p>
 \* @author Benny Bao (mailto:benny.bao@bstek.com)
 \* @since Mar 4, 2007
 \*/
public interface ContextVarsInitializer {
/\*\*
 \* 初始化隐式变量。
 \* @param vars 隐式变量映射集合。其中Map的键值为隐式变量的变量名，Map的值为隐式变量自身。
 \*/
void initializeContext(Map<String, Object> vars) throws Exception;
}