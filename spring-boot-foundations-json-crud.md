# Spring Boot Foundations

A project-based beginner tutorial for Java full-stack students

Build a REST endpoint, render Thymeleaf pages, understand request data, and complete create, read, update, and delete operations using a JSON file.

CONCEPTS  →  PROJECT SETUP  →  MVC  →  DYNAMIC DATA  →  LIST FEATURES  →  ERROR HANDLING

Classroom Edition


# Learning Path

Read the material in this order:

1. **Spring ecosystem foundations** — understand Spring Framework, Spring Boot, Maven, Tomcat, and the request-response flow.
2. **Spring Boot project setup** — generate, run, and configure the application.
3. **Spring MVC foundations** — controllers, mappings, response bodies, templates, models, and request data.
4. **Small classroom examples** — addition, multiplication, greatest number, form submission, static HTML, and JSON `fetch` requests.
5. **Student Directory project** — objects, Thymeleaf, JSON reading, sorting, searching, pagination, forms, and CRUD.
6. **Todo JSON CRUD project** — writable file storage, Lombok, `ObjectMapper`, DTOs, create, read, update, and delete.
7. **Reference material and next steps** — annotations, vocabulary, debugging, testing, and later database refactoring.

The examples deliberately progress from a single controller method to a complete CRUD application. Finish each stage before introducing a database or advanced architecture.

# How to Use This Tutorial

This tutorial turns one small Student Directory into a sequence of lessons. The first goal is conceptual clarity: browser request, controller mapping, Java object binding, JSON processing, Thymeleaf rendering, and browser response. Service layers, repositories, databases, DTOs, and advanced validation are intentionally postponed until after JSON CRUD is complete.

## Learning Outcomes

- Explain the roles of Spring, Spring Boot, Maven, Tomcat, controllers, models, and templates.
- Generate and run a Spring Boot Maven project with the Maven Wrapper.
- Distinguish @RestController from @Controller.
- Pass values and collections from Java to Thymeleaf.
- Read JSON into Java objects with Jackson.
- Receive client data using query parameters, path variables, form binding, JSON request bodies, headers, cookies, and file uploads.
- Implement sorting, ascending/descending toggling, pagination, and search with query parameters.
- Create student details and custom error pages.
- Complete create, read, update, and delete operations against a writable JSON file using minimal Spring MVC concepts.
## Prerequisites

- Core Java: classes, constructors, getters, lists, loops, exceptions, lambdas, and streams.
- HTML basics: tags, tables, links, forms, and CSS.
- Installed JDK 17 or later, Visual Studio Code, Extension Pack for Java, and Spring Boot Extension Pack.
## Final Project Routes

| Route | Purpose | Response |
| --- | --- | --- |
| / | First REST response | Plain text |
| /web/home | Dynamic welcome page | HTML |
| /web/studentlist | Searchable student directory | HTML |
| /web/students/1 | Single student profile | HTML |

# Module 1 — Spring Ecosystem Foundations

## 1.1 Spring, Spring Boot, and Maven

Spring: A Java framework that provides infrastructure for enterprise applications, including dependency management, web routing, data access, security, and testing.

Spring Boot: A Spring-based approach that reduces setup through auto-configuration, starter dependencies, and an embedded web server.

Maven: The build and dependency tool that downloads libraries, compiles code, runs tests, and packages the application.

Tomcat: The embedded servlet container that receives HTTP requests and runs the web application.

The important distinction is that Spring MVC is a module inside Spring Framework, while Spring Boot is a separate Spring project that configures Spring Framework modules conveniently.

```text
Spring ecosystem
├── Spring Framework
│   ├── Spring Core and dependency injection
│   ├── Spring MVC for servlet-based web applications
│   ├── Spring WebFlux for reactive web applications
│   ├── Spring JDBC and ORM integration
│   ├── transaction management
│   ├── aspect-oriented programming
│   └── testing support
├── Spring Boot for setup, auto-configuration, starters, and embedded servers
├── Spring Data for database access
├── Spring Security for authentication and authorization
├── Spring Cloud for distributed systems and microservices
├── Spring Batch for high-volume batch jobs
└── Spring Integration and messaging projects
```

Most work in this tutorial belongs to Spring MVC: controllers, URL mappings, request parameters, path variables, request bodies, response bodies, HTTP methods, status codes, and Thymeleaf view selection. Spring Boot starts and configures the application; Jackson handles JSON; Thymeleaf renders server-side HTML; Lombok generates Java boilerplate.

**Mental model:** Spring Framework provides the engine and components. Spring Boot assembles and configures them. Spring MVC handles web requests inside that application.

## 1.2 What Happens When a Request Arrives?

1. The browser sends a request such as GET /web/home.
1. Embedded Tomcat receives it.
1. Spring MVC finds the controller method whose mapping matches the URL.
1. The controller returns either response data or a template name.
1. For a template, Thymeleaf renders HTML and the server sends it to the browser.

## 1.3 Servlet Foundations Behind Spring MVC

Spring MVC is built on the Jakarta Servlet API. A servlet is a Java component that receives an HTTP request and produces an HTTP response. A servlet container such as Tomcat creates and manages servlet objects, listens for network requests, and calls the appropriate servlet method.

Spring MVC saves application developers from writing a separate low-level servlet for every URL. Nevertheless, servlet concepts remain underneath controllers, filters, sessions, multipart uploads, Spring Security, and the entire request lifecycle.

### A Request Without Spring MVC

A traditional servlet can handle a URL directly:

```java
package com.example.demo;

import java.io.IOException;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws ServletException, IOException {

        response.setContentType("text/plain");
        response.getWriter().write("Hello");
    }
}
```

This servlet manually handles several responsibilities:

- It declares the URL mapping.
- It overrides `doGet()` for the HTTP GET method.
- It reads values from `HttpServletRequest`.
- It chooses the response content type.
- It writes content through `HttpServletResponse`.
- It must deal with conversion errors and exceptions.

The equivalent Spring MVC endpoint is much smaller:

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }
}
```

The servlet infrastructure has not disappeared. Spring MVC is using it on behalf of the controller.

### Tomcat Is the Servlet Container

Tomcat provides the runtime environment for servlet-based web applications. Its responsibilities include:

- Opening the configured HTTP port, such as `8080`.
- Accepting browser and API-client connections.
- Creating `HttpServletRequest` and `HttpServletResponse` objects.
- Managing servlet initialization and destruction.
- Assigning request-processing threads.
- Managing low-level session and cookie support.
- Passing matching requests to Spring MVC's servlet.

When the MVC web starter is present, Spring Boot configures and starts an embedded servlet container. This is why a Spring Boot application can run through its `main()` method without installing and deploying to a separate Tomcat server.

```text
Spring Boot
    ↓ starts and configures
Embedded Tomcat
    ↓ hosts
Spring MVC DispatcherServlet
    ↓ delegates to
Application controllers
```

### `DispatcherServlet`: Spring MVC's Front Controller

Spring MVC normally routes web requests through one central servlet named `DispatcherServlet`. This design is called the **front controller pattern**: a single entry point coordinates request processing and delegates application work to controller methods.

```text
Browser or API client
        ↓ HTTP request
Embedded Tomcat
        ↓
Filters
        ↓
DispatcherServlet
        ↓
HandlerMapping selects a controller method
        ↓
HandlerAdapter invokes that method
        ↓
Argument resolvers prepare method arguments
        ↓
Controller performs application work
        ↓
HttpMessageConverter or ViewResolver handles the return value
        ↓
HTTP response
```

Important participants are:

- `HandlerMapping`: finds the handler whose annotations match the request method and path.
- `HandlerAdapter`: invokes the selected controller method.
- Argument resolvers: create values for `@PathVariable`, `@RequestParam`, `@RequestBody`, and other controller arguments.
- `HttpMessageConverter`: converts response objects to formats such as JSON and converts request bodies from JSON into Java objects.
- `ViewResolver`: locates a server-side view such as a Thymeleaf template.
- Exception resolvers: convert exceptions into error views or HTTP error responses.

Application controllers do not normally call these components. Spring MVC coordinates them automatically.

### Following a Todo Request Through the Servlet Stack

Consider:

```java
@GetMapping("/todos/{id}")
public ResponseEntity<Todo> getTodo(@PathVariable int id) {
    // Find and return the Todo
}
```

For `GET /web/todos/2`, the processing sequence is:

1. Tomcat receives the HTTP request and creates servlet request and response objects.
2. A servlet filter chain runs.
3. `DispatcherServlet` receives the request.
4. Spring combines the class-level and method-level mappings and selects `getTodo()`.
5. A Spring MVC argument resolver extracts `2` from the path.
6. Spring's conversion system changes the text `"2"` into an `int`.
7. Spring invokes `getTodo(2)`.
8. The controller returns `ResponseEntity<Todo>`.
9. Jackson, through an HTTP message converter, serializes the Todo into JSON.
10. Spring applies the status and headers from `ResponseEntity`.
11. Tomcat sends the completed HTTP response to the client.

### Spring MVC Annotations and Their Servlet Foundations

Spring MVC provides higher-level annotations over common servlet operations:

| Spring MVC feature | Underlying web or servlet operation |
| --- | --- |
| `@GetMapping` | Handle an HTTP GET request |
| `@PostMapping` | Handle an HTTP POST request |
| `@PutMapping` | Handle an HTTP PUT request |
| `@DeleteMapping` | Handle an HTTP DELETE request |
| `@RequestParam` | Read a query or form parameter, similar to `request.getParameter()` |
| `@PathVariable` | Extract a value from a mapped URL path |
| `@RequestHeader` | Read a header, similar to `request.getHeader()` |
| `@CookieValue` | Read a cookie carried by the request |
| `@RequestBody` | Read and convert the raw request body |
| `@ResponseBody` | Serialize and write a return value into the response body |
| `ResponseEntity` | Control response status, headers, and body |
| `MultipartFile` | Work with servlet multipart upload data |
| `HttpSession` | Access the servlet-managed user session |

For example, this Spring MVC method:

```java
@PostMapping("/greatest")
@ResponseBody
public int greatest(
        @RequestParam int a,
        @RequestParam int b) {
    return Math.max(a, b);
}
```

replaces low-level work similar to:

```java
int a = Integer.parseInt(request.getParameter("a"));
int b = Integer.parseInt(request.getParameter("b"));
int greatest = Math.max(a, b);

response.setContentType("text/plain");
response.getWriter().write(String.valueOf(greatest));
```

Spring additionally performs argument conversion, mapping validation, return-value handling, and consistent error processing.

### Request Parameters Versus the Request Body

The servlet request contains several distinct sources of information:

```text
GET /todos?page=2
           └── query parameter

POST form body: title=Learn+Spring
                └── form parameter

POST JSON body: {"title":"Learn Spring"}
                └── raw request body
```

Query parameters and form-encoded values can be obtained with `@RequestParam`. JSON is raw structured request-body content and is normally converted with `@RequestBody` and Jackson.

Reading the raw request body is generally a one-time stream operation. Spring's message-conversion system manages that stream and produces the Java object requested by the controller.

### Servlet Filters

A servlet filter can inspect or modify a request and response before or after `DispatcherServlet` runs:

```text
Request
   ↓
