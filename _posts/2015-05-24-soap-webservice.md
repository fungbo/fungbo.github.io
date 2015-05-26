---
layout: post
title: "SOAP Webservice"
date: 2015-05-24 15:45:00
comments: false
---

***SOAP Webservice***

与**Restful Webservice**使用标准的`HTTP(PUT/GET/POST/DELETE)`不同的是，**SOAP Webservice**通过定义自己个性化的接口方法来抽象Web服务。**Client**使用时直接调用**Server**定义好的接口方法，就像调用自己的方法一样来和**Server**进行通信，是RPC的一种。   

**SOAP**是简单对象接入协议(*simple object access protocol*), 是一种基于XML的协议，被用来在**Client**和**Server**之间交换数据。**SOAP**可以运行在多种传输协议之上，最广泛的就是`HTTP`。   

**WSDL**是Web服务描述语言(*web services description language*),它将Web服务描述定义为一组服务访问接口，**Client**可以通过这些服务访问接口与**Server**传输数据。

*SOAP是用来最终完成Web服务调用的，而WSDL则是用来描述如何使用SOAP来调用Web服务的*   

下面是一个WSDL文件的例子：  

	<?xml version='1.0' encoding='UTF-8'?>
	<wsdl:definitions xmlns:wsdl="http://schemas.xmlsoap.org/wsdl">
	<wsdl:types>
	  <schema elementFormDefault="qualified>
	    <element name="account">
			<complexType>
			  <sequence>
			    <element name="name" type="xsd:string"/>
			    <element name="amount" type="xsd:double"/>
			  </sequence>
			</complexType>
		</element>
		<element name="status" type="xsd:boolean"/>
	  <schema
	</wsdl:types>  
	
	<wsdl:message name="depositRequest">
	  <wsdl:part elment="impl:account" name="parameters"/>
	</wsdl:message>
	<wsdl:message name="withdrawRequest">
	  <wsdl:part elment="impl:account" name="parameters"/>
	</wsdl:message>
	<wsdl:message name="response">
	  <wsdl:part elment="impl:status" name="parameters"/>
	</wsdl:message>

	
	<wsdl:portType name="BankService">
	  <wsdl:operation name="deposit">
	    <wsdl:input message="impl:depositRequest" name="depositRequest"/>
	    <wsdl:output message="impl:response" name="depositResponse"/>
	  </wsdl:operation>
	  <wsdl:operation name="withdraw">
	    <wsdl:input message="impl:withdrawRequest" name="withdrawRequest"/>
	    <wsdl:output message="impl:response" name="withdrawResponse"/>
	  </wsdl:operation>
	</wsdl:portType>
	
	<wsdl:binding name="BankBinding" type="impl:BackService">
	  .......
	</wsdl:binding>
	
	<wsdl:service name="BankServiceService">
	  <wsdl:port binding="impl:BackBinding" name="accountOperating">
	    <wsdlsoap:address location="http://locahost:8080/Bank/account/operating"/>
	  </wsdl:port>
	</wsdl:service>
	</wsdl:definitions>
	   