---
layout: post
title: CSS - CSS Setup and Selectors
date: 2018-03-10 17:10:22
categories: I.T.
tags: [I.T.,CSS]
---
Reference : [Codecademy - Learn CSS - CSS Setup and Selectors](https://www.codecademy.com/learn/learn-css)
<!--more-->
Class : description
Tag : h5
```css
.description h5 {
  color: teal;
}
```
*  Classes can be reusable, while IDs can only be used once.
*  IDs are more specific than classes, and classes are more specific than tags. That means IDs will override any styles from a class, and classes will override any styles from a tag selector.
*  The `!important` flag will override any style, however it should almost never be used, as it is extremely difficult to override.
*  Multiple unrelated selectors can receive the same styles by separating the selector names with commas.
```css
h5,p{
  color: rebeccapurple !important;
  font-family: Georgia;
}
```
