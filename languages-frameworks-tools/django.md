# Django手册

量潮工作手册语言、框架和工具目录下的 Django 子手册。本手册把 Django 初始化、配置、模型层、多租户、测试、SDK 规范，以及路由、视图、模板、表单、后台、认证、中间件和部署这些 Django 工程里最常用的部分整理成可直接照着做的工作说明。

这本手册的重点不是讲 Django 的抽象概念，而是把一个 Django 项目从初始化、建模、写页面、接认证、跑测试、做多租户、打包 SDK 到部署上线的关键动作说清楚。

## 使用场景

- 需要快速搭一个 Django 项目，但又希望工程结构、配置和测试先立住。
- 需要把模型、表单、视图、模板、后台和认证一起形成闭环。
- 需要做多租户 SaaS 或 PaaS 风格的 Django 服务。
- 需要把 Django 项目包装成内部 SDK、API Client 或可复用库。
- 需要把测试、部署、迁移和发布证据一起维护。

## 工程边界

Django 适合做这些事：

- 典型的业务后台和管理系统。
- 需要后台管理界面的应用。
- 需要表单、认证、权限、模板和数据库协作的项目。
- 需要多租户 schema 隔离的服务。
- 需要稳定的管理命令、迁移、测试和部署流程。

不适合只靠 Django 解决的事：

- 纯前端交互很重、后端只提供轻量 API 的场景，可能更适合 FastAPI 或前后端分离。
- 只需要一次性脚本或批处理，不需要完整 Web 框架。
- 需要高吞吐、强异步的特定服务，不能只看“会不会写 Django”。

## 初始化

初始化阶段先把项目骨架和开发环境定下来。

常见动作：

- `django-admin startproject` 创建项目骨架。
- `python manage.py startapp` 创建应用。
- PyCharm 里把模型类、序列化类、表单类、测试类模板做成文件模板，减少重复工作。
- 先确认 Python 版本、依赖管理方式和目录结构，再开始写业务。

建议先确定这些文件/目录的责任：

| 对象 | 责任 |
|------|------|
| `manage.py` | 项目管理命令入口 |
| `settings/` | 配置加载和环境区分 |
| `urls.py` | 路由入口 |
| `views.py` | 请求处理逻辑 |
| `templates/` | 模板渲染 |
| `models.py` | 数据模型 |
| `tests/` | 测试 |

## 声明式配置

Django 项目可以使用 `Dynaconf` 提供的 Django 方案配置代替 Django 标准方案。这个思路适合把配置从代码里拆出来，避免把环境参数、密钥和部署细节写死在 Python 模块里。

配置时至少要管住这些项：

- `SECRET_KEY`
- `DEBUG`
- `ALLOWED_HOSTS`
- 数据库连接
- 缓存配置
- 静态文件和媒体文件路径
- 中间件列表
- 已安装应用

如果项目还要做多租户，`INSTALLED_APPS`、`SHARED_APPS` 和 `TENANT_APPS` 要保持一致。要特别注意：如果 `TENANT_APPS` 和 `INSTALLED_APPS` 不同步，`migrate` 可能无法把模型迁到租户 schema，对应数据库操作会出问题。

配置原则：

- 公共配置和环境配置分开。
- 密钥和外部地址不要长期散在代码里。
- 开发、测试、生产三套环境至少能区分。
- 迁移、测试和部署依赖的配置要能复现。

## URL 和视图

### 路由

Django 路由先把 URL 拆给合适的视图。常用写法是 `path()`、`include()` 和 `re_path()`。

```python
from django.contrib import admin
from django.urls import include, path, re_path

from . import views

urlpatterns = [
    path("", views.homepage, name="homepage"),
    path("accounts/", include("django.contrib.auth.urls")),
    path("admin/", admin.site.urls),
    re_path(r"^bio/(?P<username>\w+)/$", views.bio, name="bio"),
]
```

路由设计要注意：

- 入口页、业务页、认证页、后台页分开。
- 复杂场景可以用 `include()` 拆成应用级路由。
- 正则路由用 `re_path()`，但别把路由写成难维护的正则泥潭。
- 路由名字要稳定，方便模板反向引用。

### 视图

视图负责把请求变成响应。常见响应类型包括：

- `HttpResponse`
- `render()`
- `redirect()`
- `JsonResponse`

