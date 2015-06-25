---
layout: post
title: "JAXB Xmlns"
date: 2015-06-25 18:00:00
comments: false
---

##JAXB Xmlns  
使用**JAXB**来*Marshal*和*Unmarshal* xml时比较麻烦的地方之一就是Xmlns。

拿个Spring.xml当例子，如果要Marshal成下面这样，应该怎么实现   
[Spring.xml]
   
	<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

		<context:component-scan base-package="com.tw"/>

    	<mvc:annotation-driven/>
	</beans>

简单起见，不考虑component-scan和annotation-driven的内容，当成String处理，对象可以这么写    
[Beans.java]

	@XmlRootElement(name = "beans")
	@XmlAccessorType(XmlAccessType.FIELD)
	public class Beans {
    @XmlElement(name = "component-scan", namespace = "http://www.springframework.org/schema/context")
    private ComponentScan componentScan;
    @XmlElement(name = "annotation-driven", namespace = "http://www.springframework.org/schema/mvc")
    private String annotationDriven;

    public void setComponentScan(ComponentScan componentScan) {
        this.componentScan = componentScan;
    }

    public void setAnnotationDriven(String annotationDriven) {
        this.annotationDriven = annotationDriven;
    }
	} 
[ComponentScan.java]
	
	@XmlAccessorType(XmlAccessType.FIELD)
	public class ComponentScan {
    @XmlAttribute(name = "base-package")
    private String basePackage;
    @XmlValue
    private String value;

    public void setBasePackage(String basePackage) {
        this.basePackage = basePackage;
    }

    public void setValue(String value) {
        this.value = value;
    }
	}
	
加Xmlns有好几种方法，其中一种是加一个**package-info.java**文件
  
	@XmlSchema(
        elementFormDefault = XmlNsForm.QUALIFIED,
        namespace = "http://www.springframework.org/schema/beans",
        xmlns={@XmlNs(prefix="", namespaceURI="http://www.springframework.org/schema/beans"),
        @XmlNs(prefix = "xsi", namespaceURI = "http://www.w3.org/2001/XMLSchema-instance"),
        @XmlNs(prefix = "context", namespaceURI = "http://www.springframework.org/schema/context"),
        @XmlNs(prefix = "mvc", namespaceURI = "http://www.springframework.org/schema/mvc")}
	)

	package com.tw.jaxb;

	import javax.xml.bind.annotation.XmlNs;
    import javax.xml.bind.annotation.XmlNsForm;
    import javax.xml.bind.annotation.XmlSchema;

然后就是schemaLocation,可以通过给JaxbMarshaller设置属性来达到   

	JAXBMarshaller<Beans> marshaller =JAXBMarshaller.getInstance(Beans.class);
        	marshaller.setProperty(Marshaller.JAXB_SCHEMA_LOCATION, "http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd "
                + "http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd "
                + "http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd");

最后的效果是

	<?xml version="1.0" encoding="UTF-8"?>
	<beans
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <context:component-scan base-package="com.tw"></context:component-scan>
    <mvc:annotation-driven></mvc:annotation-driven>
	</beans>
	
虽然长的不是完全一样，但是内容是完全一样的，绝对可以Work，呵呵
 

