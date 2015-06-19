---
layout: post
title: "JUnit Parameterized"
date: 2015-06-19 14:00:00
comments: false
---

##JUnit Parameterized
**Junit4**以后引入了***Parameterized***，方便在大量测试数据时减少代码量。   

1.使用annotation

	@RunWith(Parameterized.class)
	public class ParameterTest {
	    @Parameterized.Parameter(value = 0)
	    public int expectedValue;

	    @Parameterized.Parameter(value = 1)
	    public int actualValue;

	    @Parameterized.Parameters({index}: name = "expected: <{0}>, but: <{1}>")
	    public static Collection<Object[]> data() {
	        return Arrays.asList(new Object[][] {
	            {0, 0}, {1, 1}, {2, 1}, {3, 2}});
	    }

	    @Test
	    public void should_test_value() {
	        assertThat(actualValue, is(expectedValue));
	    }
	}

2.使用Constructor   

	@RunWith(Parameterized.class)
	public class ParameterTest {
	    private int actualValue;
	    private int expectedValue;

	    @Parameterized.Parameters({index}: name = "expected: <{0}>, but: <{1}>")
	    public static Collection<Object[]> data() {
	        return Arrays.asList(new Object[][]{
	                {1, 1}, {1, 2}, {2, 1}, {2, 2}
	        });
	    }

	    public ParameterTest(int actualValue, int expectedValue) {
	        this.actualValue = actualValue;
	        this.expectedValue = expectedValue;
	    }

	    @Test
	    public void should_test_value() {
	        Assert.assertThat(actualValue, is(expectedValue));

	    }
	}

>使用annotaion时，field必须是public   
>第一个参数会默认加value=0，所以acutalValue后面的括号可以省略   
>在Parameters后面括号里的只会起到显示说明的作用，{index}代表第几个测试数据，{0}代表测试数据中的第一个值，{1}代表第二个，依次类推   
>测试时，actualValue和expectedValue会依次被赋予测试数据中的值，测试数据不限于两个值






