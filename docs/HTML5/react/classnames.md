---
title: classnames源码解析
group:
  path: /react
  title: react
nav:
  title: HTML 5
  order: 1
---

## 废话

react 工程中我们常常用到`className`来设置`class`关联样式。大多情况下我们会采用`classnames`来使用。那么这个包是怎么实现的呢？

## 源码地址

[git 仓库地址](https://github.com/JedWatson/classnames/blob/master/index.js)

## 使用示例

```typescript
// index.less
.a {
  background-color: #ccc;
}

.b {
  color: red;
}



// index.ts
import React, { FC } from 'react';
import classnames from 'classnames';
import styles from './index.less'

interface DemoType {}

const Demo:FC<DemoType> = props => {
  return <div className={classnames(styles.a, styles.b)}></div>;
}

export default Demo;

```

## 代码阅读注释

```javascript
(function() {
  'use strict';

  var hasOwn = {}.hasOwnProperty;

  function classNames() {
    var classes = [];

    for (var i = 0; i < arguments.length; i++) {
      var arg = arguments[i];
      if (!arg) continue;

      var argType = typeof arg;

      if (argType === 'string' || argType === 'number') {
        classes.push(arg);
      } else if (Array.isArray(arg)) {
        if (arg.length) {
          var inner = classNames.apply(null, arg);
          if (inner) {
            classes.push(inner);
          }
        }
      } else if (argType === 'object') {
        if (arg.toString === Object.prototype.toString) {
          for (var key in arg) {
            if (hasOwn.call(arg, key) && arg[key]) {
              classes.push(key);
            }
          }
        } else {
          classes.push(arg.toString());
        }
      }
    }

    return classes.join(' ');
  }

  if (typeof module !== 'undefined' && module.exports) {
    classNames.default = classNames;
    module.exports = classNames;
  } else if (
    typeof define === 'function' &&
    typeof define.amd === 'object' &&
    define.amd
  ) {
    // register as 'classnames', consistent with npm package name
    define('classnames', [], function() {
      return classNames;
    });
  } else {
    window.classNames = classNames;
  }
})();
```
