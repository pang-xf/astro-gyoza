---
title: 记一次解决图片过大造成的列表滑动卡顿问题
date: 2024-06-07
summary: 记一次解决图片过大造成的列表滑动卡顿问题。
category: 技术分享
tags: [Develop, Front]
---

### 前言：

开发（me）：要不要做分页查询，数据量过大会有性能问题；

后端：不用，没多少数据

开发：好

测试：滑动卡顿，修复一下

开发（me）：…

> 最近做了一个订票页面，已经在验收收尾阶段了，才被测出下滑卡顿，排查原因发现图片很多都是十几兆的，性能分析发现滚动起来浏览器render时间达到5秒，好吧，那就开始优化；

原始效果：

[https://img.hehe.dev/api/file/5a798c2b9e788507b0c1f.mp4](https://img.hehe.dev/api/file/5a798c2b9e788507b0c1f.mp4)

### 优化一：图片懒加载

既然是图片大导致加载渲染时间长，那先撸一个图片懒加载试试效果；

```jsx
import React, { useState, useEffect, useCallback } from 'react';
interface IProxyImage {
  src: string; // 真实图片
  defaultImgSrc?: string; // 占位图
}

export default function ProxyImage(props: IProxyImage) {
  const { src, defaultImgSrc = '', ...restProps } = props;
  const [imgLoaded, setImgLoaded] = useState(false);
  const [loadedImg, setLoadedImg] = useState(null as any);
  const loadImg = useCallback((url: string) => {
    const img = new Image();
    img.src = url;
    img.onload = () => {
      setLoadedImg(img)
    };
  }, []);

  // 真实图片加载完成，显示真实图片
  useEffect(() => {
    loadImg(src);
  }, [src]);
  return (
    <div>
      {loadedImg && <img src={loadedImg} {...restProps} />}
    </div>
  );
}

```

效果预览：

[https://img.hehe.dev/api/file/fb4562b26f186bea808cf.mp4](https://img.hehe.dev/api/file/fb4562b26f186bea808cf.mp4)

好吧，虽然有懒加载效果，但是效果并不明显，上强度！

### 优化二：图片懒加载并压缩

LazyLoadImg→Comp

```jsx
import React, { useState, useEffect, useCallback } from 'react';
interface IProxyImage {
  src: string; // 真实图片
  defaultImgSrc?: string; // 占位图
  width?: number;
  height?: number;
  [key: string]: any;
}

export default function ProxyImage(props: IProxyImage) {
  const { src, defaultImgSrc = '', width, height, ...restProps } = props;
  const [imgLoaded, setImgLoaded] = useState(false);
  const [compressedImg, setCompressedImg] = useState(null as any);
  const loadImg = useCallback((url: string) => {
    const img = new Image();
    img.src = url;
    img.crossOrigin = 'Anonymous';
    img.onload = () => {
      const canvas: any = document.getElementById('canvas');
      const ctx = canvas.getContext('2d');
      const _width = width || img.width;
      const _height = height || img.height;
      canvas.width = _width;
      canvas.height = _height;
      ctx.drawImage(img, 0, 0, _width, _height);
      const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.8);
      setCompressedImg(compressedDataUrl);
      setImgLoaded(true);
      fetch(compressedDataUrl)
        .then(res => res.blob())
        .then(blob => {
          console.log('Compressed Image Size:', blob.size, 'bytes');
        });
    };
  }, []);

  // 真实图片加载完成，显示真实图片
  useEffect(() => {
    loadImg(src);
  }, [src]);
  return (
    <div>
      {imgLoaded && <img src={compressedImg} {...restProps} />}
      <img id="originalImage" style={{ display: 'none' }} />
      <canvas id="canvas" style={{ display: 'none' }}></canvas>
    </div>
  );
}

```

How to Use:

```jsx
<div className="banner" onClick={() => setPreviewUrl(hotelImg)}>
  <LazyLoadImg
    weId={`${props.weId || ''}_h1fg7r`}
    src={hotelImg}
    alt={hotelName}
    width={360}
    height={200}
  />
</div>
```

效果预览：

[https://img.hehe.dev/api/file/19533c029bff694e66bba.mp4](https://img.hehe.dev/api/file/19533c029bff694e66bba.mp4)

### 优化三：增加图片点击放大

上面解决了图片过大造成的卡顿问题，但是还不完美，我们需要用户点击的时候预览原始搞清大图，这里我用到了react-image-lightbox来帮实现：

```jsx
...
import Lightbox from 'react-image-lightbox';
import 'react-image-lightbox/style.css';
...

<Lightbox
   weId={`${props.weId || ''}_e6rs14`}
   mainSrc={previewUrl}
   onCloseRequest={() => setPreviewUrl('')}
/>
```

效果预览：

[https://img.hehe.dev/api/file/59010c35650044f661e32.mp4](https://img.hehe.dev/api/file/59010c35650044f661e32.mp4)
