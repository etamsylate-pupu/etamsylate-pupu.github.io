---
 title: USTC-TK2016工具使用教程
 date: 2021-01-29 04:13:00
 categories: technique
 tags: []
 urlname: 8
--- 

工具下载地址：[USTC-TK2016][1]  ps：使用git较zip下载更好

该工具的github README中有基本的安装和使用步骤，针对不同的运行环境会有一些小问题，以下是本人参考[博文][2]后重新整理的安装使用教程

**环境**

Windows 10操作系统
python 3.7
powershell 5.1
splitcap 

Git安装
-----
①windows安装git：[下载地址][3]，根据网上的安装教程进行安装，安装步骤中大部分设置为默认
②打开Git Bash，进入自己的工作目录，然后输入以下对应的git clone命令

    # Clone the repository on "master" branch
    $ git clone -b master https://github.com/yungshenglu/USTC-TK2016

使用
-----
(1) 将pcap数据集放入1_Pcap文件夹

(2)执行1_Pcap2Session.ps1文件：

①ps1文件需要在powershell中运行，Windows 10系统自带powershell。在任务栏中搜索powershell并点击右键使用管理员权限运行；

②在powershell中输入`set-executionpolicy remotesigned`，更改执行策略;
![powershell_config][4]

③如果将pcap文件按照session切割，将1_Pcap2Session.ps1文件的第10和第14行取消注释，第11行和第15行注释掉；如果将pcap文件按照flow切割，与前面恰好相反；
![code][5]

④在powershell中进入USTC-TK2016目录，输入`.\1_Pcap2Session.ps1`尝试运行ps1文件，出现：`处理的异常: System.TypeInitializationException: "SplitCap.Program"的类型初始值设定项引发异常。`和`USTC-TK2016\2_Session\AllLayers\…找不到指定文件`的错误。前者是由于Splitcap版本过低导致的，在splitcap下载地址：[link][6]中下载最新版本的splitcap，将exe文件放入原来的0_Tool\SplitCap_2-1文件夹下，删除原来的SplitCap.exe文件，或者在ps1文件中修改代码；后者按照源代码中的输出文件格式`2_Session\AllLayers\$($f.BaseName)-ALL`和`2_Session\AllLayers\$($f.BaseName)-L7`（变量是pcap文件除去文件后缀的文件名），手动在指定目录下创建对应的文件夹

⑤再次运行，执行成功后在`2_Session\AllLayers\`和`2_Session\L7\`目录下会有对应的pcap文件

(3) 执行2_ProcessSession.ps1文件：

①打开ps1文件，将需要处理的源文件目录变量修改为：`$SOURCE_SESSION_DIR = "2_Session\L7"`或者`$SOURCE_SESSION_DIR = "2_Session\AllLayers"`，即上一步数据切割后保存的文件目录
②根据ps1文件中的以下代码手动创建对应的文件夹

    $paths = @(('3_ProcessedSession\FilteredSession\Train', '3_ProcessedSession\TrimedSession\Train'), ('3_ProcessedSession\FilteredSession\Test', '3_ProcessedSession\TrimedSession\Test'))

③输入`.\2_ProcessSession.ps1`尝试运行ps1文件，执行成功后

    FilteredSession\ - Get the top 60000 large PCAP files
    TrimedSession\ - Trim the filtered PCAP files into size 784 bytes (28 x 28) and append 0x00 if the PCAP file is shorter than 784 bytes
    The files in subdirectory Test\ and Train\ is random picked from dataset.

(4) 执行3_Session2Png.py文件：

①命令行输入`python 3_Session2Png.py`运行，生成的图片保存在4_Png目录下；

②运行时报错：`TypeError: slice indices must be integers or None or have an __index__ method`，根据报错提示查看对应的代码`fh = numpy.reshape(fh[:rn*width],(-1,width)) `，因为[]中的数据变成了浮点数，不能作为数组的下标，需要将其强制转换为int型，修改为`fh = numpy.reshape(fh[:int(rn*width)],(-1,width)) `，或者因为rn的值源于前面的`rn = len(fh)/width`，这里除法保留了小数部分，所以需要将单斜杠变为双斜杠只保留整数部分

(5) 执行4_Png2Mnist.py文件：

①原文件的注释中要求python>2.5，代码中的print函数使用表明为python2环境，需要修改print语句为python3版本

②如果需要添加print语句测试某些变量，在增加代码时会遇到缩进的问题，因为Git的代码通常会出现4个空格和tab键不匹配问题，可以参考[link][7]进行修改


  [1]: https://github.com/yungshenglu/USTC-TK2016
  [2]: https://blog.csdn.net/u010916338/article/details/86511009
  [3]: https://git-scm.com/download/win
  [4]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/traffic_tool/powershell_config.png
  [5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/traffic_tool/1_code.png
  [6]: https://www.netresec.com/?page=SplitCap
  [7]: https://blog.csdn.net/hhy_csdn/article/details/82263757