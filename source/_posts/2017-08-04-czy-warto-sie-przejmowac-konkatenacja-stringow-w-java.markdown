---
layout: post
title: "Czy warto się przejmować konkatenacją stringów w Java?"
date: 2017-08-04 20:37:42 +0200
comments: true
categories: JAVA
---
Używacie StringBuilder do łączenia Stringów? Każdy z nas spotkał się z stwierdzeniem mówiącym że w celu optymalizacji operacji łączenia stringów należy używać StringBuilder bądź też StringBuffer
bezpiecznego wielowątkowo. Czy na pewno słusznie?
<!--more-->

Co nam daje StringBuilder? A no jak wiadomo string jest obiektem nie zmiennym, nie da się dopisać do niego kolejnych znaków. Za każdym razem kiedy chcemy utworzyć dłuższy ciąg znaków tworzony
jest nowy obiekt. StringBuilder przechowuje poszczególne znaki w tablicy, do której podczas każdej operacji append dodawane są nowe dane. Dopiero w momencie wywołania metody toString zostaje
utworzony finalny string zawierające wszystkie znaki z tymczasowej tablicy. Jeżeli konkatenacja wykonywana jest w pętli, jak tutaj:
{% highlight java %}
  public static void main(final String... args) {
        String result = "";
        for (int i = 0; i < 100; i++) {
            result += i + " ";
        }

        System.out.println(result);
    }
{% endhighlight %}

to za każdym obiegiem pętli tworzymy w pamięci nie potrzebny obiekt coraz dłuższego Stringa. To oczywiście zajmuje nie tylko pamięć ale również czas na alokacje danych.
Dlatego każdy z nas doskonale wie że taka pętla powinna wyglądać mniej więcej tak:
{% highlight java %}
  public static void main(final String... args) {
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < 100; i++) {
            builder.append(i).append(" ");
        }

        System.out.println(builder.toString());
    }
{% endhighlight %}

Dzięki użyciu StringBuilder optymalizujemy nasz kod. Jednak takie rozwiązanie wymaga troszkę więcej pisania i czasami może zaciemniać kod. Czy na pewno musimy taki kod pisać?
Otóż nie!. Dokumentacja mówi:
>An implementation (of compiler) may choose to perform conversion and concatenation in one step to avoid creating and then discarding an intermediate String object. 
To increase the performance of repeated string concatenation, a Java compiler may use the StringBuffer class or a similar technique to reduce the number of intermediate 
String objects that are created by evaluation of an expression. For primitive types, an implementation may also optimize away the creation of a wrapper object by converting
directly from a primitive type to a string.

Swoją drogą nie wiem dlaczego ten zapis jest mały literami :P Wynika z tego że kompilator potrafi sam wstawić StringBuilder w miejsce konkatenacji stringów. Sprawdźmy!
Do zdekompilowaniu kodu z listingu 1 poleceniem javap -c Demo.class dostaje:
{% highlight java %}
  public static void main(java.lang.String...);
    Code:
       0: ldc           #2 // String
       2: astore_1
       3: iconst_0
       4: istore_2
       5: iload_2
       6: bipush        100
       8: if_icmpge     41
      11: new           #3 // class java/lang/StringBuilder
      14: dup
      15: invokespecial #4 // Method java/lang/StringBuilder."<init>":()V
      18: aload_1
      19: invokevirtual #5 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: iload_2
      23: invokevirtual #6 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      26: ldc           #7 // String
      28: invokevirtual #5 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      31: invokevirtual #8 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      34: astore_1
      35: iinc          2, 1
      38: goto          5
      41: getstatic     #9 // Field java/lang/System.out:Ljava/io/PrintStream;
      44: aload_1
      45: invokevirtual #10 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      48: return
}
{% endhighlight %}

Faktycznie w wygenerowanym byte code pojawił się magicznie string builder. A jak w takim razie wygląda kod z listingu 2 gdzie jawnie użyłem StringBuildera?

{% highlight java %}
  public static void main(java.lang.String...);
    Code:
       0: new           #2 // class java/lang/StringBuilder
       3: dup
       4: invokespecial #3 // Method java/lang/StringBuilder."<init>":()V
       7: astore_1
       8: iconst_0
       9: istore_2
      10: iload_2
      11: bipush        100
      13: if_icmpge     33
      16: aload_1
      17: iload_2
      18: invokevirtual #4 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      21: ldc           #5 // String
      23: invokevirtual #6 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      26: pop
      27: iinc          2, 1
      30: goto          10
      33: getstatic     #7 // Field java/lang/System.out:Ljava/io/PrintStream;
      36: aload_1
      37: invokevirtual #8 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      40: invokevirtual #9 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      43: return
}
{% endhighlight %}

Wygląda niemal identycznie. Nie znam się na tyle na byte code żeby stwierdzić czy różnice w tych kilku instrukcjach wpłyną znacząco na wydajność,
ale uważam że kompilator robi tutaj za nas świetną robotę i nie warto na siłe optymalizować każdej pętli gdzie łączymy stringi. Problem może być w Javie starszej niż 1.5, bo tam tego mechanizmu
nie ma ;)