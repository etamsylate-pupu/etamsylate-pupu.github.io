---
 title: typecho建站Tips
 date: 2020-12-12 00:39:00
 categories: technique
 tags: [website]
 urlname: 4
--- 
## 域名以及服务器
提供域名购买的网站有许多，包括阿里的[万网][1]、世界最大的域名注册商[Godaddy][2]、腾讯的[腾讯云][3]、华为的[华为云][4]等，域名注册的网站一般都提供首年特惠或者直接购买多年平均下来比续费划算的方案，不同的域名注册网站价格也稍有变化，国内的貌似更便宜些。

提供服务器购买的网站除了上面的阿里腾讯华为，还有[优网][5]、[主机壳][6]、[百度智能云][7]等，目前国内的两大巨头还是阿里云和腾讯云，都提供学生优惠，不过阿里率先进入这个行业，可能稳定性和安全性方面稍强（猜测？

在国内注册购买域名一般需要进行网站备案，腾讯、阿里、华为都提供较为便捷的备案流程，这里选择的是阿里的域名和它提供的学生优惠服务器。

（1）域名购买

购买域名步骤：

①实名认证
域名持有者信息需要与备案主体信息保持一致，包含姓名、证件类型、证件号码邮箱等等，其中邮箱需要进行真实性认证。

②查询想购买的域名后缀是否能备案
进入工信部网站的[中国互联网体系][8]查看工信部已经批复的域名后缀。

③域名购买
将域名加入清单进行结算，选择购买年限、选择域名持有者为个人并添加实名认证的个人信息模板，接着同意域名购买条款后购买。

（2）服务器购买
针对搭建个人小型网站的需求，轻量应用服务器就能满足。步骤如下：
①进行学生认证

②服务器地域节点选择
根据自己所在的地域选择服务器地域，目前大陆的服务器地域包含华北1（青岛）、华北2（北京）、华北3（张家口）、华北5（呼和浩特）、华东1（杭州）、华东2（上海）、华南1（深圳），选择一个较近的地域减小延时。

③镜像选择
阿里云轻量应用服务器提供应用镜像和系统镜像，系统镜像是单纯的Linux系统，而应用镜像则是部署了软件和环境的系统。对于不了解Linux系统操作或者希望一键使用的人可以直接选择相应的应用镜像，想自己配环境的则可以选择合适的系统镜像。

④绑定域名进行域名解析
为了成功进行备案，至少购买三个月的服务器。服务器购买成功后进入其管理页面，查看对应的公网IP。将购买的域名进行添加，按照提示进行A记录解析。

⑤服务器管理软件安装
如果选择的是系统镜像，为了方便可以下载提升运维效率的宝塔Linux面板，宝塔支持一键LAMP（Linux、Apache、Mysql、phpstudy）以及LNMP（Linux、Nginx、Mysql、phpstudy）等多项服务器管理功能。宝塔需要开放系统的8888端口，在安全组中的防火墙设置里添加规则将8888端口开放。

点击服务器的远程连接，在终端输入以下命令进行宝塔的安装：
`wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh`
安装成功后会出现宝塔面板的访问网址以及用户名和密码，将网站复制粘贴至导航栏进行访问，输入对应的用户名和密码，即可进入宝塔界面。宝塔登陆后会自动跳出安装web环境的界面，根据个人需求选择，比如我选择的是LAMP，如果下载失败可以点击软件商店再次下载。
![宝塔界面][9]
初始的宝塔账户和密码不方便记忆，可以点击面板设置自定义。

## 域名备案
具体的备案流程见阿里云的[ICP备案流程][10]，帮助文档比较详细。需要注意的是，在购买域名并进行实名认证后，为了确保实名认证信息入库管局，建议在实名认证完成后的三天再申请备案，否则可能存在管局审核检查不到最新域名实名认证信息，导致备案失败。

直接使用阿里的备案服务，在填写网站名称时可以根据网站名称指引进行填写，需要注意一些敏感词是不被允许的。
![网站名称填写][11]
提交备案信息后阿里的工作人员会拨打电话进行备案信息的确认，如果网站名称不合适会进行告知并且帮助修改。

在阿里提交备案信息后工信部会发送短信进行认证，需要在收到信息后的24小时内登录[工信部备案管理系统][12]进行验证。

