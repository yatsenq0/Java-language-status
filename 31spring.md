## Глава 31: Пример сервиса со Spring
### 31.1. Что за сервис мы напишем
Современные вакансии требуют знаний не только Java, но и экосистемы библиотек и фреймворков. Java обычно используется совместно со Spring Framework, а также требует знаний Maven или Gradle, баз данных, Spring Data, Hibernate и протокола HTTP. В данной главе мы создадим сервис для размещения существ на карте мира с четырьмя основными операциями:
- Размещение существа на карте в заданных координатах.
- Обновление координат и параметров существа по ID.
- Получение информации о существе по ID.
- Удаление существа по ID.
Мы будем использовать Spring вместе со Spring WebFlux, а данные будут храниться в базе данных H2.
### 31.2. Spring Initializr
Скелет приложения создается с помощью сервиса Spring Initializr на сайте [start.spring.io](start.spring.io). Он генерирует проект в соответствии с вашими настройками, который можно импортировать в IDE. Инструкции по использованию доступны на [urvanov.ru](urvanov.ru).
Для создания нового проекта в IntelliJ IDEA:
1. Выберите **File > New > Project...**
2. Если у вас многомодульный проект, выберите **File > New > Module...**
3. Выберите тип проекта **Spring Initializr** и заполните поля:
- **Name**: название проекта.
- **Location**: каталог для файлов проекта.
- **Language**: Java.
- **Type**: Maven.
- **Group**: имя группы Maven-проекта.
- **Artifact**: название артефакта.
- **Package name**: имя пакета.
- **Project SDK**: версия 17.
- **Packaging**: jar.
После заполнения полей нажмите **Next**, выберите зависимости:
- Spring Reactive Web
- Spring Data R2DBC
- H2 Database
- Validation
Нажмите **Finish**.
### 31.3. Разбор сгенерированного скелета приложения
Обратите внимание на файлы:
- `.mvn/wrapper/maven-wrapper.jar`
- `.mvn/wrapper/maven-wrapper.properties`
- `mvnw`
- `mvnw.cmd`
Эти файлы позволяют запускать Maven без его установки в систему. Для сборки и запуска проекта используйте команды:
- Windows:
```bash
mvnw.cmd clean install
mvnw.cmd spring-boot:run
```
- Linux:
```bash
./mvnw clean install
./mvnw spring-boot:run
```
Запускайте класс `CreaturesApplication` как обычную программу. Если вы используете IntelliJ IDEA Ultimate Edition, создайте конфигурацию запуска типа Spring Boot.
### Основные файлы
Файл `pom.xml` содержит зависимости, а `CreaturesApplication.java` выглядит следующим образом:
```java
package ru.urvanov.javaindynamics2022.creatures;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class CreaturesApplication {
public static void main(String[] args) {
SpringApplication.run(CreaturesApplication.class, args);
}
}
```
Минимальное приложение на Spring Boot просто и эффективно, особенно по сравнению с предыдущими версиями, требующими сложной конфигурации.
Похоже, что у вас есть базовая структура приложения на Spring Boot с использованием реактивного программирования через Reactor. Однако реализация сервиса `CreatureServiceImpl` пока не завершена и возвращает пустые значения. Чтобы сделать приложение функциональным, вам нужно реализовать логику для взаимодействия с хранилищем данных (например, базой данных) через репозиторий.

### Шаги для завершения реализации:

1. **Создание репозитория**:
   - Создайте интерфейс `CreatureRepository`, который будет содержать методы для доступа к данным.
   - Реализуйте этот интерфейс с помощью Spring Data JPA или другого подходящего фреймворка.

2. **Реализация CreatureServiceImpl**:
   - Внедряйте экземпляр репозитория в сервис.
   - Используйте методы репозитория для создания, обновления, получения и удаления существ.

3. **Конфигурация базы данных**:
   - Настройте подключение к базе данных в файле `application.properties` или `application.yml`.

### Пример реализации

#### CreatureRepository.java
```java
package ru.urvanov.javaindynamics2022.creatures.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import ru.urvanov.javaindynamics2022.creatures.domain.Creature;

public interface CreatureRepository extends JpaRepository<Creature, Long> {
}
```

