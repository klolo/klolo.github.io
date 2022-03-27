---
layout: post
title: "ArgoCD jako pomysłów na Continous deployment"
date: 2022-03-29 21:31:32 +0100
comments: true
categories: OTHERS
---

Proces wgrywania nowej wersji aplikacji na środowisko często wygląda w ten sposób, że Jenkins lub jakieś inne narzędzie buduje aplikacje ze źródeł, wykonując testy i robiąc statyczną analizę kodu. Takie podejście działa, jednak ma tę wadę, że nie jest do końca _GitOps_. Nie mamy miejsca na git, gdzie przechowywana jest wersja aktualnie wgranej aplikacji. Git nie powie nam, jaki jest stan środowiska. Alternatywą do takiego podejścia jest skorzystanie z narzędzia, które będzie monitorować nasze repozytorium git i jeżeli pojawi się zmiana wersji lub konfiguracji aplikacji to automatycznie wykona upgrade na środowisku. Dzięki temu git zawsze powie nam jakie wersje aplikacji są na środowisku, jesteśmy bardziej zgodni z podejście _X as a code_ a przy okazji możemy zwiększyć bezpieczeństwo. Brałem udział przy wdrażaniu takiego narzędzia w moim projekcie i dlatego chciałbym dzisiaj opowiedzieć w kilku słowach o nim. 

Mowa o ArgoCD, narzędziu pozwalającym na ciągłe dostarczanie naszej aplikacji na środowisko oparte o Kubernetes. Pokaże jak zainstalować to narzędzie i następnie wgramy na klaster demo aplikacji. Zapraszam.

## Instalacja ArgoCD

Pora na szybki opis jak uruchomić lokalnie, na swojej maszynie ArgoCD. Oczywiście do tego celu będziesz potrzebować uruchomionego Kubernetes (Ja korzystam z minikube). Instalacja ArgoCD sprowadza się do utworzenia namespace i zainstalowanie zasobów z pliku yamkl jaki dostarcza ArgoCD (Na potrzeby demo założę że ArgoCD jest zainstalowane na lokalnym Kubernetes):

```
kubectl create namespace argocd && \
kubectl apply \
    -n argocd \
    -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Tutaj warto zwrócić uwagę na to że pobierajy skrypt z sieci i nie mamy kontroliu nad tym co znajduje się w nim. DLatego dużo lepszym podejściem jest utworzenie dedykowanego projektu na git, w którym taki skrypt umieścimy oraz ewentualnie dodatkowe customizacje do niego. Daje nam to tą zalętę że taki projekt możemy używac do konfigurowania nowych środowisk i wszystko jest zgodne z podejściem GitOps. 
 
Po instalowaniu ArgoCD możemy wejść na konsole web-ową tej apliacji. Żeby to jednak zrobić bez wystawiania Ingress, należy wykonać forward portu z klastra Kubernetes:

```
TODO
```

Jeżeli naszym oczom pojawiła się strona logowania to bardzo dobrze. Nie pozostaje nam nic innego jak wyciągnąc generowane podczas instalacji hasło administratora z jednego z Secretów:

```
TODO
```

## Spring boot demo

Przygotujmy teraz prostą aplikację springową, którą dostarczymy na Kubernetess z wykorzystaniem ArgoCD. Żeby oszczędzić trochę czasu, wygenerowałem aplikację z wykorzystaniem [spring initializer](https://start.spring.io)


## Konfiguracja deploymentu

```
TODO: deployment.yaml
```

```
TODO: service.yaml
```

## Podpięcie aplikacji pod ArgoCD

```
TODO: add application to Argo
```

## Wnioski

ArgoCD pozwala ciągłe dostawy naszej aplikacji na środowisko zgodnie z koncepcją GitOps. Przy tym oferuje praktyczny interfejs graficzny oraz CLI. Na pewno jest to aplikacja warta rozważenia w projektach które nie mają jeszcze dobrze skonfigurowanego obszaru CD. Warto wspomnieć też o możliwości rozszerzenia ArgoCD o możliwość wykonywania rollbacków aplikacji na podstawie metryk oraz wykonywania canary releses za pomocą Argo rollout. 