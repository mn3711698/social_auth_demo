social_auth使用方式
===

TODO
---
我写这个demo之后, python-social-auth, django-social-auth的作者(一个人),
对这两个库进行了比较大的更新，pip里面甚至下掉了django-social-auth, 0.8.1这个版本，
因此让demo能够跑起来，你可能需要用我的两个fork版本.

[python-social_auth一个我的fork版本](https://github.com/duoduo369/python-social-auth)

[django-social-auth一个我的fork版本](https://github.com/duoduo369/django-social-auth)

我已经在requirements.txt中去掉了这两个关键库，请手动clone安装.

注意安装顺序，需要先装python-social-auth, 再装django-social-auth(因为里面的一些依赖我暂时没改)

    git clone https://github.com/duoduo369/python-social-auth.git
    git clone https://github.com/duoduo369/django-social-auth.git

    cd python-social-auth的目录
    pip install -e .
    cd django-social-auth的目录
    pip install -e .

由于这两个版本还在不定期更新中，因此这个demo过一段时间我会做一次更新，把详细的使用步骤在写进去。

fork版本中提供了里面原来不支持的一些三方backend，现在中国常用的几个backend基本都有了: 微信，微博，qq，人人，豆瓣，百度

Begin
---

每个网站都需要注册app，这个就不说了

这里以一个heroku的app为例: `http://llovebaimuda.herokuapp.com/`

注意social_auth的配置方式不能配置回调地址，因此**必须**在各大网站将回调地址设置为`/你的域名/complete/weibo`,
这里就是`http://llovebaimuda.herokuapp.com/complete/weibo/`,当然不同的网站略有不同。

调试注意! 本地设置hosts
---

windows `%systemroot%\system32\drivers\etc\hosts` 需要在开始菜单找到notepad.exe,右键以管理员身份运行，在打开这个文件修改

linux `sudo vi /etc/hosts

将llovebaimuda.herokuapp.com改为本地或者虚拟机的ip, 开发就靠他了。

`192.168.9.191 llovebaimuda.herokuapp.com`


各个网站配置
===

新浪
---

`http://open.weibo.com/webmaster`,下进入你的app

左侧`网站信息`基本信息可以看到域名，app key, app secret

左侧`接口管理 授权机制`找到`OAuth2.0 授权设置 授权回调页`设置为`http://llovebaimuda.herokuapp.com/complete/weibo/`

qq
---
`http://connect.qq.com/manage/index`,进入你申请开发的app

左侧头像的地方就可以看到 app id, app key

点击信息编辑，找到`回调地址`,qq回调只要填写域名即可(没有http)`llovebaimuda.herokuapp.com`

豆瓣
---
`http://developers.douban.com/apikey/`, 进入你申请开发的app

概览部分可以看到api key和secrect

豆瓣不需要填写回调地址，不过需要添加测试用户，在左侧`测试用户`部分添加用户的豆瓣id

social_auth的一些配置
===

settings
---

pipeline

    SOCIAL_AUTH_PIPELINE = (
        'social.pipeline.social_auth.social_details',
        'social.pipeline.social_auth.social_uid',
        'social.pipeline.social_auth.auth_allowed',
        'social_auth.backends.pipeline.social.social_auth_user',
        # 用户名与邮箱关联，文档说可能出现问题
        # 'social_auth.backends.pipeline.associate.associate_by_email',
        'social_auth.backends.pipeline.misc.save_status_to_session',
        'social_auth.backends.pipeline.user.create_user',
        'social_auth.backends.pipeline.social.associate_user',
        'social_auth.backends.pipeline.social.load_extra_data',
        'social_auth.backends.pipeline.user.update_user_details',
        'social_auth.backends.pipeline.misc.save_status_to_session',

    )

你想用哪几种oauth

    AUTHENTICATION_BACKENDS = (
        'social_auth.backends.contrib.douban.Douban2Backend',
        # 注意这个比较特殊,因为django-social-auth是依赖python-social-auth的
        # python-social-auth==0.1.26,已经包含的qq的backend
        # django-social-auth==0.8.1, 还没包含进来
        # 你需要在django-social-auth/social_auth/backends/contrib中添加一个文件qq.py
        # 就一行
        # from social.backends.qq import QQOAuth2 as QQBackend
        # 然后setup一下就ok
        'social_auth.backends.contrib.qq.QQBackend',
        'social_auth.backends.contrib.weibo.WeiboBackend',
        # 必须加，否则django默认用户登录不上
        'django.contrib.auth.backends.ModelBackend',
    )

模板的一些配置

    TEMPLATE_CONTEXT_PROCESSORS = (
        'django.contrib.auth.context_processors.auth',
        # login 在template中可以用 "{% url socialauth_begin 'douban-oauth2' %}"
        'social_auth.context_processors.social_auth_by_type_backends',
        'social_auth.context_processors.social_auth_login_redirect',
    )

各种重定向连接

    SOCIAL_AUTH_LOGIN_URL = '/login-url/'
    SOCIAL_AUTH_LOGIN_ERROR_URL = '/login-error/'
    SOCIAL_AUTH_LOGIN_REDIRECT_URL = '/logged-in/'
    SOCIAL_AUTH_NEW_USER_REDIRECT_URL = '/new-users-redirect-url/'
    SOCIAL_AUTH_NEW_ASSOCIATION_REDIRECT_URL = '/new-association-redirect-url/'

各种key, secret

    SOCIAL_AUTH_WEIBO_KEY = 'YOUR KEY'
    SOCIAL_AUTH_WEIBO_SECRET = 'YOUR SECRET'

    SOCIAL_AUTH_QQ_KEY = 'YOUR KEY'
    SOCIAL_AUTH_QQ_SECRET = 'YOUR SECRET'

    SOCIAL_AUTH_DOUBAN_OAUTH2_KEY = 'YOUR KEY'
    SOCIAL_AUTH_DOUBAN_OAUTH2_SECRET = 'YOUR SECRET'

urls
---

    urlpatterns = patterns('',
        ...
        url(r'', include('social_auth.urls')),
        ...
    )

template
---

注意实现一下/logout的方法，用django自带的即可

    登录
    <li><a rel="nofollow" href="{% url socialauth_begin 'weibo' %}">weibo</a></li>
    <li><a rel="nofollow" href="{% url socialauth_begin 'qq' %}">qq</a></li>
    <li><a rel="nofollow" href="{% url socialauth_begin 'douban-oauth2' %}">douban</a></li>
    注销
    <a href="/logout" > 注销 </a>

试用
===


此时User表没有任何用户,登录方式有weibo,douban,qq

    In [0]: User.objects.all()
    Out[0]: []
    In [1]: UserSocialAuth.objects.all()
    Out[1]: []

打开`http://llovebaimuda.herokuapp.com/`,别忘了改host

注意注释或者不注释此句的区别 'social_auth.backends.pipeline.associate.associate_by_email',

先看不注释,我weibo,qq,douban的邮箱都是一个邮箱

注释掉associate_by_email
---

初始状态

    In [0]: User.objects.all()
    Out[0]: []
    In [1]: UserSocialAuth.objects.all()
    Out[1]: []

1. 用weibo登录

页面重定向到`http://llovebaimuda.herokuapp.com/new-users-redirect-url/`

    In [2]: User.objects.all()
    Out[2]: [<User: 咄咄369>]
    In [3]: UserSocialAuth.objects.all()
    Out[3]: [<UserSocialAuth: UserSocialAuth object>]

2. 点击注销，在用qq登录,重定向到`http://llovebaimuda.herokuapp.com/new-association-redirect-url/`

    In [4]: User.objects.all()
    Out[4]: [<User: 咄咄369>]
    In [5]: UserSocialAuth.objects.all()
    Out[5]: [<UserSocialAuth: UserSocialAuth object>, <UserSocialAuth: UserSocialAuth object>]

3. 点击注销，在用douban登录,重定向到`http://llovebaimuda.herokuapp.com/new-association-redirect-url/`

    In [6]: User.objects.all()
    Out[6]: [<User: 咄咄369>]
    In [7]: UserSocialAuth.objects.all()
    Out[7]: [<UserSocialAuth: UserSocialAuth object>, <UserSocialAuth: UserSocialAuth object>]

4. 在不注销的情况下点击任何一种登录都会跳转到`http://llovebaimuda.herokuapp.com/new-association-redirect-url/`

5. 点击注销后，再点击任何一种登录会跳到`http://llovebaimuda.herokuapp.com/logged-in/`

开启associate_by_email
---

初始状态

    In [0]: User.objects.all()
    Out[0]: []
    In [1]: UserSocialAuth.objects.all()
    Out[1]: []

1. 用weibo登录

页面重定向到`http://llovebaimuda.herokuapp.com/new-users-redirect-url/`

    In [2]: User.objects.all()
    Out[2]: [<User: 咄咄369>]
    In [3]: UserSocialAuth.objects.all()
    Out[3]: [<UserSocialAuth: UserSocialAuth object>]

2. 点击注销，在用qq登录,重定向到`http://llovebaimuda.herokuapp.com/new-users-redirect-url/'

    In [4]: User.objects.all()
    Out[4]: [<User: 咄咄349>, <User: 咄咄>]

    In [5]: UserSocialAuth.objects.all()
    Out[5]: [<UserSocialAuth: UserSocialAuth object>, <UserSocialAuth: UserSocialAuth object>]

3. 点击注销，在用douban登录,重定向到`http://llovebaimuda.herokuapp.com/new-users-redirect-url/'

    In [4]: User.objects.all()
    Out[4]: [<User: 咄咄349>, <User: 咄咄>, <User: 43901973>]

    In [5]: UserSocialAuth.objects.all()
    Out[5]: [<UserSocialAuth: UserSocialAuth object>, <UserSocialAuth: UserSocialAuth object>, <UserSocialAuth: UserSocialAuth object>]

4. 点击注销, 用任何一种方式登录(以qq为例), 页面跳转到`http://llovebaimuda.herokuapp.com/logged-in/`

5. 在注销的情况下使用任一一种与4不同的登录方式登录，会出现异常(以豆瓣为例)

    AuthAlreadyAssociated at /complete/douban-oauth2/
    This douban-oauth2 account is already in use.

各种重定向的思考
===

`SOCIAL_AUTH_LOGIN_URL = '/login-url/'`, 暂时没发现什么用

`SOCIAL_AUTH_LOGIN_ERROR_URL = '/login-error/'`, 登录异常，应该引导用户重新去登录

`SOCIAL_AUTH_LOGIN_REDIRECT_URL = '/logged-in/'`, 成功登录页面

`SOCIAL_AUTH_NEW_USER_REDIRECT_URL = '/new-users-redirect-url/'`, django部分新建用户,在这个页面可以引导用户设置邮箱之类的附加信息

`SOCIAL_AUTH_NEW_ASSOCIATION_REDIRECT_URL = '/new-association-redirect-url/'`, django用户新建了一个关联的三方账户，此链接没想出有什么特殊需求，可以直接引导到登录成功页面
