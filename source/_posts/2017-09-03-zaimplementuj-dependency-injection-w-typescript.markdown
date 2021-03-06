---
layout: post
title: "Zaimplementuj dependency injection w Typescript"
date: 2017-09-03 15:39:30 +0200
comments: true
categories:  TYPESCRIPT
---
Typescript jest nadzbiorem javascriptu oferującym statyczne typowanie oraz możliwość programowania obiektowego w dużo bardziej wygodnej formie niż prototypy
dostępne w ES5. Dzięki wykorzystaniu dekoratorów język ten miejscami przypomina bardzo Javę, w której możemy używać adnotacji. Patrząc jednak na wygenerowany
przez kompilator kod, podobieństwa do Javy już nie znajdziemy. Znajdziemy za to zwykły Javascript wykorzystujący prototypy. Pokażę dziś jak zaimplementować mechanizm,
który będzie pozwalał na wstrzykiwanie zależności w podobny do Springa sposób. Będzie to znakomita okazja, żeby poznać praktyczne zastosowanie dekoratorów Typescriptu.

<!--more-->

## Czym są dekoratory?
Zacznijmy od wyjaśnienia, czym tak w ogóle w Ts jest dekorator. Jest to mechanizm bardzo podobny do wzorca "Dekorator" znanego z programowania obiektowego. Dekorator jest funkcją, która na wejściu dostaje
inną funkcję/właściwość i dekoruje ją dodatkową logiką. Jak to wygląda w praktyce? Zobaczmy to na przykładzie dekoratora o nazwie Deprecated, którego zadaniem będzie wypisywanie
na ekranie informacji o tym, że wywołano właśnie przestarzałą metodę. Stwórzmy klasę, w której znajdzie się taki dekorator:

{% highlight javascript %}
class User {
    public userName: string;

    constructor(userName?: string) {
        this.userName = userName;
    }

    @Deprecated
    setUserName(userName: string): void {
        this.userName = userName;
    }
}
{% endhighlight %}

Klasa bardzo prosta. Zawiera konstruktor i setter. Z jakiś powodów setter został oznaczony dekoratorem Deprecated. Użycie dekoratora przypomina tutaj do złudzenia użycie adnotacji
w Java, mimo że mechanizm bardzo się od niej różni. Adnotacje w Javie służą do oznaczania pól czy metod, nie modyfikują jednak działania tych elementów, są tylko informacją
dla logiki umieszczonej w innym miejscu o tym, że należy wykonać jakąś dodatkową operację na adnotowanym polu/metodzie. W Ts dekoratory bezpośrednio modyfikują elementy klasy, nad którymi zostały
umieszczone. Dekoratorów użytych do oznaczenia metody w klasie może byc więcej. Nie musimy ograniczać się nawet do samych metod, możemy oznaczać w ten sposób również pola, parametry metod, a nawet całe klasy.
W końcu klasa w wygenerowanym przez kompilator Ts kodzie to również funkcja. Dla klasy _User_ wygenerowany kod będzie miał taką postać:

{% highlight javascript %}
var User = (function () {
    function User(userName) {
        this.userName = userName;
    }
    User.prototype.setUserName = function (userName) {
        this.userName = userName;
    };
    __decorate([
        Deprecated,
        __metadata("design:type", Function),
        __metadata("design:paramtypes", [String]),
        __metadata("design:returntype", void 0)
    ], User.prototype, "setUserName", null);
    return User;
}());
new User();
function Deprecated(target, key, descriptor) {
    return {
        value: function () {
            var args = [];
            for (var _i = 0; _i < arguments.length; _i++) {
                args[_i] = arguments[_i];
            }
            console.warn("Using deprecated API: " + target.constructor.name + "." + key);
            return descriptor.value.apply(this, args);
        }
    };
}
{% endhighlight %}

Brrr...To, co wygenerowało się z tak prostej klasy wygląda na pierwszy rzut oka na bardzo skomplikowany kod. Jest to dobry przykład na to, jak wiele daje kompilator Ts.
Nie musimy pisać tak zawiłych konstrukcji, tylko tworzymy klasy w sposób do jakiego przywykliśmy w językach takich jak np. Java, C#. Wracając do przykładu z Deprecated, mamy klasę z metodą oznaczoną dekoratorem.
Dekorator to tak naprawdę funkcja, która ma na celu rozszerzenie działania innej funkcji. W tym przypadku funkcja taka przyjmuje 3 parametry:

  * target - funkcja w której został użyty dekorator. W powyższym przykładzie będzie to funkcja reprezentująca klasę _User_
  * key - nazwa metody, którą oznaczyliśmy adnotacją (_setUserName_)
  * descriptor - wynik wywołania funkcji _Object.getOwnPropertyDescriptor()_ zwracający podstawowe informacje o polu które zostało oznaczone dekoratorem.
  Przykładowo: _{"writable":true,"enumerable":true,"configurable":true}_

