---
layout: post
title: "Typescript wprowadzenie - część 1"
date: 2017-08-28 22:11:51 +0200
comments: true
categories: TYPESCRIPT
---
Jesteś programistą javascript chcącym pisać aplikację z wykorzystaniem Typescript? Chcesz nauczyć się Angular 2/4? Jesteś programistą Javy, który próbuje tworzyć frontend?
Doskonale trafiłeś. W serii kilku wpisów postaram się przybliżyć Ci ten język. Dzisiaj opowiem o tym czym jest ts, jak zacząć z nim pracę oraz jakie podstawowe typy
są dostępne w tym języku.

<!--more-->

## Co to jest typescript?

Krótko mówiąc Typescript jest nadzbiorem języka javascript stworzonym prze Microsoft w celu dodania do js takich elementów jak statyczne typowanie. Jest to rozszerzenie języka
javascript, stąd też każdy prawidłowy kod języka js jest również prawidłowym kodem ts. Nie działa to jednak w drugą stronę, ponieważ ts jest nie obsługiwany przez silnik js wbudowany
np. w przeglądarkę. Ts przed uruchomieniem musi zostać zamieniony na js przez specjalny kompilator o nazwie _tsc_. Proces przepisywania kodu napisanego w jednym języku na inny nosi nazwę transpilacji.

## Hello World

W celu rozpoczęcia zabawy z ts należy zainstalować Node.js i za pomocą managera pakietów npm zainstlować kompilator:
> npm install -g typescript

Po wykonaniu polecenie będziemy mogli uruchomić hello world zapisane w pliku demo.ts:

{% highlight javascript %}
function getHello(userName: string): string {
    return `Hello ${userName}`;
}

let hello: string = getHello('Kamil');
console.log(hello);
{% endhighlight %}

W celu skompilowania kodu ts do js uruchamiamy polecenia:
> tsc ./demo.ts && node ./demo.js

Jak widać w powyższym poleceniu, zostanie wygenerowany plik js który będzie zawierał wygenerowany kod js:
{% highlight javascript %}
function getHello(userName) {
    return "Hello " + userName;
}
var hello = getHello('Kamil');
console.log(hello);

{% endhighlight %}

Wszystkie fragmentu kodu związane z ts, nie będące poprawnym js znikły. Środowisko w którym będziemy uruchamiać taki kod nie będzie miało żadnej informacji
o tym że kod był tworzony w ts.

## Statyczne typowanie

Jak wiadomo js jest językiem dynamicznie typowanym, co oznacza że informacja o tym jakiego typu obiekt jest przechowywany w zmiennej określany jest w trakcie działania
aplikacji. Do zmiennej możemy w jednej linijce przypisać jakiś string, a w następnej obiekt całkowicie innego typu:

{% highlight javascript %}
let user = 'Karol';

console.log(user);

user = {
    name: 'Karol',
    age: 18
};

console.log(user);
{% endhighlight %}

Nie ma tutaj żadnego mechanizmu który sprawdzi czy w zmiennej jest obiekt prawidłowego typu, jaki jest wymagany w danej funkcji. Jest to mechanizm który sprawia że js jest
bardzo elastyczny, ekspresyjny ale zarazem jest to przyczyna wielu błędów, często trudnych do wychwycenia. Typescript daje programiście możliwość definiowania typów zmiennych, parametrów funkcji
dzięki czemu kompilator w czasie transpilacji kodu ma możliwość weryfikacji czy nie popełniliśmy błędu i nie próbujemy użyć błędnego typu w danym miejscu aplikacji. Sposób wykorzystania typów
w ts przedstawia również przykład z _hello world_. Składnia ts wymaga od nas jedynie podania po nazwie zmiennej dwukropka
po którym będzie stać typ zmiennej. W celu określenia typu zwracanego z metody należy dwukropek i typ zwracany podać miedzy nawiasem zamykającym listę argumentów a klamrą
otwierającą ciało funkcji.


## Typy proste
Do dyspozycji mamy podstawowe typu danych znane z js:

* boolean - true/false
* number - liczba zmiennoprzecinkowa
* string - ciąg znaków
* null - brak wartości zmiennej, która została zainicjalizowana
* undefined - wartość domyślna zmiennych, mówiąca o tym że zmienna nie została zainicjalizowana

Przy korzystaniu z typów prostych takich jak np. string należy jednak zwracać uwagę czy nie piszemy nazwy typu z dużej litery. Obiekty do których byśmy się wtedy odwoływali są
wrapperami na typy proste. Jest to mechanizm podobny do tego w Java. Używanie wrapperów zamiast typów prostych sprawi że nasz kod będzie mniej wydajny.

Dodatkowo typescript dodaje możliwość korzystania z kilku dodatkowych typów, które warto wyjaśnić.

