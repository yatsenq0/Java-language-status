и т. д. 
     ГЛАВА 31  Пример сервиса со Spring 31.1. Что за сервис мы напишем Если вы посмотрите современные вакансии, то увидите, что знания только Java без библиотек и фреймворков не особо полезны. Никто не пишет приложения на чис-той Java. Для того чтобы найти работу, нужно знать о целой экосистеме библиотек, утилит, фреймворков и подходов к разработке. Java в большинстве случаев исполь-зуется совместно со Spring Framework (хотя есть и другие фреймворки, но на теку-щий момент они не так популярны). Также нужно знать Maven либо Gradle, какую-нибудь базу данных, Spring Data, Hibernate, иметь общее представление о протоко-ле HTTP и много чего еще. 
Представьте, что вы работаете в фирме, которая создает конструктор виртуального   ВАЖНО Информация в этой главе может служить отправной точкой в изучении экосистемы Java, но никак не полноценным источником информации о том, что нужно делать и как. Сама Java и мир созданных для этого языка библиотек и фреймворков очень об-ширны и никоим образом не могут быть уложены в одну книгу или в какие-нибудь жесткие рамки. В процессе чтения этой главы рекомендуется обращаться к офици-альной документации: https://spring.io, https://junit.org, https://maven.apache.org, https://gradle.org, https://projectreactor.io и др., которая вам может потребоваться. 
В этом разделе мы попытаемся создать небольшое приложение в качестве примера. 
Глава 31. Пример сервиса со Spring 343 мира. Мы создадим сервис по размещению существ на карте мира. Для упрощения наш функционал будет состоять из четырех операций: 
? Размещение существа на карте в заданных координатах с заданными парамет- рами. 
? Обновление координат и параметров существа по ID. 
? Получение информации о существе по ID. 
? Удаление существа по ID. 
Сам Spring Framework очень большой, и даже внутри него существует множество разных подходов к разработке приложений. В рамках нашего примера мы будем использовать Spring вместе со Spring WebFlux. Сама информация о размещенных в мире существах будет храниться в базе данных H2 в памяти приложения. 
31.2. Spring Initializr Скелет приложения создается с помощью сервиса Spring Initializr. Можно восполь-зоваться веб-версией https://start.spring.io. Она прекрасно сгенерирует скелет при-ложения в соответствии с вашими настройками, а затем этот сгенерированный ске-лет можно будет импортировать в вашу IDE. 
На сайте https://urvanov.ru можно найти инструкцию по использованию Spring Initializr для Eclipse, NetBeans, IntelliJ IDEA с цветными скриншотами. 
Аналогично можно использовать Spring Initializr и из самой IntelliJ IDEA, но это поддерживается только в Ultimate версии IDEA, в версии Community такой воз-можности по умолчанию нет, но можно установить плагин стороннего разработчи-ка, если есть желание. Плагин стороннего разработчика выглядит очень похоже. 
Далее используется Ultimate версия IntelliJ IDEA. Да, это платная версия, но вам, скорее всего, выдадут на нее лицензию на работе, так что лично вам платить не придется. 
Создаем новый проект на Spring (рис. 31.1). Для этого в Ultimate версии IntelliJ IDEA кликаем на File \ New \ Project... 
Если вы используете многомодульный проект, как этой книге, то нужно создавать новый модуль внутри уже существующего проекта, кликнув на File\New\Module... 
(рис. 31.2) В открывшемся диалоговом окне выберите тип проекта Spring Initializr и заполни-те значения, как на изображении на рис. 31.3. 
? Name — название проекта. 
? Location — каталог на диске, где будут храниться файлы проекта. 
? Language — язык проекта. Нужно выбрать Java. 
? Type — тип менеджера зависимостей. Чаще всего используется Maven, его и нужно выбрать. 
344 Глава 31. Пример сервиса со Spring  Рис. 31.1. Создание нового проекта в IntelliJ IDEA  Рис. 31.2. Создание нового модуля в IntelliJ IDEA ? Group — имя группы Maven-проекта. 
? Artifact — название артефакта. Конечный JAR-файл будет иметь название <artifact>-<версия>.jar. 
? Package name — имя пакета, которое будет создано для проекта. 
? Project SDK — выберите 17-ю версию, по которой и написана эта книга. 
? Java — выберите 17. 
Глава 31. Пример сервиса со Spring 345  Рис. 31.3. Создание нового проекта Spring Boot с помощью Spring Initializr в IntelliJ IDEA ? Packaging — выберите jar. Раньше приложения Spring деплоились внутри кон-тейнеров сервлетов наподобие Apache Tomcat, Wildfly, WebSphere и аналогич-ных. В одном экземпляре контейнера сервлетов деплоилось несколько веб-приложений (несколько war-файлов). В большинстве современных проектов ис-пользуется Spring Boot, где конечные jar уже содержат внутри себя контейнер сервлетов и сами настраивают ваше приложение на запуск. 
После заполнения полей кликните по кнопке Next. Откроется окно выбора зависи-мостей будущего проекта (рис. 31.4). Разные проекты будут использовать разные зависимости. Для нашего примера в левом дереве нужно проставить галочки на четырех пунктах: 
? Spring Reactive Web (мы будем писать проект в реактивном стиле с Spring Web Flux). 
? Spring Data R2DBC (для доступа к базе данных в реактивном стиле). 
? H2 Database (встроенная база данных). 
? Validation (для реализации функции проверки). 
Кликните Finish. 
346 Глава 31. Пример сервиса со Spring  Рис. 31.4. Выбор зависимостей нового проекта Spring Initializr в IntelliJ IDEA 31.3. Разбор сгенерированного скелета приложения Давайте посмотрим на то, что у нас получилось. 
Обратите внимание на следующие файлы: 
? .mvn/wrapper/maven-wrapper.jar; 
? .mvn/wrapper/maven-wrapper.properties; 
? mvnw; 
? mvnw.cmd. 
Вышеперечисленные файлы — это Maven Wrapper. Он позволяет запускать Maven, не устанавливая его в свою систему. Фактически это зафиксированная версия Maven, которая будет лежать в репозитории вместе с исходными кодами. Подоб-ный подход позволит избежать проблем, связанных с совместимостью различных версий сборщика Maven и необходимостью установки нескольких версий в вашу систему. 
Сборка и запуск проекта под Windows: 
mvnw.cmd clean install mvnw.cmd spring-boot:run Сборка и запуск проекта под Linux (у вас должна быть заполнена переменная окружения JAVA_HOME значением с путем к каталогу с JDK): 
Глава 31. Пример сервиса со Spring 347 ./mvnw clean install ./mvnw spring-boot:run Но это все с консоли. Мы же будем работать в основном в IDE, так что для нас Maven Wrapper не особо значим. 
На самом деле вы можете запускать на исполнение класс CreaturesApplication из проекта точно так же, как и программу HelloWorld. Однако, если у вас есть IntelliJ IDEA Ultimate Edition, лучше воспользоваться специальной конфигурацией запуска приложений Spring Boot. 
В правом верхнем углу IDEA найдите значок молотка. Справа от него будет ниспа-дающий список с существующими конфигурациями запуска и возможностью доба-вить новую конфигурацию (рис. 31.5). 
При создании новой конфигурации запуска нужно выбрать тип Spring Boot (рис. 31.6). 
 Рис. 31.5. Run Configurations в IntelliJ IDEA  Рис. 31.6. Создание конфигурации для запуска приложения Spring Boot в IntelliJ IDEA 348 Глава 31. Пример сервиса со Spring Откроется окно со специальными настройками, характерными для приложения Spring Boot (рис. 31.7). 
