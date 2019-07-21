---
layout: post
title: jsp自定义标签
category: config
tags: [tld]
keywords: TLD文件配置详解,TLD,config,jsp自定义标签
---
## 前言
在jsp的开发中，为了提高开发效率定制标签会被频繁使用，除了常用的JSTL标签库外，往往需要在工程中自己定义符合本项目需求的定制标签，本文即详细说明如何自定义标签

## 一、配置详解
**配置样例：**
```xml
<taglib>
	<tlib-version>1.0</tlib-version>
	<jsp-version>1.2</jsp-version>
	<short-name>mytag</short-name>
	<uri>http://www.zhang.com/tag/my</uri>
	<description>MyTag</description>
	<tag>
		<name>org</name>
		<tagclass>com.zhang.mytag.OrgTag</tagclass>
		<bodycontent>empty</bodycontent>
		<attribute>
			<name>bundle</name>
			<required>false</required>
			<rtexprvalue>true</rtexprvalue>
		</attribute>
	</tag>
	<tag>
		<name>showTip</name>
		<tag-class>com.tydic.ppm.webapp.taglib.ShowTipTag</tag-class>
		<body-content>JSP</body-content>
		<description>功能提示说明使用标签</description>
		<attribute>
			<description>功能使用说明ID</description>
			<name>fnId</name>
			<required>true</required>
			<rtexprvalue>true</rtexprvalue>
			<type>java.lang.String</type>
		</attribute>
		<attribute>
			<description>默认提示</description>
			<name>defaultDesc</name>
			<rtexprvalue>true</rtexprvalue>
			<type>java.lang.String</type>
		</attribute>
	</tag>
</taglib>
```
**配置说明：**
1. <tlib-version>1.0</tlib-version>自定义标签库的版本
2. <jsp-version>1.2</jsp-version>标签库依赖jsp的版本
3. <short-name>mytag</short-name>标签简写，在引用该标签库时可再次命名标签简写，若没有命名这使用该处配置
4. <uri>http://www.zhang.com/tag/my</uri>这个标签的uri信息
5. <description>MyTag</description>标签描述
6. <tag></tag>定义Tag，可定义多个


    （1）、<name>org</name>：这个Tag的名字
    （2）、<tagclass>com.zhang.mytag.OrgTag</tagclass>
        这个Tag是由那个类实现的（这个class可以在struts.jar包中找到）
        可自定义，但必须继承Tag的基类（例如：javax.servlet.jsp.tagext.BodyTagSupport;）
    （3）、<bodycontent>empty</bodycontent>
        这个Tag可以直接结尾，不需要填写内容
        这里bodycontent有三个可选值：
        I、 jsp 标签体由其他jsp元素组成 
            如果其有jsp元素，那么标签会先解释，然后将元素的实际值传入。
            比如标签体里含有<%=attributeName%>这样子的jsp元素，此时标签会按attributeName的实际值是什么就传入什么。这个是最常用的一个。
        II、empty 标签体必须为空   
            在引用这个Tag的时候，可以<mytag:org bundle="attributeName" />，
            而不必<mytag:org bundle="attributeName" ></mytag:org> 
        III、 tagdependent 由标签解释，不带jsp转换
    （4）、<attribute> </attribute>这里标识的是这个Tag的一个参数，根据需要可配置多个
        I、<name>bundle</name>这个参数的名字
        II、<required>false</required>这个参数是否是必填相
            如果为true则必须写这个参数，否则会报错
        III、<type>java.lang.String</type>属性值类型
