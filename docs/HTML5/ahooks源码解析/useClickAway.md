---
title: useClickAway源码阅读笔记
group:
  title: ahooks
nav:
  title: HTML 5
  order: 1
---

# useClickAway

## 组件说明

优雅的管理目标元素外点击事件的 Hook。

## 使用方法

```typescript
import React, { useState, useRef } from 'react';
import { useClickAway } from 'ahooks';

export default () => {
  const [counter, setCounter] = useState(0);
  const ref = useRef<HTMLSpanElement>();
  useClickAway(() => {
    setCounter(s => s + 1);
  }, ref);

  return (
    <div>
      <span ref={ref}>
        <button type="button">box1</button>
      </span>
      <p>counter: {counter}</p>
    </div>
  );
};
```

## 源码

```typescript
import { useEffect, useRef } from 'react';
import { BasicTarget, getTargetElement } from '../utils/dom';

// 鼠标点击事件，click 不会监听右键
const defaultEvent = 'click';

type EventType = MouseEvent | TouchEvent;

export default function useClickAway(
  onClickAway: (event: EventType) => void,
  target: BasicTarget | BasicTarget[],
  eventName: string = defaultEvent,
) {
  // 实例变量 可以在整个hooks中全局使用
  const onClickAwayRef = useRef(onClickAway);
  onClickAwayRef.current = onClickAway;

  useEffect(() => {
    const handler = (event: any) => {
      const targets = Array.isArray(target) ? target : [target];
      if (
        targets.some(targetItem => {
          const targetElement = getTargetElement(targetItem) as HTMLElement;
          return !targetElement || targetElement?.contains(event.target);
        })
      ) {
        return;
      }
      onClickAwayRef.current(event);
    };

    document.addEventListener(eventName, handler);

    return () => {
      document.removeEventListener(eventName, handler);
    };
  }, [target, eventName]);
}
```

### 参数列表

| 参数        | 说明                            | 类型                   | 默认值  |
| ----------- | ------------------------------- | ---------------------- | ------- |
| onClickAway | 触发事件的函数                  | `(event) => void`      | -       |
| target      | DOM 节点或者 Ref 对象，支持数组 | `Target` \| `Target[]` | -       |
| eventName   | 指定需要监听的事件              | `string`               | `click` |

## 源码解析

### 参数介绍

`useClickAway`接受三个参数分别是：

1. `onClickAway` 触发事件的回调
2. `target` DOM 元素或者 ref，以外的区域才可以触发`onClickAway`
3. `eventName` 事件名称

### 源码思路

1. `document.addEventListener`监听用户传递的`eventName`事件。
   ```typescript
   document.addEventListener(eventName, handler);
   ```
2. 由于`1`中会对页面产生副作用，所以在页面挂载时会移出事件
   ```typescript
   return () => {
     document.removeEventListener(eventName, handler);
   };
   ```
3. 我们下面接着看监听的`handle`函数，由于`if`中语句比较长，下面的把判断语句摘出

   ```typescript
   const handler = (event: any) => {
     // 构建目标数组
     const targets = Array.isArray(target) ? target : [target];

     // 判断事件是否作用在target 或者 是否节点不存在
     const isE = targets.some(targetItem => {
       // 获取target dom节点
       const targetElement = getTargetElement(targetItem) as HTMLElement;
       // target节点不存在 或者 目标targetElement包含了传递进来的元素
       return !targetElement || targetElement?.contains(event.target);
     });
     if (isE) {
       // 不需要回调函数
       return;
     }
     // 这里表面点击了target以外的节点，触发函数回调
     onClickAwayRef.current(event);
   };
   ```

## 总结

`@alitajs/antd-mobile-plus`中`Popup`组件就是使用了`useClickAway`来完成其他点击其他区域关闭弹出框的。该组件主要是使用了`document.addEventListener`来监听事件，判断点击目标组件是否包含了传入组件来处理回调函数的。
