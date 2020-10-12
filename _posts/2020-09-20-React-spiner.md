---
layout: post
title: ' React 页面设置Spinner、form 表单操作、Authentication '
subtitle: 'react practice'
date: 2020--09-20
author: 'Joy'
header-img: 'img/about-bg3.jpg'
tags:
  - react
  - self-learning
  - spinner
---


# 实现Spinner 也就是页面加载中
google `css spinner`

# 实现form传递参数到后端数据库

首先实现的form表单，需要了解的知识是，在react中例如`<input/>`这种HTML参数可以通过传入react object类型获得属性。

### 已经定义了state中能够传入到form表单中的数据类型
```
state = {
  orderForm: {
    name: {
        elementType: 'input',
        elementConfig: {
            type: 'text',
            placeholder: 'Your name'
        },
        value: '',
        validation: {
            required: true
        },
        valid: false,
        touched: false// 表示用户有没有点击页面的order按钮
    },
  }
}
```
+ 其中**elementType**是表示是`<input/>`、`<textarea/>`还是`<select>`作为自定义的`<Input/>`中的Html标签。
+ **elementConfig**是一个object的数据类型，传入到自定义的`<Input/>`中
```
<input 
  {...props.elementConfig}
  value={props.value}
  onChange={props.changed}
/>
```
这个input里面的value和onChange都是通过form表单中的props传输的。
+ **value**表单监听的target.value
+ **validation**和**valid**通过function包裹的输入检查，可以设置长度大小，正则表达等设置**validation**，同时通过确认其正确性判断是不是正确的输入将**valid**设置为true。
+ **touched**如果用户已经按下确认按钮，需要将其没有输入正确的地方标红。这个touched就是判断用户有没有按下确认提交。


### 自定义的检查函数
```
checkValidity(value, rules) {
  // return boolean 确认返回validation
  let isValid = true; // 开始时设置为true
  // 每个情况都要判断是否是isValid 防止因为一个input需要判断多个的情况下始终为true 

  if (rules.required) {
      isValid = value.trim() !== '' && isValid;//表示非空
  };
  if (rules.minLength) {
      isValid = value.length >= rules.minLength && isValid;// 表示最小输入长度
  };
  if (rules.maxLength) {
      isValid = value.length <= rules.maxLength && isValid;// 表示最小输入长度
  };
  return isValid;
}
```
+ 两个传入参数**value,rules**分别表示用户写入每个item的值和前面的state中定义的validation的类型。
+ 这个函数返回的是确认用户是否输入正确，state.valid来源于此。

### 在render中实现以及监听函数
```
const formElementsArray = [];
for (let key in this.state.orderForm) {
    // key ["name", "street", "zipcode"...]
    formElementsArray.push({
      id: key,
      config: this.state.orderForm[key]// 这里的config所包含的就是name等右边的内容
    })
}
```

这段代码的作用是，给每个需要用户输入的item一个id，也就是这里的`formElementsArray.id`，这样做的好处是，一个eventHandler可以监听任意的一种用户修改。

form 表单定义可以通过以下代码来渲染于页面

``` 
{formElementsArray.map(formElement => (
    <Input
        key={formElement.id}
        elementType={formElement.config.elementType}
        elementConfig={formElement.config.elementConfig}
        value={formElement.config.value}
        invalid={!formElement.config.valid}
        shouldValidate={formElement.config.validation}// 区分是select还是需要填写的地方
        touched={formElement.config.touched}
        changed={(event) => this.inputChangeHandler(event, formElement.id)}
    />
))}
```

在用户每一次点击一行决定输入时可以通过自定义`<Input/>`中对应传入的`onChange={props.changed}`绑定一个handler。
```
inputChangeHandler = (event, inputIdentifier) => {
    // 用户每次输入到input中的 内容可以通过event.target.value即使获得
    // 所谓的二次绑定(two way binding)，就是指，onChange指向的函数 绑定event获得用户实时输入的值，同时在onChange中调用这个函数时加上需要确认的identifier
    // console.log(event.target.value);
    //event.target 属性返回哪个 DOM 元素触发了事件。
    const updatedOrderForm = {
        ...this.state.orderForm // 不是深拷贝 state会变化 
        //因为只创建了state.orderForm的clone但是没创建其中 {name:...}等的clone，其实还是都指向一个地方
        // 为了创建深拷贝，对updatedOrderForm中的内容再次拷贝
    };
    const updatedFormElement = {
        ...updatedOrderForm[inputIdentifier]
    };
    updatedFormElement.value = event.target.value;
    updatedFormElement.valid = this.checkValidity(updatedFormElement.value, updatedFormElement.validation);// 判断输入值是否为空
    updatedOrderForm[inputIdentifier] = updatedFormElement;
    updatedFormElement.touched = true;// 表示用户已经输入了某些地方
    let formIsValid = true;
    for (let ele in updatedOrderForm) {
        formIsValid = updatedOrderForm[ele].valid && formIsValid;
    }
    this.setState({ orderForm: updatedOrderForm, formIsValid: formIsValid });
}
```

# 实现Authentication 

实际上的auth传输形式，本地用户对server端发送请求，在server接收请求后给本地用户一个token，这个token可以存储于localStorage中，通过token，server可以确认用户信息。