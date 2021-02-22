#                         Pilot 控制协议

Pilot控制协议用于手机或PC应用在局域网内控制Pilot系列相机(Pilot Era/One/OneEE)，它基于OSC协议做了定制化修改。在搜索相机时使用了Udp。

我们的手机App [Pilot Go](https://www.labpano.com/support/download#PilotGo)  就是使用了本协议控制Pilot相机。

如果相机已熄屏，无法被搜索到。

连接到相机之后，相机UI会提示 “手机控制中...”。

本协议适用于Pilot OS  5.12.0 及以上版本。



## 1 简介

### 请求HOST

http://x.x.x.x.x:8080       

x.x.x.x是当前Pilot相机的IP，8080是相机监听的端口。



### 协议

- Udp + json： 仅在搜索相机时使用


- Http + json



### http接口调用说明

**在使用Http请求时需要添加请求头**

请求头 `Content-Type`  设置成 `application/x-www-form-urlencoded`

请求头`version`，按照接口说明设置相应的version值。

执行命令用Post请求，下载文件用Get请求。

http超时时间建议15s。



请求地址主要有以下几个：

|   用途   | 请求地址                                     |
| :----: | ---------------------------------------- |
|  执行命令  | http://x.x.x.x.x:8080/osc/commands/execute |
| 查询命令进度 | http://x.x.x.x.x:8080/osc/commands/status |
|  下载文件  | http://x.x.x.x.x:8080/osc/getFile        |



### 接口调用示例

第2章会列举每一个接口，这里仅以连接相机接口为例，介绍一下请求过程。



#### 请求

Http请求的通用json格式如下：

```json
{
    "name": "interface name",								//接口名, string类型
    "parameters": "interface input param" 	//接口输入参数, string类型
}
```



**调用方式：**  http+json,Post传参

**请求地址：**  ​http://x.x.x.x.x:8080/osc/commands/execute

**请求头version：**3.1

连接相机接口的**名字**为：camera.startSession

连接相机接口的**输入参数**为：

```json
{
    "parameters" :     {
        "alone" : true,
        "sessionId" : "53c61259-373e-4eed-975c-5d5500be176b",
        "timeout" : 1800
    }
}
```



把接口的**输入参数**转为字符串格式，与接口名组合成json，发起http请求。

```json
{
    "name": "camera.startSession",	//接口名, string类型
    "parameters": "{"parameters" : {"alone" : true,"sessionId" : "53c61259-373e-4eed-975c-5d5500be176b","timeout" : 1800}}"	 //接口输入参数, string类型
  
}
```



#### 返回

发起请求后，相机会返回一个json。数据结构如下：

|     字段     |   类型   | 描述                                       |
| :--------: | :----: | ---------------------------------------- |
|     id     | string | 接口唯一id,  name与id相同                       |
|    name    | string | 请求接口名称                                   |
|   state    | string | 有3种状态： `done`     `inProgress`      `error` |
|  results   | json节点 | 返回结果。不同接口内容不同。                           |
| parameters | json节点 | 输入参数。不同接口内容不同。                           |
|   error    | json节点 | 错误详情信息。                                  |
|    code    | string | 错误代码                                     |
|  message   | string | 错误详情                                     |




以连接相机接口为例，相机返回的json如下：

```json
{
	"id": "camera.startSession",
	"name": "camera.startSession",
	"results": {
		"channel": "standard",
		"exposureType": 0,
		"firmwareVersion": "5.13.0",
		"sessionId": "1bb27add-04e5-4458-8147-3f57b3bb5e17",
		"status": false,
		"timeout": 1800
	},
	"state": "done"
}
```




注意：如果state值为`inProgress`，需要调用**查询进度**接口，查询命令是否执行完成。



## 2 各个接口



### 搜索设备

**功能：** 与Pilot通信之前，首先要在同一个局域网内搜索到Pilot相机，通过此接口找到可用的Pilot相机。相机Udp端口号为10000。

**调用方式：**  	Udp+json

**输入参数：**

|    字段     |   类型   | 描述                        |
| :-------: | :----: | ------------------------- |
|   port    |  int   | 手机端的端口号(10000)            |
| sessionId | string | 上次连接的sessionID, 如果没连接过则传空 |
|  version  | string | 3.1                       |

**Udp参数格式：**

```json
{
    "parameters" :     {
        "port" : 10000,
        "sessionId" : "559d26f2-081f-43f5-8f93-35788a9321b4",
        "version" : "3.1"
    }
}
```

**输出参数：**   

| 字段               | 字段类型    | 描述      |
| ---------------- | ------- | ------- |
| ip               | string  | 相机端ip   |
| port             | int     | 相机端端口   |
| deviceName       | string  | 相机名称    |
| alreadyConnected | boolean | 相机是否被连接 |

**返回内容：**

```json
{
  	"parameters": {
		"ip": "192.168.4.85",
		"port": 10000
	},
	"results": {
		"ip": "",
		"port": 0,
		"deviceName": "",
		"alreadyConnected": false
	},
}
```





###  连接相机

**功能：** 连接Pilot相机

**调用方式：**  http+json,Post传参

**请求地址：**  ​	http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  	camera.startSession

**请求头参数(version)：**3.1

**输入参数：**

| 字段        | 字段类型    | 描述                                       |
| --------- | ------- | ---------------------------------------- |
| timeout   | int     | 连接超时时间，单位ms                              |
| sessionId | string  | 会话id                                     |
| alone     | boolean | 心跳超时后，会话是否需要保持。true：心跳超时，（相机正在录像、直播、漫游时）会话保持，即不停止相机当前的工作；false：心跳超时，关闭会话，相机停止工作。 |
| language  | String  | Pilot相机系统的语言。  英文:en ,中文:zh              |

**输出参数：**   

| 字段              | 字段类型    | 描述                                       |
| --------------- | ------- | ---------------------------------------- |
| sessionId       | string  | 会话id                                     |
| timeout         | int     | 连接超时时间，单位ms                              |
| channel         | string  | 渠道名，用于表示固件类型。Era为`standard`，One和OneEE为`one_standard`。后续可能增加其它的渠道号。 |
| mode            | string  | 当前相机所在的拍摄模式。  `photo`代表拍照；  `video`代表实时拼接录像；  `video_feye`代表未拼接录像；  `video_street_view`代表谷歌街景录像模式；  `video_time_lapse`代表延时摄影； `photo_tours`代表PilotTour |
| status          | boolean | 相机工作状态，`true`代表正在录像或者拍照工作中；`false`代表空闲状态 |
| firmwareVersion | string  | Pilot OS 版本号                             |
| exposureType    | int     | 曝光模式类型：0代表自动模式，1代表手动模式                   |
| modeWay         | String  | 用于隐藏拍摄者。其他模式不支持此字段                       |

**返回内容：**

```json
{
	"id": "camera.startSession",
	"name": "camera.startSession",
	"results": {
		"channel": "standard",
		"exposureType": 0,
		"firmwareVersion": "5.13.0",
		"sessionId": "1bb27add-04e5-4458-8147-3f57b3bb5e17",
		"status": false,
		"timeout": 1800
	},
	"state": "done"
}
```





###  建立心跳

**功能：** 相机与手机建立连接成功之后，手机请求心跳，相机端收到之后返回给手机端，手机端收到之后再回传，一直循环。

**调用方式：**  http+json,Post传参

**请求地址：**  ​	http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getHeart

**请求头参数(version)：** 4.7

**输入参数：**无

**输出参数：**   

| 字段              | 字段类型    | 描述                                       |
| --------------- | ------- | ---------------------------------------- |
| battery         | string  | 电池电量                                     |
| latitude        | double  | 纬度                                       |
| longitude       | double  | 经度                                       |
| lowMemory       | boolean | 存储状态。true表示低存储；false表示存储充足               |
| pushStream      | boolean | 推流状态。true表示正在推流中；false没有推流               |
| tick            | int     | 拍摄倒计时，相机实时返回。进行“拍摄隐藏拍摄者模式“拍照时，tick总时长为 “第一张图片倒计时” +“第一张图片倒计时“ |
| videoTime       | string  | 录像时长                                     |
| tourPrepare     | int     | PilotTour定位状态。0：PilotTour未定位；1：PilotTour已定位。 |
| roamStateDetail | int     | 新版PilotTour状态详情  ：  0-未初始化 1-正在初始化 2-正在跟踪 3-丢失跟踪（相机5.6.0以上使用） |
| checkTime       | boolean | 延时摄影模式下， 录制时间是否过短。false时，停止录像，不会生成录像文件   |
| sdCritical      | long    | 储存空间临界值                                  |
| sdTotalSize     | Long    | 储存空间总大小                                  |
| sdAvailableSize | Long    | 剩余可用储存空间                                 |
| temperature     | int     | 机器温度(适用于Pilot One户外版)，温度大于80℃时，提示相机Cpu温度过高 |
| taskId          | String  | 当前相机进行中的（首选）任务id，目前仅支持拍照任务               |
| code            | Int     | 直播错误码:1 - 正在直播；0 - 未开始直播； -3006 - 可重试直播(camera._restartLive)；其他错误码：调用停止直播(camera._setLiveStop) |

**返回内容：**

```json
{
	"id": "camera._getHeart",
	"name": "camera._getHeart",
	"results": {
		"battery": "97",
		"latitude": 0.0,
		"longitude": 0.0,
		"lowMemory": false,
		"pushStream": false,
		"tick": 0,
		"videoTime": "00:00:00"
	},
	"state": "done"
}
```





###  断开连接

**功能：** 关闭与相机的连接。

**调用方式：**  http+json,Post传参

**请求地址：**  ​	http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.closeSession

**请求头参数(version)：** 3.0

**输入参数：**

| 字段        | 字段类型   | 描述     |
| --------- | ------ | ------ |
| sessionId | string | 连接唯一标识 |

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera.closeSession",
	"name": "camera.closeSession",
	"state": "done"
}
```





###  开启预览

**功能：** 开启相机预览。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._startPreview

**请求头参数(version)：** 3.0

**输入参数：**

| 字段       | 字段类型 | 描述                  |
| -------- | ---- | ------------------- |
| targetId | int  | 相机预览: `1` ；直播预览：`3` |

**输出参数：**   需要调用**查询进度**接口获取结果

**返回内容：**

```json
{
	"id": "camera._startPreview",
	"name": "camera._startPreview",
	"state": "inProgress"
} 
```





###  关闭预览

**功能：** 关闭相机的预览。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._stopPreview

**请求头参数(version)：** 3.0

**输入参数：**   无

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._stopPreview ",
	"name": "camera._stopPreview ",
	"state": "done"
}
```