函数视图适合简单逻辑，类视图适合按方法拆分行为、复用通用逻辑和组织复杂页面。

```python
from django.http import HttpResponse
from django.views import View


def homepage(request):
    return HttpResponse("ok")


class MyView(View):
    def get(self, request):
        return HttpResponse("ok")
```

类视图通过 `as_view()` 暴露给路由。它适合把 GET、POST 等请求方法拆开处理，避免把所有逻辑塞进一个函数。

## 模板和表单

### 模板

模板部分的核心要求很直接：用 Django 模板引擎渲染页面，`APP_DIRS=True` 时可以在应用子目录里找模板。

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            # context processors and other options
        },
    },
]
```

模板写法要点：

- 用 `base.html` 做页面骨架。
- 用 `block` 做可替换区域。
- 需要登录、表单或消息时，配好上下文处理器。
- 静态文件和模板不要混在一起。

### 表单

表单用于收集输入、校验数据和生成可提交的页面。

```python
from django import forms


class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField()
```

表单层要关注：

- 输入校验是否在服务端完成。
- 需要模型落库时，是否改成 `ModelForm`。
- 表单模板里是否有 `{% csrf_token %}`。
- 验证失败时，错误信息是否能反馈给用户。

如果做登录页，表单模板要带 CSRF、防止回跳丢失、密码找回入口等最基本要素。

## 模型层

模型层要把 `Model`、字段、序列化类作为核心内容。

### 模型类

模型类要把主键、字段、关系和时间字段定清楚。

```python
import uuid

from django.db import models


class MyModel(models.Model):
    id = models.UUIDField(primary_key=True, editable=False, default=uuid.uuid4, verbose_name="ID")
    name = models.SlugField(max_length=128, unique=True, verbose_name="标识")
    title = models.CharField(max_length=255, verbose_name="名称")
    description = models.TextField(blank=True, verbose_name="详情")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    updated_at = models.DateTimeField(auto_now=True, verbose_name="修改时间")
```

字段的具体约定包括：

- `id` 优先使用 `UUIDField`，避免默认自增主键在分布式环境里不方便统一。
- `name` 常用 `SlugField`，作为租户内唯一标识。
- `title` 和 `description` 用于名称和详情描述。
- `created_at` 和 `updated_at` 记录创建与修改时间。
- 关联关系字段常见是 `ForeignKey`，需要时可换成 `OneToOneField` 或 `ManyToManyField`。

### 序列化类

如果项目用了 DRF，可以用 `ModelSerializer` 把模型和外部数据格式对齐。

```python
from rest_framework import serializers

from .models import MyModel


class MyModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = MyModel
        fields = "__all__"
```

### 迁移

模型改动后不要只改代码，要把迁移流程一起说清楚。

- 迁移前先确认模型导入位置正确。
- 需要时检查 `models.__init__`，避免迁移识别不到模型。
- 字段改动和数据库迁移要同步。
- 迁移失败先看字段名、导入路径和应用注册。

## 管理后台

Django 管理后台适合做内部运营、内容维护和数据查看。

启用后台要注意：

- `django.contrib.admin`
- `django.contrib.auth`
- `django.contrib.contenttypes`
- `django.contrib.messages`
- `django.contrib.sessions`
- 对应中间件和模板上下文处理器
- `admin/` 路由
- `createsuperuser`

后台常见动作：

- 注册模型。
- 配置列表页字段。
- 配置搜索、过滤和排序。
- 按业务对象定制表单和展示。

## 认证与会话

认证不要只看登录页，要看整条会话链路。

常用方式：

- `authenticate()` 校验账号密码。
- `login()` 建立会话。
- `django.contrib.auth.urls` 提供登录、登出、密码修改、密码重置等标准路径。

```python
from django.contrib.auth import authenticate, login


def my_view(request):
    username = request.POST["username"]
    password = request.POST["password"]
    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
