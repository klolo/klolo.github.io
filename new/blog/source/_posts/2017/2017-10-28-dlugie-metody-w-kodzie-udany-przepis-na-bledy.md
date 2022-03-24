---
layout: post
title: "Długie metody w kodzie – udany przepis na błędy"
date: 2017-10-28 20:25:09 +0200
comments: true
categories: JAVA
---
Każdy programista, i to bardzo wcześnie, dowiaduje się, że metody powinny być krótkie. Jednak czytając kody systemów z jakimi pracujemy, znajdujemy często metody ciągnące się przez setki, a nawet tysiące linii. Skąd w takim razie pochodzi taki paskudny kod, skoro teoretycznie wszyscy wiedzą, że długie metody są złe? A może jednak, przez zwykłe lenistwo, nie zastanawiamy się nad tym czy pisana metoda powinna zostać zrefaktorowana? A może nie wiemy jaka długość jest dopuszczalna i kiedy metodę należy rozbić na mniejsze?

<!--more-->

## Największe grzechy długich metod

Zanim zastanowimy się jaką długość mają dobre metody, omówmy najważniejsze problemy jakie wynikają z przerośniętych funkcji:

### Są trudne do czytania

Długie metody mają często bardzo wiele poziomów wcięć, przez co łatwo się zgubić i zastanawiać czy _if_, kilkanaście linijek wcześniej został już domknięty. Do tego dochodzi masa przeróżnych zmiennych deklarowanych gdzie popadnie. Czytając taki kod należy skakać po metodzie niczym kangur, sprawdzając jaka wartość jest przypisana do zmiennej i czy przypadkiem nie jest to parametr wejściowy.

### Trudne do zapamiętania

W przypadku przerośniętej metody po dokładnym przeanalizowaniu kodu możemy mieć tylko ogólny pogląd tego czego można się po niej spodziewać. Z pewnością nie będziemy pamiętać każdego detalu logiki, jaka się tam znajduje. Może to potem skutkować pojawieniem się błędu w jakimś przypadku, którego nie przetestowaliśmy.

### Trudne do testowania

Wielki kawał kodu wymaga często długiego _mockowania_ oraz bardzo trudno stwierdzić bez zaglądania w raport pokrycia czy napisany test obejmuje każdy _case_. Dodatkowo duża liczba parametrów wejściowych jakie należy spreparować w teście może zniechęcić do pisania jakiegokolwiek testu.

### Sprzyjają duplikacji kodu

Z podowu dużej ilości zmiennych czy parametrów, ze źle napisanych metod trudniej jest wydzielać mniejsze metody. W tym przypadku kusi możliwość pójścia na łatwiznę i skopiowania istniejącej logiki. Łatwo też nie zauważyć, że jakaś logika już jest dostępna w kodzie, ponieważ zaszyła się głęboko w jakiejś funkcji.

### Wiele powodów do zmiany

Długie metody nigdy nie robią tylko jednej rzeczy. Robią bardzo dużo, przez co nie da się wręcz uniknąć złamania reguły _open/close_. Reguła ta mówi o tym, że dobrze zaimplementowana jednostka kodu jest zamknięta na modyfikację, ale jednocześnie otwarta na rozszerzanie. Oznacza to tyle, że w celu dodania nowej funkcjonalności nie modyfikujemy istniejącej funkcji, lecz piszemy nową, która wywoła tą starą i dodatkowo wykona nową logikę. Natomiast w przypadku klas powinniśmy skorzystać z polimorfizmu w celu rozbudowy aplikacji o nową funkcjonalność. W długiej metodzie zawsze będzie istnieć wiele powodów do zmiany zawartości i bardzo trudno będzie nie zmodyfikować jej wnętrza. Metody takie są często gorącym miejscem aplikacji. Co kilka tygodni, miesięcy ktoś coś tam zmienia. Jeżeli taki kod nie ma testów jednostkowych, to aż sami się prosimy o błędy.

### Trudność debugowania

Stawiasz breakpoint w środku długiej metody, uruchamiasz funkcję i okazuje się, że debugger nie zatrzymał się tam gdzie chciałeś. Zaczynasz więc wczytywać się w kod i analizować, bądź przechodzisz na debuggerze linijka po linijce. Znajome? Debugując metodę mającą 100+ linii można się czuć jak ofiara i zadawać sobie pytania: Dlaczego ja!? Czasami nawet okazuje się, że taka metoda jest zbyt wielka, a żeby ją zrozumieć trzeba prześledzić jej działanie na debuggerze.

