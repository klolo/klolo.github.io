---
layout: post
title: "Dependency injection w js"
date: 2018-02-18 10:22:30 +0100
comments: true
categories: javascript
---
Czy do wstrzykiwania zależności w javascript potrzebny jest framework? Czy jest jakiś wbudowany mechanizm języka który pozwala nam to robić
w łatwy sposób? Jest!
<!--more-->

W javascript programowanie obiektowe możliwe było przez lata dzięki funkcjom i dziedziczeniu przez prototypy. Syntax sugar w postaci klas jakie pojawiły
sie razem z ES6 nie zmieniają tego. Nadaj pod spodem wykorzystywane są prototypy. Warto więc poznać możliwości jakie nam dają. Zacznijmy od prostego przykładu
w którym za pomocą funkcji definiujemy obiekt Engine:

{% highlight javascript %}
function Engine(model) {
    this.model = model;

    this.start =  function () {
        console.log(`start engine: ${this.model}`)
    };
}

const engine = new Engine('v8');
engine.start();
{% endhighlight %}

Po zdefiniowaniu funkcji, wykorzystuje ją do stworzenia obiektu w którym znajdą się zarówno dane jak i logika. Jak widać bez użycia klas mamy programowanie obiektowe.
Funkcje które są używane w js razem z _new_ nazywane są konstruktorami i zwyczajowo ich nazwa zaczyna się z dużej litery. Ok, idźmy dalej. Warto zwrócić uwagę na to że
jeżeli zmienimy funkcje _start_ na lambdę (arrow function) to w dalszej części nie będziemy mieli dostępu do zmiennej model. Wynika to w różnicy w bindowaniu
kontekstu this. Dla _function_ this domyślnie odnosi się do miejsce gdzie funkcja jest wywoływana a dla _arrow_ będzie to miejsce gdzie funkcja została zadeklarowana.
Zdefiniujmy teraz silnik Diesel który będzie dziedziczył po Engine i zawierał dodatkową logikę:

{% highlight javascript %}
function Diesel(model) {
    this.model = model;

    this.checkIsOk = function () {
        console.log('is ok');
    }
}

Diesel.prototype = new Engine();
{% endhighlight %}

Po zdefiniowaniu funkcji Diesel do prototypu podpinam obiekt Engine. Dzięki czemu obiekt typu Diesel będzie miał wszystkie funkcje z obiektu Engine. Widać to na przykładzie:

{% highlight javascript %}
const diesel = new Diesel('vw');
diesel.checkIsOk();
diesel.start();
{% endhighlight %}

Jak widać bez użycia klas tworzymy kod wykorzystujący paradygmat obiektowości i używamy przy tym dziedziczenia. Co więcej w dowolnym miejscu aplikacji możemy zmienić prototype
obiektu. Jet to coś co nie jest możliwe w takich językach jak np. Java. Tam dziedziczenie odbywa się na poziomie kodu źródłowego i po skomplikowaniu nie możemy w łatwy sposób
zmieniać klasy bazowej po której dziedziczymy. Tak więc język który na pierwszy rzu oka wygląda jakby nie był obiektowy oferuje bardzo elastyczną obiektowość. Jak możemy to
wykorzystać do wstrzykiwania zależności? Utwórzmy na początek obiekt typu Car:

{% highlight javascript %}
function Car() {
    this.drive = () => {
        this.engine.start();
        console.log('drive');
    }
}
{% endhighlight %}

W funkcji drive wykorzystałem obiekt typu Engine, ale ten nigdzie nie został ustawiony. Jak go wstrzyknąć? Również za pomocą prototypu:

{% highlight javascript %}
Car.prototype.engine = new Engine('v8');
{% endhighlight %}

Jak widać tworzymy obiekt Engine i podpinamy go do prototypu _Car_ dzięki czemu będzie on dostępny we wszystkich obiektach ego typ. W łatwy sposób możemy wstrzykiwać zależności
do obiektu, bez konieczności wykorzystywania dodatkowych bibliotek. No dobra ale co w przypadku kiedy Car będzie miał inny model silnika niż _v8_? Przy obecnym podejściu
wszystkie samochody będą miały taki sam silnik. Możemy podejść do tego problemy w taki sposób:

{% highlight javascript %}
function Car(engineModel) {
    this.model = 'another'
    this.drive = () => {
        this.engine.start.call(this);
        console.log('drive');
    }
}
{% endhighlight %}

Korzystając z metody call zmieniamy kontekst this funkcji _start_ z _Engine_, dzięki czemu mamy możliwość zmiany danych na jakich pracuje ten obiekt.


Podsumowując, javascript jest bardzo ciekawym językiem. Oferuje odmienne podejście do obiektowości niż main streamowe języki takie jak np. Java. Czy te podejście jest
lepsze? To zależy, jak zawsze. Może ono powodować problemy ze zrozumieniem wśród programistów znających inne języki, dodatkowo może utrudniać zrozumienie takiego kodu.
W końcu w klasach mamy wszystko w jednym miejscu, tutaj prototyp można zmienić w dowolnym miejscy aplikacji. Mimo to js daje większą elastyczność jeżeli chodzi
o dziedziczenie i wstrzykiwanie zależności.