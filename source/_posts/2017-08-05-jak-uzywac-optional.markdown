---
layout: post
title: "Jak prawidłowo używać Optional w Java?"
date: 2017-08-05 19:57:49 +0200
comments: true
categories: JAVA
---
Jeśli programujesz w Java 8, to pewnie zdarzyło Ci sie użyć klasy Optional. Czy zrobiłeś to we właściwy sposób?
<!--more-->

Razem z JDK 8 dostaliśmy w ręce, strumienie i możliwość programowania funkcyjnego. Pojawił się również twór dzięki któremu mieliśmy już nie być tak często męczeni przez NullPointerException.
Tym tworem jest klasa Optional. Dla nie wtajemniczonych jest to generyczny pojemnik a zmienną dowolnego typu, która może mieć wartość null. Poprzez szereg metod dostępnych w klasie powinniśmy być w stanie
zabezpieczyć się przed przypadkowym odwołaniem do null referencji. Przyjrzyjmy się jakie błędy można popełnić używając Optional.

## Błędne użycie isPresent/get

Jednak zanim przejdziemy do tych metod, spójrzmy w jaki sposób tworzy się obiekty Optional. Można to zrobić za pomocą
jeden z metod:

  * _empty()_ - tworzy pusty Optional z wartością null w środku.
  * _of(T value)_ - tworzy Optional z podaną wartością. W przypadku przekazania null dostaniemy nullPointerException.
  * _ofNullable(T value)_ - również tworzy Optional z podaną wartością, ale w przypadku przekazania null nie zostanie zgłoszony wyjątek.
  * _Optional(T value)_ - konstruktor rzucający błąd w przypadku przekazania wartości null.

Dobrze, wiemy już jak utworzyć Optional to teraz pytanie jak go użyć? Zacznijmy od dwóch najczęściej używanych metod, często również nie poprawnie. Są to:

  * _isPresent()_ - zwraca boolean mówiący czy w środku znajduje się jakaś wartość czy też null.
  * _get()_ - pobranie przechowywanego obiektu. Jeżeli takiego nie dostaniemy: _NoSuchElementException_

No dobra, ja zatem może wyglądać użycie tych metod?

{% highlight java %}
private String saveTrim(final Optional<String> input) {
    if(input.isPresent()) {
        return input.get().trim();
    }

    return "";
}
{% endhighlight %}

No i tutaj mamy przykład jak nie używać Optional. Czy taki kod nie jest prościej zastąpić zwykłym warunkiem sprawdzającym czy przekazany String nie jest nullem? Tak klasycznie, jak to się
robi w Javie od lat 90, bez dodatkowego narzutu jakim jest Optional?

{% highlight java %}
private String saveTrim(final String input) {
   return input == null ? "" : input.trim();
}
{% endhighlight %}

Oczywiście można i tak, jednak Optional powstał po to żebyśmy nie musieli umieszczać w kodzie warunków sprawdzających czy referencja ma wartość null. Wykorzystajmy zatem
Optional w odpowiedni sposób i napiszmy kod bardziej funkcyjny:

{% highlight java %}
private String saveTrim(final Optional<String> input) {
    return Optional.ofNullable(input)
                    .map(String::trim)
                    .orElse("");
}
{% endhighlight %}

W powyższym przykładzie korzystamy z metody fabrykującej która nie wyrzuci wyjątku w przypadku przekazania null-a, następnie uruchamiamy metodę map która dla wartości null również zachowa się
stosownie i zwróci pusty Optional. No i na końcu mamy metodę orElse która zwróci nam domyślną wartość jeżeli Optional będzie przechowywał null.

## Przykład poprawnego wykorzystania Optional
Jeżeli poprzedni trywialny przykład was nie przekonał to spójrzmy na bardziej rozbudowaną logikę pełną zabezpieczeń przed nullem:

{% highlight java%}
private String getCompanyFirstUserName1(final Holding holding) {
        if (holding != null) {
            final Company company = holding.getCompanies().get(0);
            if (company != null && company.getUsers() != null) {
                final User user = company.getUsers().get(0);
                if (user != null && user.getFirstName() != null) {
                    final String result = user.getFirstName();
                    if (result.length() > 0) {
                        return result;
                    }
                }
            }
        }

        return "not found";
    }
{% endhighlight %}

Taki kod można zapisać za pomocą Optional w takiej formie:
{% highlight java%}
private String getCompanyFirstUserName2(final Holding holding) {
        return Optional.ofNullable(holding)
                .map(Holding::getCompanies)
                .map(Vector::firstElement)
                .map(Company::getUsers)
                .map(Vector::firstElement)
                .map(User::getFirstName)
                .filter(name -> name.length() > 0)
                .orElse("not found");
    }
{% endhighlight %}

Czyż taka wersja nie jest czytelniejsza? Krótsza? No i nie musieliśmy skorzystać z niebezpiecznej metody get, która może rzucić wyjątek.

## Niebezpieczna metoda get
Sam pomysłodawca i współ autor rozwiązania jakim jest Optional [żałuje](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type/26328555#26328555) że nie nazwał
tej metody np. _getOrElseThrowNoSuchElementException_, tak żeby każdy zawsze miał świadomość że może ona wyrzucić wyjątek i nie powinna być używana bez uprzedniego sprawdzenia w czy Optional
jest jakaś wartość.

## Gdzie nie używać Optional?
Kolejnym problemem jaki pojawia się tam gdzie używamy Optional jest nadużywanie go. W szczególności pola w klasach DTO nie powinny być deklarowane jako Optional, ponieważ może to doprowadzić do takich
dziwnych zapisów:
{% highlight java%}
// private HashMap<String, Integer> data;
   private Optional<HashMap<String, Integer>> data;
{% endhighlight %}
Jest to bardzo nie czytelne i nie potrzebne. Co w takim razie zrobić kiedy chcemy użyć Optional do zabezpieczenia naszego kodu? Możemy go utworzyć w getterze:
{% highlight java%}
private Optional<HashMap<String, Integer>> getData() {
    return Optional.ofNullable(data);
}
{% endhighlight %}
Ta sama zasada dotyczy również przekazywania Optionali do kontruktorów i do metod. Nic nam nie dają oprócz zaciemniania kodu. Lepszym rozwiązaniem jest przekazywać zwykły obiekt
i w środku metody zamieniać go na Optional. A! No i należy pamiętać o tym że Optional nie jest serializowany i nie powinien być używany w obiektach domenowych...No jest troszkę tych zapaszków
 (ang. code smells) związanych z kontrowersyjnym Optional. Na koniec przytoczę wyjaśnienie twórcy Optional-a do czego powstał Optional.
> Our intention was to provide a limited mechanism for library method return types where there needed to be a clear way to represent “no result”,
and using null for such was overwhelmingly likely to cause errors.