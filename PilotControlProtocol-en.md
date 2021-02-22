# Pilot Control Protocol

Pilot Control Protocol is used for mobile phone or PC application to control Pilot series Camera (Pilot Era/One/OneEE) in LAN. It is customized based on OSC protocol. Udp was used when searching for Pilot Camera.

Our mobile phone App [Pilot Go](https://www.labpano.com/support/download#PilotGo) is the use case of this protocol to control the Pilot Camera.

If Pilot Camera has turned off screen, it cannot be searched.

After connected to Pilot Camera, the UI prompts "mobile controlling...".

This document applies to Pilot OS 5.12.0 and above.



## 1. Introduction



### Request HOST

http://x.x.x.x:8080

x.x is the IP,8080 of the current Pilot camera that is the port where the camera listens.



### Protocol

-Udp + json: Used only when searching for cameras


-http + json



### Interface Request

**You need to add request header when using http request**

The request header `Content-Type` is set to `application/x-www-form-urlencoded`

Request header `version` to set the corresponding version value according to the interface instructions.

Execute commands with Post request, download files with Get request.

Http timeout recommended 15s.



The main request addresses are as follows:

|     Function     | Request URL                              |
| :--------------: | ---------------------------------------- |
| Execute Commands | http://x.x.x.x:8080/osc/commands/execute |
|  Query Progress  | http://x.x.x.x:8080/osc/commands/status  |
|  Download Files  | http://x.x.x.x:8080/osc/getFile          |



### Example 

Chapter 2 lists each interfaces, and here is an example of the Connect Camera interface.



#### Request

The common json format for http requests is as follows:

```json
{
    "name": "interface name",								//interface name, string type
    "parameters": "interface input param" 	//interface input param, string type
}
```



**Call Mode:**   http+json,Post

**Request url:**   http://x.x.x.x:8080/osc/commands/execute

**request header version:**   3.1

The name of Connect Camera interface is: camera.startSession

The input param of Connect Camera interface is:

```json
{
    "parameters" :     {
        "alone" : true,
        "sessionId" : "53c61259-373e-4eed-975c-5d5500be176b",
        "timeout" : 1800
    }
}
```



The input param of the interface is converted to string and combined with the interface name into json, to initiate the http request.

```json
{
    "name": "camera.startSession",	//interface name, string type
    "parameters": "{"parameters" : {"alone" : true,"sessionId" : "53c61259-373e-4eed-975c-5d5500be176b","timeout" : 1800}}"	 //interface input param, string type
  
}
```





#### Response

After the request is initiated, the Pilot Camera returns a json. The data structure is as follows:

|   Field    |   Type    | Description                              |
| :--------: | :-------: | ---------------------------------------- |
|     id     |  string   | interface unique id, name is the same as id |
|    name    |  string   | interface name                           |
|   state    |  string   | There are 3 states: `done`     `inProgress`      `error` |
|  results   | json node | Responses the result. Different interfaces have different contents. |
| parameters | json node | Output parameters. Different interfaces have different contents. |
|   error    | json node | Error details                            |
|    code    |  string   | Error code                               |
|  message   |  string   | Error msg                                |




Take the Connect Camera interface as an example, the json returned by Pilot Camera is as follows:

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




Note: if the state is`inProgress`, you need to call the **Query Progress**  interface to see if the command is done.



## 2 Interfaces



### Search Devices

**Function:**   Before communicating with Pilot, you must search for Pilot Camera in LAN and find the available Pilot Camera through this interface. The Pilot camera UDP port is 10000.

**Call Mode:**  Udp+json

**Input params:**  

|   Field   |  Type  | Description                              |
| :-------: | :----: | ---------------------------------------- |
|   port    |  int   | port of phone side (10000)               |
| sessionId | string | Session ID of last connection, pass empty if not connected |
|  version  | string | 3.1                                      |

**Udp param:**  

```json
{
    "parameters" :     {
        "port" : 10000,
        "sessionId" : "559d26f2-081f-43f5-8f93-35788a9321b4",
        "version" : "3.1"
    }
}
```

**Output params:**  

| Field            | Type    | Description             |
| ---------------- | ------- | ----------------------- |
| ip               | string  | Pilot Camera ip         |
| port             | int     | Pilot Camera port       |
| deviceName       | string  | Pilot Camera name       |
| alreadyConnected | boolean | Is the camera connected |

**Response:**  

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





### Connect Camera

**Function:**  Connect to Pilot camera

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.startSession

**Request header param (version):**  3.1

**Input params:**  

| Field     | Type    | Description                              |
| --------- | ------- | ---------------------------------------- |
| timeout   | int     | Connection timeout in ms                 |
| sessionId | string  | session id                               |
| alone     | boolean | Whether the session needs to be maintained after the heartbeat timeout. True: The heartbeat timeout, (when Pilot Camera is recording, live streaming,Shooting PilotTour) The session is maintained, the current work of Pilot Camera is not stopped; false: The heartbeat timeout closes session, stop work. |
| language  | string  | Language of Pilot Camera system. English:en, Chinese:zh |

**Output params:**  

| Field           | Type    | Description                              |
| --------------- | ------- | ---------------------------------------- |
| sessionId       | string  | Session id                               |
| timeout         | int     | Connection timeout in ms                 |
| channel         | string  | Channel name, used to indicate the firmware type. Era is `standard`, One and One EE are `one_standard `. Other channels may be added later. |
| mode            | string  | The currnet shooting mode of Pilot Camera.  `photo` means Take Photo;  `video` means Stitched Video;  `video_feye` means Unstitched Video;  `video_street_view` means Streetview Video;  `video_time_lapse` means TimeLapse;  `photo_tours` means PilotTour |
| status          | boolean | Pilot Camera status, 'true' means recording or streaming; 'false' means idle |
| firmwareVersion | string  | Pilot OS version                         |
| exposureType    | int     | Exposure mode type: 0 for auto mode, 1 for manual mode |
| modeWay         | string  | Used to hide photographer mode. This field is not supported in other modes |

**Response:**  

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





### Heartbeat

**Function:**  After Pilot Camera has established a successful connection, the mobile phone requests the heartbeat, Pilot Camera  receives it and returns to mobile phone, and then mobile phone send back and circulates all the time.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getHeart

**Request header param (version):**  4.7

**Input params:**  None

**Output params:**  

| Field           | Type    | Description                              |
| --------------- | ------- | ---------------------------------------- |
| battery         | string  | Battery power                            |
| latitude        | double  | Latitude                                 |
| longitude       | double  | Longitude                                |
| lowMemory       | boolean | Storage status. true for low storage; false for ample storage |
| pushStream      | boolean | Stream status. true for streaming; false for noy streaming |
| tick            | int     | Shooting countdown, Pilot Camera returns in real time. Perform "shooting hidden photographer mode"  the total tick duration is "countdown to first photo " +" countdown to second photo." |
| videoTime       | string  | record duration                          |
| tourPrepare     | int     | PilotTour track status. 0: PilotTour is not tracked; 1: PilotTour is tracked. |
| roamStateDetail | int     | New version of PilotTour status details: 0-uninitialized 1-initializing 2-tracking 3-loss tracking (used by camera 5.6.0 or above) |
| checkTime       | boolean |                                          |
| sdCritical      | long    | Storage threshold                        |
| sdTotalSize     | long    | Total storage                            |
| sdAvailableSize | long    | Available Storage                        |
| temperature     | int     | Temperature (suitable for Pilot OneEE), when the temperature is greater than 80℃,  prompts camera CPU temperature is too high |
| taskId          | string  | The current camera in progress (preferred) task id, currently only supports photographing tasks |
| code            | int     | Live broadcast error code: 1-Live broadcast; 0-Live broadcast not started; -3006-Retry live broadcast (camera.restartLive); Other error codes: call stop live broadcast (camera.setLiveStop) |

**Response:**  

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





### Disconnect

**Function:**   Turn off the connection.

**Call Mode:**   http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.closeSession

**Request header param (version):**  3.0

**Input params:**  

| Field     | Type   | Description |
| --------- | ------ | ----------- |
| sessionId | string |             |

**Output params:**  None

**Response:**  

```json
{
	"id": "camera.closeSession",
	"name": "camera.closeSession",
	"state": "done"
}
```





### Start Preview

**Function:**  Open camera preview.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._startPreview

**Request header param (version):**  3.0

**Input params:**  

| Field    | Type | Description                              |
| -------- | ---- | ---------------------------------------- |
| targetId | int  | Preview for shooting: `1`; Preview for streaming: `3` |

**Output params:**   Need to call **Query Progress** to get results

**Response:**  

```json
{
	"id": "camera._startPreview",
	"name": "camera._startPreview",
	"state": "inProgress"
} 
```





### Stop Preview

**Function:**  Stop the preview.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._stopPreview

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  None

**Response:**  

```json
{
	"id": "camera._stopPreview ",
	"name": "camera._stopPreview ",
	"state": "done"
}
```





### Capture - Switch Mode

**Function:**  Switch shooting mode (Photo, Unstitched Video, stitched video, Streetview Video, PilotTour, TimeLapse.)

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._changeResolutionEx

**Request header param (version):**  3.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | `photo` means Take Photo;  `video` means Stitched Video;  `video_feye` means Unstitched Video;  `video_street_view` means Streetview Video;  `video_time_lapse` means TimeLapse;  `photo_tours` means PilotTour |

**Output params:**   Need to call **Query Progress** to get results

**Response:**  

```json
{
	"id": "camera._changeResolutionEx",
	"name": "camera._changeResolutionEx",
	"state": "inProgress"
}
```





### Query Progress

**Function:**   It will take some time for the Pilot Camera to excute a command. This interface is used to query the progress.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/status

**the command ID:**  The id of command

| Field | Type   | Description    |
| ----- | ------ | -------------- |
| id    | string | **command id** |

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  Different commands output different results.

**Response:**    eg, Take Photo

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





### Capture - Take Photo

**Function:**   Take photo.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.takePicture

**Request header param (version):**  3.0

**Input params:**  None

**Output params**: When in inProgress (currently used only to Hide photographer mode)

| Field   | Type | Description                              |
| :------ | :--- | :--------------------------------------- |
| _pIndex | int  | Photo index. 0 for first, 1 for second   |
| _tick   | int  | Remaining countdown (when the _tick value is 0, it means being photographed) |

**Output params:**   Need to call **Query Progress** to get results

| Field        | Type    | Description                        |
| ------------ | ------- | ---------------------------------- |
| dateTimeZone | string  | shooting time(2018-09-22 14:15:38) |
| fileUrl      | string  |                                    |
| width        | int     | photo width                        |
| height       | int     | phoyo height                       |
| isProcessed  | boolean |                                    |
| lat          | double  | Latitude                           |
| lng          | double  | Longitude                          |
| name         | string  | file name                          |
| size         | size    | file size                          |
| stitch       | boolean | is stitched                        |
| thumbnail    | string  | thumbnail path                     |

**Response:**  

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





### Capture - Start Video

**Function:**  Start record video.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.startCapture

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  Need to call **Query Progress** to get results

**Response:**  

```json
{
	"id": "camera.startCapture",
	"name": "camera.startCapture",
	"state": "inProgress"
}
```





### Capture - Stop Video

**Function:**  Stop video recording.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.stopCapture

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  Need to call **Query Progress** to get results

| Field        | Type    | Description                              |
| ------------ | ------- | ---------------------------------------- |
| dateTimeZone | string  | Capture Time                             |
| fileUrl      | string  |                                          |
| width        | int     | Resolution width                         |
| height       | int     | Resolution height                        |
| fps          | string  |                                          |
| name         | string  |                                          |
| size         | int     |                                          |
| stitch       | boolean | Is stitched.  true for stitched; false for unstitched |
| thumbnail    | string  | thumbnail path                           |
| codec        | string  | codec (H.264/265 , AAC)                  |

**Response:**  

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





### Download File

**Function:**  Download a file from Gallery

**Call Mode:**  http+json,Get

**Request url:**   http://x.x.x.x:8080/osc/getFile?/storage/emulated/0/DCIM/Thumb/180922_103443479.jpg

**Input params:**  None

**Output params:**  file stream

**Response:**  None



### Get File List

**Function:**  Get the list of gallery files

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.listFiles

**Request header param (version):**  3.0

**Input params:**  

| Field         | Type   | Description               |
| ------------- | ------ | ------------------------- |
| fileType      | string | `image`;  `video`;  `all` |
| startPosition | int    | 0                         |
| entryCount    | int    | 1                         |
| maxSize       | int    | 5                         |
| typeVersion   | string | only `tours`              |

**Output params:**  Need to call **Query Progress** to get results

| Field        | Type | Description              |
| ------------ | ---- | ------------------------ |
| totalEntries | int  | Total number             |
| entries      |      | json node of single file |

**Response:**  

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









### Delete Files

**Function:**  Delete photos or video files.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.delete

**Request header param (version):**  5.0.0

**Input params:**  

| Field    | Type | Description |
| -------- | ---- | ----------- |
| fileList | List | File Urls   |

Format

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

**Output params:**  None

Return success:

```json
{
	"id": "camera.delete",
	"name": "camera.delete",
	"results": [],
	"state": "done"
}
```

Return failure

```json
{
	"id": "camera.delete",
	"name": "camera.delete",
	"results": ["190314_194559664.jpg", "190314_194553689.jpg"],
	"state": "done"
}
```





### Capture - Set Countdown Switch

**Function:**  Sets whether to turn on the countdown

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setDelayOpen

**Request header param (version):**  3.0

**Input params:**  

| Field  | Type    | Description                              |
| ------ | ------- | ---------------------------------------- |
| type   | string  | `photo` means Take Photo;  `video` means Stitched Video;  `video_feye` means Unstitched Video;  `video_street_view` means Streetview Video;  `video_time_lapse` means TimeLapse;  `photo_tours` means PilotTour |
| isOpen | boolean |                                          |

**Output params:**    None

**Response:**

```json
{
	"id": "camera._setDelayOpen ",
	"name": "camera._setDelayOpen ",
	"state": "done"
}
```





### Capture - Set Countdown Time

**Function:**  Set countdown time

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setDelay

**Request header param (version):**  3.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | `photo` means Take Photo;  `video` means Stitched Video;  `video_feye` means Unstitched Video;  `video_street_view` means Streetview Video;  `video_time_lapse` means TimeLapse;  `photo_tours` means PilotTour |
| delay | int    | countdown seconds. 3、5、10、15             |

**Output params:**    None

**Response:**

```json
{
	"id": "camera._setDelay",
	"name": "camera._setDelay",
	"state": "done"
}
```





### Capture - Get Countdown

**Function:**  Get countdown configuration

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getDelay

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  

| Field                  | Type | Description                        |
| ---------------------- | ---- | ---------------------------------- |
| delayphoto             | int  | countdown time of Take Photo       |
| delayvideo             | int  | countdown time of Stitched Video   |
| delayvideo_feye        | int  | countdown time of Unstitched Video |
| delayvideo_street_view | int  | countdown time of StreetView Video |
| delayvideo_time_lapse  | int  | countdown time of TimeLapse        |
| delaytours             | int  | countdown time of PilotTour        |

**Response:**

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





### Capture - Set Resolution

**Function:**  Set resolution

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setResolution

**Request header param (version):**  3.0

**Input params:**  

| Field      | Type   | Description                              |
| ---------- | ------ | ---------------------------------------- |
| type       | string | `photo` means Take Photo;  `video` means Stitched Video;  `video_feye` means Unstitched Video;  `video_street_view` means Streetview Video;  `video_time_lapse` means TimeLapse;  `photo_tours` means PilotTour |
| resolution | string | resolution string.   `photo` and `photo_tours` are: 8192\*4096, 6144\*3072, 4096\*2048, 3072\*1536, 2048\*1024;      `video` are 7680\*3840, 5760\*2880, 3840\*1920, 1920\*960; `video_feye`  are 7680\*3840, 7040\*3520, 5760\*2880, 3840\*1920; `video_time_lapse` are 7680\*3840, 5760\*2880, 3840\*1920, 1920\*960; |

**Output params:**   Need to call **Query Progress** to get results

**Response:**

```json
{
	"id": "camera._setResolution",
	"name": "camera._setResolution",
	"state": "inProgress"
}
```





### Capture - Get Resolution

**Function:**  Get camera resolution

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getResolution

**Request header param (version):**  3.0

**Input params:**  None

**Output params:**  

| Field                      | Type   | Description                    |
| -------------------------- | ------ | ------------------------------ |
| resolutionphoto            | string | resolution of Photo            |
| resolutionvideo            | string | resolution of Stitched Video   |
| resolutionvideo_feye       | string | resolution of Unstitched Video |
| resolutionvideo_lapse_time | string | resolution of TimeLapse        |
| resolutiontours            | string | resolution of PilotTour        |

**Response:**

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





### Capture - Set Pro Options

**Function:**  Set camera professional settings parameters

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setProfessionalEx

**Request header param (version):**  5.0.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | iso  ev   distance for stitch focus   exposure for exposure time  wb for white balance |
| value | int    | iso 0 ~ 6;   ev -4 ~ 4;   distance 0 ~ 100;   exposure 0 ~ 6;  wb 0 ~ 4 |
| proId | int    | 0 for Photo;  1 for Unstitched Video;  2 for Stitched Video;  3 for Streetview Video;  4 for TimeLapse;  5 for PilotTour |
| mode  | int    | exposure mode.  1 for manual;   0 for auto |



**Output params:**    None

**Response:**

```json
{
	"id": "camera._setProfessionalEx",
	"name": "camera._setProfessionalEx",
	"state": "done"
}
```





### Capture - Get Pro Options

**Function:**  Get camera professional setting parameters

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getProfessionalEx

**Request header param (version):**  5.0.0

**Input params:**  

| Field | Type | Description                              |
| ----- | ---- | ---------------------------------------- |
| proId | int  | 0 for Photo;  1 for Unstitched Video;  2 for Stitched Video;  3 for Streetview Video;  4 for TimeLapse;  5 for PilotTour |



**Output params:**  

| Field        | Type | Description                              |
| ------------ | ---- | ---------------------------------------- |
| distance     | int  | stitch focus                             |
| ev           | int  |                                          |
| exposure     | int  |                                          |
| exposureType | int  | 1 for manual;   0 for auto               |
| iso          | int  |                                          |
| manualISO    | int  |                                          |
| proId        | int  | 0 for Photo;  1 for Unstitched Video;  2 for Stitched Video;  3 for Streetview Video;  4 for TimeLapse;  5 for PilotTour |



**Response:**

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





### Capture - Set HDR Mode

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setPhotoHdrState

**Request header param (version):**  5.0.0

**Input params:**  

| Field    | Type | Description                          |
| -------- | ---- | ------------------------------------ |
| hdrState | int  | 0: auto 1: Sunny 2: Cloudy 3: Indoor |


**Output params:**   None

**Response:**

```json
{
	"id": "camera._setPhotoHdrState",
	"name": "camera._setPhotoHdrState",
	"state": "done"
}
```

 



### Capture - Set Photo Params

**Function:**  Set photo parameters

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setPhotoArgs

**Request header param (version):**  5.0.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | `hdr` for HDR;  `gyroscope` for Auto-Level;  `flow` for Optical-Flow Stitching |
| mode  | int    | 0 for Photo;  1 for Unstitched Video;  2 for Stitched Video;  3 for Streetview Video;  4 for TimeLapse;  5 for PilotTour |
| value | bool   |                                          |

**Output params:**    None

**Response:**

```json
{
	"id": "camera._setPhotoArgs",
	"name": "camera._setPhotoArgs",
	"state": "done"
}
```





### Capture - Get Photo Params

**Function:**  Get photo parameters

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getPhotoArgs

**Request header param (version):**  5.0.0

**Input params:**  None

**Output params:**  

| Field     | Type    | Description           |
| --------- | ------- | --------------------- |
| gyroscope | boolean | Auto-Level switcher   |
| flow      | boolean | Optical-Flow switcher |
| hdr       | boolean | HDR switcher          |
| hdrState  | int     | HDR Mode              |


**Response:**

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



### Capture - Get StreetView Video FPS

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getGoogleRateList

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type   | Description                        |
| ----- | ------ | ---------------------------------- |
| type  | string | `video_street_view` for StreetView |

**Output params:**  

| Field          | Type     | Description  | Content Example            |
| -------------- | -------- | ------------ | -------------------------- |
| googleratelist | string[] | FPS list     | `[“1fps”, “2fps”, “7fps”]` |
| rate           | string   | Selected FPS | `1fps`                     |

**Response:**

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





### Capture - Set StreetView Video FPS

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setGoogleRate

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type   | Description                        |
| ----- | ------ | ---------------------------------- |
| type  | string | `video_street_view` for StreetView |
| rate  | string | eg. `7fps`                         |

**Output params:**  None

**Response:**

```json
{
	"id": "camera._setGoogleRate",
	"name": "camera._setGoogleRate",
	"state": "done"
}
```





### Capture - Get PilotSteady Orientation

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getOrientationState

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | `video` for Stitched Video; `video_feye` for Unstitched Video; `video_time_lapse` for TimeLapse |

**Output params:**  

| Field       | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| orientation | boolean | `false` for Fixed; `true` for Follow Camera; |

**Response:**

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





### Capture - set PilotSteady

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setOrientationState

**Request header param (version):**  4.12.0

**Input params:**  

| Field       | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| type        | string  | `video` for Stitched Video; `video_feye` for Unstitched Video; `video_time_lapse` for TimeLapse |
| orientation | boolean | `false` for Fixed; `true` for Follow Camera; |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setOrientationState",
	"name": "camera._setOrientationState",
	"state": "done"
}
```





### Capture - Get TimeLapse Scale Time

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getTimeLapseTimeList

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type   | Description        |
| ----- | ------ | ------------------ |
| type  | string | `video_time_lapse` |

**Output params:**  

| Feild             | Type     | Description         | Content Example                          |
| ----------------- | -------- | ------------------- | ---------------------------------------- |
| timeLapseTimeList | String[] | Scale Time List     | `[“20x  (0.6spf)”,” 50x  (1.7spf)”,…….,” 10000x  (5.6mpf)”]` |
| timeLapseTimes    | String   | Selected Scale Time | `20x  (0.6spf)`                          |

**Response:**

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





### Capture - Get TimeLapse Scale Time

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLapseTime

**Request header param (version):**  4.12.0

**Input params:**  

| Field          | Type   | Description                            |
| -------------- | ------ | -------------------------------------- |
| type           | string | `video_time_lapse`                     |
| timeLapseTimes | string | Scale time string. eg. `20x  (0.6spf)` |

**Output params:**    None

**Response:**

```json
{
	"id": "camera._setLapseTime",
	"name": "camera._setLapseTime",
	"state": "done"
}
```



### Capture - Start PilotTour

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._startTour

**Request header param (version):**  5.0.0

**Input params:**  

| Field    | Type   | Description          |
| -------- | ------ | -------------------- |
| tourName | string | PilotTour scene name |

**Response:**

```Json
{
	"id": " camera._startTour",
	"name": " camera._startTour",
	"state": "done"
}
```





### Capture - Stop PilotTour

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._stopTour

**Request header param (version):**  5.0.0

**Input params:**  

**Response:**  

```json
{
	"id": " camera._stopTour",
	"name": " camera._stopTour",
	"state": "done"
}
```





### Capture - Get Options

**feature:**  5.2.0 version added to get the Camera option

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.getOptions

**Request header param (version):**  5.0.0

**Input params:**  

| Field       | Type         | Description                              |
| ----------- | ------------ | ---------------------------------------- |
| optionNames | string Array | Options array. See option names in the table below. |

**Output params:**  

| Field   | Type                | Description                              |
| ------- | ------------------- | ---------------------------------------- |
| Options | map<string, Object> | See option names and option types in the table. |

**Response:**

**Option Table:**

| Option Name                              | Type                        | Description                              |
| ---------------------------------------- | --------------------------- | ---------------------------------------- |
| `_camera$roam$height`                    | string                      | PilotTour camera height. Float value, range [0,1000], values outside this range will be processed as boundary values. |
| `_camera$video$encode`                   | string                      | Stitched video codec, default H.264.     |
| `_camera$video$encodeSupport`            | string Array                | Codec supported by Stitched video, read-only. Currently `[“H.264”,“H.265”]` |
| `_camera$videoFishEye$encode`            | string                      | Unstitched video codec, default H.264.   |
| `_camera$videoFishEye$encodeSupport`     | string Array                | Codec supported by Unstitched video, read-only. Currently `[“H.264”,“H.265”]` |
| `_camera$videoStreetView$encode`         | string                      | StreetView video codec, default H.264.   |
| `_camera$videoStreetView$encodeSupport`  | string Array                | Codec supported by StreetView video, read-only. Currently `[“H.264”,“H.265”]` |
| `_camera$videoTimeLapse$encode`          | string                      | TimeLapse codec, default H.264.          |
| `_camera$videoTimeLapse$encodeSupport`   | string Array                | Codec supported by TimeLapse, read-only. Currently `[“H.264”,“H.265”]` |
| `_camera$photo$qry$able`                 | int                         | Photo-Hidden Photographer Mode switcher, value: 0 off, 1 on |
| ``_camera$photo$qry$p1_cd`               | int                         | Photo-Hidden Photographer Mode first photo countdown values: 3 (3s), 5 (5s), 10 (10s) |
| `_camera$photo$qry$p2_cd`                | int                         | Photo-Hidden Photographer Mode second photo countdown values: 3 (3s), 5 (5s), 10 (10s) |
| `_camera$roam$qry$able`                  | int                         | PilotTour-Hidden Photographer Mode switcher, value: 0 off, 1 on |
| `_camera$roam$qry$p1_cd`                 | int                         | PilotTour-Hidden Photographer Mode first photo countdown values: 3 (3s), 5 (5s), 10 (10s) |
| `_camera$roam$qry$p2_cd`                 | int                         | PilotTour-Hidden Photographer Mode second photo countdown values: 3 (3s), 5 (5s), 10 (10s) |
| `_camera$video$storagePartSupport`       | Array<Map<string,string>>   | Supported Segmented Storage values of Stitched video , read-only, `Map<string,string>`, `name` : display name，`value`: real value |
| `_camera$video$storagePart`              | `string`                    | Currently segmented store value of Stitched Video, and is not segmented if empty string |
| `_camera$videoFishEye$storagePartSupport` | `Array<Map<string,string>>` | Supported Segmented Storage values of Unstitched video , read-only |
| `_camera$videoFishEye$storagePart`       | `string`                    | Currently segmented store value of Unstitched Video, and is not segmented if empty string |
| `_camera$videoStreetView$storagePartSupport` | `Array<Map<string,string>>` | Supported Segmented Storage values of StreetView video , read-only |
| `_camera$videoStreetView$storagePart`    | `string`                    | Currently segmented store value of StreetView Video, and is not segmented if empty string |
| `_camera$videoTimeLapse$storagePartSupport` | `Array<Map<string,string>>` | Supported Segmented Storage values of Timelapse video , read-only |
| `_camera$videoTimeLapse$storagePart`     | `string`                    | Currently segmented store value of Timelapse Video, and is not segmented if empty string |