###  拍摄-切换模式

**功能：** 切换相机工作的模式(如图片、未拼接视频、实时拼接视频、街景、PilotTour、延时摄影等模式)

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._changeResolutionEx

**请求头参数(version)：** 3.0

**输入参数：**

| 字段   | 字段类型   | 描述                                       |
| ---- | ------ | ---------------------------------------- |
| type | string | 相机工作类型  `photo`代表拍照模式；  `video`代表实时拼接录像模式；  `video_feye`代表未拼接录像模式 ； `video_street_view`代表谷歌街景录像；   `video_time_lapse`代表延时摄影； `photo_tours`代表PilotTour； |

**输出参数：**   需要调用**查询进度**接口获取结果

**返回内容：**

```json
{
	"id": "camera._changeResolutionEx",
	"name": "camera._changeResolutionEx",
	"state": "inProgress"
}
```





###  查询进度

**功能：** 相机执行某个命令后需要一段时间才能完成。此接口用于查询命令的进度。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/status

**命令ID：**  要查询的接口id

| 字段   | 字段类型   | 描述       |
| ---- | ------ | -------- |
| id   | string | **接口ID** |

**请求头参数(version)：** 3.0

**输入参数：** 无

**输出参数：** 不同的命令输出不同的result。

**返回内容：** 以拍照查询结果为例

```json
{
	"id": "camera.takePicture",
	"name": "camera.takePicture",
	"results": {
		"dateTimeZone":"2018-09-2214:15:38",
    	"fileUrl":"/osc/getFile?/storage/emulated/0/DCIM/Pisoft/180922_141536982.jpg",	
		"height": 3072,
		"isProcessed": false,
		"lat": 0.0,
		"lng": 0.0,
		"name": "180922_141536982.jpg",
		"size": 7111125,
		"stitch": false,
		"thumbnail":"/osc/getFile?/storage/emulated/0/DCIM/Thumb/180922_141536982.jpg",
		"width": 6144
	},
	"state": "done"
}
```





###  拍摄-拍照

**功能：** 调用相机拍照。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.takePicture

**请求头参数(version)：** 3.0

**输入参数：**无

**输出参数**：处于inProgress时(目前只用于隐藏拍摄者模式)

| 字段      | 类型   | 说明                       |
| :------ | :--- | :----------------------- |
| _pIndex | int  | 拍照索引。0代表第一张，1代表第二张       |
| _tick   | int  | 剩余倒计时(当_tick值为0时表示正在拍照中) |

**输出参数：**   需要调用**查询进度**接口获取结果

| 字段           | 字段类型    | 描述                        |
| ------------ | ------- | ------------------------- |
| dateTimeZone | string  | 拍照时间(2018-09-22 14:15:38) |
| fileUrl      | string  | 文件地址                      |
| width        | int     | 照片宽                       |
| height       | int     | 照片高                       |
| isProcessed  | boolean | —                         |
| lat          | double  | —                         |
| lng          | double  | —                         |
| name         | string  | 文件名称                      |
| size         | int     | 文件大小                      |
| stitch       | boolean | 是否拼接，boolean类型            |
| thumbnail    | string  | 缩略图地址                     |

**返回内容：**

```json
{
	"id": "camera.takePicture",
	"name": "camera.takePicture",
	"results": {
		"dateTimeZone":"2018-09-2214:15:38",
		"fileUrl": "/osc/getFile?/storage/emulated/0/DCIM/Pisoft/180922_141536982.jpg",
		"height": 3072,
		"isProcessed": false,
		"lat": 0.0,
		"lng": 0.0,
		"name": "180922_141536982.jpg",
		"size": 7111125,
		"stitch": false,
		"thumbnail":"/osc/getFile?/storage/emulated/0/DCIM/Thumb/180922_141536982.jpg",
		"width": 6144
	},
	"state": "done"
}
```





###  拍摄-开始录像

**功能：** 调用相机录像。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.startCapture

**请求头参数(version)：** 3.0

**输入参数：**无

**输出参数：**   需要调用**查询进度**接口获取结果

**返回内容：**

```json
{
	"id": "camera.startCapture",
	"name": "camera.startCapture",
	"state": "inProgress"
}
```





###  拍摄-停止录像

**功能：** 停止录像。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.stopCapture

**请求头参数(version)：** 3.0

**输入参数：**无

**输出参数：**   需要调用**查询进度**接口获取结果

