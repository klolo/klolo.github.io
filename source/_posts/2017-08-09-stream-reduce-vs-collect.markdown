---
layout: post
title: "Java stream reduce vs collect"
date: 2017-08-09 22:13:08 +0200
comments: true
categories: JAVA
---
Pracując z strumieniami Javy 8 pojawia się czasami konieczność złączenia wszystkich elementów strumienia w jakiś obiekt wynikowy. Taką operacje
można wykonać zarówno za pomocą metody reduce, jak również za pomocą collect? Jaka jest różnica między tymi metodami i na jaki problem można się natknąć
korzystając z nich?

<!--more-->
Zarówno metoda reduce jak i collect są operacjami terminalnymi strumienia. Co oznacza tyle że ich wynikiem nie jest strumień na którym można dalej wykonywać
operacje, a jakiś wynikowy obiekt. Na początek spójrzmy na to jakie parametry przyjmują te metody.

## Collect
Metoda Collect ma dwie przeciążone wersje:

{% highlight java %}
<R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);

<R, A> R collect(Collector<? super T, A, R> collector);
{% endhighlight %}

Tak więc metoda wymaga podania od nas trzech metod. Bądź też obiektu Collector...zaraz zaraz, ale co znajduje się w tym obiekcie? Znajdziemy tam takie metody jak:

{% highlight java %}
Supplier<A> supplier();
BiConsumer<A, T> accumulator();
BinaryOperator<A> combiner();
{% endhighlight %}

Wygląda na to że klasa Supplier jest tak naprawdę obiektem w którym możemy ukryć implementację 3 wymaganych przez collect parametrów. Taką gotową implementację dostaje
również od Javy w postaci klasy Collectors, gdzie za pomocą metod statycznych mamy możliwość dostać się do gotowych Collectorów. Dodam jeszcze że te gotowce pokrywają 99%
przypadków z jakimi się spotkałem. Jakie metody, a właścicie Collectory znajdziemy w tej klasie?

## Collectors
Te najczęściej wykorzystywane to:

  * `toList` - pozwala zaagregować elementy strumienia w postaci listy.
    Gdzie domyślna implementacja to ArrayList. Spójrzmy na przykład wykorzystania:

    {% highlight java %}
    class Person {
        private final String name;
        private final int age;

        public Person(final String name, final int age) {
            this.name = name;
            this.age = age;
        }
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
    {% endhighlight %}

    W powyższym kodzie wszystkie przefiltrowane elementy strumienia, czyli osoby pełnoletnie zostaną umieszczone w liście która będzie wynikiem działania Collectora toList.

  * `toCollection` - pozwala zaagregować elementy strumienia w postaci kolekcji, gdzie możemy podać z jakiej implementacji chcemy skorzystać
      {% highlight java %}
      final List<Person> adultUsers = persons
                  .stream()
                  .filter(user -> user.age > 18)
                  .collect(toCollection(LinkedList::new));
      {% endhighlight %}

  * `toMap`

  * `toSet`

## Parametry metody Collect
Ale za co tak naprawdę odpowiadają funcje których implemenentacje możemy ukryć w klasie Collector? Pierwszy parametr, czyli Supplier jest to funkcja bez argumentowa która zwraca
jakiś obiekt. Ten obiekt jest naszym obiektem wynikowym, który będzie gromadził pozostałe elementy strumienia. Tutaj najczęscięj wykorsystuje sie metody fabrykujące, bądź też
referencję do kontruktora bez parametrowego. Kolejny parametr, to obiekt typu BiConsumer, czyli funkcja przyjmująca dwa parametry i nic nie zwracająca. Służy ona do łączenia poszczególnych elementów strumienia.
Jako pierwszy parametr dostaniemy takiej metody dostaniemy obiekt który utworzyliśmy poprzez Supplier, a drugim elementem jest obiekt strumienia który należy zaagregować. Tak naprawdę te dwie metody w zupelności
wystarczą żeby collect zadziałał. Spójrzmy na przykład:
{% highlight java %}
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
                            System.out.println("accumulator: " + result + " => " + element);
                            result.add(element);
                        },
                        (p1, p2) -> {
                            System.out.println("combiner: " + p1 + "," + p2);
                        }
                );

{% endhighlight %}

Wynikiem działania takiego kodu będzie:
{% highlight %}
      accumulator: [] => Max
      accumulator: [Max] => Peter
      accumulator: [Max, Peter] => Pamela
      accumulator: [Max, Peter, Pamela] => David
{% endhighlight %}

Jak widać trzecia przekazana, tajemnicza funkcja nie uruchomiła się. Czy za tem jest potrzebna? Jest! Ale tylko w przetwatrzaniu równoległym strumieni. Jak wiemy strumienie mogą być dzielone na podstrumienie
które dzięki ForkJoinPool są przetwatzane w osobnych wątkach. Każdy z tych strumieni po zakończeniu pracy musi swój wyniki złączyć z resztą resultatów oblicznonych w innych watkach. Za takie złączenie
odpowiada ta ostatnia funkcja., przekazywana jako obiekt typu BinaryOperator. A tak to wygląda w praktyce:

{% highlight java %}
      final List<Person> adults = persons
                .parallelStream()
                .collect(
                        LinkedList::new,
                        (result, element) -> {
                            final String threadName = Thread.currentThread().getName();
                            System.out.println("[" + threadName + "] accumulator: " + result + " + " + element);
                            result.add(element);
                        },
                        (subResult1, subResult2) -> {
                            final String threadName = Thread.currentThread().getName();
                            System.out.println("[" + threadName + "] combine: " +subResult1 + " + " + subResult2);
                            subResult1.addAll(subResult2);
                        }
                );
{% endhighlight %}
Gdzie wynikiem działania kodu będzie:
{% highlight %}
[main] accumulator: [] + Pamela
[ForkJoinPool.commonPool-worker-2] accumulator: [] + David
[main] accumulator: [] + Zosia
[main] combine: [Zosia] + [David]
[main] combine: [Pamela] + [Zosia, David]
[ForkJoinPool.commonPool-worker-1] accumulator: [] + Peter
[ForkJoinPool.commonPool-worker-3] accumulator: [] + Max
[ForkJoinPool.commonPool-worker-3] combine: [Max] + [Peter]
[ForkJoinPool.commonPool-worker-3] combine: [Max, Peter] + [Pamela, Zosia, David]
Adults:[Max, Peter, Pamela, Zosia, David]
{% endhighlight %}



## Reduce:
{% highlight %}

{% endhighlight %}


Zasadnicza różnica między obiema metodami jest to że collect zaprojektowane jest do pracy z obiektami które są mutowalne a nie reduce z nie mutowalnymi. Takim obiektem mutowalnym
jest na przykład Lista, do której możemy dodawać kolejne elemnty. Lista jest tworzona jest tylko raz i ten jedyny obiekt zawiera w sobie zaagregowane wszystkie pozostałe elemenyt strumienia.
Idelanie pasuje ona do metody collect. Metoda reduce działa w taki sposób że za agregując kolejne obiekty strumienia tworzy za każdym razem nowy obiekt wyjściowy.

StringBuilder, konkatenacja, brak optymalizacji dla s1+s2

:exclamation: