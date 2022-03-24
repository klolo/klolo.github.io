---
layout: post
title: "Intellij live templates - szybsze programowanie"
date: 2017-08-15 17:01:03 +0200
comments: true
categories: JAVA
---
Niezależnie od tego czy programujesz metodą Copiego i Pejsty, czy też wstukujesz kod niczym sekretarka pisząca notatkę ze spotkania szefa, czasem możesz odnieść wrażenie, że zaczynasz od identycznego fragmentu kodu. Ile razu w życiu na początku było `System.out.println` lub `public static void main(final String[] args)`? O ile nie jesteś programistą utrzymującym przy życiu produkcyjnego dinozaura, to pewnie zdarzyło Ci się kilka razy. Intellij może ułatwić Twoje życie.
<!--more-->

Pisząc kod zgodnie z regułami TDD, zaczynamy każdą implementację od napisania testu. Po otwarciu klasy z testami tworzymy zatem pusty test jednostkowy, który za chwilę zapełni się kodem. Pusty test jednostkowy zazwyczaj wygląda podobnie jak setki innych w projekcie - u góry adnotacje "test", jakaś metoda `public void` z odpowiednią nazwą, w środku metody podział
na sekcje give/when/then (o ile oczywiście taką mamy konwencję), no i jakaś assercja.

```java
@Test
public void shouldDoSomething() {
    // given

    // when

    // then
    Assert.assertTrue(true);
}
```

Za każdym razem wpisuję taką funkcję ręcznie. Czy to naprawdę konieczne? Czy trzeba kopiować istniejący test i dopiero czyścić go z tego, czego nie potrzebuje? Otóż nie! Intellij oferuje bardzo wygodne narzędzie o nazwie "Live template", dzięki któremu można tworzyć szablony kodu, które będą dostępne po wpisaniu odpowiedniego aliasu.

## Wbudowane szablony

Jakie wbudowane szablony są do dyspozycji oraz jak z nich korzystać? Każdy szablon ma swój alias, po wpisaniu którego
w menu automatycznego wypełniania pojawia się szablon o podanej nazwie. Po wybraniu szablonu, w miejscu, w którym znajduje się kursor pojawi się wybrany szablon. Jednym z wbudowanych szablonów jest _sout_, który wstawia instrukcję wypisania na konsoli: _System.out.println_.

![GitHub Logo](/images/sout.png)

Innymi dostępnymi domyślnie szablonami dla Javy są:

 * `psvm` - wstawia pustą metodę main.
 * `geti`- tworzy metodę do pobierania instancji singletona getInstance
 * `ifn` - warunek sprawdzający czy zmienna nie nullem, przeciwieństwo `inn`
 * `inst` - warunek sprawdzający czy zmienna jest instancją klasy za pomocą instanceof
 * `lazy` - sprawdza czy zmienna jest nullem i jeżeli jest to tworzy jej instancję.
 * `lst` - wyciąga ostatni element tablicy
 * `mn` - wywołuję funkcję _Math.min_, przeciwieństwo `mx`
 * `toar` - zamienia kolekcję na tablicę
 * `prsf` - _private static final_
 * `psf` - _public static final_
 * `psfi` - _public static final int_
 * `psfs` - _public static final String_
 * `St` - _String_
 * `thr` - _throw new_

Oczywiście każdy szablon związany jest również z miejscem, w którym może występować. Przykładowo dla Javy do wyboru możliwe jest kilka zakresów:

  * Statement
  * Expression
  * Declaration
  * Comment
  * String
  * Smart type completion
  * Other

  Intellij na podstawie fragmentu kodu, w którym aktualnie znajduje się kursom określi z jakiego zakresu zaproponować szablony. Zakresy różnią się w zależności od języka oraz od frameworka, w którym chcemy wykorzystać szablon.

