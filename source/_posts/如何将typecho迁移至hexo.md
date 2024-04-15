---
title: 如何将数据从typecho迁移至hexo?
date: 2023-02-11 16:13:12
categories: technique
tags: [hexo]
urlname: 10
---

## 备份typecho数据


将使用typecho搭建的网站数据库进行备份

## 将备份的数据库文件在本地导入

本地安装mysql以及数据库管理工具Navicat。使用Navicat新建本地连接，并新建typecho数据库，在typecho数据库中运行备份的sql文件，运行完成后即成功将相关的表和数据导入。
![typecho database][1]

## 编写typecho数据转成Markdown的脚本

借鉴[tanmx][2]的博客，使用pymysql模块，稍微修改后代码如下：
```
# -*- coding: utf-8 -*-

import codecs
import os
#import torndb
import pymysql
import arrow

def create_data(cursor):
    # 创建分类和标签
    sql  = "select type, slug, name from typecho_metas"
    cursor.execute(sql)
    categories = cursor.fetchall()
    print(categories)
    for cate in categories:
        path = 'data/%s' % cate[1]
        if not os.path.exists(path):
            os.makedirs(path)
        f = codecs.open('%s/index.md' % path, 'w', "utf-8")
        f.write("---")
        f.write("\n title: %s " % cate[1])
        f.write("\n date: %s" % arrow.now().format('YYYY-MM-DD HH:mm:ss'))
        # 区分分类和标签
        if cate[0] == 'category':
            f.write('\n type: "categories"')
        elif cate[0] == 'tags':
            f.write('\n type: "tags"')
        # 禁止评论
        f.write("\n comments: true")
        f.write("\n--- \n ")
        f.close()

    # 创建文章
    sql1 = "select cid, title, slug, text, created from typecho_contents where type='post'"
    cursor.execute(sql1)
    entries = cursor.fetchall()
    for e in entries:
        title = e[1]
        urlname = e[2]
        print(title)
        content = str(e[3]).replace('<!--markdown-->', '')
        tags = []
        category = ""
        # 找出文章的tag及category
        sql2 = "select type, name, slug from `typecho_relationships` ts, typecho_metas tm where tm.mid = ts.mid and ts.cid = {}".format(e[0])
        cursor.execute(sql2)
        metas = cursor.fetchall()
        for m in metas:
            if m[0] == 'tag':
                tags.append(m[1])
            if m[0] == 'category':
                category = m[2]
        path = 'data/_posts/'
        if not os.path.exists(path):
            os.makedirs(path)
        f = codecs.open(r"%s%s.md" % (path,title), 'w', "utf-8")
        f.write("---")
        f.write("\n title: %s" % title)
        f.write("\n date: %s" % arrow.get(e[4]).format('YYYY-MM-DD HH:mm:ss'))
        f.write("\n categories: %s" % category)
        f.write("\n tags: [%s]" % ','.join(tags))
        f.write("\n urlname: %s" % urlname)
        f.write("\n--- \n")
        f.write(content)
        f.close()


def main():
    # 数据库连接信息
    db = pymysql.Connection(host="127.0.0.1", database="typecho", user="你设置的用户名", password="用户名对应的密码")
    #modified
    cursor = db.cursor()
    create_data(cursor)

if __name__ == "__main__":
    main()
```

## 将生成的文件放入hexo博客目录

将data\_posts中的.md文件放入source\_posts目录下，
将data\下的其余文件夹（分类，即原来的typecho_metas数据）放入source\目录下








  [1]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/tech/typecho_database.png
  [2]: https://www.tinymind.net.cn/articles/f902089fb99b43