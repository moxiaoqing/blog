---
title: 直播API文档
date: 2019-04-02 18:51:29
tags:
---
### 斗鱼
**获取首页推荐信息**
https://m.douyu.com/api/home/mix
**获取直播分类列表**
https://m.douyu.com/api/cate/list?type=
**根据类型获取直播列表**
https://m.douyu.com/api/room/list?page=1&type=jdqs
**获取直播地址**
https://m.douyu.com/api/room/stream
```bash
请求头
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Origin: https://m.douyu.com
Referer: https://m.douyu.com/288016?type=LOL
User-Agent: Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Mobile Safari/537.36
X-Requested-With: XMLHttpRequest

请求参数
v: 250120190402
did: 28190099feb5ab030e95cd6600021531
tt: 1554202708
sign: 494cc038638dccb9848cafe3aa32613d
ver: 218092701
rid: 288016
```

{"code":0,"data":{"url":"http://hlsa.douyucdn.cn/live/12821rmHNB7Jis_550/playlist.m3u8?wsSecret=39bed54a90bf5990c228a0830691b255\u0026wsTime=1554206145\u0026token=h5-douyu-0-12821-a2abb0e8866c364a64324d03ff4e4505\u0026did=28190099feb5ab030e95cd6600021531","pwd":0}}
