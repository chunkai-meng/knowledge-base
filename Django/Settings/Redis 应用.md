# Redis 在 Django 中的应用

## 1. 记录文章阅读次数

```python
# article/views.py
# 计数

def article_detail(request, id, slug):
    article = get_object_or_404(ArticlePost, id=id, slug=slug)
    total_views = r.incr("article:{}:views".format(article.id))
    return render(request, "article/list/article_content.html", {"article": article, "total_views": total_views})
```


```python
# 计数并排序

def article_detail(request, id, slug):
    article = get_object_or_404(ArticlePost, id=id, slug=slug)
    total_views = r.incr("article:{}:views".format(article.id))
    r.zincrby('article_ranking', 1, article.id)

    # 在每次阅读文章的时候调整该文章的阅读次数并修改该文章的排名
    article_ranking = r.zrange("article_ranking", 0, -1, desc=True)[:10]    # 只挑选排名前十的文章
    article_ranking_ids = [int(id) for id in article_ranking]
    
    # 找出排名中的文章
    most_viewed = list(ArticlePost.objects.filter(id__in=article_ranking_ids))
    
    # 以lambda 函数为参数传给 list 的函数 sort()， 得到一个排序后的list
    most_viewed.sort(key=lambda x: article_ranking_ids.index(x.id))
    
    # 排名前十的作为推荐文章保存在 most_viewed 中
    return render(request, "article/list/article_content.html", {"article": article, "total_views": total_views, "most_viewed": most_viewed})
```