**Example**

```json
{
    "name" : "camera.getOptions",
    "parameters" : {"parameters" : {"optionNames" : ["_camera$video$encode",     "_camera$video$encodeSupport","_camera$videoFishEye$encode",   "_camera$videoFishEye$encodeSupport"]}}
}
```





### Capture - Set Options

**Function:**  5.2.0 version added, set camera options

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera.setOptions

**Request header param (version):**  5.0.0

**Input params:**  

| Field   | Type                | Version | Description           |
| ------- | ------------------- | ------- | --------------------- |
| options | map<string, Object> | 5.2.0   | refer to Option table |

**Output params:**    None

**Example**

```json
{
    "name" : "camera.setOptions",
    "parameters" :{"parameters" : {"options" : {"_camera$photo$qry$able" : 0}}}
}
```







### LiveStream - Get Platform Info

**Function:**  Get live stream information on a platform

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLivePlatformInfo

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type | Description    | Content                                  |
| ---------- | ---- | -------------- | ---------------------------------------- |
| platformId | int  | Platform Index | 0: rtmp,  1: weibo,  2: Facebook,  3: YouTube,  4: Self-stream |



**Output params:**  

| Field           | Type    | Description                              |
| --------------- | ------- | ---------------------------------------- |
| platformId      | int     | Platform Index                           |
| account         | string  | Platform account name                    |
| autoDefinition  | Boolean | Is open  autoDefinition                  |
| definition      | string  | Resolution                               |
| privacy         | int     |                                          |
| title           | string  |                                          |
| login           | Boolean | Is sign in                               |
| facebookPubType | int     | 0 for Timeline; 1 for Pages; 2 for Groups |
| facebookPubId   | string  | id                                       |
| facebookPubName | string  | Name                                     |