Filter 1 → Filter 2 → DispatcherServlet → Controller
   ↑                                      ↓
Response ← Filter 2 ←─────────────────────┘
```

Common filter responsibilities include:

- Authentication and security checks
- CORS handling
- Request and response logging
- Request identifiers for tracing
- Header modification
- Compression and encoding

In Spring applications, `OncePerRequestFilter` is a convenient base class:

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        System.out.println(
                request.getMethod() + " " + request.getRequestURI()
        );

        filterChain.doFilter(request, response);
    }
}
```

Calling `filterChain.doFilter()` is essential because it passes control to the next filter and eventually to `DispatcherServlet`. Spring Security is largely implemented as a chain of servlet filters.

### Filters Versus Spring MVC Interceptors

Filters belong to the Servlet API and run outside Spring MVC's controller-processing layer. Interceptors belong to Spring MVC and run around controller-handler execution.

| Filter | Handler interceptor |
| --- | --- |
| Works at the servlet-container level | Works inside Spring MVC |
| Runs before `DispatcherServlet` | Runs after Spring has selected a handler |
| Can wrap or replace request and response objects | Can inspect the selected controller handler |
| Suitable for security, CORS, encoding, and low-level logging | Suitable for controller timing, auditing, and MVC-specific checks |

The main interceptor callbacks are:

```java
preHandle()
postHandle()
afterCompletion()
```

### Request Scope, Sessions, and Cookies

HTTP is stateless: one request does not automatically remember a previous request. Servlets provide mechanisms for request-scoped and session-scoped state.

- Request attributes exist for only one request.
- Cookies are small values stored by the browser and returned in later requests.
- `HttpSession` stores server-side data associated with a session identifier, commonly carried in a `JSESSIONID` cookie.

Example session access:

```java
@GetMapping("/visit")
@ResponseBody
public String visit(HttpSession session) {
    Integer count = (Integer) session.getAttribute("count");
    count = count == null ? 1 : count + 1;
    session.setAttribute("count", count);
    return "Visit count: " + count;
}
```

Do not store large amounts of data or request-specific temporary state in a session. REST APIs are commonly designed to remain stateless.

### Forward Versus Redirect

A forward continues processing on the server using the same request and response. The browser is not asked to make another request, so its displayed URL does not change.

A redirect sends a redirect status and location to the browser. The browser then makes a new request:

```java
return "redirect:/web/todos";
```

```text
POST /web/todos
      ↓ server returns redirect
GET /web/todos
      ↓ browser makes a fresh request
Todo list page
```

Redirecting after a successful form POST prevents a browser refresh from accidentally repeating the submission. This is known as the Post/Redirect/Get pattern.

### Thread Safety in Controllers and Servlets

Tomcat can process many requests concurrently. Spring controllers are singleton objects by default, so multiple request threads may call the same controller instance at the same time.

Avoid storing request-specific mutable values in controller fields:

```java
// Unsafe: shared by multiple requests
private int currentTodoId;
```

Method-local variables are isolated to one invocation and are safer:

```java
public Todo getTodo(@PathVariable int id) {
    int currentTodoId = id;
    // ...
}
```

The file-based CRUD examples are intentionally simple, but simultaneous writes can overwrite one another. Databases and transaction management become important when applications must safely support concurrent users.

### What Servlet Knowledge Is Needed for Spring MVC?

You do not need to build the complete project with raw servlets first. Learn enough to explain:

- What a servlet container does
- The roles of `HttpServletRequest` and `HttpServletResponse`
- Why `DispatcherServlet` is called a front controller
- How filters and the filter chain work
- How sessions and cookies preserve state
- How forwarding differs from redirecting
- Why controller fields must be thread-safe
- How Spring MVC converts low-level requests into annotated method calls

The central mental model is:

> Servlets provide the low-level Java web foundation. Spring MVC's `DispatcherServlet` uses that foundation to provide controllers, annotations, data binding, validation, view resolution, exception handling, and automatic JSON conversion.

## Checkpoint Questions

- Why is Spring Boot not a replacement for Spring?
- What problem does the embedded server solve?
- What jobs does Maven perform?
- What is the responsibility of a servlet container?
- Why is `DispatcherServlet` called a front controller?
- How does a filter differ from a Spring MVC interceptor?
- Why should a controller avoid request-specific mutable fields?
- What is the difference between forwarding and redirecting?
# Module 2 — Generate and Run the Project

## 2.1 Create the Project in VS Code

1. Open the Command Palette with Ctrl+Shift+P.
1. Select “Spring Initializr: Generate a Maven Project”.
1. Choose Java and a stable Spring Boot 3.x version.
1. Use Group Id com.student and Artifact Id demo.
1. Choose JAR packaging and Java 17 or later.
1. Add the Spring Web dependency.
1. Select a folder and open the generated project.
## 2.2 Understand the Project Structure

**Project structure**

```
demo/
├── pom.xml
├── mvnw
├── mvnw.cmd
└── src/
    ├── main/
    │   ├── java/com/student/demo/
    │   │   └── DemoApplication.java
    │   └── resources/
    │       └── application.properties
    └── test/
```

The package com.student.demo is the application’s base package. Put controllers and related classes inside this package or one of its subpackages so component scanning can discover them.

## 2.3 Run with the Maven Wrapper

**Windows PowerShell or Command Prompt**

```
.\mvnw spring-boot:run
```

**macOS or Linux**

```
./mvnw spring-boot:run
```

| C |
| --- |

# Module 3 — Your First REST Endpoint

## 3.1 Create HelloController.java

**src/main/java/com/student/demo/HelloController.java**

```
package com.student.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String sayHello() {
        return "Welcome to your first Spring Boot App!";
    }
}
```

## 3.2 Read the Annotations

- @RestController marks the class as a web controller whose return values become the HTTP response body.
- @GetMapping("/") connects an HTTP GET request for / to sayHello().
- The returned String is sent as plain text to the browser.
| I |
| --- |

## 3.3 Test

Run the application and open http://localhost:8080. The browser should show the welcome message.

| C |
| --- |

# Module 4 — Application Configuration

Spring Boot uses application.properties for external configuration. Change the server port when 8080 is already occupied or when a class project uses a standard port.

**src/main/resources/application.properties**

```
server.port=9090
```

Stop the application with Ctrl+C, start it again, and open http://localhost:9090.

| C |
| --- |

# Module 5 — From REST to Server-Rendered HTML

## 5.1 Add Thymeleaf

Add the Thymeleaf starter inside the <dependencies> element of pom.xml. Keep the Spring Web dependency already generated by Initializr.

**pom.xml**

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 5.2 Create home.html

**src/main/resources/templates/home.html**

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
    <h1>This is a real HTML web page!</h1>
</body>
</html>
```

## 5.3 Create WebController.java

**src/main/java/com/student/demo/WebController.java**

```
package com.student.demo;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/web")
public class WebController {

    @GetMapping("/home")
    public String showHomePage() {
        return "home";
    }
}
```

## 5.4 @Controller vs @RestController

| Annotation | Return value means | Typical use |
| --- | --- | --- |
| @RestController | Response body | Plain text or JSON APIs |
| @Controller | View/template name | Server-rendered HTML |
| @PathVariable | Reads a value embedded in the URL path. |  |
| @ModelAttribute | Binds HTML form fields to a Java object. |  |
| @RequestBody | Converts a JSON request body into a Java object. |  |
| @RequestHeader | Reads an HTTP request header. |  |
| @CookieValue | Reads a cookie value. |  |

| M |
| --- |

# Module 6 — Pass Dynamic Data with Model

The Model is a container for values that a controller sends to a Thymeleaf template. Each value has a key that the template uses to retrieve it.

**WebController.java**

```
package com.student.demo;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/web")
public class WebController {

    @GetMapping("/home")
    public String webHome(Model model) {
        model.addAttribute("username", "Alex");
        model.addAttribute("bootcampDay", 5);
        return "home";
    }
}
```

**home.html**

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Home</title>
</head>
<body>
    <h1>Welcome back, <span th:text="${username}">Guest</span>!</h1>
    <p>
        Congratulations on reaching Day
        <span th:text="${bootcampDay}">1</span>
        of the bootcamp.
    </p>
</body>
</html>
```

## How Thymeleaf Processes the Page

- The fallback text “Guest” and “1” remains visible when the file is opened directly.
- When accessed through Spring Boot, Thymeleaf replaces the fallback text with Model values.
- Model keys are case-sensitive: username and userName are different keys.

# Current Classroom Example: Controller Paths and Development Reloading

This section records the follow-up discussion based on the current `com.example.demo` project.

## Combining Class and Method Mappings

A class-level `@RequestMapping` supplies the common prefix for every handler method in that controller. The method-level mapping is appended to it.

```java
@Controller
@RequestMapping("/web")
public class HelloController {

    @GetMapping("/home")
    public String hello() {
        return "Hello";
    }
}
```

The final URL is:

```text
http://localhost:8080/web/home
```

Spring interprets the returned string `"Hello"` as the name of the Thymeleaf template `src/main/resources/templates/Hello.html`.

## Path Variables

A path variable reads a dynamic value embedded in the URL path. In this example, `{a}` and `{b}` are placeholders:

```java
@GetMapping("/add/{a}/{b}")
@ResponseBody
public int addition(
        @PathVariable("a") int x,
        @PathVariable("b") int y) {
    return x + y;
}
```

Calling the following URL binds `10` to `x` and `20` to `y`:

```text
http://localhost:8080/web/add/10/20
```

The response is:

```text
30
```

The names in `@PathVariable` must match the placeholders in `@GetMapping`. The Java parameter names do not have to match when the placeholder name is given explicitly:

```java
@PathVariable("a") int x
```

Here, Spring reads `{a}` and stores its converted integer value in `x`. If the annotation value is omitted, using matching names is clearest:

```java
@GetMapping("/add/{a}/{b}")
public int addition(@PathVariable int a, @PathVariable int b) {
    return a + b;
}
```

Explicit names are safer when the URI placeholder and Java parameter use different names or when parameter-name metadata is unavailable at runtime.

## Why `@ResponseBody` Is Required Here

Methods in a class annotated with `@Controller` normally return a view name:

```java
@GetMapping("/home")
public String hello() {
    return "Hello";
}
```

Spring looks for a Thymeleaf template named `Hello.html`.

The addition method returns response data rather than a template name. `@ResponseBody` tells Spring to write that return value directly into the HTTP response:

```java
@GetMapping("/add/{a}/{b}")
@ResponseBody
public int addition(@PathVariable("a") int x,
                    @PathVariable("b") int y) {
    return x + y;
}
```

The distinction is:

| Controller setup | Meaning of a returned value |
| --- | --- |
| `@Controller` without `@ResponseBody` | A view or template name |
| `@Controller` with `@ResponseBody` on a method | Data written directly to the response |
| `@RestController` | Every handler method returns response data by default |

