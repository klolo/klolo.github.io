---
layout: post
title: "W oczekiwaniu na Jave 10"
date: 2017-12-17 13:39:22 +0100
comments: true
categories: JAVA
---
We wrześniu mieliśmy premiere Javy 9 oraz zapowiedź zmiany cyklu wydawniczego tego języka. Teraz nowe wersje będą pojawiać się co pół roku. W marcu oraz w wrześniu. 
Zmianie ulegnie również wersjonowanie kolejnych wydań. Na stronie Oracle kolejne wydania Javy mają oznaczenie _rok.miesiąc_. Tak więc za kilka miesięcy
zamiast Javy 10 zobaczymy Jave 18.3. Omówię dzisiaj jakie nowości mogą trafić do tej wersji Javy.
<!--more-->

## Zmiany w 18.3

Java 18.3 zaoferuje nam oprócz bug fixów zaledwie 12 większych nowości. W porównaniu z Java 9, gdzie lista nowości obejmowała prawie 100 pozycji, nie jest to aż tak 
rewolucyjne wydanie. W końcu przy pół rocznym cyklu wydawniczym nie ma sensu oczekiwać zbyt wielu nowinek w języku. 
W tych zapowiedzianych przez [OpenJDK](http://openjdk.java.net/projects/jdk/10/) nowościach (OpenJDK jest wzorcową dla Oracle implementacją jvm) tylko jedna moim zdaniem będzie
miała znaczący wpływ na to jak programujemy. Ale o tym za chwilę, najpierw zobaczmy czym są pozostałe zapowiedziane zmiany:

  * Przez lata kod źródłowy jdk podzielony był na kilka repozytoriów. Dla Javy 9 było ich aż 9, teraz planowane jest umieszczenie wszystkiego w jednym miejscu. 
  * Zmiany w G1 wprowadzające wielowątkowe odśmiecanie oraz modularność kodu GC.
  * Rozbudowa mechanizmu współdzielenia metadanych pomiędzy różnymi procesami Javy. Polega on na tym, że podczas ładowania klasy, jvm najpierw sprawdza czy 
  metadane klas znajdują się w cache, a dopiero później próbuje je wczytać. Feature ten pojawił się również w jdk-8u40 jako experymentalny. Można go uruchomić za pomocą 
  flagi `-XX:+UseAppCDS`. Jak podano na stronie OpenJDK mechanizm pozwoli przyspieszyć start aplikacji o 20-30% oraz zmniejszyć zużycie pamięci o kilka procent. 
  * Optymalizacja zarządzania wątkami przez jvm.
  * Usunięcie narzędzia _javah_ służącego do generowania nagłówków i kodu metod natywnych w C. Od jdk 1.8 zajmuje się tym kompilator javy, stąd też dodatkowe
  narzędzie nie jest już potrzebne. 
  * Obsługa nie ulotnej pamięci RAM: [NV-DIMM](http://searchstorage.techtarget.com/definition/NVDIMM-Non-Volatile-Dual-In-line-Memory-Module). 
  Oferuje ona wolniejszy dostęp dla danych stosu niż _DRAM_, ale za to korzystając z niej możemy szybszą pamięć zostawić ważniejszym procesom. 
  Jako parametr jvma będzie można podać w jakiej pamięci ma być alokowany stos. 
  * W jdk 9 pojawił się experymentalny feature - kompilacja _ahead of time_. Dzięki skompilowaniu kodu Javy bezpośrednio do kodu natywnego na danej platformie, 
  podczas runtime nie ma potrzeby spowalniać aplikacji kompilajcą JIT, która uruchamiana jest dopiero po wykonaniu dużej liczby wywołań danego fragmentu kodu
  (tzw. rozgrzanie javy) i nie daje gwarancji, że cały byte code zostanie skompilowany do C. Kompilacja AOT możliwa jest za pomocą polecenia _jaotc_ i wykorzystuje kompilator
  o nazwie _Graal_. Został on napisany w Javie i w jdk 18.3 będzie można go wykorzystać nie tylko do AOT ale również do JIT. Oczywiście są to mechanizmy których 
  nie powinniśmy dodawać jeszcze do naszych projektów, z powodu bardzo wczesnego etapu prac nad nimi. 
   
## Wisienka na torcie
  
Java jako język statycznie typowany zawsze wymagała żeby programiści deklarując zmienną podawali jakiego typu ona będzie. Dodatkowo często zdarza się, że informacja o 
tym jakiego typu jest zmienna znajduje się również w jej nazwie. Jest to niedogodność względem języków dynamicznie typowanych takich jak np. javascript, gdzie zmienne deklaruje 
się bez podania typów. Piszemy _var_ i na poziomie runtime określany jest typ danych jakie są przechowywane w zmiennej. Pisząc w javie wiele osób narzeka na boilerplate 
jaki powstaje w tym języku. Przykładowa bardziej zaawansowana deklaracja mapy w Javie 1.6 wygląda tak: 

```java
HashMap<BigDecimal, Map<String, LinkedList<String>>> dataMap
                            = new HashMap<BigDecimal, HashMap<String, LinkedList<String>>>();
``` 

Wychodzi nam porządny jamnik. Java 1.7 upraszcza ten zapis wprowadzając operator _diamond_, dzięki któremu typ generyka zostaje określony 
na podstawie deklaracji zmiennej:

```java
HashMap<BigDecimal, Map<String, LinkedList<String>>> dataMap = new HashMap<>();
``` 

Już jest lepiej. Ale co w przypadku kiedy chce wyjąć z mapy jakiś element i przypisać do zmiennej? Znów musimy podać typ elementu. Dostajemy coś takiego:

```java
Map<String, LinkedList<String>> value = dataMap.get(amount);
``` 

Nie mamy możliwości skrócenia takiego zapisu, choć jakby się zastanowić to typ zmiennej mógłby być z powodzeniem określony na podstawie typu zmiennej _dataMap_. 
Twórcy javy chcą ułatwić nam życie i dodać do języka możliwość określania typu zmiennej lokalnej na podstawie tego co do niej przypisujemy. Planowane jest wykorzystanie
w tym celu słówka kluczowego _var_. Zamiast podawać typ zmiennej, który kompilator sam za nas określi będzie można napisać: 
  
```java
var value = dataMap.get(amount);
``` 

Taka instrukcja wygląda dużo prościej niż ta z generykiem. Jednak podejrzewam że może prowadzić do sytuacji kiedy trzeba będzie się chwilę zastanowić jakiego typu
obiekt mamy w zmiennej. Oczywiście będą też przypadki w których kompilator nie domyśli się za nas jakiego typu ma być ta zmienna. Przykładowo próbując przypisać 
lambdę reprezentującą _Runnable_
dostaniemy błąd:

```java
var runable = () -> {}; // error: cannot infer type for local variable
``` 

Deklaracje zmiennych lokalnych, gdzie moglibyśmy uprościć sobie kod korzystając z _var_ to według twórców Javy ponad 90% kodu. Co ciekawe, jako alternatywę dla _var_ 
podano możliwość rozbudowy wyrażenia diament, tak żeby można było wykorzystać go z lewej strony operatora przypisania. Co dawałoby taki zapis:

```java
Map<> value = dataMap.get(amount);
``` 
 
## Po co czekać skoro jest Lombok

Rozwiązanie pozwalające pomijać podawanie typów podczas deklaracji zmiennych oferuje obecnie biblioteka _Lombok_. Oferuje ona dwa typy:

  * _val_ - zmienna final, której typ określany jest podczas kompilacji. Obecnie w wersji stabilnej biblioteki (od 0.10). 
  * _var_ - zmienna mutowalna,  której typ określany jest podczas kompilacji. Obecnie jeszcze jako experymentalny feature. Jeżeli _var_ wejdzie
   do języka w ramach _JEP 286_ to _Lombok_ z pewnościa usunie ją. Dostępna od wersji 1.16.12. 
   
Jeżeli chodzi o mutowalności _var_ w javie 18.3 to nadal trwa dyskusja wśród architektów na ten temat. Oczekując na kolejny release javy możemy ściągnąć _Lomboka_ 
i poeksperymentować z _val/var_, tak żeby znać wady i zalety tego podejścia kiedy już wejdzie oficjalnie do języka. Na koniec przykład wykorzystania _Lomboka_:   
   
```java
import lombok.val;
...
    val map = new HashMap<Integer, String>();
    map.put(0, "zero");
    map.put(5, "five");
    
    for (val entry : map.entrySet()) {
      System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
    }
...    
```   