# Django Restfull API

## Serializer
```py
# 显示外键的名字而不是外键ID值
class CommonInfoSerializer(serializers.ModelSerializer):
    category = serializers.StringRelatedField(many=False)
```

**序列化器工作原理**
```
graph TD
A(模型实例 'object') --> |串行化模型实例 Serializer 对象| B
B[创建串行化器实例 'serializer'] --> |用JSON渲染器渲染 JSONRenderer| C
C[字节流: type<bytes>] --> D
D(反序列化) --> |JSONParser | E
E[原生字典类型 data <class 'dict'>] --> |Serializer data=data| F
F[串行化器实例 'serializer'] --> G
G{serializer.is_valid} --> |True| H
H(serializer.validated_data: <class 'collections.OrderedDict'>) --> |serializer.save| A
```

- 现在当我们反序列化数据的时候，基于验证过的数据我们可以调用.save()方法返回一个对象实例。
    ```py
    comment = serializer.save()
    ```
- 调用.save()方法将创建新实例或者更新现有实例，具体取决于实例化序列化器类的时候是否传递了现有实例

    ```py
    # .save() will create a new instance.
    serializer = CommentSerializer(data=data)
    
    # .save() will update the existing `comment` instance.
    serializer = CommentSerializer(comment, data=data)
    ```

