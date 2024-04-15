---
title: 思维导图在hexo页面展示
date: 2023-02-22 09:46:23
categories: technique
tags: [hexo]
urlname: 13
---

这些天整理自己之前的产出，大多是思维导图，于是想让其优雅地在博客页面展示。

起先想到直接导出图片格式，在页面展示。但是节点和文字一多，清晰度和可操作性就大大降低。

接着考虑SVG格式，可以使用[Try markmap][1]——markmap的在线工具，当然这个得先转换成Markdown格式

那就直接使用[hexo-markmap插件][2];O) 

### processon思维导图导出
processon支持将思维导图导出成POS文件（.pos）、Xmind文件（.xmind）以及FreeMind文件（.mm）


### 思维导图转成Markdown
可以利用现成的思维导图软件将思维导图导出成markdown
当然，像我一样没有安装这些软件的可以直接使用百度脑图。
不过，百度脑图导入的文件格式存在限制：
![百度脑图文件上传][3]
processon导出的思维导图，FreeMind文件（.mm）可以在百度脑图导入成功。
导入后，可以再导出为markdown格式的文件

ps：明明支持.xmind，但是笔者导入后思维导图显示乱码

> Github也有一些将思维导图转成markdown的程序，如下：

POS文件和Xmind文件转成Markdown文件（Java）：[Xmind-to-md][4]

KityMinder文件转成Markdown文件（Python）：[km2md][5]
```
import json

class Node:
    def __init__(self,data,children:list):
        self.data = data
        self.text = ' '+data['text']
        self.children = []
        for child in self.getchildren(children):
            node = Node(child['data'],child['children'])
            self.children.append(node)
        
    def getchildren(self,children:list):
        realchildren = []
        for child in children:
            if child != None:
                realchildren.append(child)
        return realchildren

def node2md(md,node:Node,layer:int):
    if node.children is None:
        return md
    else:
        for child in node.children:
            md += '#'*layer+ child.text + '\n' + node2md('',child,layer+1)
        layer += 1
        return md

if __name__ == '__main__':
    path = 'xxx.km'
    md_path = 'xxx.md'
    
    with open(path, 'r') as f:
        content = json.load(f)
    
    root = Node(content['root']['data'],content['root']['children'])
    md = '#' + root.text + '\n'
    layer = 2

    md = node2md(md,root,layer)

    with open(md_path,'w') as f2:
        f2.write(md)
    print(md)
    print('ok,please check your md file')
```
Xmind文件转成Markdown文件（Python）：[XmindGenMarkdown][6]
```
import json
from zipfile import ZipFile


class XmindFileParser:
    content_json = "content.json"
    xmindFileContent = [content_json]
    markdown_file_content = ""

    @classmethod
    def parse(self, file_path):
        file_content = self.__unzip(self, file_path)
        file_json_content = self.__parse_json(file_content)
        self.__parse_children(self, file_json_content)
        self.__generat_file(self)

    def __unzip(self, file_path):
        with ZipFile(file_path) as xmind_file:
            for f in xmind_file.namelist():
                for key in self.xmindFileContent:
                    if f == key:
                        with xmind_file.open(f) as contentJsonFile:
                            return contentJsonFile.read().decode('utf-8')

    def __parse_json(file_content):
        return json.loads(file_content)

    def __parse_children(self, json_content):
        root_topic = json_content[0]["rootTopic"]
        cur_node = root_topic
        self.__parse_node(self, cur_node, "#")

    '''
    递归解析 children 节点
    '''

    def __parse_node(self, node, level):
        self.markdown_file_content = self.markdown_file_content + "\n" + level + " " + node["title"] + "\n"
        if "notes" in node:
            # TODO 内容 "\n" 需要替换为 "  \n"
            content = node["notes"]["plain"]["content"]
            self.markdown_file_content = self.markdown_file_content + "\n" + node["notes"]["plain"]["content"] + "\n"
        if "children" in node:
            for cur_node in node["children"]["attached"]:
                self.__parse_node(self, cur_node, level + "#")

    def __generat_file(self):
        file = open('个人学习.md', 'w', encoding="utf-8")
        file.write("[TOC]\n\n" + self.markdown_file_content)
        file.close()
      
```
### hexo思维导图展示
首先安装hexo-markmap 插件

`npm install hexo-markmap `

接着在思维导图文字内容的首部和末尾分别增加
`{% markmap 800px %}`

`{% endmarkmap %}`

### hexo的其他插件

> hexo-tag-plugins

使用该插件的<minmap>标签，使用方法（例）：
```
{% mindmap 600 %}
- 计算机网络概述
	-  计算机网络的基本概念
		- 以能够相互共享资源的方式互连起来的自治计算机系统的集合
	- 计算机网络的组成
		- 组成部分角度
            - 硬件：主机（端系统）、通信链路（双绞线、光纤等）、交换设备（路由器、交换机等）、通信处理机（网卡等
		- 工作方式角度
            - 核心部分：大量的网络和连接这些网络的路由器组成
{% endmindmap %}
```
与markmap的不同：高度单位不加px；使用缩进构成不同层级



[1]: https://markmap.js.org/repl/
[2]: https://github.com/gera2ld/markmap
[3]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/tech/naotu-upload.png
[4]: https://github.com/NotInWine/xmind-to-md
[5]: https://github.com/Ash-one/km2md
[6]: https://github.com/LiWenGu/xmind-zen2markdown