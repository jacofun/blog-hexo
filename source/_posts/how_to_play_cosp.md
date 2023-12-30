---
title: （待更新）合作方框架快速启动指南
date: 2023-11-27 21:04:00
tags: programming
---

## 0.引言 ##

刚接手新工作不到两个月，就被领导安排了代码改造的需求。没过多久，一位同事离职，紧接着和我一同加入的新同事由于工作能力原因也被调离岗位，导致目前工作过分紧张，再加上本人水平有限，实在很难将框架的知识体系完整的展现出来。
<!--more-->
起初的工作接手起来十分吃力，经过改造了两个项目的代码之后，逐渐对SpringBoot框架有了些了解。COSP框架便是基于SpringBoot框架二次开发的，很多知识体系可以从SpringFramework中无缝衔接。

请注意，本内容或仅供查阅与参考。

合作方框架收到平台的POST请求后，判断appcode和trxcode后进入相应的交易进行处理，之后向通讯前置发起连接，通讯前置代替COSP向第三方发送请求，第三方响应结果经通讯前置返回给合作方框架，合作方框架处理后作为http响应返回给平台。

（流程图有空再补吧~）

## 1.向合作方框架发送POST请求 ##

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

POST请求体中的XML或名为cosp参数部分的XML解析为CospReqEntity<OmcpTeleQryDTO.OmcpIn>类的一个实例，并传递给execute方法，于是完成了报文数据的反序列化。反序列化的目的在于便于对DTO对象的属性进行操作。

至此，COSP实际上扮演了SpringBoot中RestController的角色，在execute方法内部继续对传入对象进行处理，或者与第三方通讯。

在实务中，总行平台传输给分行的报文格式是统一的，在处理过程中，一般根据合作方的要求，筛选出需要的业务数据，并组装成合作方需要的报文格式然后通讯。接到合作方返回的报文后，根据返回的内容判定交易是否成功，进而向平台反馈。

## 3.待续 ##
