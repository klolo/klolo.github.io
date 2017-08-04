---
layout: post
title: "Wstęp do programowania funkcyjnego"
date: 2017-08-04 20:25:49 +0200
comments: true
categories: 
---
Artykuł ma na celu wyjaśnienie czym jest programowanie funkcyjne oraz przedstawienie podstawowych zasad panujących w świecie programowania funkcyjnego.

Programowanie deklaratywne vs imperatywne


Wprowadzenie do programowanie funkcyjnego zacznę od przypomnienia czym jest paradygmat programowania deklaratywnego. Jest to podejście w którym tworząc
 kod skupiamy na się na tym co ma zostać wykonane, a nie w jaki sposób. Szczegóły implementacji pozostawiamy np. bibliotece, odwracając w ten sposób
  kontrolę nad kodem. W końcu wszyscy lubią odwracanie kontroli (inversion of control) ;). Tak napisany kod jest bardziej zwięzły iczytelniejszy z powodu 
  mniejszej ilości zbędnego szumu. Ile to razy zdarzyło Ci się napisać: for(int i=0; i<..., pewnie setki razy. Czyż nie uważasz że w większości przypadku 
  tak napisany kod jest złem koniecznym , niezbędnym do osiągnięcia określonego działania aplikacji? Podejście w którym wykorzystywane są instrukcje sterujące
   naszym kodem nazywane jest programowaniem imperatywnym. Jest to przeciwieństwo programowania deklaratywnego, gdzie nie tylko musimy martwić się co ma robić nasz kod,
    ale również w jaki sposób to osiągnąć. Programowanie funkcyjne idzie w parze z programowanie deklaratywnym, dzięki czemu nasz kod będzie krótszy niż ten klasyczny 
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

Co to jest programowanie funkcyjne?


Programowanie funkcyjne to programowanie w którym najważniejszą rolę pełnią funkcje. Pewnie zapytasz: Zaraz, zaraz...Czy czasami mój obiektowy kod nie składa się 
również z funkcji? Ależ tak! Jednak zasadnicza równica jest taka że w FP (ang. Functional programming) funkcje służą do opisu przepływu sterowania i operacji, a nie
 tak jak to ma miejsce w przypadku programowania obiektowego do zmiany stanu obiektów. Z tego założenia wynika również jedna z cech programowania obiektowego - nie
  zmienność obiektów. Czy jeszcze się charakteryzuje FP? O tym w dalszej części.
Czyste funkcje

Czyste funkcje są to funkcje bez efektów ubocznych. Są one elementem bez którego nie da się pisać funkcyjnie. Nie zależą one od żadnych zmienny zewnętrznych np. pól w 
klasie, a wszystkie dane są przekazywane jako parametr/y. Dlatego też dla podanego zestawu danych czysta funkcja zawsze zwróci taki sam wynik. Zaburzeniem tego może być 
również wykorzystanie w funkcji instrukcji? random, gdzie wynik operacji nie jest stały dla danych parametrów. Czyste funkcje również nie modyfikują stanów zewnętrznych 
np. pól w klasie, lecz każdy wynik jest zwracany jako wynik funkcji.
Funkcje jako obywatele pierwszej klasy


Pod tym pojęciem kryje się możliwość języka do traktowania funkcji na równi z obiektami klas tzn. funkcje można przekazywać jako parametry do metod, można je przypisywać 
do zmiennych oraz tworzyć wewnątrz funkcji. Przed Java 8, język nie był wyposażony w elementy które na to pozwalały.
Niezmienność obiektów

Ta cecha FP jest związana z czystymi funkcjami. W językach czysto funkcyjnych często nie istnieje pojęcie zmiennej, wszystkie tworzone obiekty są niezmienne. Dzięki 
takiemu podejściu nie ma możliwości zmieniać stanów ani przekazanych do funkcji parametrów jak i pól w klasie, trudniej jest w funkcji wykonać efekt uboczny. Tak więc
final jest zawsze mile widziany. Jak ułatwić sobie życie w takim kodzie? Polecam bardzo buildery, czy to te implementowane przez was, czy też te generowane automatycznie 
np. za pomocą Lombok-a. 
Zalety programowania funkcyjnego

Dzięki temu że funkcje nie mają efektów ubocznych, wynik działania jest łatwiejszy do przewidzenia jak i przetestowania. Dodatkowo czysta funkcja dla podanych parametrów 
zawsze zwróci ten sam wynik, dlatego można z łatwością wykorzystać cache w celu optymalizacji aplikacji. Dzięki używaniu czystych funkcji znikną również największe problemy 
z wielowątkowością, takie jak jednoczesna modyfikacja wspólnych danych przez kilka wątków. Możemy skupić się również na tym co mają robić nasze funkcje, a nie w jaki sposób 
implementować np. pętle lub warunek. Łatwiej jest nam również reużywać napisany wcześniej kod. Jakie są natomiast zalety tego że  nasze obiekty są niezmienne? Łatwiej 
przewidzieć co się w nich znajduje, dzięki czemu ewentualne błędy szybciej się diagnozuje oraz poprawia.