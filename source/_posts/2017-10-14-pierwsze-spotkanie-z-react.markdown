---
layout: post
title: "Pierwsze spotkanie z React"
date: 2017-10-14 18:58:19 +0200
comments: true
categories: TYPESCRIPT
---
Jako programista full stack używający na froncie Angularjs i poznający dobrodziejstwa Angular 4, byłem przekonany o wyższości tych frameworków nad innymi. Nie byłem w stanie przyjąć do wiadomości, 
że na froncie w dzisiejszych czasach można użyć czegoś innego niż Angulara, aby stworzyć nowoczesną 
aplikację SPA. Dziś jednak ten pogląd runął i moim oczom ukazał się stwór o nazwie React. 

<!--more-->
Zanim jednak zagłębię się w szczegóły Reacta, pokazując jego prostotę, chcę wspomnieć o moim pierwszym spotkaniu z Reactem. Uczestniczyłem w warsztatach zorganizowanych
przez Łódzki [NodeSchool](https://nodeschool.io/lodz/). Spędziłem pół dnia z osobami, które doskonale znają tę technologię, poznając idee towarzyszącą React, sposób jego 
działania oraz pułapki, na jakie można wpaść przy pisaniu w React. Wszystkim mogę zdecydowanie polecić spotkania w tym pozytywnym gronie. Jest to jedna z lepszych okazji, 
żeby w darmowy sposób rozwinąć swoją wiedzę. 

## Czym jest React?
Jest to biblioteka do budowania interfejsów użytkownika. Nie jest to framework, czyli magiczna machina, która sprawia że 
aplikacja działa, wywołując czasami kawałek napisanego kodu. Jest to biblioteka, której używamy żeby z małych kawałków, jakimi są komponenty  
zbudować kompletny system w możliwie najprostszy sposób. Podejście komponentowe do budowania aplikacji staje się obecnie standardem. Tak naprawdę zawsze było: nawet 
w takich technologiach jak jsp dało się tworzyć reużywalne tagi, z których potem można było skleić całą stronę. To, co z tego wychodziło w praktyce, to już inna rzecz ;)
We współczesnych rozwiązaniach opartych o komponenty używałem do tej pory głównie Angular 1/4, jednak ten okazuje się być gigantyczną machiną dodającą bardzo dużą
warstwę abstrakcji do wszystkiego, co dzieje się w przeglądarce. Oczywiście ma to swoje plusy, takie jak np. zarządzanie zależnościami. Czasami jednak jest to mechanizm tak trudny do zrozumienia, 
że nie obejdzie się bez wczytania w dokumentację bądź ogromne ilości kodu. Sam fakt, że Angular 4 ma swój kompilator, który może działać 
jako JIT oraz AOT (ahead of time - kompilacja komponentów do html jeszcze przed wysyłaniem do przeglądarki) podkreśla złożoność Angulara. Kilkakrotnie spotkałem się z opinią, że 
Angular to overengineering i powoli zaczynam zgadzać się z tym stwierdzeniem. React przy Angularze sprawia wrażenie wyjątkowo nieskomplikowanego. Trochę czystego js, idea Web components 
i małe API pozwalają szybko zbudować aplikację. 

## Web components
Idea stojąca za React to komponenty, zbliżone do Web components. Mechanizm dostępny natywnie w HTML od wersji 5. Tak, natywnie. Co oznacza również, że można tworzyć
aplikację zbudowaną z komponentów bez użycia jakiegokolwiek dodatkowego frameworku, biblioteki. Oto przykład takiego natywnego komponentu: 

```html
<html>
<head>
<script>
class MyNativeComponent extends HTMLElement {
	constructor() {
	    super();
	    var shadow = this.attachShadow({mode: 'open'});
	    var p = document.createElement('p');
	    p.innerText = 'Hello World from component';
	    shadow.appendChild(p);
	}
}
window.customElements.define('my-native', MyNativeComponent);
</script>
</head>
<body>
	<my-native name="Anita"></my-native>
</body>
</html>
```

Wszystko co trzeba zrobić żeby klasa ES6 stała się komponentem, to odziedziczenie po klasie HTMLElement, a następnie korzystając z shadow DOM można renderować dynamicznie 
dowolne elementy. 

## Komponenty React
Wadą takich natywnych komponentów jest brak elastycznego mechanizmu do definiowania widoku html dla komponentu oraz łączenia go z logiką biznesową zawartą w klasie js.
I tutaj z pomocą przychodzi React razem z JSX, czyli rozszerzeniem składniowym js pozwalającym definiować html komponentu bezpośrednio w js. W praktyce komponent _Hello world_ napisany w React wygląda tak:

```javascript 
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props);
    this.state = { username: '' };
  }

  // uwaga: jeżeli nie używasz Arrow function to musisz zbindować this w konstruktorze
  handleChange = event => { 
    this.setState({username: event.target.value});
  }

  render() {
    return (
      <div className="center-align">
        <header>
          <img src={logo} className="App-logo" />
          <h2>Welcome {this.state.username}</h2>
        </header>
        <div className="row" style={{margin: '0 20% 0 20%'}}>
          <input placeholder="Enter your name" onChange={this.handleChange} />
        </div>
      </div>
    );
  }
}
```

Komponent React to zwykła klasa dziedzicząca po Reactowym _Component_. W metodzie _render_ zwracamy kod html zapisany za pomocą JSX. Jak widać zawiera on odwołania
do zmiennych js, może również posiadać osadzony dowolny kod js. Należy jednak pamiętać, że nie jest to zwykły html i nie wszystko, co jest prawidłowym html, będzie poprawnym
JSX. Przykładowo: nie możemy używać słowa _class_ do dołączania styli ponieważ jest to słowo kluczowe zarezerwowane przez js. Zamiast tego należy dołączać style poprzez 
_className_. Również sposób używania stylów inline różni się od tego znanego z html, ponieważ do atrybutu style należy przekazać kod js zwracający obiekt definiujący styl.
Kod js w JSX załącza się w znaczniku _{}_, natomiast druga para klamer w przykładzie pochodzi od obiektu js. Tak zapisany kod JSX zostanie przetranspilowany do wywołań
funkcji Reacta tworzących wynikowy kod html. 

W powyższym przykładzie pojawił się jeszcze jeden element wymagający wyjaśnienia. Jest to _state_. Obiekt _state_ odpowiada za przekazywanie danych do Reacta, na podstawie
których będzie renderowany widok. React nie śledzi jednak zmian w obiekcie _state_, w przeciwieństwie do Angular, który tworzy watchersy do obiektów w scope. 
React dowie się, że coś w obiekcie _state_ uległo zmianie, dopiero kiedy zostanie wywołana asynchroniczna funkcja _setState_, która spowoduje wywołanie funkcji _render_ i w rezultacie
ponowne narysowanie komponentu wraz ze zaktualizowanymi danymi.

## Jak zacząć przygodę z React?
Żeby stworzyć pierwszą aplikację wystarczy kilka prostych kroków:

  * Zainstalować node i npm
  * Zainstalować generator pustego projektu: _npm install -g create-react-app_
  * Utworzyć pusty projekt: _create-react-app jakas-dowolna-nazwa_
  * W katalogu projektu uruchomić aplikację: _npm start_

Tak stworzony projekt ma minimum konfiguracji, automatycznie wczytuje zmiany w kodzie do przeglądarki i składa się tylko w kilku plików. Wszystkie dalsze informacje
potrzebne do nauki, znajdują się [na oficjalnej stronie](https://reactjs.org/docs/installation.html)

## Podsumowanie
Kod z poprzedniego przykładu wyświetli taki prosty komponent składający się z pola tekstowego oraz etykiety, która się będzie zmianiać wraz z wpisywaniem tekstu:

![GitHub Logo](/images/react.png)

Podczas wpisywania tekstu w polu tekstowym do metody zostanie wysłany event html-a onChange, który ustawi nową wartość w state. Mechanizm bardzo prosty, nie ma tutaj
dodatkowych watchy, który śledzą zmiany i automatycznie odświeżają widok. Do dyspozycji mamy również mechanizm przekazywania danych między komponentami. 
Do tego właśnie służy parametr konstruktora _props_. Może on zawierać dane z nadrzędnego komponentu. Oczywiście w nietrywialnych aplikacjach nie zawsze dane są przekazywane 
bezpośrednio od rodzica do dziecka, stąd też w aplikacjach korzystających z React korzysta się często z Redux. Jest to kontener przechowujący dane aplikacji, które łatwo 
można przekazywać do dowolnego miejsca aplikacji. React można również połączyć z Typescript, dzięki czemu do dyspozycji mamy typowanie, interfejsy, dekoratory i wiele innych 
przydatnych rzeczy. Wszystko w React jest bardzo proste, a zarazem wystarczające, żeby budować duże systemy. Mimo że na co dzień używam Angular co zapewne nieszybko się 
zmieni, z pewnością jednak moje pierwsze spotkanie z React nie będzie ostatnim. 