Platform corresponding privacy:

Facebook: 1 Self; 3 Friends; 4-Public

YouTube: 0 Private; 1 Public; 2 Unlisted


**Response:**  

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





### LiveStream - Set Title

**Function:**  Set the stream title

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveTitle

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description    | Content                                  |
| ---------- | ------ | -------------- | ---------------------------------------- |
| platformId | int    | Platform Index | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| title      | string | title          |                                          |

**Output params:**   None

```json
{
	"id": "camera._setLiveTitle ",
	"name": "camera._setLiveTitle ",
	"state": "done"
}
```

**Response:**  





### LiveStream - Set Privacy

**Function:**  Set platform privacy

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLivePrivacy

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type | Description    | Content                                  |
| ---------- | ---- | -------------- | ---------------------------------------- |
| platformId | int  | Platform Index | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| privacy    | int  | Privacy        |                                          |

Platform privacy:

Facebook: 1 Self; 3 Friends; 4-Public

YouTube: 0 Private; 1 Public; 2 Unlisted


**Output params:**  None

**Response:**  

```json
{
	"id": "camera._setLivePrivacy ",
	"name": "camera._setLivePrivacy ",
	"state": "done"
}
```





### LiveStream - Get Resolution

**Function:**  Get a list of stream resolutions supported by each platform

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLiveDefinitionList

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description    | Content                                  |
| ---------- | ------ | -------------- | ---------------------------------------- |
| platformId | int    | Platform Index | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| language   | string |                | cn,en                                    |

