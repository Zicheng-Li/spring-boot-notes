# Notes:
[links to demo](/Users/lizicheng/Desktop/java/spring/spring-boot-3-spring-6-hibernate-for-beginners-main/01-spring-boot-overview/03-actuator-demo)

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
# Notes:
[links to demo] ()

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

[//]: # (![ioc exa]&#40;/Users/lizicheng/Desktop/java/spring/notes/ioc-de.png&#41;  )

[//]: # (![ioc exa]&#40;/Users/lizicheng/Desktop/java/spring/notes/ioc-ex.png&#41;   )

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
### REST HTTP
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