Mając te wszystkie informacje możemy napisać dekorator, który będzie informował o użyciu przestarzałej metody:
{% highlight javascript %}
function Deprecated(target: any, key: string, descriptor: any) {
    return {
        value: function (...args: any[]) {
            console.warn(`Using deprecated API: ${target.constructor.name}.${key}`);
            return descriptor.value.apply(this, args);
        }
    };
}
{% endhighlight %}
Wartość zwrócona z dekoratora zastąpi deskryptor metody _setUserName_, a dokładnie pole _value_ w tym deskryptorze. Krótko mówiąc nadpiszemy funkcję własną implementacją.
Dekorator po wylogowaniu na konsoli odpowiedniego ostrzeżenia wywołuje pierwotną metodę korzystając z _apply_, parametrów wywołania i zmiennej this. Jak widać logikę metody, nad którą
został wstawiony dekorator możemy dowolnie modyfikować, dekorować o kolejną logikę. Daje nam to potężny mechanizm do zmniejszenia ilości powielanego kodu i otwiera drogę do budowania
bardziej abstrakcyjnych mechanizmów.


## Wymagania
Przedstawiona za chwilę implementacja Typescriptowej wersji Springa będzie wyjątkowo prosta. Będzie się ona składać z dekoratora @Bean, który to pozwoli automatycznie tworzyć obiekty klas i wstrzykiwać
je w pola, które będą miały dekorator @Inject. Przeanalizujmy zatem prosty przypadek użycia takiego narzędzia. Tworzymy klasę _User_, która będzie odpowiadać za przechowywanie
danych zalogowanego użytkownika. Dla uproszczenia będzie to tylko login:

{% highlight javascript %}
@Bean()
class User {
    public login: string;
}
{% endhighlight %}

Kolejną klasą będzie _Transfer_ odpowiedzialny za wykonanie przelewu dla zalogowanego użytkownika. Obiekt użytkownika będzie wstrzykiwany przez dekorator _Inject_, bez konieczności
ręcznego ustawiania pola. Dodatkowo pole w klasie będzie miało ustawiony modyfikator dostępu private, tak żeby nikomu nie przyszło do głowy próbować go ustawiać ręcznie.
Obiekt ten będzie singletonem przechowywanym przez omawianą implementację Springa.

{% highlight javascript %}
@Bean()
class Transfer {
    @Inject
    private user: User;

    makeTransfer() {
        console.log('Create transfer for: ' + this.user.login);
    }
}
{% endhighlight %}

Ostatnią klasą będzie _Application_, mająca wstrzykniętą instancję obiektu _User_ oraz _Transfer_.

{% highlight javascript %}
class Application {

    @Inject
    private transfer: Transfer;

    @Inject
    private user: User;

    login(userName: string): void {
        this.user.login = userName;
    }

    sendTransfer(): void {
        this.transfer.makeTransfer();
    }
}

const app: Application = new Application();
app.login('Kamil');
app.sendTransfer();
{% endhighlight %}

Podczas wywołania metody _login_, imię aktualnie zalogowanego użytkownika przekazywane jest do obiektu klasy _User_. Tak się składa, że ten sam obiekt został wstrzyknięty również do obiektu Transfer.
Omawiany mechanizm tworzy tylko jedną instancję obiektu i wszędzie ją wstrzykuje. Po wywołaniu metody _makeTransfer_, obiekt _Transfer_
prawidłowo wypisuje komunikat: _Create transfer for: Kamil_. Tak więc, bez konieczności martwienia się o utworzenie i ustawienie obiektu _User_ i _Transfer_ mamy możliwość korzystania z tych obiektów
w dowolnym miejscu aplikacji.

## Implementacja
Przejdźmy teraz do właściwiej części, czyli do omówienia jak to działa. Pierwszym elementem jest moduł, za pomocą którego są przechowywane utworzone/wstrzykiwane obiekty. Dlaczego moduł?
Żeby np. nie brudzić przestrzeni nazw, mieć możliwość importu go w dowolnym miejscu aplikacji. Oto jak może wyglądać taki moduł:

{% highlight javascript %}
module ApplicationContext {
    interface IRegister {
        [key: string]: any;
    }

    const beanRegistry: IRegister = [];

    export function registerBean(name: string, value: Object) {
        if (!beanRegistry[name]) {
            const lowerCaseName: string = name.toLowerCase();
            console.log(`Register: ${lowerCaseName}`);
            beanRegistry[lowerCaseName] = value;
        }
    }

    export function getBean(name: string) {
        const result: any = beanRegistry[name.toLowerCase()];
        if (!result) {
            throw new Error(`Bean ${name.toLowerCase()} not found`);
        }

        return result;
    }
}
{% endhighlight %}

