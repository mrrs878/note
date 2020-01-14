---
title: 设计模式-OCP
date: 2019-10-17 18:12:29
tags: 设计模式 OCP
categories: 设计模式
---

## 开放封闭原则

对扩展开放，对修改封闭

假设我们是一个大型 Web 项目的维护人员，在接手这个项目时，发现它已经拥有 10 万行以上的 JavaScript 代码和数百个 JS 文件。 不久后接到了一个新的需求，即在 window.onload 函数中打印出页面中的所有节点数量

```javascript
// 普通版
window.onload = function() {
  //原有代码
  console.log(document.getElementByName("*").length);
};

// OCP版 ---- AOP(面向切面编程)
Function.prototype.after = function(fn) {
  let _self = this;
  return function() {
    let ret = _self.apply(this, arguments);
    fn && fn.apply(this, arguments);
    return ret;
  };
};
Function.prototype.before = function(fn) {
  let _self = this;
  return function() {
    fn && fn.apply(this, arguments);
    return _self.apply(this, arguments);
  };
};
window.onload = (window.onload || function() {}).after(function() {
  console.log(document.getElementByName("*").length);
});
```

## 编写符合 OCP 的代码

开放-封闭原则是一个看起来比较虚幻的原则，并没有实际的模板教导我们怎样亦步亦趋地 实现它。但还是能找到一些让程序尽量遵守开放-封闭原则的规律，明显的就是找出程序中将要发生变化的地方，然后把变化封装起来。过多的条件语句是造成程序违反开放封闭原则的一个常见原因。

### 运用 AOP

### 利用多态的思想

### 放置 hook

我们在程序有可能变化的地方放置一个 hook，hook 的返回结果决定了程序的下一步走向，这样一来原本的代码执行路径上就出现了一个分岔路口，程序未来的执行方向被预埋下多种可能性

```javascript
class Beverage {
  boilWater() {
    console.log("把水煮沸");
  }
  brew() {
    console.log("子类必须重写brew方法");
  }
  pourInCup() {
    console.log("子类必须重写pourInCup方法");
  }
  addCondiments() {
    console.log("子类必须重写addCondiments方法");
  }
  // hook
  customWantsCondiments() {
    return true;
  }
  init() {
    this.boilWater();
    this.brew();
    this.pourInCup();
    if (this.customWantsCondiments()) this.addCondiments();
  }
}

class CoffeeWithHook extends Beverage {
  brew() {
    console.log("用沸水冲泡咖啡");
  }
  pourInCup() {
    console.log("把咖啡倒进杯");
  }
  addCondiments() {
    console.log("加糖和牛奶");
  }
  // hook
  customWantsCondiments() {
    return window.confirm("请问需要饮料吗?");
  }
}

new CoffeeWithHook().init();
```

### 使用回调函数

回调函数是一种特殊的挂钩。我们可以把一部分易于变化的逻辑封装在回调函数里，然后把 回调函数当作参数传入一个稳定和封闭的函数中。当回调函数被执行的时候，程序就可以因为回 调函数的内部逻辑不同，而产生不同的结果。

### 策略模式

```javascript
let strategies = {
  ADD: function(paramA, paramB) {
    return paramA + paramB;
  },
  SUB: function(paramA, paramB) {
    return paramA - paramB;
  },
  MULTI: function(paramA, paramB) {
    return paramA * paramB;
  },
  DIV: function(paramA, paramB) {
    return paramA / paramB;
  }
};
const calc = function(paramA, paramB, type) {
  return strategies[type](paramA, paramB);
};

console.log(calc(1, 2, "ADD"));
console.log(calc(1, 2, "SUB"));
```

### 高阶函数

```javascript
const calc_ = R.curry(function(paramA, paramB, fn) {
  return fn(paramA, paramB);
});

console.log(calc_(2, 3)(R.add));
console.log(calc_(2, 3, R.add));
```

### 发布-订阅模式

### 模板方法模式

### 代理模式

### 职责链模式
