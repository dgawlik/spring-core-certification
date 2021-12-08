# VMWare Spring Professional

### Section 1: Container, Dependency and IOC

#### 1.1 What is dependency injection and what are the advantages of using it?  

**Dependency injection** is one form of broader programming
principle **Inversion of Control**. In IoC custom portions
of computer program receive flow of control from a generic
framework. In DI specifically object receives it's
dependencies from the container. In other word container
injects those dependencies into the object. The code is 
cleaner beacause object is better decoupled from it's 
dependencies, for example it doesn't need to know their
location.

#### 1.2 What is an interface and what are the advantages of making use of them in Java? 

Interface is a contract between implementing class
and outside world. Programming to the interface means hiding
implementation behind interface to abstract away the
implementation details. In Java interface is collection
of methods (behaviour) without a state. Interface facilitates
better decoupling of objects and encapsulation.

#### 1.3 What is an ApplicationContext? 

`ApplicationContext` wraps `BeanFactory` 
which is IoC container in Spring. It further provides
AOP features, message and resource handling, internationalization.
It's subtypes specialize in application specific
uses cases.

#### 1.4 How are you going to create a new instance of an ApplicationContext? 

For xml-based config use `ClassPathXmlApplicationContext` 
constructor which takes in names of config files. For
java-based config use `AnnotationConfigApplicationContext`
constructor with `@Configuration` classes as arguments.

#### 1.5 Can you describe the lifecycle of a Spring Bean in an ApplicationContext? 

All the lifecycle of the bean happens when containers
`refresh()` method is called.

1. Before instantiation of the bean, available `BeanFactoryPostProcessors`
are run. They are able to change beans definition before
the actual creation. `ConfigurableApplicationContext.getBeanFactory().addBeanPostProcessor`
is the way to register one.
2. Beans implementing `*Aware` interfaces. 
3. Any custom `BeanPostProcessor`'s `postProcessBeforeInitialization()` is called. Note that
the object already has dependencies injected.
4. `@PostConstruct` annotation 
5. Init methods are called: `InitializingBean.afterPropertiesSet` and xml's init-method.
6. Any custom `BeanPostProcessor`'s `postProcessAfterInitialization()`.
7. `@PreDestroy` annotation.
8. `DisposableBean.destroy()` method.
9. xml's destroy-method

#### 1.6 How are you going to create an ApplicationContext in an integration test?

While using `spring-test` package it is possible to create
context by extending junit test by annotating
test with `@SpringBootTest`. This annotation is meta-annotated
with junit 5 `@ExtendWith(SpringExtension.class)`. By registering
junit hooks Spring allows for automatic context creation,
it's caching, transactional support and properties resolution.

In such case annotation `@ActiveProfiles`
allows to configure beans for integration test setup.

It's also possible to create context by hand and then
`ctx.getEnvironment().setActiveProfiles("integration")`.

#### 1.7 What is the preferred way to close an application context? Does Spring Boot do this for you? 

`ConfigurableApplicationContext` provides 2 possibilities for non-web applications.
First you can call `.close()` by yourself. Second to intercept
application's termination signal you have to add JVM's shutdown
hook `ctx.registerShutdownHook` to close context gracefully.

#### 1.8 Are beans lazily or eagerly instantiated by default? How do you alter this behavior?

XML:
* `<beans default-lazy-init="true" ...`
* `<bean lazy-init="true" ...`

JavaConfig:
* `@Lazy` annotation on bean definition

By default singletons are created eagerly.

#### 1.9 What is a property source? How would you use @PropertySource? 

PropertySource is abstraction over key-value pairs collection.
Inside `Environment` property sources form hierarchy with
top sources overwriting values from bottom ones. 

`@PropertySource` annotation is convenient way to add
property sources to your context's environment.

#### 1.10 What is a BeanFactoryPostProcessor and what is it used for? When is it invoked? 

It is an interface that allows for registering for hook
in container lifecycle. Implementing class is called
after all bean definitions are loaded but prior to 
actual bean instantiation. It allows to modify bean 
definition which impacts how beans are created.

#### 1.11 What is a BeanPostProcessor and how is it different to a BeanFactoryPostProcessor? What do they do? When are they called? 

BeanPostProcessor operates on single bean after it is created
but before other hooks. It allows to change bean properties
or wrap it in proxy object.

#### 1.12 What does component-scanning do? 

Component scanning starts at base packages and proceeds
recursively down to sub-packages to detect all classes
marked as spring bean (`@Component` annotation and it's 
descendants). Components can further add to the pool of
bean definitions by having `@Bean` methods (shallow mode).
Scanning can be narrowed down by specifying custom filters.

#### 1.13 What is the behavior of the annotation @Autowired with regards to field injection, constructor injection and method injection? 

`@Autowired` causes dependencies to which types are
assignable to field be automatically injected.
For it to happen exactly one dependency must be
assignable otherwise exception is thrown. Having
multiple classes implementing interface we can
define field as list and all dependencies can be
injected. Annotation `@Order` determines their
ordering. `@Autowired` can be set on constructor
setter or field. If there is only one constructor
no annotation is needed. There is also possiblity
to skip missing dependencies with `@Autowired(required=false...`
and fields are set to null. We can also set one
bean as `@Primary` in case of many beans of same type.