**Output params:**     

| Field             | Type     | Description           |
| ----------------- | -------- | --------------------- |
| platformId        | int      | Platform Index        |
| definition        | string   | selected Resolution   |
| definitionStrList | string[] | Resolution name list  |
| definitionList    | string[] | Resolution value list |

**Response:**

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





### LiveStream - Set Resolution

**Function:**  Set stream resolution for each platform.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveDefinition

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description       | Content                                  |
| ---------- | ------ | ----------------- | ---------------------------------------- |
| platformId | int    | Platform Index    | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| definition | string | Stream resolution | eg. 3648*2280                            |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setLiveDefinition ",
	"name": "camera._setLiveDefinition ",
	"state": "done"
}
```





### LiveStream - Get Selected Bitrate

Gets selected bitrate for the resolutions supported by each platform.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLiveSelectBitrateList

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description    | Content                                  |
| ---------- | ------ | -------------- | ---------------------------------------- |
| platformId | int    | Platform Index | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |
| language   | string | language       | cn, en                                   |

**Output params:**     

| Field             | Type     | Description                   | Content                                  |
| ----------------- | -------- | ----------------------------- | ---------------------------------------- |
| platformId        | int      | Platform Index                | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| k8BitRation       | string   | 60 Mbps                       | Selected bitrate for 8K                  |
| k4BitRation       | string   | 12 Mbps                       | Selected bitrate for 4K                  |
| highBitRation     | string   | 8 Mbps                        | Selected bitrate for HD                  |
| standardBitRation | string   | 5 Mbps                        | Selected bitrate for SD                  |
| smoothBitRation   | string   | 3 Mbps                        | Selected bitrate for Fluent              |
| definitionStrList | string[] | Stream resolution string list |                                          |
| definitionList    | string[] | Stream resolution value list  |                                          |

**Response:**

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





### LiveStream - Get Bitrate List

Gets a list of bitrate supported by each platform for each resolution

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLiveBitrateList

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description       | Content                                  |
| ---------- | ------ | ----------------- | ---------------------------------------- |
| platformId | int    | Platform Index    | 0: rtmp  1: weibo  2: facebook  3: youtube  4: pi |
| definition | string | Stream resolution | eg.  3648*2280                           |

**Output params:**     

| Field       | Type     | Description     |
| ----------- | -------- | --------------- |
| platformId  | int      | Platform Index  |
| bitrate     | string   | seleted bitrate |
| bitrateList | string[] | bitrate list    |



**Response:**

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





### LiveStream - Set Bitrate

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveBitrate

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description       | Content                                  |
| ---------- | ------ | ----------------- | ---------------------------------------- |
| platformId | int    | Platform Index    | 0: rtmp;1: weibo;2: facebook;3: youtube;4: self |
| definition | string | Stream resolution | eg.  3648*2280                           |
| bitrate    | string | bitrate           | eg.  80 Mbps                             |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setLiveBitrate ",
	"name": "camera._setLiveBitrate ",
	"state": "done"
}
```





