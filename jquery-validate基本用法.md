---
date: 2014-10-19 15:39
tags: 
- jQuery
- js
title: jquery-validate基本用法
---

## 简单的demo
1.引入js文件
```js
<script src="jquery-1.10.2.min.js"></script>
<script src="jquery.validate.min.js"></script>
```
2.写一个简单的form
```html
<form role="form" id="form1" class="form-horizontal form-inline">
    <div class="input-group col-md-5">
        <label for="exampleInputName" class="input-group-addon">Name</label>
        <input type="text" required="true" class="form-control" id="exampleInputName"
               name="exampleInputName"
               placeholder="Enter Name">
    </div>
    <div class="input-group col-md-5">
        <label for="exampleInputAge" class="input-group-addon">Age</label>
        <input type="text" number="true" required="true" class="form-control" id="exampleInputAge"
               name="exampleInputAge"
               placeholder="Age">
    </div>
    <div class="input-group col-md-1">
        <button type="submit" class="btn btn-default">Submit</button>
    </div>
</form>
```
3.添加此表单的验证
```js
$().ready(function () {
    //---------form验证 start---------------------
    $(form1).validate({
        //修改提示方式
        showErrors: function (errorMap, errorList) {
            layer.tips(errorList[0].message
                            , errorList[0].element
                            , { style: ['background-color:#78BA32; color:#fff', '#78BA32']}
            );
            //下面是默认提示方式
            //this.defaultShowErrors();
        }
        , onkeyup: true
        , submitHandler: function (form) {
            //如果没有通过验证是不会弹出submit的
            alert("submit");
        }
    });
    //---------form验证 end---------------------
});
```
## 其它说明
### 中文提示
将下面的js单独放到一个js中然后引入
```html
<script src="messages_zh.js"></script>
```
```js
/*
 * Translated default messages for the jQuery validation plugin.
 * Locale: ZH (Chinese, 中文, 汉语, 漢語)
 */
jQuery.extend(jQuery.validator.messages, {
        required: "必填字段",
		remote: "请修正该字段",
		email: "请输入正确格式的电子邮件",
		url: "请输入合法的网址",
		date: "请输入合法的日期",
		dateISO: "请输入合法的日期 (ISO).",
		number: "请输入合法的数字",
		digits: "只能输入整数",
		creditcard: "请输入合法的信用卡号",
		equalTo: "请再次输入相同的值",
		accept: "请输入拥有合法后缀名的字符串",
		maxlength: jQuery.validator.format("请输入一个长度最多是 {0} 的字符串"),
		minlength: jQuery.validator.format("请输入一个长度最少是 {0} 的字符串"),
		rangelength: jQuery.validator.format("请输入一个长度介于 {0} 和 {1} 之间的字符串"),
		range: jQuery.validator.format("请输入一个介于 {0} 和 {1} 之间的值"),
		max: jQuery.validator.format("请输入一个最大为 {0} 的值"),
		min: jQuery.validator.format("请输入一个最小为 {0} 的值")
});

jQuery.extend(jQuery.validator.defaults, {
    errorElement: "span"
});
```
### 自定义验证
```js
//自定义检查规则
//notnull 自定义的验证规则名称，在input中表现为notnull="true"
//value为input中输入的值，element为input的Jquery对象，返回true：表示验证通过不提示
//"不能为空"：验证不通过时的提示信息
$.validator.addMethod("notnull"
    , function (value, element) {
        return value != "";
    }
    , "不能为空"
);
```
### 注意事项
在用Jquery-validate进行验证时，如果表单项<strong>没有name属性</strong>，则Jquery-validate<strong>只验证第一个</strong>表单项。