В этом окне можно задать активные профили, параметры JVM, параметры самого приложения, а также огромное количество других настроек. 
Для нас пока достаточно заполнить поля Name и Main class, как это сделано на рис. 31.7. 
 Рис. 31.7. Настройка конфигурации запуска приложения Spring Boot в IntelliJ IDEA Для запуска нужно использовать кнопку, находящуюся справа от ниспадающего списка, в котором мы создавали новую конфигурацию запуска (рис. 31.8). 
 Рис. 31.8. Запуск созданного сервиса из IntelliJ IDEA Пока наше приложение просто прослушивает порт 8080 и больше ничего не делает. 
Фактически у нас есть два важных файла: pom.xml и CreaturesApplication.java. Файл pom.xml содержит зависимости, которые автоматически добавились в наш проект в соответствии с тем, что мы указали в Spring Initializr. 
Мы пока не будем просматривать зависимости. Посмотрите на сгенерированный файл CreaturesApplication.java: 
Глава 31. Пример сервиса со Spring 349 CreaturesApplication.java package ru.urvanov.javaindynamics2022.creatures; 
 import org.springframework.boot.SpringApplication; 
import org.springframework.boot.autoconfigure.SpringBootApplication; 
 @SpringBootApplication public class CreaturesApplication {   public static void main(String[] args) {  SpringApplication.run(CreaturesApplication.class, args); 
 }  }  Как видите, минимальное приложение на Spring Boot выглядит довольно просто. 
