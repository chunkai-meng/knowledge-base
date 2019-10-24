# How to use Django model inheritance with signals?

写一个公共的signal，根据sender实例名的不同采取不同的动作


```
# sender 是抽象模型类
@receiver(m2m_changed, sender=CommonInfo.users_like.through)
def info_like_changed(sender, **kwargs):
    action = kwargs.pop('action', None)
    pk_set = kwargs.pop('pk_set', None)
    instance = kwargs.pop('instance', None)

    if instance.__class__.__name__ == 'info' and (action == 'post_add' or 'post_remove'):
        instance.num_like = instance.users_like.count()
        instance.num_favorite = instance.my_favorite.count()
        instance.save()

```