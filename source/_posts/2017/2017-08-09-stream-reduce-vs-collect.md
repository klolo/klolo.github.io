---
layout: post
title: "Java stream reduce vs collect"
date: 2017-08-09 22:13:08 +0200
comments: true
categories: JAVA
---
Pracując z strumieniami Javy 8 pojawia się czasami konieczność złączenia wszystkich elementów strumienia w jakiś obiekt wynikowy. Taką operacje można wykonać zarówno za pomocą metody reduce, jak również za pomocą collect. Jaka jest różnica między tymi metodami i na jaki problem można się natknąć korzystając z tych metod?

<!--more-->
Zarówno metoda reduce jak i collect są operacjami terminalnymi strumienia. Co oznacza tyle że ich wynikiem nie jest strumień na którym można dalej wykonywać operacje, ale jakiś wynikowy obiekt. Na początek spójrzmy na to jakie parametry przyjmują te metody.

## Collect
Metoda Collect ma dwie przeciążone wersje:

```java
<R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);

<R, A> R collect(Collector<? super T, A, R> collector);
```

Tak więc metoda wymaga podania od nas trzech parametrów, bądź też obiektu _Collector_...zaraz zaraz, ale co znajduje się w tym obiekcie? Znajdziemy tam takie metody jak:

```java
Supplier<A> supplier();
BiConsumer<A, T> accumulator();
BinaryOperator<A> combiner();
```

Wygląda na to że klasa _Collector_ służy jako obiekt w którym możemy ukryć implementację 3 wymaganych przez metodę collect parametrów. Razem z JDK dostajemy kilka gotowych implementacji tej klasy. Za pomocą metod statycznych w klasie _Collectors_ mamy możliwość dostać się do tych gotowych implementacji. Dodam jeszcze że te gotowce pokrywają 90%
przypadków z jakimi się spotkałem. Jakie metody statyczne w klasie _Collectors_ są najczęściej używane?

## Collectors
Te najczęściej wykorzystywane to:

  * `toList` - pozwala zaagregować elementy strumienia w postaci listy.
    Gdzie domyślna implementacja to _ArrayList_. Spójrzmy na przykład wykorzystania:

    ```java
    class Person {
        private final String name;
        private final int age;

        public Person(final String name, final int age) {
            this.name = name;
            this.age = age;
        }

        ... getters/setters...
    }
    ...
    final List<Person> persons = Arrays.asList(
                new Person("Max", 18),
                new Person("Peter", 23),
                new Person("Pamela", 24),
                new Person("David", 12));

    final List<Person> adultUsers = persons
                .stream()
                .filter(user -> user.age > 18)
                .collect(toList());
    ```

W powyższym kodzie wszystkie przefiltrowane elementy strumienia, czyli osoby pełnoletnie zostaną umieszczone w liście która będzie wynikiem działania obiektu _Collector_, utworzonego metodą fabrykującą toList.

  * `toCollection` - pozwala zaagregować elementy strumienia w postaci kolekcji podanego typu.
      ```java
      final List<Person> adultUsers = persons
                  .stream()
                  .filter(user -> user.age > 18)
                  .collect(toCollection(LinkedList::new));
      ```

  * `toSet` - agreguje elementy strumienia w zbiór.
      ```java
        final Set<Person> adultUserSet = persons
                .stream()
                .filter(user -> user.age > 18)
                .collect(toSet());
      ```

  * `toMap` - łączy elementy strumienia w mapę. W najprostszej wersji wymaga podania funkcji tworzącej klucze do mapy i funkcji zwracającej elementy dla klucza. Dostępne sa również przeciążone wersje metody przyjmujące również funkcje do rozwiązywania kolizji wstawianych obiektów oraz _Supplier_ za pomocą którego można utworzyć mapę. Utwórzmy więc mapę w której kluczem jest imię użytkownika, a wartością jego wiek:

```java
final Map<String, Integer> adultMap = persons
            .stream()
            .filter(user -> user.age > 18)
            .collect(toMap(Person::getName, Person::getAge));
```


## Parametry metody Collect
Znając już kilka głównych implementacji klasy Collector, przyjrzyjmy się do czego tak naprawdę metoda collect wykorzystuje dostarczone klasie Collect bądź jako osobne parametry implementacje metod supplier, accumulator, combiner. Pierwszy parametr trójargumentowej wersji metody collect to _Supplier_. Jest to funkcja bez argumentowa która zwraca jakiś obiekt. Ten właśnie obiekt jest naszym obiektem wynikowym, który będzie gromadził pozostałe elementy strumienia. Tutaj najczęściej wykorzystuje sie metody fabrykujące, bądź też referencję do konstruktora bez argumentowego. Kolejny parametr, to obiekt typu _BiConsumer_, czyli funkcja przyjmująca dwa parametry i nic nie zwracająca. Służy ona do łączenia poszczególnych elementów strumienia. Jako pierwszy parametr w _BiConsumer_ dostaniemy obiekt który utworzyliśmy poprzez _Supplier_, a drugim elementem jest obiekt strumienia który należy zaagregować. Tak naprawdę te dwie metody w zupełności
wystarczą żeby metoda collect zadziałała. Spójrzmy na przykład:

```java
 final List<Person> persons = Arrays.asList(
                new Person("Max", 18),
                new Person("Peter", 23),
                new Person("Pamela", 24),
                new Person("David", 12));

     final List<Person> adults = persons
                .stream()
                .collect(
                        LinkedList::new,
                        (result, element) -> {
                            System.out.println(
                                "accumulator: " + result + " => " + element
                            );
                            result.add(element);
                        },
                        (p1, p2) -> {
                            System.out.println(
                                "combiner: " + p1 + "," + p2
                            );
                        }
                );

```

Wynikiem działania takiego kodu będzie:
```java
      accumulator: [] => Max
      accumulator: [Max] => Peter
      accumulator: [Max, Peter] => Pamela
      accumulator: [Max, Peter, Pamela] => David
```

Jak widać trzecia przekazana, tajemnicza funkcja nie uruchomiła się. Czy więc jest potrzebna? Jest! Ale tylko w przetwarzaniu równoległym strumieni. Jak wiemy strumienie mogą być dzielone na podstrumienie dzięki _ForkJoinPool_. Są one wtedy przetwarzane w osobnych wątkach. Każdy z tych wątków po zakończeniu pracy musi swój wyniki złączyć z danymi obliczonymi w innych watkach. Za takie złączenie odpowiada ta ostatnia funkcja - _combiner_. Przekazywana jest ona jako obiekt typu _BinaryOperator_. Jaki będzie wynik działa strumienia uruchomionego równolegle w osobnym wątku? Będzie to oczywiście obiekt również utworzony za pomocą przekazanego obiektu _supplier_. A tak to wygląda w praktyce:

```java
        final List<Person> adults = persons
                .parallelStream()
                .collect(
                        () ->{
                            System.out.println("create list");
                            return new LinkedList<>();
                        },
                        (result, element) -> {
                            final String threadName = Thread.currentThread().getName();
                            System.out.println(
                                    "[" + threadName + "] accumulator: " + result + " + " + element
                            );
                            result.add(element);
                        },
                        (subResult1, subResult2) -> {
                            final String threadName = Thread.currentThread().getName();
                            System.out.println(
                                    "[" + threadName + "] combine: " + subResult1 + " + " + subResult2
                            );
                            subResult1.addAll(subResult2);
                        }
                );
```
Gdzie wynikiem działania kodu będzie:

```java
create list
create list
create list
create list
[ForkJoinPool.commonPool-worker-2] accumulator: [] + David
[ForkJoinPool.commonPool-worker-1] accumulator: [] + Peter
[main] accumulator: [] + Pamela
create list
[ForkJoinPool.commonPool-worker-3] accumulator: [] + Max
[ForkJoinPool.commonPool-worker-1] accumulator: [] + Zosia
[ForkJoinPool.commonPool-worker-1] combine: [Zosia] + [David]
[ForkJoinPool.commonPool-worker-1] combine: [Pamela] + [Zosia, David]
[ForkJoinPool.commonPool-worker-3] combine: [Max] + [Peter]
[ForkJoinPool.commonPool-worker-3] combine: [Max, Peter] + [Pamela, Zosia, David]
```

Należy zwrócić tutaj uwagę na kilka rzeczy:
  * obiekt _LinkedList_ został utworzony aż 5 razy i tylko jeden z tych obiektów jest tym który dostaniemy jako wynik operacji, pozostałe służą jako obiekty tymczasowe do których mogą być zapisywane dane w osobnych wątkach. 
  * accumulator w pierwszym parametrze dostał listę, która aktualnie jest procesowana w danym wątku, a jako drugi parametr obiekt który należy zaagregować. Zachowanie identyczne jak w przetwarzaniu jednowątkowym.
  * wreszcie został uruchomiony combiner. Dostał on jako pierwszy parametry dwie listy. Pierwsza jest obiektem wynikowym, jaki zostanie zwrócony z metody _collect_, a drugi jest wynikiem działania strumienia w osobnym wątku.

Oczywiście powyższy przykład można znacznie uprościć korzystając z referencji do metod:
```java
final List<Person> adults = persons
                .parallelStream()
                .collect(
                        LinkedList::new,
                        LinkedList::add,
                        LinkedList::addAll
                );
```