| 字段           | 字段类型    | 描述                                    |
| ------------ | ------- | ------------------------------------- |
| dateTimeZone | string  | 拍摄时间                                  |
| fileUrl      | string  | 文件地址                                  |
| width        | int     | 分辨率-宽                                 |
| height       | int     | 分辨率-高                                 |
| fps          | string  | 帧率                                    |
| lat          | double  | 纬度                                    |
| lng          | double  | 经度                                    |
| name         | string  | 文件名称                                  |
| size         | int     | 文件大小                                  |
| stitch       | boolean | 是否拼接，boolean类型  true表示已拼接  false表示未拼接 |
| thumbnail    | string  | 缩略图地址                                 |
| codec        | string  | 编码类型(H.264/265 , AAC)                 |

**返回内容：**

```json
{
	"id": "camera.stopCapture",
	"name": "camera.stopCapture",
	"results": {
		"codec": "H.264 , AAC",
		"dateTimeZone": "20190314T112118.000Z",
		"duration": "03:28",
		"fileUrl": "/osc/getFile?/sdcard/DCIM/Videos/Unstitched/190314_191746118.sti",
		"fps": "24",
		"height": 2200,
		"isProcessed": false,
		"lat": 0.0,
		"lng": 0.0,
		"name": "190314_191746118.sti",
		"size": 3231933437,
		"stitch": false,
		"thumbnail": "/osc/getFile?/storage/emulated/0/DCIM/Thumbs/190314_191746118.jpg",
		"width": 3520
	},
	"state": "done"
}
```





###  下载文件

**功能：** 下载图库中的某个文件

**调用方式：**  http+json,Get传参

**请求地址：**  <http://x.x.x.x.x:8080/osc/getFile>?/storage/emulated/0/DCIM/Thumb/180922_103443479.jpg

**输入参数：**无

**输出参数：**文件流

**返回内容：**无



###  获取图库文件列表

**功能：** 获取相机图库文件列表

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.listFiles

**请求头参数(version)：** 3.0

**输入参数：**

| 字段            | 字段类型   | 描述                                       |
| ------------- | ------ | ---------------------------------------- |
| fileType      | string | 文件类型  1) image:图片类型  2) video:视频类型  3) all:图片与视频 |
| startPosition | int    | 0                                        |
| entryCount    | int    | 1                                        |
| maxSize       | int    | 5                                        |
| typeVersion   | String | 必须传：`tours`                              |

**输出参数：**   需要调用**查询进度**接口获取结果

| 字段           | 字段类型 | 描述     |
| ------------ | ---- | ------ |
| totalEntries | int  | 文件总数   |
| entries      |      | json节点 |

**返回内容：**

```json
{
	"id": "camera.listFiles",
	"name": "camera.listFiles",
	"results": {
		"entries": [{
			"dateTimeZone": "2018-09-22 14:15:38",
			"fileUrl":         "/osc/getFile?/storage/emulated/0/DCIM/Pisoft/180922_141536982.jpg",
			"height": 3072,
			"isProcessed": false,
			"lat": 0.0,
			"lng": 0.0,
			"name": "180922_141536982.jpg",
			"size": 7111125,
			"stitch": false,
			"thumbnail": "/osc/getFile?/storage/emulated/0/DCIM/Thumb/180922_141536982.jpg",
			"width": 6144
		}],
		"totalEntries": 5
	},
	"state": "done"
}
```









###  删除文件

**功能：** 删除相机照片、视频文件。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.delete

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段       | 字段类型 | 描述                                       |
| -------- | ---- | ---------------------------------------- |
| fileList | List | fileUrls 文件名:String    isTour 是否是PilotTour文件：boolean |

格式

```Json
{
    "fileList" :[
                {
            "fileUrls" : "testPhoto.jpg",
            "isTour" : 0
        },
                {
            "fileUrls" : "testPhoto.jpg",
            "isTour" : 0
        },
                {
            "fileUrls" : "testPhoto.jpg",
            "isTour" : 0
        }
    ]
}
{"parameters" : {"fileList" : [{"fileUrls" : "201026_105149651.mp4","isTour" : false}]}}
```

**输出参数：**   无

返回删除成功:

```json
{
	"id": "camera.delete",
	"name": "camera.delete",
	"results": [],
	"state": "done"
}
```

返回删除失败

```json
{
	"id": "camera.delete",
	"name": "camera.delete",
	"results": ["190314_194559664.jpg", "190314_194553689.jpg"],
	"state": "done"
}
```





###  拍摄-设置倒计时开关

**功能：** 设置是否打开倒计时

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setDelayOpen

**请求头参数(version)：** 3.0

**输入参数：**

| 字段     | 字段类型    | 描述                                       |
| ------ | ------- | ---------------------------------------- |
| type   | string  | 设置类型`photo`代表拍照模式  `video`代表实时拼接模式  `video_feye`代表鱼眼模式  `video_street_view`代表街景视频  `video_time_lapse`代表延时摄影   `photo_tours`代表PilotTour拍摄 |
| isOpen | boolean | 倒计时开关是否打开true 打开   false关闭               |

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._setDelayOpen ",
	"name": "camera._setDelayOpen ",
	"state": "done"
}
```





###  拍摄-设置倒计时

**功能：** 设置倒计时时间

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setDelay

**请求头参数(version)：** 3.0

**输入参数：**

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| type  | string | 类型`photo`代表拍照模式  `video`代表实时拼接模式  `video_feye`代表鱼眼模式  `video_street_view`代表街景视频  `video_time_lapse`代表延时摄影    `photo_tours`代表PilotTour拍摄 |
| delay | int    | 参数3、5、10、15                              |

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._setDelay",
	"name": "camera._setDelay",
	"state": "done"
}
```





###  拍摄-获取倒计时

**功能：** 获取倒计时配置

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getDelay

**请求头参数(version)：** 3.0

**输入参数：**无

**输出参数：**   

| 字段                     | 字段类型 | 描述                 |
| ---------------------- | ---- | ------------------ |
| delayphoto             | int  | 照片倒计时时间            |
| delayvideo             | int  | 实时拼接视频倒计时时间        |
| delayvideo_feye        | int  | 未拼接视频倒计时时间         |
| delayvideo_street_view | int  | 街景倒计时时间            |
| delayvideo_time_lapse  | int  | 延时拍摄倒计时时间          |
| delaytours             | int  | PilotTour倒计时时间(新增) |

**返回内容：**

```json
{
	"id": "camera._getDelay",
	"name": "camera._getDelay",
	"results": {
		"delayphoto": 5,
		"delayvideo": 0,
		"delayvideo_feye": 0,
		"delayvideo_street_view": 3,
		"delayvideo_time_lapse":0,
		"delaytours":0
	},
	"state": "done"
}
```





###  拍摄-设置分辨率

**功能：** 设置分辨率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setResolution

**请求头参数(version)：** 3.0

**输入参数：**

| 字段         | 字段类型   | 描述                                       |
| ---------- | ------ | ---------------------------------------- |
| type       | string | 设置分辨率类型`photo`代表拍照模式  `video`代表实时拼接模式   `video_feye`代表鱼眼模式   `video_street_view`代表街景视频  `video_time_lapse`代表延时摄影  `photo_tours`代表PilotTour拍摄 |
| resolution | string | 分辨率字符串。   `photo`和`photo_tours`对应分辨率：`8192*4096` `6144*3072` `4096*2048` `3072*1536` `2048*1024`； `video`对应分辨率：`7680*3840` `5760*2880` `3840*1920` `1920*960`； `video_feye`对应分辨率：`7680*3840` `7040*3520` `5760*2880` `3840*1920`； `video_time_lapse`对应分辨率：`7680*3840` `5760*2880` `3840*1920` `1920*960` |

