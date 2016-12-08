# Spring Session 공유 방법

lib 에 spring Security 관련 jar 파일이 있다는 전제하에 기술
스프링을 사용하지 않는다면 3,4 번만 적용하면 됨

###1. web.xml 에 아래 추가

session timeout 관련은 30분 그룹 보안 가이드 라인에 따름

```
<!-- Spring Security -->
<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>

<!-- session timeout -->
<session-config>
  <session-timeout>30</session-timeout>
</session-config>
```

###2. spring context 에 아래의 내용 추가

- 세션의 공격에 대비하기 위하여 고정 세션키를 방지하고자  Spring에서는 JSESSIONID를 요청시마다 키를 바꾸고 값을 복사해서 내려 주나 타시스템과의 연동에는 문제가 있어 사용을 하지 못함. 그래서 아무 동작하지 않게 세팅함.
- 기존 security 사용시에도 `<session-management session-fixation-protection="none" />` 는 넣어줘야 함. 그렇지 않으면 세션 공유가 안됨.
- tomcat 8080, tomcat 8081 는 서로 다른 서버 또는 서비스
- tomcat 8080 : a.test.com/context1, a.test.com/context2 context1,context2는 서로 다른  app
- tomcat 8081 : b.test.com/context3, b.test.com/context4 context3,context4는 서로 다른  app

```
<http auto-config="true" use-expressions="true">
    <session-management session-fixation-protection="none" />
</http>
<authentication-manager></authentication-manager>
```

###3. tomcat context 간 세션 공유 설정
tomcat/conf/server.xml
```
<Host ....>
<Context docBase="web" path="/web" crossContext="true" reloadable="true" sessionCookieName="everJSESSIONID" sessionCookiePath="/" sessionCookieDomain=".everland.com" useHttpOnly="true"> </Context>
...
<Context.....></Context>
<Context.....></Context>
...
</Host>
```
- `crossContext="true" sessionCookieName="everJSESSIONID" sessionCookiePath="/" sessionCookieDomain=".everland.com"` 와 같은 어트리뷰트 등록하여 Cookie를 통해 다른 App간 세션을 공유함

###4. redis 를 통한 세션 공유 방법

**tomcat/conf/context.xml**
```
<Context...>
<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
         host="127.0.0.1"
         port="6379"
         database="0"
         maxInactiveInterval="60" />
</Context>
```
**tomcat/conf/lib**

jar 파일 추가
- tomcat-redis-session-manager-2.0.0.jar
- jedis-2.5.2.jar
- commons-pool2-2.2.jar

```
wget -q  https://github.com/rmohr/tomcat-redis-session-manager/releases/download/2.0-tomcat-7/tomcat-redis-session-manager-2.0.0.jar && \
wget -q http://central.maven.org/maven2/redis/clients/jedis/2.5.2/jedis-2.5.2.jar && \
wget -q http://central.maven.org/maven2/org/apache/commons/commons-pool2/2.2/commons-pool2-2.2.jar
```