Особенно если сравнить его с тем, как мы раньше инициализировали Spring Framework с кучей XML-конфигураций и файлов настроек. 
В качестве примера старой инициализации можете посмотреть пример приложения https://github.com/urvanov-ru/keystore. Особое внимание уделите файлам src/main/ webapp/META-INF/context.xml и src/main/webapp/WEB-INF/web.xml из проекта keystore. 
Именно так раньше и настраивался Spring Framework. 
31.4. Добавление конечных точек Для каждой из операций с существом мы создадим по одной конечной точке. Соз-дание существа в определенной точке будет происходить HTTP-методом POST, изменение его характеристик и координат — методом PUT, получение информации о существе — методом GET, а удаление существа будет происходить с помощью HTTP-метода DELETE. 
Фактически нам нужно создать класс с четырьмя методами. Классам, которые об-рабатывают HTTP-запросы, обычно дают название с суффиксом Controller. Пусть наш класс называется CreatureController, и пусть он располагается в пакете ru.urvanov.javaindynamics2022.creatures.controller: 
CreaturesController.java package ru.urvanov.javaindynamics2022.creatures.controller; 
 import org.springframework.web.bind.annotation.RequestMapping; 
import org.springframework.web.bind.annotation.RestController; 
 @RestController @RequestMapping("/creatures") public class CreaturesController { } 350 Глава 31. Пример сервиса со Spring Обратите внимание на аннотации @RestController и @RequestMapping. 
Аннотация @RestController на самом деле состоит из двух аннотаций: @Controller и @ResponseBody. 
@Controller указывает на то, что класс обрабатывает HTTP-запросы. 
@ResponseBody указывает, что результат, возвращенный методом, должен стать телом ответа на запрос. 
С помощью аннотации @RequestMapping мы указали, что методы нашего класса будут обрабатывать запросы к пути creatures. 
Теперь напишем наши четыре метода: 
CreaturesController.java package ru.urvanov.javaindynamics2022.creatures.controller; 
 import org.slf4j.Logger; 
import org.slf4j.LoggerFactory; 
import org.springframework.web.bind.annotation.*; 
 @RestController @RequestMapping("/creatures") public class CreaturesController {   private static Logger logger  = LoggerFactory.getLogger(CreaturesController.class); 
  @PostMapping  public void create() {  logger.debug("Создание существа"); 
 }   @PutMapping("{creatureId}")  public void update(@PathVariable Long creatureId) {  logger.debug("Обновление информации о существе"); 
 }   @GetMapping("{creatureId}")  public void info(@PathVariable Long creatureId) {  logger.debug("Получение информации о существе"); 
 }   @DeleteMapping("{creatureId}")  public void delete(@PathVariable Long creatureId) {  logger.debug("Удаление существа"); 
 } } Глава 31. Пример сервиса со Spring 351 Как видите, мы добавили четыре метода с соответствующими аннотациями: 
? @PostMapping обрабатывает HTTP POST; 
? @PutMapping обрабатывает HTTP PUT; 
? @GetMapping обрабатывает HTTP GET; 
? @DeleteMapping обрабатывает HTTP DELETE. 
Аннотация @PathVariable указывает, что значение переменной будет заполняться из параметров запроса. Например, для метода delete URL-адреса будут такими: 
creatures/1 creatures/34 creatures/99 Здесь 1, 34 и 99 — это значения, которые будут передаваться в параметр creatureId метода delete. 
Вы можете прямо сейчас попробовать запустить проект на исполнение. Он начнет слушать порт 8080. Попытайтесь в браузере открыть страницу: 
http://localhost:8080/creatures/3 Если вы поставите точку остановки на метод info, то увидите, что вы туда попадае-те, метод log.debug отрабатывает, но в консоли пока ничего не происходит, т. к. 
у нас по умолчанию уровень логирования info. 
ПРИМЕЧАНИЕ Исходные коды готового сервиса creatures выложены на сайте https://urvanov.ru. Если по какой-либо причине у вас не получается запустить сервис на текущем этапе, то вы можете скачать готовый и посмотреть, чего вам не хватает. 
А теперь обратите внимание на файл src/main/resources/application.properties. На теку-щий момент он пустой. В этом файле хранятся настройки, в том числе и настройки логирования. Мы можем включить уровень debug для нашего приложения, добавив туда строчку: 
logging.level.ru.urvanov.javaindynamics2022=debug После этого снова попытайтесь открыть страницу http://localhost:8080/creatures/3. 
В консоли будет выведено сообщение: "Получение информации о существе". 
Настройку уровня логирования не обязательно заполнять в файле application.properties. 
Можно определить ее для конфигурации запуска в IntelliJ IDEA, как это показано на рис. 31.9. 
31.5. Слой бизнес-сервисов Обычно контроллеры только вызывают методы из слоя бизнес-сервисов, которые, собственно, и занимаются основной логикой. 
Для нашего проекта достаточно одного сервиса. Логики особо никакой нет, нам нужно только сохранять данные в БД и вытаскивать из нее (так называемый CRUD — create, read, update, delete). 
352 Глава 31. Пример сервиса со Spring  Рис. 31.9. Настройка уровня логирования Поскольку мы уже дошли до слоя сервисов, то пора бы договориться о параметрах существ, которые мы будем хранить в БД. Класс "существо" будет иметь такой вид: 
Creature.java package ru.urvanov.javaindynamics2022.creatures.domain; 
 import java.util.Objects; 
 public class Creature {  private long id; 
 private double health; 
 private double x; 
 private double y; 
  public long getId() {  return id; 
 }   public void setId(long id) {  this.id = id; 
 } Глава 31. Пример сервиса со Spring 353  public double getHealth() {  return health; 
 }   public void setHealth(double health) {  this.health = health; 
 }   public double getX() {  return x; 
 }   public void setX(double x) {  this.x = x; 
 }   public double getY() {  return y; 
 }   public void setY(double y) {  this.y = y; 
 }   @Override  public String toString() {  return "Creature{" +  "id=" + id +  ", health=" + health +  ", x=" + x +  ", y=" + y +  '}'; 
 } }  Мы будем хранить идентификатор существа, уровень здоровья и координаты суще-ства в мире. Все методы в классе просто сгенерированы IDE. 