#### Обновленная версия CreatureServiceImpl.java
```java
package ru.urvanov.javaindynamics2022.creatures.service;

import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;
import ru.urvanov.javaindynamics2022.creatures.domain.Creature;
import ru.urvanov.javaindynamics2022.creatures.repository.CreatureRepository;

@Service
public class CreatureServiceImpl implements CreatureService {

    private final CreatureRepository repository;

    public CreatureServiceImpl(CreatureRepository repository) {
        this.repository = repository;
    }

    @Override
    public Mono<Long> create(Creature creature) {
        return Mono.just(repository.save(creature).getId());
    }

    @Override
    public Mono<Creature> update(Creature creature) {
        return Mono.just(repository.save(creature));
    }

    @Override
    public Mono<Creature> getById(Long creatureId) {
        return Mono.justOrEmpty(repository.findById(creatureId));
    }

    @Override
    public Mono<Void> delete(Long creatureId) {
        repository.deleteById(creatureId);
        return Mono.empty();
    }
}
```

Эта реализация предполагает использование Spring Data JPA для работы с БД и автоматического сохранения/обновления объектов через репозиторий.

### Дополнительные советы

- Убедитесь, что вы добавили необходимые зависимости в файл `pom.xml` (если используете Maven):
  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <!-- Зависимость от БД -->
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
  </dependency>

  <!-- Для реактивного программирования -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
  </dependency>

  ```

- Если вы используете Gradle (`build.gradle`), то аналогично добавьте зависимости:
  
```groovy  
dependencies {
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
implementation 'org.springframework.boot:spring-boot-starter-webflux'
}
```
  
- Настройте подключение к БД в файле конфигурации (`application.properties` или `application.yml`). Например:

```properties  
spring.datasource.url=jdbc:h2:mem:testdb  
spring.datasource.driverClassName=org.h2.Driver  
spring.datasource.username=sa  
spring.datasource.password=  

# Конфигурация Hibernate/JPA 
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect 
```
Или аналогично на YAML:

```yml   
datasource:   
url: jdbc:h2:mem:testdb   
driver-class-name: org.h2.Driver   
username: sa   
password:   

jpa:
database-platform: org.hibernate.dialect.H2Dialect    
show-sql=true    
hibernate ddl-auto:update     
properties.hibernate.format_sql=true      
```
Это позволит вам использовать HSQLDB как временную БД во время разработки.

Похоже, что у вас есть базовая структура приложения на Spring Boot с использованием реактивного программирования через Reactor. Однако реализация сервиса `CreatureServiceImpl` пока не завершена и возвращает пустые значения. Чтобы сделать приложение функциональным, вам нужно реализовать логику для взаимодействия с хранилищем данных (например, базой данных) через репозиторий.

### Шаги для завершения реализации:

1. **Создание репозитория**:
   - Создайте интерфейс `CreatureRepository`, который будет содержать методы для доступа к данным.
   - Реализуйте этот интерфейс с помощью Spring Data JPA или другого подходящего фреймворка.

2. **Реализация CreatureServiceImpl**:
   - Внедряйте экземпляр репозитория в сервис.
   - Используйте методы репозитория для создания, обновления, получения и удаления существ.

3. **Конфигурация базы данных**:
   - Настройте подключение к базе данных в файле `application.properties` или `application.yml`.

### Пример реализации

#### CreatureRepository.java
```java
package ru.urvanov.javaindynamics2022.creatures.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import ru.urvanov.javaindynamics2022.creatures.domain.Creature;

public interface CreatureRepository extends JpaRepository<Creature, Long> {
}
```

#### Обновленная версия CreatureServiceImpl.java
```java
package ru.urvanov.javaindynamics2022.creatures.service;

import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;
import ru.urvanov.javaindynamics2022.creatures.domain.Creature;
import ru.urvanov.javaindynamics2022.creatures.repository.CreatureRepository;

@Service
public class CreatureServiceImpl implements CreatureService {

    private final CreatureRepository repository;

    public CreatureServiceImpl(CreatureRepository repository) {
        this.repository = repository;
    }

    @Override
    public Mono<Long> create(Creature creature) {
        return Mono.just(repository.save(creature).getId());
    }

    @Override
    public Mono<Creature> update(Creature creature) {
        return Mono.just(repository.save(creature));
    }

