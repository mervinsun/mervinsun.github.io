---
layout: post
title:  "java整合groove实战—xml文件解析"
date:   2012-04-15 20:10:15
categories: java
excerpt: 
---

* content
{:toc}


## 序

在JAVA中执行groovy有多种方式，个人觉得最舒服的莫过于使用IDE插件创建groovy工程，这样可以直接在java代码中引用groovy的类。因groovy的IDE插件会自动对groovy脚本编译，并生成相应的class放于工程的output目录下，这样在编译时就不会报错，直接使用groovy类了。这种方式在协作开发时就有了不便，这要求团队中其它人也要使用groovy插件，否则别人的环境中就会编译报错，且个人觉得目前eclipse对groovy的支持不是很好，如果没有创建groovy工程，那么脚本默认不会自动编译（可能是我个人没有找到好的方式），所以如果要在老的项目中使用一些groovy实现，那么可能还是需要使用脚本引擎或者groovy的GroovyShell或GroovyClassLoader。

本文是JAVA与groovy交互的例子，通过简单的解析xml配置的例子，涉及了JAVA与groovy中互相调用、使用groovy解析xml的方法。

---

## 一、工程准备


1. 环境

	需下载groovy的发布包,解压后将其lib下如下jar加入工程的classpath,本文使用的是groovy1.8.6版本。
    
    ![ruby-gems]({{ "/css/pics/20151220/groove-1.png"}})


1. 代码

    代码结构：

    ![ruby-gems]({{ "/css/pics/20151220/groove-2.png"}})    

---

## 二、代码

先给出解析的xml配置文件，一个简单的用户管理配置users.xml，主要包含两个elements，分别是users、roles,即用户和角色：

    <?xml version="1.0" encoding="UTF-8"?> 
    <userMgr> 
    	<users role="administrator"> 
    		<user name="root"> 
   				<authType>local</authType> 
   				<type>admin</type> 
    		</user> 
    		<user name="domain"> 
    			<authType>LDAP</authType> 
    			<type>admin</type> 
   			</user> 
    	</users> 
     
    	<roles> 
   			<role name="administrator"> 
    			<auth>user</auth> 
    			<auth>res</auth> 
    		</role> 
    	</roles> 
    </userMgr> 

再给出解析该xml的groovy脚本parseUserXml.groovy，该脚本通过JAVA client传递的输入流解析xml并打印输出各节点，并且将xml转为java模型存储到JAVA的缓存中,代码有些冗长但多数代码是在做打印输出和模型转换，而具体的xml的解析实际是很简单的: 

    import groovy.CacheXml; 
    import model.*; 
     
    def doParse(InputStream ins){ 
 
    	// 使用输入流构造XmlParser 
    	def userMgr = new XmlParser().parse(ins) 
 
    	println "---------- users ----------" 
    	println "role:" + userMgr.users[0].attribute("role") 
 
    	// 对于非遍历的情况类似数据的操作，可直接用下标引用 
    	userMgr.users[0].user.each { 
        	println "\nuser name: " + it.attribute("name") 
        	println "user authType: " + it.authType.text() 
        	println "user type: " + it.type.text() 
 
        	// 转换为java模型并加入缓存 
        	User user = new User(); 
        	user.setAuthType(it.authType.text()) 
        	user.setName(it.attribute("name")) 
        	user.setType(it.type.text()) 
        	CacheXml.getInstance().put(user.getName(), user); 
   		} 
 
    	println "\n---------- roles ----------" 
 
    	// 闭包嵌套 
    	userMgr.roles[0].role.each {role -> 
        	println "role type" + role.attribute("name") 
         
        	// 转换为java模型并加入缓存 
        	Role rol = new Role() 
        	rol.setName(role.attribute("name")) 
        	Set authSet = new HashSet(); 
         
        	role.each { 
            	println "auth: " + it.text() 
            	authSet.add(it.text()); 
        	} 
         
        	rol.setAuths(authSet); 
        	CacheXml.getInstance().put(rol.getName(), rol) 
    	} 
    }