Keep `@Controller` for a mixed controller that renders Thymeleaf pages and returns data from only selected methods. Changing the class to `@RestController` would cause strings such as `"Hello"` to appear as plain text instead of rendering `Hello.html`.

## Avoiding Manual Restarts During Development

Add Spring Boot DevTools as a runtime dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

Use these development settings in `src/main/resources/application.properties`:

```properties
spring.application.name=demo
spring.thymeleaf.cache=false
spring.devtools.restart.enabled=true
spring.devtools.livereload.enabled=true
```

Start the application once on Windows:

```powershell
.\mvnw.cmd spring-boot:run
```

- After changing a Thymeleaf template or static resource, save it and refresh the browser. Disabling the Thymeleaf cache allows the saved template to be read again.
- After changing Java code, save and compile it. DevTools detects the changed classpath and restarts the application context automatically.
- A LiveReload browser extension can refresh the browser automatically when the embedded LiveReload server reports a change.
- In VS Code, use the Extension Pack for Java and enable automatic saving. In IntelliJ IDEA, enable automatic project building while the application is running.

DevTools performs an automatic application-context restart for compiled Java changes. It removes the need to stop and manually launch the server again, although it is not the same as replacing every Java class in place without a restart.

# Module 7 — Render a List of Student Objects

## 7.1 Create Student.java

**Student.java**

```
package com.student.demo;

public class Student {
    private int studentId;
    private String fullName;
    private String gender;
    private int age;
    private String city;
    private String state;
    private String remarks;
    private String image;
    private String motherName;
    private String fatherName;
    private String favSportsPerson;

    public Student() {
    }

    public int getStudentId() { return studentId; }
    public String getFullName() { return fullName; }
    public String getGender() { return gender; }
    public int getAge() { return age; }
    public String getCity() { return city; }
    public String getState() { return state; }
    public String getRemarks() { return remarks; }
    public String getImage() { return image; }
    public String getMotherName() { return motherName; }
    public String getFatherName() { return fatherName; }
    public String getFavSportsPerson() { return favSportsPerson; }
}
```

| W |
| --- |

## 7.2 First In-Memory List

**Conceptual first step**

```
@GetMapping("/directory")
public String showDirectory(Model model) {
    List<Student> students = new ArrayList<>();
    students.add(new Student(/* constructor values */));
    model.addAttribute("students", students);
    return "directory";
}
```

The source lesson first introduces a hardcoded list so students can focus on Model and th:each. The next module moves the data to JSON, which becomes the project’s final structure.

## 7.3 Thymeleaf Iteration

**Thymeleaf loop**

```
<tbody>
    <tr th:each="student : ${students}">
        <td th:text="${student.fullName}">John Doe</td>
        <td th:text="${student.gender}">Male</td>
        <td th:text="${student.age}">20</td>
        <td th:text="${student.city}">Metropolis</td>
        <td th:text="${student.state}">State</td>
    </tr>
</tbody>
```

The expression th:each="student : ${students}" corresponds to the Java enhanced loop for (Student student : students).

# Module 8 — Read Students from a JSON File

## 8.1 Create students.json

**src/main/resources/students.json**

```
[
  {
    "studentId": 1,
    "fullName": "Priya Sharma",
    "gender": "Female",
    "age": 22,
    "city": "Mumbai",
    "state": "Maharashtra",
    "remarks": "Excellent in math",
    "image": "https://placehold.co/150?text=PS",
    "motherName": "Anita",
    "fatherName": "Rajesh",
    "favSportsPerson": "Virat Kohli"
  },
  {
    "studentId": 2,
    "fullName": "Rahul Patel",
    "gender": "Male",
    "age": 24,
    "city": "Ahmedabad",
    "state": "Gujarat",
    "remarks": "Great team player",
    "image": "https://placehold.co/150?text=RP",
    "motherName": "Meena",
    "fatherName": "Suresh",
    "favSportsPerson": "MS Dhoni"
  }
]
```

Add more objects using the same property names. The original lesson expands this dataset to 50 records so that pagination produces multiple pages.

## 8.2 Load JSON with ObjectMapper

**Add this helper inside WebController**

```
private List<Student> loadStudents() throws Exception {
    ObjectMapper mapper = new ObjectMapper();

    try (InputStream input =
             new ClassPathResource("students.json").getInputStream()) {
        return mapper.readValue(
            input,
            new TypeReference<List<Student>>() {}
        );
    }
}
```

## Why the Empty Constructor?

Jackson creates an empty Student object and fills its properties from JSON. The no-argument constructor supports this object-creation process. Property names in JSON must match the Java properties.

| F |
| --- |

# Module 9 — Sorting and Student Details

## 9.1 Sort with Query Parameters

A URL such as /web/studentlist?sort=age&dir=desc sends two query parameters. @RequestParam reads them into method arguments.

**Sorting helper**

```
private Comparator<Student> comparatorFor(String sortMethod) {
    if (sortMethod == null) return null;

    return switch (sortMethod) {
        case "fullName" -> Comparator.comparing(Student::getFullName);
        case "gender"   -> Comparator.comparing(Student::getGender);
        case "age"      -> Comparator.comparingInt(Student::getAge);
        case "city"     -> Comparator.comparing(Student::getCity);
        case "state"    -> Comparator.comparing(Student::getState);
        default         -> null;
    };
}
```

For descending order, call comparator.reversed(). The HTML header links send the next direction so each click toggles ascending and descending order.

**Clickable Age header**

```
<a th:href="@{/web/studentlist(
        sort='age',
        dir=${currentSort == 'age' and currentDir == 'asc' ? 'desc' : 'asc'},
        search=${keyword},
        page=1)}">
    Age ↕
</a>
```

## 9.2 Student Details Route

**Details controller method**

```
@GetMapping("/students/{id}")
public String showStudentDetails(
        @PathVariable("id") int studentId,
        Model model) {

    try {
        Student foundStudent = loadStudents().stream()
            .filter(student -> student.getStudentId() == studentId)
            .findFirst()
            .orElse(null);

        model.addAttribute("student", foundStudent);
    } catch (Exception e) {
        model.addAttribute("student", null);
    }

    return "studentdetails";
}
```

**Details link in each table row**

```
<a class="details-btn"
   th:href="@{/web/students/{id}(id=${student.studentId})}">
    View Details
</a>
```

## 9.3 Ways Spring Receives Request Data

Query parameters are only one source of request data. Spring MVC can bind values from the URL path, query string, HTML forms, JSON body, headers, cookies, and uploaded files directly to controller method parameters.

### A. Query parameters with @RequestParam

Query parameters appear after ? in a URL. They are best suited to optional controls such as search, filtering, sorting, and pagination.

**Example request and controller**

```
GET /web/studentlist?page=2&sort=age&dir=desc

@GetMapping("/studentlist")
public String showStudents(
        @RequestParam(defaultValue = "1") int page,
        @RequestParam(required = false) String sort,
        @RequestParam(defaultValue = "asc") String dir) {
    return "studentlist";
}
```

### B. Path variables with @PathVariable

A path variable is embedded in the URL itself. Use it when the value identifies a particular resource.

**Single and multiple path variables**

```
@GetMapping("/students/{id}")
@ResponseBody
public String getStudent(@PathVariable int id) {
    return "Student ID: " + id;
}

@GetMapping("/students/{studentId}/courses/{courseId}")
@ResponseBody
public String getEnrollment(
        @PathVariable int studentId,
        @PathVariable int courseId) {
    return "Student " + studentId + ", Course " + courseId;
}
```

Prefer /students/10 to identify student 10. Prefer /students?city=Hyderabad&page=2 when applying optional filters or display controls.

### C. Simple HTML form values with @RequestParam

For a small form, each input can be received individually. The HTML name attribute must match the request parameter name.

**HTML form and controller**

```
<form action="/web/register" method="post">
    <input type="text" name="fullName">
    <input type="number" name="age">
    <button type="submit">Register</button>
</form>

@PostMapping("/register")
@ResponseBody
public String registerStudent(
        @RequestParam String fullName,
        @RequestParam int age) {
    return fullName + " registered with age " + age;
}
```

#### Worked Example: Greatest of Two Numbers with a POST Form

Query-string values and fields submitted by a standard HTML form are both request parameters in Spring MVC. Therefore, both can be accessed with `@RequestParam`.

Their usual locations are different:

| Request | Where the parameters are sent | Spring annotation |
| --- | --- | --- |
| `GET /web/mul?a=10&b=20` | In the URL query string after `?` | `@RequestParam` |
| A standard `POST` form | In the request body as form-encoded data | `@RequestParam` |
| A `POST` request containing JSON | In the request body as JSON | `@RequestBody` |

Create `src/main/resources/templates/Greatest.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Find Greatest Number</title>
</head>
<body>
    <h1>Find the Greatest Number</h1>

    <form action="/web/greatest" method="post">
        <label for="a">First number:</label>
        <input id="a" type="number" name="a" required>

        <label for="b">Second number:</label>
        <input id="b" type="number" name="b" required>

        <button type="submit">Find Greatest</button>
    </form>
</body>
</html>
```

Add the GET and POST handler methods to a controller whose class-level mapping is `@RequestMapping("/web")`:

```java
@GetMapping("/greatest")
public String showGreatestForm() {
    return "Greatest";
}

@PostMapping("/greatest")
@ResponseBody
public int findGreatest(
        @RequestParam int a,
        @RequestParam int b) {
    return Math.max(a, b);
}
```

Open `http://localhost:8080/web/greatest` to display the form. When the form is submitted, the browser sends values similar to `a=10&b=20` in the POST request body. Spring matches the HTML `name` attributes to the Java parameters:

```text
name="a"  →  @RequestParam int a
name="b"  →  @RequestParam int b
```

`@ResponseBody` writes the result of `Math.max(a, b)` directly into the HTTP response. If both inputs are equal, that same value is returned.

#### Serving the Form Without a GET Controller

Spring Boot serves files placed in `src/main/resources/static` directly. If `Greatest.html` contains only regular HTML and does not need Thymeleaf processing, it can be placed here:

```text
src/main/resources/static/Greatest.html
```

It is then available without a GET handler at:

```text
http://localhost:8080/Greatest.html
```

The static form can still submit to the POST handler at `/web/greatest`. Files under `src/main/resources/templates`, by contrast, are intended for server-side template processing and normally need a controller method that returns the template name. Use `static` for plain HTML, CSS, and JavaScript; use `templates` when the page needs Thymeleaf features such as `th:text`, `th:if`, `th:each`, model values, or `th:action`.

### D. Complete form binding with @ModelAttribute

When a form contains many related fields, Spring can create a Java object and bind matching form values to its properties. The class needs a no-argument constructor plus getters and setters.

**Bind the complete form to a Student object**

```
@PostMapping("/register")
public String registerStudent(
        @ModelAttribute Student student,
        Model model) {
    model.addAttribute("student", student);
    return "registration-success";
}
```

### E. JSON request data with @RequestBody

Frontend applications such as React, Angular, Next.js, and mobile clients commonly send JSON. Jackson converts the JSON request body into a Java object.

