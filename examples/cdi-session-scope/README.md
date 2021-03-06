index-group=Unrevised
type=page
status=unpublished
title=CDI @SessionScoped
~~~~~~

This example show the use of `@SessionScoped` annotation for injected objects. An object
which is defined as `@SessionScoped` is created once for every HTTPSession and is shared by all the
beans that inject it throughout the same HTTPSession.

##### Run the application:

    mvn clean install tomee:run 
	
# Example

This example has an end point wherein a user provides a request parameter 'name' which is persisted as a feild in a session scoped bean SessionBean and 
then retrieved through another endpoint.

#Request

GET http://localhost:8080/cdi-session-scope-8.0.0-SNAPSHOT/set-name?name=Puneeth

#Response

done, go to /name servlet 

#Request

GET http://localhost:8080/cdi-session-scope-8.0.0-SNAPSHOT/name

#Response

name = {Puneeth} 
 
##SessionBean

The annotation @SessionScoped specifies that a bean is session scoped ie there will be only one instance of the class associated with a particular
HTTPSession.  

@SessionScoped
public class SessionBean implements Serializable {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}  

##InputServlet

InputServlet is a generic servlet which is mapped to the url pattern '/set-name'.
The session scoped bean 'SessionBean' has been injected into this servlet, and the incoming request parameter is set to the feild name of the bean. 

@WebServlet(name = "input-servlet", urlPatterns = {"/set-name"})
public class InputServlet extends HttpServlet {

    @Inject
    private SessionBean bean;

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        final String name = req.getParameter("name");
        if (name == null || name.isEmpty()) {
            resp.getWriter().write("please add a parameter name=xxx");
        } else {
            bean.setName(name);
            resp.getWriter().write("done, go to /name servlet");
        }

    }
}

##AnswerBean

AnswerBean is a request scoped bean with an injected 'SessionBean'. It has an postconstruct method wherein the value from the sessionBean is retrieved and set to a feild.

public class AnswerBean {

    @Inject
    private SessionBean bean;

    private String value;

    @PostConstruct
    public void init() {
        value = '{' + bean.getName() + '}';
    }

    public String value() {
        return value;
    }
}

##OutputServlet

OutputServlet is another servlet with  'AnswerBean' as an injected feild. When '/name' is called the value from 'Answerbean' is read and written to the response.

@WebServlet(name = "output-servlet", urlPatterns = {"/name"})
public class OutputServlet extends HttpServlet {

    @Inject
    private AnswerBean bean;

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        final String name = bean.value();
        if (name == null || name.isEmpty()) {
            resp.getWriter().write("please go to servlet /set-name please");
        } else {
            resp.getWriter().write("name = " + name);
        }
    }
}