#### 1.14 How does the @Qualifier annotation complement the use of @Autowired? 

`@Autowired` drills down candidates to those of given type,
`@Qualifier` further picks this one of specified name. For only matching by
name `@Resource` should be used. Note that in case
of conflict Spring falls back to breaking ties in 
autowiring by matching bean and field name by
default. 

#### 1.15 What is a proxy object and what are the two different types of proxies Spring can create?

Proxy is design pattern in which original object
is wrapped in another object extending it's original
behaviour in some way also implementing same interface
as the inside object.

Spring by default uses JDK's dynamic proxies for
proxying however the class must implement interface.
For objects not implementing interfaces Spring falls
back to CGlib. For cglib final methods cannot be
advised as they can't be overriden during subclassing.

#### 1.16 What does the @Bean annotation do?

`@Bean` is used to mark methods producing beans.
Mostly it is used inside `@Configuration` it's 
invocation is intercepted by cglib proxy and
scoping applied. It can also be placed in any
`@Component` but then it is processed in lite-mode.
Lite-mode doesn't wrap factory method invocation
in proxy so inter-bean dependencies are just
plain Java method calls.

#### 1.17 What is the default bean id if you only use @Bean? How can you override this?

Default bean id follows method name. If names
are specified in the `@Bean` annotation id follows
all names but not method name.

#### 1.18 Why are you not allowed to annotate a final class with @Configuration?

`@Configuration` class is enhanced with CGlib to
intercept `@Bean` methods. Bean interdependencies
are traced and go through container to enforce
scoping and lifecycle.

#### 1.19 How do you configure profiles? What are possible use cases where they might be useful?

`@Profile` annotation specifies that annotated
bean is eligible for registration given one of
mentioned profiles is active. It is available as
type level annotation with `@Component`, method
level with `@Bean` and as meta annotation.

Activating profile is done either via 
`Environment.setActiveProfiles` or via property
`spring.profiles.active`.

#### 1.20 Can you use @Bean together with @Profile? 

Yes, see 1.19.

#### 1.21 Can you use @Component together with @Profile? 

Yes, see 1.19.

#### 1.22 How many profiles can you have? 

There should be as many profiles as many environments
in which application is run.

#### 1.23 How do you inject scalar/literal values into Spring beans?

Spring treats properties as dependencies that
can be injected. `@Value` annotation allows that.
Before injecting the value it checks scalar type
and runs matching default registered converter.

Also subtitution in property sources and @Value itself
can be done with spEL `${}`. In this case new
property sources are matched against currently
known property sources.

#### 1.24 What is Spring Expression Language (SpEL for short)? 

SpEL is expression language allowing querying and
manipulating object graph at runtime.

#### 1.25 What is the Environment abstraction in Spring?

Environment encapsulates two concepts: profiles
and properties. For profiles it allows querying
which profiles are currently active. For properties
it provides getters for property names iterating
over hierarchy of property sources.

#### 1.26 Where can properties in the environment come from – there are many sources for properties – check the documentation if not sure. Spring Boot adds even more.

Higher in list overriding lower.

Spring Framework:

1. ServletConfig parameters
2. ServletContext parameters
3. JNDI environment variables
4. JVM system properties (-D commandline)
5. JVM system environment (operating system)

Spring Boot:

1. Devtools global properties on home directory
2. `@TestPropertySource` annotations on tests
3. properties attribute on `@SpringBootTest`s
4. commandline arguments
5. properties from `SPRING_APPLICATION_JSON`
6. Spring Framework property hierarchy
7. `RandomValuePropertySource` that has properties
only in `random.*`
8. profile-specific application properties outside
jar
9. profile-specific application properties inside
jar
10. application properties outside jar
11. application properties inside jar
12. `@PropertySource` annotation on `@Configuration`
13. default properties

#### 1.27 What can you reference using SpEL? 

You can reference variables that have been set on
ExpressionParser and if the bean resolver has
been set then also any bean resolver provides.

#### 1.28 What is the difference between $ and # in @Value expressions? 

`#` always represents root expression parser's context
`$` is used for substitution of properties.

### Section 2: Aspect Oriented Programming (AOP)

#### 2.1 What is the concept of AOP? Which problem does it solve? What is a cross cutting concern? 

AOP is programming paradigm that aims to increase
modularization by allowing separation of cross-cutting
concerns. It does so by adding additional behaviour
(advice) without modyfing code itself instead
separately specyfing which code is modified via
pointcut specification. 

Cross cutting concern is behaviour that is spread
across several components. For example it would
be requirement to log result of all methods whose
name starts with "set". 

#### 2.2 What is a pointcut, a join point, an advice, an aspect, weaving? 

Aspect is a modularization of  cross cutting concern (for ex. transactionality)
that is represented in Spring AOP as regular class
annotated with `@Aspect`.

Join point is a point during execution of a program
such as invocation of method or handling of exceptions.
In Spring join point is always invocation of method.

Pointcut is predicate that matches join points.

Advice is action taken by aspect ay particular
join point. There are several types like around,
before and after. Spring maintains chain of 
interceptors around join points. 

Weaving is linking aspect with other objects to 
create advised object. This can be done during
compile time (AspectJ), load time or runtime (Spring AOP).

#### 2.3 How does Spring solve (implement) a cross cutting concern?