**输出参数：**  需要调用**查询进度**接口获取结果

**返回内容：**

```json
{
	"id": "camera._setResolution",
	"name": "camera._setResolution",
	"state": "inProgress"
}
```





### 拍摄-获取分辨率

**功能：** 获取相机分辨率配置

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getResolution

**请求头参数(version)：** 3.0

**输入参数：**无

**输出参数：**

| 字段                         | 字段类型   | 描述           |
| -------------------------- | ------ | ------------ |
| resolutionphoto            | string | 图片分辨率        |
| resolutionvideo            | string | 实时视频类型分辨率    |
| resolutionvideo_feye       | string | 未拼接视频类型分辨率   |
| resolutionvideo_lapse_time | string | 延时摄影类型分辨率    |
| resolutiontours            | String | PilotTour分辨率 |

**返回内容：**

```json
{
	"id": "camera._getResolution",
	"name": "camera._getResolution",
	"results": {
		"resolutionphoto": "6144*3072",
		"resolutionvideo": "1920*960",
		"resolutionvideo_feye": "7040*3520",
    	"resolutionvideo_lapse_time":"7680*3840"
	},
	"state": "done"
}
```





###  拍摄-设置专业选项

**功能：** 设置相机专业设置参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setProfessionalEx

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| type  | string | 专业设置值类型。 iso 感光度；ev 曝光补偿； distance 拼接距离； exposure 曝光时间；wb白平衡 |
| value | int    | 对应不同专业设置，设置相应类型的值。iso 0到6；  ev -4 到 4； distance 0 到 100； exposure 0到6； wb 0到4 |
| proId | int    | 图片、未拼接、实时、街景类型不同类型。  0图片  1未拼接  2实时拼接  3街景  4延时摄影 5PilotTour |
| mode  | int    | pro模式类型 1表示手动模式   0表示自动模式                |



**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._setProfessionalEx",
	"name": "camera._setProfessionalEx",
	"state": "done"
}
```





###  拍摄-获取专业选项

**功能：** 获取相机专业设置参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getProfessionalEx

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段    | 字段类型 | 描述                                       |
| ----- | ---- | ---------------------------------------- |
| proId | int  | 图片、未拼接、实时、街景类型不同类型  0图片  1未拼接  2实时拼接 3街景 4延时摄影 5 PilotTour |



**输出参数：**

| 字段           | 字段类型 | 描述                                       |
| ------------ | ---- | ---------------------------------------- |
| distance     | int  | 拼接距离                                     |
| ev           | int  | 曝光补偿                                     |
| exposure     | int  | 曝光时间                                     |
| exposureType | int  | 曝光类型0自动模式1手动模式                           |
| iso          | int  | 感光度                                      |
| manualISO    | int  | 手动模式iso值                                 |
| proId        | int  | 图片、未拼接、实时、街景类型不同类型0图片  1未拼接  2实时拼接  3街景 4延时摄影 5 PilotTour |



**返回内容：**

```json
{
	"id": "camera._getProfessionalEx",
	"name": "camera._getProfessionalEx",
	"results": {
		"distance": 100,
		"ev": 0,
		"exposure": 1,
		"exposureType": 1,
		"iso": 0,
		"manualISO": 3,
		"proId": 0
	},
	"state": "done"
}
```





###  拍摄-设置HDR模式

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setPhotoHdrState

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段       | 字段类型 | 描述                     |
| -------- | ---- | ---------------------- |
| hdrState | int  | 0: auto 1:晴天 2:阴天 3:室内 |


**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setPhotoHdrState",
	"name": "camera._setPhotoHdrState",
	"state": "done"
}
```

 



###  拍摄-设置拍照参数

**功能：** 设置拍照参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setPhotoArgs

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| type  | string | 拍照参数类型 `hdr`是否打开hdr  `gyroscope`是否开启画面稳定  `flow`是否开启光流 |
| mode  | int    | 0:拍照 1:未拼接录像  2:实时拼接录像  3:谷歌街景录像  4:延时摄影 5:PilotTour |
| value | bool   |                                          |

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._setPhotoArgs",
	"name": "camera._setPhotoArgs",
	"state": "done"
}
```





###  拍摄-获取拍照参数

**功能：** 获取拍照参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getPhotoArgs

**请求头参数(version)：** 5.0.0

**输入参数：**无

**输出参数：**

| 字段        | 字段类型    | 描述       |
| --------- | ------- | -------- |
| gyroscope | boolean | 是否开启画面稳定 |
| flow      | boolean | 是否开启光流   |
| hdr       | boolean | 是否开启hdr  |
| hdrState  | int     | HDR 选中模式 |


**返回内容：**

```json
{
	"id": "camera._getPhotoArgs",
	"name": "camera._getPhotoArgs",
	"results": {
		"gyroscope": true,
		"flow": false,
		"hdr": true
	},
	"state": "done"
}
```



###  拍摄-获取街景视频帧率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getGoogleRateList

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段   | 字段类型   | 描述                        |
| ---- | ------ | ------------------------- |
| type | String | `video_street_view`代表街景视频 |

**输出参数：**

| 字段             | 字段类型     | 描述     | 标准参数                       |
| -------------- | -------- | ------ | -------------------------- |
| googleratelist | String[] | 谷歌帧率列表 | `[“1fps”, “2fps”, “7fps”]` |
| rate           | String   | 选中的帧率  | `1fps`                     |

**返回内容：**

```json
{
	"id": "camera._getGoogleRateList ",
	"name": "camera._getGoogleRateList ",
	"results": {
		"googleratelist":["1fps","2fps","7fps"], 
		"rate":"1fps"
	},
	"state": "done"
}
```





###  拍摄-设置街景视频帧率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setGoogleRate

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段   | 字段类型   | 描述                        |
| ---- | ------ | ------------------------- |
| type | String | `video_street_view`代表街景视频 |
| rate | String | 如：`7fps`                  |

**输出参数：** 无

**返回内容：**

```json
{
	"id": "camera._setGoogleRate",
	"name": "camera._setGoogleRate",
	"state": "done"
}
```





###  拍摄-获取PilotSteady

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getOrientationState

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段   | 字段类型   | 描述                                       |
| ---- | ------ | ---------------------------------------- |
| type | String | `video`代表实时拼接模式  `video_feye`代表鱼眼模式   `video_time_lapse`代表延时摄影 |

**输出参数：**

| 字段          | 字段类型    | 描述                       |
| ----------- | ------- | ------------------------ |
| orientation | boolean | `false` 固定朝向；`true` 跟随相机 |

**返回内容：**

```json
{
	"id": "camera._getOrientationState ",
	"name": "camera._getOrientationState ",
	"results":{
       "orientation": false
	},
	"state": "done"
}
```





###  拍摄-设置PilotSteady

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setOrientationState

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段          | 字段类型    | 描述                                       |
| ----------- | ------- | ---------------------------------------- |
| type        | String  | `video`代表实时拼接模式  `video_feye`代表鱼眼模式    `video_time_lapse`代表延时摄影 |
| orientation | boolean | false: 固定  true  跟随相机朝向                  |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setOrientationState",
	"name": "camera._setOrientationState",
	"state": "done"
}
```





