---
layout: post
title: "Wstęp do programowania funkcyjnego"
date: 2017-08-04 20:25:49 +0200
comments: true
categories: METHODOLOGY
---

Ten wpis ma na celu wyjaśnienie czym jest programowanie funkcyjne oraz przedstawienie podstawowych zasad panujących w świecie programowania funkcyjnego.
<!--more-->

## Programowanie deklaratywne vs imperatywne

Wprowadzenie do programowania funkcyjnego zacznę od przypomnienia czym jest paradygmat programowania deklaratywnego. Jest to podejście, w którym tworząc
 kod skupiamy na się na tym co ma zostać wykonane, a nie w jaki sposób. Szczegóły implementacji pozostawiamy np. bibliotece, odwracając w ten sposób
  kontrolę nad kodem. W końcu wszyscy lubią odwracanie kontroli (inversion of control) ;). Tak napisany kod jest bardziej zwięzły i czytelniejszy z powodu
  mniejszej ilości zbędnego szumu. Pewnie setki razy zdarzyło Ci się napisać: for(int i=0; i<... Czy nie uważasz, że w większości przypadków
  tak napisany kod jest złem koniecznym, niezbędnym do osiągnięcia określonego działania aplikacji? Podejście, w którym wykorzystywane są instrukcje sterujące
   kodem nazywane jest programowaniem imperatywnym. Jest to przeciwieństwo programowania deklaratywnego, gdzie nie tylko trzeba martwić się co ma robić kod,
    ale również w jaki sposób to osiągnąć. Programowanie funkcyjne idzie w parze z programowaniem deklaratywnym, dzięki czemu kod będzie krótszy niż ten klasyczny
    obiektowo imperatywny.
{% highlight java %}
final List<String> users = Arrays.asList("Ewa", "Bartek", "Adam");

// 1. imperatywnie
for (int i = 0; i < users.size(); i++) {
    System.out.println(users);
}

// 2. deklaratywnie
users.forEach(System.out::print);
{% endhighlight %}

## Co to jest programowanie funkcyjne?


Programowanie funkcyjne to programowanie, w którym najważniejszą rolę pełnią funkcje. Możesz śmiało zaprotestować: Zaraz, zaraz...Czy czasami mój obiektowy kod nie składa się
również z funkcji? Ależ tak! Jednak zasadnicza równica jest taka, że w FP (ang. Functional programming) funkcje służą do opisu przepływu sterowania i operacji - a nie
 tak jak to ma miejsce w przypadku programowania obiektowego - do zmiany stanu obiektów. Z tego założenia wynika również jedna z cech programowania obiektowego - niezmienność obiektów.
 Czym jeszcze charakteryzuje się FP? O tym w dalszej części.

## Czyste funkcje
Czyste funkcje to funkcje bez efektów ubocznych. Są one elementem, bez którego nie da się pisać funkcyjnie. Nie zależą one od żadnych zmienny zewnętrznych np. pól w
klasie, a wszystkie dane są przekazywane jako parametry. Dlatego też dla podanego zestawu danych czysta funkcja zawsze zwróci taki sam wynik. Zaburzeniem tego może być
również wykorzystanie w funkcji instrukcji random, w której wynik operacji nie jest stały dla danych parametrów. Czyste funkcje również nie modyfikują stanów zewnętrznych
np. pól w klasie, lecz każdy wynik jest zwracany jako wynik funkcji.

## Funkcje jako obywatele pierwszej klasy
Pod tym pojęciem kryje się zdolność języka do traktowania funkcji na równi z obiektami klas tzn. funkcje można przekazywać jako parametry do metod, można je przypisywać
do zmiennych oraz tworzyć wewnątrz funkcji. Przed Java 8, język nie był wyposażony w elementy, które na to pozwalały. Jak to zrealizować w Java 8 pokazuje poniższy przykład.

{% highlight java %}
final double PI = 3.14;

// Przypisanie funkcji do zmiennej
final Function<Double, Double> circleArea = r -> r * r * PI;

// Przekazanie funkcji do metody
void showCalculatedArea(Function<Double, Double> algorithm, double r) {
    System.out.println(algorithm.apply(r));
}
{% endhighlight %}

## Niezmienność obiektów

Jest to cecha FP związana z czystymi funkcjami. W językach czysto funkcyjnych często nie istnieje pojęcie zmiennej, tak więc wszystkie tworzone obiekty są niezmienne. Dzięki
takiemu podejściu nie ma możliwości zmiany stanów ani przekazanych do funkcji parametrów, jak i pól w klasie, trudniej jest w funkcji wykonać efekt uboczny. W związku z czym
final jest zawsze mile widziany. Jak ułatwić sobie życie w takim kodzie? Polecam bardzo buildery, czy to te implementowane przez Ciebie, czy też te generowane automatycznie
np. za pomocą Lombok-a. Builder ma dwie zasadnicze zalety. Pierwsza to to że nie musimy używać konstruktorów z wieloma parametrami gdzie oraz to, że mimo użycia tylko
stałych pól w klasie możemy utworzyć obiekt w kilku krokach. Jak wygląda najprostszy builder?

{% highlight java %}
    public class User {
        private final String name;
        private final int age;

        private User(final String name, final int age) {
            this.name = name;
            this.age = age;
        }

        public static UserBuilder builder() {
            return new UserBuilder();
        }

        public static class UserBuilder {
            private String name;
            private int age;

            public UserBuilder age(final int age) {
                this.age = age;
                return this;
            }

            public UserBuilder name(final int name) {
                this.name = name;
                return this;
            }

            public User build() {
                return new User(name, age);
            }
        }
    }
{% endhighlight %}

Jak widać wszystkie pola w klasie User są stałe i nie mamy potrzeby eksponować na zewnątrz _brzydkiego_ konstruktora z wszystkimi polami. Dodatkowo w metodzie build możemy
umieścić logikę walidującą stan danych przed utworzeniem obiektu. Przykładowe użycie buildera wygląda następująco:

{% highlight java %}
    final UserBuilder builder = User.builder();
    builder
            .age(18)
            .name("Frank");
{% endhighlight %}

Dzięki wykorzystaniu wzorca fluent interface mamy możliwość łączenia metod w ciąg wywołań.

## Podsumowanie

Dzięki temu, że funkcje nie mają efektów ubocznych, wynik działania jest łatwiejszy zarówno do przewidzenia jak i do przetestowania. Dodatkowo czysta funkcja dla podanych parametrów
zawsze zwróci ten sam wynik, dlatego można z łatwością wykorzystać cache w celu optymalizacji aplikacji. Dzięki używaniu czystych funkcji znikną również największe problemy
z wielowątkowością, takie jak jednoczesna modyfikacja wspólnych danych przez kilka wątków. Możemy skupić się również na tym co mają robić nasze funkcje, a nie w jaki sposób
implementować np. pętle lub warunek. Łatwiej jest nam również reużywać napisany wcześniej kod. Jakie są natomiast zalety tego, że obiekty są niezmienne? Łatwiej
przewidzieć co się w nich znajduje, dzięki czemu można szybciej zdiagnozować i poprawić ewentualne błędy.