## 二、标签类
在上述tagclass中，配置了标签类，那么就需要有相对应的类去实现了，这里就需要在本地新建一个类来完成了，一个tag需要对应一个类，这里用showTip示例:
```java

/**
* 可继承BodyTagSupport
* 重载BodyTagSupport类的方法
* 编写标签对应的实现类时，需要重载BodyTagSupport类的几个方法:doStartTag(),setBodyContent(),doInitBody(),doAfterBody(),doEndTag()
* 执行顺序:doStart(),doInitBody(),setBodyContent(),doAfterBody(),doEndTag
*
*/
public class ShowTipTag extends BodyTagSupport {
	/**
	 * 
	 */
	private static final long serialVersionUID = -5502864865570248395L;

	private static final Log log = LogFactory.getLog(ShowTipTag.class);

	private String fnId; // 功能使用说明ID
	private String defaultDesc="请配置功能使用说明！"; // 默认提示
	private String disabled;

	

	public int doEndTag() throws JspException {
		try {
			if (StringUtils.isBlank(fnId)) {
				return EVAL_PAGE;
			}
		    DictBeanVO beanVo = getTipByFnId();
			StringBuffer options = new StringBuffer("");
			String title = "";
			if(beanVo==null){
				title = defaultDesc;
			}else{
				if(StringUtils.isBlank(beanVo.getText())){
				title = defaultDesc;	
				}else{
				title = beanVo.getText();	
				}
			}
			StringBuilder sb = new StringBuilder();
			sb.append("<").append(fnId).append(">").append(title);
			
			HttpServletRequest re = (HttpServletRequest) this.pageContext.getRequest();
			options.append("<img").append(" id=\"").append(fnId).append("\" ");
			options.append(" class=\"").append("tip").append("\" ");
			options.append(" alt=\"").append("").append("\" ");
			options.append(" src=\"").append(re.getContextPath()).append("/images/tip.png").append("\" ");
			options.append(" onmouseover=\"");
			options.append("$('#"+fnId+"').dicTips({");
			options.append("title : '"+sb.toString()+"',");
			options.append("direction : 'e',");
			options.append("spaceX : 0,");
			options.append("spaceY : 0,");
			options.append("opacity : 1,");
			//options.append("width : 100,");
			options.append("showTime : 500,");
			options.append("hideTime : 1000");
			options.append("})").append("\" ");
			options.append("/>");
			try {
				this.pageContext.getOut().print(options.toString());
			} catch (IOException ex) {
				log.error(ex.getMessage());
			}
			
		} catch (Exception e) {
			log.error(e);
		}
		return EVAL_PAGE;
	}
	
	public DictBeanVO getTipByFnId() {
	      DictBeanVO beanVO = null;
		try {
			DataDictService dataDictService = (DataDictService) SpringContextUtils.getBean("service_prd_DataDictServiceImpl");
			beanVO = dataDictService.getTipByFnId(fnId);
		} catch (Exception ex) {
			log.error("ShowTipTag.getTipByFnId SpringContextUtils.getBean('service_prd_CodeServiceImpl') ", ex);
		}

		return beanVO;
	}


	public String getFnId() {
		return fnId;
	}

	public void setFnId(String fnId) {
		this.fnId = fnId;
	}

	public String getDefaultDesc() {
		return defaultDesc;
	}

	public void setDefaultDesc(String defaultDesc) {
		this.defaultDesc = defaultDesc;
	}

	public String getDisabled() {
		return disabled;
	}

	public void setDisabled(String disabled) {
		this.disabled = disabled;
	}
	
}

```
**说明：** 

配置中的attribute的name和类中的属性名称和类型对应

## 三、标签使用
在web.xml中加载自定义标签
```xml
<taglib>
  <taglib-uri>http://www.zhang.com/tag/my</taglib-uri>
  <taglib-location>/WEB-INF/my.tld</taglib-location>
</taglib>
```
在jsp头文件中声明自定义标签
```jsp
<%@ taglib prefix="my" uri="/www.zhang.com/tag/my" %>
```
这里有个prefix是可选的，，若配置了则标签名就是这个，若没有则标签名为TLD配置中的short-name

上述加载声明好后，，就可以在jsp中使用给标签了，示例：
```jsp
<my:showTip fnId="123"/>
<div class="hideA">
	    <input type="hidden" id="isTemple" name="isTemple"/>
	    <input type="hidden" id="offerTemplateZId" name="offerTemplateZId" />
</div>
```