###  拍摄-获取缩时倍数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getTimeLapseTimeList

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段   | 字段类型   | 描述                       |
| ---- | ------ | ------------------------ |
| type | String | “video_time_lapse”代表延时摄影 |

**输出参数：**

| 字段                | 字段类型     | 引入版本   | 描述      | 标准参数                                     |
| ----------------- | -------- | ------ | ------- | ---------------------------------------- |
| timeLapseTimeList | String[] | 4.12.0 | 缩时倍数列表  | `[“20x  (0.6spf)”,” 50x  (1.7spf)”,…….,” 10000x  (5.6mpf)”]` |
| timeLapseTimes    | String   | 4.12.0 | 选中的缩时倍数 | `20x  (0.6spf)`                          |

**返回内容：**

```json
{
	"id": "camera._getTimeLapseTimeList ",
	"name": "camera._getTimeLapseTimeList ",
	"results": {
		"timeLapseTimeList": ["20x  (0.6spf)"," 50x  (1.7spf)",…….," 10000x  (5.6mpf)"], 
		"timeLapseTimes":" 20x  (0.6spf)"
	},
	"state": "done"
}
```





###  拍摄-设置缩时倍数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLapseTime

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段             | 字段类型   | 描述                       |
| -------------- | ------ | ------------------------ |
| type           | String | “video_time_lapse”代表延时摄影 |
| timeLapseTimes | String | 20x  (0.6spf)            |

**输出参数：**   无

**返回内容：**

```json
{
	"id": "camera._setLapseTime",
	"name": "camera._setLapseTime",
	"state": "done"
}
```



###  拍摄-开始PilotTour

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._startTour

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段       | 字段类型   | 描述                           |
| -------- | ------ | ---------------------------- |
| tourName | String | PilotTour名字 空串、null、不传 将返回错误 |

**返回内容：**

```Json
{
	"id": " camera._startTour",
	"name": " camera._startTour",
	"state": "done"
}
```





###  拍摄-结束PilotTour

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._stopTour

**请求头参数(version)：** 5.0.0

**输入参数：**

**返回内容：**

```json
{
	"id": " camera._stopTour",
	"name": " camera._stopTour",
	"state": "done"
}
```





###  拍摄-获取设置选项

**功能：** 5.2.0 版本增加，获取Camera选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.getOptions

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段          | 字段类型         | 描述               |
| ----------- | ------------ | ---------------- |
| optionNames | String Array | 选项数组。参考下方表格中选项名。 |

**输出参数：**

| 字段      | 字段类型                | 描述              |
| ------- | ------------------- | --------------- |
| Options | map<String, Object> | 参考表格中选项名称及选项类型。 |

**返回内容：**

**选项列表**

| 选项名称                                     | 类型                          | 描述                                       |
| ---------------------------------------- | --------------------------- | ---------------------------------------- |
| `_camera$roam$height`                    | String                      | PilotTour相机高度。浮点数值，取值范围[0,1000]，不在此范围内的设置值会处理为边界值。 |
| `_camera$video$encode`                   | String                      | 实时拼接视频编码，默认H.264。                        |
| `_camera$video$encodeSupport`            | String Array                | 实时拼接视频支持的编码，只读。当前为`[“H.264”,“H.265”]`    |
| `_camera$videoFishEye$encode`            | String                      | 未拼接视频编码。默认H.264。                         |
| `_camera$videoFishEye$encodeSupport`     | String Array                | 未拼接视频支持的编码，只读。当前为`[“H.264”,“H.265”]`     |
| `_camera$videoStreetView$encode`         | String                      | 街景视频编码，默认H.264。                          |
| `_camera$videoStreetView$encodeSupport`  | String Array                | 街景视频支持的编码，只读。当前为`[“H.264”,“H.265”]`      |
| `_camera$videoTimeLapse$encode`          | String                      | 延时摄影视频编码，默认 H.264。                       |
| `_camera$videoTimeLapse$encodeSupport`   | String Array                | 延时摄影视频支持的编码，只读。当前为`[“H.264”,“H.265”]`    |
| `_camera$photo$qry$able`                 | int                         | 拍照隐藏拍摄者模式的开关值，可选值：0 关闭 、1 开启             |
| ``_camera$photo$qry$p1_cd`               | int                         | 拍照隐藏拍摄者模式的第一张图片倒计时，可选值：3（3s）、5（5s）、10（10s） |
| `_camera$photo$qry$p2_cd`                | int                         | 拍照隐藏拍摄者模式的第二张图片倒计时，可选值：3（3s）、5（5s）、10（10s） |
| `_camera$roam$qry$able`                  | int                         | PilotTour隐藏拍摄者模式开关值,可选值：0（关闭）、1（开启）      |
| `_camera$roam$qry$p1_cd`                 | int                         | PilotTour隐藏拍摄者模式第一张图片倒计时，可选值：3（3s）、5（5s）、10（10s） |
| `_camera$roam$qry$p2_cd`                 | int                         | PilotTour隐藏拍摄者模式第二张图片倒计时，可选值：3（3s）、5（5s）、10（10s） |
| `_camera$video$storagePartSupport`       | Array<Map<String,String>>   | 实时视频支持的分段存储列表值，只读， `Map<String,String>`中包含`name`:显示名称，`value`:实际值 |
| `_camera$video$storagePart`              | `String`                    | 实时视频当前分段存储值，空串时不分段                       |
| `_camera$videoFishEye$storagePartSupport` | `Array<Map<String,String>>` | 未拼接视频支持的分段存储值，只读，具体格式参考`_camera$video$storagePartSupport` |
| `_camera$videoFishEye$storagePart`       | `String`                    | 未拼接视频当前分段存储值，空串时不分段                      |
| `_camera$videoStreetView$storagePartSupport` | `Array<Map<String,String>>` | 街景视频支持的分段存储列表值，只读，具体格式参考`_camera$video$storagePartSupport` |
| `_camera$videoStreetView$storagePart`    | `String`                    | 街景视频当前分段存储值，空串时不分段                       |
| `_camera$videoTimeLapse$storagePartSupport` | `Array<Map<String,String>>` | 延时视频支持的分段存储列表值，只读，具体格式参考`_camera$video$storagePartSupport` |
| `_camera$videoTimeLapse$storagePart`     | `String`                    | 延时视频当前分段存储值，空串时不分段存储                     |

**示例**

```json
{
    "name" : "camera.getOptions",
    "parameters" : {"parameters" : {"optionNames" : ["_camera$video$encode",     "_camera$video$encodeSupport","_camera$videoFishEye$encode",   "_camera$videoFishEye$encodeSupport"]}}
}
```





###  拍摄-设置选项

**功能：** 5.2.0 版本增加，设置camera选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera.setOptions

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段      | 字段类型                | 引入版本  | 描述                      |
| ------- | ------------------- | ----- | ----------------------- |
| options | map<String, Object> | 5.2.0 | 参考设置选项接口中支持选项表格中名称及值类型。 |

**输出参数：**   无

**示例**

```json
{
    "name" : "camera.setOptions",
    "parameters" :{"parameters" : {"options" : {"_camera$photo$qry$able" : 0}}}
}
```







###  直播-获取平台信息 

**功能：** 获取直播某个平台信息

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLivePlatformInfo

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型 | 描述   | 标准参数                                     |
| ---------- | ---- | ---- | ---------------------------------------- |
| platformId | int  | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |



**输出参数：**

| 字段              | 字段类型    | 描述                    |
| --------------- | ------- | --------------------- |
| platformId      | int     | 平台id                  |
| account         | String  | 平台账号名                 |
| autoDefinition  | Boolean | 是否打开自动清晰度             |
| definition      | String  | 清晰度(分辨率)              |
| privacy         | int     | 平台隐私                  |
| title           | String  | 平台标题                  |
| login           | Boolean | 平台是否登录                |
| facebookPubType | int     | 时间线（个人主页）：0；主页：1；小组：2 |
| facebookPubId   | String  | id                    |
| facebookPubName | String  | Name                  |

平台对应隐私:

Facebook: 1-仅自己（Self） 3-所有朋友(All friend)  4-公开(Public)

Youtube: 0-私享(Private)  1-公开(Public)  2-不公开(Unlisted)


**返回内容：**

```json
{
	"id": "camera._getLivePlatformInfo ",
	"name": "camera._getLivePlatformInfo ",
	"results": {
		"platformId": 2,
		"account": "",
		"autoDefinition": false,
		"definition": "1920*960",
		"privacy": 1,
		"title": "",
		"login": false
},
	"state": "done"
}
```





###  直播-设置标题

**功能：** 设置标题

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveTitle

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| title      | String | 标题内容 |                                          |

**输出参数：**  无

```json
{
	"id": "camera._setLiveTitle ",
	"name": "camera._setLiveTitle ",
	"state": "done"
}
```

**返回内容：**





###  直播-设置平台隐私

**功能：** 设置平台隐私

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLivePrivacy

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型 | 描述   | 标准参数                                     |
| ---------- | ---- | ---- | ---------------------------------------- |
| platformId | int  | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| privacy    | int  | 平台隐私 | 平台隐私对应参数参照一下规则                           |

平台对应隐私:

Facebook: 1-仅自己（Self） 3-所有朋友(All friend)  4-公开(Public)

Youtube: 0-私享(Private)  1-公开(Public)  2-不公开(Unlisted)


**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLivePrivacy ",
	"name": "camera._setLivePrivacy ",
	"state": "done"
}
```





