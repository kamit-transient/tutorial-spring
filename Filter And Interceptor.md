
[Filter VS Interceptor](http://www.mkjava.com/tutorial/filter-vs-interceptor/#:~:text=Interceptor,access%20to%20all%20Spring%20context.)
## Filter

  A filter as the name suggests is a Java class executed by the servlet container for each incoming HTTP request and for each http response. This way, is possible to manage HTTP incoming requests before them reach the resource, such as a JSP page, a servlet or a simple static page; in the same way is possible to manage HTTP outbound response after resource execution.

## Interceptor

Spring Interceptors are similar to Servlet Filters but they acts in Spring Context so are many powerful to manage HTTP Request and Response but they can implement more sophisticated behavior because can access to all Spring context.

