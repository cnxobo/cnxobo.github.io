---
title: 导出CSV编码选择
tags:
  - java
  - encoding
  - utf-8
  - gbk
categories: Java
date: 2025-06-16
---

Excel 2013及之前版本打开CSV都是使用系统默认编码，Excel 2016开始支持UTF8-BOM格式的CSV。
考虑到系统的兼容性，早期的系统基本都是使用GBK编码输出CSV。
但是当英文操作系统打开GBK编码格式的CSV就会显示乱码，同时GBK不支持一些偏僻字还有emoji。
```Java
// 使用字节流 outputStream 时
byte[] bom = new byte[] {(byte) 0xEF, (byte) 0xBB, (byte) 0xBF};
os.write(bom);

// 使用字符流 writer时
writer.write('\uFEFF'); // UTF-8 BOM

```