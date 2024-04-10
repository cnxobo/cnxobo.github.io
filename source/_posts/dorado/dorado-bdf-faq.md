---
title: dorado & BDF常见问题
tags:
  - bdf
  - dorado
  - Spring Boot
id: '537'
categories:
  - - dorado
date: 2017-05-08 14:06:02
---

部分记录了我在 使用 dorado 开发当中遇到的问题，该问题列表会不断补充，也同时欢迎在评论里补充新问题。

# 相关网站

**dorado nexus 私服** [http://nexus.bsdn.org](http://nexus.bsdn.org "dorado nexus 私服" target="_blank") 
**dorado client api** [http://dorado7.bsdn.org/jsdoc/](http://dorado7.bsdn.org/jsdoc/ "dorado client api" target="_blank") 
**wiki** [http://wiki.bsdn.org/](http://wiki.bsdn.org/ "wiki" target="_blank") 
**BDF2项目创建向导** [http://bsdn.org/projects/bdf/deploy/bdf2-new-project-wizard/view.Wizard.d](http://bsdn.org/projects/bdf/deploy/bdf2-new-project-wizard/view.Wizard.d "BDF2项目创建向导" target="_blank") 
**dorado eclipse plugin** 
百度网盘: [https://pan.baidu.com/s/1chalW5ebFOC3cKkYJLvkig](https://pan.baidu.com/s/1chalW5ebFOC3cKkYJLvkig  "百度网盘" target="_blank") 提取码:xobo

# Dorado

#### Q: 想在 Spring Boot 环境下使用 dorado

**A:** BDF3 就是基于 Spring Boot2 的，可以在 BDF3 的基础之上开发或学习其如何把 dorado 和 Spring Boot2 集成在一起的。 https://github.com/muxiangqiu/bdf3.git

## Dorado IDE

#### Q: 打开view 文件时"Loading Model...遇到问题"。

**A:** 需要更新 dorado 规则， ![更新 dorado IDE 规则](https://i.loli.net/2021/04/22/JKCSLUd2Z6yGh3j.jpg "更新 dorado IDE 规则") 具体可以参考[dorado配置规则](https://xobo.org/dorado-config-rule)。建议使用在线方式更新。 在线模式下 ServerName对应着域名或ip, Port就是端口号, App Name就是contextpath，如果为空可以不填。图片中是示例访问地址为: http://localhost:8080/waterdrop 。如果 项目访问地址是: http://localhost:8080

> Server Name: localhost Port: 8080 App Name:

## Widget

### AjaxAction

#### Q: AjaxAction如何根据后台执行的或失败执行不同的Javascript代码?

**A:** ajaxAction 的对应的后台 @Expose 方法，返回一个字符串。如果返回空就是执行成功，错误就把错误信息返回。然后在回调函数中根据返回值，进行不同的判断。

```javascript
ajaxAction.execute(function(msg){
    if(!msg) {
        // do ok
    } else {
        // do error
    }
})；
```

### View

#### Q: 如何切换皮肤？

**A:**

```Java
WebConfigure.set(DoradoContext.SESSION, "view.skin", skinName);
```

#### Q: 为什么我的控件显示是英文

**A:** dorado 默认是支持多语言的，dorado 的默认控件会根据HTTP Header中 accept-language 的值来选择显示语言。如果没有多语言需求的话可以把语言固定下来。在dorado home目录下的context.xml里添加如下配置:

```XML
<bean id="dorado.localeResolver"
        class="com.bstek.dorado.view.resource.SpringLocaleResolverAdapter">
        <property name="springLocaleResolver">
            <bean
                class="org.springframework.web.servlet.i18n.FixedLocaleResolver">
                <constructor-arg index="0" value="zh_CN" />
            </bean>
        </property>
    </bean>
```

#### Q: 如何在外部获取 dorado 控件？

**A:** 在 dorado 控件的事件内，可以通过 view.id/get 等方法来获取其它 dorado 控件。对于事件以外的地方作用域里没有 view 对象，这个时候我们可以使用 viewMain 或 $id来获取。 viewMain 其实就是 view 但是在dorado7 低版本中不支持； $id 只能通过控件 id 来选取控件，而且其返回的是dorado.ObjectGroup,我们需要通过.objects0来获取具体的控件。 $id("widgetId").objects\[0\]

### DataSet

#### Q: 为什么DataProvider的代码里加载出数据，而的dataSet 中没有呢？

**A:** 如果开启分页，即 dataSet 中 pageSize 值大于 0， dataSet 会从 page.entities 内取值。

#### Q: 设置的DataProvider明明存在，为什么会提示无法在"XXXXX"类中查找到唯一匹配的"XXXX"方法?

**A:**如果开启分页，@DataProvider 标记的方法必须有 Page page 属性，其中T 为相应的 POJO 类。

#### Q: 为什么 DataSet刷新后中有数据，用调试器也能取出来为什么JS代码取不出来呢？

**A:** DataSet 的刷新有两个方法，同步与异步 flush, flushAsync。 最直白的理解使用 flush 刷新数据时，代码会等数据加载完成之后再执行后续代码；使用flushAsync刷新数据时，先处理后续代码，最后处理数据加载。 所以遇到该情况，多是使用了 flushAsync 刷新 DataSet 然后直接在下一行代码里就从 DataSet 中取数据。建议使用异步加回调的方式。

```javascript
dataSet.flushAsync(function(){
  //get data from dataSet
});
```

### AutoForm

#### Q: 为什么AutoForm 的 entity 有时候是 dorado.Entity 有时候是 JavaScript Object?

**A:** AutoForm 绑定 dataSet 时返回的是 entity，如果没有返回的是 JavaScript Object。aufoform 一直是与 dataSet 同时出现的，如果确实不需要 dataSet，可以将 AutoForm 的 createPrivateDataSet 属性设为 true,它会自行创建一个自有 DataSet。

#### Q: 如何在一组输入框内，实现选择日期区间选择，如下图所示。

[![日期区间选择](https://i.loli.net/2018/10/10/5bbdca714ff97.png "日期区间选择")](https://i.loli.net/2018/10/10/5bbdca714ff97.png "日期区间选择") **A:** 详见DateRangeDemo.view.xml附件。 [DateRangeDemo.view.xml](http://xobo.org/wp-content/uploads/2017/05/DateRangeDemo.view_.xml_.zip)

#### Q: 如何限制文件框只能输入两个字符

**A:** 可以利用editor的onKeyPress事件

```JavaScript
view.get("#autoFormContractMain.#amount.editor").bind("onKeyPress", function(self, arg) {
    var orignText = self.doGetText();
    var keyCode = arg.keyCode;
    if (orignText && orignText.length>1) {
        arg.returnValue = false;
    }
})
```

### DataGrid

#### Q: 重新加载数据后，dataGrid 启用 client filter 时，dataGrid 数据为什么不更新？

**A:** 这个可以确认为 dorado7 的 Bug, 目前通过以下代码手动刷新 dataGrid。
```JavaScript
function resetFilteredDataGrid(dataGrid){
    var filterEntity = dataGrid.get("filterEntity");
    var lastCriteria = filterEntity.toJSON();
    filterEntity.fromJSON({});
    dataGrid.filter();
    setTimeout(function(){
        filterEntity.fromJSON(lastCriteria);
        dataGrid.filter();
        }, 100);
} 
```
#### Q: DataGrid 如何设置某些数据不能选中

在 RowSelectorColumn 的 onRenderCell 事件里，修改默认渲染动作。

```JavaScript
var selectable = function(){
    // 计算是否可选
    return ;
};
if (selectable()) {
    arg.processDefault = true;
} else {
    jQuery(arg.dom).empty();
    arg.processDefault = false;
}
```

#### Q: 如何在 DataGrid 的 Column上显示按钮？

_A:_ 可以在 DataColumn 的 onRenderCell 事件里重写渲染方法。

```JavaScript
    jQuery(arg.dom).empty().xCreate({
        tagName: "button",
        content: "查看",
        style: { // 定义按钮的style
            color: "#1b8ce0",
            "font-weight": "bold"
        },
        onclick: function(){ // 定义onclick事件
            view.id("dialog").show();
        }
    });
    arg.processDefault = false; // 禁止默认渲染
```

#### Q: 如何在 DataGrid 的 Column上显示链接？

_A:_ 可以在 DataColumn 的 onRenderCell 事件里重写渲染方法。

```JavaScript
xobo.renderClickLink(arg, '详情', function(){
    dialog.show();
});

$namespace("xobo")
xobo.renderClickLink = function(arg, content, callback, config){
    if (!config) {
      config = {};
    }
    jQuery(arg.dom).empty().xCreate({
      tagName: "a",
      content: content  "",
      style: config.style 
      {
        "color": "#1b8ce0",
        "font-weight": "bold",
      }
    });
    jQuery(arg.dom).css("cursor", config.cursor  "pointer");
    if (click) {
      jQuery(arg.dom).click(callback);
    }
    arg.processDefault = false;
  }
```

### DownloadAction

下载控件，我用过的最难用的 dorado 控件之一，极不推荐使用。该 Action 提交是采用表单提交的方式，一旦在该 action 的后台处理上遇到异常直接就把用户页面转到 5xx 的错误页面去了。

#### Q: 为什么通过 DownloadAction 的方式传递到后台的代码有乱码?

A: 在web.xml中配置编解码Filter(org.springframework.web.filter.CharacterEncodingFilter). 一定要做为第一个 fitler才行。

```XML
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### Q: 为什么通过DownloadAction 传递到后台的参数都变成 String ？

**A:** 对于 dorado 的绝大多数 action 控件来说，能准确的把前台赋值给 parameter 属性的值在后台转换为相应的 Java 值。但是 downloadAction 会把 parameter 的值都转换为字符串。已经习惯了把 dorado 对象直接丢到 parameter 里传到后台的，在这里肯定要踩坑。兼容方案就是在前台把要传递的值先转化为 JSON 字符串，然后把这个字符串传到后台，在后台再把字符串转回 Map。dorado 提供了工具类 dorado.JSON.stringify 来把对象转化为 JSON 字符串。

### UploadAction

#### Q: 如何给uploadAction添加阻塞的任务指示器?

**A:**

```JavaScript
 xobo.maskUpload = function(uploadAction, msg) {
    if (!uploadAction) {
      return;
    }
    uploadAction.bind("onError", function(self, arg) {
      dorado.util.TaskIndicator.hideTaskIndicator(self.taskId);
    });
    uploadAction.bind("beforeFileUpload", function(self, arg) {
      self.taskId = dorado.util.TaskIndicator.showTaskIndicator(msg  "上传中...", "main");
    });

    uploadAction.bind("onFileUploaded", function(self, arg) {
      dorado.util.TaskIndicator.hideTaskIndicator(self.taskId);
    });
  }

 xobo.maskUpload('uploadActionId');
```

### dropdown

#### Q: DataSetDropDown 添加自定义列之后，如何显示过滤框?

**A:** 在 DataSetDropDown 的 onOpen 事件里，添加如下代码:

```JavaScript
setTimeout(function(){
    self.get("box.control").set("showFilterBar", true)
}, 200)
```

### 日期选择器

#### Q: 如何把日期选择器的默认日期改为输入框的值？

**A:** dorado 的日期选择框默认打开的时候都是当前时间。创建一个新的DateDropDown控件，然后在其 onCreate 事件中插入如下代码:
```JavaScript
self.initDropDownBox = function(box, editor){
    var dropDown = this, datePicker = dropDown.get("box.control");
    if (datePicker) {
        var date = editor.get("value");
        var year, month;
        if (typeof date == "string") {
           date = new Date(date);
        } else {
            if (!(date instanceof Date)) {
                date = new Date();
            }
        }
        datePicker.set("date", date);
    }
};
```
## 杂项

### dorado滚动条

dorado滚动条默认是窄窄的一条，只有鼠标指上去之后才会变宽。但是对于部分客户这个设定是不可以接受的。又由于 dorado 布局相关的原因，没有计划去支持实体的滚动条。只能通过调整滚动条的宽度来照顾该部分客户。 我们需要配置如下两个属性: widget.scrollerSize是默认滚动条宽度; widget.scrollerExpandedSize是展开后滚动条宽度，建议设置为 10.

dorado.Setting\["widget.scrollerSize"\]=10;
dorado.Setting\["widget.scrollerExpandedSize"\]=10;

### 文件下载

触发文件下载有两种方式

1.  使用 window.location.href = "downloadUrl";
2.  通过 iframe.
```JavaScript
function downloadFile(url){
    var iframeId = "iframeDownloadFile";
    var iframe = document.getElementById(iframeId);
    if (!iframe) {
        iframe = document.createElement('iframe');
        iframe.style.display = "none";
        iframe.id = iframeId;
        document.body.appendChild(iframe);
    }
    iframe.src = url;
}
```
#### 文件下载Java端代码片段
```Java
  public void download(String filename, InputStream inputStream, HttpServletResponse response)
      throws IOException {
    response.setContentType("application/octet-stream");

    String encodFilename = URLEncoder.encode(filename, "utf-8");
    response.setHeader("Content-Disposition",
        String.format("attachment; filename=\\"%1$s\\"; filename\*=utf-8''%1$s", encodFilename));

    OutputStream outputStream = null;
    try {
      outputStream = response.getOutputStream();
      IOUtils.copy(inputStream, outputStream);
    } catch (FileNotFoundException fne) {
    } finally {
      IOUtils.closeQuietly(inputStream);
      IOUtils.closeQuietly(outputStream);
    }
  }
```
### 父子页面交互

在一些业务场景下，我们会需要在子页面来操作父页面的元素，或者反过来。  
```JavaScript
// sub page
top.doSth = function(params){
    //do what you want
}

// parent page
if (jQuery.isFunction(top.doSth)){
    top.doSth(datas);
}
```
## 性能

#### Q: 在前台大批量操作数据，页面很卡怎么办?

**A:** 在批量操作时禁止dataSet通知观察者(dataSet.disableObservers())，等操作完之后启用并通知出来。

```JavaScript
// 禁止DataSet将消息发送给其观察者。
dataSet.disableObservers();

// 做批量操作
doBatchOperations();

// 允许DataSet将消息发送给其观察者。
dataSet.enableObservers();
// 通知DataSet的所有观察者刷新数据。
dataSet.notifyObservers();
```

# BDF2

### Maven 依赖

**Q: BDF2 突然不能正常启动，报NoSuchMethodError异常:**

> Initialization of bean failed; nested exception is java.lang.NoSuchMethodError: com.bstek.bdf2.core.context.ContextHolder.getApplicationContext()Lorg/springframework/web/context/WebApplicationContext;

**A:** bdf2-orm(v2.1.0)为了兼容 Spring Boot 把ContextHolder.getApplicationContext()的返回值由WebApplicationContext改为ApplicationContext, 而 bdf2 的其它模块基本上都依赖于最新的 bdf2-orm, 导致 bdf2 的maven 项目出现不兼容的新老模块混用情况以致抛出方法未找到异常。 解决办法就是统一 bdf2 的 jar 包依赖： 1：升级其它 BDF2 模块到 2.1.0 及其以后版本； 2: 降级bdf2-orm。 Maven 项目可以通过排除依赖的bdf2-orm，然后指定bdf2-orm的版本为 2.0.7。 ![](http://xobo.org/wp-content/uploads/2017/05/maven-exclude-300x100.png) 添加依赖:

 com.bstek.bdf2
  bdf2-orm-hibernate3
  2.0.7 

### Home

**Q: BDF2 如何通过JavaScript在首页中打开新的Tab页:** **A:** 在子页面调用 top.openUrlInFrameTab 方法。

```JavaScript
window.openUrlInFrameTab(url, name, icon);
// example
window.openUrlInFrameTab("Test.d", "测试页面", "");
```

### eclipse
#### xsd
因为各种原因， xsd 文件加载缓慢或者网站升级文件会找不到，导致xml文件校验慢甚至失败。可以在eclipse指定xsd。

![[image-20240408142608724.png]]


[bdf2.0.xsd下载](bdf2.0.xsd)