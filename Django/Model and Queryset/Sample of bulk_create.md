# Sample of bulk_create

#optimization

![](https://img.shields.io/static/v1.svg?logo=Python&label=Python&message=3.7&color=informational) 

> Define in a SuperSerializer  
The sub serializer inherit create def and get the current model by `self.Meta.model`  
The Model of manytomany field can be retrieve by `instance.<manytomany filed name>.model`

**Create multiple objects and all them to a manytomany fields in one line**

1. This function included SQL in `for`, which easy to crash when list is too long
2. Create multiple entries at a time is much more efficient with [`bulk_create.()`](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#bulk-create)

```python

class CommonInfoSerializer(serializers.ModelSerializer):
    class Meta:
        abstract = True

    def create(self, validated_data):
        tags_data = validated_data.pop('tags')
    
        info_model = self.Meta.model
        instance = info_model.objects.create(**validated_data)
        tag_model = instance.tags.model
    
        for tag_data in tags_data:
            tag_name = tag_data.get('name').lower()
            try:
                tag = tag_model.objects.get(name=tag_name)
            except tag_model.DoesNotExist:
                tag = tag_model.objects.create(**tag_data)
                # tag = Tag.objects.create(name=tag_data["name"])
                tag.save()
            instance.tags.add(tag)
    
        return instance
```

**After refactoring:**

```python
class CommonInfoSerializer(serializers.ModelSerializer):
    class Meta:
        abstract = True
        
    def create(self, validated_data):
        tags_data = validated_data.pop('tags')
        new_tags = [tag['name'].lower() for tag in tags_data]
    
        info_model = self.Meta.model
        instance = info_model.objects.create(**validated_data)
    
        tag_model = instance.tags.model
        exist_objs = tag_model.objects.filter(name__in=new_tags)                # QuerySet
        exist_tags = list(exist_objs.values_list('name', flat=True))            # List(String)
        to_create_tags = [tag for tag in new_tags if tag not in exist_tags]     # List(String)
        to_create_objs = [tag_model(name='%s' % i) for i in to_create_tags]     # List(Object)
        created_objs = tag_model.objects.bulk_create(to_create_objs)            # List(Object)
        to_add_objs = list(exist_objs) + created_objs                           # List(Object)
        instance.tags.add(*to_add_objs)
        return instance
```

**A roughly test with 8 tags:**

Before and after:
165ms vs 461ms