On startup context scans packages for components that are
aspects. If found it examines pointcuts and proxies
all objects with matching methods so that interceptors
can be called.

In Spring AOP objects self invocation bypasess
proxying mechanism so interceptors are not applied.
This is not the case for AspectJ which uses compile-time
weaving.

Proxying is done by JDK's dynamic proxies falling back
to CGlib for classes.

#### 2.4 Which are the limitations of the two proxy-types?

JDK's dynamic proxies can only wrap classes that 
implement interfaces.

CGLib can't wrap final classes.

#### 2.5 How many advice types does Spring support? Can you name each one?

* @Before
* @AfterReturning
* @AfterThrowing
* @After (finally)
* @Around - first argument needs to be ProceedingJoinPoint
 
#### 2.6 If shown pointcut expressions, would you understand them?

* execution(<retType> <fqnClassName>.<methodName>(<argTypes>)) limit's 
advice to execution of Spring's bean method. 

Examples:

execution(* com.foo..*(..)) - execution of any method in com.foo package

execution(* Class1.set(..)) - execution of Class1's set method with any arguments

execution(* *(Integer, String, ..)) - execution of any method that it's first parameters are Integer and String

* within(<type>) limit scope to methods in given type
Example: execution(* valueOf(..)) && within(Integer)

* this(<type>) limit scope to proxy's type
* target(<type>) limit scope to wrapped bean type
* args(<type>, <type>, ...) limit scope to methods with given types
* @target, @args, @within corresponding to having annotation
* @annotation wrapped method has annotation

#### 2.7 What is the JoinPoint argument used for?

JoinPoint can be injected to @Before @After @AfterThrowing and @AfterReturing.
It allows to get reflective access to static part which is place where
interception is occuring enriched with chain of interceptors.

We can for example get method signature or it's arguments.

#### 2.8 What is a ProceedingJoinPoint? Which advice type is it used with?

ProceedingJoinPoint extends JoinPoint with method proceed, it is used in
@Around advice. It allows to call actual method and get result returned.


### Section 3:  Data Management: JDBC, Transactions

#### 3.1 What is the difference between checked and unchecked exceptions?

Checked exceptions are those whose caching is enforced
in compile time, io exceptions fall into this category.
Runtime exceptions are caught in runtime. Subclasses of
Error and RuntimeExceptions are unchecked exceptions and
everything else is checked exception. Having checked exception
one has either catch it or name it in method signature with
throws clause.

#### 3.2 How do you configure a DataSource in Spring?

To connect to physical data source you need third-party
driver implementing javax.sql.DataSource. By instantiating
it you are able to get connections to physical datasource.
You can also get DataSource by injecting JavaEE container's
datasource. 

JDBC standard specifies how to create DataSource. Basically
you instantiate drivers DataSource and set URL, USERNAME and PASSWORD
and DRIVER_CLASS if you are using connection pool.

Then you register this DataSource as @Bean.

#### 3.3 What is the Template design pattern and what is the JDBC template?

Template method is behavioral pattern that specifies
outline method as composition of smaller methods
which can be overriden by client subclasses modyfying
their behaviour. It allows to reuse some parts of algorithm
while change/extend others.

This very design underlies Spring's `JdbcTemplate`
`NamedParameterJdbcTemplate` which enclose core JDBC
workflow code leaving application code to provide SQL and
extract results. 

It's core method execute gets connection, applies settings
gets statement, executes statement and closes connection.
doInStatement is extended by inheritance in several places
and adds functionality like supplying arguments, parsing results
from resultset etc.

#### 3.4 What is a callback? What are the JdbcTemplate callback interfaces that can be used with queries? What is each used for? 

Callback is element of IoC where library
calls client provided code. Another name
is hook. It is a method of extending existing
tool, reacting to lifecycle or async programming.

* `PreparedStatementCreator` - given Connection creates `PreparedStatement`
* `PreparedStatementSetter` - given `PreparedStatement` sets some of it's properties
* `CallableStatementCreator` - given `Connection` returns `CallableStatement`
* `PreparedStatementCallback` - allows to execute any number of operations on single `PreparedStatement`
* `CallableStatementCallback` - same for `CallableStatement`
* `ResultSetExtractor` - does extraction of data from `ResultSet` for queries methods
* `RowCallbackHandler` - processing of `ResultSet` on per row basis
* `RowMapper` - transforms row to object
* `SQLExceptionTranslator` - does the translation of `SQLException` to Spring's `DataAccessException`

#### 3.5 Can you execute a plain SQL statement with the JDBC template?

Yes there are:

* exectute
* update*
* batchUpdate
* query
* queryFor*

methods accepting plain SQL.

#### 3.6 When does the JDBC template acquire (and release) a connection, for every method called or once per template? Why?

JDBC template acquires/releases connection on each method call.
Connection should have shortest scope possible
to avoid resource leaks and timeouts.

Different story is while in transaction then Connection is kept
open not abort transaction.

#### 3.7 How does the JdbcTemplate support queries? How does it return objects and lists/maps of objects?

For single result `queryForObject` should be used, 
for single row `queryForMap` and normal results use
`queryForList`. You can provide custom RowMapper but Spring
provides two: `ColumnMapRowMapper` that converts row
to `Map<String, Object>` and `BeanPropertyRowMapper` that
uses reflection to set properties on bean class.

#### 3.8 What is a transaction? What is the difference between a local and a global transaction?

Transaction is single indivisible unit of work that either
succeeds or fails as a whole. It will never be partially
complete.

Transactional resource is an entity that has support for
doing transational work, database or message queue for example.
Local transaction considers a single resource. When transaction
spans multiple resources it is called an global transcation.

#### 3.9 Is a transaction a cross cutting concern? How is it implemented by Spring?

Transaction is cross cutting concern because it can span
multiple method calls and objects.

Core abstraction is `PlatformTransactionManager` for 
imperative code and `ReactiveTransactionManager` for
reactive code. They are strategies implemented differently
for different transactional resources, however the API
is common. 

Use of transactions can be either programmatic or declarative.
Programatically we can use `TransactionTemplate` or inject
and use `TransactionManager` directly. Declarative transaction
management is implemented by AOP. Transactional methods
are marked by `@Transactional` annotation which when detected
causes target object to be wrapped with generated code
that uses `TransactionManager` to enforce transactionality.

#### 3.10 How are you going to define a transaction in Spring?

You put `@Transactional` annotation on class or method. Parameters:

* value - which transaction manager if many
* propagation
* isolation
* timeout
* readOnly
* rollbackFor - classes for exceptions which cause rollback
by default only unchecked exceptions are rolled back
* noRollbackFor - don't rollback
* rollbackForClassName
* noRollbackForClassName
* label

#### 3.11 Is the JDBC template able to participate in an existing transaction?

JDBC template uses `DataSourceUtils` which is low level Spring code
that uses `TransactionSynchronizationManager` singleton that applies
transactional logic to underlying `Connection`. It does this by
binding `Connection` and transcation objects to thread as `ThreadLocal`.

#### 3.12 What is @EnableTransactionManagement for?

`@EnableTransactionManagement` enables scanning for `@Transactional`.

#### 3.13 How does transaction propagation work?

Transaction propagation determines what happens when
transcations are nested. Spring uses concept of logical
transcations that are mapped to physical transactions.

PROPAGATION_REQUIRED - nested logical transaction is mapped
to same physical transaction. When nested transcation is
rolled back outer transcation hasn't yet finished and this
results in unexpected behavior. So outer transaction catches
`UnexpectedRollbackException` so it knows nest transaction failed.

PROPAGATION_REQUIRES_NEW - nested logical transaction is
mapped to another physical transaction so they can
act independently

PROPAGATION_NESTED - nested uses same physical transaction
but uses JDBC savepoints to partially rollback. So it is
possible that outer transaction can commit without failed
nested parts.

#### 3.14 What happens if one @Transactional annotated method is calling another @Transactional annotated method inside a same object instance?

Default advice mode is JDK proxy so call from inside will silently
execute without trasactionality applied. For such cases
consider aspectj with compile-time weaving.

#### 3.15 Where can the @Transactional annotation be used? What is a typical usage if you put it at class level?

Transcational annotation can be used on class or method with method taking predecence.
Declared on class it adds transactionality to all public methods and
of it's descendants. You put general settings on class level and override
them at method level.

#### 3.16 What does declarative transaction management mean?

Declarative is different from imperative. You annotate
method/class with declaration and it gets interpreted automatically/
turns into code.

#### 3.17 What is the default rollback policy? How can you override it?

Rollback for unchecked exceptions no rollback for checked exceptions. You
can override it with parameter `rollbackFor`.

#### 3.18 What is the default rollback policy in a JUnit test, when you use the @RunWith(SpringJUnit4ClassRunner.class) in JUnit 4 or @ExtendWith(SpringExtension. class) in JUnit 5, and annotate your @Test annotated method with @Transactional?

Methods annotated with @Transcational are run in test-managed transaction. Default
rollback policy is to roll back on method exit. Note that lifecycle
methods like @BeforeAll are not supported and to run
them in transaction you have to inject PlatformTransactionManager
by yourself. If methods are annotated with @Transactional 
and @Commit then method commits the transaction.

#### 3.19 Are you able to participate in a given transaction in Spring while working with JPA?

`JpaTransactionManager` supports exposing plain `DataSource` access
for mixing JPA with JDBC. So you are possible to mix
JPA and JDBC code within same transaction.

#### 3.20 Which PlatformTransactionManager(s) can you use with JPA?

`JpaTransactionManager`, `HibernateTransactionManager` and `JtaTransactionManager`.

#### 3.21 What do you have to configure to use JPA with Spring? How does Spring Boot make this easier?

First you have to include dependency for Hibernate 5+ in pom.
Then you create `LocalEntityManagerFactoryBean` configured with persistence unit name.
Then you can inject `EntityManagerFactory` to your dao using 
`@PersistenceUnit` annotation or transactional `EntityManager` with
`@PersistenceContext`. JPA natively includes transaction support


### Section 4: Spring Data JPA

#### 4.1 What is a Spring Data Repository interface?

`Repository` interface is a marker interface for dynamically
generated Spring Data Repositories. It takes two type arguments
domain entity and it's ID's type. Extending this interface or it's descendands
`CrudRepository` and `PagingAndSortingRepository`
allows to generate queries by method name and inherit
standard JPA's crud operations. 

#### 4.2 How do you define a Spring Data Repository interface? Why is it an interface not a class?

