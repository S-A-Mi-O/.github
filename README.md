

# **SCHEMA-AGNOSTIC MICROSERVICES ORCHESTRATOR (S.A.Mi.O)**

## **Introduction**
This is a personal research project that simplifies tedious aspects of **schema evolution** in a **microservices architecture**. It generalizes **non-domain logic** and **configuration** to a degree that allows setting up a service by writing only **domain-specific code**. In essence that means: **no entity- or property-names** will be used outside a (domain)-service's directly domain-related code. Within this project that would be considered hard-coding and is not necessary at all. This way the **need for tedious adjustments across services and libraries related to schema-evolution becomes obsolete** and code repetition across services is minimized. It is now possible to **exclusively focus on domain topics** like data-modelling and business logic. This on the one hand **increases development speed and code-consistency**, and on the other hand reduces complexity and the risk of human error. Since there is no such thing as free-lunch this comes at the cost of extensive use of reflection. The resulting performance impact is in part mitigated through caching.

### **Tech Stack:**
- **Language:** Kotlin 
- **Framework:** Spring Boot
- **Databases:** PostgreSQL (via Hibernate)
- **Messaging & Streaming:** Apache Kafka & Zookeeper
- **Service Discovery:** Netflix Eureka
- **Caching:** Redis
- **Containerization:** Docker

🚧 **This project is a work in progress and is not intended for production.**

---

### **Key Features & Architecture** 🚀

This project introduces a schema-agnostic microservices orchestration approach, eliminating redundant code and streamlining service development through automation and abstraction.

✅ **Schema-Agnostic CRUD & Search**

- Pre-implemented CRUD functionality works without requiring explicit logic.

- Supports schema-agnostic Search-Filter algorithms with caching.

- Handles nested complex objects and various data types.

- Maintains type safety while avoiding unnecessary DTOs.

✅ **Event-Driven Architecture & Kafka Integration**

- Zero-code dynamic Kafka topic declaration ensures seamless messaging across services.

- Zero-code automatic Kafka listener registration dynamically detects and registers required listeners.

- Uses a single, general-purpose event (EntityEvent) for all entity modifications.

- Differentiates between upstream & downstream entities using strict conventions.

✅ **Dynamic Service Discovery & Gateway**

- Automatic service discovery using Netflix Eureka.

- Centralized API Gateway managing:

🔹 Authentication via JWT tokens.

🔹 Role-Based Access Control (RBAC).

🔹 Permission-Based Access Control configurable at the service level.

🔹 Rate limiting to prevent abuse.

🔹 Activity tracking for automatic log-out when inactive.

✅ **Dynamic Entity Augmentation**

- REST-based pseudo-property augmentation, allowing entities to be extended dynamically at runtime.

- Type-safe, runtime-validated pseudo-properties enable flexible schema evolution without database schema modifications.

✅ **Developer Productivity & API Documentation**

- Uses general-purpose, type-safe DTOs (CreateRequest, UpdateRequest, DeleteRequest, SearchRequest, EntityEvent).

- Swagger-generated API documentation for all services.

- Database seeding with mock data for effortless local development and testing.

✅ **This system follows a convention-over-configuration paradigm, ensuring high automation while preserving extensibility for advanced customization.** 

---

## **Conventions**

### **Entity Naming Conventions**
- **All services must use UUIDs as identifiers.**
- **Upstream vs. Downstream Entities**:
    - **Upstream entities**: The **single source of truth**.
    - **Downstream entities**: Can **only be modified via events**.
    - **Downstream entities must be prefixed with an underscore (`_`)**.
    - Example: `Order` (upstream) → `_Order` (downstream).

### **Private Fields**
- Sensitive data fields should be **private and encrypted**.
- Private fields must be **prefixed with `_`** to ensure seamless serialization/deserialization.

### **Event-Based Updates**
- Kafka topics, listeners, and event deserialization work based on **naming conventions**.
- **All-purpose Event**:

```kotlin
data class EntityEvent(
    val entityClassName: String,
    val id: UUID,
    val type: ModificationType,
    val properties: Map<String, Any?>
)
```
Of course additional event types may be created. This would require manual topic declaration and listener configuration, though.

- The `ModificationType` enum **categorizes** changes (creation, deletion, etc.).

### **Schema-Agnostic DTOs**
- DTOs eliminate schema-specific request models. Instead, entities reference **generic DTOs** like:

```kotlin
data class SearchRequest(
    val params: List<SearchParam> = emptyList()
)
```

- **Example Search Parameter**:

```kotlin
data class SearchParam(
    val operator: Operator,
    val searchValue: Any? = null,
    val path: String // Example: "userInfo.address.city"
)
```

---

## **Getting Started**

### **Setup Instructions**
Since the project uses **GitHub Packages** and **Personal Access Tokens (PATs)**, you need to adjust dependencies and environment variables.

### **Steps:**
1. **Clone all repositories**.
2. **Push them into repositories you own**.
3. **Update dependencies:**
    - Replace `S.A.Mi.O Core` in dependencies with **your own version**.
    - Update **Dockerfiles** to reference your repositories.
    - Adjust **annotations** in main classes.
4. **Set up environment variables**, for example in `.env`:

```plaintext
GITHUB_USERNAME=<YOUR GITHUB USERNAME>
GITHUB_TOKEN=<YOUR GITHUB PERSONAL ACCESS TOKEN>
JWT_SECRET=<YOUR SECRET>
GUEST_PASSWORD=guest
REDIS_HOST=redis
REDIS_PORT=6379
USERSERVICE_DATASOURCE_URL=jdbc:postgresql://postgresql_user-service:5432/userservicedb
USERSERVICE_DATASOURCE_USERNAME=username
USERSERVICE_DATASOURCE_PASSWORD=password
GATEWAYSERVICE_DATASOURCE_URL=jdbc:postgresql://postgresql_user-service:5432/userservicedb
GATEWAYSERVICE_DATASOURCE_USERNAME=username
GATEWAYSERVICE_DATASOURCE_PASSWORD=password
PSEUDOPROPERTYSERVICE_DATASOURCE_URL=jdbc:postgresql://postgresql_user-service:5432/propertyservicedb
PSEUDOPROPERTYSERVICE_DATASOURCE_USERNAME=username
PSEUDOPROPERTYSERVICE_DATASOURCE_PASSWORD=password
KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

5. **Install Docker Desktop** and ensure the **Docker context is correct**.

6. **Build the project:**
```sh
docker compose build --no-cache --build-arg GITHUB_USERNAME=<YOUR_GITHUB_USERNAME> --build-arg GITHUB_TOKEN=<YOUR_GITHUB_ACCESS_TOKEN>
```

7. **Run the project with mock users:**
```sh
CREATE_USERS=<NUMBER_OF_MOCK_USERS> docker compose up
```

🔍 **Monitor logs & services in Docker Desktop**.

8. **Try REST calls using HTTP requests** pre-implemented in the `http-folder` of `gateway-service` (if you are using IntelliJ).

9. **Access API documentation:**
    - Open in browser: `http://localhost:<SERVICE_PORT>/swagger-ui`

---

## **Set Up a New Service**

