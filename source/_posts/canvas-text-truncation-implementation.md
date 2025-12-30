---
title: Canvas文本截断的实现与优化
date: 2025-06-03 12:00:00
tags:
  - Canvas
  - JavaScript
  - 算法
  - 前端
categories:
  - 技术分享
---

{% asset_img header-image.jpg 封面图 %}

### 一、前言

由于本人在公司内部开发的 BI 产品使用的是 Visactor 团队提供的 [VTable](https://www.visactor.io/vtable) / [VChart](https://www.visactor.io/vtable) 组件，在机缘巧合下看到内部团队正在招人，于是果断投出了简历。面试过程中压力也非常大，结果也和我预想中的一样，被面试官花式拿下。

在面试的最后，面试官提出了一道手写题，内容是"实现一个在 Canvas 中文字截断的函数，最长宽度是 40px，文本超出时展示省略号，函数输出截断后的结果"。

<!-- more -->

### 二、设想

在听到这个题目时，心里想的是这题并不是很难，只需判断文本宽度是否超出最大宽度，在超出宽度时截取字符串，并加上省略号的后缀即可，于是便写下这份答案。

```javascript
function measureText(txt) {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  const textMeasure = ctx.measureText(txt);
  return textMeasure.width;
}

function textEllpsis(txt, maxWidth) {
  const width = measureText(txt);
  if (width < maxWidth) return txt;
  const text = [];
  txt.reduce((pre, cur) => {
    const curWidth = pre + measureText(cur);
    if (pre > maxWidth) {
      return;
    }
    text.push(cur);
    return pre + measureText(cur);
  }, 0);
  return text.join("") + "...";
}
```

乍一看，貌似没什么毛病。。。

但当我写完这个功能时，面试官提出了几个核心的错误点：

- 后缀的宽度需要算到最终宽度里面
- 边界处理
- 计算具体是在哪个字符截止的算法

这就很尴尬了，没想到错的地方这么多。

痛定思痛，于是翻阅了 Visactor/VRender 的源码之后，才了解到如何去优雅的截断文本。

### 三、核心要点

在实现整个功能之前，先来针对这几个问题分析一下：

#### 后缀宽度

前面提到过，要实现超出最大宽度截断文本的效果，必须要考虑到 ellpsis 的宽度，也就是说我们文本的长度只能是 **最大宽度 - ellpsis 的宽度** 。

`textWidth = maxWidth - ellpsisWidth`

{% asset_img suffix-width-diagram.png 后缀宽度示意图 %}

如图所示，假如最大宽度为 100px，ellpsis 宽度为 30px，那我们文本占的长度只有 70px。

#### 边界处理

关于边界条件的处理，主要是以下几点：

1. ellpsis 为空文本：不调整宽度，直接进入文本裁剪
2. 待测量文本为空字符串：直接返回
3. 待测量文本宽度 < 最大宽度：直接返回原文本
4. ellpsis 宽度 > 最大宽度：返回空文本

#### 如何找到截止字符

截止字符的查找，本质上还是去字符串中找到符合条件的字符。如果通过遍历整个文本去查找最后一个字符的话，在文本量大的情况下，会导致耗时过长。

那么怎么才能优化时间复杂度呢？

可以采用最简单也最常用的一个算法：二分查找。

以 `@Visactor/VRender` 为例 ，宽度为 400，如何通过二分查找去找到截断后的最后一个字符呢？这里有几种情况，我们来分别走一遍流程。

1. 最大宽度为 200

{% asset_img binary-search-200.png 最大宽度为200的二分查找 %}

2. 最大宽度为 150

{% asset_img binary-search-150.png 最大宽度为150的二分查找 %}

3. 最大宽度为 250

{% asset_img binary-search-250.png 最大宽度为250的二分查找 %}

可以看出二分查找的核心其实就是找到能够使截断文本长度介于最靠近最大宽度的索引，我们来看下具体是怎么做到的：

{% asset_img binary-search-flow.png 二分查找核心流程 %}

通过二分查找的方式，我们可以快速定位到文本截断的位置，时间复杂度从 O(n) 优化到 O(log n)。

### 四、功能实现

功能分析完了，我们来看下代码是如何实现的，首先我们定义一个类 `TextMeasure` ，专门用于处理文本。

```javascript
class TextMeasure {
  constructor({ canvas }) {
    this.context = canvas.getContext("2d");
  }
}
```

#### 文本宽度的测量

文本宽度的测量较为简单，可以采用原生 canvas 自带的测量方法。

```javascript
class TextMeasure {
  // ...
  // 定义一个函数，用于测量文本的宽度
  measureTextWidth(text, options) {
    this.context.font = `${options.fontSize}px ${options.fontFamily}`;
    const { width } = this.context.measureText(text);
    return width;
  }
}
```

#### 二分查找超出的文字边界

文字边界的查找，就是一个简单的递归

```javascript
class TextMeasure {
  // ...
  _clipTextEnd(text, options, width, leftIdx, rightIdx) {
    if (leftIdx === rightIdx) {
      const subText = text.substring(0, rightIdx + 1);
      return { str: subText, width: this.measureTextWidth(subText, options) };
    }
    const middleIdx = Math.floor((leftIdx + rightIdx) / 2);
    const subText = text.substring(0, middleIdx + 1);
    const strWidth = this.measureTextWidth(subText, options);
    let length;
    if (strWidth > width) {
      // 如果仅有一个字，还是大于宽度的话，没有必要返回了
      if (subText.length <= 1) {
        return { str: "", width: 0 };
      }
      // 取出左侧的那个字符
      const str = text.substring(0, middleIdx);
      // 如果到左侧的字符小于或等于width，那么说明就是左侧的字符
      length = this.measureTextWidth(str, options);
      if (length <= width) {
        return { str, width: length };
      }
      // 返回leftIdx到middleIdx
      return this._clipTextEnd(text, options, width, leftIdx, middleIdx);
    } else if (strWidth < width) {
      // 如果字符串的宽度小于限制宽度

      // 如果已经到结尾了，返回text
      if (middleIdx >= text.length - 1) {
        return { str: text, width: this.measureTextWidth(text, options) };
      }
      // 先判断是不是右侧的那个字符
      const str = text.substring(0, middleIdx + 2);
      // 如果到右侧的字符大于或等于width，那么说明就是这个字符串
      length = this.measureTextWidth(str, options);
      if (length >= width) {
        return { str: subText, width: strWidth };
      }
      // 返回middleIdx到rightIdx
      return this._clipTextEnd(text, options, width, middleIdx, rightIdx);
    }
    // 如果相同，那么就找到text
    return { str: subText, width: strWidth };
  }
  //...
}
```

#### 边界处理

边界的处理不应该跟文本截断的核心处理的逻辑耦合在一起，所以这里我们对所有边界条件的判断以及前置处理进行统一收口，将其放到 `clipTextWithSuffix`中。

```javascript
class TextMeasure {
  //...
  // 文本测量入口
  clipTextWithSuffix(text, options, width, suffix) {
    // 空后缀判断，直接进入到 _clipText
    if (suffix === "") {
      return this._clipText(text, options, width, 0, text.length - 1, suffix);
    }
    // 空文本处理
    if (text.length === 0) {
      return { str: "", width: 0 };
    }
    const length = this.measureTextWidth(text, options);
    // 未超出宽度
    if (length <= width) {
      return { str: text, width: length };
    }
    const suffixWidth = this.measureTextWidth(suffix, options);
    // 处理后缀宽度超出最大宽度
    if (suffixWidth > width) {
      return { str: "", width: 0 };
    }

    // 调整剩余宽度
    width -= suffixWidth;
    const data = this._clipText(text, options, width, 0, text.length - 1, suffix);

    data.str = data.result;
    data.width += suffixWidth;
    return data;
  }

  // 剪切文本
  _clipText(text, options, width, leftIdx, rightIdx, suffix) {
    let data;
    // 为了扩展性提前将对应逻辑抽离
    data = this._clipTextEnd(text, options, width, leftIdx, rightIdx);
    suffix && (data.result = data.str + suffix);

    return data;
  }
  //...
}
```

注意看到这里还使用了一个 `_clipText` ，至于为什么要加这么一个看似多余的操作，我们后续会讨论。

### 五、扩展

这篇文章仅仅讨论了单行文本 suffix 的处理，但是在实际的开发中，用户往往不只有这一个需求。比如说省略号显示在文本的开头或是中间，该如何进行处理？以及文本宽度计算如何进行优雅降级，用于支持服务端渲染或者是不存在 Canvas 元素的情况？多行文本最大高度以及最大宽度如何进行处理等等。

我们以不同的省略号位置为例，需要改变的是 subText 和 str 的截取逻辑，整体上采用的还是二分查找。比如我们要将省略号放到最前面，我们就需要将 subText 改为 `text.substring(middleIdx - 1, text.length)`，判断文本末尾的长度，通过二分法从文本末尾往前递归，直至找到符合要求的文本。

这时候 `_clipText` 的作用就体现出来了，我们可以在该函数中对不同的位置采用不同的逻辑，对文本截断做收口操作，VRender 源码中是这么进行处理的：

```javascript
class ATextMeasure {
  // ...
  private _clipText(
    text: string,
    options: TextOptionsType,
    width: number,
    leftIdx: number,
    rightIdx: number,
    position: 'start' | 'end' | 'middle',
    suffix: string | false
  ): { str: string; width: number; result?: string } {
    let data: { str: string; width: number; result?: string };
    if (position === 'start') {
      data = this._clipTextStart(text, options, width, leftIdx, rightIdx);
      suffix && (data.result = suffix + data.str);
    } else if (position === 'middle') {
      const d = this._clipTextMiddle(text, options, width, '', '', 0, 0, 1);
      data = { str: 'none', width: d.width, result: d.left + suffix + d.right };
    } else {
      data = this._clipTextEnd(text, options, width, leftIdx, rightIdx);
      suffix && (data.result = data.str + suffix);
    }
    return data;
  }
  //...
}
```

这些复杂功能在 VRender 当中都已经支持了，感兴趣的可以去翻阅下对应的源码 [@Visactor/VRender](https://github.com/VisActor/VRender/blob/develop/packages/vrender-core/src/core/contributions/textMeasure/AtextMeasure.ts)

### 六、结语

一个看似简单的文本截断功能，在 CSS 中通过几行代码便可实现，而在 Canvas 中却需要这么多复杂的计算，这还只是图表渲染引擎中的冰山一角，可见想要实现一个图表渲染引擎是多么的复杂。
