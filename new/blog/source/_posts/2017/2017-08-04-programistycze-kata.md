---
layout: post
title: "Programistyczne kata"
date: 2017-08-04 23:25:49 +0200
comments: true
categories: METHODOLOGY
---
Chcesz pisać lepszy kod? Zainteresuj się ćwiczeniami kata.

<!--more-->

<div style="text-align:center">
    <img src="https://www.martialtribes.com/wp-content/uploads/2018/05/kata1-1-1024x512.jpg" />
</div>

Co w ogóle oznacza to słowo?


Kata to nazwa zestawu technik ze sztuk walki, które wykonuje się setki razy, żeby osiągnąć perfekcję w każdej z nich. Każde kolejne kata jest inne niż poprzednie. Za każdym razem dochodzi jakaś nowe kopnięcie, cios, którego wcześniej adept sztuk walki nie znał. Tak samo sprawa ma się z kata programistycznym. Jest to proste ćwiczenie, którego realizacja powinna zając od kilkunastu minut do kilku godzin. Ma ono na celu przećwiczenie nowego języka, nowej techniki i rozbudowanie umiejętności algorytmicznych. Oczywiście to czy skorzystasz z czegoś nowego podczas rozwiązywania takiego zadania zależy od Ciebie. Jednak wykonywanie kata korzystając ciągle z tych samych metod, nie przyniesie tak dobrych wyników jak eksperymentowanie, szukanie nowych rozwiązań. Tak więc, jeżeli pracujesz na co dzień z kodem obiektowym, spróbuj wykonywać kata w języku funkcyjnym.

Gdzie można znaleźć kata? Polecam odwiedzić https://www.codewars.com. Po założeniu konta dostaniesz dostęp do kilku tysięcy kata, które można rozwiązywać w kilku językach. Po rozwiązaniu problemu jest także opcja porównania rozwiązań z innymi użytkownikami. Jak wygląda przykładowe kata z codewars? Na przykład tak: 

> Write a function that takes an (unsigned) integer as input, and returns the number of bits that are equal to one in the binary representation of that number. Example: The binary representation of 1234 is 10011010010, so the function should return 5 in this case.

Do tak zdefiniowanego kata należy napisać kod, który przechodzi takie testy jednostkowe:

```java
@Test
public void testGame() {
    assertEquals(1, BitCount.countBits(4));
    assertEquals(5, BitCount.countBits(1234));
    assertEquals(3, BitCount.countBits(7));
    assertEquals(2, BitCount.countBits(9));
    assertEquals(2, BitCount.countBits(10));
}
```

PS. Nie korzystaj z metody bitCount ;)