```

认证页要关注：

- 是否接入 `request.user`。
- 是否区分匿名用户和已登录用户。
- 是否有权限控制和页面跳转。
- 是否保留密码重置和退出路径。

## 中间件

中间件是在请求和响应之间加的一层处理逻辑。它适合做跨页面、跨视图的通用能力。

中间件要遵循这些原则：

- 顺序重要。
- 每个中间件尽量只做一件事。
- 不要把业务逻辑塞进中间件。
- 注意性能和可维护性。

常见中间件方向：

- 安全
- 会话
- 认证
- 消息
- CSRF
- 点击劫持防护

自定义中间件适合做：

- 耗时统计。
- 请求标识注入。
- 简单审计。
- 统一日志上下文。

## 测试

Django 测试可以使用 `pytest` 或 `unittest`。Django 项目至少要把单元测试、数据库测试和集成测试分开看。

测试至少覆盖：

- 正常输入。
- 空输入。
- 错误输入。
- 外部依赖失败。
- 环境变量缺失。
- 回归风险。

数据库测试的一个关键点是 `test --keepdb`。它可以保留测试库，避免每次都重建；如果测试被中断，下一次可以选择自动销毁旧测试库，用 `--noinput` 避免交互提示。

```bash
python manage.py test --keepdb
python manage.py test --noinput
```

测试建议再细一点：

- Model 测试看字段、约束、迁移和默认值。
- View 测试看路由、权限、状态码和响应内容。
- Form 测试看校验、错误提示和清洗结果。
- Auth 测试看登录、登出、权限和跳转。

## 多租户

多租户隔离可以作为明确应用场景，推荐 `django-tenants` + PostgreSQL。

### 使用场景

- PaaS / SaaS 服务。
- 需要按租户隔离数据和 API 的项目。

### 隔离思路

租户数据常见三种方案：

- 数据库隔离。
- schema 隔离。
- 租户 ID 标记。

多租户方案更偏向 schema 隔离，PostgreSQL 更适合服务内多租户。

租户 API 常见两种入口：

- 子域名。
- 子路径。

### 配置要点

`INSTALLED_APPS`、`SHARED_APPS` 和 `TENANT_APPS` 要同步。

如果必须先手工维护，至少要保证：

```python
INSTALLED_APPS = list(SHARED_APPS) + [app for app in TENANT_APPS if app not in SHARED_APPS]
```

但这个做法只是备选，不是首选。更稳妥的做法是把配置管理做好，别让应用列表在多个位置漂移。

## SDK 和库设计

如果 Django 项目要对外提供稳定调用方式，可以把业务能力再包装成 SDK 或 API Client。

核心思路是：

- `settings` 模块提供初始化参数。
- `api` 模块提供默认参数和单例 `api_client`。
- 通过 `BaseAPIClient` 加多个子模块 mixin 组合成最终客户端。

这样做的好处是：

- 对外调用入口稳定。
- 模块边界清楚。
- 维护时不容易把大类写成一坨。

## 部署

部署要把环境、安全、静态资源、数据库和运行方式一起考虑。

部署前至少确认：

- `DEBUG=False`
- `ALLOWED_HOSTS` 正确
- `SECRET_KEY` 不明文进仓库
- 数据库连接可用
- 迁移已经执行
- 静态文件已收集
- 日志可追踪
- 健康检查可访问

推荐检查项：

- `python manage.py check`
- `python manage.py check --deploy`
- `python manage.py migrate`
- `python manage.py collectstatic`

部署方式通常会落在：

- WSGI
- ASGI
- 容器
- 虚拟机
- 反向代理 + 静态文件服务

部署时不要只看“页面能打开”，还要看：

- 登录是否正常。
- 表单提交是否正常。
- 管理后台是否正常。
- 测试数据库和生产数据库是否分开。
- 多租户 schema 是否按预期落表。

## 提交前检查

- Django 的范围是全栈还是专题，标题里是否写清。
- URL、视图、模板、表单、模型、后台、认证、中间件、测试和部署是否都补上。
- 初始化和配置是否明确。
- `Dynaconf` 或其他配置方式是否讲清。
- `UUIDField`、`SlugField`、`created_at`、`updated_at`、`ForeignKey` 等字段约定是否保留。
- `ModelSerializer`、`APIClient` 这类扩展点是否写明。
- 多租户是否说明 `django-tenants`、PostgreSQL、`INSTALLED_APPS` / `SHARED_APPS` / `TENANT_APPS`。
- 测试是否写到 `keepdb`。
- 部署是否写到 `collectstatic`、`migrate`、`check --deploy`。
- 入口页和目录页是否同步。
