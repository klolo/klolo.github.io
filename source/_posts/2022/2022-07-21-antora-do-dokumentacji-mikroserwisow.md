---
layout: post
title: "Antora - narzędzie do dokumentacji mikroserwisów"
date: 2022-07-21 21:31:32 +0100
comments: true
categories: OTHERS
---

# Dokumentacja techniczna dla mikroserwisów

## Dokumentacja? a po co to komu?

Jedną z największych zalet architektury opartej na mikroserwisach jest duża autonomia zespołów, osiągana poprzez niezależny cykl życia serwisu. Jednym z kroków do osiągnięcia tej autonomii jest często rozbicie systemu na wiele osobnych projektów, czasami są ich nawet setki. Podejście to może wpłynąć na dość nieoczekiwany aspekt, jakim jest tworzenie dokumentacji. Serwisy i biblioteki, które wytwarzamy składają się na system, w którym na porządku dziennym jest budowanie nowych funkcjonalności na podstawie istniejącego już kodu. Skąd jednak wiadomo jak działa kod, który chcemy wykorzystać? Skąd informację o projekcie ma czerpać programista, który dopiero w projekt się wdraża? Samodokumentujący się kod jest zazwyczaj tylko jednorożcem, o którym każdy słyszał, a nikt nie widział. Dodatkowo istnieją też informacje, które nie pasują do javadoc a są przydatne. Uważam, że każdy projekt powinien mieć, chociaż podstawową formę dokumentacji, tak żeby nie trzeba było za każdym razem analizować jego zawartości. Można tutaj skorzystać z podejścia gdzie każdy mikroserwis, czy też biblioteka ma dokumentację przechowywaną razem z kodem źródłowym. Dzięki temu developer, chcąc ją zaktualizować nie musi opuszczać IDE. Podejście takie promuje tworzenie oraz aktualizowanie dokumentacji.

Alternatywą jest trzymanie dokumentacji na jakiejś dedykowanej stronie wiki bądź narzędziu typu Confluence. Tutaj jednak pojawia się dużo większe ryzyko tego, że developerzy nie będą tak chętnie aktualizować dokumentacji oraz potencjalne ryzyko, że to zewnętrzne narzędzie będzie miało swoje problemy np. powolne wyszukiwanie. Z własnego doświadczenia wiem, że zwykłe pliki tekstowe trzymane razem z kodem są częściej czytane, pisane, aktualizowane oraz po krótkiej nauce składni sprawiają mniej problemów z edycją. Pozostaje jednak pytanie, co w przypadku gdy chcemy dostarczyć dokumentację tworzonych przez nas mikroserwisów, czy też bibliotek innym zespołom bądź też klientom? Co w przypadku gdy chcemy mieć możliwość łatwego wyszukiwania informacji w całej dokumentacji, nie zależnie gdzie ona się znajduje? Co w przypadku gdy z naszej dokumentacji chcą skorzystać testerzy bądź analitycy, czy muszą mieć wtedy dostęp do kodów źródłowych? Powyższe problemy możemy rozwiązać za pomocą narzędzia o nazwie _Antora_, które chciałbym pokazać, bazując na tym, w jaki sposób narzędzie to wykorzystaliśmy w projekcie, w którym aktualnie pracuje.

## Akt 1, czyli jak zacząć