You either make your repository interface extend `Repository` interface or
or mark it as `@RepositoryDefinition`.  It's an interface
to enable dynamic creation of queries by method names.

#### 4.3 What is the naming convention for finder methods in a Spring Data Repository interface?

Parsing query names is divided into subject (find..By) and predicate.
Property resolution algorithm splits string at camel case parts
and checks if domain entity is given with that name, if no
it proceeds to the next part. Property names can be joined with with and and or.

#### 4.4 How are Spring Data repositories implemented by Spring at runtime?

Repositories are scanned on application startup, and for 
each repository singleton bean is generated by JDK dynamic proxies
and registered in context.

#### 4.5 What is @Query used for?

There are times that query is too complex to 
be generated by method name. Then one can annotate
method with `@Query` and write the query by oneself.
Arguments from the method match ?1 placeholders.

There is also possiblity to use JPA's @NamedQuery.
`@Query` allows to specify native mode and write untranslated
SQL. 

### Section 5: Spring MVC Basics 

#### 5.1 What is the @Controller annotation used for?

`@Controller` is specialization of `@Component` and
it is used for autodetecting web components
during classpath scanning. It is typically used with
annotated handler methods (`@RequestMapping`).

#### 5.2 How is an incoming request mapped to a controller and mapped to a method?

Processing starts with `DispatcherServlet` that maintains
application context in servletcontext under special key.

Locale and theme resolvers are bound to request.

If multipart file resolver is specified the request is 
examined for multiparts. If they are found while `HttpServletRequest`
is wrapped in `MultipartHttpServletRequest`.

An appropirate handler is searched for. If found execution
chain associated with handler (preprocessors, postprocessors
and controllers) is run to prepare model for rendering.

Model is returned and view is rendered.

`HandlerExceptionResolver` is used throughout the process
to resolve exceptions.

Mappings are added to the registry based on requestmapping
annotations.

#### 5.3 What is the difference between @RequestMapping and @GetMapping?

`@GetMapping` is specialization of `@RequestMapping` with
method set to `GET`. It is meta annotated by it.

#### 5.4 What is @RequestParam used for?

`@RequestParam` is used to bind servlet request's parameters
or form data to method arguments. If wrapped with optional
or set with `required=false` it can be nullable. Type conversion
is automatically applied. If annotated on Map<String, String>
all parameters are populated in this map. It is possible
to have multiple values for same parameter in such case
annotate it on array or list

#### 5.5 What are the differences between @RequestParam and @PathVariable?

`@PathVariable`  is used to capture parameters encoded in URI.

#### 5.6 What are the ready-to-use argument types you can use in a controller method?

* primitive types
* POJOs
* ServletRequest, ServletResponse
* HttpSession
* Principal
* Locale
* TimeZone, ZoneId
* Errors, BindingResult

#### 5.7 What are some of the valid return types of a controller method?

* @ResponseBody
* HttpEntity<B>, ResponseEntity<B>
* HttpHeaders
* String
* View
* ModelAndView
* void
* DeferredResult<V>
* Callable<V>
* ResponseBodyEmitter, SseEmitter
* StreamingResponseBody
* Mono, Flux

### Section 6: Spring MVC REST

#### 6.1 What does REST stand for?

REST is an acronym for REpresentational State Transfer.
It is a set of principles and constraints for making
distributed hypermedia systems.

#### 6.2 What is a resource?

Resource is any piece of information that can be named.
For example it can be document or image. The state of 
resource is known as resource representation which consists
of the data, metadata describing data and hypermedia links
that can help clients to transition to next state.

#### 6.3 Is REST secure? What can you do to secure it?

REST is only as secure as the means that have been undertaken
to maintain integrity of messages and confidentiality.

We can secure plain HTTP communication by:

* using TLS
* hashing passwords
* never exposing information in urls
* delegating security to oauth provider
* adding timestamp to metadata
* validating input parameters
* keeping design simple

#### 6.4 Is REST scalable and/or interoperable?

Scalable: one of it's principles is layered architecture
constraining components behaviour. Further loadbalancing
can be applied and components scaled horizontally.

Interoperable: state's representation is language agnostic
and actions are driven through hyperlinks.

#### 6.5 Which HTTP methods does REST use?

GET - retrieve information about resource
POST - create resource
PUT - update resource
DELETE - delete resource
PATCH - prequisite is resource existing, update exact fields

#### 6.6 What is an HttpMessageConverter?

`HttpMessageConverter` is a mean to convert plain `HttpMessage`
to and from Java object. It provides a list of Class objects
for which it can be used. Converters are stored in list
used by `HandlerAdapter` to convert `@RequestBody` and
`@ResponseBody` before controller is called. Default converters
are registered for primitive types and json. You can 
register your own converters by WebMvcConfigurer.

#### 6.7 Is @Controller a stereotype? Is @RestController a stereotype?

`@Controller` is stereotype and specialization of `@Component`.
It marks class as web component detected through scanning.
`@RestController` is convenience annotatation denoting
Controller whose `@RequestMapping` methods have `@ResponseBody`
annotation.

#### 6.8 What is the difference between @Controller and @RestController?

`@RestController` is meta annotated with `@Controller`.
It is used for RESTful API.

#### 6.9 When do you need to use @ResponseBody?

