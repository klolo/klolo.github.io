---
layout: post
title: "Dlaczego obiekty niezmienne w projekcie to błogosławieństwo i o tym jak nie pisać testów"
date: 2017-10-17 20:31:09 +0200
comments: true
categories: JAVA
---
Dostałem ostatnio drobną poprawkę do wykonania w kodzie. Zwykły null pointer spowodowany tym, że na bazie nie ma części danych. Nie ma, bo nie ma, nie są obligatoryjne. Na backend leci więc podstępny null, pojawia się w polu, gdzie powinna być data. Ten null, w funkcji, która ma posortować listę obiektów po dacie, powoduje exception, krwisto czerwony niczym stek. Sprawa prosta, piszę test jednostkowy, żeby zreplikować problem, robię prosty null save, buduję projekt, przechodzę code review, push na serwer i zapominam o całej sprawie. Nic z tych rzeczy. Poprawka powróciła do mnie niczym owczarek szkocki w jednym z familijnych filmów i srogo się zemściła. Zanim jednak opowiem, jako przestroga, o tym na jaką pułapkę natrafiłem wyjaśnijmy czym jest _immutability_ i niezmienne obiekty

<!--more-->

## Niezmienne parametry


Do tej pory miałem (nie)przyjemność analizować problemy produkcyjne spowodowane tym, że ktoś w długiej metodzie podmieniał parametr wejściowy, a ktoś inny oczekiwał, że parametr wejściowy się nie zmieni. Wyglądało to mniej więcej tak:

```java
public void makeFoo(Input input) {
    log.info("Input: {}", input);
    
    // ...jakis kod ~100 lini...
    input = makeBoo();
    
    // ... jakis kod ~100 lini...
    input.performSomeOperation(); // exeption
}
```

W obiekcie input dostaje komplet danych potrzebnych do wykonania _performSomeOperation_, jednak w połowie funkcji _makeFoo_ ktoś podmienia parametr wejściowy na jakiś inny, nie mający już kompletu danych. W rezultacie dostaje gdzieś dalej w kodzie wyjątek. Analiza loggera na początku metody _makeFoo_ wskazuje, że na wejściu był komplet danych. Szukam zatem w _performSomeOperation_ oraz w dalszych metodach. Czas mija, produkcja stoi, telefon dzwoni, stres maksymalny. Sytuacji, o dziwo, wcale nie ułatwia fakt, że jest to system legacy bez testów jednostkowych, metoda _makeFoo_ ciągnie się jak jamnik na sterydach i wygląda na stęsknioną ostrego refactoringu. Mijały godziny zanim zauważyłem w czym jest błąd. Problem z oczami wykluczam, bo nie analizowałem kodu sam. Identyczną sytuację miałem od tamtego czasu jeszcze jakieś 3 razy. Po tych przykrych doświadczenia pokochałem słowo _final_. Dzięki niemu nie można podmienić parametru wejściowego do metody. Jest to już jakieś zabezpieczenie. Niektórzy powiedzą, że jest to więcej pisania, za dużo roboty. Jeżeli napisanie kilka razy dziennie słowa _final_ w kodzie jest zbyt ciężką pracą - można i tak... Można też stwierdzić, że to jest szum w kodzie, zbędna informacja, że przy prawidłowo napisanych metodach mieszczących się na ekranie bez przewijania, jest to zbędne. Niekoniecznie. Zamiast czytać całą metodę w celu ustalenia czy podmieniamy ten parametr czy nie, wystarczy zerknąć na listę parametrów i mieć tę informację od razu. Jakież to szczęście, że w moim obecnym projekcie mamy konwencję, która wymusza na nas używanie final w parametrach.

## Niezmienne zmienne lokalne

Po dłuższym zastanowieniu da się odkryć, że _final_ w parametrach to czasami za mało. Zamiast podmiany parametru można w końcu podmienić zmienną deklarowaną na początku funkcji i oczekiwać pierwotnej wartości gdzieś dalej w kodzie. Po co kusić los? Uważam, że zmienne deklarowane w metodach również powinny być _final_, dzięki czemu kod staje się bardziej przewidywalny, jest mniej zmiany stanów, łatwiej go analizować i testować. Początkowo mogą pojawić się zarzuty - że blokujemy możliwość przypisania czegoś do zmiennej, że nie da się tak programować. Ano da się. Kiedy przyjrzałem się klasom w różnych systemach, które rozwijam ze zdziwieniem stwierdziłem, że jakieś 90% zmiennych po zadeklarowaniu nie ulega zmianie, 5% można tak napisać, żeby się nie zmieniały, a tylko kilka procent kodu faktycznie wymaga zmiennej z różnymi stanami, bo tak jest dużo prościej. Dla mnie sprawa z deklarowaniem zmiennych jako _final_ jest prosta. Sprawia to, że tworzony kod jest mnniej _bugogenny_. Przy takim podejściu łatwiej również przestawić się na programowanie funkcyjne, gdzie czyste funkcje nie modyfikują stanu zmiennych.