### LiveStream - Start Live Streaming

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveStart

**Request header param (version):**  4.11.0

**Input params:**  

| Field      | Type   | Description    | Content                                  |
| ---------- | ------ | -------------- | ---------------------------------------- |
| platformId | int    | Platform Index | 0: rtmp;1: weibo;2: facebook;3: youtube;4: self |
| type       | int    | Live type      | 0 for 8K live;  1 or normal live         |
| language   | string | language       | cn, en                                   |



**Response:**

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setLiveStart ",
	"name": "camera._setLiveStart",
	"state": "done"
}
```





### LiveStream - Stop Live Streaming

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveStop

**Request header param (version):**  4.11.0

**Input params:**  None

**Output params:**  None

**Response:**  

```Json
{
	"id": "camera._setLiveStop ",
	"name": "camera._setLiveStop ",
	"state": "done"
}
```





### LiveStream - Get Pro Options

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getProfessionalEx

**Request header param (version):**  5.0.0

**Input params:**  

| Field | Type | Description    | Content |
| ----- | ---- | -------------- | ------- |
| proId | int  | Platform Index | 0       |

**Output params:**  

| Field    | Type | Description  | Content |
| -------- | ---- | ------------ | ------- |
| ev       | int  | EV           |         |
| iso      | int  | ISO          |         |
| distance | int  | Stitch Focus |         |

**Response:**

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





### LiveStream - Set Pro Options

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setProfessionalEx

**Request header param (version):**  5.0.0

**Input params:**  

| Field | Type   | Description | Content                                  |
| ----- | ------ | ----------- | ---------------------------------------- |
| type  | string | Type        | `ev` ; `distance` ; `iso`                |
| value | int    | value       | `ev`: 0 ~ 8   `distance`: 0 ~ 100   `iso`: 0 ~ 6 |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setProfessionalEx ",
	"name": "camera._setProfessionalEx ",
	"state": "done"
}
```



