---
layout: post
title: "Antora-narzędzie do dokumentacji mikroserwisów"
date: 2022-03-29 21:31:32 +0100
comments: true
categories: OTHERS
---

# Dokumentacja techniczna dla mikroserwisów

## wstęp

Jedną z zalet architektury opartej na mikro serwisach jest nie zależność zespołów w procesie wytwarzania i wdrażania zmian. Rzeczą, która jest z tym powiązana jest rozbicie systemu, który tworzymy na wiele projektów, czasami są to nawet setki projektów na git. Serwisy i biblioteki, które wytwarzamy składają się na system, w którym na porządku dziennym jest budowanie nowych funkcjonalności w opartciu o istniejące już kody. Skąd jednak wiadomo jak działa kod, który chcemy wykorzystać?  Skąd informację o projekcie ma czerpać programista który dopiero w projekt się wdraża? Samodokumentujący się kod jest zazwyczaj tylko jednorożcem, o którym każdy słyszał a nikt nie widział. Dodatkowo istnieją też informacje, które nie pasują do javadoc a są przydatne.
Jedną z zalet architektury opartej na mikro serwisach jest nie zależność zespołów w procesie wytwarzania i wdrażania zmian. Rzeczą, która jest z tym powiązana jest rozbicie systemu, który tworzymy na wiele projektów, czasami są to nawet setki projektów na git. Serwisy i biblioteki, które wytwarzamy składają się na system, w którym na porządku dziennym jest budowanie nowych funkcjonalności w opartciu o istniejące już kody. Skąd jednak wiadomo jak działa kod, który chcemy wykorzystać?  Skąd informację o projekcie ma czerpać programista który dopiero w projekt się wdraża? Samodokumentujący się kod jest zazwyczaj tylko jednorożcem, o którym każdy słyszał a nikt nie widział. Dodatkowo istnieją też informacje, które nie pasują do javadoc a są przydatne.

Nie oceniona jest w takich przypadkach dokumentacja techniczna obrazująca jak dany projekt skonfigurować, uruchomić, za jest odpowiedzialny, jaka jest struktura kodu etc. Najbardziej pospolitym podejściem jest umieszczenie pliku README.md w katalogu głównym projektu. Jest to rozwiązanie proste, ale niekoniecznie sprawdza się w systemie złożym z dużej ilości projektów.  Bo co w przypadku gdy nie wiemy, w jakim dokładnie projekcie szukać jakiegoś mechanizmu?  Można pokusić się o przechowywanie dokumentacji w jakimś zewnętrznym narzędziu np. Wiki, Confluence, które pozwoli nam na wygodne przeszukiwanie. To podejście jednak ma tą wadę że developer w celu aktualizacji dokumentacji musi wyjść z swojego IDE i otworzyć dodatkowe narzędzie, jakim jest przeglądarka. W dzisiejszej modzie na X as a code jest to troche niechciane podejście.

Chciałem się z wami podzielić narzędziem, dzięki któremu nadal możemy trzymać dokumentację techniczną tam gdzie powinna być, czyli blisko kodu a jednocześnie rozwiązuje problem wyszukiwania informacji na wielu repozytoriach. Tym narzędziem jest Antora. Jest to aggregator dokumentacji z projektów na git. Po zaagregowaniu dokumentacji tworzony jest wynikowy html, którego możemy wystawić na jakiś serwerze i nawet osoby niemające dostępu do kodów mogą zapoznać się z dokumentacją.

## agregowanie dokumetacji

TODO

## custom UI

TODO

## diagramy as a code

TODO

## kolorowanie składni

TODO

## podsumowanie

TODO