**REST API request body**

```
POST /api/students
Content-Type: application/json

{
  "fullName": "Priya Sharma",
  "age": 22,
  "city": "Mumbai"
}

@PostMapping("/api/students")
@ResponseBody
public Student createStudent(@RequestBody Student student) {
    return student;
}
```

#### Worked Example: Sending JSON with `fetch`

The following JavaScript prevents the form's normal submission and sends the two input values as JSON:

```javascript
function divide(event) {
    event.preventDefault();

    fetch("http://localhost:8080/web/div", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            a: Number(document.dform.a.value),
            b: Number(document.dform.b.value)
        })
    })
    .then(response => {
        if (!response.ok) {
            throw new Error("Request failed: " + response.status);
        }
        return response.json();
    })
    .then(result => console.log(result))
    .catch(error => console.error(error));
}
```

`Content-Type: application/json` tells Spring that the request body contains JSON rather than form-encoded parameters. A request from this example has a body like:

```json
{
  "a": 20,
  "b": 4
}
```

Create a Java type whose structure matches those JSON properties:

```java
package com.example.demo;

public record NumberInput(double a, double b) {
}
```

Then receive it with `@RequestBody`:

```java
@PostMapping("/div")
@ResponseBody
public double divide(@RequestBody NumberInput input) {
    return input.a() / input.b();
}
```

Jackson reads the JSON and creates a `NumberInput` value. It maps JSON property `a` to record component `a` and JSON property `b` to record component `b`.

`NumberInput` is not a Java or Spring keyword. It is a developer-chosen type name and can be renamed, for example:

```java
public record CalculationRequest(double a, double b) {
}
```

If the type is renamed, the controller parameter type must also be changed:

```java
public double divide(@RequestBody CalculationRequest input) {
    return input.a() / input.b();
}
```

The individual parts have these roles:

| Part | Role |
| --- | --- |
| `record` | Java keyword for declaring a compact data-carrier class |
| `NumberInput` | Developer-chosen type name |
| `double a, double b` | Record components that match the JSON properties |
| `@RequestBody` | Tells Spring/Jackson to convert the JSON body into the declared Java type |
| `input` | Developer-chosen method-parameter name |

A standard HTML form submission such as `a=20&b=4` uses `@RequestParam`. A JSON body such as `{"a":20,"b":4}` uses `@RequestBody`. Serve the HTML through Spring—for example, from `src/main/resources/static`—instead of opening it through a `file://` URL, which can lead to browser-origin restrictions.

### F. Request headers with @RequestHeader

Headers carry request metadata such as authentication tokens, API keys, content type, language preferences, and client information.

```
@GetMapping("/client")
@ResponseBody
public String getClient(
        @RequestHeader("User-Agent") String userAgent) {
    return "Client: " + userAgent;
}
```

### G. Cookies with @CookieValue

```
@GetMapping("/theme")
@ResponseBody
public String getTheme(
        @CookieValue(name = "theme", defaultValue = "light") String theme) {
    return "Selected theme: " + theme;
}
```

### H. Uploaded files with MultipartFile

A file upload form must use multipart/form-data. Spring receives the selected file as a MultipartFile.

**File upload form and controller**

```
<form action="/web/upload" method="post"
      enctype="multipart/form-data">
    <input type="file" name="document">
    <button type="submit">Upload</button>
</form>

@PostMapping("/upload")
@ResponseBody
public String uploadFile(
        @RequestParam("document") MultipartFile file) {
    return "Uploaded file: " + file.getOriginalFilename();
}
```

### Choosing the Correct Annotation

Use @PathVariable for a required resource identity, @RequestParam for optional query or simple form values, @ModelAttribute for a server-rendered form object, and @RequestBody for JSON sent to a REST API. Use @RequestHeader, @CookieValue, and MultipartFile for their specific request components.

### Recommended Teaching Order

1. @RequestParam — query parameters and simple form fields
1. @PathVariable — resource identifiers in URL paths
1. @ModelAttribute — complete Thymeleaf form objects
1. @RequestBody — JSON request bodies for REST APIs
1. @RequestHeader and @CookieValue — request metadata and cookies
1. MultipartFile — uploaded files
# Module 10 — Understanding Thymeleaf URLs and Links

## 10.1 Why @ Appears in th:href

In Thymeleaf, @{...} is a URL expression. It tells Thymeleaf to build an application-aware URL before the HTML is sent to the browser.

**Thymeleaf source**

```
<a th:href="@{/web/students}">Students</a>
```

**HTML received by the browser**

```
<a href="/web/students">Students</a>
```

The browser never sees th:href or @{...}. Thymeleaf runs on the server and produces normal HTML. Using @{...} also allows Thymeleaf to include the application context path when the application is deployed under a path such as /studentapp.

## 10.2 URL Parameters Inside Parentheses

```
<a th:href="@{/web/studentlist(sort='age', page=1)}">
    Sort by Age
</a>
```

Values inside the parentheses become query parameters. The generated URL is approximately /web/studentlist?sort=age&page=1.

## 10.3 Understanding the Age Sorting Link

```
<a th:href="@{/web/studentlist(
        sort='age',
        dir=${currentSort == 'age' and currentDir == 'asc' ? 'desc' : 'asc'},
        search=${keyword},
        page=1)}">
    Age ↕
</a>
```

- sort='age' tells the controller to sort by the age property.
- dir=... toggles the next click: ascending becomes descending; otherwise the next direction is ascending.
- search=${keyword} keeps the current search filter when the user sorts.
- page=1 resets the result to the first page because sorting changes the order of all records.
If currentSort is age, currentDir is asc, and keyword is Mumbai, Thymeleaf generates approximately /web/studentlist?sort=age&dir=desc&search=Mumbai&page=1.

## 10.4 Path Variables in Thymeleaf Links

```
<a th:href="@{/web/students/{id}(id=${student.studentId})}">
    View Details
</a>
```

The {id} part is a placeholder. The value supplied in parentheses replaces it. For studentId 5, Thymeleaf generates /web/students/5.

```
@GetMapping("/students/{id}")
public String showStudent(
        @PathVariable int id,
        Model model) {
    // Find the student and add it to the Model.
    return "studentdetails";
}
```

## 10.5 Common Mistake: th:text Replaces Child Content

th:text replaces all content inside the element on which it appears. If an anchor is placed inside an li that also has th:text, the anchor is removed during rendering.

**Incorrect**

```
<li th:each="student : ${students}"
    th:text="${student.fullName}">
    <a th:href="@{/web/students/{id}(id=${student.studentId})}">
        View Details
    </a>
</li>
```

**Correct**

```
<li th:each="student : ${students}">
    <span th:text="${student.fullName}">Student Name</span>
    -
    <a th:href="@{/web/students/{id}(id=${student.studentId})}">
        View Details
    </a>
</li>
```

For fixed text such as View Details, write it directly inside the anchor. th:text is unnecessary. Also prefer ${student.fullName} over ${student.getFullName()}; Thymeleaf property access automatically uses the getter.

# Module 11 — Thymeleaf Forms and Data Binding

## 11.1 The Main Purpose of Thymeleaf in a Form

Thymeleaf connects a server-side Java object to HTML form controls. It helps display existing object values, generate correct field names, submit values to the correct URL, and redisplay user input when the form has an error.

Three concepts must be kept separate: data binding maps request values into Java properties; client-side validation is performed by the browser; server-side validation is performed by Java after the request reaches the application.

## 11.2 Show an Empty Form Object

```
@GetMapping("/students/new")
public String showAddForm(Model model) {
    model.addAttribute("student", new Student());
    return "studentform";
}
```

The controller sends an empty Student object to the template under the key student.

## 11.3 Connect the Form with th:object

```
<form th:action="@{/web/students}"
      th:object="${student}"
      method="post">

    <label>Full Name</label>
    <input type="text" th:field="*{fullName}" required>

    <label>Age</label>
    <input type="number" th:field="*{age}" min="18" required>

    <button type="submit">Save Student</button>
</form>
```

- th:object=${student} selects the Student object used by the form.
- th:field=*{fullName} connects the input to student.fullName.
- th:action builds the form submission URL.
- method=post tells the browser to send a POST request.
## 11.4 What th:field Generates

**Template**

```
<input type="text" th:field="*{fullName}">
```

**Generated HTML, approximately**

```
<input type="text" id="fullName" name="fullName" value="">
```

The name attribute is essential. When the browser submits fullName=Priya, Spring can match that name to the fullName property and call student.setFullName("Priya"). This process is data binding.

## 11.5 Receive the Structured Form with @ModelAttribute

```
@PostMapping("/students")
public String addStudent(
        @ModelAttribute Student student) {

    // Spring has already filled the Student object.
    return "redirect:/web/students";
}
```

Spring creates a Student, converts submitted text to the required Java types, and calls matching setter methods. Mapping therefore answers “which request value belongs to which Java property?” It does not by itself prove that the values are logically valid.

## 11.6 Why Getters, Setters, and a No-Argument Constructor Matter

- Spring needs a no-argument constructor to create an empty Student during form binding.
- Spring uses setter methods to place submitted values into the object.
- Thymeleaf uses getter methods to display values in create, edit, and error redisplay scenarios.
## 11.7 Create and Edit Forms

The same th:field expressions work for both create and edit pages. On an edit page, the controller sends an existing Student object. Thymeleaf automatically fills each control with the current value.

```
@GetMapping("/students/{id}/edit")
public String showEditForm(
        @PathVariable int id,
        Model model) {

    Student student = findStudentById(id);
    model.addAttribute("student", student);
    return "studenteditform";
}
```

If student.fullName is Priya Sharma, <input th:field="*{fullName}"> is rendered with that value already filled in.

## 11.8 Different Form Controls

```
<input type="text" th:field="*{fullName}">
<input type="number" th:field="*{age}">
<textarea th:field="*{remarks}"></textarea>

<select th:field="*{gender}">
    <option value="">Select gender</option>
    <option value="Male">Male</option>
    <option value="Female">Female</option>
</select>

<input type="checkbox" th:field="*{active}">
```

th:field manages the name, current value, selected option, or checked state as appropriate for the control.

## 11.9 Client-Side Validation

HTML validation runs in the browser before the request is sent. Thymeleaf does not perform this validation; it only helps generate the HTML control.

```
<input type="text"
       th:field="*{fullName}"
       minlength="3"
       maxlength="50"
       required>

<input type="number"
       th:field="*{age}"
       min="18"
       max="60"
       required>
```

Client-side validation improves usability, but it can be bypassed with browser tools, JavaScript, or Postman. It must not be treated as application protection.

## 11.10 Minimal Server-Side Validation for This Stage

Before introducing validation annotations, use simple controller checks so students can clearly see the request and response flow.

```
@PostMapping("/students")
public String addStudent(
        @ModelAttribute Student student,
        Model model) {

    if (student.getFullName() == null ||
        student.getFullName().isBlank()) {
        model.addAttribute("error", "Full name is required");
        return "studentform";
    }

    if (student.getAge() < 18) {
        model.addAttribute("error", "Age must be at least 18");
        return "studentform";
    }

    // Save to JSON.
    return "redirect:/web/students";
}
```

**Display the message in the template**

```
<p th:if="${error != null}"
   th:text="${error}">
</p>
```

Because the same Student object remains connected to the form, th:field can redisplay the values the user already entered.

## 11.11 Form Request-Response Flow

1. GET /web/students/new asks for the form.
1. The controller adds an empty Student to the Model and returns studentform.
1. Thymeleaf generates normal HTML fields and sends the page to the browser.
1. The user submits the form with POST /web/students.
1. @ModelAttribute binds request fields into a Student object.
1. The controller validates and saves the object, then returns a redirect response.
1. The browser follows the redirect with a new GET request for the student list.
# Module 12 — Complete CRUD with a Writable JSON File

## 12.1 Scope of This Classroom Stage

This module intentionally keeps all logic in one controller and uses one JSON file. The goal is to understand requests, mappings, Java objects, JSON read/write operations, Models, templates, and redirects before introducing architecture.

- Use @Controller, @GetMapping, @PostMapping, @PathVariable, @ModelAttribute, Model, Thymeleaf, ObjectMapper, and redirect:.
- Do not introduce @Service, @Repository, JPA, PostgreSQL, DTOs, interfaces, ResponseEntity, or advanced validation yet.
## 12.2 Use a Writable File Location

A file under src/main/resources is suitable for read-only seed data, but it is not a reliable writable location after the application is packaged as a JAR. For CRUD practice, create data/students.json in the project root and read and write that same file.

```
demo/
├── data/
│   └── students.json
├── pom.xml
└── src/main/
    ├── java/com/student/demo/
    │   ├── DemoApplication.java
    │   ├── Student.java
    │   └── WebController.java
    └── resources/templates/
        ├── studentlist.html
        ├── studentdetails.html
        ├── studentform.html
        └── studenteditform.html
```

## 12.3 CRUD Routes

```
GET  /web/students                 Read all students
GET  /web/students/{id}            Read one student
GET  /web/students/new             Show create form
POST /web/students                 Create a student
GET  /web/students/{id}/edit       Show edit form
POST /web/students/{id}/update     Update a student
POST /web/students/{id}/delete     Delete a student
```

HTML forms directly support GET and POST. For this beginner server-rendered project, POST is used for update and delete. REST-style PUT and DELETE can be introduced later.

## 12.4 JSON Read and Write Helpers

```
private final ObjectMapper mapper = new ObjectMapper();
private final File studentFile = new File("data/students.json");

private List<Student> readStudentsFromJson() {
    try {
        if (!studentFile.exists()) {
            studentFile.getParentFile().mkdirs();
            mapper.writerWithDefaultPrettyPrinter()
                  .writeValue(studentFile, new ArrayList<Student>());
        }

        return mapper.readValue(
                studentFile,
                new TypeReference<List<Student>>() {});

    } catch (Exception e) {
        throw new RuntimeException("Unable to read students.json", e);
    }
}

private void writeStudentsToJson(List<Student> students) {
    try {
        mapper.writerWithDefaultPrettyPrinter()
              .writeValue(studentFile, students);
    } catch (Exception e) {
        throw new RuntimeException("Unable to write students.json", e);
    }
}
```

Each write operation follows the same pattern: read the complete JSON array into a List<Student>, modify the list, and write the complete list back to the file.

## 12.5 Read All Students

```
@GetMapping("/students")
public String showStudents(Model model) {
    List<Student> students = readStudentsFromJson();
    model.addAttribute("students", students);
    return "studentlist";
}
```

Request: GET /web/students. Response: Thymeleaf renders studentlist.html using the students value from the Model.

## 12.6 Read One Student

```
@GetMapping("/students/{id}")
public String showStudent(
        @PathVariable int id,
        Model model) {

    Student student = readStudentsFromJson().stream()
            .filter(item -> item.getStudentId() == id)
            .findFirst()
            .orElse(null);

    model.addAttribute("student", student);
    return "studentdetails";
}
```

The ID is part of the URL path, so @PathVariable is the clearest choice.

## 12.7 Show the Create Form

```
@GetMapping("/students/new")
public String showAddForm(Model model) {
    model.addAttribute("student", new Student());
    return "studentform";
}
```

## 12.8 Create a Student

```
@PostMapping("/students")
public String addStudent(
        @ModelAttribute Student student,
        Model model) {

    if (student.getFullName() == null ||
        student.getFullName().isBlank()) {
        model.addAttribute("error", "Full name is required");
        return "studentform";
    }

    List<Student> students = readStudentsFromJson();

    int newId = students.stream()
            .mapToInt(Student::getStudentId)
            .max()
            .orElse(0) + 1;

    student.setStudentId(newId);
    students.add(student);
    writeStudentsToJson(students);

    return "redirect:/web/students";
}
```

redirect: produces a redirect response instead of rendering a template immediately. The browser then sends a fresh GET request. This prevents accidental duplicate insertion when the user refreshes the result page.

## 12.9 Create Form Template

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Add Student</title>
</head>
<body>
<h1>Add Student</h1>

<p th:if="${error != null}" th:text="${error}"></p>

<form th:action="@{/web/students}"
      th:object="${student}"
      method="post">

    <label>Full Name</label>
    <input type="text" th:field="*{fullName}" required>

    <label>Gender</label>
    <select th:field="*{gender}">
        <option value="">Select gender</option>
        <option value="Male">Male</option>
        <option value="Female">Female</option>
    </select>

    <label>Age</label>
    <input type="number" th:field="*{age}" min="18" required>

    <label>City</label>
    <input type="text" th:field="*{city}">

    <label>State</label>
    <input type="text" th:field="*{state}">

    <button type="submit">Save Student</button>
</form>

<a th:href="@{/web/students}">Back to Student List</a>
</body>
</html>
```

## 12.10 Show the Edit Form

```
@GetMapping("/students/{id}/edit")
public String showEditForm(
        @PathVariable int id,
        Model model) {

    Student student = readStudentsFromJson().stream()
            .filter(item -> item.getStudentId() == id)
            .findFirst()
            .orElse(null);

    model.addAttribute("student", student);
    return "studenteditform";
}
```

## 12.11 Update a Student

```
@PostMapping("/students/{id}/update")
public String updateStudent(
        @PathVariable int id,
        @ModelAttribute Student updatedStudent) {

    List<Student> students = readStudentsFromJson();

    for (int index = 0; index < students.size(); index++) {
        if (students.get(index).getStudentId() == id) {
            updatedStudent.setStudentId(id);
            students.set(index, updatedStudent);
            break;
        }
    }

    writeStudentsToJson(students);
    return "redirect:/web/students";
}
```

The path identifies which existing student must change. @ModelAttribute contains the new form values. The code preserves the original ID before replacing the object in the list.

## 12.12 Edit Form Template

```
<form th:action="@{/web/students/{id}/update(id=${student.studentId})}"
      th:object="${student}"
      method="post">

    <input type="text" th:field="*{fullName}" required>
    <input type="number" th:field="*{age}" min="18" required>
    <input type="text" th:field="*{city}">
    <input type="text" th:field="*{state}">

    <button type="submit">Update Student</button>
</form>
```

Because the Model contains an existing Student, th:field automatically pre-populates the controls.

## 12.13 Delete a Student

```
@PostMapping("/students/{id}/delete")
public String deleteStudent(@PathVariable int id) {
    List<Student> students = readStudentsFromJson();

    students.removeIf(
            student -> student.getStudentId() == id);

    writeStudentsToJson(students);
    return "redirect:/web/students";
}
```

**Delete form inside studentlist.html**

```
<form th:action="@{/web/students/{id}/delete(id=${student.studentId})}"
      method="post"
      style="display:inline">
    <button type="submit">Delete</button>
</form>
```

Use a form rather than a GET link for deletion because GET requests should display data, not change it.

## 12.14 Student List Actions

```
<a th:href="@{/web/students/new}">Add New Student</a>

<ul>
    <li th:each="student : ${students}">
        <span th:text="${student.fullName}">Student Name</span>

        <a th:href="@{/web/students/{id}(id=${student.studentId})}">
            View Details
        </a>

        <a th:href="@{/web/students/{id}/edit(id=${student.studentId})}">
            Edit
        </a>

        <form th:action="@{/web/students/{id}/delete(id=${student.studentId})}"
              method="post"
              style="display:inline">
            <button type="submit">Delete</button>
        </form>
    </li>
</ul>
```

## 12.15 CRUD Request-Response Summary

```
CREATE
GET form → POST form data → bind Student → write JSON → redirect

READ ALL
GET list → read JSON → Model → Thymeleaf → HTML response

READ ONE
GET /{id} → extract path variable → find Student → details response

UPDATE
GET edit form → prefill with Student → POST changed data → write JSON → redirect

DELETE
POST /{id}/delete → remove matching Student → write JSON → redirect
```

## 12.16 Classroom Completion Checklist

- Students can explain the difference between query parameters and path variables.
- Students can explain @{...}, ${...}, and *{...} in Thymeleaf.
- Students can trace a form field from th:field to @ModelAttribute and a Java setter.
- Students can distinguish data binding from validation.
- Students can explain why client-side validation is not sufficient.
- Students can create, read, update, and delete a Student in data/students.json.
- Students can explain Model versus redirect responses.
- Students can explain why a delete operation uses POST rather than a GET link.
## 12.17 Concepts Intentionally Postponed

After JSON CRUD is fully understood, the same application can be refactored gradually. The recommended progression is: extract JSON access into a helper, introduce a service layer, introduce repository responsibility, replace JSON with PostgreSQL and Spring Data JPA, add Bean Validation, add exception handling, and finally build a REST API. These are later improvements, not requirements for this module.

# Module 13 — Pagination

Pagination divides a large result into smaller pages. With pageSize = 10, page 2 uses list indexes 10 through 19.

**Pagination calculation**

```
int pageSize = 10;
int totalStudents = studentList.size();
int totalPages = (int) Math.ceil((double) totalStudents / pageSize);

if (page < 1) page = 1;
if (page > totalPages && totalPages > 0) page = totalPages;

int startIndex = (page - 1) * pageSize;
int endIndex = Math.min(startIndex + pageSize, totalStudents);

List<Student> pageStudents = studentList.isEmpty()
    ? List.of()
    : studentList.subList(startIndex, endIndex);
```

## Pagination Links

**Pagination block**

```
<div th:if="${totalPages > 1}">
    <a th:if="${currentPage > 1}"
       th:href="@{/web/studentlist(
           page=${currentPage - 1},
           sort=${currentSort},
           dir=${currentDir},
           search=${keyword})}">
        « Previous
    </a>

    <a th:each="pageNumber : ${#numbers.sequence(1, totalPages)}"
       th:text="${pageNumber}"
       th:href="@{/web/studentlist(
           page=${pageNumber},
           sort=${currentSort},
           dir=${currentDir},
           search=${keyword})}">
        1
    </a>

    <a th:if="${currentPage < totalPages}"
       th:href="@{/web/studentlist(
           page=${currentPage + 1},
           sort=${currentSort},
           dir=${currentDir},
           search=${keyword})}">
        Next »
    </a>