### Zbyt dużo oczywistych komentarzy

Jako fan czystego kodu zgadzam się ze stwierdzeniem, że komentarze nie są potrzebne w kodzie. Co więcej widziałem i rozwijałem aplikacje, gdzie komentarzy nie było. No, może z paroma wyjątkami: tam gdzie intencja autora nie była jasna pojawiało się wyjaśnienie, dlaczego coś działa w taki, a nie inny sposób. W systemach tych nikt nie pisał w komentarzach jak działa dany fragment kodu. Metody, klasy były na tyle małe, że stwierdzenie co robią było równie szybkie jak przeczytanie komentarza, który zresztą zawsze może być nieaktualny, bądź niezrozumiale napisany. Nikt w tych systemach nie używał komentarzy, a w kodzie nie było zbędnego szumu, który przeszkadzałby w czytaniu. Nigdy w tych aplikacjach nie spotkałem się z stwierdzeniem, że przydałyby się obowiązkowe komentarze. Komentarze natomiast widywałem w systemach legacy, o bardzo długich metodach. Programiści pisali je mimo że nie mieli takiego obowiązku wynikającego z przyjętych zasad. Często wyglądało to tak:

```java
// some comment
…code…

// some comment
…code…
```

Czytając funkcje, w której fragmenty kodu są poprzedzone komentarzem aż się prosi, żeby powydzielać mniejsze metody, których nazwa będzie mówić dokładnie to samo co komentarz. Może autorzy kodu próbowali oczyścić sumienie i myśleli, że chociaż w taki sposób ułatwią kolejnym programistom życie?

## Inne problemy

Problemów z długimi metodami jest oczywiście dużo więcej. Można chociażby jeszcze wspomnieć o dużej liczbie zależności, ścisłym powiązaniach w kodzie (_coupling_), trudności z optymalizacją. Myślę jednak, że wymienione wyżej problemy są argumentem za tym, że metody powinny być krótkie.

## Jak długa powinna być metoda?

Pojawia się jednak pytanie jak krótka powinna być funkcja? Czy 100 lini to za dużo? Czy 25 to maksimum? Niektórzy twierdzą, że dobra metoda powinna mieścić się w całości na ekranie. Można wtedy zadać pytanie o rozmiar czcionki i rozdzielczość. Clean code wspomina o tym, że dobra metoda powinna wykonywać tylko jedną rzecz, ale za to dobrze. Kiedy spojrzymy jednak na np. strumienie w Java 8, to dzięki podejściu deklaratywnym w kilku linijkach możemy zrobić bardzo wiele rzeczy. Dzieje się to dzięki temu, że deklarujemy, co kod ma robić, a nie w jaki sposób. Trudno sobie wyobrazić funkcje pracujące na strumieniach i robiące tylko jedną rzecz. Musiałyby składać się z 2,3 linijek. Tak duża granularność metod może również prowadzić do problemów z czytaniem kodu i wprowadza zbędną złożoność. Nie wspomnę już o wydajności. JVM zamiast wykonywać faktyczny algorytm, spędza więcej czasu na kolejnych wyołaniach. Czynniki te są oczywiście ważne i należy o nich pamiętać, jest jednak jeszcze jedna zasada określająca czy metoda wymaga rozbicia. Są to poziomy abstrakcji. Nigdy nie powinno się ich mieszać. Co to oznacza w praktyce? Spójrzmy na poniższą wymyśloną metodę. Mimo że jest ona krótka, robi zdecydowanie za dużo, mieszając przy tym poziomy abstrakcji:

Uwaga: W kodzie pojawia się doklejanie danych z parametru do zapytania bazodanowego. Nie rób tak, narażasz się na sql injection.

```java
	/**
	* Metoda pobiera itemy z bazy i wysyla je do innego systemu.
	*/
    void someBusinessLogic(Input in) {
        if (in.getSth() == null) {
            throw new IllegalArgumentException();
        }

        try {
            // pobranie danych z bazy
            Connection con = new Connection("127.0.0.1:1521");
            List<Item> items = (List<Item>) con.makeQuery("select * from items where sth=" + in.getSth());
            
            List<ItemForSend> itemsForSend = new LinkedList<>();

            // pobranie szczegółów dla każdego elementu
            for (Item i : items) {

                // przefiltrowanie danych, uzupelnienie o szczegóły
                if (i.isSomething(in)) {
                    ItemDetails details = (ItemDetails) con.makeQuery("select * from details where id = " + i.id);
                    itemsForSend.add(new ItemForSend(details));
                }
            }

            // wysłanie daych do innego systemu
            String connectorInput = "";
            for (ItemForSend itemForSend : itemsForSend) {
                connectorInput += itemForSend.toJson();
            }

            connectorInput = connectorInput.trim() + in.getAdditionalData();
            AnotherSystemConnector connector = new AnotherSystemConnector();
			connector.send(connectorInput);
        }
        catch (SomeException e) {
            handle();
        }
    }
```

