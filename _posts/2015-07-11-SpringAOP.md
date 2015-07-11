---
layout: post
title: "Refactoring"
date: 2015-07-11 14:30:00
comments: false
---

##Spring AOP
**SpringAOP** 典型应用之一是统计方法运行的时间。有两种方式实现：   
1.定义一个*annotation*，在需要统计时间的方法上加上这个*annotation*，方法运行之后打出运行时间    

	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface Metric {
	}
定义**Advisor**

	@Component
    public class MetricAdvisor extends   AbstractPointcutAdvisor {
    public static final long serialVersionUID = 1L;

    private final StaticMethodMatcherPointcut pointcut = new StaticMethodMatcherPointcut() {
        public boolean matches(Method method, Class<?> aClass) {
            return method.isAnnotationPresent(Metric.class);
        }
    };

    @Autowired
    private MetricInterceptor interceptor;

    public Pointcut getPointcut() {
        return this.pointcut;
    }

    public Advice getAdvice() {
        return this.interceptor;
    }
	}

**Advice**在AOP代表要执行的*Action*，**Advisor**在每个方法调用时会去判断方法上有没有Metric注释，如果有会执行interceptor中的代码。   

	@Component
	public class MetricInterceptor implements MethodInterceptor {
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        StopWatch stopWatch = new StopWatch(methodInvocation.getMethod().toGenericString());
        stopWatch.start("invocation.proceed");
        try {
            return methodInvocation.proceed();
        } finally {
            stopWatch.stop();
            System.out.println(stopWatch.prettyPrint());
        }
    }
	}
	
还要在Spring的配置文件中加上   
	    
	<bean 	class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator">
        <property name="proxyTargetClass" value="true"/>
    </bean>

**Interceptor**中执行计时操作，Spring内置了*StopWatch*。   
需要注意的是：执行方法所在的类必须是要Spring注入的。   
这种方法的优点是比较清晰，给需要加切面的方法添加注释，使代码比较清晰。    
缺点是要实现**Annotation，Advisor，Interceptor**好几个类

2.不需要annotation，直接通过指定包名，类名，方法名等给方法加切面   
	定义一个MetricAspect类，假设要在com.baidu包中的getName方法上加上切面来统计时间。   
	
	@Aspect
	@Component
	@Configuration
	@EnableAspectJAutoProxy
	public class MetricAspect {

    @Pointcut("target(com.baidu)")
    public void baidu() {

    }

    @Pointcut("execution(public * getName())")
    public void getName() {

    }

    @Around("baidu() && getName()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        try {
            return joinPoint.proceed();
        } finally {
            stopWatch.stop();
            System.out.println(stopWatch.prettyPrint());
        }
    }
	}
不需要修改Spring配置文件，只需这一个类就可以统计。   
这种方法的优点是非常灵活，可以完全不修改商业代码的逻辑。   
缺点是目前这种方式只能加到方法上，并且这样写代码不够清晰。   
另外，Advice有**Around，Before, After returning, After throwing**。如果只需要在方法前或后加的推荐只使用*Before*和*After*，只有在前后都需要加的地方才用Around。   

Reference:   
[http://blog.javaforge.net/post/76125490725/spring-aop-method-interceptor-annotation](http://blog.javaforge.net/post/76125490725/spring-aop-method-interceptor-annotation)   
[http://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html)


