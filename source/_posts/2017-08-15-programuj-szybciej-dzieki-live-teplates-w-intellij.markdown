---
layout: post
title: "Programuj szybciej dzięki live teplates w Intellij"
date: 2017-08-15 17:01:03 +0200
comments: true
categories: JAVA
---
Obojętnie czy programujesz metodą Copiego i Pejsty, czy też wstukujesz swój kod niczym sekretarka pisząca notatkę ze spotkania szefa, możesz mieć wrażenie
że czasami zaczynasz od identycznego fragmentu kodu. Ile razu w życiu napisałeś `System.out.println` lub `public static void main(final String[] args)`? O ile nie jesteś programistą
utrzymującym przy życiu produkcyjnego dinozaura, czytając godzinami logi to pewnie zdarzyło Ci się kilka razy. Opowiem Ci jak intellij ułatwił moje życie
z takim powtarzającym się kodem.
<!--more-->

Pisząc kod zgodnie z regułami TDD, zaczynamy implementacje od napisania testu. Po otwarciu klasy z testami tworzymy zatem pusty test jednostkowy który za chwilę się zapełni kodem.
Pusty test jednostkowy zazwyczaj wygląda tak samo. U góry mamy adnotacje test, jakąś metodę `public void` z odpowiednią nazwę, w środku metody podział na sekcje give/when/then (o ile oczywiście taką mamy
konwencje).

{% highlight java %}
@Test
public void shouldDoSomething() {
    // given

    // when

    // then
    Assert.assertTrue(true);
}
{% endhighlight %}

Czy naprawdę za każdym razem muszę wpisywać taką funkcję ręcznie? Ewentualnie, czy muszę kopiować istniejący test i go dostosowywać? O tóż nie!
Intellij oferuje bardzo wygodne narzędzie o nazwie live template, dzięki któremu możemy tworzyć szablony kodu, które będą dostępne po wpisaniu odpowiedniego aliasu.

## Wbudowane szablony
Zanim opowiem w jaki sposób tworzyć własne szablony, zobaczmy jakie wbudowane mamy do dyspozycji oraz jak z nich korzystać. Jak już wspomniałem każdy szablon ma swój alias, po wpisaniu którego
w menu automatycznego wypełniania pojawia się szablon o podanej nazwie. Po wybraniu szablonu, w miejscu w którym znajduje się szablon pojawi się wybrany szablon. Jednym z wbudowanych szablonów
jest _sout_, który wstawia instrukcje wypisania na konsoli: _System.out.println_.

![GitHub Logo](/images/sout.png)

Oczywiście każdy szablon związany jest również z miejscem w którym może wystepować. Do wyboru możliwe jest kilka zakresów:

 * `psvm` = `public static void main() { }`
 * lore

## Tworzenie własnych szablonów
lorem ipsum

## Bardziej skomplikowane szablony
lorem ipsum


## Podsumowanie
Eclipse również udostępnia podobną opcję.