Najbardziej pospolitym podejściem do dokumentowania projektów jest umieszczenie pliku _README.md_ w katalogu głównym projektu. Jest to rozwiązanie proste, ale niekoniecznie sprawdza się w systemie złożonym z dużej ilości projektów.  
Bo co w przypadku gdy nie wiemy, w jakim dokładnie projekcie szukać jakiegoś mechanizmu? Tutaj z pomocą przychodzi _Antora_, która pozwala agregować dokumentacje z różnych projektów na git. Po zebraniu plików tekstowych z projektów git-a tworzony jest wynikowy html, którego możemy wystawić na jakiś serwerze i nawet osoby niemające dostępu do kodów mogą zapoznać się z dokumentacją oraz w łatwy sposób ją przeszukiwać. Żeby zacząć pracę z _Antora_ należy zainstalować Node.js i utworzyć plik package.json zgodnie z oficjalną [dokumentacją](https://docs.antora.org/antora/latest/install-and-run-quickstart). Kolejnym krokiem jest utworzenie pliku antora-playbook.yaml, w którym to możemy zacząć konfigurować narzędzie. Trochę bardziej rozbudowany niż w oficjalnej dokumentacji przykład takiego pliku:

```yaml
sources: # 1
  - url: ${GITLAB_URL}/some-repo.git
    start_path: docs
  ...
antora:
    extensions: # 2
        - './../antora/extensions/maven-version-parser.js'
output: # 3
    dir: ./aggregated
    destinations:
        - provider: fs
          clean: false
runtime: 
    cache_dir: /tmp/antora-cache/ # 4
site:
    title: Your application name
    start_page: some-project::index.adoc
```

1. Sekcja sources zawiera listę repozytoriów git, do których możemy się uwierzytelnić na wiele [różnych sposobów](https://docs.antora.org/antora/latest/playbook/git-credentials-path-and-contents/). Każdy z tych projektów musi posiadać w katalogu wskazanym przez pole `start_path` plik _antora.yaml_ z konfiguracją danego projektu.
2. _Antora_ pozwala na tworzenie rozszerzeń w języku javascript, którymi to możemy wpłynąć na proces generowania dokumentacji. U nas w projekcie pojawiła się potrzeba korelacji wersji wydawanej dokumentacji z wersją artefaktu z pliku pom.xml. W tym przypadku taki plugin doskonale się sprawdził. 
3. W tej sekcji możemy skonfigurować miejsce, gdzie ma pojawić się wynikowy plik html z dokumentacją. _Antora_ pozwala tutaj skorzystać z zarówno ze zwykłego katalogu, paczki zip, jak również pozwala podpiąć własny mechanizm publikowania kodów zaimplementowany w javascript.
4. W przypadku gdy implementujemy własny plugin, zmieniający sposób generowania dokumentacji, przydatne może się okazać wskazanie, do jakiego katalogu _Antora_ ma pobierać wskazane w sekcji sources projekty.
5. Konfiguracja domyślnej strony, która może pochodzić z dowolnego projektu wskazanego w sekcji `sources`.  

Warto zwrócić tutaj uwagę na to, że pliki, w jakich przechowujemy informacje to nie markdown, lecz [asciidoc](https://asciidoc.org). Oba rozwiązania są bardzo podobne, jednak Asciidoc oferuje znacznie więcej niż to, co możemy zrobić w pliku markdown. Jest to niewielki minus w przypadku kiedy korzystaliśmy wcześniej z plików md i chcemy przenieść się na _Antore_. Koniecznie trzeba będzie przerobić istniejącą dokumentacje na inną składnię oraz przede wszystkich poświęcić chwilę na jej opanowanie. 

## Dostosowanie wyglądu

W przypadku gdy potrzebujemy dostosować wygląd naszej dokumentacji do standardów firmowych bądź wymagań klienta możemy to zrobić poprzez ściągnięcie [szablonów](https://gitlab.com/antora/antora-ui-default/-/tree/master/src), z których _Antora_ generuje wynikowy html i zmianę wybranych fragmentów. Po rozpakowaniu plików należy wskazać, gdzie się one znajdują:

```yaml
ui:
  bundle:
    url: ./../antora/ui # 1
    supplemental_files: ./../antora/supplemental-ui # 2
antora:
    extensions:
      - '@antora/lunr-extension' # 3
asciidoc:
  attributes:
    source-highlighter: highlight.js #4
```

1. Katalog, w którym przechowujemy pobrane szablony. Można tutaj również przechowywać je w formie paczki na zdalnym serwerze.
2. Dodatkowe pliki statyczne użyte w wygenerowanym html np. [wyszukiwarka lunr](https://github.com/Mogztter/antora-lunr).
3. Rejestracja rozszerzeń zainstalowanych poprzez polecenie `npm install xxx`. W tym przypadku jest to plugin do przeszukiwania dokumentacji.
4. Włączenie kolorowania składni, o tym dokładniej za chwilę.

Mając pobrane szablony, możemy dowolnie je modyfikować. Warto tutaj skonfigurować kolorowanie składni fragmentów kodów z wykorzystaniem biblioteki [highlightjs](https://highlightjs.org/usage/). Można to zrobić poprzez dodanie fragmentu szablonu, z którego w wynikowym html pojawi się import skryptu oraz wywołanie funkcji inicjującej mechanizm kolorowania np. w pliku _footer-scripts.hbs_:

```
<script src="{{{uiRootPath}}}/js/site.js"></script>
<script src="{{{uiRootPath}}}/js/vendor/highlight.js"></script>
{{#if env.SITE_SEARCH_PROVIDER}}
{{> search-scripts}}
{{/if}}

<script>hljs.highlightAll();</script>
```

## Diagrams as a code

Dobrą praktyką, jest tworzenie i przechowywanie diagramów oraz rysunków technicznych w formie kodu. Ułatwia to ich edycję, w końcu edytor tekstowy ma każdy a jakieś dziwne narzędzie do edycji pliku graficznego już nie koniecznie. Jednym z narzędzi, które pozwalają na tworzenie takich diagramów za pomocą własnego języka jest [PlantUML](https://plantuml.com). Niestety domyślnie Antora nie poradzi sobie z wyrenderowaniem takich diagramów. Do tego celu możemy wykorzystać narzędzie o nazwie [Kroki](https://kroki.io), które to generuje diagramy, wspierając przy tym różne formaty plików źródłowych. Narzędzie to jest dostępne w internecie, ale ze względów bezpieczeństwa powinniśmy korzystać z takiej opcji, dlatego najprościej jest uruchomić lokalnie własny serwer i do niego się łączyć. Najszybszą opcją jest skorzystanie z [obrazu dockera](https://hub.docker.com/r/yuzutech/kroki) bądź też [helm charta](https://artifacthub.io/packages/helm/cowboysysop/kroki) w celu uruchominienia serwera _Kroki_ wewnątrz naszej infrastruktury. Po uruchomieniu serwera w pliku _antora-playbook.yaml_ powinniśmy wskazać adres na nasz serwer:

```yaml
asciidoc:
    extensions:
        - asciidoctor-kroki # 1
    attributes:
        kroki-fetch-diagram: true
        kroki-server-url: <URL> #2
        kroki-http-method: adaptive
        kroki-plantuml-include: https://<URL>/-/raw/master/puml-theme.puml #3
        source-language: asciidoc@
        table-caption: false
```

1. Włączenie rozszerzenia obsługującego generowania diagramów z wykorzystaniem serwera Kroki. [Plugin ten](https://www.npmjs.com/package/asciidoctor-kroki), należy wcześniej doinstalować.
2. Wskazanie na adres, pod jakim serwer będzie dostępny.
3. Jeżeli chcemy mieć globalnie ostylizowane diagramy zgodnie z wyglądem naszej dokumentacji bądź standardem wewnątrz firmowym to tutaj możemy wskazać adres z konfiguracją [styli dla PlantUML](https://plantuml.com/theme).

Mając skonfigurowaną Antore, możemy tworzyć diagramy w kodzie. Polecam tutaj podejście, w którym pliki z kodem diagramu trzymamy w dedykowanym katalogu (Antora wymusza jego nazwę: _partials_) i do treści pliku adoc dołączamy diagram w następujący sposób:

```asciidoc
[plantuml]
----
include::partial$some-diagram.puml[]
----
```

Dzięki takiemu podejściu zachowamy większą czytelności plików i będzie je łatwiej edytować.

## Budowanie dokumentacji

Mając skonfigurowaną Antore, można wykonać ostatni krok, jakim jest automatyzacja budowania i wdrażania dokumentacji na środowisko. W tym przypadku skorzystaliśmy z podejścia gdzie nasz pipeline uruchamia w obrazie dockera prosty skrypt, który uruchamia budowanie Antory:

```jshelllanguage
const fs = require('fs');
const fsExtra = require('fs-extra');
const spawn = require('child_process').spawn;
const os = require('os');
const tempDir = os.tmpdir();
const yaml_config = require('node-yaml-config');
const zipper = require('zip-local');
const replace = require('replace');

if (fs.existsSync('target')) {
    fsExtra.removeSync('target')
}

fs.mkdirSync('target');
fsExtra.copySync('./antora/playbook', 'target')

const antoraBuildEnv = process.env;
antoraBuildEnv.DOCSEARCH_ENABLED = 'true';
antoraBuildEnv.DOCSEARCH_ENGINE = 'lunr';
antoraBuildEnv.DOCSEARCH_INDEX_VERSION = 'latest';
antoraBuildEnv.FORCE_SHOW_EDIT_PAGE_LINK = 'true';
antoraBuildEnv.CACHE_DIR = tempDir + '/antora-cache/';

# uruchomienie budowania dokumentacji
const npxProcess = spawn('npx', ['antora', '--fetch', '--stacktrace', 'target/antora-playbook.yml'], {env: antoraBuildEnv})

npxProcess.stdout.on('data', data => console.log(data.toString()));
npxProcess.stderr.on('data', data => console.log(data.toString()));
npxProcess.on('error', err => console.log(err.toString()));
```

Tutaj oczywiście można proces budowania rozwiązać na wiele różnych sposobów, ten jest dla nas optymalny i pozwala dodać z poziomu skryptu dodatkowe kroki konieczne w naszym procesie CI.

## Podsumowanie

Antora jest bardzo dobrym rozwiązaniem dla projektów, które mają rozproszoną dokumentację, a muszą zapewnić jeden centralny punkt, gdzie ta dokumentacja jest dostępna. Pozwala w łatwy sposób agregować dokumenty oraz zmodyfikować wygląd docelowej strony z dokumentacją.
Jakby tego było mało, Antora pozwala na używanie rozszerzeń, które rozszerzają jej możliwości. Do dyspozycji mamy wiele stworzonych przez community, a jeżeli te się okażą niewystarczające, to w łatwy sposób możemy dopisać własne w javascript.