To set up a new service, create it using your preferred method, such as [Spring Initializr](https://start.spring.io/).

### **Versions**
This project is built on:
- **Java**: 21
- **Spring Boot**: 3.3.4
- **Kotlin**: 1.9.25
- **Hibernate**: 6.6.3
- **Gradle**: 8.10.2

### **Required Dependencies**
Ensure your service includes the following dependencies:
- Spring Web
- Spring Data JPA
- Spring Kafka
- Hibernate Core
- PostgreSQL
- Hypersistence
- **S.A.Mi.O Core** (your own repository)

Make sure your **S.A.Mi.O Core** is published and accessible via GitHub Packages (PAT setup required).

### **Essential Classes**
A fully functional service requires the following components:

#### **1. Entities**
- Use **PostgreSQL** with Hibernate.
- Mark sensitive fields as **private** and encrypt them with the utility provided by S.A.Mi.O Core. For an entity ``User`` with a private field ``_password`` it would look like this:
```kotlin
@Entity
@Table(name = "users")
class User(
    @ValidPassword
    @NotBlank(message = "Password is mandatory")
    private var _password: String = "",
  ) : AugmentableBaseEntity() {

    open var password: String
        get() = _password
        set(value) {
            if (!PasswordValidator.isValid(value, null)) {
                throw IllegalArgumentException("Password does not meet the requirements")
            }
            _password = PasswordCrypto.hashPassword(value)
        }

}

```

#### **2. Controller for Each Upstream Entity**
Controllers are easy to set up:

```kotlin
@RestController
@RequestMapping("/your-upstream-entity-plural")
@ControllerFor(YourUpstreamEntityClass::class)
class YourUpstreamEntityController : RestControllerTemplate<YourUpstreamEntity>()
```
By inheriting from `RestControllerTemplate`, the following endpoints become available **without additional code**:

| Method  | Endpoint              | Parameters                            | Response                  |
|---------|-----------------------|---------------------------------------|---------------------------|
| **POST**  | `/create`             | CreateRequest                         | ResponseEntity<T>         |
| **PATCH** | `/update`             | UpdateRequest                         | ResponseEntity<T>         |
| **DELETE** | `/delete/{id}`       | UUID Path Variable                    | HTTP-Status               |
| **GET**  | `/getSingle/{id}`     | UUID Path Variable                    | ResponseEntity<T>         |
| **GET**  | `/getMultiple/{ids}`  | List<UUID>, Page & PageSize Params    | ResponseEntity<Page<T>>   |
| **GET**  | `/getAllPaged/all`    | Page & PageSize Params                | ResponseEntity<Page<T>>   |
| **GET**  | `/search`             | SearchRequest, Page & PageSize Params | ResponseEntity<Page<T>>   |

#### **3. Service Class for Each Entity**

```kotlin
@Transactional
@Service
@RestServiceFor(YourUpstreamEntity::class)
class YourUpstreamEntityRestService : RestServiceTemplate<YourUpstreamEntity>()
```
Custom business logic can be added to this class.

#### **4. Persistence Adapter for Each Upstream Entity**

```kotlin
@Service
@PersistenceAdapterFor(PseudoProperty::class)
class PseudoPropertyPersistenceAdapter : EntityPersistenceAdapter<PseudoProperty>()
```

#### **5. Repository Interface for Each Upstream Entity**

```kotlin
interface PseudoPropertyRepository : EntityRepository<PseudoProperty, UUID>
```

🔹 **Downstream Entities** follow the same structure but must inherit from `DownstreamRestControllerTemplate` or `DownstreamRestServiceTemplate`. Prefix their names with an **underscore (`_`)**.

---

## **Configuration Files**

### **`application.yml`**

#### **Define a Port**
```yaml
server:
  port: 8082
```

#### **Spring Configuration**
```yaml
spring:
  main:
    allow-bean-definition-overriding: true

  application:
    name: pseudoproperty-service

  kafka:
    bootstrap-servers: kafka:9092
    consumer:
      auto-offset-reset: earliest

  datasource:
    url: jdbc:postgresql://postgresql_pseudoproperty-service:5432/pseudopropertyservicedb
    username: username
    password: password

  jpa:
    hibernate:
      ddl-auto: update
      types:
        registration: hypersistence-utils
        json-dialect: true
    show-sql: false
    properties:
      '[hypersistence.utils.enable_types_contributor]': false
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: false
    database-platform: org.hibernate.dialect.PostgreSQLDialect

  data:
    redis:
      host: redis
      port: 6379
```

#### **Enable Service Discovery**
```yaml
eureka:
  client:
    enabled: true
    service-url:
      defaultZone: http://discovery-service:8761/eureka/
```

#### **Enable Method-Level Permissions**
```yaml
permissions: true
```
🔹 Methods can be annotated with `@ExecutionRestrictedToPermissions(List<String>)`. Permissions are managed via the `PermissionController` from **S.A.Mi.O Core**.

#### **Restrict Service Access to Specific Roles**
```yaml
service-level-access:
  restricted-to:
    - SUPER_ADMIN
```
Available roles: **GUEST, REGISTERED_USER, CUSTOMER, ADMIN, SUPER_ADMIN** (modifiable in `UserRole` enum in Core).

## **Final Steps**
- **Create a Dockerfile**
- **Add the new service to `docker-compose.yml` in Gateway**

---

## **Known Issues & Future Improvements**

- Centralized caching size setting is not implemented in a clean way.
- Security vulnerabilities need patching.
- `gateway-service` requires two restarts to receive events.
- Permission logic is untested.
- Kafka can sometimes get caught in loops.
- Build process is slow.
- scheduled tasks are polluting the logs
- not all exceptions propagate to the http response
- local profiles are not consistently implemented
- user-service is soon to be deprecated. It will be replaced with an account service which will feature multi-user-accounts
- crypto-functionality still has a misleadig password-related naming

📢 **This project is constantly evolving—stay tuned for improvements!**

---

## **Contribute & Explore**
This is an **exploratory project**, feel free to **experiment**, suggest improvements, or fork it for your own use! 🚀

👤 Author

This project is developed and maintained by Aron J. Vaupel.

📧 Email: vaupelaron@gmail.com
🔗 [GitHub](github.com/aronvaupel) 
💼 [LinkedIn](https://linkedin.com/in/aron-joachim-vaupel-445749222)

