---
title: （好累啊更不动了┭┮﹏┭┮）Cosp请求处理流程
date: 2023-11-27 21:04:00
tags: programming
---
最近在看开放式多渠道缴费应用和COSP框架的代码，自己试图总结一下框架的基本处理流程。本内容或仅供查阅与参考。
<!--more-->
COSP收到平台的POST请求后，判断appcode和trxcode后进入相应的交易进行处理，之后向通讯前置发起连接，通讯前置代替COSP向第三方发送请求，第三方响应结果经通讯前置返回给COSP，COSP处理后作为http响应返回给平台。

（流程图有空再补吧~）

## 1.向COSP发送POST请求 ##

初学者就暂且把COSP看作一个RESTFUL Api吧，通过接受POST（一般来说）请求来响应内容。一个想要让COSP响应的基本Http请求的格式应该是这样的：

请求行格式


    POST http://your.example.domain:port/context_path/access?appcode={$appcode}&trxcode={$trxcode}&cosp={$cosp}


请求头格式

    Conten-Type:x-www-form-urlencoded

传递给cosp的数据以键值对的形式发送，键值对之间用 & 符号分隔，键与值之间用 = 符号连接。一般来说，其中cosp的参数主要以XML报文或JSON报文为主。因此，如果将请求头写成：

    Content-Type:application/xml

则需要把xml写入body部分。


**示例：一个要传送给COSP的XML报文如下：**

	<?xml version="1.0" encoding="UTF-8"?>
    <MAPS>
    	<APPROOT>
   			<DATA>![CDATA[some stuff]</DATA>
    	</APPROOT>
    </MAPS>

那么完整的POST请求可以写作：

    POST http://localhost:9080/cospnew/access?appcode=nxtele&trxcode=87061&cosp=<?xml version="1.0" encoding="UTF-8"?><MAPS><APPROOT><DATA>![CDATA[some stuff]</DATA></APPROOT></MAPS>

	Content-Type:x-www-form-urlencoded

或者

    POST http://localhost:9080/cospnew/access?appcode=nxtele&trxcode=87061

	Content-Type:application/xml

	<?xml version="1.0" encoding="UTF-8"?>
    <MAPS>
    	<APPROOT>
   			<DATA>![CDATA[some stuff]</DATA>
    	</APPROOT>
    </MAPS>

## 2.COSP处理POST请求 ##

COSP如何知道调用的是哪个服务的Controller呢？

我们在请求行中指明了appcode(nxtele)和trxcode(87061)，cosp框架的某个handler（后面补充）取到POST请求的参数后，会在如下文件中寻找相应服务：

> src/main/resources/cospconfig/TrxDefinition.json

比如：

      {
    "httpServiceInfo": "http://localhost:9080/omcp/cosp/client/teleQry",
    "invokerType": "http",
    "reqDataFormat": "xml",
    "appCode": "nxtele",
    "trxCode": "87061",
    "httpInvokerCharset": "UTF-8",
    "trxDesc": "接入http方式，下游请求http方式测试，请求报文xml格式",
    "msgWrapperBeanName": "com.icbc.cosp.omcp.model.nxTele.OmcpTeleQryDTO",
    "beginTime": "000000",
    "endTime": "235959",
    "extendConfig": "[]",
    "logDefinition": {
      "level": "2",
      "pattern": "%d %5p [%t] [%c] appSeqNo[%X{appSeqNo}] [%X{className}] - %m%n",
      "file": "${WEB_ROOT}/logs/trade/nxtele/87061.log"
    }
      },

这个微服务的appcode和trxcode是“nxtele”和“87061”。因此程序执行@PostMapping("/cosp/client/teleQry")的方法。


找到对应Controller类中的execute方法：

	@PostMapping("/cosp/client/teleQry")
    public CospRespEntity execute(@RequestBody CospReqEntity<OmcpTeleQryDTO.OmcpIn> cospReqEntity)

POST请求体中的XML或名为cosp参数部分的XML解析为CospReqEntity<OmcpTeleQryDTO.OmcpIn>类的一个实例，并传递给execute方法，于是完成了报文数据的反序列化（人话就是将XML转换为了POJO）。

好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜好累啊不想干了呜呜呜

## 3.结尾 ##
结尾在哪啊┭┮﹏┭┮