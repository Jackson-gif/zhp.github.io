# Hugo搭建博客（下）


**这是Hugo搭建博客的第二期，第一期请参考：[Hugo搭建博客（上） - Okzhp](http://okzhp.tk/hugo搭建博客-1/)**



## 1.Hugo(LoveIt)美化

本站对主题的改造体现在以下几个方面：

1. 鼠标点击爆炸/评论区输入框输入爆炸+震动
2. 卜算子访问量统计及网站运行时间
3. 设置valine评论
4. 副标题引用一言api

> 注意，本文所述js文件都放在static/js文件夹下。
>
> hugo静态文件一般放在static或者assets文件夹下，但是favicon.png在assets文件夹下无法打包至public，因此我把所有静态文件都放在static文件夹下了。

### 1.鼠标点击爆炸和输入框爆炸

> 效果：点击页面即可看到效果，文末评论框也可看到效果。
>
> 出处：点击特效未找到出处，输入特效是我在github中发现的一个项目[传送门](https://github.com/disjukr/activate-power-mode).

1. 将[click-boom.js](https://image.okzhp.tk/js/click-boom.js)、[input-boom.js](https://image.okzhp.tk/js/input-boom.js)下载至js文件夹。

2. 将`themes/LoveIt/layouts/partials/assets.html`复制到`layouts/partials/assets.html`（根目录下的文件优先级高于主题内的文件，在外部改动将不影响后续主题版本更新）

3. 编辑assets.html，在末尾引入以上两个js

   ```html
   <!-- 点击爆炸特效 -->
   <script type="text/javascript" src="/js/click-boom.js"></script>
   <!-- 输入框爆炸+震动 -->
   <script type="text/javascript" src="/js/input-boom.js"></script>
   <script type="text/javascript">
   POWERMODE.colorful = true; // make power mode colorful
   POWERMODE.shake = true; // turn off shake
   document.body.addEventListener('input', POWERMODE);
   </script>
   ```


至此，两个特效已经实现了。但是评论系统在开发环境默认是关闭的，LoveIt集成了好几种评论系统，以valine为例，在配置文件中搜索valine，把其下的enable = false改为true，然后在执行`hugo server -e production`即可看到评论系统。此时评论系统还只是半成品，需要设置其服务端才可以正常使用。这个后文再讲。

### 2.卜算子访问量统计/网站运行时间

官网[传送门](http://busuanzi.ibruce.info/)

效果如下，（本地网络的次数有误，上线后正常）

![](https://image.okzhp.tk/img/20230103004209.png)

1. 在assets.html引入js处添加以下：

``` html
 <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
<!-- 运行时间 -->
<script type="text/javascript" src="/js/cal-runtime.js"></script>
```

2. 在static/js文件夹下创建`cal-runtime.js`，粘贴以下内容：

```javascript
/* 站点运行时间 */
function siteTime() {
    let seconds = 1000;
    let minutes = 60000;
    let hours = 3600000;
    let days = 86400000;
    let years = 31536000000;
    let diff = new Date() - new Date('12/31/2022 00:00:00');
    // let diffYears = Math.floor(diff / years);
    let diffDays = Math.floor(diff / days);
    let diffHours = Math.floor((diff%days)/hours);
    let diffMinutes = Math.floor((diff%hours) / minutes);
    let diffSeconds = Math.floor((diff%minutes) / seconds);
    let runbox = document.getElementById('run-time');
    runbox.innerHTML = '本站已运行<i class="far fa-clock fa-fw"></i> ' +
        ((diffDays < 10) ? '0' : '') + diffDays + ' 天 ' +
        ((diffHours < 10) ? '0' : '') + diffHours + ' 时 ' +
        ((diffMinutes < 10) ? '0' : '') + diffMinutes + ' 分 ' +
        ((diffSeconds < 10) ? '0' : '') + diffSeconds + ' 秒 ';
}
setInterval(siteTime, 1000);
```

3. 将`themes/LoveIt/layouts/partials/footer.html`复制到`layouts/partials/footer.html`
4. 在`<div class="footer-container">`这一行下边添加以下：

```html
<!-- 运行时间 -->
        {{ if .Site.Params.runtime.enable -}}
        <div class="footer-line">
            <span id="run-time"></span>
        </div>
        {{- end -}}

        <!-- 访问量 -->
        <!-- busuanzi -->
        {{ if .Site.Params.busuanzi.enable -}}
        <div class="busuanzi-footer">
            {{ if .Site.Params.busuanzi.sitePV -}}
            
            <span id="busuanzi_container_site_pv">
                <i class="far fa-eye"></i>
                <span id="busuanzi_value_site_pv"></span>次
            </span>
            {{- end -}}
            &nbsp;|&nbsp;
            {{ if .Site.Params.busuanzi.siteUV -}}
            <span id="busuanzi_container_site_uv">
                <i class="fas fa-users"></i>
                <span id="busuanzi_value_site_uv"></span>人次
            </span>
            {{- end -}}
        </div>
        {{- end -}}
```

至此，卜算子访问量统计和网站运行时间已经实现。

> 在配置文件中，可以找到  [params.runtime]和  [params.busuanzi]这两项配置，通过这些开关可以控制访问量和运行时间是否进行展示

### 3.valine评论

官网[传送门](https://valine.js.org/quickstart.html)

需要登录leanCloud，创建一个应用，获取AppId和AppKey，并在配置文件中进行配置(`ctrl+f`查找)，除此外还需配置severUrls才可正常使用评论功能。

三个参数分别如下图：

![](https://image.okzhp.tk/img/20230103002056.png)

如果有域名，最好绑定自己的域名。

设置好以上三个参数就可以正常使用评论功能了。valine已经没有邮箱评论通知的功能了，如果需要评论邮箱提醒需要使用`valine-admin`，使用方法请自行查阅在此不再赘述。还有其他几种评论系统，请自行尝试。

顺带提一下，在后台也可以对评论进行一些操作：

![image-20230103183610395](https://image.okzhp.tk/img/image-20230103183610395.png)

### 4.副标题引用一言Api

官网[一言开发者中心 | 用代码表达言语的魅力，用代码书写山河的壮丽。 (hitokoto.cn)](https://developer.hitokoto.cn/)

首先在配置文件中搜索`subtitle`，将其值设为`" "`，注意有一个空格，如果没有空格将不存在副标题。空格只是打开开关，然后由js设置为一言的内容。

类似的，在上文提到的assets.html末尾添加以下内容：

```html
<!-- typeit -->
<script src="https://cdn.jsdelivr.net/npm/typeit@6.1.1/dist/typeit.min.js"></script>
<!-- 副标题文字请求一言接口 -->
<script type="text/javascript">

    // a动画b漫画c游戏d文学e原创f来自网络g其他h影视i诗词j网易云k哲学l抖机灵
    let url = "https://v1.hitokoto.cn/?c=a&c=b&c=k&c=j&c=i&c=c&encode=text"
    oneWord();
    function oneWord() {
        let req = new XMLHttpRequest();
        req.open("GET", url);
        req.responseType = "text"
        req.send();
        req.onload = function() {
            let text = "一个人在他停止学习的时候就已经死了";
            if (req.status === 200) { // 分析响应的 HTTP 状态
                text = req.responseText;
            }
            new TypeIt("#id-1", {
                strings: text,
            }).go();
        };
    }
</script>

```

> 这里需要引入typeit的js，即使主题已经有了包含了typeit，但此处不引入会导致副标题多一个换行。



## 2.部署及优化

> 秉着能省则省的原则，我将用零成本打造一个尽可能舒适的博客体验。

### 1.部署到github pages

当调整完布局和内容后，我们就可以开始部署（托管）到github pages了。

首先需要在github建立一个public仓库，仓库名为 `用户名.github.io`。

在博客的根目录执行`hugo`命令，此命令将在public文件夹中构建出该博客网站。

使用git前先设置git的用户名和邮箱：

```shell
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

> git网络不太稳定，很有可能push失败，最好使用翻墙网络。
>
> 在cmd中设置http和https代理方法：
>
> set http_proxy=http://127.0.0.1:端口号
> set https_proxy=http://127.0.0.1:端口号
>
> 通常clash http端口号为7890

```shell
cd public
#将public初始化为git仓库，后续将通过git管理内容
git init

# 关联远程仓库
git remote add origin https://github.com/okzhp/okzhp.github.io.git
# 将所有内容添加到git本地仓库
git add .
git commit -m "第一次提交"
#提交到git远程仓库 首次提交可能需要身份认证，点击在浏览器中认证
git push -u origin master
```

push完后在仓库页面，选择部署分支为当前分支master，

![](https://image.okzhp.tk/img/20230103172740.png)

github将自动部署该博客，通过`用户名.github.io`即可访问。

后续更新文章通过命令:

```shell
#查看仓库状态
git status
git add .
git commit -m "填写提交备注"
# 如果push失败，可能是网络不行或者远程仓库和本地仓库内容有差异，请先执行git pull master然后再push
git push origin master
```



### 2.域名申请/绑定

通过`用户名.github.io`虽然可以访问，但还差点意思，如果通过自己的域名访问那就更好了。

国内许多平台可以买域名，但是国内的域名需要备案才能使用。因此我选择了国外的域名，而且是免费的。

> 这部分涉及的网站需要科学上网才可访问，请自行解决。

我申请免费域名的方法暂时已经失效了，域名的问题请自行解决，网络上应该有许多免费域名的获取方法。

在域名服务商的管理台正确配置DNS解析。

github.io的ip地址为： 

1. 185.199.108.153 
2.  185.199.109.153
3.  185.199.110.153
4.  185.199.111.153

在DNS解析处添加以下两条配置：

1. A类型  你的域名(例如okzhp.tk)   上面四个ip之一(例如185.199.108.153)
2. CNAME  www   仓库地址(例如okzhp.github.io)

上边的配置将okzhp.tk解析到185.199.108.153，将www.okzhp.tk解析到okzhp.github.io

DNS解析通常要几分钟才能生效，随后在github page页面配置该域名即可：

![image-20230103175410983](https://image.okzhp.tk/img/image-20230103175410983.png)

至此就完成了对域名的配置。

### 2.图床配置

图床，即存储图片的服务器，通过一个链接即可方便的进行访问。例如本站图片都是存储在七牛云服务器上，配置完图床对于处理图片将会非常的方便。本站的图床配置是通过[七牛云](https://sso.qiniu.com/)和[PicGo](https://picgo.github.io/PicGo-Doc/zh/guide/#下载安装)实现的。

> 注意：图床配置需要有自己的域名才可以，因为七牛云只提供一个有效期一个月的仅供测试用的域名，因此需要绑定自己的域名。七牛云有若干个地区，国内的地区使用域名必须有备案，换言之，没有备案的域名只能使用国外的存储空间。例如我用的存储区域就在北美。

#### 七牛云

首先注册登录七牛云，需要进行身份认证和邮箱绑定，完成后每个月将有10g的存储空间和10g的下载流量。对于个人博客来说，简直不要太完美。

首先打开对象存储-空间管理-新建空间，如图：

![](https://image.okzhp.tk/img/20230103011907.png)

新建一个空间，创建完毕后，在对应位置记住红框内的五个参数，后面要使用：

![image-20230103180735685](https://image.okzhp.tk/img/image-20230103180735685.png)

#### PicGo

在[PicGo is Here | PicGo](https://picgo.github.io/PicGo-Doc/zh/guide/#下载安装)页面下载PicGo。在设置中进行参数配置：

![image-20230103181243713](https://image.okzhp.tk/img/image-20230103181243713.png)

设定访问网址即七牛云提供的临时域名或者绑定的域名。区域则填对应的区域代码：华东z0，华北z1，华南z2，北美na0，东南亚as0 

存储路径则是你上传的路径。

复制一张图片，使用`ctrl`+`shift`+`p`快捷键进行上传，上传成功即可。

随后在md编辑器typora偏好设置-图像 选择picgo安装路径：

![image-20230103181817426](https://image.okzhp.tk/img/image-20230103181817426.png)

配置完毕后，在typora粘贴图片时将自动上传至七牛云并引用其超链接，可以很方便的引用图片。

> typora目前的版本都是收费的，其支持图床配置的最后一个免费版本是0.9.96

## 3.后续

整个博客的搭建大概花了三天时间，第一天搭建了主要的骨架，第二天处理域名和valine评论相关，第三天做了一些美化和图床处理。本打算第四天发文记录，结果拖到第五天才开始下笔，愣是写了两天才才把大致过程略讲了一遍，而且有很多地方一笔带过了。总之，这也是一个总结复盘的过程。

个人认为，博客还存在以下几个方面待改进：

- 图片存储在国外，访问速度较慢
- 域名虽是免费的，但没有备案无法在国内使用，且有效期仅有一年。
- 尚未完成部署脚本一键部署
- 评论邮箱提醒功能缺失（由于leanCloud自动唤醒被禁止了）

至此，博客已经搭建完毕🤖

如果有问题欢迎留言讨论🤖

总之，本次博客搭建参考了大量网络内容，包括但不限于以下参考：

- [Github Pages + Hugo 搭建个人博客 - 渣渣的夏天 (zz2summer.github.io)](https://zz2summer.github.io/github-pages-hugo-搭建个人博客/)

- [Hugo系列(3.1) - LoveIt主题美化与博客功能增强 · 第二章 - Yulin Lewis' Blog (lewky.cn)](https://lewky.cn/posts/hugo-3.1.html/)

  

