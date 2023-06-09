# Notes:
<!-- [links to demo](/Users/lizicheng/Desktop/java/spring/spring-boot-3-spring-6-hibernate-for-beginners-main/01-spring-boot-overview/03-actuator-demo) -->

--
Actuator
---
how: `starter-actuator`					

<br>
`/health`
`/info`
`/beans`
`/mappings`
`/threaddump`
`/auditevents`

--
common properties:
`core web security`  
log levels: `TRACE INFO DEBUG`  
context path of the app: `servlet.context-path=/my-silly-app/hello`  
Actuator: `exclude=beans,mapping` and `include=*`  
Security: `.user.name=admin` and `.user.password=8888`  
[links to spring props](https://www.luv2code.com/spring-boot-props)

--
### Inversion of Control(IoC)  
The approach of outsourcing the construction and management of objects.  
Spring container is a factory pattern  
configuration for Spring container by:  
1. java annotations  
2. java source code  

---
### Depndency injection
1. constructor injection
2. setter injection

---

![ioc exa](ioc-ex.png)  

![ioc exa](ioc-de.png)  

`@Componment` is a SpringBean  
`@Autowired` Spring look for a class that match, by type: class or interface  
`no usages` because Spring do the work behind  
### behind the scenes  
Help you automatically new objects  
### Setter injection  
use the autowried for setter methods

---
component scanning: scanning the same packages  
explicitly scanning: scanBasePackages={"www.luv.com"}  

---
for multiple beans, we can use `@Qualified`  
`@Qualifier("baseballCoach")` the bean id is the first letter is lowercase, the same as the class name  
We can use `@Primary` on the class name.
`@Qualifier` has higher priority than `@Primary` 

---
By default, all beans are initialized, every class will be newed.  
### lazy initialization  
only when the class is needed, created it. We can use `@Lazy`  
global configuration: `spring.main.lazy-initialization=ture`  
### beans scope
the lifecycle of bean  
how many objects are created  
how long the bean lives  
The default bean is singleton  
Explicit bean scope: `@Scope` use this at the class file
* protottpe scope: `@Scope socpe_prototype` 
* request
* session
* global-session  

`@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)`  
`@GetMapping("/check")
public String check() {
return "Compare the two coaches: coach == anothercoach, " + (myCoach==anotherCoach);
}`  
### lifecycle of bean
Bean lifecycle method  
`@PostConstruct` once Spring started running  
`@PreDestroy` once Spring finished running  
**For "prototype" scoped beans, Spring does not call the destroy method. Gasp!**  
**Prototype beans are lazy by default. There is no need to use the @Lazy annotation for prototype scopes beans.**    

In contrast to the other scopes, Spring does not manage the complete lifecycle of a prototype bean: the container instantiates, configures, and otherwise assembles a prototype object, and hands it to the client, with no further record of that prototype instance.

Thus, although initialization lifecycle callback methods are called on all objects regardless of scope, in the case of prototypes, configured destruction lifecycle callbacks are not called. The client code must clean up prototype-scoped objects and release expensive resources that the prototype bean(s) are holding.  

### configuration for beans  
used `@Beans` because sometimes have no access to the thrid party source code, but need to make that class available for the bean  
Example: AWS S3 cloud, we want to communicate our app with the S3 service.  
The bean id is default to the method name. `public Coach swimCoach()` change the id of bean `@Bean("badname")`  

# 03

### Hibernate & JPA  
Hibernate is a framework that persisting the Java objects in the database.  
* handle low level sql operations
* provide Object-to-Relational Mapping

It can map the Java code to sql create table  
JPA: Jakarta Persistence API, it is a specification  
JPA allow us to do sql jobs using only Java. JPA is another abstract layer on top of JDBC. `EntityManager`  
We can use commandlineRunner to execute: `return runner -> {
System.out.println("Hello World!");
}`  
you can set up the connection to database: `spring.datasource.url=jdbc:mysql://localhost:3306/student_tracker`  
`Entity Classes` is a java class that maps the database table. Annotations `@Entity`. The class must have public or protected constructor.  
`@Column`, `@Table` is optional, but recommended. The default name of the table and column will be the same as the class name.  
`GenerationType.AUTO`, `GenerationType.IDENTITY`, `GenerationType.SEQUENCE`, `GenerationType.TABLE` 
### Bonus: 
you can define your own custom generation strategy.  
`Project Lombok` 
### CRUD  
Create, read,update and delete  
DAO: Date Access Object, kind of like a helper class for communicate with the database  
Steps: 
1. Define DAO interface
2. Define DAO implementation, inject the entity manager
3. update main app  

saving a java object
handled transaction management: `@Transactional`, for DAO: `@Repository` just like `@Component`  

The difference between beans and components:  
1. @Component is a class-level annotation, but @Bean is at the method level, so @Component is only an option when a class's source code is editable. @Bean can always be used, but it's more verbose.  
2. @Component is compatible with Spring's auto-detection, but @Bean requires manual class instantiation.  
3. Using @Bean, decouples the instantiation of the bean from its class definition. This is why we can use it to make third-party classes into Spring beans. It also means we can introduce logic to decide which of several possible instance options for a bean to use.

creating a new method `private void createStudent(StudentDAO studentDAO)` then `studentDAO.save(tempStudent);`, inside the DAOimplement `public void save(Student theStudent) {
entityManager.persist(theStudent);
}`  
### save multiple students
we can test multiple students in Java.  
we can change the index of MySQL `ALTER TABLE student_tracker.student AUTO_INCREMENT=1000`  
we can also TRUNCATE the index of MySQL `TRUNCATE student_tracker.student`  
### retrieve objects
we don't need to add `@Transactional` because we just retrieved the objects, no modification.
use:`@Override
public Student findById(Integer id) {
return entityManager.find(Student.class, id);
}`  
Steps:
1. Define DAO interface
2. Define DAO implementation, inject the entity manager
3. update main app
## !important  
`public Student() {
    }`  
In JPA and some beans, we have to have a default constructor, otherwise it will throw an exception.  
`Student myStudent = studentDAO.findById(theId);`  
### query objects
JPA have its query language called JPQL, based on the entity name and entity field name  
we can first create a query object using TypedQuery  
`TypedQuery<Student> query = entityManager.createQuery("FROM Student order by lastName desc", Student.class);`  
inside the query, we can say order by lastName desc.  
Then we can return the query  
`return query.getResultList();`  
we can also have a method:  
`private void queryStudent(StudentDAO studentDAO) {
List<Student> students = studentDAO.findAll();
        for (Student student : students) {
            System.out.println(student);
        }
	}`  
**note that all name in the query is entity field name, not the real column name or table name in MySQL.** e.g. lastName  
JPQL named parameters are prefixed with a colon, think of that as the placeholder, `:lastName`  
the code: 
``` 
public List<Student> findByLastName(String thelastName) {
        // create query
        TypedQuery<Student> query = entityManager.createQuery("FROM Student where lastName = :lastName order by lastName desc", Student.class);

        // set parameter
        query.setParameter("lastName", thelastName);

        // return query results
        return query.getResultList();
    }
   private void queryStudentByLastname(StudentDAO studentDAO) {
		// get a list of all students
		List<Student> students = studentDAO.findByLastName("jin");

		//display all students
		for (Student student : students) {
            System.out.println(student);
        }
	}
```
### update object
 we need to add `@Transactional` 
code:
```
public void update(Student theStudent) {
        entityManager.merge(theStudent);
    }
    
    private void updateStudent(StudentDAO studentDAO) {
		int id=1;
		System.out.println("Retrieving student with id: " + id);
		Student student = studentDAO.findById(id);

		student.setFirstName("Scooby");
        System.out.println("Updating student with id: " + id);
		studentDAO.update(student);
		System.out.println("Updated student: " + student);
	}
```  
### delete object
code:
```
@Transactional
    public int deleteAll() {
        int numberOfRows = entityManager.createQuery("DELETE FROM Student").executeUpdate();
        return numberOfRows;
    }
    private void deleteAllStudent(StudentDAO studentDAO) {
		System.out.println("Deleting all students");
		int count=studentDAO.deleteAll();
		System.out.println("Deleted "+count+" rows students");
	}
```  
### create table using Java
inside the property file:  
PROPERTY-VALUE  
1. none
2. create-onlu
3. **create** this will first drop table if existed, then create table. Don't recommend for company use because it will lose any previous data.
4. drop
5. create-drop
6. validate
7. **update** it will keep the existing data

Be caution with update model. it will keep updated the table with latest code.  
### REST API 
business problems: we want to create an app provides the weather report for a city  
Application architecture: my weather app will pass a `string` city name to the external service to give the data.  
we can make REST API calls over HTTP in order to connect to the service. REST is independent to language, the client can use any language, and service can use any language.  
REST can use any data format. XML, JSON  
use the online weather service API provided by [API](https://www.openweathermap.org) we can read the API documentation. 
REST calls can be made over HTTP  
We will create a CRM service.  
JavaScript Object Notation: JSON  
The left part of the pair, is always double quotes, nested JSON objects, JSON arrays: use `["","","","","","]`  
### REST HTTP CURD
`POST` : create an entity  
`GET` : read a list of entities  
`PUT` : Update an entity  
`DELETE` : delete an entity 
Request message: request line, Header, Body
Response message: response line, Header, Body  
range: 100-199: informational, 200-299: successful, 300-399: Redirection, 400-499: client error, 500-599: server error  
message format: MINE content type, Multipurpose Internet Mail-Extension. Basic syntax: type/sub-type, e.g. text/html, text/plain  
Client tools for send HTTP requests to the REST web service/API. e.g. curl, postman.  
for advanced REST testing: POST, PUT use Postman for better support.  
So what we did is, the REST API is the backend, we are creating endpoint, we can write `@RestController`, `@GetMapping` 
code:
```
@RestController
@RequestMapping("/test")
public class DemoRestController  {
    @GetMapping("/hello")
    public String hello(){
        return "hello world";
    }
}
```
so when we visit `/hello, it will return hello world.  
### Java JSON Data Binding
Data binding is the process of converting JSON data to JAVA POJO.  
Spring use Jackson project behind the scenes, handling data binding, call setter method go from JSON to POJO. Have to have setter method, match the name of JSON.  
Jackson call getter method go from POJO to JSON.  
spring automatically use Jackson, JSON pass to the REST controller is converted to POJO, java object returned from REST controller is converted to JSON.  
### create a new service for student
code:
````agsl
@RestController
@RequestMapping("/api")
public class StudentRestController {
    @GetMapping("/students")
    public List<Student> getAllStudents() {
        List<Student> students = new ArrayList<>();
        students.add(new Student("Smith", "John"));
        students.add(new Student("Mary", "Zuick"));
        students.add(new Student("qxc", "Ted"));
        return students;
    }
````
````agsl
public class Student {
    private String lastName;
    private String firstName;
````
### path variable
retrieve a single student by id. `/api/students/{studentId}`  
we can use `@PostConstruct` to construct date only once when bean created, we can use `@PathVariable` for `/{studentId}`  
````agsl
    @GetMapping("/students/{studentId}")
    public Student getStudent(@PathVariable("studentId") int studentId) {  // by default, variables name should match
        return students.get(studentId);
    }
````  
### exception handling
1. create StudentErrorResponse class
2. create StudentNotFoundException extends RuntimeException
3. make a if statement
4. use `@ExceptionHandler`
code for the index out of range exception:
we need to first throw an exception
````agsl
// check the student id against list size
        if((studentId >= students.size()) || (studentId < 0)) {
            throw new StudentNotFoundException("student id not found" + studentId);
        }
````  
then we can create the error response
````agsl
public ResponseEntity<StudentErrorResponse> handleException(StudentNotFoundException exc) {
        // create a StudentErrorResponse
        StudentErrorResponse errorResponse = new StudentErrorResponse();
        errorResponse.setStatus(HttpStatus.NOT_FOUND.value());
        errorResponse.setMessage(exc.getMessage());
        errorResponse.setTimeStamp(System.currentTimeMillis());
        // return ResponseEntity
        return new ResponseEntity<>(errorResponse,HttpStatus.NOT_FOUND);
    }
````  
### exception handling for general case
````agsl
@ExceptionHandler
    public ResponseEntity<StudentErrorResponse> handleException(Exception exc) {
        // create a StudentErrorResponse
        StudentErrorResponse errorResponse = new StudentErrorResponse();
        errorResponse.setStatus(HttpStatus.BAD_REQUEST.value());
        errorResponse.setMessage(exc.getMessage());
        errorResponse.setTimeStamp(System.currentTimeMillis());
        // return ResponseEntity
        return new ResponseEntity<>(errorResponse,HttpStatus.BAD_REQUEST);
    }
````
### global exception handling
now it is only for 1 REST controller, but we need for all other controllers. So we need to have global.  
**Aspect-oriented programming**  
globalExceptionHandler is best practice for large projects  
code:
```agsl
@ControllerAdvice
public class StudentRestExceptionHandler{
 // the same exception as above
}
```  
### API design process
1. review API requirements
2. Identify main resource / entity
3. use HTTP method to assign action on resource  

   `POST` : /api/employees create an entity  
   `GET` : /api/employees or /api/employees/{employeeId} read a list of entities  
   `PUT` : Update an entity  
   `DELETE` : delete an entity  

do not do:
include the action in the endpoints  
do: use HTTP method to assign action on resource  
convention is to use plural form on API  
### create JPA DAO
standard JPA DAO API  
first create entity package and employee class, then create DAO interface and implement  
code:
```agsl
@Autowired
    public EmployeeDAOImp(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
    public List<Employee> findAll() {

        // create a query
        TypedQuery<Employee> query = entityManager.createQuery("from Employee", Employee.class);

        // execute the query
        List<Employee> results = query.getResultList();

        // return the results
        return results;
    }
```  
code for EmployeeRestController:
```agsl
@RestController
@RequestMapping("/api")
public class EmployeeRestController {
    private EmployeeDAO employeeDao;
    public EmployeeRestController(EmployeeDAO employeeDao) {
        this.employeeDao = employeeDao;
    }
    @GetMapping("/employees")
    public List<Employee> findAll() {
        return employeeDao.findAll();
    }
```
Now we need to have a service layer implementation, which is between REST controller and DAO. It is **service facade** design pattern  
it can integrate multiple data sources, e,g. Skills dao, Payroll dao. it is common to see in large Enterprise applications.  
specialized annotations for services, `@Service` is a mutation of `@Component`  
remember do the injection for every component  
### add, save, delete for DAO
remember to add `@Transactional` on the service layer, no the DAO.  
code for DAO:  
```agsl
@Override
    public Employee save(Employee employee) {
        //save the employee
        Employee dbEmployee= entityManager.merge(employee);  // if id is ==0, then insert the employee, else update

        // return the dbEmployee
        return dbEmployee;
    }
```  
code for service layer.
```agsl
@Transactional
    @Override
    public void deleteById(int id) {
        employeeDAO.deleteById(id);
    }
```  
### in the REST controller
code for GET
```agsl
// add mapping for GET /employees/{employeeId} by Id
    @GetMapping("/employees/{employeeId}")
    public Employee getEmployee(@PathVariable int employeeId) {
        Employee employee = employeeService.findById(employeeId);
        if(employee == null) {
            throw new RuntimeException("employee id not found" + employeeId);
        }
        return employee;
    }
```  
code for POST
````agsl
@PostMapping("/employees")
    public Employee addEmployee(@RequestBody Employee employee) {
        // set the id to 0 in case of they pass an id to Json
        // this is force a save of new item
        employee.setId(0);
        Employee dbemployee = employeeService.save(employee);
        return dbemployee;
    }
````  
code for PUT:
```agsl
@PutMapping("/employees")
    public Employee updateEmployee(@RequestBody Employee employee) {
        Employee dbemployee = employeeService.save(employee);
        return dbemployee;
    }
```  
mistake I made, `@PutMapping("/employees/")`  
code for DELETE  
```agsl
@DeleteMapping("/employees/{employeeId}")
    public String deleteEmployee(@PathVariable int employeeId) {

        Employee dbemployee = employeeService.findById(employeeId);
        // check for exception
        if (dbemployee == null){
            throw new RuntimeException("employee id not found" + employeeId);
        }
        employeeService.deleteById(employeeId);

        return "delete employee # " + employeeId;
    }
```  
### 06-jpaRepository
use CRUD methods for every DAO, no need to implement the class. use the Spring Data JPA, no need to write DAO implement code.  
we can remove the ` @Transactional` since we use the JPA interface.  
interface:
```agsl
public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
    // that is it, no need to write anything
}
```
new feature in Java 8 using the optional:
````agsl
@Override
    public Employee findById(int id) {
        Optional<Employee> result = employeeRepository.findById(id);
        Employee employee = null;
        if (result.isPresent()) {
            employee=result.get();
        }
        else {
            throw new RuntimeException("Couldn't find employee" + id);
        }
        return employee;
    }
````
### 07-JPA for rest endpoint
the same method as above  
only need 3 items for Spring data REST:  
1. the entity: Employee
2. Jpa repository extends jpaRepository
3. Maven POM dependency: 

terminology: HATEOAS: Hypermedia as the Engine of Application State: the meta-data for REST  
We can add dependencies 
```agsl
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-rest</artifactId>
		</dependency>
```
we don't need controller and services package. spring will be provided. we only need the entity  
noticed, for the rest endpoint, when we try PUT we need to use `http://localhost:8080/magic-api/employees/5` we need to specify the id in the URL.  
### pluralized forms for REST
problems: some words can not have the plural forms are complex.  
properties available for application. 
we can use `@RepositoryRestResource(path="members")` to change the path name.
we can also use sorting, sorting by last name. `http://localhost:8080/magic-api/employees?sort=lastName,desc`  
pagination: `spring.data.rest.default-page-size=3`

## spring security
implementing spring Servlet filters, two methods of securing: declarative and programmatic.
### declarative security
use `@Configuration` 
use spring.security.user.name=scott  
we can use `spring.security.user.password=test123` to modify username and password  
### configuration for security 
1. create a new class
```agsl
@Configuration
public class DemoSecurityConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsManager() {
        UserDetails susan = User.builder()
                .username("susan")
                .password("{noop}test123")
                .roles("EMPLOYEE", "MANAGER", "ADMIN")
                .build();
        return new InMemoryUserDetailsManager(john, mary, susan);
    }
}
```
### restricting access to roles
cross site request forgery(CSRF) generally, we don't need to use CSRF for stateless REST api.  
code:
```agsl
@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(configurer ->
                configurer
                        .requestMatchers(HttpMethod.GET, "/api/employees").hasRole("EMPLOYEE")
                        .requestMatchers(HttpMethod.GET, "/api/employees/**").hasRole("EMPLOYEE")
                        .requestMatchers(HttpMethod.POST, "/api/employees").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.PUT, "/api/employees").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.DELETE, "/api/employees/**").hasRole("ADMIN")
        );
        // use HTTP Basic authentication
        http.httpBasic(Customizer.withDefaults());
        // disable CSRF, in general, not required for stateless REST APIs
        http.csrf(csrf -> csrf.disable());
        return http.build();
    }
```
### Database access in plain text
step 1. Create sql query
step 2. add JDBC driver in POM file
step 3. create JDBC properties file  
nice things is SpringBoot will use database data to do security login each new login attempt, no need to restart the application.  
So first set up the password and username, admin role in the database, then add the following:
```agsl
@Bean
    public UserDetailsManager UserDetailsManager(DataSource dataSource) {
        return new JdbcUserDetailsManager(dataSource); //
    }
```
### Database access with encryption
The password in database will never decrypt. Bcrypt is a one way encryption. The process of login is:  
1. encrypt the password that user entre
2. compare with the Bcrypt code with the database ones. 

we don't need to change the java source code, we only need to change the sql script, with `{bcrypt}$2a$10$qeS0HEh7urweMojsnwNAR.vcXJeXR1UcMRZ2WcGQl9YeuspUdgF.q` where the bcrypt code we need to generate.
### security custom tables
you need to provide a query, tell spring find the name in database. Nothing is match with default spring schema. 
code:
```agsl
@Bean
    public UserDetailsManager UserDetailsManager(DataSource dataSource) {
        JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);
        // define the SQL query to get a user by username
        jdbcUserDetailsManager.setUsersByUsernameQuery("select user_id, pw, active from members where user_id=?");
        // define the SQL query to get a role by username
        jdbcUserDetailsManager.setAuthoritiesByUsernameQuery("select user_id, role from roles where user_id=?");
        return jdbcUserDetailsManager; //
    }
```
### Thymeleaf
is a separate  template, but a lot of synergy with spring. spring will automatically go to /resource/templates/xxx.HTML  
some code for thymeleaf:
```agsl
<! DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org/">
<head>
    <title>
        Thymeleaf Demo
    </title>
</head>
<body>
<p th:text="'Time on the server is ' + ${theDate}" />
</body>
</html>
```
the code for the GET mapping:
```agsl
@GetMapping("/hello")
    public String sayHello(Model theModel) {
        theModel.addAttribute("theDate", new java.util.Date());
        return "helloworld";
    }
```
### use CSS for thymeleaf
we have 2 opions: 1. local CSS file as part of the project. 2.referencing remote CSS file  
add the link tag in the header: `<link rel="stylesheet" th:href="@{/css/demo.css}" />` then create a new CSS file under static/css
### thymeleaf project
we can add an index.html file to redirect to the `/employees/list` by default. So we can just go to localhost:8080 and it will redirect. Add:
```agsl
<meta http-equiv="refresh"
      content="0; URL='employees/list'">
```
thymeleaf has special expression for binding Spring MVC from data. Automatically setting and retrieving data from a Java object.  
`Post/Redirect/Get` method  
```agsl
// the default path is to look at the templates folder for the view
		return "employees/list-employees";
```
Add a bottom to add employees: 
```agsl
<a th:herf="@{/employees/showFormForAdd}" class="btn btn-primary btn-sm mb-3"> Add Employee</a>
```
thymeleaf can blind the data by itself. For showing the form, add this to controller:
```agsl
@GetMapping("/showFormForAdd")
    public String showFormForAdd(Model theModel) {
        // create model attribute to bind form data
        Employee theEmployee = new Employee();
        theModel.addAttribute("employee", theEmployee);
        return "employees/employee-form";
    }
```
`*` is meant to select properties on referenced th:object  
When the form is loaded, it will first call the getter method. When we submit the form, it will call setter method. We need to create a new HTML file, core code for the form:
```agsl
<form action="#" th:action="@{/employees/save}"
                        th:object="${employee}" method="post">
            <input type="text" th:field="*{firstName}"
                   class="form-control mb-4 w-25" placeholder="First name">
            <input type="text" th:field="*{lastName}"
                   class="form-control mb-4 w-25" placeholder="Last name">
            <input type="text" th:field="*{email}"
                   class="form-control mb-4 w-25" placeholder="email">
            <button type="submit" class="btn btn-info col-2">
                Save 
            </button>
        </form>
```
for the save mapping, we use a `Post/Redirect/Get` method:
```agsl
@PostMapping("/save")
	public String saveEmployee(@ModelAttribute("employee") Employee theEmployee) {
		// save the employee
		employeeService.save(theEmployee);
		// use a redirect to prevent duplicate submissions
		return "redirect:/employees/list";
	}
```
the spring JPA will parse the method name and create the query for you, you only need to write the function name, findAllBy is part of pattern, and it read order by lastname.  sort by lastName:
```agsl
 public List<Employee> findAllByOrderByLastNameAsc();
 // update the impl
 	@Override
	public List<Employee> findAll() {
		return employeeRepository.findAllByOrderByLastNameAsc();
	}
```
### update employees
first need to add update bottom, this append to the URL ?employeeId=xxx:
```agsl
<a th:href="@{/employees/showFormForUpdate(employeeId=${tempEmployee.id})}"
					class="btn btn-info btn-sm">
						Update
					</a>
```
the code for the controller, it will pre-populate the form:
```agsl
@GetMapping("/showFormForUpdate")
	public String showFormForUpdate(@RequestParam("employeeId")  int theId, Model theModel) {
	// get the employee from the service
		Employee theEmployee = employeeService.findById(theId);
		// set employee as a model attribute to pre-populate the form
		theModel.addAttribute("employee", theEmployee);
		// send over to our form
		return "employees/employee-form";
	}
```
then need to add the following to form.html: ` <!--add hidden form field to handle the update -->
<input type="hidden" th:field="*{id}"/>`
### delete employee
add a delete button:
```agsl
<a th:href="@{/employees/delete(employeeId=${tempEmployee.id})}"
					   class="btn btn-danger btn-sm"
					onclick="if (!(confirm('Are you sure you want to delete this employee?'))) return false">
						Delete
					</a>
```
add the controller code for deleting:
```agsl
@GetMapping("/delete")
    public String delete(@RequestParam("employeeId") int theId) {
		// delete the employee
        employeeService.deleteById(theId);
        // redirect to /employees/list
        return "redirect:/employees/list";
	}
```
### Spring Security with Servlet Filters
spring security will automatically look you configuration and user and password. use the `@Configuration` to config. Spring give you the default login form. You can also custom the login form.
### security demo project
problem: when you change your source code, if the user already logged in, it will not ask the user to login again, this is because login is based on browser session.  
solve the problem: quit the browser and start again, or use a different browser session. or use incognito session.
### security demo code
```agsl
@Configuration
public class DemoSecurityConfig {
    @Bean
    public InMemoryUserDetailsManager UserDetailsManager() {
        UserDetails john = User.builder()
                .username("john")
                .password("{noop}test123")
                .roles("EMPLOYEE")
                .build();
                return new InMemoryUserDetailsManager(john, mary, susan);
```
### custom login form
first we need to modify the security config.  
we don't need to write code for `/authenticateTheUser` because spring will handle this.  
`@` in the path, it means it is a context path, which refer to the root path of your application, like `http://localhost:8080/my-app`. The advantage of using the context root path it is change independent, which mean when you change the app name, the link will still be valid. 
code for configuring file:
```agsl
@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http .authorizeHttpRequests(configurer ->
                configurer
                        .anyRequest().authenticated() )
                .formLogin(form ->
                        form
                                .loginPage("/showMyLoginPage")
                        .loginProcessingUrl("/authenticateTheUser")
                        .permitAll());
        return http.build();
    }
```
the code for the controller: we only need to return html file name. like here is returned ` return "plain-login";`  
the code for the html login page, notice here we need to have `<html lang="en" xmlns:th="http://www.thymeleaf.org">` to add a tag name `th` the code:
```agsl
<body>
<h3>My Custom Login Page</h3>
<form action="#" th:action="@{/authenticateTheUser}" method="POST">
    <p>
        User name: <input type="text" name="username" />
    </p>
    <!-- spring will read from the data and authenticate the user using these default form field name-->
    <p>
        Password: <input type="password" name="password" />
    </p>
    <input type="submit" value="Login" />
</form>
</body>
```
### login in form error message
spring will send a URL with `?error` so we can just write code for that.  
code:
```agsl
<div th:if="${param.error}">
        <i class="failed">
            Sorry! You entered the wrong password or username.
        </i>
```
### logout
the URL `/logout` will handle by spring, you will get this for free. no code needed. By default, must use POST method. GET method is disabled by default.  
code for logout:
```agsl
<!-- Check for logout -->
                                            <div th:if="${param.logout}">
                                                <div class="alert alert-success col-xs-offset-1 col-xs-10">
                                                    You have been logged out.
                                                </div> </div>
```
code on the home screen:
```agsl
<form method="POST" action="#" th:action="@{/logout}">
<input type="submit"  value="Logout" />
```
code add on config file:
```agsl
.logout(logout ->logout.permitAll() 
```
### display for the user role and ID
we only need to add some html on the home screen:
```agsl
<p>
    User: <span sec:authentication="principal.username"></span>
    <br><br>
    Role(s): <span sec:authentication="principal.authorities"></span>
</p>
```
### restrict URL based on the role
first add new html file for manager:
`<a th:href="@{/}"> back to home page</a>`  
we add the link on the home page:
```agsl
<p>
    <a th:href="@{/leaders}">Leadership Meeting</a>
    (only available for managers)
</p>
```
add this to the controller:
```agsl
@GetMapping("/leaders")
    public String showLeaders() {
        return "leaders";
}
```
add this to the security configuration:
```agsl
.requestMatchers("/").hasRole("EMPLOYEE")
                        .requestMatchers("/leaders/**").hasRole("MANAGER")
                        .requestMatchers("/systems/**").hasRole("ADMIN")
```
the code for the system peeps, is similar to above.
### custom assess denied page
add this to the security configuration:
```agsl
.exceptionHandling(configurer -> configurer.accessDeniedPage("/access-denied"));
```
add this to the login controller:
```agsl
 @GetMapping("/access-denied")
    public String showAccessDenied() {
        return "access-denied";
    }
```
create a html page:
### display content based on the role:
the browser sources code will not have the html for this page. We can add role on the home page HTML tags:
```agsl
<div sec:authorize="hasRole('MANAGER')">
    <!-- this will only display for manager-->
    <p>
        <a th:href="@{/leaders}">Leadership Meeting</a>
        (only available for managers)
    </p>
</div>
```
### use database to store password
spring security have default database, the table name. we have to match the name of the database. one is called users, and another is called authorities.  
1. run sql script for sql
2. add Maven dependence
3. update security configuration
4. add code to use database  

add Maven dependency:
```agsl
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>com.mysql</groupId>
			<artifactId>mysql-connector-j</artifactId>
			<scope>runtime</scope>
		</dependency>
```
add this to security configuration, it can tell spring to use JDBC authentication with data source:
```agsl
@Bean
    public UserDetailsManager userDetailsManager(DataSource dataSource){

        return new JdbcUserDetailsManager(dataSource);
    }
```
we can look at the log to check is connected to database. `logging.level.org.springframework.jdbc.core=TRACE`
### spring security with password encryption
we cover this before, only update the sql script.
### custom sql table
we cover this before, same things.
## JPA advanced mapping
### one-to-one mapping
Cascade mapping: apply same operation to related entities. if we save the teachers, then we will save the teacher's details. If we delete a teacher then we will delete the teacher's details.  
depend on the case, we can delete a student and delete the course.  
Eager and lazy fetching, eager will retrieve everything, and lazy fetch only the requested.  
ui mean one direction, bi mean 2 direction.
### entity life cycle
Detach: it is not associated with a Hibernate session.
merge: it will reattach the session  
presist: transition new instances to managed state, next flush or commit will save on the db.  
remove: managed to remove the session, it will remove from db.  
refresh: it will reload the data from database  
cascade: it is the same as the above. Except ones, it has ALL, which is the combination of all.  
by default, there is no opreation are cascaded.
### command line runner
```agsl
@Bean
	public CommandLineRunner commandLineRunner(String[] args) {
		return runner -> {
			System.out.println("Hello World");
		};
```
we can turn the banner off and only show the warning message
```agsl
spring.main.banner-mode=off
logging.level.root=warn
```
now we need to first create the Instructor class
1. create the Instructor class
2. create the InstructorDetail class
3. create the DAO class
### DAO
it will also save for the InstructorDetail class, because the cascaded.  
we can inject the app dao here, it uses beans, so it automatically inject.
```agsl
@Bean
	public CommandLineRunner commandLineRunner(AppDAO appDAO) {
		return runner -> { 
			System.out.println("Hello World");
		};
```
### JPA log
```agsl
logging.level.org.hibernate.SQL=trace // this will see log sql statements
logging.level.org.hibernate.orm.jdbc.bind=trace // this will see values for sql
```
this is the code for commonline runner:
```agsl
private void createInstructor(AppDAO appDAO) {
		 // create Instructor
		Instructor tempInstructor = new Instructor("Susan", "Public", "susan@rbc.com");
		// create instructor detail
        InstructorDetail tempInstructorDetail = new InstructorDetail("http://www.youtube.com", "Video Games");
		// associate the objects
		tempInstructor.setInstructorDetail(tempInstructorDetail);
		// save the instructor
		// this will also save the details object because of CascadeType.ALL
		System.out.println("saving instructor: " + tempInstructor);
		appDAO.save(tempInstructor);
		System.out.println("Done!");
```
don't forget to use `@Repository`
### find instructor by ID
code:
```agsl
@Override
    public Instructor findById(int theId) {
        return entityManager.find(Instructor.class ,theId);
    }
```
we need to add a method to find by the ID:
```agsl
private void findInstructor(AppDAO appDAO) {
		int id=2;
		System.out.println("findInstructor with id " + id);
		Instructor tempInstructor = appDAO.findById(id);
		System.out.println("Instructor: " + tempInstructor);
		System.out.println("InstructorDetail: " + tempInstructor.getInstructorDetail());
	}
```
also for the DAO:
```agsl
@Override
    public Instructor findById(int theId) {
        return entityManager.find(Instructor.class ,theId);
    }
```
add these to the command line runner:
```agsl
private void deleteInstructor(AppDAO appDAO) {
		int id=2;
		System.out.println("delete with id " + id);
		appDAO.deleteById(id);
		System.out.println("Done!");
	}
```
add these to the DAO:
```agsl
@Override
    @Transactional
    public void deleteById(int theId) {
        // retrieve instructor by id
        Instructor tempinstructor = entityManager.find(Instructor.class ,theId);
        // delete the instructor
        if(tempinstructor != null) {
            entityManager.remove(tempinstructor);
        }
    }
```
### bi-direction mapping
`mappedby` will map the instructor detail in the instructor class.  
1. we need to add new field to instructor
2. add getter/setter method
3. add `@one-to-one`
```agsl
@OneToOne(mappedBy = "instructorDetail", cascade= CascadeType.ALL) 
    //mappedBy is used to specify the name of the field in the entity class that is the inverse of this relationship.
    private Instructor instructor;
```
4. add DAO method:
```agsl
@Override
    public InstructorDetail findInstructorDetailById(int theId) {
        return entityManager.find(InstructorDetail.class ,theId);
    }
```
add these methods to command line runner:
```agsl
private void findInstructorDetail(AppDAO appDAO) {
		// get the instructor detail object
		int id = 1;
		System.out.println("InstructorDetail with id " + id);
		InstructorDetail tempInstructorDetail = appDAO.findInstructorDetailById(id);
		System.out.println("InstructorDetail: " + tempInstructorDetail);
		System.out.println("Instructor: " + tempInstructorDetail.getInstructor());
		System.out.println("Done!");
	}
```
### cascade delete instructor detail and instructor
add to command line runner:
```agsl
private void deleteInstructorDetail(AppDAO appDAO) {
		int id=3;
        System.out.println("delete with id " + id);
        appDAO.deleteInstructorDetailById(id);
        System.out.println("Done!");
	}
```
add to DAO:
```agsl
 @Override
    @Transactional
    public void deleteInstructorDetailById(int theId) {
        // retrieve instructor by id
        InstructorDetail tempinstructorDetail = entityManager.find(InstructorDetail.class ,theId);
        // delete the instructor
        if(tempinstructorDetail != null) {
            entityManager.remove(tempinstructorDetail);
        }
```
### we only want to delete details
we need to modify the Instructor DAO:
```agsl
@OneToOne(mappedBy = "instructorDetail", cascade= {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
```
we also need to modify the DAO, remove the Instructor for InstructorDetail:
```agsl
// remove the associated object reference
        // break bidirectional link
        if(tempinstructorDetail != null) {
            tempinstructorDetail.getInstructor().setInstructorDetail(null);
        }
```
## many-to-one relationship
do not apply cascading delete. we also need to set up the bi-direction between course and instructor.  
now we need to create a class for course.  
we also need to an add method in the instructor class:
```agsl
// add convenience methods for bidirectional relationship
    public void add(Course tempCourse){
        if(courses == null){
            courses = new java.util.ArrayList<>();
        }
        courses.add(tempCourse);
        tempCourse.setInstructor(this);
    }
```
we also need to add the one-to-many relationship in the instructor class:
```agsl
 @OneToMany(mappedBy = "instructor", cascade = {CascadeType.DETACH,CascadeType.MERGE,CascadeType.PERSIST,CascadeType.REFRESH})//mappedBy is the name of the field in the Course class
    private List<Course> courses;
```
this is what we did for many-to-one relationships in the course class:
```agsl
@ManyToOne(cascade = {CascadeType.DETACH,CascadeType.MERGE,CascadeType.PERSIST,CascadeType.REFRESH})
    @JoinColumn(name="instructor_id")
    private Instructor instructor;
```
in the database, it set the ID of the course starting from the 10 `AUTO_INCREMENT=10`  
we need to add code on the APP Runner:
```agsl
// save the instructor
		// note: this will ALSO save the course because of CascadeType.ALL
		System.out.println("Saving instructor: " + tempInstructor);
		System.out.println("Saving the course " + tempInstructor.getCourses() );
		appDAO.save(tempInstructor);
```
### fetch types: Eager vs Lazy
eager will retrieve everything, but lazy only retrieve request. we want to search by name.  
**the real world prefer lazy loading to eager loading.**  
real world example: we can search instructor by name, this is lazy loading, then an optional for users to view the details, then we can use eager loading.  
default fetch type:   
* for one-to-one is eager loading
* one-to-many is lazy loading
* many-to-one is eager loading
* many-to-many is lazy  

we can overload the methods. The key of the lazy loading, it will request the Hibernate session is opened, if the session is closed, it will throw an exception  
this will only load the instructor:
```agsl
private void findInstructorWithCourse(AppDAO appDAO) {
		Instructor instructor = appDAO.findById(1);
		System.out.println("finding the instructor");
        System.out.println(instructor);
	}
```
this will throw an exception:
```agsl
private void findInstructorWithCourse(AppDAO appDAO) {
		Instructor instructor = appDAO.findById(1);
		System.out.println("finding the instructor");
        System.out.println(instructor);
		System.out.println("the course" + instructor.getCourses()); // this line error
	}
```
we can use change the fetch type: `fetch = FetchType.EAGER`  
### lazy: find course
we need to have this method in the DAO:
```agsl
@Override
    public List<Course> findCoursesByInstructorId(int theId) {
        TypedQuery<Course> query = entityManager.createQuery("from Course where instructor.id = :data" ,Course.class);
        query.setParameter("data" ,theId);
        List<Course> courses = query.getResultList();
        return courses;
    }
```
we can have this method in the runner:
```agsl
private void findCourseForInstructor(AppDAO appDAO) {
		Instructor instructor = appDAO.findById(1);
		System.out.println("finding the instructor");
		System.out.println(instructor);
		// find courses for the instructor
		System.out.println("finding the course with the id");
		List<Course> courses = appDAO.findCoursesByInstructorId(1);
		System.out.println(courses);
        System.out.println("done!");
```
### use lazy but also only need 1 query
we can use JOIN FETCH, it is similar to eager. 
add a query on the DAO:
```agsl
@Override
    public Instructor findInstructorByIdJoinFetch(int theId) {
        // create a query
        TypedQuery<Instructor> query = entityManager.createQuery("select i from Instructor i " + "JOIN FETCH i.courses " + "where i.id = :data" ,Instructor.class);
        query.setParameter("data" ,theId);
        Instructor instructor = query.getSingleResult();
        return instructor;
    }
```
we can also add theses methods on the runner:
```agsl
private void findInstructorWithCourseJoinFetch(AppDAO appDAO) {
		Instructor instructor = appDAO.findInstructorByIdJoinFetch(1);
        System.out.println("finding the instructor");
        System.out.println(instructor);
        // find courses for the instructor
        System.out.println("finding the course with the id" + instructor.getCourses());
        System.out.println("done!");
	}
```
we can also try to minimize the query into one query:
```agsl
TypedQuery<Instructor> query = entityManager.createQuery("select i from Instructor i " + "JOIN FETCH i.courses "+ "JOIN FETCH i.instructorDetail "+ "where i.id = :data" ,Instructor.class);
```
### update instructor
we need to first find the instructor, then call the setter method, then update the DAO.  
we need to first write a save method in DAO:
```agsl
@Override
    @Transactional
    public void updateInstructor(Instructor instructor) {
        entityManager.merge(instructor);
    }
```
then we can have this method on the runner.
```agsl
private void UpdateInstructor(AppDAO appDAO) {
		Instructor instructor = appDAO.findById(1);
        System.out.println("finding the instructor");
        System.out.println(instructor);
        // update the instructor
        instructor.setLastName("tester");
        // save the instructor
        System.out.println("Saving instructor: " + instructor);
        appDAO.updateInstructor(instructor);
        System.out.println("done!");
	}
```
### update course
the same thing as above
### delete the instructor
**we need to first break associations** of all instructor's course, then delete.  
code:
```agsl
// get the courses for the instructor
        List<Course> courses = tempinstructor.getCourses();
        // delete the associated object reference
        // break bidirectional link
        for (Course course : courses) {
            course.setInstructor(null);
        }
```
### delete the course
we can add these code to the DAO:
```agsl
 @Override
    @Transactional
    public void deleteCourseById(int theId) {
        Course theCourse = entityManager.find(Course.class, theId);
        entityManager.remove(theCourse);
    }
```
### one-to-many relationship
we need to create a new entity class, and we need to add new methods on the course class:
```agsl
@OneToMany(fetch = FetchType.LAZY, cascade= CascadeType.ALL)
    @JoinColumn(name="course_id")
    private List<Review> reviews;
    // add a convenience method
    public void addReview(Review theReview){
        if(reviews==null){
            reviews=new ArrayList<Review>();
        }else {
            reviews.add(theReview);
        }
    }
```
### Create course and review and save
we need this method on the runner:
```agsl
	private void createCourseAndReview(AppDAO appDAO) {
		Course course = new Course("x++");
		course.addReview(new Review("love this one"));
		course.addReview(new Review("love this x++"));
		course.addReview(new Review("love this java"));
		course.addReview(new Review("love this c"));
		appDAO.save(course);
		System.out.println("saving the course");
		System.out.println(course);
		System.out.println("course reviews: " + course.getReviews());
		System.out.println("done!");
	}
```
we will a save method to save the course:
```agsl
@Override
    @Transactional
    public void save(Course course) {
        entityManager.persist(course); // it will also save the comments
    }
```
### retrieve course and review
we need to retrieve by query:
```agsl
        // create a query
        TypedQuery<Course> query = entityManager.createQuery("select c from Course c " + "JOIN FETCH c.reviews " + "where c.id = :data" ,Course.class);
        query.setParameter("data" ,theId);
        // execute query
        Course course = query.getSingleResult();
        return course;
    }
```
### delete the course
we simply need to have a method to delete the course in the Runner:
```agsl
private void deleteCourseAndReview(AppDAO appDAO) {
		int theId=10;
		System.out.println("Deleting the course and review with id: " + theId);
		appDAO.deleteCourseById(theId); // this will course and reviews
		System.out.println("done!");
	}
```
## many-to-many
we need a join table to maintainer the relationship between students and course. `@JoinTable` and also `Inverse`  
this first part is to create students entity. Then we need to update the course class:
```agsl
 @ManyToMany(fetch = FetchType.LAZY,cascade = {CascadeType.DETACH,CascadeType.MERGE,CascadeType.PERSIST,CascadeType.REFRESH})
    @JoinTable(name="course_student",
    joinColumns = @JoinColumn(name="course_id"),
    inverseJoinColumns = @JoinColumn(name="student_id"))
    private List<Student> students;
```
we need to update the students class:
```agsl
@ManyToMany(fetch = FetchType.LAZY,cascade = {CascadeType.DETACH,CascadeType.MERGE,CascadeType.PERSIST,CascadeType.REFRESH})
    @JoinTable(name="course_student",
            joinColumns = @JoinColumn(name="student_id"),
            inverseJoinColumns = @JoinColumn(name="course_id"))
    private List<Course> courses;
```
we need to add a new member to the runner class:
```agsl
Course course = new Course("javascript");
		Student student1 = new Student("jjjj","sttt","jjlin@rbc.com");
		Student student2 = new Student("qqq","hanland","whanland@rbc.com");
		course.addStudent(student1);
		course.addStudent(student2);
        System.out.println("saving the course");
		System.out.println("the students: " + course.getStudents());
        System.out.println(course);
		appDAO.save(course);
        System.out.println("done!");
```
### find course and students, first find course
we need to have a query by join fetch to find:
```agsl
TypedQuery<Course> query = entityManager.createQuery("select c from Course c " + "JOIN FETCH c.students " + "where c.id = :data" ,Course.class);
        query.setParameter("data" ,theId);
        // execute query
        Course course = query.getSingleResult();
        return course;
```
### first find student, then course
just need to change the query by join fetch to find:
```agsl
TypedQuery<Student> query = entityManager.createQuery("select s from Student s " + "JOIN FETCH s.courses " + "where s.id = :data" ,Student.class);
```
### update the student
```agsl
@Override
    @Transactional
    public void update(Student student) {
        entityManager.merge(student);
    }
```
### update the students
we can have a update method:
```agsl
 @Override
    @Transactional
    public void update(Student student) {
        entityManager.merge(student);
    }
```
then we can have a new method on the runner:
```agsl
int theId = 2;
		Student student = appDAO.findStudentAndCoursesById(theId);
		Course course1 = new Course("spring");
		Course course2 = new Course("spring-boot");
		student.addCourse(course1);
		student.addCourse(course2);
		System.out.println("updating student" + student);
		System.out.println("associated course:" + student.getCourses());
		appDAO.update(student);
		System.out.println("done!");
```
notice that although the print statement will show `associated course:[Course{id=12, title='java'}, Course{id=0, title='spring'}, Course{id=0, title='spring-boot'}]`  
but the database will generate the course id in the database.
### only delete the course and student relationships
we can just call the delete method, notice it will also delete the course and student relationships: `delete from course_student where course_id=?`
### delete only student
we add a new delete method:
```agsl
@Override
    @Transactional
    public void deleteStudentById(int theId) {
        // retrieve student by id
        Student tempStudent = entityManager.find(Student.class, theId);
        // delete the student
        entityManager.remove(tempStudent);
    }
```
then we can just delete it.
## Aspect-Oriented programming
problem: code tangling and scattering  
solved: aspect means cross-cutting concerns, aspect can be reused at multiple locations, it is a class.  
AOP solution: use proxy design pattern.  
### terminology
aspect: module of code for cross-cutting concerns, advice: what action is taken and when it should be appiled.  
join point: when to apply code during program execution, pointcut: a predicate expression for where advice should be applied. 
### advice types
before advice: run before the method, after advice: run after the method, after returning advice: run after the method(success execution), after throwing advice: run after method(if exception thrown), around advice: run before and after method.  
### weaving 
### spring AOP
spring use this. use run-time weaving aspects, slower. simpler than AspectJ
### AspectJ
full aop support.
### before advice
use case: logging and security, api management. We also have after return.
### AOP starter
first we need to add Maven dependencies for AOP.  
we also need to aspect class:
```agsl
@Aspect
@Component
public class MyDemoLoggingAspect {
    @Before("execution(public void add())")
    public void beforeAddAccountAdvice() {
        System.out.println("\n=====>>> Executing @Before advice on method");
```
notice that, we don't add any code in the main app, the aspect will run behind the scenes. So right now it actually run before the main app:
```agsl
=====>>> Executing @Before advice on method
class lzc.com.example.AOPdemo.dao.AccountDAOlmpl: DOING MY DB WORK: ADDING AN ACCOUNT
```
## pointcut expression
pointcut expression: a predicate expression for where advice is applied
### match on method name
using the wildcard `*`  
`@Before("execution(public void add*())")`  
it will match any method name start with add.  
`   @Before("execution(public void addAccount())")` this will match addAccount method in any account.  
`    @Before("execution(public void lzc.com.example.AOPdemo.dao.AccountDAO.addAccount())")` this will only match the AccountDAO.addAccount().  
` @Before("execution(public void add*())")` this will match any method begin with *  
`@Before("execution(* add*())")` this will match any method with any return type
### parameter pattern wildcard
() match a method with no arguments
(*) match a method with one argument of any type
(..) match a method with 0 or more arguments of any type  
` @Before("execution(* add*(lzc.com.example.AOPdemo.Account))")` we can specify the path of the parameter in the add method.  
we also need to create a new object in the main app:
```agsl
private void demoTheBeforeAdvice(AccountDAO accountDAO, MembershipDAO membershipDAO) {
		Account myAccount = new Account();
		accountDAO.addAccount(myAccount);
		membershipDAO.addSillyMember();
	}
```
this `..` will match any number of parameters after Account:
`   @Before("execution(* add*(lzc.com.example.AOPdemo.Account, ..))")`  
this will match any number of parameters:
`@Before("execution(* add*(..))")`  
the grand finale:
` @Before("execution(* lzc.com.example.AOPdemo.dao.*.*(..))")`  
**the first * is return type, then it is the package name, then it any class, then it any method with the 0 or more arguments of any type**  
### pointcut declarations
problem: we want to reuse the same pointcut.  
we can have `@pointcut` to create a pointcut:
```agsl
@Pointcut("execution(* lzc.com.example.AOPdemo.dao.*.*(..))")
    private void forDaoPackage() {}
```
the name of the pointcut is forDaoPackage, it is just like the variable name. for example, we can create reused the pointcut:
```agsl
@Before("forDaoPackage()")
    public void performApiAnalytics() {
    System.out.println("\n=====>>> Performing API analytics");
```
### combine pointcut
problem: apply multiple pointcut to single advice, execute an advice only if certain conditions meet.  
we can use logical operators to combine pointcut:
```agsl
 @Pointcut("forDaoPackage() && !(getter() || setter())")
    private void forDaoPackageNoGetterSetter() {}
```
then we can set the `@Before` only execute for not getter and setter:
```agsl
 @Before("forDaoPackageNoGetterSetter()")
    public void beforeAddAccountAdvice() {
        System.out.println("\n=====>>> Executing @Before advice on method");
        
        @Before("forDaoPackageNoGetterSetter()")
    public void performApiAnalytics() {
    System.out.println("\n=====>>> Performing API analytics");
```
so when you run getter and setter it will not run the pointcut.
### control aspects order
the order number goes from low to high, for example, order(1),order(8),order(12). also allow negative number: order(-22), order(-4), order(3).  
If there are collisions, two order with the same number, it is undefined.  
We can also create a new class for all pointcut expressions, adding a `@Aspect` is optional if there is only pointcut. we need to give the fully qualified name of the pointcut: <package> + <class>
```agsl
public class AopExpression {
    @Pointcut("execution(* lzc.com.example.AOPdemo.dao.*.*(..))")
    public void forDaoPackage() {}
    // create pointcut for getter methods
    @Pointcut("execution(* lzc.com.example.AOPdemo.dao.*.get*(..))")
    public void getter() {}
    // create pointcut for setter methods
    @Pointcut("execution(* lzc.com.example.AOPdemo.dao.*.set*(..))")
    public void setter() {}
    // create pointcut: include package ... exclude getter/setter
    @Pointcut("forDaoPackage() && !(getter() || setter())")
    public void forDaoPackageNoGetterSetter() {}
}
```
then we need to update the name for the `@Before`, `@Before("lzc.com.example.AOPdemo.aspect.AopExpression.forDaoPackageNoGetterSetter()")` the abs path.  
now we can just add `@Order(1)`  
### JoinPoints
we also want to access the parameters when we are in aspect.  
The following can display the signature of the method:
```agsl
 public void beforeAddAccountAdvice(JoinPoint joinPoint)
// desplayed method signature
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        System.out.println("Method: " + methodSignature);
```
code for the method parameters:
```agsl
Object[] args = joinPoint.getArgs();
        for (Object arg : args) {
            System.out.println(arg);
            if (arg instanceof Account) {
                Account account = (Account) arg;
                System.out.println("account name: " + account.getName());
                System.out.println("account level: " + account.getLevel());
            }
            if (arg instanceof String) {
                System.out.println("account name: " + arg);
                System.out.println("account level: " + arg);
            }
```
noticed here we can just  System.out.println(arg) then it will print the hashcode of the arg.
## after returning
this is what we need for after returning:
```agsl
@AfterReturning(
            pointcut = "execution(* lzc.com.example.AOPdemo.dao.AccountDAO.findAccounts(..))",
            returning = "result"
    )
    public void afterReturning(JoinPoint joinPoint, List<Account> result) {
        System.out.println("\n=====>>> Executing @AfterReturning on method");
       // print out the method we are advising on
        String method = joinPoint.getSignature().toShortString();
        System.out.println("Method: " + method);
        // print out the result of the method call
        System.out.println("Result: " + result);
    }
```
this is for our main app:
```agsl
private void demoTheAfterReturningAdvice(AccountDAO accountDAO) {
		// call the method to find the accounts
		List<Account> accounts = accountDAO.findAccounts();
		// display the account
		System.out.println("\n Main Program: AoPdemoApplication  "+ accounts);
	}
```
we can also do some post-processing data:  
1. post process the date before returning to the caller.
2. format the data or enrich the data.

this will change the result before returning to the caller:
```agsl
private void convertAccountNamesToUpperCase(List<Account> result) {
        // loop through accounts
        for (Account account : result) {
            // get uppercase version of name
            String theUpperName = account.getName().toUpperCase();
            // update the name on the account
            account.setName(theUpperName);
        }
    }
```
then we can call this function again, the result will be changed:
```agsl
  convertAccountNamesToUpperCase(result);
        System.out.println("Result: " + result);
```
## after throwing
you can notify the DevOps teams via email or SMS.
this is the code for the after throwing:
```agsl
private void demoTheAfterThrowingAdvice(AccountDAO accountDAO) {
		// call the method to find the accounts
		List<Account> accounts = null;
		try {
			// add a boolean flag to simulate an exception
            boolean tripWire = true;
            accounts = accountDAO.findAccounts(tripWire);
        }
```
this is the code for the after throwing:
```agsl
@AfterThrowing(
            pointcut = "execution(* lzc.com.example.AOPdemo.dao.AccountDAO.findAccounts(..))",
            throwing = "exc"
    )
    public void afterThrowing(JoinPoint joinPoint, Throwable exc) {
        // print out which method we are advising on
        String method = joinPoint.getSignature().toShortString();
        System.out.println("\n=====>>> Executing @AfterThrowing on method: " + method);
        // log the exception
        System.out.println("\n=====>>> The exception is: " + exc);
    }
```
## after advice no matter successes or failed
use case for this: log the exception or perform auditing, code to run regardless of the method outcome.  
```agsl
@After("execution(* lzc.com.example.AOPdemo.dao.AccountDAO.findAccounts(..))")
    public void afterAdvice(JoinPoint joinPoint){
        System.out.println("\n=====>>> Executing @After advice on method");
        // print out which method we are advising on
        String method = joinPoint.getSignature().toShortString();
        System.out.println("\n=====>>> Executing @After (finally) advice on method: " + method);
    }
```
this after will run no matter there is an exception or not. 
## around advice--run before and after program
use case: profiling code, instrumentation, pre-processing and pro-processing.  
`ProceedingJoinPoint point` this is the handle to the target method. we need to have this to compute the runtime of our program.  
`duration / 1000.0` converts milliseconds to seconds  
we create a new class called `TrafficFortuneService`  
this is the code to compute the runtime:
```agsl
@Around("execution(* lzc.com.example.AOPdemo.service.*.getFortune(..))")
    public Object aroundFortune(ProceedingJoinPoint point) throws Throwable {
        // print out method we are advising on
        String method = point.getSignature().toShortString();
        System.out.println("\n=====>>> Executing @Around on method: " + method);
        // get begin timestamp
        long begin = System.currentTimeMillis();
        // now, let's execute the method
        Object result = point.proceed();
        // get end timestamp
        long end = System.currentTimeMillis();
        // compute duration and display it
        long duration = end - begin;
        System.out.println("\n=====> Duration: " + duration / 1000.0 + " seconds");
        return result;
    }
```
this is the ouput for this program:
```agsl
=====> Duration: 5.005 seconds
 Main Program: Fortune is Expect heavy traffic this morning
 Main Program: Finished
```
## around advice--handle exception
the great thing is the around advice will handle the exception before the main app. the main app will never know the exception.  
this is for the exception:
```agsl
@Override
    public String getFortune(boolean tripWire) {
        if(tripWire){
            throw new RuntimeException("Major accident! Highway is closed!");
        }
        return getFortune();
    }
```
the exception will never be thrown to the main app, because exceptions will be handled in the around advice.  
we add this code to aspect:
```agsl
 try {
            result = point.proceed();
        }
        catch (Exception e) {
            // log the exception
            System.out.println(e.getMessage());
            result="Major accident! But no worries, your private AOP helicopter is on the way!";
        }
```
this is the output:
```agsl
====>>> Executing @Around on method: TrafficFortuneServiceImpl.getFortune(..)
Major accident! Highway is closed!
=====> Duration: 0.001 seconds
 Main Program: Fortune is Major accident! But no worries, your private AOP helicopter is on the way!
```
now we can also rethrow the exception.  
we just need to change one line:
```agsl
try {
            result = point.proceed();
        }
        catch (Exception e) {
            // log the exception
            System.out.println(e.getMessage());
//            result="Major accident! But no worries, your private AOP helicopter is on the way!";
            // rethrow exception
            throw e;
        }
```



