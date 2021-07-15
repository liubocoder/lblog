---
layout: post
title:  "ElementUI表单使用"
categories: Web JavaScript
tags:   vue elementui
author: liubo
typora-root-url: ..
---

* content
{:toc}


## 前言
最近业务中一个比较复杂表单，使用的是ElementUI，记录下表单的使用和遇到的问题

## 1.基本使用
```
<el-form>
    <el-form-item>
    </el-form-item>
</el-form>
```
在el-form标签内部可以嵌套其他标签




## 2.el-form-item多子标签情况

若使用rules作为校验输入合法性，`<el-form-item></el-form-item>`中只能有一个作为有效输入的控件，比如可以在其中添加checkbox和input，checkbox仅作为input是否能输入的标记，input绑定到表单数据中，el-form-item的prop属性需要指向input绑定的相对路径。

例如：el-form的model绑定了formdata，input绑定myformdata.a.b.c，prop则需要是a.b.c，prop也可以动态计算。

```
<el-form :model="myformdata">
    <el-form-item prop="a.b.c" :rules="xx">
        <el-checkbox></<el-checkbox>
        <el-input v-model="myformdata.a.b.c"></el-input>
    </el-form-item>
</el-form>
```

## 3.rule的使用

具体[参考这里](https://www.cnblogs.com/xyyt/p/13366812.html)，

(1) 自定义rule直接定义在return函数中，this指向的不是vue，需要单独定义出来
```
data() {
    const ruleValidator = (rule, value, callback) => {
        ... this.myformdata
    }
    return {
        myformdata: {},
        myrules: [
        {
          validator: ruleValidator,
          trigger: ['blur', 'change']
        }
      ]
    }
}
```

(2) 在validator函数中必须要调用一次callback()函数，否则会导致提交的时候一直等待校验结果。

## 4.焦点定位到第一个校验不过的位置
```
this.$refs[formName].validate((valid) => {
        if (valid) {
            ...
        } else {
            setTimeout(() => document.getElementsByClassName("is-error")[0].querySelector('input').focus(), 100)
            ...
        }
    })
```

## 5.注意v-if与v-show的区别
判断不通时前者过会移除DOM，后者不会

## 6.正则表达式

正则表达式是非常复杂的，单独写几篇都记录不完，这里仅简单写下使用遇到的问题，以JavaScript为例。

(1) 定义正则对象方式
```
let re1 = \abc\
let re = \abc\i
let nre = new RegExp('abc')
let nre1 = new RegExp('abc', 'i')
```

(2) 全词匹配
需要用到两个符号^$，^匹配输入字符的开始位置，&匹配输入字符的结束位置，例如`\^(abc)$\i`，忽略大小写的abc全词匹配。

(3) 操作符优先级
操作符是有优先级的，因此不太明确的时候可以加'()'来标示一个子的匹配项，用'|'（其含义为'或'）标示选择匹配，'|'的优先级最低。


## 参考资料

[vue](https://cn.vuejs.org/v2/guide/computed.html)

[ElementUI](https://element.eleme.io/#/zh-CN/component/form)

[正则表达式教程](https://www.runoob.com/regexp/regexp-tutorial.html)

[ElementUI表单学习](https://www.cnblogs.com/xyyt/p/13366812.html)