Powyższy pseudo kod, oprócz potencjalnych problemów z _sql injection_ pokazuje pomieszanie poziomów abstrakcji. W jednej metodzie mamy zarówno logikę biznesową jak i operacje niższego poziomu (np. pobranie danych z bazy danych) oraz operacje o jeszcze niższym poziomie abstrakcji takie jak sklejanie danych wejściowych do wywołania usługi jakiegoś zewnętrznego systemu oraz obsługę błedów. Za każdym razem kiedy zmienią się wymagania biznesowe funkcja ta będzie zmieniana. Bo jak inaczej np. zmienić dane wejściowe do zewętrznego systemu w ostatnich linijkach _someBusinessLogic_?

Metoda z przykładu powinna zostać podzielona względem poziomów abstrakcji i w jej wnętrzu powinny zostać praktycznie tylko wywołania innych metod. Dodatkowo każdy z bloków funkcji realizujących jedną rzecz został opatrzony komentarzem, który po podziale na mniejsze funkcje nie będzie potrzebny. Nawet komentarz nad metodą jest zdeaktualizowany, ponieważ nie zawiera informacji o tym, że dla każdego elementu pobierane są dodatkowo szczegóły. Zrefaktorowany kod mógłby wyglądać tak:

```java
    void someBusinessLogic(Input in) throws SomeException {
        List<Item> items = filterItems(getItems(in), in);
        List<ItemForSend> itemsForSend = convertToItemToSend(items);

        String connectorInput = getConnectorInput(itemsForSend, in);
        sendToAnotherSystem(connectorInput);
    }

    List<Item> filterItems(List<Item> items, Input in) {
        List<Item> result = new LinkedList<>();
        for (Item i : items) {
            if (i.isSomething(in)) {
                result.add(i);
            }
        }
        return result;
    }

    List<Item> getItems(Input in) {
        return (List<Item>) con.makeQuery("select * from items where sth=" + in.getSth());
    }

    List<ItemForSend> convertToItemToSend(List<Item> items) {
        List<ItemForSend> result = new LinkedList<>();
        for (Item i : items) {
            ItemDetails details = getItemDetails(i);
            result.add(new ItemForSend(details));
        }

        return result;
    }

    ItemDetails getItemDetails(Item i) {
        return (ItemDetails) con.makeQuery("select * from details where id = " + i.id);
    }

    String getConnectorInput(List<ItemForSend> itemsForSend, Input in) throws SomeException {
        String connectorInput = "";
        for (ItemForSend itemForSend : itemsForSend) {
            connectorInput += itemForSend.toJson();
        }

        return connectorInput.trim() + in.getAdditionalData();
    }

    void sendToAnotherSystem(String input) {
        AnotherSystemConnector connector = new AnotherSystemConnector();
        connector.send(input);
    }
```

Walidacja i obsługa wyjatków została przeniesiona do metody nadrzędnej. Nieaktualny komentarz opisujący działanie został usunięty, zamiast tego jest krótki kod, który łatwo zrozumieć. Mówi on dokładnie co biznesowo robi metoda bez wchodzenia w szczegóły techniczne. Mamy odseparowany wysoki poziom abstrakcji, reprezentujący logikę biznesową. Połaczenie do bazy zostało wyniesione na zewnątrz metody, dzieki czemu możemy być reużywane. Jest to przy okazji optymalizacja wydajności. Każda z utworzonych metod jest krótka, łatwo ją reużyć, przetestować. Jeżeli zmienią się wymagania biznesowe możemy dopisać nową funkcję i wpiąć ją w dowolne miejsce procesu bez konieczności modyfikacji istniejących metod.

Na pierwszy rzut oka widać, że kod znacznie sie wydłużył. Mimo to łatwo zrozumieć co robi, a jeżeli skorzystamy z strumieni Javy 8 i wykonany refactoring, skrócimy go znacznie.

## Podsumowanie

Wiemy już dlaczego długie metody są złe oraz jak powinno się je dzielić. Nie pozostaje teraz nic innego jak pisać kod, który będzie można łatwiej zrozumieć, utrzymywać, a w dodatku nie bedzie powodem do wstydu.