#KS3 SDK For Node.js使用指南
---

##SDK下载地址
* 下载地址：[https://github.com/StoneRen/ks3-node-sdk.git](https://github.com/StoneRen/ks3-node-sdk.git)
* 主要提供[`KS3 nodejs SDK`](#ks3-nodejs-sdk说明)和[`命令行工具`](#命令行工具说明)

## KS3 nodejs SDK说明
###概述
该SDK为NodeJs开发者提供使用金山云存储服务的入口，基于金山云存储官方API构建。使用该SDK将数据安全的存储到金山云存储上会变得非常的便捷。   
SDK主要包含以下几个方面：
* 生成授权(token)；
* bucket相关操作；
* object相关操作；
* 上传下载操作（包括大文件的分块上传下载操作）；
* [命令行工具](#命令行工具)（主要针对大文件的上传和下载提供断点续传）。

###快速入门
开发前准备：   
* NodeJs 0.8.0(包含)以上版本；   
* 获取秘钥，参见[AccessKeyID和AccessKeySecret](http://ks3.ksyun.com/doc/api/index.html)

1、安装：

```
npm install ks3
```
或者将代码下载到你本地：
```
git clone https://github.com/StoneRen/ks3-node-sdk.git
```

2、运行：   

```
var KS3 = require('ks3');
// AK = AccessKeyID; SK = AccessKeySecret
var ks3 = new KS3(AK,SK);
ks3.bucket.put({Bucket: 'mybucket'}, function() {
    // 处理逻辑
})
```

### SDK详细介绍

####方法摘要
* **bucket：对Bucket的所有相关操作，包含以下可调用方法：**
    - [Create Bucket](#create-bucket:) : 创建一个新的Bucket   
    - [Delete Bucket](#delete-bucket:) : 删除指定Bucket     
    - [list Objects](#list-objects:) : 枚举Bucket内的Object   
    - [Head Bucket](#head-bucket:) : 验证指定的Bucket是否存在     
    - [GET Bucket ACL](#get-bucket-acl:) : 获取Bucket的ACL(有关ACL请参阅[官方文档](http://doc.ksyun.com/doc/api/index.html#service))   
    - [PUT Bucket ACL](#put-bucket-acl:) : 设置Bucket的ACL  
    - [GET Bucket Logging](#get-bucket-logging:) : 获取Bucket的日志信息   
    - [PUT Bucket Logging](#put-bucket-logging:) : 设置Bucket的日志信息
    - [GET Service](#get-service) : 列出用户所有的Bucket信息(名称、创建时间、所有者)


* **object：对Object的所有相关操作，包含以下调用方法：**
    - [DELETE Object](#delete-object) : 删除指定Object  
    - [GET Object](#get-object) : 下载指定的Object数据    
    - [PUT Object](#put-object) : 不超过5G的文件上传(同名Object上传，覆盖原有Object) 
    - [GET Object ACL](#get-object-acl) : 获得Object的ACL 
    - [PUT Object ACL](#put-object-acl) : 设置object的ACL 
    - [HEAD Object](#head-object) : 获取指定Object元数据  
    - [Initiate Mutitpart Upload](#initiate-mutitpart-upload) : 调用这个接口会初始化一个分块上传并且返回一个upload id, upload id 用来标识属于当前object的具体的块，并且用来标识完成分块上传或者取消分块上传  
    - [Upload Part](#upload-part) : 初始化分块上传后，上传分块接口  
    - [Complete Multipart Upload](#complete-multipart-upload) : 组装之前上传的块，然后完成分块上传。通过你提供的xml文件，进行分块组装。在xml文件中，块号必须使用升序排列。必须提供每个块的ETag值  
    - [Abort Upload](#abort-upload) : 取消分块上传  
    - [List Parts](#list-parts) : 罗列出已经上传的块  

####详情介绍
#####构造函数：
初始化一个KS3实例；      
new KS3(AK, SK);   
**参数：**   
* AK: AccessKeyID;   
* SK: AccessKeySecret


#####Create Bucket:
创建一个新的Bucket 【`ks3.bucket.put(params, cb)`】;   
**参数说明**
* params: object对象，包含：    
    * Bucket: 必须传，要创建的bucketName      
    * ACL: 非必传，bucket权限设置 [`private` || `public-read` || `public-read-write` || `authenticated-read`]; 默认为private  
* cb: 回调函数，创建成功与否都会调用此方法。参见[回调函数说明](#关于回调函数中的返回参数:) 


#####Delete Bucket:
删除指定的Bucket 【`ks3.bucket.del()`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要删除的bucketName   
* cb: 回调函数，删除成功与否都会调用此方法。参见[回调函数说明](#关于回调函数中的返回参数:) 

#####list Objects:
枚举Bucket内的Object 【`ks3.bucket.get(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 需要枚举的bucketName
    * delimiter: 是用来对Object名字进行分组的一个字符，不能为空字符串
    * maker: 指定需要列出object的开始位置
    * max-keys: 返回的object的数量，默认是1000
    * prefix: 限定返回object的前缀
* cb: 回调函数，获取list成功与否都会调用此方法；参见[回调函数说明](#关于回调函数中的返回参数:) 

#####Head Bucket:
验证指定的Bucket是否存在 【`ks3.bucket.head(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 需要验证的bucketName
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####GET Bucket ACL:
获取Bucket的ACL(有关ACL请参阅[官方文档](http://doc.ksyun.com/doc/api/index.html#service))  【`ks3.bucket.getACL(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要获取ACL的bucketName
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####PUT Bucket ACL:
设置Bucket的ACL(有关ACL请参阅[官方文档](http://doc.ksyun.com/doc/api/index.html#service))  【`ks3.bucket.puttACL(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要设置ACL的bucketName
    * ACL: bucket权限 [`private` || `public-read` || `public-read-write` || `authenticated-read`]
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####GET Bucket Logging:
获取Bucket的日志信息 【`ks3.bucket.getLogging(params, cb)`】
**参数说明**
* params: object对象，包含：
    * Bucket: 要获取Logging的bucketName
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####PUT Bucket Logging:
设置Bucket的日志信息 【`ks3.bucket.putLogging(params, cb)`】
**参数说明**
* params: object对象，包含：
    * Bucket: 要设置Logging的bucketName
    * BucketLoggingStatus: object对象，设置日志信息，可以为空对象，但必须传，当为空时表示`关闭日志`；当需要开启日志时候需要传入以下对象：
        * LoggingEnabled: object对象，包括：
            * TargetBucket: 要存放日志的bucket
            * TargetPrefix: 日志文件的前缀
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####GET Service:
列出用户所有的Bucket信息(名称、创建时间、所有者) 【`ks3.service.get(cb)`】   
**参数说明**
* cb: 回调函数，成功与否否会执行；参见[回调函数说明](#关于回调函数中的返回参数:) 


#####DELETE Object:
删除指定Object 【`ks3.objectdel(params, cb)`】   
**参数说明**  
* params: object对象，包含：
    * Bucket: 指定要操作的bucket;
    * Key: object key
* 回调函数，删除成功与否都会调用；参见[回调函数说明](#关于回调函数中的返回参数:) 

#####GET Object:
下载指定object 【`ks3.object.get(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要操作的bucketName;
    * Key: object key
    * range: 实现分块多线程下载，值为字符串，格式：`range: 'bytes=x-y'`，x、y为整数，且y>=x
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 


#####PUT Object:
不超过5G的文件上传(同名Object上传，覆盖原有Object)  【`ks3.object.put(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要操作的bucketName;
    * Key: 上传的object的名称
    * filePath: 可以为文件路径 || string || buffer
*cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####GET Object ACL:
获取指定object的ACL  【`ks3.object.getAcl(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要操作的bucketName;
    * Key: object key;
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####PUT Object ACL:
设置object的ACL  【`ks3.object.putAcl(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 要操作的bucketName;
    * Key: object key;
    ACL: object权限 [`private` || `public-read` || `public-read-write` || `authenticated-read`];
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####HEAD Object:
获取object的元数据  【`ks3.object.head()`】   
**参数说明**
* params: object对象，包含：
    * Key: object key;
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####Initiate Mutitpart Upload:
初始化一个分块上传，接口返回一个upload ID，upload ID与当前分块上传的所有块相关联，在后续的请求已经上传的块、完成分块上传、取消分块上传的时候都会用到次ID。 【`ks3.object.multitpart_upload_init(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 指定上传到该bucket下；
    * Key: object key
    * ACL: 上传的object的key;
*cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####Upload Part:
开始执行分块上传，初始化分块上传之后调用该接口  【`ks3.object.upload_part(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 指定上传到该bucket下；
    * Key: object key;
    * type: 指定Content-Type;
    * PartNumber: 当前块是第几块；
    * UploadId: upload id;
    * body: 当前上传快的内容；
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####Complete Multipart Upload:
当分块上传完成之后，通过该接口告知服务器上传完成，并通过你提供的xml文件，进行分块组装。在xml文件中，块号必须使用升序排列。必须提供每个块的ETag值 【`ks3.object.upload_complete(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 指定上传到该bucket下；
    * Key: object key;
    * UploadId: upload ID;
    * ACL: 上传的object的key;
    * body: 指定块号(必须升序排列)和每块Etag值的xml
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####Abort Upload:
取消分块上传 【`ks3.object.upload_abort(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 当前正在进行分块上传的目标bucketName;
    * Key: object key;
    * uploadId: upload ID;
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####List Parts:
罗列出分块上传已经上传成功的快 【`ks3.object.upload_list_part(params, cb)`】   
**参数说明**
* params: object对象，包含：
    * Bucket: 当前正在进行分块上传的目标bucketName;
    * Key: object key;
    * uploadId: upload ID;
* cb: 回调函数，参见[回调函数说明](#关于回调函数中的返回参数:) 

#####关于回调函数中的返回参数:
每次请求成功后都会调用用户传入的回调函数，并传入参数（所有的回调函数都会传入相同的参数）：
* rerr: 返回错误信息，如果没有错误，则返回`null`，如果有错误则返回object对象，包含:
    * code: 服务端返回的状态码；
    * error: 服务端返回的错误信息；
* data: 服务端返回后经过处理的数据（会根据配置文件中的数据类型来处理，`json || xml`）；
* res: 服务端返回结果
* result: 服务端返回的未经处理的结果   


###命令行工具
本工具主要是满足用户`上传`和`下载`文件的需求,尤其是大文件情况.在上传和下载过程中都会进行断点续传,节省时间和带宽资源.   

####准备条件
安装[nodejs](http://nodejs.org/)以及[npm](https://www.npmjs.org)

####安装
* **安装稳定版**
```
npm install -g ks3
```
* **安装最新版**
```
git clone $gitpath --depth=1
npm install -g ./ks3-node-sdk
```

####命令解释
#####ks3 reset
重置开发者配置,清空历史记录

```
ks3 reset
```


##### ks3 init   
主要执行初始化操作.在初始化过程中,可以指定开发者的`AK`和`SK`.    
命令行使用分为两种模式: 直接指定,或者进入交互模式.在`交互模式`下,会指导你填写必须的参数.如果所需参数没有提供完全,也会进入`交互模式`.

```
ks3 init -a [ak] -s [sk] -b [bucket]
```

`-a --ak` : 开发者的AK(Access Key)   
`-s --sk` : 开发者AK对应的SK(Access Key Secret)   
`-b --bucket` : 开发要使用哪个Bucket,非必须,可以在每个具体命令中指定

也可以直接输入`ks3 init`进入交互模式

##### ks3 upload   
上传文件以及文件夹.程序会根据文件大小进行`简单上传`和`分块上传`.

```
ks3 upload -p [path] -b [bucket] -k [key] --withsubdir
```
`-p --path` : 开发者制定要上传文件夹或者文件的地址   
`-b --bucket` : 指定上传文件存储的bucket   
`-k --key` : 如果上传的path为文件,则为文件名,如果上传的path是文件夹,则为文件夹名称   
`--withsubdir` : 在上传文件夹的过程中,是否上传子文件夹,默认不上传,加上本参数,上传子文件夹内容   
例如:   
把`/Users/ren/Desktop/Life\ Of\ Johnson.txt`传递到 `ks3-sdk-test` bucket根目录下

```
ks3 upload /Users/ren/Desktop/Life\ Of\ Johnson.txt
// 或者
ks3 upload "/Users/ren/Desktop/Life Of Johnson.txt"
// 然后进入交互模式 填写bucket和key

// 也可以直接传递参数
ks3 upload "/Users/ren/Desktop/Life Of Johnson.txt" -b ks3-sdk-test -k "Life Of Johnson.txt" 

```

把 `D:\Program Files (x86)\Foxmail` 整个文件夹(包括子文件夹)上传,如下:

```
ks3 upload -p "D:\Program Files (x86)\Foxmail" --withsubdir
// 进入交互模式 填写bucket和key
```
也可以直接输入`ks3 upload`进入交互模式

##### ks3 download

下载金山云存储的文件,可以进行断点续传.现在暂时只能下载文件,不能下载文件夹

```
ks3 download -b $bucket -k $key -p $localpath
```
    
`-b --bucket` : 指定下载文件所在的bucket   
`-k --key` : 指定下载文件对应的key,**注意,key不包含bucket名称**   
`-p --path` : 下载文件的本地存储地址

例如:
把`ks3-sdk-test` bucket下的 `test_download.txt` 下载到本地 `D盘` 根目录下,并且重命名为`test.txt`

```
ks3 download -k test_download.txt -p d:\test.txt
// 由于没有输入bucket名称,会进入交互模式,让你输入bucket名称

// 或者直接指定bucket名称
ks3 download -b ks3-sdk-test -k test_download.txt -p d:\test.txt
```



更多详细信息,请参阅[官方文档](http://ks3.ksyun.com/doc/api/index.html)