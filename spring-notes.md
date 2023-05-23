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