В идеале нам нужно бы было переопределить equals и hashCode таким образом, чтобы они использовали комбинацию уникальных полей из Creature. Но в нашем классе сейчас нет подходящих полей, к тому же мы не используем коллекции Set в нашем приложении, да и само приложение достаточно простое, поэтому проблем не возникнет. 
ВАЖНО Методы equals и hashCode для сущностей нужно переопределять. В реальном при-ложении нужно переопределять equals и hashCode для сущностей таким образом, чтобы они использовали только неизменяемые уникальные поля, например иденти-фикатор сотрудника. 
354 Глава 31. Пример сервиса со Spring Обратите внимание на наименование пакета, в котором хранится класс Creature. 
Мы храним его в подпакете domain. Обычно идет следующее разделение на пакеты: 
? controller — для хранения контроллеров, обрабатывающих HTTP-запросы; 
? service — для хранения классов, обрабатывающих бизнес-логику; 
? domain — для хранения сущностей предметной области; 
? repository или dao (data access objects) — для хранения классов, предназначен-ных для информации о состоянии сущностей. 
В бизнес-слое для нашего случая будет только один класс CreatureServiceImpl вместе с интерфейсом CreatureService, который мы создадим в подпакете service. 
CreatureService.java package ru.urvanov.javaindynamics2022.creatures.service; 
 import reactor.core.publisher.Mono; 
import ru.urvanov.javaindynamics2022.creatures.domain.Creature; 
 public interface CreatureService {  Mono<Long> create(Creature creature); 
  Mono<Creature> update(Creature creature); 
  Mono<Creature> getById(Long creatureId); 
  Mono<Void> delete(Long creatureId); 
}  Реализацию пока напишем самую простую: 
CreatureServiceImpl.java package ru.urvanov.javaindynamics2022.creatures.service; 
 import org.springframework.stereotype.Service; 
import reactor.core.publisher.Mono; 
import ru.urvanov.javaindynamics2022.creatures.domain.Creature; 
 @Service public class CreatureServiceImpl implements CreatureService {  @Override  public Mono<Long> create(Creature creature) {  return Mono.empty(); 
 }   @Override  public Mono<Creature> update(Creature creature) { Глава 31. Пример сервиса со Spring 355  return Mono.empty(); 
 }   @Override  public Mono<Creature> getById(Long creatureId) {  return Mono.empty(); 
 }   @Override  public Mono<Void> delete(Long creatureId) {  return Mono.empty(); 
 } }  В текущем варианте реализации методы пока ничего не делают. 