Podsumowując działanie metody _collect_. Przyjmuje ona 3 parametry. Pierwszy odpowiada za utworzenie obiektu który będzie przechowywał dane z strumienia, za pomocą drugiej możemy agregować elementy, natomiast trzecia łączy wyniki pracy strumienia w osobnym wątku z obiektem wyjściowym. Tak więc o parametrze _agregate_ można powiedzieć że służy on do dopisywania danych do wynikowego obiektu, który tworzony jest raz i jest *mutowalny*

## Reduce:
Tak, tak te słowo jest tutaj kluczowe. Ponieważ metoda _reduce_ różni się właśnie tym że została przystosowana o pracy z nie mutowalnymi obiektami. Metoda ta również jest przeciążona:

```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
```

Na początek przyjrzyjmy się tej pierwszej wersji metody. Przyjmuje ona jako pierwszy parametr obiekt takiego samego typu jak ten wynikowy z metody _reduce_. Przykładowo jeżeli _reduce_ miało by złączyć wszystkie elementy strumienia typu integer do jednej liczby, to właśnie jak pierwszy parametr należy podać liczbę do której będą dodawane kolejne. Drugi parametr to metoda za pomocą której będą łączone elementy strumienia. Zobaczmy jako to wygląda w praktyce:

```java
        final Integer adults = persons
                .stream()
                .mapToInt(Person::getAge)
                .reduce(
                        0,
                        (sum, age) -> sum + age
                );
```

Po utworzeniu strumienia obiektów typu Person zamieniamy strumień obiektów na strumień intów w celu uniknięcia zbędnego boxingu. Następnie do liczby 0, która jst naszym parametrem identity zaczynamy dodawać kolejne obiekty strumienia, otrzymując w rezultacie zsumowany wiek ludzi z danej kolekcji. Tak na marginesie identycznie działanie ma metoda _sum_ i z niej powinniśmy korzystać w prawdziwym kodzie :)

Druga wersja metody _reduce_ pozwala nam pominąć przekazywanie początkowej wartości, jednak zwraca wynik jako obiekt typu _Optional_.

```java
        persons
                .stream()
                .mapToInt(Person::getAge)
                .reduce(Integer::sum)
                .ifPresent(System.out::print);
```

Trzecia wersja metody wymaga od nas przekazania również obiektu _combiner_. Który podobnie jako to miało miejsce w funkcji collect łączy wyniki pracy osobnych strumieni.

## Pułapka metody reduce
W powyższych przykładach wykorzystania metody _reduce_ posłużyłem się dodawaniem intów. Przyjrzyjmy się teraz troszkę innemu przykładowi:

```java
        persons
                .stream()
                .map(Person::getName)
                .reduce("",
                        (s1, s2) -> s1 + s2
                );
```

Jaki jest problem z tym kodem? Jest nie optymalny. Przy każdym wywołaniu stringi łączone są za pomocą konkatenacji. Zobaczmy jak wygląda najbardziej trywialne porównanie szybkości działania metody reduce oraz collect:

```java
        final List<Person> persons = new LinkedList<>();
        for(int i=0; i< 1_0000; i++) {
            persons.add(new Person("James", 18));
        }

        long start = System.currentTimeMillis();
        persons
                .stream()
                .map(Person::getName)
                .collect(StringBuilder::new,
                        StringBuilder::append,
                        StringBuilder::append
                );

        System.out.println(System.currentTimeMillis() - start + " ms");
        start = System.currentTimeMillis();

        persons
                .stream()
                .map(Person::getName)
                .reduce("",
                        (s1, s2) -> s1 + s2
                );

        System.out.println(System.currentTimeMillis() - start + " ms");

```

Co daje wynik:

```java
18 ms  // collect + StringBuilder
217 ms // reduce + konkatenacja
```

Widać że mamy tutaj różnice jednego rzędu wielkości. Tak na marginesie, łączenie Stringów za pomocą _collect_ jest już zaimplementowane. W klasie Collectors mamy do dyspozycji przeciążoną metodę _joining_, zwracającą _Collector_ który robi to w optymalny sposób.

## Podsumowanie
Artykuł przybliżył działanie funkcji _reduce_ oraz _collect_. Na przykładach zobaczyliśmy jak działają te funkcje oraz za co odpowiadają ich parametry. Przedstawiłem również kilka gotowych implementacji klasy _Collector_, które rozwiązują najczęściej występujące problemy. Na koniec zobaczyliśmy na jaki problem można się natknąć przy używaniu reduce w błędny sposób.

