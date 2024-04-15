---
 title: 搭建Github个人图床
 date: 2020-12-25 06:26:00
 categories: technique
 tags: [image host]
 urlname: 5
--- 


## 定义
图床一般指储存图片的服务器，将图片上传至图床后可以直接通过外链访问，利用图床进行图片存储可以减轻网站服务器的压力、提供CDN加速等等。

## 平台
目前的有许多图床平台供选择，比如[腾讯云COS][1]、Github图床、[阿里云OSS][2]、[七牛云图床][3]、[SM.MS图床][4]等，更多国内图床可以参考知乎链接：[盘点国内免费好用的图床][5]。在对比不同图床的稳定性、容量、价格等因素后，选择了免费又可靠的Github图床。

## 工具

图床工具：PicGo  [下载地址][6]
选择本地的图片上传工具可以方便我们快速上传图片并且获取图片外链，PicGo是较为推荐的工具，它提供剪切板图片上传、多种图片外链格式、支持多种图床……
![PicGo][7]

## 配置
（1）Github配置

①登录[Github][8]，创建新的仓库，权限设置为public

②点击settings->Developer settings->Personal access tokens->tokens (classic)->Generate new token，选择repo权限后点击绿色的Generate token按钮，成功生成token，需要将其记录下来避免再次生成。
![token][9]

（2）PicGo配置
点击PicGo中的Github图床配置，按照“账户名/仓库名”的格式设置仓库名；分支名填写github仓库的默认分支（2020.10.01后，github的默认分支名由master变更为main，也可以选择其他分支，但是自定义域名格式处需要对应）；将Github配置图床时的token粘贴至设定token的文本框中；存储路径可选；为了使用CDN加快图片的访问速度，自定义域名格式为：https://cdn.jsdelivr.net/gh/GitHub用户名/图床仓库名@main；点击确定。
![PicGo配置][10]

## 图片上传

（1）使用PicGo上传图片至指定的Github图床仓库，如果要进行图片的删除，在PicGo中操作不会对Github中的图片进行删除。

（2）如果上传的图片过大或者因为服务器不稳定导致PicGo上传失败，可以直接进入Github的图床仓库上传图片，访问的外链则使用PicGo中的自定义域名加具体图片路径。

  [1]: https://cloud.tencent.com
  [2]: https://www.aliyun.com/product/oss/
  [3]: https://portal.qiniu.com
  [4]: https://sm.ms
  [5]: https://zhuanlan.zhihu.com/p/35270383
  [6]: https://github.com/Molunerfinn/PicGo
  [7]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/ImgHost/PicGo.png
  [8]: https://github.com/
  [9]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/ImgHost/token.png
  [10]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/ImgHost/PicGo-settings.png