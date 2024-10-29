---
slug: "bilibili-bullets-data-step2"
title: "B站弹幕大数据(2)—弹幕的获取和解析"
keywords: "B站,bilibili,弹幕,大数据"
description: "B站弹幕大数据第二步，普通视频和番剧视频的弹幕获取方式，弹幕格式解析。"
date: 2018-01-02T00:48:36+08:00
draft: false
---

弹幕是通过cid（也就是comment id）获取的，不同类型的视频获取cid的方法不同

## 普通视频通过AV号获取弹幕

aid = av id 也就是 视频id 也就是 AV号
cid = comment id 也就是 弹幕id

通过aid获取cid
http://www.bilibili.com/widget/getPageList?aid=[aid]

通过cid获取弹幕
http://comment.bilibili.com/[cid].xml

以AV7为例：
视频地址 http://www.bilibili.com/video/av7/
获取AID=7的CID http://www.bilibili.com/widget/getPageList?aid=7
获取CID=3625120的弹幕内容 http://comment.bilibili.com/3625120.xml

## 番剧视频通过番剧id和分集id获取弹幕

season_id 也就是 番剧id，按每个番剧的每一季分开
episode_id 也就是 分集id
要获取番剧视频的cid，有两种方式

#### 1.获取season_id下episode_id的cid

http://bangumi.bilibili.com/web_api/get_ep_list?season_id=[season_id]

以番剧844第五话为例：
https://bangumi.bilibili.com/anime/844/play#15185
season_id为844，episode_id为15185

http://bangumi.bilibili.com/web_api/get_ep_list?season_id=844
取得json:

```json
{
    "code": 0,
    "message": "success",
    "result": [
        {
            "avid": 71026,
            "cid": 245281,
            "episode_id": 15190,
            "index": "0"
        },
        {
            "avid": 77011,
            "cid": 245279,
            "episode_id": 15189,
            "index": "1"
        },
        {
            "avid": 80017,
            "cid": 5852080,
            "episode_id": 15188,
            "index": "2"
        },
        {
            "avid": 82593,
            "cid": 139632,
            "episode_id": 15187,
            "index": "3"
        },
        {
            "avid": 85332,
            "cid": 144834,
            "episode_id": 15186,
            "index": "4"
        },
        {
            "avid": 87845,
            "cid": 149498,
            "episode_id": 15185,
            "index": "5"
        },
        {
            "avid": 90597,
            "cid": 151919,
            "episode_id": 15184,
            "index": "6"
        },
        {
            "avid": 92902,
            "cid": 156251,
            "episode_id": 15183,
            "index": "7"
        },
        {
            "avid": 95117,
            "cid": 160880,
            "episode_id": 15182,
            "index": "8"
        },
        {
            "avid": 97185,
            "cid": 161758,
            "episode_id": 15181,
            "index": "9"
        },
        {
            "avid": 99403,
            "cid": 165706,
            "episode_id": 15180,
            "index": "10"
        },
        {
            "avid": 102808,
            "cid": 170654,
            "episode_id": 15179,
            "index": "11"
        },
        {
            "avid": 105403,
            "cid": 174833,
            "episode_id": 15178,
            "index": "12"
        },
        {
            "avid": 107885,
            "cid": 179470,
            "episode_id": 15177,
            "index": "13"
        },
        {
            "avid": 111029,
            "cid": 184807,
            "episode_id": 15176,
            "index": "14"
        },
        {
            "avid": 113813,
            "cid": 189414,
            "episode_id": 15175,
            "index": "15"
        },
        {
            "avid": 116439,
            "cid": 194330,
            "episode_id": 15174,
            "index": "16"
        },
        {
            "avid": 119440,
            "cid": 198458,
            "episode_id": 15173,
            "index": "17"
        },
        {
            "avid": 122500,
            "cid": 203634,
            "episode_id": 15172,
            "index": "18"
        },
        {
            "avid": 125701,
            "cid": 208943,
            "episode_id": 15171,
            "index": "19"
        },
        {
            "avid": 129619,
            "cid": 216698,
            "episode_id": 15170,
            "index": "20"
        },
        {
            "avid": 133373,
            "cid": 222256,
            "episode_id": 15169,
            "index": "21"
        },
        {
            "avid": 136978,
            "cid": 228121,
            "episode_id": 15168,
            "index": "22"
        },
        {
            "avid": 140204,
            "cid": 233023,
            "episode_id": 15167,
            "index": "23"
        },
        {
            "avid": 143012,
            "cid": 237316,
            "episode_id": 15166,
            "index": "24"
        },
        {
            "avid": 145992,
            "cid": 242082,
            "episode_id": 15165,
            "index": "25"
        },
        {
            "avid": 148841,
            "cid": 246675,
            "episode_id": 15164,
            "index": "26"
        }
    ]
}
```

在json中获取episode_id为15185的cid为149498
http://comment.bilibili.com/149498.xml

#### 2.直接获取episode_id的cid（post方式）

http://bangumi.bilibili.com/web_api/get_source

post一个数据"episode_id=15185"
取得json：

```json
{
    "code": 0,
    "message": "success",
    "result": {
        "aid": 87845,
        "cid": 149498,
        "episode_status": 2,
        "pay_user": {
            "status": 0
        },
        "payment": {
            "price": "0"
        },
        "player": "vupload",
        "pre_ad": 0,
        "season_status": 2,
        "vid": "51598974"
    }
}
```

cid为149498
http://comment.bilibili.com/149498.xml

## 弹幕格式

```xml
<d p="[时间],[模式],[字号],[颜色],[发送时间戳],[弹幕池],[用户id的CRC32b加密],[弹幕id]">弹幕内容</d>
```