### LiveStream - Set Self-stream Name

**Call Mode:** http+json,Post

**Request url:**   http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLivePushId

**Request header param (version):**  4.14.0

**Input params:**  

| Field      | Type   | Description                              |
| ---------- | ------ | ---------------------------------------- |
| pushId     | string | Self-stream Name                         |
| platformId | int    | 0: rtmp  1: weibo  2: facebook  3: youtube  4: self |

**Output params:**   None

**Response:**

```Json
{
	"id": "camera._setSettingLedParams",
	"name": "camera._setSettingLedParams",
	"state": "done"
}
```







### LiveStream - Reset State

**Function:**  If stream error, you need to call this interface and set the state to not streaming.

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveCode

**Request header param (version):**  4.11.0

**Input params:**  

| Field | Type | Description | Content |
| ----- | ---- | ----------- | ------- |
| code  | int  | error code  | 0       |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setLiveCode ",
	"name": "camera._setLiveCode",
	"state": "done"
}
```





### LiveStream - Heartbeat Field

 Parsing:

0: idle

1: streaming

2: create live failed for normal stream.

3: create live failed for 8K stream.

< 0: live error





### LiveStream - Get Platform

Get current platform ID

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLiveShowPlatform

**Request header param (version):**  4.11.0

**Input params:**  None

**Output params:**  None

**Output params:**  None
| Field      | Type | Description    | Content                                  |
| ---------- | ---- | -------------- | ---------------------------------------- |
| platformId | int  | Platform Index | 0: rtmp  1: weibo  2: facebook  3: youtube  4：self |

**Response:**

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





### LiveStream - Set Auto Resolution

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveAutodefinition

**Request header param (version):**  4.11.0

**Input params:**  

| Field          | Type    | Description             | Content                                  |
| -------------- | ------- | ----------------------- | ---------------------------------------- |
| platformId     | int     | Type                    | 0: rtmp  1: weibo  2: facebook  3: youtube  4：self |
| autoDefinition | boolean | is open auto resolution |                                          |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setLiveAutodefinition",
	"name": "camera._setLiveAutodefinition",
	"state": "done"
}
```





