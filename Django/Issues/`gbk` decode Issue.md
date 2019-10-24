# `gbk` decode Issue

[原文链接](https://blog.csdn.net/yixiaotian1993/article/details/89190213)

> 💡 `'gbk' codec can't decode byte 0xa6 in position 9737: illegal multibyte sequence`

相信用django的同学，不管是新手还是老手，对这行报错再熟悉不过了吧。
刚接触的时候我看到这个也很懵逼，到处查原因，stackoverflow什么的也翻烂了。
后来慢慢用的多了，就明白了，小白们听好了，这个不是什么奇怪的错误，就是你语法写错了，具体哪里错了，你要把terminal往上翻就可以看到了。

具体一点，假如你用的是pycharm的话，点击terminal，然后Ctrl+K清空，然后再复现一下你的错误，然后把terminal里的报错信息翻到最上面，从上面开始往下一行一行看，就能找到你的原始错误点了。
报这个错的原因应该是debug.py文件中的编码问题，具体解决方式：
打开django/views下的debug.py文件，**331**行：
把

```python
with Path(CURRENT_DIR, 'templates', 'technical_500.html').open() as fh
# 改成：
with Path(CURRENT_DIR, 'templates', 'technical_500.html').open(encoding="utf-8") as fh
```

应该就可以了
