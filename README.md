
这个是一个springboot工程



配置了Windows  C:\Windows\System32\drivers\etc\hosts下面
127.0.0.1 www.cas.com   模拟cas认证中心
127.0.0.1 www.ssoclient.com   cas客户端



blog是一些网上参考的博客




cas-4.1.0是sso认证服务端的官方代码
思路：前后端分离部署的方式请借助Nginx。
app接入请用pac4j。




**************************************************************************************************************************************
sso-client-rep-master是客户端shiro-cas(1.2.4)的代码-使用了Redis作为shiro的缓存,【但是没有实现app接入和rest登录，没有实现单点登录登出！】
controller是一些测试获取信息
RestController.java是模拟rest登录获取tgt和st


**************************************************************************************************************************************
shiroTest 仅仅是测试shiro在Springboot里面集成！


**************************************************************************************************************************************
springboot_cas_shiro_forkUse_jpa是客户端shiro-cas(1.2.4)的代码-jpa数据层，实现单点登录登出，【但是没有实现app接入和rest登录！】
springboot_shiro_cas_forkUse_jpa :原作地址：https://github.com/maojun2016/springboot_cas_shiro


**************************************************************************************************************************************
client_shiro_cas_pac4j-不使用shiro-cas(1.2.4)，rest登录换成pac4j,实现单点登录登出，实现app接入和rest登录！






**************************************************************************************************************************************
testredisshiro.sql是数据库代码-mysql----/cas-4.1.0/cas-server-webapp/src/main/webapp/WEB-INF/deployerConfigContext.xml------md5加密
【<!-- 通过数据库验证身份，这个得自己去实现 admin  admin -->
	<bean id="primaryAuthenticationHandler"
		class="org.jasig.cas.adaptors.jdbc.QueryDatabaseAuthenticationHandler"
		p:dataSource-ref="dataSource" p:passwordEncoder-ref="passwordEncoder"
		p:sql="select password from tb_user where username = ? and active = 1" />

	<!-- 设置数据源 -->
	<!-- <bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
		<property name="url"
			value="jdbc:mysql://127.0.0.1:3306/db_test?useUnicode=true&amp;characterEncoding=utf8"></property>
		<property name="username" value="root"></property>
		<property name="password" value="HSJissmart"></property>
	</bean> -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
		<property name="driverClassName">
			<value>com.mysql.jdbc.Driver</value>
		</property>
		<property name="url" 
		          value="jdbc:mysql://127.0.0.1:3306/testredisshiro?useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true">
		</property>
		<property name="username" value="root" />
		<property name="password" value="HSJissmart" />
		<property name="initialSize" value="10" />
		<property name="maxActive" value="500" />
		<property name="maxWait" value="60000" />
		<property name="validationQuery">
			<value>select 1</value>
		</property>
	</bean>】

********************************************************************************************************************************
cas4.1.0返回更多信息【http://blog.csdn.net/chenhai201/article/details/50623395】
四、自定义登录后的可传递字段，方便客户端读取
　　在我们的应用场景中，客户端需要的参数不仅仅是用户名。还需要诸如userid等各类信息，那么，接下来我们就来配置获取自定义字段。
1、找到cas/WEB-INF/deployerConfigContext.xml，注释以下代码：【替换为后面】
<bean id="attributeRepository" class="org.jasig.services.persondir.support.NamedStubPersonAttributeDao"  
          p:backingMap-ref="attrRepoBackingMap" />  
  
    <util:map id="attrRepoBackingMap">  
        <entry key="uid" value="uid" />  
        <entry key="eduPersonAffiliation" value="eduPersonAffiliation" />  
        <entry key="groupMembership" value="groupMembership" />  
        <entry>  
            <key><value>memberOf</value></key>  
            <list>  
                <value>faculty</value>  
                <value>staff</value>  
                <value>org</value>  
            </list>  
        </entry>  
    </util:map>  
 
    <bean id="attributeRepository" class="org.jasig.services.persondir.support.jdbc.SingleRowJdbcPersonAttributeDao">  
        <constructor-arg index="0" ref="dataSource" />  
        <constructor-arg index="1" value="SELECT id,user_name,mobile,cid FROM user_info WHERE {0}" />  
            <property name="queryAttributeMapping">  
                <map>  
                    <entry key="username" value="user_name" />  
                </map>  
            </property>  
            <property name="resultAttributeMapping">  
                <map>  
                    <entry key="id" value="userId" />  
                    <entry key="user_name" value="username" />  
                    <entry key="mobile" value="mobile" />  
                    <entry key="cid" value="cid" />  
                </map>  
            </property>  
    </bean>  
其中的sql只需要写前半部分，如示例，entry的key代表上面sql查询的字段，value代表服务端传给客户端的参数名，客户端可以通过value取出对应的值。
2、修改cas/WEB-INF/view/jsp/protocol/2.0/casServiceValidationSuccess.jsp，增加下面这段
[plain] view plain copy
<cas:serviceResponse xmlns:cas='http://www.yale.edu/tp/cas'>  
    <cas:authenticationSuccess>  
        <cas:user>${fn:escapeXml(principal.id)}</cas:user>  
        <!-- 这段 -->  
        <c:if test="${fn:length(assertion.chainedAuthentications[fn:length(assertion.chainedAuthentications)-1].principal.attributes) > 0}">  
            <cas:attributes>  
                <c:forEach var="attr" items="${assertion.chainedAuthentications[fn:length(assertion.chainedAuthentications)-1].principal.attributes}">  
                    <cas:${fn:escapeXml(attr.key)}>${fn:escapeXml(attr.value)}</cas:${fn:escapeXml(attr.key)}>  
                </c:forEach>  
            </cas:attributes>  
        </c:if>  
        <!-- 这段 end-->  
        <c:if test="${not empty pgtIou}">  
            <cas:proxyGrantingTicket>${pgtIou}</cas:proxyGrantingTicket>  
        </c:if>  
        <c:if test="${fn:length(chainedAuthentications) > 0}">  
            <cas:proxies>  
                <c:forEach var="proxy" items="${chainedAuthentications}" varStatus="loopStatus" begin="0" end="${fn:length(chainedAuthentications)}" step="1">  
                    <cas:proxy>${fn:escapeXml(proxy.principal.id)}</cas:proxy>  
                </c:forEach>  
            </cas:proxies>  
        </c:if>  
    </cas:authenticationSuccess>  
</cas:serviceResponse>  
********************************************************************************************************************************







说明：
待完成cas 认证中心返回更多信息！！！！
经典教程
参考博客：http://m.blog.csdn.net/u010475041/article/details/78140643
github:https://github.com/kawhii/sso

http://blog.csdn.net/a7695895/article/details/53262494

