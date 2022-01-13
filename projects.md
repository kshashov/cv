# Projects
Unfortunately, my commercial experience has nothing to do with open source, so I can only share my home projects created while studying some technologies.

## Vaadin Compose UI Framework ([demo](https://vaadin-compose.herokuapp.com/), [maven](https://jitpack.io/#kshashov/vaadin-compose), [repo](https://github.com/kshashov/vaadin-compose))

Provides an ability to build web pages on Kotlin in reactive way as if you are working with Flutter, Vue.js, React or other modern UI frameworks. Shortly, this is a combination of [Flutter](https://flutter.dev/) and [Vaadin Flow](https://vaadin.com/flow) frameworks.

> The main goal was to implement the primitive copy of Flutter's architecture, where the developer can simply declare reactive interfaces, and the framework reuses the already created render components between UI updates.

![Counter](https://github.com/kshashov/vaadin-compose/blob/main/img/vaadin-compose-counter.gif?raw=true "Counter")

```kotlin
@Route("counter")
class Counter : BaseComposablePage() {

   override fun build(context: BuildContext) = MainWidget()

   class MainWidget : StatefulWidget() {
      override fun createState() = MainState()

      class MainState : StatefulWidget.WidgetState<MainWidget>() {
         private var counter: Int = 0

         override fun build(context: BuildContext): Widget {
            return Container(
               direction = FlexLayout.FlexDirection.COLUMN,
               childs = listOf(
                  Label("Counter: $counter") {
                     style {
                        set("font-weight", "bold")
                     }
                  },
                  Button("+1", {
                     setState { counter++ }
                  })
               )
            )
         }
      }
   }
}
```

* Kotlin
  * Custom DSL
* Vaadin at the render side
* RxJava
* CI/CD
  * Builds: Travis CI
  * Code coverage: Jacoco -> Codecov
  * Deployment: Github Packages, JitPack
  * Demo deployment: Heroku

## Telegram Spring Boot Starter ([maven](https://search.maven.org/search?q=a:spring-boot-starter-telegram), [repo](https://github.com/kshashov/spring-boot-starter-telegram))

Provides an ability to create Telegram bots in Spring MVC style. Supports webhooks and long polling, sessions, custom arguments for contorller methods. 

```java 
    @BotRequest(value = "/hello", type = {MessageType.CALLBACK_QUERY, MessageType.MESSAGE})
    public BaseRequest hello(User user, Chat chat) {
        return new SendMessage(chat.id(), "Hello, " + user.firstName() + "!");
    }

    @MessageRequest("/hello {name:[\\S]+}")
    public String helloWithName(@BotPathVariable("name") String userName) {
        // Return a string if you need to reply with a simple message
        return "Hello, " + userName;
    }
```

* Spring Boot
    * [Custom scope](https://github.com/kshashov/spring-boot-starter-telegram/blob/master/src/main/java/com/github/kshashov/telegram/TelegramScope.java) The scope stores beans in the cache by the chat id. During the request processing the chat id is stores in the `ThreadLocal` variable.
    * [Auto-configuration](https://github.com/kshashov/spring-boot-starter-telegram/blob/master/src/main/java/com/github/kshashov/telegram/TelegramAutoConfiguration.java) The library is implemented as a Spring Boot Starter, so it is activated as soon as the project specifies a dependency on this project. The configuration prepare bot's configurations, declares all necessary beans, adds custom scope to BFPP, defines application listener that starts and stops the executor service service
    * [Bean Post Processor](https://github.com/kshashov/spring-boot-starter-telegram/blob/master/src/main/java/com/github/kshashov/telegram/TelegramControllerBeanPostProcessor.java) The BPP checks beans and their methods for certain annotations and adds the mathched ones to the repository. Later, this repository is used to find the appropriate request handler.
* Reflection. Firstly, I collect some metadata from handler methods in my BPP. Additionally, I added the ability to use the custom arguments for controller's methods. For this purpose I provided interfaces to add the custom argument resolvers and result value handlers and executes them in my [code](https://github.com/kshashov/spring-boot-starter-telegram/blob/master/src/main/java/com/github/kshashov/telegram/handler/processor/TelegramInvocableHandlerMethod.java). For example, [this class](https://github.com/kshashov/spring-boot-starter-telegram/blob/master/src/main/java/com/github/kshashov/telegram/handler/processor/arguments/BotRequestMethodPathArgumentResolver.java#L16) adds the support for path parameters: 
        ```java
        @BotPathVariable("name") String userName
        ```
* I use `Javalin` server to handler Telegram webhooks in case this option is chosen in the bot settings
* Unit tests with JUnit5 + Mockito. 
I covered 50% of the library. Mostly, it is the reflection usages, built-in resolvers and handlers container. I didn't create the integration Spring tests
* Jmx. I use the `io.dropwizard.metrics.io.dropwizard.metrics` library to expose some important metrics
* The API is covered by javadocs 
* CI/CD
    * Builds: CircleCI, Travis CI
    * Code coverage: Jacoco -> Codecov
    * Deployment: Maven Central, JitPack

## Scoped Methods Spring Boot Starter ([maven](https://search.maven.org/search?q=a:scoped-methods-spring-boot-starter), [repo](https://github.com/kshashov/scoped-methods))

Provides an ability to use custom scopes over Spring bean methods. Supports both proxy and aspectj interception approaches, meta-annotations and overriden methods.

```java
@Service
public class Service1 {

    @ScopedMethod("inner")
    public void doSomething() {
        log.info(ScopedMethodsHolder.getCurrent());
    }
}

@Service
public class Service2 {

    @ScopedMethod("outer0")
    @ScopedMethod("outer")
    public void doSomething() {
        log.info(ScopedMethodsHolder.getCurrent());    // outer
        service1.doSomething();                        // inner
        log.info(ScopedMethodsHolder.getCurrent());    // outer
    }
}
```

* Spring Boot
  * Configuration-related features
    *  [ImportSelector](https://github.com/kshashov/scoped-methods/blob/main/src/main/java/io/github/kshashov/scopedmethods/ScopedMethodsConfigurationSelector.java) Imports target configurations based on the `EnableScopedMethods` annotation parameters
    * [ImportAware](https://github.com/kshashov/scoped-methods/blob/main/src/main/java/io/github/kshashov/scopedmethods/BaseScopedMethodConfiguration.java) Populate additional settings based based on `EnableScopedMethods` annotation parameters
  * [DefaultPointcutAdvisor](https://github.com/kshashov/scoped-methods/blob/main/src/main/java/io/github/kshashov/scopedmethods/ProxyScopedMethodsConfiguration.java) Catches all invocations of `ScopedMethod` annotated methods. Starts scope before target method invocation and stops it after.
* [AspectJ](https://github.com/kshashov/scoped-methods/blob/main/src/main/java/io/github/kshashov/scopedmethods/ScopedMethodAspect.java) The `AspectJ` analog of the custom `DefaultPointcutAdvisor` 
* Unit test with JUnit5. Therea are [plain](https://github.com/kshashov/scoped-methods/blob/main/src/test/java/io/github/kshashov/scopedmethods/ScopedMethodsManagerTest.java) and [integration](https://github.com/kshashov/scoped-methods/tree/main/src/test/java/io/github/kshashov/scopedmethods/integration/proxy) unit tests. The code have `>80%` coverage.  
* CI/CD
    * Builds: CircleCI
    * Code coverage: Jacoco -> Codecov
    * Deployment: Maven Central, JitPack

## Translate It ([demo](https://kshashov.github.io/translate-it/#/), [front](https://github.com/kshashov/translate-it), [back](https://github.com/kshashov/Translates-API))
Simple analog of [writing section of Puzzle English](https://puzzle-english.com/writing/verb-tenses). Each user has role with permissions allowing him to modify other's user roles and solve, create, modify or delete exercises. The exercise can be solved by gradually translating each phrase from the source language to the target language. 

> The main goal was to try the VueJS since I had never worked with such frameworks before. The backend part is as simple as possible.

![exercise](https://github.com/kshashov/translate-it/raw/master/images/exercise.png "exercise")

### Frontend

* VueJS 2. Business logic is concentrated in the [store](https://github.com/kshashov/translate-it/tree/master/src/store) split into several modules. I also have [mixins](https://github.com/kshashov/translate-it/tree/master/src/mixins) for more convenient store using from ui components. 
  * Vuetify with many different components
  * Vuelidate, [for example](https://github.com/kshashov/translate-it/blob/master/src/views/exercise/edit/shared/StepsForm.vue)
* [CASL](https://github.com/kshashov/translate-it/blob/master/src/plugins/casl.js) Just show [custom component](https://github.com/kshashov/translate-it/blob/master/src/components/ProtectedComponent.vue) when the use dont have permission to see the content.
* [Axios](https://github.com/kshashov/translate-it/blob/master/src/plugins/axios.js) with a couple of interceptors to include token and handle errors
* CI/CD
    * Builds: GitHub Actions
    * Deployment: GitHub Pages

### Backend
* Java 11
* Spring Boot
  * Data
    * [JPA repositories](https://github.com/kshashov/Translates-API/tree/master/data/src/main/java/com/github/kshashov/translates/data/repos)
    * [SQL Flyway migrations](https://github.com/kshashov/Translates-API/tree/master/web/src/main/resources/db/migration)
    * H2 for local profile and PostgreSQL for prod
  * Web
    * [Controllers](https://github.com/kshashov/Translates-API/tree/master/web/src/main/java/com/github/kshashov/translates/web/controllers)
    * [ControllerAdvice](https://github.com/kshashov/Translates-API/blob/master/web/src/main/java/com/github/kshashov/translates/web/exceptions/RestExceptionHandler.java) 
  * Security
    * [OAuth2 Client](https://github.com/kshashov/Translates-API/tree/master/web/src/main/java/com/github/kshashov/translates/web/security/oauth2) (Google/GitHub). I followed [this tutorial] to add the OAuth2 support to the rest backend. There is an interesting part with custom `AuthorizationRequestRepository` that adds redirect url and authorization request to the short-lived cookie because by default requests are saved in the session
    * JWT tokens. The tokens [are created](https://github.com/kshashov/Translates-API/blob/master/web/src/main/java/com/github/kshashov/translates/web/security/oauth2/OAuth2AuthenticationSuccessHandler.java) after successful responses from OAuth2 servers and validated in the [custom filter](https://github.com/kshashov/Translates-API/blob/master/web/src/main/java/com/github/kshashov/translates/web/security/TokenAuthenticationFilter.java) 
    * `@PreAuthorize` in the [services](https://github.com/kshashov/Translates-API/blob/master/web/src/main/java/com/github/kshashov/translates/web/services/)
* [Swagger](https://translate-it-api.herokuapp.com/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config)

## Flutter Codenames ([demo](https://kshashov.github.io/codenames-web/), [repo](https://github.com/kshashov/flutter-codenames/))

A simple implementation of [Codenames](https://en.wikipedia.org/wiki/Codenames_(board_game)) board game on [Flutter](https://flutter.dev/) & [Firebase](https://firebase.google.com/).
> The main goal was to try the Flutter Web 

![codenames web](https://github.com/kshashov/flutter-codenames/raw/master/docs/codenames_web.png "Web")

* Flutter 2.8.1 stable channel
  * Bloc architecture with `rxdart` and `provider` libraries
  * Responsive layout depending on `MediaQuery`
  * Support for Web and Android builds
* Firebase real-time database as backend
  * Support for 'online' player statuses
* CI/CD
    * Deployment: Github Pages

## TaskTracker ([demo](https://time-tracker1.herokuapp.com/), [repo](https://github.com/kshashov/TimeTracker))
The application is a simple time tracker. All users can create projects, bind other users with specific roles to them. Users with the required permissions can create, modify and delete work logs in projects, view other people's logs, commit them to prohibit further changes.
> The main goal was to try the Vaadin Flow

![daily work](https://github.com/kshashov/TimeTracker/raw/master/images/daily.png)

* Spring Boot
  * Data
    * [JPA repositories](https://github.com/kshashov/TimeTracker/tree/master/timetracker-data/src/main/java/com/github/kshashov/timetracker/data/repo)
    * [SQL Flyway migrations](https://github.com/kshashov/TimeTracker/tree/master/timetracker-web/src/main/resources/db/migration)
    * Transactions
    * Entity Graphs
    * H2 for local profile and PostgreSQL for prod
  * Integration tests for [Data Layer](https://github.com/kshashov/TimeTracker/tree/master/timetracker-data/src/test/java/com/github/kshashov/timetracker/data)
    * JUnit5
    * Mockito
  * Security
    * OAuth2 client
  * [Vaadin](https://github.com/kshashov/TimeTracker/tree/master/timetracker-web/src/main/java/com/github/kshashov/timetracker/web/ui/views)
  * [CSS](https://github.com/kshashov/TimeTracker/tree/master/timetracker-web/frontend/styles/)
* CI/CD
    * Builds: Travis CI
    * Code coverage: Jacoco -> Codecov
    * Deployment: Heroku

## Cloud Experiments ([repo](https://github.com/kshashov/cloud-experiments))

In fact, this is not a project at all, it is just a collection of sub-projects using various Spring Cloud features.

> The main goal was to try the Spring Cloud stack.

* [Eureka](https://github.com/kshashov/cloud-experiments/tree/main/eureka)
* [Config](https://github.com/kshashov/cloud-experiments/tree/main/config). I use 'discovery first' approach, so config microservice is discovered by other services via Eureka
* [Gateway](https://github.com/kshashov/cloud-experiments/blob/main/gateway). I [aggregate](https://github.com/kshashov/cloud-experiments/blob/main/gateway/src/main/java/com/github/kshashov/cloud/gateway/GatewayApplication.java) internal APIs in the gateway Swagger
* [Feign](https://github.com/kshashov/cloud-experiments/blob/main/generator/src/main/java/com/github/kshashov/cloud/generator/client/ProducerClient.java)
* Stream
  * [@EnableBinding](https://github.com/kshashov/cloud-experiments/blob/main/producer/src/main/java/com/github/kshashov/cloud/producer/services/TaskRegistry.java)
* Kafka
  * [KafkaTemplate](https://github.com/kshashov/cloud-experiments/blob/main/producer/src/main/java/com/github/kshashov/cloud/producer/services/TaskRegistry.java)
  * [@KafkaListener](https://github.com/kshashov/cloud-experiments/blob/main/sum/src/main/java/com/github/kshashov/cloud/sum/SumApplication.java)
* Test
  * [TestRestTemplate](https://github.com/kshashov/cloud-experiments/blob/main/producer/src/test/java/com/github/kshashov/cloud/producer/controllers/TasksControllerTest.java) for Web
  * [MessageCollector](https://github.com/kshashov/cloud-experiments/blob/main/producer/src/test/java/com/github/kshashov/cloud/producer/controllers/TasksControllerTest.java) for Stream
  * [EmbeddedKafkaBroker](https://github.com/kshashov/cloud-experiments/blob/main/producer/src/test/java/com/github/kshashov/cloud/producer/controllers/TasksControllerTest.java) for Kafka
  * [WireMockServer](https://github.com/kshashov/cloud-experiments/blob/main/generator/src/test/java/com/github/kshashov/cloud/generator/client/FeignClientTasksProducerTest.java) for Feign service

## Other

* Atlassian Jira Plugin ([market](https://marketplace.atlassian.com/apps/1218423/jitlab-connect), [repo](https://github.com/JitLabConnect/jitlabconnect.github.io))
* Android Apps 
  * [Test project for Yandex](https://github.com/kshashov/Android-translate)
  * [Drugs catalog on Kotlin with SQLite](https://github.com/kshashov/Drugs)
  * [Learning project for Altarix](https://github.com/kshashov/Shashov.Kirill)
