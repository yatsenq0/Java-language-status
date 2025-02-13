 ## ГЛАВА 16 Optional

### 16.1. Теория

В Java 8 появился полезный класс `java.util.Optional`. Он используется для избавления от большого числа проверок значений на `null`, которые затрудняют чтение кода.

Пусть, например, у нас есть класс:

```java
package ru.urvanov.javaindynamics2022.optional;

import java.util.Optional;

class Satyr {
    public void myMethod() {
    }

    public Integer getSomeValue() {
        return 2;
    }
}
```

Пример кода с экземпляром этого класса и проверкой на `null`:

```java
public static Integer testMethod(Satyr myClass) {
    if (myClass != null) {
        myClass.myMethod();
        return myClass.getSomeValue();
    }
    return null;
}

public static void main(String[] args) {
    Satyr myClass = new Satyr();
    Integer result1 = testMethod(myClass);
}
```

Аналогичный код с использованием модного класса `Optional`:

```java
public static Integer testMethodWithOptional(Optional<Satyr> myClass) {
    myClass.ifPresent(Satyr::myMethod);
    return myClass.map(Satyr::getSomeValue).orElse(null);
}

public static void main(String[] args) {
    Satyr myClass = new Satyr();
    int result2 = testMethodWithOptional(Optional.ofNullable(myClass));
}
```

Как видно из этого примера, код метода `testMethodWithOptional` стал на две строки короче, чем код `testMethod`. Если в исходном примере было бы еще больше проверок на `null`, то сокращение размера кода было бы гораздо заметнее.

Давайте теперь разберемся, что же тут происходит. Сначала обратите внимание на вызов `Optional.ofNullable(myClass)`. Этот вызов статического метода создает экземпляр класса `Optional`. Созданный экземпляр выступает в качестве контейнера, который может либо содержать, либо не содержать не `null` значение. Для проверки того, содержит ли контейнер значение, нужно использовать метод `isPresent()`, который возвращает `true`, если контейнер содержит значение. В примере выше мы его не использовали, но метод весьма полезен.

В дальнейшем вся работа происходит через методы этого контейнера. Метод `ifPresent` позволяет вызвать какой-либо код, использующий объект из контейнера, но этот код будет выполняться только в том случае, если контейнер содержит значение. В данном случае мы передаем туда ссылку на метод `myMethod`.

Если же нам нужно вызвать какой-то код, результатом которого нам нужно воспользоваться в дальнейшем, то нам следует использовать метод `map`, который возвращает экземпляр `Optional`, содержащий возвращенное значение, если оно **НЕ** `null`. В примере выше внутри метода `map` мы вызываем наш метод `getSomeValue` у класса `Satyr`.

Но из нашего метода `testMethodWithOptional` нам нужно вернуть не сам контейнер, а хранящееся в нем значение или `null`, если значения нет. Для этого мы вызываем метод `orElse`, который возвращает хранящееся в контейнере `Optional` значение, если оно есть, либо значение, которое мы передадим в качестве аргумента метода. В данном случае в качестве аргумента мы передаем `null`.

Сокращение размера кода и проверок на `null` становится очень сильно заметно, если нам нужно подряд вызвать сначала один метод объекта, затем, если вернулось **НЕ** `null`, вызвать другой метод над возвращенным объектом и т. д. Пример:

`OptionalExample1.java`

```java
package ru.urvanov.javaindynamics2022.optional;

import java.util.Arrays;
import java.util.Optional;

public class OptionalExample1 {
    static class A {
        public B getB() {
            return new B();
        }
    }

    static class B {
        public C getC() {
            return new C();
        }
    }

    static class C {
        public D getD() {
            return new D("from C");
        }
    }

    static class D {
        private String value;

        public D(String value) {
            this.value = value;
        }

        @Override
        public String toString() {
            return "D [value=" + value + "]";
        }
    }

    static class E {
    }

    public static void main(String[] args) {
        D d = Optional.ofNullable(new A()).map(A::getB)
                .map(b -> b.getC()).map(C::getD)
                .orElse(new D("from orElse"));
        System.out.println("d = " + d);
    }
}
```

Попробуйте поиграть с кодом выше. Например, поправить код класса `C` или `B`, что-бы они возвращали `null`, тогда вы увидите, что экземпляр класса `D` будет содержать строку `"from orElse"`, т. к. он создан в методе `orElse`, где ему в конструктор передалось именно это значение. В текущем же варианте кода экземпляр класса `D` будет содержать строку `"from C"`.

Если метод вашего класса уже возвращает экземпляр `Optional`, то использование метода `map` приведет к тому, что в результирующем значении будет `Optional` внутри другого `Optional`. Для того чтобы этого избежать, используйте метод `flatMap`:

`OptionalExample2.java`

```java
package ru.urvanov.javaindynamics2022.optional;

import java.util.Optional;

public class OptionalExample2 {

    static class MyClass {
        public Optional<Integer> getSomeValue() {
            return Optional.ofNullable(2);
        }
    }

    public static void main(String[] args) {
        Integer result = Optional.ofNullable(new MyClass())
                .flatMap(MyClass::getSomeValue)
                .orElse(Integer.valueOf(-1));
    }
}
```

Если же мы вызываем цепочкой какие-то методы, и на самом последнем этапе нам нужно либо вернуть значение, либо бросить какое-то конкретное исключение, если значения нет, то нужно использовать метод `orElseThrow`:

`OptionalExample3.java`

```java
package ru.urvanov.javaindynamics2022.optional;

import java.util.Optional;

public class OptionalExample3 {

    static class MyClass {
        public Optional<Integer> getSomeValue() {
            return Optional.ofNullable(2);
        }
    }

    static class NoResultValueException extends Exception {
    }

    public static void main(String[] args)
            throws NoResultValueException {
        Integer result = Optional.ofNullable(new MyClass())
                .flatMap(MyClass::getSomeValue)
                .orElseThrow(NoResultValueException::new);
    }
}
```

Обратите внимание, что существуют два похожих метода создания экземпляра `Optional`:

```java
public static <T> Optional<T> of(T value)
```

а также

```java
public static <T> Optional<T> ofNullable(T value)
```

Какая между ними разница? Разница в том, что `Optional.of` бросит исключение `NullPointerException`, если ему передать значение `null` в качестве параметра. `Optional.ofNullable` вернет `Optional`, не содержащий значение, если ему передать `null`.

Примеры:

```java
Integer value = null;
Optional<Integer> optional1 = Optional.of(value); // throws
// NullPointerException
Integer value = null;
Optional<Integer> optional1 = Optional.ofNullable(value); // returns
// Optional with
// empty value
```

В каких случаях нам нужно использовать `Optional`? Этот класс появился только в Java 8, до этого мы много лет жили без него.

В JavaDoc-ах к `Optional` точно сказано, что класс `Optional` создан в первую очередь для использования в качестве возвращаемого значения для методов, которые могут вернуть `null`.

```java
Optional<MyEntity> findById(Long id); // OK.
```

Не стоит принимать `Optional` в качестве параметра. Вот так не нужно:

```java
void myMethod(Optional<MyObject> arg0); // Плохо!
```

Почему не нужно принимать его в качестве параметра? Все очень просто. Если мы будем принимать `Optional` в качестве параметра метода, то нам придется вставлять в метод дополнительные проверки. Вместо одной проверки на `null` нам придется проверять на `null` и пустой `Optional` без значения.

И уж тем более ни в коем случае не стоит использовать `Optional` в качестве поля класса. В этом вообще нет никакого смысла, а также это может привести к проблемам в будущем, т. к. класс `Optional` несериализуемый (см. главу 21 "Сериализация")