    @Override
    public Mono<Creature> getById(Long creatureId) {
        return Mono.justOrEmpty(repository.findById(creatureId));
    }

    @Override
    public Mono<Void> delete(Long creatureId) {
        repository.deleteById(creatureId);
        return Mono.empty();
    }
}
```

Эта реализация предполагает использование Spring Data JPA для работы с БД и автоматического сохранения/обновления объектов через репозиторий.

### Дополнительные советы

- Убедитесь, что вы добавили необходимые зависимости в файл `pom.xml` (если используете Maven):
  ```xml  
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <!-- Зависимость от БД -->
  <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
  </dependency>

  <!-- Для реактивного программирования -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
  </dependency>

```
  
- Если вы используете Gradle (`build.gradle`), то аналогично добавьте зависимости:

```groovy  
dependencies {
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'com.h2database:h2'
implementation 'org.springframework.boot:spring-boot-starter-webflux'
}
```
  
- Настройте подключение к БД в файле конфигурации (`application.properties` или `application.yml`). Например:

```properties  
spring.datasource.url=jdbc:h2:mem:testdb  
spring.datasource.driverClassName=org.h2.Driver  
spring.datasource.username=sa  
spring.datasource.password=  

# Конфигурация Hibernate/JPA 
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect 
```
Или аналогично на YAML:

```yml   
datasource:   
url: jdbc:h2:mem:testdb   
driver-class-name: org.h2.Driver   
username: sa   
password:   

jpa:
database-platform: org.hibernate.dialect.H2Dialect    
show-sql=true    
hibernate ddl-auto:update     
properties.hibernate.format_sql=true      
```
Это позволит вам использовать HSQLDB как временную БД во время разработки.

Эти шаги помогут вам завершить основную функциональность вашего приложения по работе со существами (Creatures).

Если у вас возникнут вопросы по конкретным моментам конфигурации или дополнительной функциональности (например, контроллеры), не стесняйтесь задавать их!

В вашем коде используется аннотация `@RestController`, которая является специализированной версией `@Controller`. Она объединяет в себе функциональность `@Controller` и `@ResponseBody`, что позволяет автоматически сериализовать возвращаемые объекты в формате JSON или XML без необходимости дополнительной аннотации каждого метода[1][2][5].

### Основные различия между `@Controller` и `@RestController`

- **Цель использования**:
  - **`@Controller`**: Используется для классических контроллеров Spring MVC, которые могут обрабатывать запросы и возвращать представления (например, HTML страницы).
  - **`@RestController`**: Предназначен для создания RESTful API. Он автоматически сериализует данные из методов контроллера в форматы JSON или XML.

- **Сериализация ответа**:
  - При использовании `@Controller`, если вы хотите вернуть данные напрямую (не представление), необходимо использовать аннотацию `@ResponseBody`.
  - С помощью `@RestController`, эта аннотация уже включена, поэтому все методы контроллера автоматически сериализуют свои результаты[1][2].

### Пример использования

В вашем коде (`CreaturesController.java`) уже используется аннотация `@RestController`. Это означает, что каждый метод этого контроллера будет автоматически сериализовать свои результаты:

```java
@RestController
@RequestMapping("/creatures")
public class CreaturesController {
    
    @PostMapping
    public Mono<Long> create(@RequestBody Creature creature) {
        // ...
    }
    
    @PutMapping("/{creatureId}")
    public Mono<Void> update(@PathVariable Long creatureId, @RequestBody Creature creature) {
        // ...
    }
    
    @GetMapping("/{creatureId}")
    public Mono<Creature> info(@PathVariable Long creatureId) {
        // ...
    }
    
    @DeleteMapping("/{creatureId}")
    public Mono<Void> delete(@PathVariable Long creatureId) {
        // ...
    }
}
```

Обратите внимание на исправление путей маппинга (`"/creatures"` вместо просто `"creatures"`), а также на использование правильных HTTP-методов (`PUTMapping`, `GetMapping`, etc.) для соответствующих операций[3]. 

Также важно правильно указывать пути к ресурсам с помощью `{}` для переменных пути:

```java
@GetMapping("/{creatureId}")
public Mono<Creature> info(@PathVariable Long creatureId) { ... }

// Аналогично для других методов...
```

Эта реализация обеспечивает полноценную поддержку CRUD-операций через REST API.

