---
slug: "bilibili-bullets-data-step1"
title: "B站弹幕大数据(1) - 视频列表的获取"
keywords: "B站,bilibili,弹幕,大数据"
description: "B站弹幕大数据第一步，获取分类视频、番剧视频等列表。"
date: 2017-10-07T21:10:10+08:00
draft: false
---

不同类型视频的获取方法不同：按分类获取最新视频、按分类获取热门视频、番剧视频获取

---

## 普通视频列表

#### 1.按每个分类的tid分开获取

**动画**
* tid-24 MAD·AMV
* tid-25 MMD·3D
* tid-47 短片·手书·配音
* tid-27 综合

**国创**
* tid-153 国产动画
* tid-168 国产原创相关
* tid-169 布袋戏
* tid-170 资讯

**音乐**
* tid-28 原创音乐
* tid-31 翻唱
* tid-30 VOCALOID·UTAU
* tid-59 演奏
* tid-29 三次元音乐
* tid-54 OP/ED/OST
* tid-130 音乐选集

**舞蹈**
* tid-20 宅舞
* tid-154 三次元舞蹈
* tid-156 舞蹈教程

**游戏**
* tid-17 单机游戏
* tid-171 电子竞技
* tid-172 手机游戏
* tid-65 网络游戏
* tid-173 桌游棋牌
* tid-121 GMV
* tid-136 音游
* tid-19 Mugen

**科技**
* tid-124 趣味科普人文
* tid-122 野生技术协会
* tid-39 演讲·公开课
* tid-96 星海
* tid-95 数码
* tid-98 机械
* tid-176 汽车

**生活**
* tid-138 搞笑
* tid-21 日常
* tid-76 美食圈
* tid-75 动物圈
* tid-161 手工
* tid-162 绘画
* tid-175 ASMR
* tid-163 运动
* tid-174 其他

**鬼畜**
* tid-22 鬼畜调教
* tid-26 音MAD
* tid-126 人力VOCALOID
* tid-127 教程演示

**时尚**
* tid-157 美妆
* tid-158 服饰
* tid-164 健身
* tid-159 资讯

**广告**
* tid-166 广告

**娱乐**
* tid-71 综艺
* tid-137 明星
* tid-131 Korea相关

**影视**
* tid-182 影视杂谈
* tid-183 影视剪辑
* tid-85 短片
* tid-184 预告·资讯
* tid-86 特摄

返回的json中键"data"中"archives"就是该版块的最新视频数据，这里需要获取的数据是aid（也就是AV号）

例如游戏/单机游戏tid为17
最新投稿视频json：https://api.bilibili.com/archive_rank/getarchiverankbypartion?tid=17
上面这个是旧接口，更新后接口为：https://api.bilibili.com/x/web-interface/newlist?rid=17
新旧接口都可以用，新接口的[rid]等同于旧接口的[tid]，旧接口输出的是unicode字符，新接口直接输出中文
可选参数[pn]页码 [ps]每页条数 例如：https://api.bilibili.com/x/web-interface/newlist?rid=17&pn=1&ps=20
不带[ps]参数时，旧接口默认输出20条，新接口默认输出50条

获取的数据
* aid AV号
* videos 分P数量
* title 标题
* pubdate 发布时间
* duration 视频长度（秒）

#### 2.获取所有最新投稿视频

https://www.bilibili.com/newlist.html
但是这个接口并不理想，获取到的html需要用xpath解析，这里随便记录下

---

## 热门视频列表

例如游戏/单机游戏tid为17
热门视频json：https://api.bilibili.com/x/web-interface/ranking/region?rid=17
可选参数[day]几天内的 [original]是否原创（0为全部，1为原创）
不带[day]参数默认为3天内，不带[original]参数默认为全部

---

## 番剧视频列表

番剧列表json获取地址：https://bangumi.bilibili.com/web_api/timeline_global
返回的json中键"result"中包含：之前6天、当天、之后6天 共13天内放送的各个番剧数据

获取的数据
* day_of_week 一周中的第几天
* title 番剧名称
* favorites 追番人数
* season_id 番剧id
* ep_id episode_id，最新一集的分集id
