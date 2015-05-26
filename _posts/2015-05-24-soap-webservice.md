---
layout: post
title: "SOAP Webservice"
date: 2015-05-24 15:45:00
comments: false
---

##SOAP Webservice

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
	   
>在这个文件中定义了两个接口方法：   
>1.***deposit***，参数是*depositRequest*，实现是*account*，返回是*status*   
>2.***withdraw***，参数是*withdrawRequest*，现实是*account*，返回是*status*

###客户端如何工作
1.**SoapUI**，把wsdl文件导入SoapUI，软件会自动解析各个操作，可以直接给Server发送请求，接收Server的返回。也可以写SoapUI的Project，实现自动化测试。  
2.**浏览器插件**，例如Firefox上RestfulClient，Chrome上的PostMan，把soap请求直接通过Http post给Server，接收Server的返回。*如果当前是跑在Http协议上。*   
3.**CXF**，如果在Java代码中使用，可以使用CXF提供的库把wsdl转换成Java接口类，例如可以添加一个ant任务转换wsdl文件。

	<target name="wsldConverter" description="Convert wsdl to java">
	  <java classname="org.apache.cxf.tools.wsdlto.WSDLToJava" fork="true">
	    <arg value="-verbose"/>
	    <arg value="-p"/>
	    <arg value="package.name"/>
	    <arg value="-client"/>
	    <arg value="-d"/>
	    <arg value="dest.dir"/>
	    <arg value="bank.wsdl"/>
	    <classpath>
	      <path refid="cxf.classpath"/>
	    </classpath>
	  </java>
	</target>
>*-p*：指定生成文件的包名   
>*-client*：生成客户端代码   
>*-d*：生成文件的位置   
>*bank.wsdl*：指定wsdl文件   
>*classpath*：cxf库文件存放的路径   
>*生成Java代码以后，就可以向调用本地方法以后和Server通信了。同步方法，返回之前程序会阻塞*   

4.**Savon**，如果要写Cucumber测试，可以使用wsdlToRuby把wsdl文件转换为Ruby文件，类似于Java。更简单的方法是使用savon或者wsdlDriver直接写一个SoapClient。下面是使用Savon的例子。

	require 'savon'
	
	class BankClient
	  def initialize(url = 'bank.wsdl')
	     @client = savon.client(
	       :wsdl => url,
	       :open_timeout => 10,
	       :read_timeout => 10,
	       :log => false)
	  end
	  
	  def deposit(name, amount)
	    status = @client.call(:deposit, :message => {:name => name, :amount => amount)
	    rescue Savon::SOAPFault => e
	      pending e.message
	  end
	  
	  def withdraw(name, amount)
	    status = @client.call(:withdraw, :message => {:name => name, :amount => amount)
	    rescue Savon::SOAPFault => e
	      pending e.message
	  end
	end
	
	client = BankClient.new
	client.deposit("Joe", 20)
	client.withdraw("Joe", 10)

