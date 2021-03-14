---
layout: post
title: "spring boot: Wystawianie endpointów jednej aplikacji na wielu portach"
date: 2021-03-07 21:31:32 +0100
comments: true
categories: JAVA SPRING
---

Standardem jest tworzenie aplikacji webowych, gdzie wystawiamy usługi na jednym wybranym porcie. 
Czasami jednak życie może nas zmusić do stworzenia jednej aplikacji słuchającej na większej ilości portów.
 
<!--more-->
 
Przykładem może być sytuacja, gdy chcemy publiczne api mieć na innym porcie niż api przeznaczone do wykorzystywania wewnątrz firmy. 
Idealnym rozwiązaniem z punktu widzenia bezpieczeństwa byłoby uruchomienie dwóch różnych instancji aplikacji, gdzie ta przeznaczona dla ruchu publicznego nie udostępniałaby usług wewnętrznych.
Gdy jednak z jakichś powodów np. chęć uproszczenia infrastruktury na środowisku testowym, chcemy mieć jedną aplikację z różnymi 
zestawami usług na różnych portach, to możemy to dość prosto to skonfigurować....ale mała uwaga w tym miejscu: **poniższy sposób
zakłada, że Tomcat jest używanym przez spring boot-a serwerem**. Dla innych serwerów konfiguracja ta może inaczej wyglądać lub nie być możliwa.
Alternatywą jest bardziej skomplikowane rozwiązanie, jakim jest uruchomienie osobnego kontekstu aplikacji na osobnym porcie.
Rozwiązanie takie używane jest używane przez spring-a, kiedy ustawimy properte `management.server.port` na inną wartość niż port, na którym uruchamiamy aplikację.
Ciekawskim mogę polecić klasę w kodach Actuatora: `ManagementContextAutoConfiguration.java`.

Zabierzmy się zatem za skonfigurowanie aplikacji tak, żeby na porcie 8080 były wystawione usługi publiczne, a na porcie 5555 usługi wewnętrzne.
Spring boot pozwala nam skonfigurować uruchamiany serwer poprzez dedykowane property w konfiguracji, niestety te tutaj są niewystarczające. Drugim sposobem na dokonfigurowanie 
 serwera jest implementacje interfejsu [`WebServerFactoryCustomizer`](https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/howto-embedded-web-servers.html#howto-configure-webserver),
 gdzie uzyskujemy dostęp do klasy `WebServerFactory`. Ten sposób właśnie wykorzystałem do rejestracji dodatkowego connectora Tomcat-a, słuchającego na dedykowanym porcie: 

{% highlight kotlin %}
@Configuration
class DemoConfiguration : 
        WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    override fun customize(factory: TomcatServletWebServerFactory) {
        val connector = Connector(Http11NioProtocol::class.simpleName)
        connector.scheme = "http"
        connector.port = 5555
        factory.addAdditionalTomcatConnectors(connector)
    }

}
{% endhighlight %}

Dzięki takiej konfiguracji serwer uruchomi się na domyślnym porcie `8080` oraz na `5555`. Jedna oba porty obsłużą 
wszystkie requesty, czego byśmy nie chcieli. W celu uniemożliwienia wywoływania wewnętrznych usług na porcie nieprzeznaczonym do tego
możemy oprócz konfiguracji spring security dodać również prosty filtr, który będzie wycinał ruch na podstawie ścieżki do usługi:

{% highlight kotlin %}
@Component
class IntegrationApiAccessFilter : Filter {

    override fun doFilter(request: ServletRequest, 
                          response: ServletResponse, chain: FilterChain) {
        val isInternalService = 
                (request as RequestFacade).requestURI.startsWith("/internal")
        val incorrectPortForInternal= request.serverPort.toString() != 5555

        if (isInternalService && incorrectPortForIntegration) {
            (response as ResponseFacade).sendError(404)
            return
        }

        chain.doFilter(request, response)
    }

}
{% endhighlight %}

To wszystko, mamy aplikację wystawiającą na dwóch różnych portach, różne zestawy endpointów. Pamiętaj, jednak że taka konfiguracja nie jest czymś standardowym 
i należy się dobrze zastanowić czy na pewno problem należy tak właśnie rozwiązać.