## Surround templates
Oprócz szablonów wstawianych w danym miejscu kodu, mamy do dyspozycji szablony otaczające kod. Szablony takie działają na zasadzie otaczania zaznaczonego kodu jakimś predefiniowanym szablonem, przykładowo blokiem synchronized. Z takiego szablonu można skorzystać po zaznaczeniu fragmentu kodu i wybraniu skrótu klawiszowego _Ctrl-Alt-J_ oraz _Ctrl-Alt-T_, który wyświetli również kilka przydatnych szablonów wbudowanych w IDE, takie jak np. otoczenie kodu w blok try-catch.

![GitHub Logo](/images/templatesMenu.png)

## Tworzenie własnych szablonów
Szablon można utworzyć wchodząc w ustawieniach IDE w zakładkę Live templates i wybierając przycisk plusika z prawej strony. Jest tam również opcja dodania grupy szablonów, w przypadku kiedy chcemy je w jakiś logiczny sposób zgrupować. Podczas tworzenia szablonu należy podać jego alias, poprzez który będziemy się do niego odwoływać, opis, kod szablonu oraz kontekst, w którym ma być dostępny. Proste, prawda? Nie do końca jeżeli spojrzy się na przykłady wbudowanych w Intellij szablonów. Występują tam różnego rodzaju zmienne, dzięki którym kod nie jest tylko wpisanym na sztywno tekstem, ale faktycznie szablonem na podstawie, którego utworzy się fragment przydatnego kodu. Oprócz zmiennych, mamy nawet dostęp do kilku predefiniowanych funkcji, które można wywoływać w szablonie. Lista zmiennych i metod dostępnych w Live templates różni się w zależności od zainstalowanych pluginów. Te najbardziej podstawowe to:

  * $END$ - określa miejsce, w którym ma znaleźć się kursor po wstawieniu szablonu
  * $SELECTION$ - w przypadku szablonu otaczającego kodu zmienna zawiera zaznaczony przez użytkownika tekst.

Mamy również do dyspozycji funkcje, które pozwolą jeszcze bardziej rozbudować szablon. Kilka, według mnie, najbardziej przydatnych:

  * `camelCase(String)` - zwraca podany tekst jako camel case.
  * `methodName()` - nazwa metody, w której się znajdujemy
  * `date(sDate)` - zwraca datę w podanym formacie
  * `currentPackage()` - aktualny pakiet

Funkcji nie możemy jednak wywoływać bezpośrednio w szablonie. W szablonie odwołujemy się do zmiennych, które przechowują wartość wywołań funkcji. Zmienne mogą mieć dowolną nazwę, jednak trzeba im później przypisać wywołanie funkcji w okienku dostępnym po wybraniu _Edit variables_. Przykład szablonu tworzącego zmienną przechowującą nazwę funkcji oraz logującego jakiś komunikat do logów. Komunikat ten można podać po wstawieniu loggera, z powodu zmiennej _$END$_.

![GitHub Logo](/images/newLiveTemplate.png)

## Podsumowanie
Kod można pisać w bardzo szybki sposób, nie korzystając przy tym z kopiowania i wklejania. Myślę, że jest to podejście, które sprawi, że kod będzie miał mniej błędów. Ile razy zdarzyło Ci się przenieść jakiś fragment kodu z innego miejsca systemu i naciąć się na to, że nie działa? A wystarczyłoby napisać go ręcznie i wyłapać błędy. Dbajmy o to, aby nasza praca była wydajniejsza i lepsza. Tak na marginesie, Eclipse również udostępnia podobną opcję, nazywa się to Templates, jednak nie potrafię określić na ile różni się to z Live Templates dostępnymi w Intellij. Oprócz Live templates Intellij posiada również mechanizm szablonów kodu, który w menu znajdziecie jako File and code templates. Szablony te są dostępne w menu pojawiającym się po wciśnięciu alt+insert. Jest tam nawet pusty szablon metody testowej JUnit ;)

Wszystkie informacje na temat Live templates znajdziecie [tutaj](https://www.jetbrains.com/help/idea/live-templates.html)