Moduł składa się z tablicy przechowującej utworzone obiekty. Tablica ta została zadeklarowana za pomocą interfejsu w celu umożliwienia odwoływania sie do elementów tablicy
poprzez klucz-string. Bez tego interfejsu moglibyśmy wyciągać obiekty tylko poprzez indeks. Moduł oprócz tablicy zawiera funkcję, która zapisuje nowe obiekty w tablicy pod przekazaną nazwą.
Nazwa jest zamieniana na małe litery, podobnie jak w dalszej części kodu. Jest to rozwiązanie bardzo proste, jednak posiada tą wadę, że w przypadku gdy pole oznaczone _Inject_ będzie się nazywać
inaczej niż nazwa klasy pisana małymi literami, to mechanizm nie znajdzie obiektu. W przypadku gdy obiekt istnieje już w tablicy nic nie zostanie zrobione. Warto tutaj się pokusić o wyrzucenie
błędu z informacją: _multiple bean definition found_. Druga metoda modułu wyciąga z tablicy utworzony obiekt i go zwraca. W przypadku braku obiektu wyrzucany jest błąd.

Zajmijmy się teraz dekoratorem odpowiedzialnym za rejestrowanie komponentów w _beanRegistry_:

{% highlight javascript %}
function Bean(): any {
    return function (target: any) {
        const original = target;

        function construct(constructor: any, args: any, ) {
            var c: any = function () {
                return constructor.apply(this, args);
            }
            c.prototype = constructor.prototype;
            return new c();
        }

        let result: any = construct(original, []);
        ApplicationContext.registerBean(target.name, result);

        let f: any = function (...args: any[]) {
            console.log('create:' + target.name.toLowerCase());
            let result: any = construct(original, args);
            return result;
        }

        f.prototype = original.prototype;
        return f;
    }
}
{% endhighlight %}

Dekorator _Bean_ tworzy funkcję _f_, która zastępuje konstruktor klasy oznaczonej tym dekoratorem. Funkcja ta odpowiada za wywołanie oryginalnego konstruktora, zapisanie utworzonego
obiektu w tablicy _beanRegistry_ za pomocą funkcji _registerBean_ oraz zwrócenie instancji obiektu obiektu. Mamy mechanizm rejestrowania komponentów, spójrzmy teraz jak działa
wstrzykiwanie tych komponentów:

{% highlight javascript %}
function Inject(target: any, key: any) {
    let _val = this[key];

    const getter = function () {
        if (!_val) {
            console.log('create:' + key.toLowerCase());
            _val = ApplicationContext.getBean(key);
            this[key] = _val;
        }
        console.log(`Get: ${key} => ${JSON.stringify(_val)}`);
        return _val;
    };

    if (delete this[key]) {
        Object.defineProperty(target, key, {
            get: getter,
            enumerable: true,
            configurable: true
        });
    }
}
{% endhighlight %}

Główną rzeczą jaką robi ten dekorator jest nadpisywanie gettera dla właściwości, z którą został użyty. Podczas odwoływania się do takiego pola, zostaje uruchomiony zdefiniowany tutaj
getter. Zwraca on instancję obiektu o nazwie takiej, jak nazwa pola gdzie został użyty _Inject_. Nazwa ta musi oczywiście odpowiadać nazwie klasy, którą chcemy wstrzyknąć. Ot, to cała magia kryjąca się
za wstrzykiwaniem zależności.

## Quo vadis?
Za pomocą dekoratorów stworzyliśmy bardzo prosty mechanizm DI. Nic nie stoi na przeszkodzie, żeby go rozbudować i korzystać z niego w np. aplikacjach serwerowych pisanych w Node.js. Jak to mówią,
_sky is the limit_.
Jednak żeby stworzone tutaj rozwiązanie było w pełni wartościowe, należy zatroszczyć się o kilka rzeczy:

  * możliwość definiowania sposobu tworzenia komponentów. Obecnie zawsze jest to singleton i nie ma możliwości tworzenia kolejnych obiektów za pomocą tego mechanizmu
  * usuwanie obiektów z rejestru, kiedy zostaje usunięty ostatni obiekt korzystający z komponentu. Pozwoli to uniknąć wycieków pamięci. Może warto dodać jakiś licznik w rejestrze?
  * definiowanie dowolnej nazwy komponentu, w tej formie zawsze nazwa pola musi pokrywać się z nazwą klasy
  * w przypadku obsługi wielu użytkowników w aplikacji, rejestr powinien być związany z sesją użytkownika.

Zachęcam do samodzielnej próby rozbudowy tego mechanizmu, kto wie, może to Twoja implementacja będzie w przyszłości popularnym DI dla Node.js ;)