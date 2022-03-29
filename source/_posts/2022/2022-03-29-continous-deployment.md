---
layout: post
title: "ArgoCD jako pomysł na continuous deployment"
date: 2022-03-29 21:31:32 +0100
comments: true
categories: OTHERS
---

Proces wgrywania nowej wersji aplikacji na środowisko często wygląda w ten sposób, że Jenkins lub jakieś inne narzędzie buduje aplikacje ze źródeł, wykonując testy, robiąc statyczną analizę kodu i push na środowisko. Takie podejście działa, jednak ma tę wadę, że nie jest do końca _GitOps_. Nie mamy miejsca na git, gdzie przechowywana jest wersja aktualnie wgranej aplikacji. Git nie powie nam, jaki jest stan środowiska. Alternatywą do takiego podejścia jest skorzystanie z narzędzia, które będzie monitorować nasze repozytorium git i jeżeli pojawi się zmiana wersji lub konfiguracji aplikacji to automatycznie wykona upgrade na środowisku. Dzięki temu git zawsze powie nam jakie wersje aplikacji są na środowisku, jesteśmy bardziej zgodni z podejściem _X as a code_ a przy okazji możemy zwiększyć bezpieczeństwo. Brałem udział przy wdrażaniu takiego narzędzia w moim projekcie i dlatego chciałbym dzisiaj opowiedzieć w kilku słowach o nim. 

Mowa o ArgoCD, narzędziu pozwalającym na ciągłe dostarczanie naszej aplikacji na środowisko oparte o Kubernetes. Pokaże jak zainstalować to narzędzie i następnie wgramy je na klaster demo aplikacji.

## Instalacja ArgoCD

Pora na szybki opis jak uruchomić lokalnie, na swojej maszynie ArgoCD. Oczywiście do tego celu będziemy potrzebować uruchomionego Kubernetes-a (Ja korzystam z Minikube). Instalacja ArgoCD sprowadza się do utworzenia namespace i zainstalowanie zasobów z pliku yaml, jaki dostarcza ArgoCD:

```
kubectl create namespace argocd && \
kubectl apply \
    -n argocd \
    -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Tutaj warto zwrócić uwagę na to, że pobieramy skrypt z sieci i nie mamy kontroli nad tym, co znajduje się w środku. Dużo lepszym podejściem jest utworzenie dedykowanego projektu na git, w którym taki skrypt umieścimy wraz z ewentualnymi dodatkowymi customizacjami do niego. Daje nam to tą zaletę, że taki projekt możemy używać do konfigurowania nowych środowisk i wszystko jest zgodne z podejściem GitOps. 
 
Po zainstalowaniu ArgoCD możemy wejść na konsole web-ową tej aplikacji. Żeby to jednak zrobić bez konfigurowania Ingress, należy wykonać forward portu z klastra Kubernetes:

```
kubectl port-forward -n argocd service/argocd-server 8080:443
```

Jeżeli naszym oczom pojawiła się strona logowania to bardzo dobrze.

![GitHub Logo](/images/argo-ui.png)

Nie pozostaje nam nic innego jak wyciągnąć generowane podczas instalacji hasło administratora z jednego z secretów:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Po zalogowaniu, w przeglądarce możemy wyklikać wszystkie rzeczy, jakie będą dalej opisane, jednak rekomenduje zainstalowanie sobie ArgoCD CLI i konfigurację aplikacji za pomocą konsoli. Będzie łatwiej to wykorzystać z poziomu Jenkins/Gitlab. CLI można zainstalować poleceniem: `brew install argocd`

## Spring boot demo

Mając uruchomione Argom możemy stworzyć prostą aplikację springową, którą dostarczymy na Kubernetes z wykorzystaniem ArgoCD. Żeby oszczędzić trochę czasu, wygenerowałem aplikację z wykorzystaniem [spring initializer](https://start.spring.io) i dodałem do niej zależność do spring-web oraz wrzuciłem najprostszy możliwy endpoint:

```kotlin
@RestController
class HelloEndpoint {
    @GetMapping("foo")
    fun foo() = "hello mister"
}
```

Sprawdzam czy aplikacja poprawnie się buduje do jar i następnie mogę przystąpić do dodania konfiguracji deploymentu.

## Konteneryzacja aplikacji

Pierwszym krokiem do uruchomienia naszej aplikacji na Kubernetes jest jej konteneryzacja. Tworzymy bardzo prosty plik dockerfile w katalogu głównym projektu:

```yaml
FROM adoptopenjdk/openjdk11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

> Żeby minikube miał dostęp do obrazów które zbudujesz lokalnie dodatkowo wykonaj:
> `eval $(minikube -p minikube docker-env)`

Następnie budujemy obraz Dockera za pomocą polecenia: 

```
docker build . -t spring-demo:1.0.0
```

Po zakończeniu budowania powinniśmy mieć nowy obraz na liście lokalnych obrazów: 
![GitHub Logo](/images/docker-images.png)

## Konfiguracja deploymentu

Żeby nasza aplikacja prawidłowo zadziałała na Kubernetes w katalogu *manifests* tworzę pliki z konfiguracją deploymentu:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-demo
  namespace: apps
  labels:
    app: spring-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: spring-demo
  template:
    metadata:
      labels:
        app: spring-demo
    spec:
      containers:
        - name: spring-demo
          image: spring-demo:1.0.0
          imagePullPolicy: Never # for local docker with minikube
          ports:
            - containerPort: 8080
```

Oraz service dla powyższego deploymentu: 

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: spring-demo-service
  namespace: apps
spec:
  selector:
    app: spring-demo
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
```

## Podpięcie aplikacji pod ArgoCD

Najwyższa pora poinformować ArgoCD o tym, że mamy aplikację, którą powinien się zająć. Aplikację chcę mieć zainstalowaną w dedykowanym namespace za pomocą CLI Argo. Żeby jednak CLI zadziałało konieczne jest najpierw zalogowanie: 

```
argocd login localhost:8080
```

Teraz tworzymy namespace i nową aplikację po stronie Argo, używając namiarów na projekt w git: 

```
kubectl create namespace apps && 
argocd app create spring-demo \
    --repo https://github.com/klolo/spring-demo.git \
    --path ./manifests \
    --dest-namespace apps \
    --dest-server https://kubernetes.default.svc \
    --directory-recurse
```

![GitHub Logo](/images/argo-deployed.png)

Aplikacja wraz z zasobami Kubernetesa powinna się pojawić na konsoli Argo. I teraz najważniejsza rzecz...jeżeli na repozytorium git pojawi się jakaś zmiana w katalkogu manifests to ArgoCD potrafi zaktualizować stan środowiska, tak żeby odpowiadał temu, co jest w kodzie. Możemy przykładowo podbić wersję obrazu dockerowego a Argo wykonan za nas upgrade aplikacji. 

## Wnioski

ArgoCD pozwala na ciągłe dostawy naszej aplikacji na środowisko zgodnie z koncepcją GitOps. Przy tym oferuje praktyczny interfejs graficzny oraz CLI. Na pewno jest to aplikacja warta rozważenia w projektach, które nie mają jeszcze dobrze skonfigurowanego obszaru CD. Warto wspomnieć też o możliwości rozszerzenia ArgoCD o możliwość wykonywania rollbacków aplikacji na podstawie metryk oraz wykonywania canary releses za pomocą Argo rollout. 

*PS. Wyszedł bardzo obszerny wpis, a to tylko wierzchołek góry lodowej....prawdopodobnie powinienem jeszcze napisać coś na temat Argo 🤫*