###  直播-获取平台的分辨率

**功能：** 获取每个平台支持的分辨率列表

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLiveDefinitionList

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| language   | String | 语言类型 | Zh_cn,en  默认英文                           |

**输出参数：**   

| 字段                | 字段类型     | 描述      |
| ----------------- | -------- | ------- |
| platformId        | int      | 平台id    |
| definition        | String   | 选中的分辨率  |
| definitionStrList | String[] | 分辨率文案列表 |
| definitionList    | String[] | 分辨率列表   |

**返回内容：**

```json
{
	"id": "camera._getLiveDefinitionList ",
	"name": "camera._getLiveDefinitionList ",
		"results": {
		"platformId": 2,
		"definition": "3648*2280",
		"definitionStrList": ["8k", "4k", "HD", "SD", "Fluent"],
		"definitionList": ["3648*2280", "3840*1920", "2560*1280", "1920*960", "1280*640"]
	},
	"state": "done"
}
```





###  直播-设置平台分辨率

**功能：** 设置分辨率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveDefinition

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| definition | String | 分辨率  | 如:3648*2280                              |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLiveDefinition ",
	"name": "camera._setLiveDefinition ",
	"state": "done"
}
```





###  直播-获取已选中的码率

获取每个平台支持的分辨率的对应码率列表

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLiveSelectBitrateList

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| language   | String | 语言类型 | Zh_cn,En_cn 默认英文                         |

**输出参数：**   

| 字段                | 字段类型     | 描述      | 标准参数                                     |
| ----------------- | -------- | ------- | ---------------------------------------- |
| platformId        | int      | 平台id    | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| k8BitRation       | String   | 60 Mbps | 8k选中的码率                                  |
| k4BitRation       | String   | 12 Mbps | 4k选中的码率                                  |
| highBitRation     | String   | 8 Mbps  | HD选中的码率                                  |
| standardBitRation | String   | 5 Mbps  | SD选中的码率                                  |
| smoothBitRation   | String   | 3 Mbps  | 流畅选中的码率                                  |
| definitionStrList | String[] | 分辨率文案列表 |                                          |
| definitionList    | String[] | 分辨率列表   |                                          |

**返回内容：**

```json
{
	"id": "camera._getLiveBitrateList ",
	"name": "camera._getLiveBitrateList ",
	"results": {
		"platformId": 2,
		"selectBitrateList": ["80 Mbs", "60 Mbs"],
		"definitionStrList": ["8k", "4k", "HD", "SD", "Fluent"],
		"definitionList": ["3648*2280", "3840*1920", "2560*1280", "1920*960", "1280*640"]
	},
	"state": "done"
}
```





###  直播-获取码率列表

获取每个平台对应每个分辨率支持的码率列表

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLiveBitrateList

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| definition | String | 分辨率  | 如:3648*2280                              |

**输出参数：**   

| 字段          | 字段类型     | 描述    |
| ----------- | -------- | ----- |
| platformId  | int      | 平台id  |
| bitrate     | String   | 选中的码率 |
| bitrateList | String[] | 码率列表  |



**返回内容：**

```json
{
	"id": "camera._getLiveBitrateList ",
	"name": "camera._getLiveBitrateList ",
	"results": {
		"platformId": 2,
		"bitrate": "80 Mbps",
		"bitrateList": ["80 Mbps", "70 Mbps", "60 Mbps", "50 Mbps", "40 Mbps"]
	},
	"state": "done"
}
```





###  直播-设置码率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveBitrate

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp;1: weibo;2: facebook;3: youtube;4: self |
| definition | String | 分辨率  | 如:3648*2280                              |
| bitrate    | String | 码率   | 如：80 Mbps                                |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLiveBitrate ",
	"name": "camera._setLiveBitrate ",
	"state": "done"
}
```





###  直播-开始直播

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveStart

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段         | 字段类型   | 描述   | 标准参数                                     |
| ---------- | ------ | ---- | ---------------------------------------- |
| platformId | int    | 平台id | 0: rtmp;1: weibo;2: facebook;3: youtube;4: self |
| type       | int    | 直播类型 | 根据分辨率-平台ID 判断是否是8K或者其他 0-8k 其他值（其他）       type 推流类型 0：8k  1:普通直播 |
| language   | String | 语言类型 | Zh_cn,En_cn 默认英文                         |

注意：type 推流类型 0：8k  1:普通直播

**返回内容：**

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLiveStart ",
	"name": "camera._setLiveStart",
	"state": "done"
}
```





###  直播-停止直播

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveStop

**请求头参数(version)：** 4.11.0

**输入参数：**  无

**输出参数：**  无

**返回内容：**

```Json
{
	"id": "camera._setLiveStop ",
	"name": "camera._setLiveStop ",
	"state": "done"
}
```





###  直播-获取专业选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getProfessionalEx

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段    | 字段类型 | 描述   | 标准参数 |
| ----- | ---- | ---- | ---- |
| proId | int  | 平台id | 传0就行 |

**输出参数：**

| 字段       | 字段类型 | 描述   | 标准参数 |
| -------- | ---- | ---- | ---- |
| ev       | int  | Ev值  |      |
| iso      | int  | Iso值 |      |
| distance | int  | 拼接距离 |      |

**返回内容：**

```json
{
	"id": "camera._getProfessionalEx ",
	"name": "camera._getProfessionalEx",
		"results": {
		"ev": 0,
		"iso": 0,
		"distance ":100
	},
	"state": "done"
}
```





###  直播-设置专业选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setProfessionalEx

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段    | 字段类型   | 描述   | 标准参数                                     |
| ----- | ------ | ---- | ---------------------------------------- |
| type  | String | 类型   | `ev` ; `distance` ; `iso`                |
| value | int    | 值    | `ev`: 0 ~ 8   `distance`: 0 ~ 100   `iso`: 0 ~ 6 |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setProfessionalEx ",
	"name": "camera._setProfessionalEx ",
	"state": "done"
}
```