</div>
```

| S |
| --- |

# Module 14 — Search and the Order of Operations

The correct processing order is Read → Filter → Sort → Paginate. Searching after pagination would inspect only the current page and miss matching students elsewhere in the list.

**Filter before sorting and pagination**

```
if (keyword != null && !keyword.trim().isEmpty()) {
    String lowerKeyword = keyword.trim().toLowerCase();

    studentList = studentList.stream()
        .filter(student ->
            student.getFullName().toLowerCase().contains(lowerKeyword)
            || student.getCity().toLowerCase().contains(lowerKeyword)
            || student.getState().toLowerCase().contains(lowerKeyword))
        .toList();
}
```

**Search form**

```
<form method="get" th:action="@{/web/studentlist}">
    <input type="text"
           name="search"
           th:value="${keyword}"
           placeholder="Search by name, city, or state">

    <input type="hidden" name="sort" th:value="${currentSort}">
    <input type="hidden" name="dir" th:value="${currentDir}">

    <button type="submit">Search</button>
    <a th:href="@{/web/studentlist}">Clear</a>
</form>
```

**Empty result message**

```
<p th:if="${#lists.isEmpty(students)}">
    No students found matching your search.
</p>
```

# Module 15 — Assemble the Read-Only Controller

The following controller combines file loading, filtering, sorting, pagination, list rendering, and details lookup. It represents the completed server-side logic from the lesson sequence.

**Complete WebController.java**

```
@GetMapping("/students/{id}")
public String showStudentDetails(
        @PathVariable("id") int studentId,
        Model model) {

    try {
        Student foundStudent = loadStudents().stream()
            .filter(student -> student.getStudentId() == studentId)
            .findFirst()
            .orElse(null);

        model.addAttribute("student", foundStudent);
    } catch (Exception e) {
        model.addAttribute("student", null);
    }

    return "studentdetails";
}
```

# Module 16 — Read-Only Thymeleaf Templates

## 13.1 studentlist.html

**src/main/resources/templates/studentlist.html**

```
<a class="details-btn"
   th:href="@{/web/students/{id}(id=${student.studentId})}">
    View Details
</a>
```

## 13.2 studentdetails.html

**src/main/resources/templates/studentdetails.html**

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Student Details</title>
    <style>
        body { font-family: Arial, sans-serif; padding: 24px; }
        .card { max-width: 520px; border: 1px solid #ccc; padding: 22px;
                border-radius: 8px; box-shadow: 0 2px 10px #ddd; }
        .profile-img { width: 150px; height: 150px; object-fit: cover;
                       border-radius: 50%; }
    </style>
</head>
<body>
    <div class="card" th:if="${student != null}">
        <img class="profile-img" th:src="${student.image}" alt="Student profile">
        <h1 th:text="${student.fullName}">Student Name</h1>
        <p><strong>Age:</strong> <span th:text="${student.age}">20</span></p>
        <p><strong>Gender:</strong> <span th:text="${student.gender}">Gender</span></p>
        <p><strong>Location:</strong>
           <span th:text="${student.city + ', ' + student.state}">City, State</span></p>
        <hr>
        <p><strong>Mother's Name:</strong>
           <span th:text="${student.motherName}">Name</span></p>
        <p><strong>Father's Name:</strong>
           <span th:text="${student.fatherName}">Name</span></p>
        <p><strong>Favorite Sports Person:</strong>
           <span th:text="${student.favSportsPerson}">Athlete</span></p>
        <p><strong>Remarks:</strong>
           <span th:text="${student.remarks}">Remarks</span></p>
    </div>

    <div th:if="${student == null}">
        <h1>Student Not Found</h1>
    </div>

    <p><a th:href="@{/web/studentlist}">← Back to Directory</a></p>
</body>
</html>
```

# Module 17 — Custom Error Pages

Spring Boot looks in templates/error for templates named after HTTP status codes. No custom controller is required for the basic 404 page.

**src/main/resources/templates/error/404.html**

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head><meta charset="UTF-8"><title>Page Not Found</title></head>
<body>
    <h1>404</h1>
    <h2>Oops! We lost that page.</h2>
    <p>The page you requested does not exist or has moved.</p>
    <a th:href="@{/web/studentlist}">Return to Directory</a>
</body>
</html>
```

**src/main/resources/templates/error/error.html**

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head><meta charset="UTF-8"><title>Something Went Wrong</title></head>
<body>
    <h1>System Error</h1>
    <p>We encountered an unexpected problem.</p>
    <a th:href="@{/web/studentlist}">Go Back to Directory</a>
</body>
</html>
```

## HTTP Status Code Review

- 2xx: request completed successfully.
- 4xx: the request has a client-side problem, such as a missing path.
- 5xx: the server failed while processing the request.
# Module 18 — Classroom Testing Checklist

| Test | Action | Expected result |
| --- | --- | --- |
| REST endpoint | Open / | Plain welcome message |
| HTML rendering | Open /web/home | Dynamic username and day |
| List loading | Open /web/studentlist | Student records from JSON |
| Ascending sort | Click Age once | Youngest first |
| Descending sort | Click Age again | Oldest first |
| Pagination | Open page 2 | Next set of students |
| Search | Search Mumbai | Only matching records |
| State retention | Sort search results and change page | Search and sort remain active |
| Details | Click View Details | Correct student profile |
| Missing ID | Open /web/students/9999 | Student Not Found |
| 404 | Open /web/unknown | Custom 404 page |

## Debugging Guide

Port already in use: Change server.port or stop the process using the current port.

Template not found: Check that the file is under src/main/resources/templates and that the controller returns the filename without .html.

Model value is blank: Verify that the Java Model key and Thymeleaf expression match exactly, including capitalization.

JSON cannot be read: Validate JSON syntax and ensure property names match Student fields.

No getter error: Add the required public getter to Student.

Ambiguous mapping: Find duplicate final paths across controllers and change a class-level or method-level mapping.

Sort disappears: Preserve sort, dir, and search in links and hidden form fields.

# Module 19 — Practice and Later Extensions

## Guided Exercises

1. Add email and course fields to Student, JSON, the list, and the details page.
1. Add sorting by studentId.
1. Search by favorite sports person.
1. Display a label showing “Page X of Y”.
1. Highlight the active page number.
1. Show the total number of matching students.
## Later Refactoring Tasks (After JSON CRUD)

1. Move JSON read/write operations into a StudentService only after students can explain the controller-only version.
1. Replace the JSON file with PostgreSQL and Spring Data JPA while preserving the URLs and templates.
1. Refactor the completed JSON CRUD form validation to Bean Validation annotations.
1. Write controller tests for list, search, and details routes.
## Exit Questions

- Why must filtering happen before pagination?
- What information is lost when query parameters are not retained in links?
- How does ${student.fullName} connect to Java code?
- Why does the details route need a stable studentId?
- Which responsibilities should eventually move out of the controller?

# Current Classroom Example: Todo List with a Writable JSON File

This example implements Todo CRUD without a database. The application reads and writes an external JSON file using Jackson.

## Todo File Location

Create the writable file at the project root, outside `src/main/resources`:

```text
demo/
├── data/
│   └── todos.json
├── src/
└── pom.xml
```

Start it with an empty JSON array:

```json
[]
```

Reference it with:

```java
private final File todoFile = new File("data/todos.json");
```

This is a relative path, resolved from the application's working directory. When the project runs from `C:\spring4`, it refers to `C:\spring4\data\todos.json`, not `C:\spring4\src\data\todos.json`. To diagnose path confusion, print:

```java
System.out.println(todoFile.getAbsolutePath());
```

Use `src/main/resources` for read-only packaged data. Use an external `data` directory when create, update, or delete operations must persist while the application runs.

## Create the `Todo` Type with Lombok

`List<Todo>` requires a Java type describing one item. For mutable CRUD data, a regular class with Lombok-generated constructors, getters, and setters is convenient:

```java
package com.example.demo;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Todo {
    private int id;
    private String title;
    private boolean completed;
    private String description;
}
```

Add Lombok to `pom.xml`:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

A Java record can also represent a Todo:

```java
public record Todo(
        int id,
        String title,
        boolean completed,
        String description
) {}
```

Records are not limited to reading data. Jackson can deserialize JSON into records and serialize records back to JSON. Records are immutable, however, so an update creates a new record instead of calling setters. A mutable Lombok class is often easier for an introductory CRUD exercise.

## Role of `ObjectMapper`

`ObjectMapper` converts between JSON and Java objects:

```text
todos.json
    ↓ readValue()
List<Todo>
    ↓ Java add/update/delete operations
List<Todo>
    ↓ writeValue()
todos.json
```

Declare it once in the controller:

```java
private final ObjectMapper mapper = new ObjectMapper();
```

In Spring Boot 4 with Jackson 3, the imports are:

```java
import tools.jackson.core.type.TypeReference;
import tools.jackson.databind.ObjectMapper;
```

## Reusable Read and Write Helpers

Use one helper to load the complete JSON array and another to persist the modified list:

```java
private List<Todo> readTodos() {
    try {
        if (!todoFile.exists()) {
            todoFile.getParentFile().mkdirs();
            mapper.writerWithDefaultPrettyPrinter()
                    .writeValue(todoFile, new ArrayList<Todo>());
        }

        return mapper.readValue(
                todoFile,
                new TypeReference<List<Todo>>() {}
        );
    } catch (Exception e) {
        throw new RuntimeException("Unable to read todos.json", e);
    }
}

private void writeTodos(List<Todo> todos) {
    try {
        mapper.writerWithDefaultPrettyPrinter()
                .writeValue(todoFile, todos);
    } catch (Exception e) {
        throw new RuntimeException("Unable to write todos.json", e);
    }
}
```

`TypeReference<List<Todo>>` tells Jackson that the root JSON array contains `Todo` objects. `writerWithDefaultPrettyPrinter()` makes the saved file readable but does not change its data.

## Read All Todos

```java
@GetMapping("/todos")
@ResponseBody
public List<Todo> getAllTodos() {
    return readTodos();
}
```

Request:

```text
GET http://localhost:8080/web/todos
```

If the response is `[]`, the endpoint is working but the file being read contains an empty JSON array. Check the absolute path and edit `data/todos.json` at the project root.

## Add a Todo

The client supplies only fields it is allowed to choose:

```java
package com.example.demo;

public record TodoPostReq(String title, String description) {}
```

The server generates the ID and initial completion state:

```java
@PostMapping("/addTodo")
@ResponseBody
public Todo addTodoItem(@RequestBody TodoPostReq request) {
    List<Todo> todos = readTodos();

    int newId = todos.stream()
            .mapToInt(Todo::getId)
            .max()
            .orElse(0) + 1;

    Todo newTodo = new Todo(
            newId,
            request.title(),
            false,
            request.description()
    );

    todos.add(newTodo);
    writeTodos(todos);
    return newTodo;
}
```

