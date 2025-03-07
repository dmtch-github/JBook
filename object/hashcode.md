# java.lang.Object#hashCode

## Введение

Что такое `hash`? Это некоторое число, генерируемое на основе объекта и описывающее его состояние. Это число вычисляется на основе `hash`-функции.  
Метод `int hashCode()` является реализация `hash`-функции для `Java`-объекта. Возвращаемое число в `Java` считается `hash`-кодом объекта.

Объявление метода выглядит так:

```java
/**
 * Answers an integer hash code for the receiver. Any two 
 * objects which answer <code>true</code> when passed to 
 * <code>.equals</code> must answer the same value for this
 * method.
 *
 * @return	the receiver's hash.
 *
 * @see	#equals
 */
public int hashCode() {
	return J9VMInternals.fastIdentityHashCode(this);
}
```
> Возможно, вас напугала стрчока J9VMInternals.fastIdentityHashCode(this), но пока для простоты считайте, 
> что эта реализация зависит от `JVM`.

В `hash` - таблицах или ассоциативных маассивах (их еще называют словари, мапы) важность `hashCode`-а нельзя недооценивать.

Представим, что у нас есть массив пар ключ-значение.
Для поиска по ключу в таком массиве вам надо обойти весь массив, заглянуть в каждую ячейку и сравнить ключ.
Долго? Долго!

А теперь вспомним, что хэш - это по сути метаинформация о состоянии объекта, некоторое число.
И мы можем просто взять хэш, по нему понять в какой ячейке хранится искомая пара и просто взять ее оттуда, без перебора всего массива!
Быстро? Гораздо быстрее, чем просто итерация по всему массиву!

Да, могут быть коллизии - ситуации, когда разным объектам будет сгенерирован одинаковый хэш,
но решить коллизию можно через `equals` и это все еще быстрее, чем проходить весь массив.

По сути, главное преимущество в быстродействии ассоциативных массивов основано на работе с `hashCode`. 
Именно за счет хэша мы можем вставлять и получать данные за `O(1)`, то есть за время пропорциональное вычислению хэш-функции.

Если планируется использовать объекты в качестве ключа в ассоциативном массиве, то переопределять `hashCode` надо обязательно.

## Переопределение

### Требования

Еще раз посмотрим на метод:

```java
/**
 * Answers an integer hash code for the receiver. Any two 
 * objects which answer <code>true</code> when passed to 
 * <code>.equals</code> must answer the same value for this
 * method.
 *
 * @return	the receiver's hash.
 *
 * @see	#equals
 */
public int hashCode() {
	// ......
}
```

Контракт `hashCode` предъявляет следующие требования к реализации:

* Если вызвать метод `hashCode` на одном и том же объекте, состояние которого не меняли, то будет возвращено **одно и то же** значение.
* Если два объекта равны, то вызов `hashCode` для каждого **обязан** давать один и тот же результат.
    > Равенство объектов проверяется через вызов метода [equals](./equals.md).
* Если два объекта имеют один и тот же `hash`-код, то это не гарантирует равенства объектов.

Проще говоря, разный `hash`-код у двух объектов - это гарантия того, что объекты не равны, в то время как одинаковый `hash`-код не гарантирует равенства.

Cитуация, когда разные объекты имеют одинаковые `hash`-код, называется `collision` или коллизией.

> Логично, что коллизии возможны, так как размер `int` ограничен, да и реализации хэш-функции могут быть далеко не идеальны.

Реализация метода `hashCode` по-умолчанию возвращает разные значения `hash`-кодов для разных объектов:

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(new Person(4).hashCode());
        System.out.println(new Person(4).hashCode());
    }
}

class Person {
    private int age;

    public Person(int age) {
        this.age = age;
    }
}
```

Что это за значения? По-умолчанию `hashCode()` возвращает значение, которое называется идентификационный хэш (identity hash code).
В [документации](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#hashCode--) сказано:

> This is typically implemented by converting the internal address of the object into an integer, but this implementation technique is not required by the Java™ programming language.

Поэтому реализация зависит от `JVM`.

Более подробно об этом можно прочесть в разделе [полезные ссылки](#полезные-ссылки).

Пока можете не акцентировать внимание на том, как реализована `hash`-функция по-умолчанию и запомнить, что для разных объетов будут всегда возвращены разные значения `hash`-кодов, не взирая на состояние объекта.

Исходя из описания контракта метода можно понять, что `hashCode`, наряду с `equals`, играет важную роль в сравнении объектов.
По сути он показывает изменилось ли состояние объекта, которое мы используем в `equals` для сравнения.

Именно поэтому, если вы переопределяете `equals`, то вы **обязаны** переопределить `hashCode`.

## Переопределение hashCode

Самый очевидный и самый плохой пример как можно переопределить `hashCode` - это всегда возвращать константу:

```java
@Override
public int hashCode() {
    return 14;
}
```

Для равных объектов такой метод вернет одно и то же число, что удовлетворяет спецификации. 
Но, делать так **категорически** не рекомендуется.

Во-первых, если состояние объекта с такой реализацией `hashCode`, будет изменено, то это никак не отразится на `hashCode`. 
Что уже наталкивает на мысль, что такая реализация не совсем верна.

Во-вторых, такая реализация гарантирует возникновение коллизий **всегда**.
А значит, использовать такой подход - это лишить себя всех преимуществ, которые дает нам использование ассоциативных массивов.

Теперь, когда мы решили, что возвращать всегда одно и то же значение - это плохое решение, давайте поговорим о том, что нужно включать в рассчет `hashCode`.

Перво-наперво, необходимо исключить избыточные поля, которые не участвуют в `equals`.

Далее необходимо выбрать базу: число, которое будет основной вычисления `hash`-кода.

По историческим причинам обычно за базу берут число `31`.
Кто-то говорит, что это взято из-за близости к числу `32`, т.е степени двойки `2^5 - 1`.
Кто-то утверждает, что был проведен эксперимент и наиболее лучшей базой получились числа `31` и `33`, а `31` понравилось больше.

В целом, вы можете выбрать за базу что хотите, но обычно выбирают `31`.
Многие `IDE` (например, `IDEA`) генерят `hashCode` именно с такой базой.

Правила вычисления `hashCode`:

* Присваиваем переменной result ненулевое значение - базу.
* Далее для каждого значимого поля в объекте вычисляем `hashCode`, по следующим правилам:

    1. Если поле `boolean` - `(f ? 1 : 0)`

    2. Если поле `byte`, `char`, `short` или `int` - `(int) f`

    3. Если поле `long` - `(int)(f ^ (f >>> 32))`

    4. Если поле `float`, то `Float.floatToIntBits(f);`

    5. Если поле `double`, то `Double.doubleToLongBits(f)`, а затем как с `long`.

    6. Если поле это ссылка на другой объект, то рекурсивно вызовите `hashCode()`

    7. Если поле `null`, то возвращаем `0`.

    8. Если поле это массив, то обрабатываем так, как будто каждый элемент массива - это поле объекта.

* После каждого обработанного поля объединяем его `hashCode` с текущим значением:

    ```java
    result = 31 * result + c; // c - это hashCode обработанного поля.
    ```

* Возвращаем результат.

Пример приведем с помощью многострадального класса `Person`:

```java
public class Person {
    private int age;
    private int number;
    private double salary;
    private String name;
    private CarKey carKey;