###  直播-设置自身推流直播名

http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLivePushId

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段         | 字段类型   | 描述          |
| ---------- | ------ | ----------- |
| pushId     | String | 自身推流直播名     |
| platformId | int    | 4 自身推流的平台ID |

**输出参数：**  无

**返回内容：**

```Json
{
	"id": "camera._setSettingLedParams",
	"name": "camera._setSettingLedParams",
	"state": "done"
}
```







###  直播-重置直播状态

**功能：** 直播出错后，需要调用这个接口，把状态设置为未直播。

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveCode

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段   | 字段类型 | 描述   | 标准参数 |
| ---- | ---- | ---- | ---- |
| code | int  | 错误码  | 0    |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLiveCode ",
	"name": "camera._setLiveCode",
	"state": "done"
}
```





###  直播-心跳增加字段code

对应解析：

0:相机闲置状态

1:相机直播中

2:普通直播间创建失败

3:8k直播创建失败

<0: 直播错误





###  直播-获取直播平台

获取当前的平台ID

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLiveShowPlatform

**请求头参数(version)：** 4.11.0

**输入参数：**无

**输出参数：**  无

**输出参数：**  无
| 字段         | 字段类型 | 描述   | 标准参数                                     |
| ---------- | ---- | ---- | ---------------------------------------- |
| platformId | int  | 平台ID | 0: rtmp  1: weibo  2: facebook  3: youtube  4：self |

**返回内容：**

```json
{
	"id": "camera._getLiveShowPlatform",
	"name": "camera._getLiveShowPlatform",
		"results": {
		"platformId": 2
	},
	"state": "done"
}
```





###  直播-设置自动分辨率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveAutodefinition

**请求头参数(version)：** 4.11.0

**输入参数：**

| 字段             | 字段类型    | 描述   | 标准参数                                     |
| -------------- | ------- | ---- | ---------------------------------------- |
| platformId     | int     | 类型   | 0: rtmp  1: weibo  2: facebook  3: youtube  4：self |
| autoDefinition | boolean | 自动   | false or true                            |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setLiveAutodefinition",
	"name": "camera._setLiveAutodefinition",
	"state": "done"
}
```





###  直播-还原分辨率

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._resetResolution

**请求头参数(version)：** 4.11.0

**输入参数：**无

**输出参数：**  无

```json
{
	"id": "camera._resetResolution",
	"name": "camera._resetResolution",
	"state": "done"
}
```

**返回内容：**





###  直播-重试直播

**功能:**      心跳接口返回的code为-3006时，可使用此接口

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._restartLive

**请求头参数(version)：** 4.11.0

**输入参数：**无

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._restartLive ",
	"name": "camera._restartLive ",
	"state": "done"
}
```



###  直播-获取PilotSteady

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getLiveOrientationState

**请求头参数(version)：** 4.14.0

**输入参数：**无

**输出参数：**

| 字段               | 字段类型 | 描述              |
| ---------------- | ---- | --------------- |
| steadyValue      | int  | 0:关闭   1:开      |
| orientationValue | int  | 0: 固定  1:跟随相机朝向 |

**返回内容：**

```json
{
	"id": "camera._getLiveOrientationState",
	"name": "camera._getLiveOrientationState",
	"results":{
	  	"steadyValue": 0.
	    "orientationValue":0
	},
	"state": "done"
}
```





###  直播-设置PilotSteady

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setLiveOrientationState

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段    | 字段类型 | 描述                                       |
| ----- | ---- | ---------------------------------------- |
| type  | int  | 0:pilotSteady稳定   1:画面朝向                 |
| value | int  | PilotSteady稳定(0:关 1:开)  画面朝向：(0:固定 1:跟随相机朝向) |

**输出参数：**  无

**返回内容：**

```Json
{
	"id": "camera._setLiveOrientationState",
	"name": "camera._setLiveOrientationState",
	"state": "done"
}
```





###  直播-获取设置选项

**功能：** 5.2.0 版本增加，获取Live选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.getOptions

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段          | 字段类型         | 描述                        |
| ----------- | ------------ | ------------------------- |
| optionNames | String Array | 需要获取值的选项名称数组。参考**直播选项表**。 |

**输出参数：**

| 字段      | 字段类型                | 描述          |
| ------- | ------------------- | ----------- |
| options | map<String, Object> | 参考**直播选项表** |

**返回内容：**

**直播选项表**：

| 选项名称                       | 值类型          | 描述                                 |
| -------------------------- | ------------ | ---------------------------------- |
| `_live$rtmp$encode`        | String       | rtmp编码，默认H.264。                    |
| `_live$rtmp$encodeSupport` | String Array | 只读，rtmp支持的编码。当前为[“H.264”,“H.265”]  |
| `_live$self$encode`        | String       | 自身推流编码，默认H.264。                    |
| `_live$self$encodeSupport` | String Array | 只读，自身推流支持的编码。当前为 [“H.264”,“H.265”] |


###  直播-设置选项

**功能：** 5.2.0 版本增加，设置Live选项

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.setOptions

**请求头参数(version)：** 5.0.0

**输入参数：**

| 字段      | 字段类型                | 描述          |
| ------- | ------------------- | ----------- |
| options | map<String, Object> | 参考**直播选项表** |

**输出参数：**   无

**返回内容：**



###  直播-设置Facebook直播类型(未开放)

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.facebook.setPubType

**请求头参数(version)：** 可不设置

**输入参数：**

| 字段      | 字段类型   | 描述                           |
| ------- | ------ | ---------------------------- |
| pubType | int    | 设置直播类型：时间线（个人主页）：0；主页：1；小组：2 |
| pubId   | String | id                           |

**输出参数:**  无

**返回内容：**

```json
{
	"id": " _live.facebook.setPubType",
	"name": " _live.facebook.setPubType",
	"state": "done"
}
```





###  直播-获取Facebook的pages(未开放)

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.facebook.pages

**请求头参数(version)：** 可不设置

**输入参数：** 无

**输出参数:**  需要调用**查询进度**接口获取结果

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| curId | String | 当前显示的ID                                  |
| list  | Array  | 从Facebook获取到的page数据(返回结果中没有该字段，说明获取数据失败，可重试) |
| id    | String | 一个page的id                                |
| name  | String | 一个page的名字                                |

**返回内容：**

```json
{
  "curId":"xxxx", 
  "list": [
    {
      "id"":"xxxx", 
      "name":"xxxx"
    }
  }
}
```





###  直播-获取Facebook的groups(未开放)

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.facebook.groups

**请求头参数(version)：** 可不设置

**输入参数：**无

**输出参数:**  需要调用**查询进度**接口获取结果

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| curId | String | 当前显示的ID                                  |
| list  | Array  | 从Facebook获取到的group数据(返回结果中没有该字段，说明获取数据失败，可重试) |
| id    | String | 一个group的id                               |
| name  | String | 一个group的名字                               |

**返回内容：**

```json
{
  "curId":"xxxx", 
  "list": [
    {
      "id"":"xxxx", 
      "name":"xxxx"
    }
  }
}
```





###  直播-获取Facebook发布类型（未开放）

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  _live.facebook.pubTypes

**请求头参数(version)：** 可不设置

**输入参数：**

| 字段       | 字段类型   | 描述             |
| -------- | ------ | -------------- |
| language | String | 中文：zh    英文：en |

**输出参数:**  

| 字段    | 字段类型   | 描述                  |
| ----- | ------ | ------------------- |
| curId | String | 当前显示的ID             |
| list  | Array  | 从Facebook获取到的page数据 |
| id    | String | 一个page的id           |
| name  | String | 一个page的名字           |

**返回内容：**

```json
{
  	"curType": 1
	"list":[{
     	"type": 1
     	"name":"string"    
	}] 
}
```





###  系统-获取底部logo配置

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getCameraWatermarkList

**请求头参数(version)：** 4.14.0

**输入参数：**无

**输出参数：**

| 字段             | 字段类型     | 描述                        |
| -------------- | -------- | ------------------------- |
| watermarkState | int      | 是否显示底部logo。 0:打开状态 1:关闭状态 |
| logoSize       | int      | 界面进度条的值                   |
| imageSelect    | String   | 当前使用的logo文件名              |
| imgList        | String[] | Logo 列表                   |



###  系统-设置底部logo开关

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setCameraWatermarkState

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段             | 字段类型 | 描述            |
| -------------- | ---- | ------------- |
| watermarkState | int  | 0:打开状态 1:关闭状态 |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setCameraWatermarkState",
	"name": "camera._setCameraWatermarkState",
	"state": "done"
}
```