## any
Przechodząc z js na ts możemy robić to stopniowo, nie musimy przepisywać od razu całej aplikacji. Może się jednak pojawić problem z tym że istniejący kod zwraca obiekty
jakiegoś typu i ts nie może zweryfikować co to za typ. W takich przypadkach przydatny jest specjalny typ _any_ pozwalający przypisać do zmiennej obiekt dowolnego typu, bez walidowania tego typu.
Możecie zapytać dlaczego nie możemy wykorzystać _Object_ jako typu dla danych które przechodzą z jakiejś np. biblioteki? Świetnie to wyjaśnia przykład kodu wprost z dokumentacji ts:

{% highlight javascript %}
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)
 
let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
{% endhighlight %}

Oczywiście należy pamiętać o tym żeby zmiennych typu _any_ w kodzie było jak najmniej, a nawet żeby ich nie było w ogóle. Ponieważ korzystając z _any_ tracimy dobrodziejstwa statycznego
typowania jakie daje nam ts. Kompilator można uruchomić z flagą, która sprawi że kod który zawiera _any_ nie skompiluje się.

## void
Typ ten jest wykorzystywany do informowania kompilatora o tym że funkcja nie będzie nic zwracać. Zachowanie analogiczne do tego znanego z np. Javy. Możemy użyć instrukcji
return, ale tylko gdy nie będzie ona zawierać żadnych danych które mają zostać zwrócone. Funkcja może zwracać jedynie null, undefined bądź nic. Używanie void w ts zabezpiecza nas przed
przypadkowym zwróceniem jakiejś wartości z funkcji, bądź próbą przypisania wyniku takiej funkcji do zmiennej typu innego niż void.

{% highlight javascript %}
function voidFun(): void {
    return null; // ewentualnie undefined
}

let x: void = voidFun();
console.log(x);
{% endhighlight %}

## never
Never jest bardzo ciekawym typem prostym wbudowanym w ts. Oznacza się nim funkcje które nic nie mogą zwrócić. Jednak w przeciwieństwie do void, taka funkcja nie może
ani posiadać instrukcji return ani się kończy w prawidłowy sposób. Metody oznaczone jako never mogą w środku zawierać jedynie nieskończone pętle, bądź też wyrzucać wyjątek,
zamiast normalnego wyjścia z metody.

{% highlight javascript %}
function neverFun(): never {
    while (true) {
    }
}

function neverFunWithThrow(): never {
    console.log('neverFunWithThrow');
    throw new Error('this is never enging function');
}
{% endhighlight %}

## tuples
Jest to konstrukcja pozwalająca w tablicy umieścić pewną ilość zmiennych, których typ jest znany podczas definiowania tablicy. Semantycznie wygląda to tak że deklarujemy
tablice podając typ każdego z elementów tej tablicy. Kompilator sprawdzi za nas czy w poszczególne komórki tablicy przypisujemy prawidłowe typy oraz czy wyciągnięte dane
z tablicy są prawidłowo użyte.

{% highlight javascript %}
const tupleUser: [string, number] = ['Kamil', 18];
tupleUser[0] = 1; // error
tupleUser[0] = 'Bartek'; // ok

const userName: string = tupleUser[0].toLowerCase();
const user: string = tupleUser[1]; // error
{% endhighlight %}

## enum
Często zdarza się że w aplikacji mamy kilka stałych, które mają pewien wspólny kontekst. W czystym js można było je zgrupować za pomocą np. odpowiednich nazw, bądź
jako statyczne pola użyte razem z konstruktorem:

{% highlight javascript %}
function Color() { }

Color.RED = 'RED';
Color.GREEN = 'GREEN';
Color.BLUE = 'BLUE';
{% endhighlight %}

Wadą takiego rozwiązania jest to że wartości tych stałych mogą się zmienić dynamicznie w trakcie działania aplikacji. Oczywiście js pozwala nam obejść ten problem za pomocą
bardziej zaawansowanych technik, jednak rozwiązania takie będą już skomplikowane. Ts pozwala używać enumów, dzięki którym możemy zgrupować zmienne bez obawy przed tym że ich wartość zmieni się w trakcie działania aplikacji. Dodatkowo kompilator sprawdzi czy w miejsce gdzie ma być przekazana stała z enuma przekazujemy poprawną wartość. Wartościom stałych można ustawiać dowolne wartości. Jeżeli pominiemy ustawianie wartości to stałe w enum będą miały wartości kolejno: 0, 1, 2 itd.

{% highlight javascript %}
enum ColorRGB {
    RED =  'red',
    GREEN = 'green',
    BLUE = 'blue'
}

ColorRGB.RED = 'green'; // error

const red: ColorRGB = ColorRGB.RED;
{% endhighlight %}

## Podsumowanie
Typescript jest językiem programowania bazującym na javascript, który pozwala nam w bardziej bezpieczny sposób implementować aplikację. Statyczne typowanie, zapewni że
to właśnie kompilator znajdzie błędne użycie typów w momencie kompilacji a nie klient na środowisku produkcyjnym. Ts jest językiem obiektowym, pozwalającym używać wbudowanych
typów oraz definiować nowe. Jak to robić, opowiem w kolejnym wpisie.