---
layout: post
title: "JAXB CDATA"
date: 2015-06-26 10:15:00
comments: false
---
##JAXB CDATA
使用**JAXB**来*Marshal*和*Unmarshal* xml时比较麻烦的地方之二就是**CDATA**。  

有好几种方法可以处理CDATA的问题，比较好的一种是**XMLSerializer**。  

举个栗子：   

	JAXBContext ctx = JAXBContext.newInstrance(Customer.class)
	Marshaller marshaller = ctx.createMarshaller();
	
	StringWriter writer = new StringWriter();
	marshaller.marshal(new Customer(), getXmlSerializer(writer);
	writer.toString();
	
	
	XMLSerializer getXmlSerializer(StringWriter writer) {
	OutputFormat outputFormat = new OutputFormat();
	outputFormat.setCDataElement(new String[] {
	    "http://www.baidu.com/customer^email",
	    "^street});
	outputFormat.setIndenting(true);
	return new XMLSerializer(writer, outputFormat);
	}
	}

marshal方法的一个重载是第二个参数是ContentHandler接口，而XmlSerializer实现了这个接口。构造XmlSerializer又需要**Writer**和**OutputFormat**。可以把需要加CDATA的XmlElement加到OutputFormat中。   
这里会把email和street最终加上CDATA，**^**之前的是Xmlns。   

更好的实现可以反射检测哪个XmlElement需要加CDATA，这里只是简单hard code。


   
   			



