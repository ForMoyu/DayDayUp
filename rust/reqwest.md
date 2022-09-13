# reqwest库学习

- [reqwest库学习](#reqwest库学习)
  - [rust学习感受](#rust学习感受)
  - [serde坑](#serde坑)
  - [reqwest请求bilibili的api](#reqwest请求bilibili的api)

## rust学习感受
最近一直再看[rust圣经](https://course.rs/about-book.html), 已经看得受不了了, 想赶紧实操, 我感觉在实操中学习更有感触

## serde坑
[在新版本的serde中使用derive须要开启对应的features](http://www.javashuo.com/article/p-xvkhztqr-vg.html)

```
serde = { version = "1.0", features = ["derive"] }
```

## reqwest请求bilibili的api

