# IOS、Android端查询OTA审核单列表

**简要描述：** 
- IOS、Android端查询OTA审核单列表

**请求URL：** 
- http://ali-dev.ahotels.tech/oyo-supplier-portal/api/v1/OTA/listOtaAuditForm
  
**请求方式：**
- GET

**请求heard：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|client_id |string |是|客户端id(公共参数)|
|client_type  |string |是|客户端类型(公共参数)|
|ticket  |string |是|sso单点jwtticket(公共参数) |

**请求参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|hotelId  |long  |是|酒店Id|
|pageNum |integer |-|请求第几页数据 默认第1页|
|pageSize |Integer|-|一页多少条数据 默认10条|

**返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|code  |string |-|返回码 200 正常返回，-1 通用异常|
|costTime  |integer|-|接口耗时 |
|data  |	HotelOtaAuditFormListResponse|-|数据|
|displayMessage  |string |-|提示信息 |
|errorMessage  |string |-|错误信息 |

**HotelOtaAuditFormListResponse 返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|ContinueToAddNewSigns |boolean |-|是否可以继续新增审批单|
|OtaAuditFormList  |	PageableRespons<HotelOtaAuditFormResponse>|-|数据|

**HotelOtaAuditFormResponse 返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|id |long |-|OTA审批单Id|
|otaAuditFormNo |string |-|OTA审批单申请编号|
|hotelId |long|-|OTA审批单酒店Id|
|auditStatus |integer |-|OTA审批单审批状态|
|auditStatusName |string  |-|OTA审批单审批状态名称|
|auditType |integer |-|OTA审批单审批类型|
|auditTypeName |string  |-|OTA审批单审批类型名称|
|submissionDate |string(yyyy-MM-dd HH:mm:ss)  |-|OTA审批单提交日期|
|auditDate |string(yyyy-MM-dd HH:mm:ss)  |-|OTA审批单审核日期|
|submissionRemark |string  |-|OTA审批单提交备注|
|auditRemark |string |-|OTA审批单审批备注|
|submitterId |long  |-|OTA审批单提交人Id|
|submitterEmail |String |-|OTA审批单提交人邮箱|
|auditorId |long  |-|OTA审批单审批人Id|
|auditorEmail |string |-|OTA审批单审批人邮箱|
|createDate |string(yyyy-MM-dd HH:mm:ss)   |-|OTA审批单创建日期|
|modifyDate |string(yyyy-MM-dd HH:mm:ss)  |-|OTA审批单修改日期|
|OtaInfoList |List<HotelOtaInfoResponse>  |-|OTA审批单OTA信息列表|

**HotelOtaInfoResponse 返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|id |long |-|OTAId|
|otaPlatform |long |-|OTA平台id|
|otaPlatformName |string|-|OTA平台名称|
|otaAccount |string |-|OTA平台账号|
|otaAccountPwd |string  |-|OTA平台密码|
|otaAuditStatus |integer |-|OTA审核状态|
|otaAuditStatusName |string  |-|OTA审核状态名称|

 **返回示例：**
```
{
    "code": "200",
    "displayMessage": null,
    "errorMessage": null,
    "costTime": null,
	"data":{
	"ContinueToAddNewSigns":true,
	"HotelOtaAuditFormResponse": {
        "currentPage": 1,
        "hasNext": false,
        "totalPage": 1,
        "list": [],
        "totalRecord": 3,
        "pageSize": 10,
        "currentPageSize": 3
    	}
	}
}

```


# IOS、Android端保存OTA审核单

**简要描述：** 
- IOS、Android端保存OTA审核单

**请求URL：** 
- http://ali-dev.ahotels.tech/oyo-supplier-portal/api/v1/OTA/saveOtaAuditForm
  
**请求方式：**
- GET

**请求heard：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|client_id |string |是|客户端id(公共参数)|
|client_type  |string |是|客户端类型(公共参数)|
|ticket  |string |是|sso单点jwtticket(公共参数) |

**请求参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|hotelId  |long  |是|酒店Id|
|pageNum |integer |-|请求第几页数据 默认第1页|
|pageSize |Integer|-|一页多少条数据 默认10条|

**返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|code  |string |-|返回码 200 正常返回，-1 通用异常|
|costTime  |integer|-|接口耗时 |
|data  |	Boolean|-|数据|
|displayMessage  |string |-|提示信息 |
|errorMessage  |string |-|错误信息 |


 **返回示例：**
```
{
    "code": "200",
    "displayMessage": null,
    "errorMessage": null,
    "costTime": null,
	"data":true
}

```


# IOS、Android端提交申请OTA审核单

**简要描述：** 
- IOS、Android端提交申请OTA审核单

**请求URL：** 
- http://ali-dev.ahotels.tech/oyo-supplier-portal/api/v1/OTA/submitOtaAuditForm
  
**请求方式：**
- GET

**请求heard：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|client_id |string |是|客户端id(公共参数)|
|client_type  |string |是|客户端类型(公共参数)|
|ticket  |string |是|sso单点jwtticket(公共参数) |

**请求参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|hotelId  |long  |是|酒店Id|

**返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|code  |string |-|返回码 200 正常返回，-1 通用异常|
|costTime  |integer|-|接口耗时 |
|data  |	Boolean|-|数据|
|displayMessage  |string |-|提示信息 |
|errorMessage  |string |-|错误信息 |


 **返回示例：**
```
{
    "code": "200",
    "displayMessage": null,
    "errorMessage": null,
    "costTime": null,
	"data":true
}

```


# IOS、Android端撤回OTA审核单

**简要描述：** 
- IOS、Android端撤回OTA审核单

**请求URL：** 
- http://ali-dev.ahotels.tech/oyo-supplier-portal/api/v1/OTA/withdrawalOtaAuditForm
  
**请求方式：**
- GET

**请求heard：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|client_id |string |是|客户端id(公共参数)|
|client_type  |string |是|客户端类型(公共参数)|
|ticket  |string |是|sso单点jwtticket(公共参数) |

**请求参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|hotelId  |long  |是|酒店Id|

**返回参数：** 

|参数名|类型|必选|说明|
|-|-|-|-|
|code  |string |-|返回码 200 正常返回，-1 通用异常|
|costTime  |integer|-|接口耗时 |
|data  |	Boolean|-|数据|
|displayMessage  |string |-|提示信息 |
|errorMessage  |string |-|错误信息 |


 **返回示例：**
```
{
    "code": "200",
    "displayMessage": null,
    "errorMessage": null,
    "costTime": null,
	"data":true
}

```