由于阿里的备案流程界面短信校验进度不会及时更新，所以只要看到“尊敬的ICP用户：您的短信核验已全部完成，该请求将提交管局审核”就不必担心，耐心等待审核结果。

验证成功后将会提交至管局进行审核，审核成功后将会收到工业和信息化部发送的ICP备案号，至此备案成功。

## typecho模板使用
在宝塔界面点击网站添加站点，填写对应网站的信息，根目录默认为/www/wwwroot，添加成功后会创建相应域名的文件夹。安装typecho模板和主题的步骤如下：

①模板下载安装
进入tepecho官网[下载页][13]，查看其正式版的环境需求（宝塔中安装的php版本需要满足在5.4以上），下载后进行解压，将文件放置在对应的域名文件夹下，如果还未备案成功需要通过IP访问进行typecho安装，备案成功后经域名解析通过IP和域名都可进行访问，按照页面提示创建账户进行模板安装。

②主题安装
提供typecho主题的站点：typecho官方主题站点[Typecho Themes][14]、[Npcink][15]等，选择喜欢的主题下载后将文件夹放入/usr/themes目录下

③主题应用
访问自己的网站进入后台点击控制台选项中的外观，启用安装好的主题，本人主题使用说明[Akina][16]

## 开启HTTPS
阿里目前提供免费的SSL证书服务，详情如下图
![阿里SSL][17]
证书部署步骤如下：

①认证
按照阿里提示的操作步骤，购买成功后提交证书申请材料并进行域名所有权认证，成功后证书签发。
![已签发][18]

②下载和配置
点击证书下载按钮，根据服务器类型下载对应证书。我的服务器为Apache，证书包含的文件如下
![证书][19]
进入宝塔界面，点击个人网站中的设置，选项中存在SSL，点击其他证书，将下载的证书文件中的key文件粘贴至密钥key文本框，如果是Apache服务器的证书，将public.crt放在前面，chain.crt放在后面粘贴在证书PEM格式的文本框中，若是Nginx则按照PEM格式证书拼接的提示。粘贴完毕后点击保存按钮并开启窗口右上角的强制HTTPS，证书部署成功后显示绿色的文本框提示。
![BT-SSL][20]
进入typecho文件目录，编辑config.inc.php文件，添加如下代码：

    define('__TYPECHO_SECURE__', 'true');

浏览器上进入网站后台，在设置->基本设置中的站点地址输入开启HTTPS后的URL链接

## 配置伪静态
（1）如果直接使用宝塔面板进行管理，则可以进入宝塔管理界面，点击网站->设置->伪静态，在“规则转换工具”左侧的下拉框中选择“wordpress”或者“typecho”进行保存（这里宝塔的版本不同会有差异）。
![伪静态配置][21]

（2）如果不使用宝塔面板的伪静态配置，可以参考[Apache伪静态][22]，但是需要注意apache配置文件所在的位置以及修改添加的代码中的个人网站目录

（3）进入网站后台，点击设置->永久链接，启用地址重写功能，出现错误后勾选仍然启用。

PS：由于是之前搭建的网站，有一些细节方面可能说的不太准确，欢迎指正(╹ڡ╹ )


  [1]: https://wanwang.aliyun.com/
  [2]: https://sg.godaddy.com/zh
  [3]: https://dnspod.cloud.tencent.com/
  [4]: https://www.huaweicloud.com/product/domain.html
  [5]: https://www.youwebcloud.com/
  [6]: http://www.zhujike.com/
  [7]: https://cloud.baidu.com/
  [8]: http://xn--eqrt2g.xn--vuq861b/?spm=a2c4g.11186623.2.25.43314dc1O7oJzn#
  [9]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/BT-1.png
  [10]: https://help.aliyun.com/document_detail/61819.html?spm=a2c4g.11186623.3.2.56fc7bb3drkZ0l
  [11]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/step-1.png
  [12]: https://beian.miit.gov.cn/?spm=a2c4g.11186623.2.19.56d14b11fEgHLN#/Integrated/ComplaintA
  [13]: http://typecho.org/download
  [14]: https://typecho.me/
  [15]: https://www.npc.ink/tag/typecho-theme
  [16]: https://zhebk.cn/Web/userAkina.html
  [17]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/SSL.png
  [18]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/cert.png
  [19]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/crt.png
  [20]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/BT-SSL.png
  [21]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/construct/apache-config.png
  [22]: https://www.typechodev.com/servers/remove_index_for_apache.html