Use the maximum existing ID plus one. `todos.size() + 1` can create a duplicate ID after an item has been deleted.

The browser can submit JSON with:

```javascript
function addTodo(event) {
    event.preventDefault();

    fetch("/web/addTodo", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            title: document.todoform.title.value,
            description: document.todoform.description.value
        })
    })
    .then(response => {
        if (!response.ok) {
            throw new Error("Unable to add todo");
        }
        return response.json();
    })
    .then(todo => {
        console.log("Added:", todo);
        document.todoform.reset();
    })
    .catch(error => console.error(error));
}
```

Use `.value` to send the entered text rather than the HTML input element itself. Returning the created `Todo` provides JSON for `response.json()`; a `void` controller response has no JSON body to parse.

## Read One Todo by ID

`ObjectMapper` converts the entire JSON array into `List<Todo>`. Java then searches that list because a plain JSON file cannot perform a database-style indexed query.

```java
@GetMapping("/todos/{id}")
@ResponseBody
public ResponseEntity<Todo> getTodoById(@PathVariable int id) {
    return readTodos().stream()
            .filter(todo -> todo.getId() == id)
            .findFirst()
            .map(ResponseEntity::ok)
            .orElseGet(() -> ResponseEntity.notFound().build());
}
```

Request Todo 2 with:

```text
GET http://localhost:8080/web/todos/2
```

The `.map(ResponseEntity::ok)` call is an `Optional.map()` operation, not Jackson's `ObjectMapper`.

`ResponseEntity` is not required merely to return a Todo, but it lets the controller accurately represent different outcomes. A found Todo returns `200 OK`; a missing ID returns `404 Not Found` instead of an ambiguous empty `200 OK` response.

## Update a Todo

Use `PUT` when the client sends the complete set of editable Todo values. Keep the identity in the path and do not accept a replacement ID from the request body:

```java
package com.example.demo;

public record TodoUpdateReq(
        String title,
        String description,
        boolean completed
) {}
```

The endpoint finds the Todo, changes the mutable Lombok object, persists the list, and returns the updated object:

```java
@PutMapping("/todos/{id}")
@ResponseBody
public ResponseEntity<Todo> updateTodo(
        @PathVariable int id,
        @RequestBody TodoUpdateReq request) {

    List<Todo> todos = readTodos();

    return todos.stream()
            .filter(todo -> todo.getId() == id)
            .findFirst()
            .map(todo -> {
                todo.setTitle(request.title());
                todo.setDescription(request.description());
                todo.setCompleted(request.completed());

                writeTodos(todos);
                return ResponseEntity.ok(todo);
            })
            .orElseGet(() -> ResponseEntity.notFound().build());
}
```

Add the import:

```java
import org.springframework.web.bind.annotation.PutMapping;
```

Example browser request:

```javascript
function updateTodo(id) {
    fetch(`/web/todos/${id}`, {
        method: "PUT",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            title: "Learn Spring Boot CRUD",
            description: "Complete the update operation",
            completed: true
        })
    })
    .then(response => {
        if (response.status === 404) {
            throw new Error("Todo not found");
        }
        if (!response.ok) {
            throw new Error("Unable to update Todo");
        }
        return response.json();
    })
    .then(todo => console.log("Updated:", todo))
    .catch(error => console.error(error));
}
```

### Why Both `filter()` and `map()` Appear

The operations have different responsibilities:

```text
filter()     → selects the Todo whose ID matches
findFirst()  → produces Optional<Todo>
map()        → updates the found Todo and creates the success response
orElseGet()  → creates the 404 response when nothing matched
```

`map()` alone does not search for one Todo; a stream `map()` transforms every element. With an immutable record, transforming the complete list can be useful because the matching record must be replaced with a new value. With the current mutable Lombok class, finding one Todo and updating it through setters is clearer.

## Delete a Todo

Use HTTP DELETE and return `204 No Content` when removal succeeds:

```java
@DeleteMapping("/todos/{id}")
@ResponseBody
public ResponseEntity<Void> deleteTodo(@PathVariable int id) {
    List<Todo> todos = readTodos();

    boolean removed = todos.removeIf(
            todo -> todo.getId() == id
    );

    if (!removed) {
        return ResponseEntity.notFound().build();
    }

    writeTodos(todos);
    return ResponseEntity.noContent().build();
}
```

Required imports include:

```java
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
```

Browser JavaScript can call the endpoint with:

```javascript
function deleteTodo(id) {
    fetch(`/web/todos/${id}`, {
        method: "DELETE"
    })
    .then(response => {
        if (response.status === 204) {
            console.log("Todo deleted");
            return;
        }

        if (response.status === 404) {
            throw new Error("Todo not found");
        }

        throw new Error("Unable to delete todo");
    })
    .catch(error => console.error(error));
}
```

Do not call `response.json()` for a `204 No Content` response because it intentionally has no body.

### Common Delete Bug: Modifying a Different List

This code does not persist the removal:

```java
List<Todo> todos = readTodos();
boolean removed = readTodos().removeIf(todo -> todo.getId() == id);
writeTodos(todos);
```

The first call produces one list, while the second call reads a separate list. The second list is modified, but the unchanged first list is written back to the file.

All operations must use the same list instance:

```java
List<Todo> todos = readTodos();
boolean removed = todos.removeIf(todo -> todo.getId() == id);
writeTodos(todos);
```

`removeIf()` deletes the matching object only from the in-memory list. `writeTodos()` makes that change persistent by overwriting `todos.json` with the updated list. The returned boolean is `true` when an item matched and `false` when the ID was not found.

# Spring Ecosystem Learning Roadmap

Spring is modular. Learn the projects required for ordinary backend development first, then select specialized projects according to real application requirements. Do not try to learn every Spring project at the same time.

## Current Position

The material completed so far covers:

- Spring Boot project generation, configuration, and embedded Tomcat
- Servlet foundations and the Spring MVC request lifecycle
- MVC controllers and HTTP method mappings
- Query parameters, path variables, form data, and JSON request bodies
- `ResponseEntity` and HTTP status codes
- Jackson, `ObjectMapper`, and file-based JSON storage
- Basic Thymeleaf pages and forms
- Create, read, update, and delete operations

The immediate goal is to finish the Todo application and then separate it into controller, service, and repository responsibilities.

## Phase 0 — Java, HTTP, SQL, and Build Prerequisites

### Java

Be comfortable with:

- Classes, objects, interfaces, and constructors
- Encapsulation, inheritance, and polymorphism
- Exceptions
- Generics
- `List`, `Set`, and `Map`
- Streams, lambdas, method references, and `Optional`
- Records and enums
- Date and time APIs
- Basic file handling and concurrency
- Annotation and reflection concepts

### HTTP

Understand:

- Requests and responses
- HTTP methods
- URLs, paths, and query strings
- Headers, cookies, and sessions
- Request bodies
- JSON and form encoding
- Content types
- Status codes
- Stateless communication
- CORS
- Authentication versus authorization

Example request:

```http
PUT /api/todos/10
Content-Type: application/json
Accept: application/json
```

`Content-Type` describes the request body; `Accept` describes the response format requested by the client.

### SQL

Before database persistence, learn:

- Tables, rows, and columns
- Primary and foreign keys
- `SELECT`, `INSERT`, `UPDATE`, and `DELETE`
- Joins
- Constraints
- Indexes
- Normalization
- Transactions

### Maven

Understand `pom.xml`, dependencies, plugins, scopes, the build lifecycle, and the Maven Wrapper:

```powershell
.\mvnw.cmd clean
.\mvnw.cmd test
.\mvnw.cmd package
.\mvnw.cmd spring-boot:run
```

**Milestone:** Build a plain Java Todo application using collections and files without Spring.

## Phase 1 — Servlet Foundations

Learn enough Servlet API to understand what Spring MVC abstracts:

- Servlet containers and Tomcat
- `HttpServletRequest` and `HttpServletResponse`
- Servlet lifecycle
- Filters and `FilterChain`
- Sessions and cookies
- Forward versus redirect
- Thread-per-request processing
- Thread safety
- `DispatcherServlet`

Do not spend months building servlet-only applications. The goal is to understand the web foundation beneath Spring MVC.

**Milestone:** Create one raw servlet that accepts two parameters, calculates the greatest number, and writes the response.

## Phase 2 — Spring Core

Spring Core provides the IoC container and dependency injection foundation.

### IoC and dependency injection

Learn:

- Inversion of Control
- `ApplicationContext`
- Spring beans
- Component scanning
- Constructor injection
- Bean lifecycle
- Circular dependency problems

Important annotations include:

```java
@Component
@Service
@Repository
@Configuration
@Bean
```

Constructor injection example:

```java
@Service
public class TodoService {

    private final TodoRepository repository;

    public TodoService(TodoRepository repository) {
        this.repository = repository;
    }
}
```

Also learn component annotations, Java configuration with `@Bean`, and the purpose of older XML configuration. Use `@Bean` when object construction must be explicit or the class belongs to an external library.

### Scopes and lifecycle

Learn singleton, prototype, request, session, and application scopes, plus initialization and destruction callbacks such as `@PostConstruct` and `@PreDestroy`.

### Configuration and profiles

Learn:

```java
@Value
@ConfigurationProperties
@Profile
```

### AOP foundations

Understand aspect, advice, pointcut, proxy, and the self-invocation limitation. Spring AOP supports capabilities such as:

```java
@Transactional
@Cacheable
@Async
```

Start by understanding proxy behavior rather than writing complex custom aspects.

**Milestone:** Refactor the Todo logic into `TodoService`, a `TodoRepository` interface, and a `JsonTodoRepository` implementation connected through constructor injection.

## Phase 3 — Spring Boot

Learn:

- `@SpringBootApplication`
- Auto-configuration
- Starter dependencies
- Embedded servers
- Component-scan boundaries
- Executable JAR packaging
- Properties and YAML configuration
- Profiles
- DevTools
- Logging

`@SpringBootApplication` combines the main ideas of:

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

Use environment-specific configuration:

```text
application.properties
application-dev.properties
application-test.properties
application-prod.properties
```

Prefer typed configuration for related properties:

```java
@ConfigurationProperties(prefix = "todo.storage")
public record TodoStorageProperties(Path location) {}
```

Never commit real production passwords or secrets.

**Milestone:** Run the layered Todo application with a configurable JSON location, development/test profiles, logging, and executable JAR packaging.

## Phase 4 — Spring MVC and REST

Master:

```java
@Controller
@RestController
@RequestMapping
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

### Request and response handling

Learn:

```java
@PathVariable
@RequestParam
@RequestBody
@RequestHeader
@CookieValue
@ModelAttribute
@RequestPart
MultipartFile
@ResponseBody
ResponseEntity
```

Also study:

- Content negotiation
- HTTP message converters
- Jackson serialization and deserialization
- File upload and download
- Filters and MVC interceptors
- Global CORS configuration
- Pagination, sorting, filtering, and API versioning
- Safe methods, idempotency, and HTTP caching concepts

Use resource-oriented routes:

```text
POST   /api/todos
GET    /api/todos
GET    /api/todos/{id}
PUT    /api/todos/{id}
PATCH  /api/todos/{id}
DELETE /api/todos/{id}
```

**Milestone:** Turn the Todo application into a complete REST API with correct routes, JSON bodies, status codes, and error outcomes.

## Phase 5 — DTOs, Validation, and Error Handling

Create separate API contracts:

```text
CreateTodoRequest
UpdateTodoRequest
TodoResponse
```

Do not allow clients to control server-generated values such as IDs.

Learn Jakarta Bean Validation:

```java
@NotNull
@NotBlank
@Size
@Min
@Max
@Email
@Pattern
@Positive
@Past
@Future
@Valid
```

Example:

```java
public record CreateTodoRequest(
        @NotBlank
        @Size(max = 100)
        String title,

        @Size(max = 500)
        String description
) {}
```

Learn centralized error handling:

```java
@ControllerAdvice
@RestControllerAdvice
@ExceptionHandler
@ResponseStatus
ProblemDetail
```

Return consistent errors:

```json
{
  "status": 404,
  "title": "Todo not found",
  "detail": "Todo 20 does not exist"
}
```

**Milestone:** Add DTOs, validation, `TodoNotFoundException`, and global exception handling to the Todo API.

## Phase 6 — Thymeleaf and Server-Rendered MVC

Learn this phase deeply when building server-rendered websites:

- Template resolution
- `Model` and `ModelAndView`
- `th:text`, `th:if`, and `th:each`
- `th:href` and `th:action`
- `th:object` and `th:field`
- Form binding and validation messages
- Template fragments and layouts
- Static resources
- Redirect and flash attributes
- Post/Redirect/Get

If a separate React or Angular frontend consumes the REST API, learn Thymeleaf fundamentals but prioritize REST.

**Milestone:** Create a Thymeleaf Todo interface using the same service layer as the REST controller.

## Phase 7 — Application Architecture

Move from a large controller to separated responsibilities:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Storage
```

Study separation of concerns, dependency inversion, domain models, DTO mapping, business rules, exceptions, transactions, and package organization.

Suggested package structure:

```text
com.example.todo
├── controller
├── dto
├── service
├── repository
├── domain
├── exception
└── config
```

Start with a well-structured modular monolith. Do not jump directly to microservices.

## Phase 8 — JDBC, JPA, Hibernate, and Spring Data

### Spring JDBC

Learn `DataSource`, `JdbcTemplate`, parameter binding, row mapping, and transaction management. This reveals the SQL operations that an ORM later abstracts.

### JPA and Hibernate

Learn:

```java
@Entity
@Id
@GeneratedValue
@Column
@Table
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

Understand:

- Persistence context and entity states
- Dirty checking
- Lazy versus eager loading
- Cascades and orphan removal
- JPQL and fetch joins
- The N+1 query problem
- Optimistic locking
- Schema migrations with Flyway or Liquibase

### Spring Data JPA

Learn:

```java
JpaRepository
Pageable
Page
Sort
@Query
```

Example:

```java
public interface TodoRepository
        extends JpaRepository<Todo, Long> {

    List<Todo> findByCompleted(boolean completed);
}
```

Use H2 for initial exercises, then PostgreSQL or MySQL for realistic development.

**Milestone:** Replace `todos.json` with PostgreSQL while preserving the REST API contract.

## Phase 9 — Transaction Management

Learn:

```java
@Transactional
```

Understand atomicity, commit, rollback, isolation, transaction boundaries, read-only transactions, exception behavior, proxy-based transaction management, self-invocation, and lazy loading.

Place business transaction boundaries primarily in the service layer:

```java
@Service
public class TodoService {

    @Transactional
    public Todo updateTodo(...) {
        // Perform one complete business operation
    }
}
```

**Milestone:** Implement an operation that modifies multiple database records atomically.

## Phase 10 — Testing

### Unit tests

Learn JUnit, AssertJ, Mockito, stubs, mocks, and testing services without loading Spring.

### MVC tests

Learn:

```java
@WebMvcTest
MockMvc
```

Test mappings, JSON conversion, validation, status codes, headers, and error responses.

### Integration tests

Learn:

```java
@SpringBootTest
@DataJpaTest
```

Then study test profiles, transactional rollback, Testcontainers, database integration tests, and end-to-end tests.

**Milestone:** Test valid creation, invalid input, existing and missing IDs, updates, deletion, filtering, and pagination.

## Phase 11 — Spring Security

Learn in this order:

1. Authentication versus authorization
2. Password hashing
3. `SecurityFilterChain`
4. Form login
5. Session authentication
6. Roles and authorities
7. Method security
8. CORS and CSRF
9. JWT-based APIs
10. OAuth 2.0 and OpenID Connect

Important APIs include:

```java
UserDetailsService
PasswordEncoder
SecurityFilterChain
@PreAuthorize
```

Learn normal session-based authentication before starting JWT.

**Milestone:** Allow users to manage only their own Todos while administrators can manage all Todos.

## Phase 12 — Production Readiness

Learn:

- Structured logging
- Spring Boot Actuator
- Health checks
- Metrics and Micrometer
- Environment variables and secret management
- Graceful shutdown
- Database connection pooling
- Error monitoring
- Docker
- CI/CD
- Reverse proxies and HTTPS
- Performance profiling

Do not expose sensitive Actuator endpoints publicly.

**Milestone:** Package the Todo application with Docker, PostgreSQL, health checks, and production configuration.

## Phase 13 — Common Application Capabilities

Learn these when required:

### Scheduling and asynchronous execution

```java
@Scheduled
@Async
```

Study thread pools, error handling, and task cancellation.

### Caching

```java
@EnableCaching
@Cacheable
@CachePut
@CacheEvict
```

Start with local caching and then learn Redis.

### External APIs

Learn HTTP clients, timeouts, authentication headers, retries, error mapping, resilience, and client testing.

### WebSockets

Use WebSockets for bidirectional real-time features such as chat and live notifications.

## Phase 14 — Messaging and Event-Driven Systems

Choose according to project requirements:

- Spring for Apache Kafka
- Spring AMQP with RabbitMQ
- Spring for Apache Pulsar
- Spring Integration

Learn producers, consumers, topics, queues, acknowledgements, consumer groups, retries, dead-letter queues, idempotency, eventual consistency, and the transactional outbox pattern.

**Milestone:** Publish a `TodoCompleted` event and process it asynchronously to create an activity log.

## Phase 15 — Optional Specializations

### Spring WebFlux

Use for genuinely reactive, non-blocking workloads. Learn Reactor, `Mono`, `Flux`, `WebClient`, backpressure, and R2DBC. Avoid combining blocking JPA operations carelessly with reactive request processing.

### Spring Batch

Use for high-volume scheduled or offline processing:

```text
ItemReader → ItemProcessor → ItemWriter
```

### Spring Cloud

Learn after building and deploying a strong monolith. Topics include configuration, gateway, service discovery, circuit breakers, distributed tracing, OpenFeign, messaging, and Kubernetes integration.

### Spring Modulith

Use it to structure and verify a large Spring Boot monolith around domain-oriented application modules.

### Other projects

- Spring for GraphQL for flexible graph-shaped APIs
- Spring AI for model clients, embeddings, vector stores, RAG, and tool calling
- Spring gRPC for strongly typed service-to-service communication
- Spring REST Docs for test-generated API documentation
- Spring Session for distributed session management

## Recommended Project Sequence

Build these projects in order:

1. JSON Todo CRUD — current project
2. Layered Todo API — controller, service, and repository
3. Validated Todo API — DTOs and global errors
4. Database Todo API — JPA and PostgreSQL
5. Tested Todo API — unit, MVC, and integration tests
6. Secure Todo API — users, roles, and ownership
7. Production Todo service — Actuator, Docker, and configuration
8. E-commerce monolith — relationships and transactions
9. Event-driven order system — Kafka or RabbitMQ
10. Microservices system — only after mastering the monolith

## Employment-Focused Priority

Master these first:

```text
Java
HTTP and SQL
Spring Core
Spring Boot
Spring MVC
REST API design
Validation and exception handling
Layered architecture
Spring Data JPA
Transactions
Testing
Spring Security
Git, Docker, and PostgreSQL
```

Treat these as later specializations:

```text
WebFlux
Spring Cloud
Kafka
Batch
GraphQL
Spring AI
gRPC
Kubernetes
```

## Immediate Next Steps

```text
Finish Todo update
    ↓
Separate TodoController
    ↓
Create TodoService
    ↓
Create JsonTodoRepository
    ↓
Add validation and global exception handling
    ↓
Write MockMvc tests
    ↓
Replace JSON with PostgreSQL and Spring Data JPA
```

Continue with the [official Spring Guides](https://spring.io/guides/) and the [Spring Projects catalog](https://spring.io/projects/) when beginning each new phase.

# Appendix A — Final Project Structure

**Final structure**

```
demo/
├── pom.xml
├── mvnw
├── mvnw.cmd
└── src/main/
    ├── java/com/student/demo/
    │   ├── DemoApplication.java
    │   ├── HelloController.java
    │   ├── Student.java
    │   └── WebController.java
    └── resources/
        ├── application.properties
        ├── students.json
        └── templates/
            ├── home.html
            ├── studentlist.html
            ├── studentdetails.html
            └── error/
                ├── 404.html
                └── error.html
```

# Appendix B — Annotation Reference

| Annotation | Purpose |
| --- | --- |
| @Controller | Marks a server-rendered MVC controller. |
| @RestController | Marks a controller whose return values become response bodies. |
| @RequestMapping | Defines a base path or general request mapping. |
| @GetMapping | Maps HTTP GET requests. |
| @RequestParam | Reads a query parameter. |

# Appendix C — Key Vocabulary

Dependency: A library required by the application.

Starter: A curated Spring Boot dependency bundle.

Auto-configuration: Spring Boot configuration inferred from dependencies and application context.

Controller: A class that handles web requests.

Model: Data made available to a template.

Template: Server-side HTML containing expressions and attributes.

Query parameter: A key-value value in the URL after ?.

POJO: A simple Java object used to represent data.

Jackson: The JSON serialization and deserialization library used by Spring Boot.

Pagination: Dividing a result into pages.

## Form and CRUD Vocabulary

Data binding: Matching submitted request field names to Java object properties and converting values to Java types.

th:object: Selects the object that a Thymeleaf form is bound to.

th:field: Connects one form control to one property of the selected form object.

@ModelAttribute: Creates or receives an object whose properties are populated from request parameters.

Client-side validation: Browser checks such as required, minlength, min, and max before submission.

Server-side validation: Java checks performed after the request reaches the application.

Redirect: A response that instructs the browser to send a new request to another URL.

CRUD: Create, Read, Update, and Delete operations.

Writable JSON file: An external file such as data/students.json that the running application can modify.

# Source and Editing Note

This classroom edition was reorganized from the supplied Gemini learning conversation and the subsequent Spring MVC teaching discussion. The document deliberately presents a controller-first JSON CRUD implementation so students can understand request-response flow before architectural best practices are introduced.