Особое внимание обратите на аннотацию @Service. Она означает, что класс нужно добавить как бин в контекст Spring. 
Для того чтобы разрабатывать приложения на Spring Framework, нужно иметь хотя бы общее представление о контексте Spring и его бинах. Мы не создаем сами экземпляры классов и не присваиваем сами значения зависимостям одного класса от другого. Вместо этого мы помещаем классы в контекст Spring и берем зависи- мости из него же. 
Наш основной класс CreaturesApplication находится в пакете ru.urvanov. 
javaindynamics2022.creatures. Он помечен аннотацией @SpringBootApplication, которая фактически включает в себя три аннотации: 
? @EnableAutoConfiguration включает автоматическую настройку Spring Boot, основываясь на jar-зависимостях, которые были добавлены. 
? @ComponentScan включает сканирование классов в пакете, в котором расположен класс, и в подпакетах. 
? @SpringBootConfiguration включает регистрацию дополнительных бинов в кон-тексте для дополнительных конфигурационных классов. 
Нам сейчас нужно обратить внимание на @ComponentScan. Он означает, что все клас-сы пакета ru.urvanov.javaindynamics2022.creatures, помеченные аннотацией @Component и ее производными, будут добавлены в контекст Spring-а. 
Аннотация @Service сама помечена как @Component. Фактически она делает абсо-лютно то же самое, что и @Component, и существует только для смыслового выделе-ния бизнес-сервисов. 
Аннотация @RestController помечена как @Controller, которая помечена как @Component, поэтому наш CreaturesController тоже попадет в контекст Spring Framework (рис. 31.10). 
Теперь нам нужно добавить в CreaturesController вызовы методов CreatureService. Для этого используется аннотация @Autowired, которая производит поиск подходящего по типу бина и внедряет его: 
356 Глава 31. Пример сервиса со Spring  Рис. 31.10. Бины в контексте Spring Framework CreaturesController.java package ru.urvanov.javaindynamics2022.creatures.controller; 
 import org.slf4j.Logger; 
import org.slf4j.LoggerFactory; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.web.bind.annotation.*; 
import reactor.core.publisher.Mono; 
import ru.urvanov.javaindynamics2022.creatures.domain.Creature; 
import ru.urvanov.javaindynamics2022.creatures.service.CreatureService; 
 @RestController @RequestMapping("/creatures") public class CreaturesController {   private static Logger logger  = LoggerFactory.getLogger(CreaturesController.class); 
   @Autowired  private CreatureService creatureService; 
  @PostMapping  public Mono<Long> create(@RequestBody Creature creature) {  logger.debug("Создание существа"); 
 return creatureService.create(creature); 
 }   @PutMapping("{creatureId}")  public Mono<Void> update(@PathVariable Long creatureId,  @RequestBody Creature creature) {  logger.debug("Обновление информации о существе"); 
 creature.setId(creatureId); 
Глава 31. Пример сервиса со Spring 357  creatureService.update(creature); 
 return Mono.empty(); 
 }   @GetMapping("{creatureId}")  public Mono<Creature> info(@PathVariable Long creatureId) {  logger.debug("Получение информации о существе"); 
 return creatureService.getById(creatureId); 
 }   @DeleteMapping("{creatureId}")  public Mono<Void> delete(@PathVariable Long creatureId) {  logger.debug("Удаление существа"); 
 return creatureService.delete(creatureId); 
 } }  Благодаря аннотации @Autowired в поле creatureService будет внедрен бин CreatureService. На самом деле это будет не сам класс CreatureServiceImpl, а бин, который реализует интерфейс CreatureService и в дополнение к своим действиям вызывает методы CreatureServiceImpl при вызове своих методов. 
31.6. Работа с базой данных Осталось доделать работу с базой данных. Мы используем Spring Data R2DBC с базой данных H2 Database, которая сохраняет данные в ОЗУ. В нашем приложе-нии обычные методы чтения, сохранения, изменения и удаления данных. Они реа-лизуются с помощью интерфейса ReactiveCrudRepository, от которого нам нужно унаследовать свой интерфейс: 
CreatureRepository.java package ru.urvanov.javaindynamics2022.creatures.repository; 
 import org.springframework.data.repository.reactive.ReactiveCrudRepository; 
import ru.urvanov.javaindynamics2022.creatures.domain.Creature; 
 public interface CreatureRepository  extends ReactiveCrudRepository<Creature, Long> { }  Когда при сканировании классов Spring обнаружит интерфейс, наследующийся от ReactiveCrudRepository, он создаст подходящий бин с реализацией базовых мето-дов из этого интерфейса. Нам же достаточно использовать CreatureRepository из CreatureServiceImpl: 
358 Глава 31. Пример сервиса со Spring CreatureServiceImpl.java package ru.urvanov.javaindynamics2022.creatures.service; 
 import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Service; 
import org.springframework.transaction.annotation.Transactional; 
import reactor.core.publisher.Mono; 
import ru.urvanov.javaindynamics2022.creatures.domain.Creature; 
import ru.urvanov.javaindynamics2022.creatures.repository.CreatureRepository; 
 @Service public class CreatureServiceImpl implements CreatureService {   @Autowired  private CreatureRepository creatureRepository; 
  @Override  @Transactional  public Mono<Long> create(Creature creature) {  return creatureRepository.save(creature).map(Creature::getId); 
 }   @Override  @Transactional  public Mono<Creature> update(Creature creature) {  return creatureRepository.save(creature); 
 }   @Override  @Transactional(readOnly = true)  public Mono<Creature> getById(Long creatureId) {  return creatureRepository.findById(creatureId); 
 }   @Override  @Transactional  public Mono<Void> delete(Long creatureId) {  return creatureRepository.deleteById(creatureId); 
 } }  Мы просто вызываем методы из CreatureRepository. Сама реализация остается на Spring Data R2DBC. 
