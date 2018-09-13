---
title: JS控制CSS3，添加浏览器兼容前缀
date: 2018-05-13 16:12:53
tags: JS CSS3
---
不同的浏览器对于有些css3属性名定义的时候，会加上一些前缀，比如`transform`,
```
div
{
    transform: rotate(30deg);
    -ms-transform: rotate(30deg);        /* IE 9 */
    -webkit-transform: rotate(30deg);    /* Safari and Chrome */
    -o-transform: rotate(30deg);        /* Opera */
    -moz-transform: rotate(30deg);        /* Firefox */
}
```

有时候我们通过js控制css3属性。


### 重点：
```
var sty = document.createElement("div").style;
```
通过chrome控制台打印出来的是这样的
![document.createElement("div").style](/images/1/style.png)

### 遍历样式表
现在前缀主要有这四种：webkit,moz,o,ms   
通过遍历来匹配对应的前缀并返回
```
/**
 * 这个函数用来判断浏览器前缀
 * 返回standard表示不需要前缀
 * @param {string} prop 
 */
function vendor(prop) {
    // 处理有中划线(-)分隔开的属性，事实上是以驼峰命名法
    let afterProp = transformCamels(prop);
    let transformNames = {
        webkit: `webkit${afterProp}`,
        moz: `moz${afterProp}`,
        ms: `ms${afterProp}`,
        o: `o${afterProp}`,
        standard: `${prop}`
    }
    for(var key in transformNames) {
        if(elementStyle[transformNames[key]] !== undefined) {
            return key;
        }  
    }
    return false
}

function transformCamels(prop){
    let camels = prop.split('-');
    camels.forEach((item, index) => {
        camels[index] = item.charAt(0).toUpperCase() + item.substr(1);
    })
    let afterProp = camels.join('');
    return afterProp;
}
```

### 使用
封装成函数就可以使用了
```
function prefixStyle(style) {
    let prefix = vendor(style);
    if(prefix === false) {
        return false;
    }
    if(prefix === 'standard') {
        return style;
    }
    return prefix + style.charAt(0).toUpperCase() + style.substr(1);
}
```

### 测试
![测试结果](/images/1/test.png)

### 完整代码

[完整代码](./js-add-css3-prefixer.js)