### LiveStream - Reset Resolution

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._resetResolution

**Request header param (version):**  4.11.0

**Input params:**  None

**Output params:**  None

```json
{
	"id": "camera._resetResolution",
	"name": "camera._resetResolution",
	"state": "done"
}
```

**Response:**  





### LiveStream - Restart Streaming

**Function:**  This interface can be used when the code returned by the heartbeat interface is -3006 

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._restartLive

**Request header param (version):**  4.11.0

**Input params:**  None

**Output params:**  None

**Response:**  

```json
{
	"id": "camera._restartLive ",
	"name": "camera._restartLive ",
	"state": "done"
}
```



### LiveStream - Get PilotSteady

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getLiveOrientationState

**Request header param (version):**  4.14.0

**Input params:**  None

**Output params:**  

| Field            | Type | Description                       |
| ---------------- | ---- | --------------------------------- |
| steadyValue      | int  | 0 for Off;   1 for On             |
| orientationValue | int  | 0 for Fixed;  1 for Follow Camera |

**Response:**

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





### LiveStream - Set PilotSteady

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setLiveOrientationState

**Request header param (version):**  4.14.0

**Input params:**  

| Field | Type | Description                       |
| ----- | ---- | --------------------------------- |
| type  | int  | 0 for Off;   1 for On             |
| value | int  | 0 for Fixed;  1 for Follow Camera |

**Output params:**   None

**Response:**

```Json
{
	"id": "camera._setLiveOrientationState",
	"name": "camera._setLiveOrientationState",
	"state": "done"
}
```





### LiveStream - Get Options

**Function:**  5.2.0 version added to get the Live options

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.getOptions

**Request header param (version):**  5.0.0

**Input params:**  

| Field       | Type         | Description                              |
| ----------- | ------------ | ---------------------------------------- |
| optionNames | string Array | Option arrary, refer to **Live options table**. |

**Output params:**  

| Field   | Type                | Description                      |
| ------- | ------------------- | -------------------------------- |
| options | map<string, Object> | refer to **Live options table**. |

**Response:**

**Live options table**:

| Option                     | Type         | Description                              |
| -------------------------- | ------------ | ---------------------------------------- |
| `_live$rtmp$encode`        | string       | Selected codec for rtmp stream. Default is H.264 |
| `_live$rtmp$encodeSupport` | string Array | Supported codecs for rtmp stream, read only。  `[“H.264”,“H.265”]` |
| `_live$self$encode`        | string       | Selected codec for self stream. Default is H.264 |
| `_live$self$encodeSupport` | string Array | Supported codecs for self stream, read only。  `[“H.264”,“H.265”]` |


### LiveStream - Set Options

**Function:**  5.2.0 version added, set Live options

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.setOptions

**Request header param (version):**  5.0.0

**Input params:**  

| Field   | Type                | Description                     |
| ------- | ------------------- | ------------------------------- |
| options | map<string, Object> | refer to **Live options table** |

**Output params:**  None

**Response:**  



### LiveStream - Set Facebook Live Type (private interface)

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.facebook.setPubType

**Request header param (version):**  

**Input params:**  

| Field   | Type   | Description                              |
| ------- | ------ | ---------------------------------------- |
| pubType | int    | 0 for TimeLine;  1 for Pages;   2 for Groups |
| pubId   | string | id                                       |

**Output params:**   None

**Response:**

```json
{
	"id": " _live.facebook.setPubType",
	"name": " _live.facebook.setPubType",
	"state": "done"
}
```





### LiveStream - Get Facebook Pages (private interface)

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.facebook.pages

**Request header param (version):**  

**Input params:**  None

**Output params:**  Need to call **Query Progress** to get results

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| curId | string | current ID                               |
| list  | Array  | Pages data. If you can't get this data, please retry. |
| id    | string | First page ID                            |
| name  | string | First page name                          |

**Response:**

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





### LiveStream - Get Facebook Groups (private interface)

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.facebook.groups

**Request header param (version):**  

**Input params:**  None

**Output params:**  Need to call **Query Progress** to get results

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| curId | string | current ID                               |
| list  | Array  | Groups data. If you can't get this data, please retry. |
| id    | string | First group id                           |
| name  | string | First group name                         |

**Response:**

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





### LiveStream - Get Facebook Publication Types (private interface)

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**command ID:**  _live.facebook.pubTypes

**Request header param (version):**  

**Input params:**  

| Field    | Type   | Description |
| -------- | ------ | ----------- |
| language | string | zh, en      |

**Output params:**    

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| curId | string | current ID                               |
| list  | Array  | Groups data. If you can't get this data, please retry. |
| id    | string | First group id                           |
| name  | string | First group name                         |

**Response:**

```json
{
  	"curType": 1
	"list":[{
     	"type": 1
     	"name":"string"    
	}] 
}
```





### System - Get Nadir Logo Config

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getCameraWatermarkList

**Request header param (version):**  4.14.0

**Input params:**  None

**Output params:**  