`@ResponseBody` means that instead rendering a view,
returned object should be taken directly or converted with
`HttpMessageConverter`.

#### 6.10 What are the HTTP status return codes for a successful GET, POST, PUT or DELETE operation?

200 OK

#### 6.11 When do you need to use @ResponseStatus?

If you want to override default settings (200 OK). For example
correct return type for successful POST is 201 Created.

#### 6.12 Where do you need to use @ResponseBody? What about @RequestBody?

`@ResponseBody` is used on top of the method or class. `@RequestBody`
is used on methods parameter.

#### 6.13 If you saw example Controller code, would you understand what it is doing? Could you tell if it was annotated correctly?

No

#### 6.14 What Spring Boot starter would you use for a Spring REST application?

`spring-boot-starter-web` or `spring-boot-starter-webflux`.

#### 6.15 If you saw an example using RestTemplate, would you understand what it is doing?

Notable methods are:

* exchange
* getForEntity
* getForObject
* postForEntity
* postForObject
* postForLocation
* headForHeaders
* optionsForAllow
* put

### Section 7: Security

#### 7.1 What are authentication and authorization? Which must come first?

Authentication - confirming identity of client and server

Authorization - confining access to resources to
authenticated users that are granted permissions

#### 7.2 Is security a cross cutting concern? How is it implemented internally?

Yes, it's implemented as Servlet's filter.

#### 7.3 What is the delegating filter proxy?

It is Servlet's Filter implementation that matches
Filter with Spring's ApplicationContext. It nests
logical filter beans that are registered in this context.

#### 7.4 What is the security filter chain?

Security filter chain is used by filter chain
proxy (nested in delegating filter proxy) to
determine which Spring Security Filter's should
be invoked. Instead of standard filters url-based
invocation decision which filter to invoke can 
be conditional on any HttpServletRequest attribute.

#### 7.5 What is a security context?

`SecurityContext` is obtained from `SecurityContextHolder`
which stores it in `ThreadLocal` variable. Security filter
chain ensures it is properly cleared. `SecurityContextHolder`
contains `Authentication` object that:
* is input to `AuthenticationManager` to provide credentials
user used to authenticate
* represents currently authenticated user
* contain authorities, the `GrantedAuthority` object of 
permissions granted to the user

#### 7.6 What does the ** pattern in an antMatcher or mvcMatcher do?

Ant: matches zero or more directories in a path
Mvc: zero or more directories until the end of the path

#### 7.7 Why is the usage of mvcMatcher recommended over antMatcher?

* mvcMatcher restricts access to http method
* mvcMatcher restricts use of ** for matching multiple
path segments so that it's only allowed at the end of the
pattern. This eliminates many cases of ambiguity when
choosing the best matching pattern for given request.

#### 7.8 Does Spring Security support password encoding?

Spring Security's `PasswordEncoder` interface is used to
perform one way password transformation to store it securely.

#### 7.9 Why do you need method security? What type of object is typically secured at the method level (think of its purpose not its Java type).

There are 2 types of autorization in Spring Security. First
is global per request authorization based on url via HttpSecurity.

The second one is using `@PreAuthorize` annotation on any objects method.

As the codebase gets larger you want to have authorization rules aligned with
the code itself. It greatly simplifies the design.

#### 7.10 What do @PreAuthorized and @RolesAllowed do? What is the difference between them?

`@RolesAllowed` is standard Java's JSR-250 annotation that Spring
implemented for compliance. 
`@PreAuthorized` is more powerful and finegrained it uses
SpEL.

#### 7.11 How are these annotations implemented?

Spring Security provides interceptors to secure method invocations.
This is done with standard spring's AOP. Pre-invocation 
decision if the invocation is allowed is made by
`AccessDecisionManager`. It has `decide` method which is 
passed all neccessary information to make decision plus
secure object (method invocation).

#### 7.12 In which security annotation, are you allowed to use SpEL?

`@PreAuthorized`, `@PostAuthorized`, `@PreFilter`, `@PostFilter`

### Section 8: Testing

#### 8.1 What type of tests typically use Spring?

* Unit Tests
* Integration Tests

#### 8.2 How can you create a shared application context in a JUnit integration test?

You provide `@ExtendWith(SpringExtension.class)` or
`@RunWith(SpringRunner.class)` with
`@ContextConfiguration(classes=..)` or 
`@ContextConfiguration(locations=..)`.

You can also use `@SpringBootTest(classes=...)`

#### 8.3 When and where do you use @Transactional in testing?

`@Transactional` is used on classes (every test method in class
hierarchy is transactional) or method. By default transaction
is rolled back. Note that JUnit lifecycle methods are not
transactional. Method not annotated with @Transactional is
not run within transaction.

#### 8.4 How are mock frameworks such as Mockito or EasyMock used?

Spring Boot enables adding mocked class to context via
`@MockBean` and `@SpyBean`.

Other way around is to add mock to `@Configuration` and
autowire it in test.

#### 8.5 How is @ContextConfiguration used?

You annotate it on test's class. It's attributes specify
`@Configuration` or xml to load or context initializers.
It is inherited and can be meta-annotated.

#### 8.6 How does Spring Boot simplify writing tests?

