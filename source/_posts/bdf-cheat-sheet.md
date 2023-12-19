---
title: BDF Cheat Sheet
tags: []
id: '169'
categories:
  - - Cheat sheet
date: 2012-08-06 15:55:12
---

1.  ### BDF上传 调用
    
`var upload = new bdf.UploadFile(); var upload = new bdf.UploadFile(); upload.show({ maxFileCount: 3, caption: "我的上传", onSuccess: function (result) { for (var i = 0; i < result.length; i++) { var info = result[i]; console.log(info.id); console.log(info.fileName); } } });`3.  ### Datagrid&Autoform个性化
    
在系统属性表中bdf.datagrid.profile.support,bdf.autoform.profile.support值为'Y'. view中需引入bdfComponentProfile包． `//DataGrid个性化 var componentProfile = new bdf.ComponentProfile.DataGrid(); componentProfile.saveGridSet(view,"dataGridId"); //AutoForm个性化显示 var componentProfile = new bdf.ComponentProfile.AutoForm(); componentProfile.saveAutoFormSet(view,"autoFormId",{hideMode:"display"});`5.  ### 枚举类型的引用
    
`//JS动态创建Dropdown var enumdropdownPosition = new bdf.EnumDropDown(this, "userPositionDropDown"); enumdropdownPosition.loadEnum("employee.position"); //JS动态创建Dropdown 过滤或者排序 var enumdropdownPosition = new bdf.EnumDropDown(this, "userPositionDropDown"); enumdropdownPosition.loadEnum("employee.position", function (entityList) { entityList.each(function (entity) { if (entity.get("name") == "董事长") { entity.remove(); } }) }); //EL表达式引用枚举信息 ${dorado.getDataProvider("bdf.commonPR#findEnums").getResult("枚举key值")}`7.  ## 固定URL访问权限设定
    
A IS\_AUTHENTICATED\_ANONYMOUSLY F IS\_AUTHENTICATED\_FULLY R IS\_AUTHENTICATED\_REMEMBERED `/test/*.* A /images/*.jpg F bdf.security.specialUrlAccessDefinition=/login.jsp,A;`9.  ### 替换默认登录界面
    
bdf.security.loginFormUrl username\_ password\_ kaptcha\_ img src="bdf.loginController.generateCaptcha.c"