User.java和Role.java只是两个JavaBean，其属性对应着xml中相关配置，实现时只是重写了toString方法（使用eclipse的缺省实现），这里省略其代码。CacheXml是一个map缓存用来缓存配置转换后的模型对象：

    import java.util.HashMap; 
    import java.util.Map; 
    import java.util.Map.Entry; 
    import java.util.Set; 
     
    public class CacheXml 
    { 
    	private static CacheXml cx = null; 
 
    	private static Map<String, Object> map1 = new HashMap<String, Object>(); 
 
    	public static synchronized CacheXml getInstance() 
    	{ 
        	if (cx == null) 
        	{ 
            	return new CacheXml(); 
        	} 
        	return cx; 
    	} 
 
    	public static void put(String key, Object value) 
    	{ 
       		map1.put(key, value); 
    	} 
 
    	public String toString() 
    	{ 
        	StringBuilder sb = new StringBuilder(); 
 
        	Set<Entry<String, Object>> set = map1.entrySet(); 
 
        	for (Entry<String, Object> entry : set) 
        	{ 
            	sb.append("key:" + entry.getKey() + ",value:" + entry.getValue() 
                    + "\n"); 
        	} 
 
        	return sb.toString(); 
    	} 
    } 

最后给出调用groovy的JAVA client

    package groovy; 
     
    import java.io.IOException; 
    import java.io.InputStream; 
    import java.io.InputStreamReader; 
     
    import javax.script.Invocable; 
    import javax.script.ScriptEngine; 
    import javax.script.ScriptEngineManager; 
    import javax.script.SimpleBindings; 
     
    public class XmlParserWithEngineManager 
    { 
 
    	public static void main(String[] args) 
    	{ 
        	// 使用java script API获取脚本引擎 
        	ScriptEngineManager sem = new ScriptEngineManager(); 
        	ScriptEngine engine = sem.getEngineByName("groovy"); 
 
        	// 绑定入参 
        	SimpleBindings bindings = new SimpleBindings(); 
        	InputStream inXml = XmlParserWithEngineManager.class 
                	.getResourceAsStream("/users.xml"); 
        	bindings.put("ins", inXml); 
 
        	InputStream in = null; 
        	try 
        	{ 
            	in = XmlParserWithEngineManager.class 
                    	.getResourceAsStream("/groovy/parseUserXml.groovy"); 
             
            	Invocable inv = (Invocable) engine; 
             
            	engine.eval(new InputStreamReader(in), bindings); 
            	inv.invokeFunction("doParse", new Object[] { inXml }); 
             
            	System.out.println("---------- IN JAVA ----------\n" 
                        	+ CacheXml.getInstance().toString()); 
             
        	} 
        	catch (Exception ex) 
        	{ 
            	// TODO
            	ex.printStackTrace(); 
        	} 
        	finally 
        	{ 
            	if (in != null) 
            	{ 
                	try 
                	{ 
                    	in.close(); 
                	} 
                	catch (IOException e) 
                	{ 
                    	// TODO 
                    	e.printStackTrace(); 
                	} 
            	} 
            	if (inXml != null) 
            	{ 
                	try 
                	{ 
                    	inXml.close(); 
                	} 
                	catch (IOException e) 
                	{ 
                    	// TODO 
                    	e.printStackTrace(); 
                	} 
            	} 
        	} 
    	} 
    } 

## 三、运行 ##

运行XmlParserWithEngineManager类，有如下控制台输出：

    ---------- users ----------
    role:administrator

    user name: root
    user authType: local
    user type: admin

    user name: domain 
    user authType: LDAP 
    user type: admin 
 
    ---------- roles ----------
    role typeadministrator 
    auth: user 
    auth: res 

    ---------- IN JAVA ---------- 
    key:root,value:User [name=root, authType=local, type=admin] 
    key:administrator,value:Role [name=administrator, auths=[res, user]] 
    key:domain,value:User [name=domain, authType=LDAP, type=admin]