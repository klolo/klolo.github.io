---
layout: post
title: "Programistyczne kata"
date: 2017-08-04 23:25:49 +0200
comments: true
categories: METHODOLOGY
---
Chcesz stać się lepszym programistą? Pisać lepszy kod? Powinieneś zainteresować się ćwiczeniami kata. Co tak w ogóle oznacza to słowo?
<!--more-->

<div style="text-align:center">
    <img src="http://totalcombat.net/wp-content/uploads/2012/12/shotokan-kata-1.jpg" />
</div>

Kata jest to nazwa zestawu technik ze sztuk walki, które wykonuje się setki razy, żeby osiągnąć perfekcję w każdej z tej technik. Każde kolejne
kata jest inne niż poprzednie. Za każdym razem dochodzi jakaś nowe kopnięcie, cios, którego wcześniej adept sztuk walki nie znał. Tak samo sprawa ma się
z kata programistycznym. Jest to proste ćwiczenie, którego realizacja powinna zając od kilkunastu minut do kilku godzin. Ma ono na celu pozwolić
programiście przećwiczyć nowy język, nową technikę i rozbudować umiejętności algorytmiczne. Oczywiście to czy programista skorzysta z czego nowego podczas
rozwiązywania takiego zadania zależy od niego. Jednak ciągłe wykonywanie kata korzystając z tych samych metod nie przyniesie tak dobrych wyników
jak ciągłe eksperymentowanie. Tak więc, jeżeli jesteś programistą pracującym na codzień z kodem obiektowym, spróbuj wykonywać kata w języku funkcyjnym.

Gdzie można znaleźć kata? Polecam odwiedzić https://www.codewars.com. Po założeniu konta mamy dostęp do kilku tysięcy kata, które możemy rozwiązywać w kilku językach.
Po rozwiązaniu problemu możemy porównać nasze rozwiązanie z rozwiązaniami innych użytkowników. Jak wygląda przykładowe kata z codewars? no tak:
> Write a function that takes an (unsigned) integer as input, and returns the number of bits that are equal to one in the binary representation of that number.
  Example: The binary representation of 1234 is 10011010010, so the function should return 5 in this case.

Do tak zdefiniowanego kata należy napisać kod który przechodzi takie testy jednostkowe:
{% highlight java %}
@Test
public void testGame() {
    assertEquals(1, BitCount.countBits(4));
    assertEquals(5, BitCount.countBits(1234));
    assertEquals(3, BitCount.countBits(7));
    assertEquals(2, BitCount.countBits(9));
    assertEquals(2, BitCount.countBits(10));
}
{% endhighlight %}

PS. Nie korzystaj z metody bitCount ;)