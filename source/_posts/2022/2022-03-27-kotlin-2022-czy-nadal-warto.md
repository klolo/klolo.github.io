---
layout: post
title: "Kotlin zamiast Javy w 2022... Czy nadal warto?"
date: 2022-03-27 21:31:32 +0100
comments: true
categories: KOTLIN
---

Po kilku latach przerwy od pisania w Kotlinie zastanawiam czy nadal język ten jest wart rozważenia jako zamiennik Javy, przecież ta cały czas się rozwija i co kilka miesięcy mamy nowe wydania. Może Java ma już to wszystko, czym Kotlin górował nad nią kilka lat temu? Porównajmy zatem kilka nowych mechanizmów Javy do tego, co oferuje Kotlin.

## Pattern matching

Release Javy 18 przyniósł przeniesienie JEP-420 w faze second preview. Propozycja zmiany kryjąca się pod tym numerkiem to nic innego jak pattern matching i ulepszony switch. Nie miałem jeszcze okazji pracować z tym mechanizmem, bo projekt, który rozwijam nadal jest na jvm 11 i podbicie wersji mamy dopiero w planach. Jednak to, co widzę w dokumentacji Javy wygląda jak...kopia tego, co oferuje Kotlin. Składnia i możliwości tego, co daje Java wygląda bardzo podobnie. Ciężko mi wskazać jakieś istotne różnice. Na pewno nie jest to jeden z powodów, dla których warto wybrać jeden jęxyk zamiast drugiego. Dla porównania składnia nowego switcha z Javy:

```java
class Foo {
    static String formatterPatternSwitch(Object o) {
        return switch (o) {
            case Integer i -> String.format("int %d", i);
            case Long l    -> String.format("long %d", l);
            case Double d  -> String.format("double %f", d);
            case String s  -> String.format("String %s", s);
            default        -> o.toString();
        };
    }
}
```
Oraz Kotlin:

```kotlin
fun formatter(o: Any) =
    when (o) {
        is Int -> String.format("int %d", o)
        is Long -> String.format("long %d", o)
        is Double -> String.format("double %f", o)
        else -> "unknown"
    }
```

## Records

Bardzo często programując tworzymy klasy, które są niczym więcej jak tylko workiem na jakieś dane. Takie klasy wymagają od nas zdefiniowania boilerplate, takiego jak gettery, settery, toString(), equals(), hashcode(). Żeby radzić sobie z takim brzydkim kodem w projekcie w którym pracuje używa się Lomboka. Za pomocą kilku adnotacji możemy automatycznie wygenerować kod, na który nie chcemy patrzeć a tym bardziej pisać go. Nowe wydania Javy wprowadzają mechanizm, który częściowo rozwiązuje ten problem, przynajmniej dla prostych DTO. Są to rekordy, czyli nie mutowalne pudełka na dane, dla których Java wygeneruje podstawowe metody. Przykład deklaracji takiego rekordu w Javie:

```java
public record SomeData(String key, String value) {}
```

Kotlin oferował bardzo podobny mechanizm, już kilka lat wcześniej, są to `data class`. Przykład deklaracji:

```kotlin
data class SomeData(val key: String, val value: String)
```

Znów wygląda to bardzo podobnie. Gdzie tkwi różnica? 

* Na pewno w tym, co się dzieje pod spodem, z jakich mechanizmów jvm korzystają oba rozwiązania. Jednak pominę ten aspekt celowo, chce się skupić na tym, co widzi developer. 
* W przypadku Kotlina możemy mieć pola, które są mutowalne oraz możemy definiować pola w takiej klasie, daje to większe możliwości. 
* Idąc dalej, Kotlin generuje dla nas metodę `copy`, która pozwala stworzyć kopię obiektu modyfikując przy okazji wybrane pola.  
* Mechanizm w Javie do utworzonej klasy dodaje dziedziczenie po klasie `Record`, przez co rekord nie może dziedziczyć po innej klasie.   
 
Oba języki oferują podobne możliwości w tym zakresie, jednak mechanizm z Kotlin-a wydaje się tutaj odrobinę bardziej elastyczny.

## Multiline string

Temat wydaje się prosty. Kotlin miał wielolinijkowe Stringi, teraz Java go dogoniła i niczym mu nie ustępuje...No nie do końca...Diabeł tkwi w szczegółach. Jak w takim bloku tekstu odwołać się do zmiennej, wykonać wyrażenie? Niestety w Javie nie da się. Można próbować skorzystać z funkcji `format`:

```java
class Example {
    void foo(String type) {
        String code = String.format("""
                public void print(%s o) {
                    System.out.println(Objects.toString(o));
                }
                """, type);
    }
}
```

Nie wygląda to zbyt pięknie. Do porównania blok tekstu w Kotlinie, gdzie możemy odwołać się do jakieś zmiennej lub nawet wykonać wyrażenie:

```kotlin
fun main(args: Array<String>) {
    val a = 5
    val b = 6

    val myString = """ Some example:
    |${if (a > b) a else b}
    |end of example
    """
    println("Larger number is: ${myString.trimMargin()}")
}
```

## Sealed class

Jest to mechanizm, który pozwala określić nam kto będzie dziedziczył po naszej klasie. Dzięki czemu kompilator jest w stanie sprawdzić, czy uwzględniliśmy wszystkie podtypy w instrukcji switch. Oba języki oferują aktualnie ten mechanizm. Podobnie jak w przypadku pattern matching i switch, ciężko mi tutaj wskazać jakieś istotne różnice w obu językach. Nie jest to element, który powinien wpływać na decyzję, który język wybrać.  

## Podsumowanie

Java nie przespała kilku ostatnich lat i ciągle się rozwija. Widać, że nadrobiła kilka braków względem Kotlin, ale co jeszcze pozostało do poprawienia?

* **składnia** — Java posiada barokową składnię, która wymaga dużo większej ilości kodu niź Kotlin. Brałem udział w projekcie, który przenosiliśmy z Javy na Kotlin i okazało się, że około 40% znikło podczas tej migracji. A jak wiadomo mniej kodu to czytelniejszy kod i lepsze jego zrozumienie.
* **null save** — Zrobić NullPointerException w Kotlinie jest dość ciężko, trzeba tego chcieć. W Javie natomiast musimy dodawać brzydkie if-y i kompilator nam nie pomaga w walce z tym powszechnym błędem. Jedyne co tutaj się poprawiło to informacja o tym, co jest nullem w stacktrace, no ale to nie kwestia języka a jvm. Do dyspozycji w Javie mamy jeszcze Optional... uważam, jednak że to marna namiastka tego bezpieczeństwa, jakie daje Kotlin. 
* **funkcje extension** — Kotlin pozwala deklarować dodatkowe funkcje do istniejących klas, nawet tych z bibliotek czy jdk. Funkcje te nie rozszerzają faktycznie tamtych klas, ale możemy je wywoływać tak jakby w tych klasach były. Bardzo przydatne, pozwala uniknąć pisania utily.
* **coroutine** — zestaw narzędzi ułatwiających pisanie czytelnego kodu asynchronicznego. Nie miałem okazji używać, wiem, że są i mogą ułatwić życie.
* **inline class** — pisząc kod domenowy często potrzebujemy prostych małych klas, żeby opakować np. String który jest jakimś id. Dzięki temu mamy dużo bardziej bezpieczny kod z pomocą kompilatora, który sprawdza, czy przekazujemy prawidłowe dane. Jest to dużo bardziej eleganckie podejście niż przekazywanie wszędzie stringów. Problem pojawia się natomiast gdy takich klas mamy bardzo dużo, może to wpłynąć na wydajność aplikacji. Kotlin przychodzi nam tutaj z pomocą i pozwala takie klasy mieć tylko na poziomie źródeł a później w runtime korzysta z typów, które opakowujemy. 
* **delegation** — alternatywa do dziedziczenia, która otwiera nowe techniki, wzorce do wykorzystania w kodzie.
* **aliasy** — jeżeli mamy jakąś długą skomplikowaną nazwę np. z powodu użycia typów generycznych, możemy stworzyć dla niej alias przez co kod staje się czytelniejszy. Korzystałem wielokrotnie i naprawdę pomaga to zwiększyć czytelność. 
* etykiety używane razem z return — pozwalają łatwiej wyjść z zagnieżdżonego bloku kodu
* programowanie funkcyjne — dzięki prostszej składni jest nie porównywalnie **lepiej pisać funkcyjnie w Kotlinie niż w Javie**. Dodatkowo w Kotlinie możemy tworzyć funkcje, które nie są osadzone w żadnej z klas, co otwiera drzwi do pisania w bardzo funkcyjny sposób. 
* **strumienie** — co tu dużo mówić, Kotlin oferuje sporo więcej metod i w połączeniu z bardziej nowoczesną składnią jest czytelniejszy. Jeżeli mamy jeden parametr w lambdzie to nawet nie musimy pisać nazwy parametru (no chyba że chcemy) i używamy domyślnego `it`. Dzięki takim szczegółom kod staje się bardzo zwięzły.
* możliwości podbijania wersji języka bez konieczności aktualizacji jvm, co znacznie ułatwia bycie up to date.
* dodatkowe możliwości kompilatora, takie jak **kompilacja do js** czy do **kodu natywnego uruchamianego bez jvm**.

Nie poruszyłem tutaj bardzo wielu innych aspektów, ale juź widać, że mimo starań, Java nadal jest językiem, który bardzo ustępuje Kotlinowi. Biorąc pod uwagę niski prób wejścia, jestem zwolennikiem tego, żeby Kotlin był językiem pierwszego wyboru na jvm.