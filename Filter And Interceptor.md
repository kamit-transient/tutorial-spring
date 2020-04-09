
[Filter VS Interceptor](http://www.mkjava.com/tutorial/filter-vs-interceptor/#:~:text=Interceptor,access%20to%20all%20Spring%20context.)
[Spring doc on interceptor](https://docs.spring.io/spring/docs/3.0.x/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)

## calling sequence of Filter and Interceptor in spring
![](https://synaren-app.com/images/spring-filter-interceptor-flow.png)

## Filter

 A filter as the name suggests is a Java class executed by the servlet container for each incoming http request and for each http response. This way, is possible to manage HTTP incoming requests before them reach the resource, such as a JSP page, a servlet or a simple static page; in the same way is possible to manage HTTP outbound response after resource execution.

This behaviour allow to implement common functionality reused in many different contexts.
![](http://www.mkjava.com/tutorial/images/filter.png)
As shown in the figure above, the filter runs in the web container so its definition will also be contained in the web.xml file. In the case of the ListFeeds project one of main filter is the CORS management:

```
//web.xml

<filter>
    <filter-name>CORSFilter</filter-name>
    <filter-class>com.listfeeds.components.CORSFilter</filter-class>
    <init-param>
        <param-name>fake-param</param-name>
        <param-value>fake-param-value</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CORSFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
In the filter definition the filter implemented by the class com.listfeeds.filters.CORSFilter with the all endpoints that satisfy the expression: `/*` (in this case all)

```

// com.listfeeds.filters.CORSFilter filer class


package com.listfeeds.filters;

import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;

public class CORSFilter implements Filter {

    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {

        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
        chain.doFilter(req, res);
    }

    public void init(FilterConfig filterConfig) {}

    public void destroy() {}

}

```

The filer include three main methods:
* init: executed to initialize filter using init-param element in filter definition
* doFilter: executed for all HTTP incoming request that satisfy "url-pattern"
* release resources used by the filter


## Interceptor

Spring Interceptors are similar to Servlet Filters but they acts in Spring Context so are many powerful to manage HTTP Request and Response but they can implement more sophisticated behavior because can access to all Spring context.
![http://www.mkjava.com/tutorial/images/interceptor.png](http://www.mkjava.com/tutorial/images/interceptor.png)

The Spring interceptor are execute in SpringMVC context so they have be defined in rest-servlet.xml file:

```
<mvc:interceptors>
    <bean class="com.listfeeds.interceptors.LogContextInterceptor" />
    <bean class="com.listfeeds.interceptors.TimedInterceptor" />
</mvc:interceptors>
```
The com.listfeeds.interceptors.LogContextInterceptor interceptor class used by ListFeeds project to add parameters to the Log4j Thread context.

```
package com.listfeeds.interceptors;

import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang.StringUtils;
import org.apache.log4j.MDC;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerMapping;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

public class LogContextInterceptor extends HandlerInterceptorAdapter {

    private static final Logger log = LoggerFactory.getLogger(LogContextInterceptor.class);

    public static final String LOG_IDENTIFYING_TOKEN = "logIdentifyingToken";

    @Override
    public void afterCompletion(
            HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {

        HandlerMethod methodHandler = (HandlerMethod) handler;
        log.debug("END EXECUTION method {} request: {}", methodHandler.getMethod().getName(), request.getRequestURI());

        Boolean settato = (Boolean) request.getAttribute(LOG_IDENTIFYING_TOKEN);
        if(settato != null && settato) {
            MDC.remove(LOG_IDENTIFYING_TOKEN);
        }
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {

        try {
            if( MDC.get(LOG_IDENTIFYING_TOKEN) == null ) {

                /* Retrieve parameters useful for logging */
                @SuppressWarnings("unchecked")
                Map<String,String> pathVariables = 
                (Map<String, String>) request.getAttribute(HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE);

                String applicationId = null;
                if ( pathVariables != null ) 
                    applicationId = pathVariables.get("applicationId");

                if ( StringUtils.isEmpty(applicationId) ) 
                    applicationId = request.getParameter("applicationId");

                String loggingToken = 
                        String.format("ApplicationId: %s", applicationId);

                MDC.put(LOG_IDENTIFYING_TOKEN, loggingToken);
                request.setAttribute(LOG_IDENTIFYING_TOKEN, Boolean.TRUE);
            }


        }
        catch ( IllegalArgumentException e)
        {
            log.warn("Prehandle",e);
            return true;
        }
        finally {
            HandlerMethod methodHandler = (HandlerMethod) handler;
            //logger.debug("START EXECUTION " + methodHandler.getMethod().getName());
            log.debug("START EXECUTION method {} request: {}", methodHandler.getMethod().getName(), request.getRequestURI());


        }


        return true;
    }

}
```

The interceptor include three main methods:

* __preHandle:__ executed before the execution of the target resource
* __afterCompletion:__ executed after the execution of the target resource (after rendering the view)
* __posttHandle:__ Intercept the execution of a handler