Spring boot provides `@SpringBootTest` annotation that
finds default context to load by find `@SpringBootApplication`.
It by default sets up mock environment for testing web endpoints.
Starter includes hamcrest, assertj, JSONassert, JsonPath.
It adds `@MockBean` and `@SpyBean` to inject mocks to the context.
It adds functionality of autoconfigured tests where you specify
by annotation that some part of tools should be autoconfigured.
@JsonTest, @WebMvcTest, @JdbcTest, @DataJpaTest, @WebFluxTest are
the examples.

#### 8.7 What does @SpringBootTest do? How does it interact with @SpringBootApplication and @SpringBootConfiguration.

`@SpringBootTest` works up the package it is currently
defined until it finds `@SpringBootApplication` or 
`@SpringBootConfiguration`. `@SpringBootApplication` inherits
`@SpringBootConfiguration` which is primary source of configuration
for spring boot. As an alternative to `@Configuration` it can be
automatically detected.


### Section 9: Spring Boot Basics 

#### 9.1 What is Spring Boot?

Spring Boot is a project that helps to create Spring
applications with minimal configuration and coding.
It does so by providing sensible defaults, autoconfiguration
that can be overridden and dependency management of compatible
spring modules and third party tools.

#### 9.2 What are the advantages of using Spring Boot?

* ability to run web applications by `java -jar` thanks
to `spring-boot-loader` that can assemble and load 
super-jars encompassing all the dependencies
* autoconfiguration by classpath scanning, if bean is
detected on application startup defaults are configured
to reduce boilerplate from user site
* `spring-boot-devtools` provide automatic application
restarts on changed file
* `spring-boot-starters` eliminate need to specify
exact versions of spring modules and third party tools.
They are self-contained sets of dependencies grouped by
topic. Thus user is always assured he is using jars compatible
with each other.
* embedded application containers
* `spring-boot-actuator` adds production-grade features
to application
* YAML configuration files support

#### 9.3 What things affect what Spring Boot sets up?

Spring boot checks for the presence of file `META-INF/spring.factories`
with `org.springframework.boot.autoconfigure.EnableAutoConfiguration=...` in jar
on the class path. The value is `@Configuration` with additional
`@Conditional..` annotations that affects when the classes and beans are loaded.

#### 9.4 What is a Spring Boot starter? Why is it useful?

Spring Boot starter contains code to load, autoconfigure
and customize infrastructure of given technology. If
included as dependency it sets up all necessary dependencies
with preconfigured defaults.

#### 9.5 Spring Boot supports both properties and YML files. Would you recognize and understand them if you saw them?

...

### 9.6 Can you control logging with Spring Boot? How?

Spring supports Java Util Logging, Logback and Log4J2.
If using starters, by default Logback is used. File output
can be set by `logging.file.name` or `logging.file.path`
property. Levels can also be set in properties by setting
`logging.level.*` property. Custom configurations are 
taken from framework-specific config files in root of
the classpath or config prop `logging.config`. These are
for example `logback-spring.xml` or `log4j2-spring.xml`.

#### 9.7 Where does Spring Boot look for application.properties file by default?

From the classpath:
* classpath root
* classpath /config package

From current directory:
* current directory
* /config subdirectory and its children

#### 9.8 How do you define profile specific property files?

You put in scanned places file in format application-{profile}.properties.

#### 9.9 How do you access the properties defined in the property files?

* property placeholders in other files
* `@Value` annotation
* `@ConfigurationProperties(prefix=...)` on bean
* inject `Environment` and get them from there


#### 9.10 What properties do you have to define in order to configure external MySQL?

`spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass`

Spring Boot does it's best to deduce driver from url, but
sometimes setting `spring.datasource.driver-class-name`
can be helpful. By default HikariCP is used as connection
pool.

#### 9.11 How do you configure default schema and initial data?

`import.sql` in classpath root is executed on startup if
Hibernate creates schema from scratch (`spring.jpa.hibernate.ddl-auto=none|validate|update|create|create-drop`).

Spring Boot for any JDBC datasource can create schema by 
loading from standard root `schema.sql` and `data.sql`.
They can also be in form of `schema-{platform}.sql` where
platform is of `spring.sql.init.platform` property.

It can also be done using flyway.

#### 9.12 What is a fat jar? How is it different from the original jar?

Fat jar contains all application dependencies nested. Standard
solutions unpack all the classes in all jars and put them in
one place which is a problem. Spring Boot solves it by custom
jar format that preserves jar organization.

#### 9.13 What embedded containers does Spring Boot support?

Tomcat, Jetty, Undertow for Servlets.

Tomcat, Jetty, Undertow, Netty for WebFlux.

### Section 10:  Spring Boot Auto Configuration 

#### 10.1 How does Spring Boot know what to configure?

Spring scans classpath to find `factories` which contains
autoconfiguration root. From there using `@Conditional` annotations
it registers needed beans.

#### 10.2 What does @EnableAutoConfiguration do?

It enables the above, it is already included
in `@SpringBootApplication`.

#### 10.3 What does @SpringBootApplication do?

* marks class as root configuration
* enables component scanning
* enables autoconfiguration

#### 10.4 Does Spring Boot do component scanning? Where does it look by default?

If scanning path is not specified it is done from the current package.

#### 10.5 How are DataSource and JdbcTemplate auto-configured?

