---
layout: post
title: "Antora-deploy wielu aplikacji"
date: 2022-03-30 21:31:32 +0100
comments: true
categories: OTHERS
---

## Co oferuje ApplicationSet?

W poprzednim wpisie na temat ArgoCD, opowiedziałem o tym do czego służy to narzędzie i jak je skonfigurować. Dzisiaj postaram się pokazać jak Argo adresuje problemy:

* wdrażania aplikacji na wiele klastrów
* wdrażanie wielu aplikacji bazując na konfiguracji w monorepo
* instalacji aplikacji multitenant

Mechanizmem pozwalającym rozwiązać powyższe problemy jest ApplicationSet. Domyślna instalacja ArgoCD zawiera dodatkowy controller, który dodaje wsparcie dla CRD (custom resource definition), jakim jest ApplicationSet. Ten nowy zasób pozwala na generowanie zbioru aplikacji za pomocą 7 predefiniowanych generatorów. Kilka przykładowych generatorów: 

* list generator-bazuje na stałej liście klucz-wartość wpisanych w yaml-u (świetny przykład użycia jest w dokumentacji)
* cluster generator-bazuje na konfiguracji klastrów, jaką dodaliśmy do ArgoCD 
* git generator-bazuje na projekcie git, który zawiera konfigurację wielu aplikacji
* matrix generator-łączy wykorzystanie dwóch lub więcej generatorów

Korzystając z powyższych generatorów możemy automatycznie wygenerować konfigurację aplikacji, jaką zrozumie ArgoCD. Potrafi to zaautomatyzować i ułatwić pracę przy bardziej skomplikowanej infrastrukturze.

## Wdrożeniu wielu aplikacji z mono repo 

W celu skorzystania z generatora aplikacji opartego o git utworzyłem w projekcie na git folder `application-set-demo` w którym znajduje się konfiguracji zbioru aplikacji:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: demo-applications
  namespace: argocd
spec:
  # konfiguracja generatora
  generators:
    - git:
        repoURL: https://github.com/klolo/spring-demo.git
        revision: HEAD
        directories:
          - path: application-set-demo/apps/*
  template: # szablony użyty do generowania konfiguracji aplikacji
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/klolo/spring-demo.git
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: apps
```

Konfiguracja ta sięga do katalogu `application-set-demo/apps` i wyszukuje w nim folderów z aplikacji. Mogą tam być wszystkie wspierane przez ArgoCD formaty (np. Kubernetes resource, Helm, Kustomize). Dla każdej z aplikacji, jaką ArgoCD znajdzie, zostanie utworzona konfiguracja, która pozwoli zarządzać aplikacją z poziomu ArgoCD. W moim przypadku w apps są dwa katalogi `some-app` i `another-app` w którym znajduje się plik z konfiguracją deploymentu kubernetes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: some-app
  name: some-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: some-app
  template:
    metadata:
      labels:
        app: some-app
    spec:
      containers:
        - image: nginxdemos/hello
          name: echoserver
```

Tak więc dla aplikacji `some-app` i `another-app` ApplicationSet wygeneruje konfigurację korzystając z szablonu, jaki został dostarczony w yamlu. Taka wygenerowana konfiguracja będzie wyglądać następująco: 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: some-app
  namespace: apps
spec:
  project: default
  source:
    repoURL: https://github.com/klolo/spring-demo.git
    targetRevision: HEAD
    path: application-set-demo/apps/some-app
  destination:
    server: https://kubernetes.default.svc
    namespace: apps
```

Żeby wszystko ruszyło zostało nam tylko podpiąć nasz zbiór aplikacji pod ArgoCD z wykorzystaniem CLI:

```
argocd app create application-set-demo \
    --repo https://github.com/klolo/spring-demo.git \
    --path ./application-set-demo \
    --dest-namespace apps \
    --dest-server https://kubernetes.default.svc \
    --directory-recurse
```

Na konsoli powinniśmy zobaczyć wszystkie aplikacje, jakie znajdują się w katalogu apps:

![GitHub Logo](/images/application-set.png)

W analogiczny sposób możemy generować zbiory aplikacji dla wielu klastrów, czy też robić multitenant deployment. Proste a zarazem bardzo użyteczne narzędzie.