###  系统-设置logo大小

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setCameraWatermarkSize

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段            | 字段类型 | 描述         |
| ------------- | ---- | ---------- |
| watermarkSize | int  | 最小：0 最大 26 |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setCameraWatermarkSize",
	"name": "camera._setCameraWatermarkSize",
	"state": "done"
}
```





###  系统-设置选中logo

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setCameraWatermarkSelect

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段          | 字段类型   | 描述                      |
| ----------- | ------ | ----------------------- |
| imageSelect | String | 文件名  “”空 字符串代表设置默认logo! |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setCameraWatermarkSelect",
	"name": "camera._setCameraWatermarkSelect",
	"state": "done"
}
```





###  系统-上传logo文件

**功能：** 上传logo文件

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._uploadWatermarkFile

**输入参数：**

| 字段       | 字段类型   | 描述   |
| -------- | ------ | ---- |
| fileName | string | 文件名  |

**输出参数：**  无

**返回内容：** 无



###  系统-查询上传图片进度

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getUploadProgress

**请求头参数(version)：** 4.14.0

**输入参数：**无

**输出参数：**

| 字段        | 字段类型   | 描述                      |
| --------- | ------ | ----------------------- |
| uploadMsg | String | 提示信息：上传成功  上传失败         |
| imgPath   | String | 上传成功，则返回图片的路径； 上传失败则为空。 |







###  系统-获取声音参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getVoiceParams

**请求头参数(version)：** 4.12.0

**输入参数：** 无 

**输出参数：**

| 字段                 | 字段类型    | 描述      | 标准参数  |
| ------------------ | ------- | ------- | ----- |
| micVoiceMax        | int     | 麦克风最大值  | 100   |
| micVoiceMin        | int     | 麦克风最小值  | 0     |
| micVoiceValue      | int     | 麦克风当前值  | 70    |
| speakerVoiceMax    | int     | 扬声器最大值  | 15    |
| speakerVoiceMin    | int     | 扬声器最小值  | 0     |
| speakerVoiceValue  | int     | 扬声器当前值  | 3     |
| pictureVoiceSwitch | boolean | 拍照快门声开关 | false |
| videoVoiceSwitch   | boolean | 录像快门声开关 | false |
| liveVoiceSwitch    | boolean | 直播快门声开关 | false |

**返回内容：**

```json
{
	"id": "camera._getVoiceParams",
	"name": "camera._getVoiceParams",
	"results": {
	  	"micVoiceMax":100, 
     	"micVoiceMin":0,
	  	"micVoiceValue":70,
	   	"speakerVoiceMax":15,
	    "speakerVoiceMin":0,
		"speakerVoiceValue":3,
		"pictureVoiceSwitch":false,
		"videoVoiceSwitch":false,
		"liveVoiceSwitch":false,
	},
	"state": "done"
}
```





###  系统-设置声音参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setVoiceParams

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段    | 字段类型   | 描述                                       |
| ----- | ------ | ---------------------------------------- |
| type  | String | `speaker`代表扬声器  `mic`代表麦克风  `picture`代表拍照快门声  `video`代表视频快门声  `live`代表直播快门声 |
| value | int    |                                          |


**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setVoiceParams",
	"name": "camera._setVoiceParams",
	"state": "done"
}
```





###  系统-获取风扇状态

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getFanParms

**请求头参数(version)：** 4.12.0

**输入参数：**无

**输出参数：**

| 字段    | 字段类型 | 描述                      |
| ----- | ---- | ----------------------- |
| value | int  | 1: auto  2:一直开启  0:一直关闭 |

**返回内容：**

```json
{
	"id": "camera._getFanParms",
	"name": "camera._getFanParms",
	"results": {
	  "value":1
	},
	"state": "done"
}
```





###  系统-设置风扇状态

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setFanParms

**请求头参数(version)：** 4.12.0

**输入参数：**

| 字段    | 字段类型 | 描述                      |
| ----- | ---- | ----------------------- |
| value | int  | 1: auto  2:一直开启  0:一直关闭 |

**输出参数：**  无

**返回内容：**

 

```json
{
	"id": "camera._setFanParms",
	"name": "camera._setFanParms",
	"state": "done"
}
```







###  系统-获取提示灯参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._getSettingLedParams

**请求头参数(version)：** 4.14.0

**输入参数：**无

**输出参数：**

| 字段         | 字段类型 | 描述            |
| ---------- | ---- | ------------- |
| ledPhoto   | int  | 1:打开状态 0:关闭状态 |
| ledVideo   | int  | 1:打开状态 0:关闭状态 |
| ledLive    | int  | 1:打开状态 0:关闭状态 |
| ledBattery | int  | 1:打开状态 0:关闭状态 |

**返回内容：** 





###  系统-设置提示灯参数

**调用方式：**  http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._setSettingLedParams

**请求头参数(version)：** 4.14.0

**输入参数：**

| 字段    | 字段类型 | 描述                     |
| ----- | ---- | ---------------------- |
| type  | int  | 0: 拍照 1:录像 2:直播  3:低电量 |
| value | int  | 1打开 0:关闭               |

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._setSettingLedParams",
	"name": "camera._setSettingLedParams",
	"state": "done"
}
```

 



###  初始化相机预览流

**功能：**  初始化预览流 退出底部logo时调用

**调用方式：** http+json,Post传参

**请求地址：**  http://x.x.x.x:8080/osc/commands/execute

**命令ID：**  camera._initSurfacePreview

**请求头参数(version)：** 4.14.0

**输入参数：** 无

**输出参数：**  无

**返回内容：**

```json
{
	"id": "camera._initSurfacePreview",
	"name": "camera._initSurfacePreview",
	"state": "done"
}
```





###  取消任务

仅用于 开启“隐藏拍摄者模式"后调用camera.takePicture



**调用方式：**  http+json,Post传参

**请求地址：**  <http://x.x.x.x.x:8080/osc/commands/_cancel>

**请求头参数(version)：** 可不设置

**命令ID：**  需要查询结果的id

| 字段   | 字段类型   | 描述        |
| ---- | ------ | --------- |
| id   | string | 需要取消的任务id |

**输入参数：**无

**输出参数:**  无