Data source is conditional on `spring.datasource.url` if it is not
detected Spring Boot autoconfigures in memory store. Otherwise
`spring.datasource.*` are used to configure it. Jdbc template
is injected the datasource.

#### 10.6 What is spring.factories file for?

see 9.3

#### 10.7 How do you customize Spring Boot auto configuration?

If you define class, bean, property Spring boot automatically backs off.

#### 10.8 What are the examples of @Conditional annotations? How are they used?

`@ConditionalOnClass`, `@ConditionalOnBean`, `@ConditionalIfPropertyPresent`

### Section 11:  Spring Boot Actuator

#### 11.1 What value does Spring Boot Actuator provide?

Actuator adds production ready features to Spring Boot
by adding endpoints for controlling and monitoring application.

#### 11.2 What are the two protocols you can use to access actuator endpoints?

JMX and HTTP. By default all JMX endpoints are enabled and exposed with
exception of /shutdown. By default all HTTP endpoints are disabled and
not exposed with exception to /health.

#### 11.3 What are the actuator endpoints that are provided out of the box?

* /actuator/auditevents
* /actuator/beans
* /actuator/caches
* /actuator/conditions
* /actuator/configprops
* /actuator/env
* /actuator/flyway
* /actuator/health
* /actuator/httptrace
* /actuator/info
* /actuator/integrationgraph
* /actuator/loggers
* /actuator/liquibase
* /actuator/metrics
* /actuator/mappings
* /actuator/quartz
* /actuator/scheduledtasks
* /actuator/sessions
* /actuator/shutdown
* /actuator/startup
* /actuator/threaddump

#### 11.4 What is info endpoint for? How do you supply data?

/info is for exposing arbitrary application information.
The data is collected from `InfoContributor` beans registered
in application. By default if feasible Spring provides
information of build, environment, git and java.

If `env` contributor is enabled you can supply arbitrary
data but setting `info.*` spring properties. You can also
extend `InfoContributor` interface and register bean in
context.

#### 11.5 How do you change logging level of a package using loggers endpoint?

curl http://localhost:8080/actuator/loggers/{logger}
{"configuredLevel":{null to reset},"effectiveLevel":"{INFO|WARN|DEBUG|TRACE|ERROR|FATAL|OFF|null}"}

#### 11.6 How do you access an endpoint using a tag?

Tags are for metrics. Example:

curl 'http://localhost:8080/actuator/metrics/jvm.memory.max' -i -X GET

it shows available tags then to drill down pass tag as query
parameter.

#### 11.7 What is metrics for?

Metrics are numerical statistics for the purpose of
monitoring non-functional aspects of application for
example performance. Spring Boot provides autoconfiguration
for Micrometer an application metrics facade that supports
numerous monitoring systems including InfluxDB and Prometheus.

#### 11.8 How do you create a custom metric?

You register bean, inject `MeterRegistry` and invoke
its methods.

#### 11.9 What is Health Indicator?

Health indicator shows status of running application
and possibly systems it's depending. It shows info
collected from `HealthContributor` beans, some of them
are registered by default.

#### 11.10 What are the Health Indicators that are provided out of the box?

`db`, `ping`, `diskspace`, and others.

#### 11.11 What is the Health Indicator status?

It's value object representing state of component or a
system.

#### 11.12 What are the Health Indicator statuses that are provided out of the box?

* UNKNOWN 
* UP - functioning as expected
* DOWN - faced unexpected failure
* OUT_OF_SERVICE - had been taken down for maintenance

#### 11.13 How do you change the Health Indicator status severity order?

You list orders under property:

`management.endpoint.health.status.order` from most severe
to least.

#### 11.14 Why do you want to leverage 3rd-party external monitoring system?

For ex. to centralize access to observability to one place.

### Section 12:  Spring Boot Testing 

#### 12.1 When do you want to use @SpringBootTest annotation?

`@SpringBootTest` should be used when you want:

* have context automatically created same as for application
* test slices of context
* use Spring Boot's features
* use autoconfiguration

#### 12.2 What does @SpringBootTest auto-configure?

* JSON tests
* MVC tests
* WebFlux tests
* Jpa/Jdbc tests
* rest/webservices clients

#### 12.3 What dependencies does spring-boot-starter-test brings to the classpath?

* JUnit 5
* Spring Test and Spring Boot Test
* AssertJ
* Hamcrest
* Mockito
* JSONassert
* JsonPath

#### 12.4 How do you perform integration testing with @SpringBootTest for a web application?

You let it create the context and autowire tested components into
the test.

#### 12.5 When do you want to use @WebMvcTest? What does it auto-configure?

`@WebMvcTest` is used to test whether controllers work
as expected. It configures Spring MVC infrastructure and
limits scanning only to web layer. Then you can inject
`MockMvc` that wraps controllers and lets you to do 
assertions.

#### 12.6 What are the differences between @MockBean and @Mock?

`@Mock` is JUnit Extension's annotation to create mock for class.
`@MockBean` is Spring Boots annotation to further inject this mock
into container.

#### 12.7 When do you want @DataJpaTest for? What does it auto-configure?

You use it to test `@Entity` classes and it autoconfigures
Spring Data Repositories. If embedded database is on 
classpath it configures it as well. Further `JdbcTemplate`
is available to use. By default JPA tests are transactional
and transactions are rolled back.