## Niezmienne obiekty
Idąc krok dalej, zacząłem się zastanawiać czy nie warto używać final do pól w klasie. Wszystkich pól. I myślę, że to także jest dobry pomysł, funkcjonujący pod nazwą _immutable objects_. Obiekt po utworzeniu nie może zmienić swojego stanu. Przykładami takich obiektów jest np. _String_, _BigDecimal_. Pojawia się pytanie - jak ustawiać pola w takich obiektach? Jeżeli jest dużo parametrów - będzie potrzebny wielki konstruktor, jeżeli część danych nie jest wymagana - mogą powstawać konstruktory teleskopowe. A co w przypadku, kiedy część danych nie jest dostępna od razu, ale trzeba je pobrać z innego miejsca systemu? Rozwiązaniem problemu jest wzorzec builder. Implementowanie tego wzorca dla każdego nowego obiektu DTO możemy być uciążliwe, dlatego warto skorzystać z automatycznego generowania buildera przez IDE. Jednak tutaj przy zmianie DTO, należy go ręcznie edytować lub jeszcze raz wygenerować. Alternatywą jest biblioteka _Lombok_, która w momencie kompilacji kodu na podstawie annotacji w kodzie generuje dodatkowy kod,
taki jak np. gettery, settery, toString czy też kod buildera. Korzystanie z _Lombok_ sprawia, że w kodzie nie mamy zbędnego _boilerplate_ tylko faktyczną logikę biznesową. Oto prosty przykład połączenia _Lombok_ i niemutowalnego obiektu:

```java
@Builder
public class Account {
    private final AccountType type;
    private final String number;
    private final BigDecimal amount;
    private final Currency currency;
}

Account.builder()
        .number("865423")
        .type(AccountType.RO2)
        .amount(new BigDecimal("0"))
        .currency(Currency.CHF)
        .build();
```

Bez pisania kodu buildera możemy stopniowo wypełniać danymi obiekty, które później nie zmienią swojego stanu i będą bardziej przewidywalne. Dodatkowo takie obiekty ułatwią programowanie wielowątkowe. Warto wspomnieć o tym, że korzystając z _Lombok_ można korzystać z [val](https://projectlombok.org/features/val). Czy nie zaczyna to jednak za bardzo przypominać innego języka jvm-a...?

## O tym, jak nie pisać testów jednostkowych

Na koniec chciałbym opowiedzieć o tym jak zemścił się na mnie brak niemutowalnych obiektów. Napisałem test jednostkowy do null pointera, o którym wspomniałem na początku. Test wyglądał mniej więcej tak:

```java
@Test
public void shouldNotFailedWhenDateIsNull() {
    // given
    final List<Item> items = FluentIterable
                                    .from(Mock.items)
                                    .transform( item => {
                                        item.setDate(null);
                                        return item;
                                    })
                                    .toList();
    //when 
    someService.sortByDate(items);

    //then
    Assert.....
}
```

Bardzo prosty test, w którym pobieram listę jakiś obiektów z mocka, ustawiam datę jako null, wywołuję metodę, gdzie był błąd _NullPointerException_ i robię kilka dodatkowych asercji. Test po uruchomieniu przechodzi, projekt się buduje, osoba robiąca code review nie zgłosiła uwag. Problem pojawił się jednak kolejnego dnia, kiedy okazało się, że środowisko testowe nie działa, ponieważ CI nie może zbudować projektu. Dziwna sprawa, inne testy jednostkowe przestały działać. Pomyślałem najpierw, że to pewnie przez jakieś inne zmiany w kodzie. W końcu osób commitujących do projektu jest kilka. Jeden z developerów usiadł i zaczął analizować problem. No i się zaczęło. Szok i niedowierzanie. Identyczny kod jak na środowisku developerskim, a nie buduje się na Jenkins. Po całym dniu analizy okazało się, że to przez wspomniany test jednostkowy. Modyfikuje on dane mockowe, które są wykorzystywane w innych testach. Po ustawieniu null na obiekcie inne testy nie mają prawa działać. Dlaczego jednak ten problem nie występuje na środowisku developerskim? Przypadek :) JUnit nie gwarantuje kolejności wykonywania testów. Oczywiście jeżeli w kodzie obiekty byłyby niemutowalne, to test musiałbym zaimplementować inaczej i nie byłoby problemu. Czego się nauczyłem? Że niemutowalność jest bardzo dobrą praktyką i pozwala uniknąć wielu podstępnych błędów. Na koniec jeszcze [kilka słów](https://github.com/junit-team/junit4/wiki/Test-execution-order) o kolejności wykonywania testów w JUnit:

> By design, JUnit does not specify the execution order of test method invocations. Until now, the methods were simply invoked in the order returned by the reflection API. However, using the JVM order is unwise since the Java platform does not specify any particular order, and in fact JDK 7 returns a more or less random order. Of course, well-written test code would not assume any order, but some do, and a predictable failure is better than a random failure on certain platforms. From version 4.11, JUnit will by default use a deterministic, but not predictable, order (MethodSorters.DEFAULT). To change the test execution order simply annotate your test class using @FixMethodOrder and specify one of the available MethodSorters.