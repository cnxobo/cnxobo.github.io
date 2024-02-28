---
title: 从⽅不是方到Unicode正规化NFD, NFC, NFKD, NFKC
tags:
  - pdf
  - unicode
id: '942'
categories:
  - - Java
comments: false
date: 2022-01-13 21:00:18
---

在做PDF解析的时候，发现甲方的`方`字一直匹配不上。解析出来的`⽅`字的Unicode值`\u2F45`，而我正常打字打出的`方`字Unicode值是`\u65B9`。 之前也遇到过从PDF中复制出来的字，看着像这个字其实是另一个字，一直以为是PDF源文件的问题没有深究。这次PDF是我用Word另存为生成的，可以稳定复现问题，有机会深入研究。 "方"(\\u65B9)字在原始Word文档里可以搜的到，说明原始文件没有问题。在PDF阅读器里也能搜到，唯独我的通过pdfbox解析，只能解析出"⽅"`\u2F45`。通过[unicode-table](https://unicode-table.com/en/blocks/kangxi-radicals/) 发现`2F00-2FDF`是康熙字典部首段。说明这些字与部首之间存在某种关联，这些关联还是这些编辑器及阅读器都在遵循的。

## Unicode Normalization

这个关联就是Unicode Normalization，Unicode正规化。Unicode 还有一个特点，等价性（Unicode equivalence）。由于Unicode收录了很多字符，而这些字符功能上会和其他字符或者字符序列等价。因此，Unicode将一些码位序列定义成相等的。Unicode提供了两种等价概念：标准等价和兼容等价。

### 标准等价

保持视觉上和功能上等价。例如 '≠' (\\u2260) 标准等价于 '=' 和 '̸'(\\u003d\\u0338)的组合。区别在于一个是一个字符，一个是两个字符。

### 兼容等价

兼容等价比标准等价范围更广，也更关注纯文字的等价，并把一些语义上的不同形式归结在一起。 如果字符是标准等价，那么他们一定是兼容等价，反之就不一定了。 例如 '㊋１ａ' (\\u328b\\uff11\\uff41) 带圆圈的火，全角的数字和字母兼容等价与 '火1a' (\\u706b\\u0031\\u0061)。

### 四种组合

在此基础之后Unicode又定义了两种形式，完全合成，完全分解。由此组合出四种形式: NFC、NFD、NFKC、NFKD。 C 代表 Composition 就是组合的意思 D 代表 Decomposition 就是分解的意思。 K 代表 Compatibility 就是兼容等价模式，如果不带K就是标准等价。 分解和组合主要用于是字母符号。汉字我没找到可以分解和组合的例子。汉字的兼容等价场景就很多，除了我遇到的部首`⽅`，还有 `㊥` 与 `中`、 `㍿` 与 `株式会社`、全角与半角。。。 NFC 以标准等价组合。 NFD 以标准等价分解。 NFKC 以兼容等价组合。 NFKD 以兼容等价分解。 所以这个时候，我们只需要用兼容等价(NFKC/NFKD)的方式就可以把部首转化为文字。 Java代码:

```Java
Normalizer.normalize("⽅", Form.NFKC);
```

JavaScript代码:

```JavaScript
'⽅'.normalize('NFKC')
```

可以在浏览的控制台直接尝试如下代码验证

```JavaScript
// 字符长度 1
'≠'.length == 1;
'≠'.normalize('NFC').length == 1;

// 以标准等价分解, 字符长度 2
'≠'.normalize('NFD').length == 2;

// 以标准等价分解,  再以标准等价组合。字符长度 1
'≠'.normalize('NFD').normalize('NFC').length == 1;

// 以兼容等价组合,全角字体变半角，带圈带框的字体变标准字体。
'㊋１ａ'.normalize('NFKC')=='火1a'

// 左边是两个康熙字典的部首，右边是标准字体。
'⼩不' != '小不'
// 以兼容等价组合, 部首变标准字体。
'⼩不'.normalize('NFKC') == '小不'
```

## 产生原因的推测

Unicode Normalization 并没有提供一个把文字转化为部首的方案，理论上来讲文字也不可能无损的转化为部首。出错的地方使用的字体是苹果的苹方字体。该字体下两个`方`字长相一模一样。所以推测是字体的原因，PDF引入外部字体时为了保证各个平台下的一致性，都会做字体嵌入。有可能这两个字在苹方字体下它的内部编码是一致的，转化为PDF再还原出来时Unicode值变成了更靠前的值(部首)。 创建一个新的Word文档，写两行`甲方`。第一行用宋体，第二行用平方，另存为PDF，再解析。得到

> \\u7532\\u65b9 \\u7532\\u2f45 PDF 内部存储文字并不是以Unicode的方式，而是有独立的编码方式 ANSI、Identity-H等。然后在用户复制文字的时候通过`/ToUnicode`转换出Unicode编码。

## 参考链接

Normalization Charts https://www.unicode.org/charts/normalization/chart\_Latin.html 在线unicode转中文,中文转unicode-BeJSON.com https://www.bejson.com/convert/unicode\_chinese/ Normalizing Text (The Java™ Tutorials > Internationalization > Working with Text) https://docs.oracle.com/javase/tutorial/i18n/text/normalizerapi.html Unicode等价性 - 维基百科，自由的百科全书 https://zh.wikipedia.org/wiki/Unicode%E7%AD%89%E5%83%B9%E6%80%A7 miscellaneous/浅谈pdf乱码.md at master · yunhailuo/miscellaneous https://github.com/yunhailuo/miscellaneous/blob/master/%E6%B5%85%E8%B0%88pdf%E4%B9%B1%E7%A0%81.md