    public Person(int age, int number, String name, double salary, CarKey carKey) {
        this.age = age;
        this.number = number;
        this.name = name;
        this.salary = salary;
        this.carKey = carKey;
    }

    @Override
    public int hashCode() {
      int result = 31;
      result = result * 17 + age;
      result = result * 17 + number;

      long lnum = Double.doubleToLongBits(number);
      result = result * 17 + (int)(lnum ^ (lnum >>> 32));

      result = result * 17 + name.hashCode();
      result = result * 17 + carKey.hashCode();

      return result;
    }

    // override equals
    // ...
}
```

Также, с версии `Java 8+` в классе `java.util.Objects` есть вспомогательные методы для генерации `hashCode`:

```java
@Override
public int hashCode() {
    return Objects.hash(age, number, salary, name, carKey);
}
```

## Использование hashCode в equals

Так как `hashCode` допускает возникновение коллизий, то использовать его в `equals` нет смысла.
Методы идут в паре, но использование одного в другом не желательно и бесмыссленно.

`hashCode` призван облегчить поиск объектов в каких-то структурах, например, ассоциативных массивах.
`equals` - это уже непосредственно про сравнение объектов.

## Классы-обертки над примитивами

Помните, что типы-обертки, которые по размеру меньше или равны `int`, возвращают в качестве `hashCode` свое значение.

Продемонстрируем:

```java
public class Test {
    public static void main(String[] args) {
        Integer a = 5000;
        System.out.println(a.hashCode()); // 5000
    }
}
```

Так как `Long` превосходит `int`, то, разумеется, возвращать свое значение в качестве `hashCode` он не может.
Вместо этого используется более сложный алгоритм, который работает с значением `long` (как в примере `Person`-а).

## Массивы

Помните, что хэш-код массива не зависит от хранимых в нем элементов, а присваивается при создании массива!

```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(arr.hashCode());

        // меняем элемент в массиве
        arr[0] = 100;
        // hashCode массива не изменяется
        System.out.println(arr.hashCode());
```

Результат:

```java
1580066828
1580066828
```

Решением является использовать статический метод для рассчета `hashCode` у массивов: `Arrays.hashCode(...)`:

```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(Arrays.hashCode(arr));

        arr[0] = 100;
        System.out.println(Arrays.hashCode(arr));
```

Результат:

```java
29615266
121043845
```

## Заключение

Хэш - это некоторое число, генерируемое на основе объекта и описывающее его состояние.
Метод `hashCode`, наряду с `equals`, играет важную роль в сравнении объектов.
По сути он показывает изменилось ли состояние объекта, которое мы используем в `equals` для сравнения.

Если планируется использовать объекты в качестве ключа в ассоциативном маассиве, то переопределять `hashCode` обязательно.
Если в классе переопределяется `equals`, то обязательно надо переопределять `hashCode` и наоборот.

Эти методы всегда должны определяться парой!

Плохая реализация `hash`-функции даст вам большое количество коллизий, что сведет на нет все преимущества использования ассоциативных массивов.

> Помните, что большинство IDE сейчас легко сгенерируют вам `hashCode`, чтобы вы не писали его вручную.
>
> Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).
> Существуют и сторонние библиотеки, помогающие в вычислении `hashCode`, например [apache commons](https://commons.apache.org/).

## Полезные ссылки

1. [Владимир Долженко — Внутрь VM сквозь замочную скважину hashCode](https://www.youtube.com/watch?v=GS2YqQ1DNJU)
2. [Как работает hashCode() по умолчанию?](https://habr.com/ru/company/mailru/blog/321306/)
3. [Разбираемся с hashCode() и equals()](https://habr.com/ru/post/168195/)
4. [What role does hashCode play when comparing two objects](https://stackoverflow.com/questions/29949305/what-role-does-hashcode-play-when-comparing-two-objects)
5. [Java equals() and hashCode() Contracts](https://www.baeldung.com/java-equals-hashcode-contracts)