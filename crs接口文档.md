---
title: crs接口文档
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


# 通用
## 公共入参出参说明
## 数据字典查询接口**
## 酒店模糊查询
    
**简要描述：** 
- 酒店模糊查询

**请求URL：** 
- ` http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/common/suggestHotel`
  
**请求方式：**
- POST 

**请求参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|hotelName |String|是|酒店名称|
|searchType   |String|是|查询类型|
|cityId   |String|-|城市ID|
|hotelIds   |long数组|根据ID查询时必传|酒店ID|

**返回参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|city_name |String|-|城市名称|
|id   |String|-|酒店ID|
|city_id   |String|-|城市ID|
|hotel_name   |String|-|酒店名称|

**酒店名模糊查询请求示例：**
```json
{
    "hotelName":"oyo",
    "cityId":"550",
    "searchType":"hotelNameSearch"
}
```

**酒店名模糊查询响应示例：**

```json
{
    "code": "000",
    "data": {
        "searchType": "hotelNameSearch",
        "hotels": [
            {
                "city_name": "大兴安岭",
                "id": "36139",
                "city_id": "550",
                "hotel_name": "OYO 8090 新湖主题酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "18208",
                "city_id": "550",
                "hotel_name": "OYO 8009 鹏鑫宾馆"
            },
            {
                "city_name": "大兴安岭",
                "id": "18545",
                "city_id": "550",
                "hotel_name": "OYO 8012 华悦酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "18762",
                "city_id": "550",
                "hotel_name": "OYO 8017 鈺龙湾"
            },
            {
                "city_name": "大兴安岭",
                "id": "36285",
                "city_id": "550",
                "hotel_name": "OYO 8092 新铭都酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "17783",
                "city_id": "550",
                "hotel_name": "OYO 8003 月湾酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "17866",
                "city_id": "550",
                "hotel_name": "OYO 8006 京波酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "36353",
                "city_id": "550",
                "hotel_name": "OYO 8088 深圳柠檬酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "17782",
                "city_id": "550",
                "hotel_name": "OYO 8005 莲之香酒店"
            },
            {
                "city_name": "大兴安岭",
                "id": "1",
                "city_id": "550",
                "hotel_name": "OYO 舒适测试酒店"
            }
        ],
        "displayName": null
    },
    "msg": "成功"
}

```

**酒店id精确查询请求示例：**

```json
{
    "searchType":"hotelIdSearch",
    "hotelIds":[1,17782]
}
```

**酒店名模糊查询响应示例：**

```json
{
    "code": "000",
    "data": {
        "searchType": "hotelIdSearch",
        "hotels": [
            {
                "id": "1",
                "hotel_name": "OYO 舒适测试酒店"
            },
            {
                "id": "17782",
                "hotel_name": "OYO 8005 莲之香酒店"
            }
        ],
        "displayName": null
    },
    "msg": "成功"
}
```

## 城市三级联动接口
    
**简要描述：** 
- 省市区列表

**请求URL：** 
- [http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/common/threeLinkage](http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/common/threeLinkage "http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/common/threeLinkage") 
  
**请求方式：**
- GET 

**请求参数说明：** 

**返回参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|stateId |int|-|省ID|
|stateName   |String|-|省名称|
|cityVOList   |ArrayList|-|市列表|

**cityVOList参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|cityId |int|-|市ID|
|cityName   |String|-|市名称|
|clusterVOList   |ArrayList|-|区列表|

**clusterVOList参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|clusterId |int|-|区ID|
|clusterName   |String|-|区名称|

**响应示例：**

```json
{
    "code": "000",
    "msg": "成功",
    "data": [
        {
            "stateId": 37,
            "stateName": "Guangdong",
            "cityVOList": [
                {
                    "cityId": 550,
                    "cityName": "Shenzhen",
                    "clusterVOList": [
                        {
                            "clusterId": 1306,
                            "clusterName": "Futian"
                        },
                        {
                            "clusterId": 1307,
                            "clusterName": "Yantian"
                        },
                        {
                            "clusterId": 1308,
                            "clusterName": "Longgang"
                        },
                        {
                            "clusterId": 1309,
                            "clusterName": "Nanshan"
                        },
                        {
                            "clusterId": 1310,
                            "clusterName": "Bao'an"
                        },
                        {
                            "clusterId": 1311,
                            "clusterName": "Luohu"
                        },
                        {
                            "clusterId": 1312,
                            "clusterName": "Longhua"
                        },
                        {
                            "clusterId": 1313,
                            "clusterName": "Pingshan"
                        },
                        {
                            "clusterId": 1314,
                            "clusterName": "Dapeng"
                        },
                        {
                            "clusterId": 1315,
                            "clusterName": "Guangming"
                        }
                    ]
                } 
            ]
        }
    ]
}

```


## crs下单渠道列表

    
**简要描述：** 
- crs下单渠道

**请求URL：** 
- `http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/channel/listCrsChannel `
  
**请求方式：**
- GET 

**请求参数说明：** 

**返回参数说明：** 

|参数名|类型|必须|说明|
|:----    |-----   |-----   |-----   |
|channelCode |String|-|渠道Code|
|channelId   |Long|-|渠道ID|
|channelName   |String|-|渠道名称|

**请求示例：** 

- `http://backend-crs-biz-service.dev.ahotels.tech/api/crs/v1/channel/listCrsChannel `

**响应示例：** 
```json
{
    "code": "000",
    "data": [
        {
            "channelCode": "BookingSource_0005",
            "channelId": 5,
            "channelName": "不试了"
        },
        {
            "channelCode": "BookingSource_0015",
            "channelId": 15,
            "channelName": "不试了15"
        },
        {
            "channelCode": "BookingSource_0020",
            "channelId": 20,
            "channelName": "1"
        },
        {
            "channelCode": "BookingSource_0023",
            "channelId": 23,
            "channelName": "OTA渠道"
        },
        {
            "channelCode": "BookingSource_0026",
            "channelId": 26,
            "channelName": "渠道3"
        },
        {
            "channelCode": "BookingSource_0035",
            "channelId": 35,
            "channelName": "不试了1"
        },
        {
            "channelCode": "BookingSource_0037",
            "channelId": 37,
            "channelName": "尊享"
        },
        {
            "channelCode": "BookingSource_0039",
            "channelId": 39,
            "channelName": "尊享1"
        }
    ],
    "msg": "成功"
}
```
# 订单管理
## 
# 酒店管理
# 渠道管理
# 价格策略管理
# 权限管理
# 酒店基本信息