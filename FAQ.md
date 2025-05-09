# ![Lakeside Mutual Logo](./resources/logo-32x32.png) Frequently Asked Questions and Troubleshooting

## Can I use other IDEs instead of the Spring Tools Suite?

See [IDE instructions](./IDE_INSTRUCTIONS.md) for details on how to use IntelliJ IDEA or Visual Studio Code.

## Which Spring anotations are used and what do they do? 

We plan to provide a table that gives an overview here, with links to API doc as well as a tutorial that explains how they work (future work).

## How can I make sure that the backend applications are running?

You can send a test request to the application or use the [Spring Boot Admin application](spring-boot-admin/README.md). 

### Customer Core
Run the application with `mvn spring-boot:run` and then check the state with `curl --header 'Authorization: Bearer b318ad736c6c844b' http://localhost:8110/customers?limit=1` (after the application has started). The curl command should return a customer:

```json
{
  "filter" : "",
  "limit" : 1,
  "offset" : 0,
  "size" : 50,
  "customers" : [ {
    "customerId" : "bunlo9vk5f",
    "firstname" : "Ado",
    "lastname" : "Kinnett",
...
```

### Customer Management Backend
Curl command: `curl http://localhost:8100/customers?limit=1`

Expected result:

```json
{
  "filter" : "",
  "limit" : 1,
  "offset" : 0,
  "size" : 50,
  "customers" : [ {
    "customerId" : "bunlo9vk5f",
    "firstname" : "Ado",
    "lastname" : "Kinnett",
    "birthday" : "1975-06-13T22:00:00.000+0000",
    "streetAddress" : "2 Autumn Leaf Lane",
    "postalCode" : "6500",
    "city" : "Bellinzona",
    "email" : "akinnetta@example.com",
    "phoneNumber" : "055 222 4111",
    "moveHistory" : [ ]
  } ]
}
```

### Customer Self-Service Backend
Curl command: `curl --header 'Authorization: Bearer b318ad736c6c844b' http://localhost:8080/cities/8640`

Expected result:

```json
{
  "cities" : [ "Hurden", "Rapperswil SG" ]
}
```

### Policy Management Backend
Curl command: `curl --header 'Authorization: Bearer b318ad736c6c844b' http://localhost:8090/policies`

Expected result:

```json
{
  "limit" : 10,
  "offset" : 0,
  "size" : 1,
  "policies" : [ {
    "policyId" : "fvo5pkqerr",
    "customer" : "rgpp0wkpec",
    "creationDate" : "2020-09-04T08:53:03.431+0000",
    "policyPeriod" : {
      "startDate" : "2018-02-04T23:00:00.000+0000",
      "endDate" : "2018-02-09T23:00:00.000+0000"
    },
...
```

All Spring Boot backends also offer an `/actuator/health` endpoint that returns whether the service is UP or not:

```
curl http://localhost:8110/actuator/health 

{
  "status" : "UP"
}                         
```

## I'm getting a Connection refused: connect exception on startup 

Don't worry if you're getting an exception about a refused connection on startup:

```
2018-11-16 13:31:08.492 WARN 1592 --- [gistrationTask1] d.c.b.a.c.r.ApplicationRegistrator : Failed to register application as Application(name=Customer Self-Service Backend, managementUrl=http://localhost:8080/actuator, healthUrl=http://localhost:8080/actuator/health, serviceUrl=http://localhost:8080/, metadata={startup=2018-11-16T13:31:02.779+01:00}) at spring-boot-admin ([http://localhost:9000/instances]): I/O error on POST request for "http://localhost:9000/instances": Connection refused: connect; nested exception is java.net.ConnectException: Connection refused: connect. Further attempts are logged on DEBUG level
```

This just means that the application was unable to connect to the [Spring Boot Admin](spring-boot-admin) application. If you haven't started the Spring Boot Admin, the warning can be safely ignored.

## I'm getting a file not found: failure to deploy on docker builds

````bash
/bin/sh: 1: ./mvnw: not found
Error response from daemon: The command '/bin/sh -c ./mvnw -B dependency:go-offline' returned a non-zero code: 127
Failed to deploy 'lakesidemutual/customer-core Dockerfile: customer-core/Dockerfile': Can't retrieve image ID from build stream
````

This issue occurs when the mvnw file does not have the correct encoding.
Each operating system has its own specific file encoding.

- Windows: CRLF (`\r\n`)
- Linux and macOS: LF (`\n`)

While Linux and macOS commonly work with LF encodings, Windows utilizes CRLF encodings.
Typically, Docker containers rely on Linux-based images and thus require files to be encoded with LF.
In case of additional problems, try to change the file encoding according to the tool you utilize and to your needs.
Be aware, this issue could also occur on other occasions such as executing the run_all_applications scripts.

In order to resolve potential future problems on Windows, change your personal git repository settings as follows.
```bash
git config core.autocrlf true
```
Setting core.autocrlf to true will automatically convert the file encodings to CRLF on a checkout and convert them back to LF on a commit.
This ensures that you will have all files encoded according to your operating system, while maintaining a consistent LF encoding on the repository.

Since Linux and macOS work with LF encodings, it is recommended to use the following setting on those two operating systems.
```bash
git config core.autocrlf input
```
Setting core.autocrlf to input will stop files with CRLF encoding to be pushed to the repository in the first place.
All file encodings will be set to LF on a git commit.

## Why aren't you using Lombok?
The DTO (data transfer object) classes require a lot of repetitive code (e.g., getters, setters, code to map between entities and DTOs, etc).
We could use a code generator like [Lombok](https://projectlombok.org/) to get rid of this boilerplate. However, we decided against using a tool like this, because they usually require additional IDE plug-ins which complicates the initial setup process.