Обратите внимание на аннотацию @Transactional. Она управляет транзакциями в нашем приложении. При вызове метода бина, помеченного этой аннотацией, Глава 31. Пример сервиса со Spring 359 будет автоматически начата новая транзакция либо будет использована текущая транзакция, если она есть. 
Следующим шагом нужно дать небольшие подсказки, чтобы поля класса Creature могли правильно мапиться на поля из H2 Database: 
Creature.java package ru.urvanov.javaindynamics2022.creatures.domain; 
 import org.springframework.boot.autoconfigure.domain.EntityScan; 
import org.springframework.data.annotation.Id; 
import org.springframework.data.relational.core.mapping.Table; 
 import javax.annotation.processing.Generated; 
import java.util.Objects; 
 @Table("creature") public class Creature {  @Id  private long id; 
 private double health; 
 private double x; 
 private double y; 
 ... 
 Мы добавили две аннотации: 
? @Table для указания наименования таблицы; 
? @Id для указания, что поле id является первичным ключом. 
Также нужно создать таблицу creature в базе данных. Hibernate может генерировать сами необходимые таблицы, но в реальных приложениях эту возможность не ис-пользуют. Скрипты создания разместим в src/main/resources/schema.sql: 
schema.sql CREATE TABLE creature (  id SERIAL PRIMARY KEY,  health DOUBLE PRECISION,  x DOUBLE PRECISION,  y DOUBLE PRECISION ); 
 INSERT INTO creature(health, x, y) VALUES(100.0, 10.2, -3.4); 
360 Глава 31. Пример сервиса со Spring INSERT INTO creature(health, x, y) VALUES(200.0, 50.2, -34.4); 
 INSERT INTO creature(health, x, y) VALUES(150.0, 3.2, 54.4); 
31.7. Вызов методов с Postman В этом разделе нам понадобится установить Postman с сайта https://www. 
postman.com. Для обучения хватает бесплатной версии. 
Дальше нам нужно проверить наши методы, вызвав их на выполнение. Внутри про-екта рядом с файлом pom.xml лежит файл creatures-postman.json. Импортируйте его содержимое в Postman через File ? Import. В коллекции запросов должен появить-ся пункт java-in-dynamics-2022-creatures как минимум с четырьмя запросами: get, create, update, delete (рис. 31.11). Каждый из них проверяет один из методов наше-го сервиса. 
 Рис. 31.11. Тестирование методов сервиса с Postman 31.8. Docker В большинстве случаев приложение Spring Boot упаковывается в какой-нибудь контейнер вместе со всеми своими зависимостями, это облегчает его развертывание и запуск. Одной из платформ для разработки и запуска контейнерных приложений является Docker. 
Docker состоит из следующих компонентов: 
? Docker daemon — это сервис, работающий в фоновой режиме, через который осуществляется все взаимодействие с контейнерами. 
? Docker client — интерфейс командной строки для отправки команд в Docker daemon. 
Глава 31. Пример сервиса со Spring 361 ? Docker image (образ) — неизменяемый образ, из которого разворачиваются кон-тейнеры, содержащие ОС и запускаемое приложение. 
? Docker container — работающее приложение, развернутое из образа. Его можно представлять как маленький виртуальный компьютер с какой-нибудь операци-онной системой (чаще всего Linux) и нашим приложением. 
? Docker registry (реестр) — репозиторий с образами. Есть публичные и приватные реестры. Распространенный публичный репозиторий https://hub.docker.com. 
? Dockerfile — инструкция для сборки образа. Простой текстовый файл, содержа-щий команды. 
Docker можно использовать и под Linux, и под Windows, и под MacOS. 
Для начала нам нужно установить и настроить Docker согласно инструкции на официальном сайте (https://docs.docker.com/engine/install/). Установка и настройка сильно отличаются в зависимости от вашей операционной системы. 
После установки Docker на нашем компьютере добавим поддержку создания Docker image в наш проект. В самом простом варианте нам достаточно добавить файл Dockerfile со следующим содержимым: 
Dockerfile FROM openjdk:17-alpine ARG JAR_FILE=target/*.jar COPY ${JAR_FILE} app.jar ENTRYPOINT ["java","-jar","/app.jar"]  После этого мы уже можем создавать Docker image и запускать контейнер на ис-полнение. 
Сначала давайте соберем наше приложение, чтобы у нас были итоговые файлы для запуска Docker, для этого нужно либо выполнить команду: 
./mvn package из каталога с исходными кодами, либо кликнуть на вкладке Maven в правой части IntelliJ IDEA и запустить цель package (рис. 31.12). 
Сборка docker image и простановка метки urvanovru/java-in-dynamics-2022-creatures. 
docker build -t urvanovru/java-in-dynamics-2022-creatures . 
Аналогичного результата можно добиться с помощью Spring Boot Maven Plugin: 
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=urvanovru/java-in-dynamics-2022-creatures Развертывание и запуск контейнера из docker image с меткой urvanovru/java-in-dynamics-2022 с пробрасыванием порта 8080 с компьютера хоста (на котором мы работаем) на развернутый контейнер: 
docker run -p 8080:8080 urvanovru/java-in-dynamics-2022-creatures 362 Глава 31. Пример сервиса со Spring  Рис. 31.12. Запуск mvn package из IntelliJ IDEA Сам Dockerfile содержит описание image: 
? FROM указывает базовый docker image, из которого будет строиться новый docker image с нашим приложением. В данном случае у нас будет дистрибутив Alpine Linux с предустановленной Java 17. 
? С помощью ARG объявляем переменную JAR_FILE, содержащую шаблон поиска собранного jar-файла нашего приложения из каталога target. 
? Команда COPY копирует jar-файл с нашим приложением в файл app.jar внутри нашего docker image. Этот содержит наше приложение, упакованное со всеми зависимостями. 
? ENTRYPOINT указывает команду, которая должна запускать наше приложение внутри развернутого контейнера (java –jar /app.jar). 
Безопаснее запускать приложение не от пользователя root, а от непривилегирован-ного пользователя, для чего нужно модифицировать Dockerfile: 
Dockerfile FROM openjdk:17-alpine RUN addgroup -S creatures && adduser -S creatures -G creatures USER creatures:creatures ARG JAR_FILE=target/*.jar COPY ${JAR_FILE} app.jar ENTRYPOINT ["java","-jar","/app.jar"] Глава 31. Пример сервиса со Spring 363 Если мы пересоберем Docker image и запустим его, то в консоли можно будет уви-деть, что запуск приложения был от пользователя creatures (/app.jar started by creatures): 
Starting CreaturesApplication v0.0.1-SNAPSHOT using Java 17-ea on f75ba7215d05 with PID 1 (/app.jar started by creatures in /) 31.9. Kubernetes Мы уже рассмотрели Docker, который облегчает перенос и развертывание прило-жения и необходимого ему окружения. Современное приложение обычно содержит большое количество сервисов, которые нужно не только развернуть, но и поддер-живать в рабочем состоянии, следить за работоспособностью, перезапускать и масштабировать при необходимости. 
Основные понятия в Kubernetes, которые нам понадобятся в этой главе: 
? Node — это физическая машина в кластере Kubernetes. 
? Pod — контейнер, можно рассматривать как один небольшой виртуальный ком-пьютер с установленной операционной системой, нашим приложением и необ-ходимыми зависимостями. 
? Persistence Volume — постоянное хранилище, которое могут использовать не-сколько pod-ов. 
? Service — абстракция, которая логически объединяет несколько pod-ов и доступ к ним. 
? ReplicaSet — абстракция, поддерживающая указанное количество копий pod-ов в рабочем состоянии, уничтожая и создавая новые при необходимости. 
? Deployment — описание желаемого состояния, которое Kubernetes будет под-держивать. 
? Kubectl — консольный клиент Kubernetes, с помощью которого мы будем им управлять. 
Minikube — это локальный Kubernetes, который обычно используется для разра-ботки и изучения. Именно его вам и нужно будет установить по инструкции для вашей операционной системы: 
https://minikube.sigs.k8s.io/docs/start После установки и настройки Minikube у вас должны работать команды его запуска и остановки: 
$ minikube start $ minikube stop А следующая команда должна отображать версию kubectl: 
$ kubectl version Для того чтобы наш проект мог существовать в Kubernetes, нам нужно добавить новую зависимость: 
364 Глава 31. Пример сервиса со Spring <dependency>  <groupId>org.springframework.boot</groupId>  <artifactId>spring-boot-starter-actuator</artifactId> </dependency> в файл pom.xml нашего сервиса creatures. 
Зависимость spring-boot-starter-actuator добавит новый endpoint/actuator/health в проект, благодаря которой Kubernetes сможет узнавать, что наш сервис в рабочем состоянии и его не надо перезапускать. Вызовите с помощью Postman метод GET для localhost:8080/actuator/health с запущенным проектом, и вы получите ответ: 
{  "status": "UP" } В главе про Docker мы уже собирали docker image. Но docker registry и Minikube хранят docker image в разных местах, которые никак не связаны. Minikube не смо-жет просто забрать docker image из прошлой статьи. Нам нужно настроить Docker так, чтобы он собирал свои docker image в Minikube: 
$ minikube docker-env export DOCKER_TLS_VERIFY="1" export DOCKER_HOST="tcp://192.168.49.2:2376" export DOCKER_CERT_PATH="/home/fedor/.minikube/certs" export MINIKUBE_ACTIVE_DOCKERD="minikube"  # To point your shell to minikube's docker-daemon, run: 
# eval $(minikube -p minikube docker-env) $ eval $(minikube -p minikube docker-env) НАСТРОЙКА DOCKER НА СБОРКУ DOCKER IMAGE В MINIKUBE Обратите внимание, что eval $(minikube -p minikube docker-env) нужно запус-кать в каждом запущенном терминале, в котором вы собираетесь собирать образы Docker. 
Сборка образа Docker с помощью плагина Maven от Spring (вызывать из каталога с проектом creatures): 
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=urvanovru/java-in-dynamics-2022-creatures А теперь давайте проверим, работает ли Minikube: 
$ kubectl get all NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE service/kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 12m Проверка подов в пространстве urvanovru (обычно каждый проект развертывается в свое пространство имен, а не в пространство имен по умолчанию): 
$ kubectl get pods --namespace urvanovru No resources found in urvanovru namespace. 
Пока никаких pod-ов нет. 
Глава 31. Пример сервиса со Spring 365 А теперь, собственно, займемся развертыванием приложения в Kubernetes. Для это-го нам нужно создать deployment. Консольная утилита kubectl (не забудьте сделать alias для него, если используете Minikube, как это указано в инструкции по уста-новке) сделает это за нас (команды вызвать из каталога с проектом creatures): 
$ kubectl create deployment java-in-dynamics-2022-creatures --image=urvanovru/java-in-dynamics-2022-creatures --dry-run=client -o=yaml > deployment.yaml $ echo --- >> deployment.yaml $ kubectl create service clusterip java-in-dynamics-2022-creatures --tcp=8080:8080 --dry-run=client -o=yaml >> deployment.yaml После выполнения этих команд создастся файл deployment.yaml. Откройте его в IDE и добавьте imagePullPolicy, как показано ниже: 
deployment.yaml ... 
 spec: 
 containers: 
 - image: urvanovru/java-in-dynamics-2022-creatures  name: java-in-dynamics-2022-creatures  imagePullPolicy: Never  resources: {} ... 
 Мы указали Never в ImagePullPolicy, чтобы Kubernetes не пытался выкачать docker image из центрального Docker Hub, так наш docker image существует только на на-шем компьютере. 
Для запуска проекта в Kubernetes нужно применить сгенерированный deployment.yaml следующей командой: 
$ kubectl apply -f deployment.yaml --namespace urvanovru Проверим, что проект запустился: 
$ kubectl get pods --namespace urvanovru Если что-то не получилось, то можно удалить deployment из Kubernetes: 
$ kubectl delete deployment java-in-dynamics-2022-creatures --namespace urvanovru А потом проверить, что делали неправильно, и заново создать его: 
$ kubectl apply -f deployment.yaml --namespace urvanovru Можно смотреть лог пода: 
$ kubectl get pods --namespace urvanovru NAME READY STATUS RESTARTS AGE java-in-dynamics-2022-creatures-56cb9d5444-7rvhk 1/1 Running 0 7m7s $ kubectl logs java-in-dynamics-2022-creatures-56cb9d5444-7rvhk --namespace  urvanovru  366 Глава 31. Пример сервиса со Spring Setting Active Processor Count to 12 WARNING: Unable to convert memory limit "max" from path "/sys/fs/cgroup/ memory.max" as int: memory size "max" does not match pattern "^([\\d]+)([kmgtKMGT]?)$" Calculating JVM memory based on 16104212K available memory Calculated JVM Memory Configuration: -XX:MaxDirectMemorySize=10M -Xmx15707271K -XX:MaxMetaspaceSize=89740K -XX:ReservedCodeCacheSize=240M -Xss1M (Total Memory: 16104212K, Thread Count: 50, Loaded Class Count: 13430, Headroom: 0%) ... 
Также в Minikube есть своя визуальная панель управления, доступная по команде: 
$ minikube dashboard После нее откроется браузер по умолчанию с запущенной административной пане-лью. Не забудьте там выбрать нужное пространство имен, как показано на рис. 31.13.