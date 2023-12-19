---
title: 清空dom对象内元素
tags:
  - dom
  - javascript
id: '377'
categories:
  - - JavaScript
date: 2014-09-11 23:16:06
---

// 1. removeChild
while (box.firstChild) {
  box.removeChild(box.firstChild);
}

// 2. innerHTML
box.innerHTML = '';


// 3. empty
$(box).empty();

// 4. innerText
box.innerText = ''