| Field          | Type     | Description                              |
| -------------- | -------- | ---------------------------------------- |
| watermarkState | int      | Is show nadir logo.   0 for On;  1 for Off |
| logoSize       | int      | logo size                                |
| imageSelect    | string   | selected logo file                       |
| imgList        | string[] | Logo list                                |



### System - Set Nadir Logo Switcher

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setCameraWatermarkState

**Request header param (version):**  4.14.0

**Input params:**  

| Field          | Type | Description                              |
| -------------- | ---- | ---------------------------------------- |
| watermarkState | int  | Is show nadir logo.   0 for On;  1 for Off |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setCameraWatermarkState",
	"name": "camera._setCameraWatermarkState",
	"state": "done"
}
```





### System - Set Nadir Logo Size

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setCameraWatermarkSize

**Request header param (version):**  4.14.0

**Input params:**  

| Field         | Type | Description |
| ------------- | ---- | ----------- |
| watermarkSize | int  | 0 ~ 26      |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setCameraWatermarkSize",
	"name": "camera._setCameraWatermarkSize",
	"state": "done"
}
```





### System - Set Selected Nadir Logo

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setCameraWatermarkSelect

**Request header param (version):**  4.14.0

**Input params:**  

| Field       | Type   | Description                              |
| ----------- | ------ | ---------------------------------------- |
| imageSelect | string | logo file name.  It will use defualt logo if imageSelect is empty. |

**Output params:**   None

**Response:**

```json
{
	"id": "camera._setCameraWatermarkSelect",
	"name": "camera._setCameraWatermarkSelect",
	"state": "done"
}
```





### System - Upload Logo File

**Function:**  Upload logo file 

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._uploadWatermarkFile

**Input params:**  

| Field    | Type   | Description |
| -------- | ------ | ----------- |
| fileName | string | file name   |

**Output params:**  None

**Response:**  None



### System - Query Upload Image Progress

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getUploadProgress

**Request header param (version):**  4.14.0

**Input params:**  None

**Output params:**  

| Field     | Type   | Description                              |
| --------- | ------ | ---------------------------------------- |
| uploadMsg | string | Message of success or failed.            |
| imgPath   | string | If upload is successful, return the image path ; If the upload fails, return empty. |







### System - Get Sound Params

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getVoiceParams

**Request header param (version):**  4.12.0

**Input params:**  None

**Output params:**  

| Field              | Type    | Description                     | Content |
| ------------------ | ------- | ------------------------------- | ------- |
| micVoiceMax        | int     | Max value of Mic                | 100     |
| micVoiceMin        | int     | Min value of Mic                | 0       |
| micVoiceValue      | int     | Current value of Mic            |         |
| speakerVoiceMax    | int     | Max value of speaker            | 15      |
| speakerVoiceMin    | int     | Min value of speaker            | 0       |
| speakerVoiceValue  | int     | Current value of speaker        |         |
| pictureVoiceSwitch | boolean | Shutter sound switcher of Photo |         |
| videoVoiceSwitch   | boolean | Shutter sound switcher of Video |         |
| liveVoiceSwitch    | boolean | Shutter sound switcher of Live  |         |

**Response:**

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





### System - Set Sound Params

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setVoiceParams

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type   | Description                              |
| ----- | ------ | ---------------------------------------- |
| type  | string | `speaker`,  `mic`,  `picture` for Shutter sound switcher of Photo,  `video` for Shutter sound switcher of Video,   `live` for Shutter sound switcher of Live |
| value | int    |                                          |


**Output params:**   None

**Response:**

```json
{
	"id": "camera._setVoiceParams",
	"name": "camera._setVoiceParams",
	"state": "done"
}
```





### System - Get Fan Status

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getFanParms

**Request header param (version):**  4.12.0

**Input params:**  None

**Output params:**  

| Field | Type | Description                              |
| ----- | ---- | ---------------------------------------- |
| value | int  | 1 for Auto;   2 for always on;   0 for always off |

**Response:**

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





### System - Set Fan Status

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setFanParms

**Request header param (version):**  4.12.0

**Input params:**  

| Field | Type | Description                              |
| ----- | ---- | ---------------------------------------- |
| value | int  | 1 for Auto;   2 for always on;   0 for always off |

**Output params:**   None

**Response:**

 

```json
{
	"id": "camera._setFanParms",
	"name": "camera._setFanParms",
	"state": "done"
}
```







### System - Get Prompt Lamp Params

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._getSettingLedParams

**Request header param (version):**  4.14.0

**Input params:**  None

**Output params:**  

| Field      | Type | Description          |
| ---------- | ---- | -------------------- |
| ledPhoto   | int  | 1 for On,  0 for Off |
| ledVideo   | int  | 1 for On,  0 for Off |
| ledLive    | int  | 1 for On,  0 for Off |
| ledBattery | int  | 1 for On,  0 for Off |

**Response:**  





### System - Set Prompt Lamp Params

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._setSettingLedParams

**Request header param (version):**  4.14.0

**Input params:**  

| Field | Type | Description                              |
| ----- | ---- | ---------------------------------------- |
| type  | int  | 0 for photo led;  1 for video led;  2 for live led;  3 for battery low power led. |
| value | int  | 1 for On;  0 for Off                     |

**Output params:**  None

**Response:**  

```json
{
	"id": "camera._setSettingLedParams",
	"name": "camera._setSettingLedParams",
	"state": "done"
}
```

 



### Init Preview Stream

**Function:**  Init preview stream. Call this when exits the bottom logo

**Call Mode:**  http+json,Post

**Request url:**  http://x.x.x.x:8080/osc/commands/execute

**Command ID:**  camera._initSurfacePreview

**Request header param (version):**  4.14.0

**Input params:**  None

**Output params:**  None

**Response:**  

```json
{
	"id": "camera._initSurfacePreview",
	"name": "camera._initSurfacePreview",
	"state": "done"
}
```





### Cancel Command

Only used to call camera.takePicture after turning on hide photographer mode.



**Call Mode:**  http+json,Post

**Request url:**  < http://x.x.x.x:8080/osc/commands/_cancel>

**Request header param (version):**  may not be set

**the command ID:**  requires the id of the query results

| Field | Type   | Description |
| ----- | ------ | ----------- |
| id    | string | command id  |

**Input params:**  None

**Output params:**  None