![image](https://i.imgur.com/TLu9uN5.png)

> Bytes 对象只负责以二进制字节序列的形式记录所需记录的对象，至于该对象到底表示什么（比如到底是什么字符）则由相应的编码格式解码所决定。我们可以通过调用 bytes() 类（没错，它是类，不是函数）生成 bytes 实例，其值形式为 b'xxxxx'，其中 'xxxxx' 为一至多个转义的十六进制字符串（单个 x 的形式为：\xHH，其中 \x 为小写的十六进制转义字符，HH 为二位十六进制数）组成的序列，每个十六进制数代表一个字节（八位二进制数，取值范围 0-255），对于同一个字符串如果采用不同的编码方式生成 bytes 对象，就会形成不同的值：


**ViewSet create() 工作原理**
```
graph TD
O(ViewSet) --> A
A[ViewSet.create] --> B
B[perform_create] --> C
C[serializer.save] --> D
D[serializer.create/update]

```


**重点：`create()` 做了很多脏活累活**

- 接受客户端发送过来的数据
- 装入序列化器
- 校验数据合法性
- 执行 perform_create()

```
class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': str(data[api_settings.URL_FIELD_NAME])}
        except (TypeError, KeyError):
            return {}
```



```
json_string_for_new_game = '{"name":"Tomb Raider Extreme Edition","release_date":"2016-05-18T03:02:00.776594Z","game_category":"3D RPG","played":false}'
json_bytes_for_new_game = bytes(json_string_for_new_game, encoding="UTF-8")
stream_for_new_game = BytesIO(json_bytes_for_new_game)
parser = JSONParser()
parsed_new_game = parser.parse(stream_for_new_game)
print(parsed_new_game)
```

## Serializer Fields
Field 字段有很多中，参见[官方介绍](https://www.django-rest-framework.org/api-guide/fields/)


市场需要添加模型没有的字段，或者需要关联查询，需要计算得出结果的字段。
数据库为了满足三个范式，有些字段不应该直接存在数据库里，所以模型里也就没有

### 普通类型字段
#### ChardField
```
    # get_<model field>_display 是模型中的方法，可以用于带有 choice 参数的模型字段，例如，性别，某事物类型等
    # Django 自动为这类字段添加了 get_FOO_display() 方法，返回可读值
    msg_type = serializers.CharField(source='get_msg_type_display')
```

#### IntegerField
```
    模型中
    info_count = serializers.IntegerField(read_only=True)
```

#### ChoiceDisplayField
选项字段 GET 自动给出的是数据库存储的值,
如果希望 GET 的时候返回可读值 readable value，POST 的时候又是数值，可以用 ChoiceDisplayField，仅修改输出不影响输入

```
# in serializer
status = ChoiceDisplayField(
        choices=CommonInfo.STATUS_CHOICES, default=CommonInfo.PUBLISHED)
```
```
# in model
class CommonInfo(models.Model)
    INACTIVE = 0
    PUBLISHED = 1
    PENDING = -1
    DRAFT = -2
    REPORTED = -3
    DELETED = -4
    STATUS_CHOICES = (
        (INACTIVE, 'INACTIVE'),
        (PUBLISHED, 'PUBLISHED'),
        (PENDING, 'PENDING'),
        (DRAFT, 'DRAFT'),
        (REPORTED, 'REPORTED'),
        (DELETED, 'DELETED'),
    )
    status = models.SmallIntegerField(
            choices=STATUS_CHOICES, default=PUBLISHED)
```

#### ReadOnlyField
```
    # 个人认为跟普通类型字段相似，只是类型跟随数据源类型，顺便把只读属性加上了
    info_owner = serializers.ReadOnlyField(source='info.owner.id')
```

### 关联字段

#### StringRelatedField
```
    # source 表明了数据来源（模型中的字段或方法），如果序列化器的字段名跟数据源同名可以不写 source。
    # StringRelated 表明数据源是外键，这里要取外键的字符串值，我估计是调用 __str__(self) 方法吧。
    category_name = serializers.StringRelatedField(source='category')
```
#### SlugRelatedField
```
    # StringRelatedField 只是读取到外键对象的描述名，
    # SlugRelatedField 可以读到外键对象内部的字段，
    # 在模型中有个叫 tags 的 manytomany 字段，关联对象是 Tag，要输出每个 tag 对象的 name 自字段内容
    tags = serializers.SlugRelatedField(
        many=True, read_only=True, slug_field='name')
```
```
class Post(models.Model):
    ....
    tags = models.ManyToManyField('Tag', blank=True)
    
class Tag(models.Model):
    name = models.CharField(max_length=64)
    created_at = models.DateTimeField(auto_now_add=True)
```

- 只写字段
```py
class RolePermissionSerializer(serializers.ModelSerializer):
    company_name = serializers.CharField(source='company.name', read_only=True)
    group_name = serializers.CharField(source='group.name', read_only=True)

    class Meta:
        model = Role
        fields = [
            'id',
            'role_type',
            'company',
            'company_name',
            'group',
            'group_name',
            'users',
        ]
        extra_kwargs = {
            'company': {'write_only': True},
            'group': {'write_only': True},
        }
```

response:
```
"count": 9,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": 1,
            "role_type": "owner",
            "company_name": "Singh",
            "group_name": "boonrod-Singh",
            "users": []
        },
```

### 复杂数据类型字段
#### DictField
```
    # 需要说明一下，在模型中没有 msg_body 字段，所以要写
    msg_body = serializers.DictField()
```

```
# 这是模型方法就要返回一个字典
def msg_body(self):
    ....
    return {'text': text_content, 'html': html_content}
```

#### ListField
跟 DictField 同理可知



自定义字段中 source 的作用，知名数据的来源：
- 模型中的字段
- 模型中的自定义方法
- 模型中的 Django 默认方法
- 序列化器中的方法
- View 或者 ViewSet 中 QuerySet.annotation
```
    tags = queryset.annotate(info_count=Count('info'))
```
对应的 serializer 中
```
    info_count = serializers.IntegerField(read_only=True)
```

既然 source 的来源有很多种，那么我们用来求值的方法放在哪里就可以有很多选择。

### 选择何处放置处理逻辑原则就是：
- 重 model，尽量把处理逻辑放在 model
- 如果涉及到当前用户的，可以放到 serializer 里
> A context is passed to the serializer in REST framework, which contains the request by default. So you can just use self.context['request'].user inside your serializer.
```
class ActivitySerializer(serializers.ModelSerializer):

    # Create a custom method field
    current_user = serializers.SerializerMethodField('_user')

    # Use this method for the custom field
    def _user(self, obj):
        request = getattr(self.context, 'request', None)
        if request:
            return request.user

    class Meta:
        model = Activity
        # Add our custom method to the fields of the serializer
        fields = ('id','current_user')
```

- 在 View 里 QuerySet 可以带 annotate ，这样就可以直接采用 Django 自带的很多方法（查询 Django 文档即可），也可以知道当前用户：request.user


```py
from django.contrib.auth.models import User
from django.utils.timezone import now
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    days_since_joined = serializers.SerializerMethodField()

    class Meta:
        model = User

    def get_days_since_joined(self, obj):
        return (now() - obj.date_joined).days
```

## Browing Django RESTful API
- pip install markdown # Markdown为可视化 API 提供了支持
- pip install django-filter
- Edit
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAdminUser',),
    'PAGINATE_BY': 10
}
```

```
# <porject_folder>/urls.py
urlpatterns = [
    ....
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]
```
- Login API  
[localhost:8000/api-auth/login](localhost:8000/api-auth/login)

- 主页添加登陆连接
```html
<li class="nav-item"><a class="nav-link" href="{% url 'rest_framework:login' %}">API-AUTH</a> </li>
```



### The following packages are optional:
```
coreapi (1.32.0+) - Schema generation support.
Markdown (2.1.0+) - Markdown support for the browsable API.
django-filter (1.0.1+) - Filtering support.
django-crispy-forms - Improved HTML display for filtering.
django-guardian (1.1.1+) - Object level permissions support.
```

## Views & ViewSets 的写法
### Serializer
- serializers.Serializer  
类似于 Form
- serializers.ModelSerializer  
类似于 ModelForm

### API Views
- @api-view decorator
- APIView Class-based views  
定义 def get, put, delete
```py
class SnippetDetail(APIView):
    """
    Retrieve, update or delete a snippet instance.
    """
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

- Using mixins
还需要自己写get post

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import mixins
from rest_framework import generics

class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

- Using generic class-based views

```
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

### 在Serializer中加入关联项

```
class PostSerializer(serializers.ModelSerializer):
    images = serializers.PrimaryKeyRelatedField(many=True, queryset=Image.objects.all())
    """
    这里source 指明这个新加的field： owner 用什么样的属性：copy owner对象的username的属性
    等同于CharField(read_only=True)
    """
    owner = serializers.ReadOnlyField(source='owner.username')
    
    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
        
    class Meta:
        model = Post
        fields = '__all__'
```

- images 是Image Model中定义的post外键的关联名。也就给post 用来引用其所有关联images时使用的名称

- 这样，PostSerializer 对象中就多了一个field，是一个图片数组。

## 将Post关联到对应到User（owner）

可以打印serializer

```python
from snippets.serializers import SnippetSerializer
serializer = SnippetSerializer()
print(repr(serializer))
# SnippetSerializer():
#    id = IntegerField(label='ID', read_only=True)
#    title = CharField(allow_blank=True, max_length=100, required=False)
#    code = CharField(style={'base_template': 'textarea.html'})
#    linenos = BooleanField(required=False)
#    language = ChoiceField(choices=[('Clipper', 'FoxPro'), ('Cucumber', 'Gherkin'), ('RobotFramework', 'RobotFramework'), ('abap', 'ABAP'), ('ada', 'Ada')...
#    style = ChoiceField(choices=[('autumn', 'autumn'), ('borland', 'borland'), ('bw', 'bw'), ('colorful', 'colorful')...
```

## Hyperlink API

用Hyperlinking 来处理entities之间的关系，也就是关联查询

Serializer类继承：HyperlinkedModelSerializer，而不是 ModelSerializer

有三点改变：
- It does not include the id field by default.
- It includes a url field, using HyperlinkedIdentityField.
- Relationships use HyperlinkedRelatedField, instead of PrimaryKeyRelatedField.

## T6 ViewSets & Routers

- REST framework 提供处理ViewSets的抽象概念，让开发者聚焦在建模API的状态和交互，框架自动完成URL的构架
- 类似View 类，但提高read， update操作而不是get put等处理方法
- 一个ViewSet只在最后实例化一组views等时候才绑定到一组处理方法上，通常用router类处理复杂等URL 配置

- 只读ViewSet

```
from rest_framework import viewsets

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    """
    This viewset automatically provides `list` and `detail` actions.
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
```

- 绑定request method 到 class method

```
from snippets.views import SnippetViewSet, UserViewSet, api_root
from rest_framework import renderers

snippet_list = SnippetViewSet.as_view({
    'get': 'list',
    'post': 'create'
})
snippet_detail = SnippetViewSet.as_view({
    'get': 'retrieve',
    'put': 'update',
    'patch': 'partial_update',
    'delete': 'destroy'
})
snippet_highlight = SnippetViewSet.as_view({
    'get': 'highlight'
}, renderer_classes=[renderers.StaticHTMLRenderer])
user_list = UserViewSet.as_view({
    'get': 'list'
})
user_detail = UserViewSet.as_view({
    'get': 'retrieve'
})
```

```
graph TD
浏览器请求发到router --> router找到对应ViewSets例如SnippetsViewSet
router找到对应ViewSets例如SnippetsViewSet --> 根据请求方法的不同找到对应等Views例如/snippet/对应'list','create'
根据请求方法的不同找到对应等Views例如/snippet/对应'list','create' --> View类根据请求方法不同调用不同调用不同的类方法'get':'list'
```

## Tutorial 7: Schemas & client libraries
Schema 应该翻译成什么呢？翻译成概要，还要再翻译什么是概要，等于没翻
Schema是机器能读懂的文档，它描述了：
- API 的可用端点/终点
- URLS
- 以及它所支持的操作
- 它可以自动生成文档
- 也可用来驱动可用与 API 交互的动态客户端库
要写这个schema 就不得不提Core API

### Core API

`Core API` 是用来表示 Web API 的，与格式无关的 DOM 文档对象模型


### Custom Site UserAdmin
**extend the default UserAdmin**

https://simpleisbetterthancomplex.com/tutorial/2016/11/23/how-to-add-user-profile-to-django-admin.html

```
class CustomUserAdmin(UserAdmin):
    UserAdmin.list_display = ('email', 'first_name', 'last_name', 'is_active', 'date_joined', 'is_staff')
    
admin.site.unregister(User)
admin.site.register(User, UserAdmin)
```

## Test API in PyCharm

```
# create a API.rest file input:
https://now.httpbin.org/

###

GET 0.0.0.0:8000/games/games/

###

GET localhost:8000/games/games/

###
```