---
layout: post
title: "java: zwięzłe deklarowanie beanów"
date: 2021-03-07 00:38:50 +0100
comments: true
categories: JAVA
---

<!--more-->

Projekt bankowości korporacyjnej w jakim uczestniczę musi sprostać bardzo różnorodnym wymaganiom wielu klientów. Niesie to ze sobą konieczność wykorzystania w projekcie szerokiego zakresu technologii. 
Odizolowanie aspektów czysto technicznych od biznesu jest w takim przypadku koniecznością. Kto chciałby mieć domenę biznesową ubrudzoną adnotacjami i zależnościami do różnych frameworków? 
Takie zależności to prosta droga, żeby elastyczny system zmieniał się w kod legacy.

Wiele warstw systemu i oddzielenie domeny od infrastruktury nie jest jednak czymś, co możemy dostać za darmo. W przypadku naszego projektu, w którym wykorzystujemy framework Spring, 
konieczne okazało się dodanie w warstwie infrastruktury konfiguracji, która zamieni zwykłe klasy Javy w komponenty Spring, później wstrzykiwanych jako zależności. Jeszcze parę lat temu, 
na początku tego projektu, konfiguracja taka mogła zostać napisana na dwa różne sposoby: za pomocą XML lub w formie klas Javy zawierających konfigurację beanów z wykorzystaniem adnotacji Spring. 
Wykorzystaliśmy ten drugi sposób. Przykładowy kod definiujący komponenty w naszym przypadku wyglądał tak:

{% highlight java %}
@Configuration
public class EventFactoryConfiguration {
    @Bean
    public AccountsBalancesSynchronizationEventFactory accountsBalancesSynchronizationEventFactory(
            InterfaceNameResolver interfaceNameResolver,
            AccountRepository accountRepository,
            BalanceProvider balanceProvider) {
        return new AccountsBalancesSynchronizationEventFactory(
                        interfaceNameResolver,accountRepository, balanceProvider
                    );
    }

    @Bean
    public AccountSynchronizationEventFactory accountSynchronizationEventFactory(
            InterfaceNameResolver interfaceNameResolver) {
        return new AccountSynchronizationEventFactory(interfaceNameResolver);
    }

    @Bean
    public StopAccountDispositionSynchronizationEventFactory stopAccountDispositionSynchronizationEventFactory(
            InterfaceNameResolver interfaceNameResolver) {
        return new StopAccountDispositionSynchronizationEventFactory(interfaceNameResolver);
    }

    @Bean
    public AccountBalancesSynchronizationEventFactory accountBalancesSynchronizationEventFactory(
            InterfaceNameResolver interfaceNameResolver) {
        return new AccountBalancesSynchronizationEventFactory(interfaceNameResolver); 
    }
}
{% endhighlight %}

Czy ten kod nie wygląda jak zwykły boilerplate? Wygląda. A w 99% przypadków jest czymś, z czym Spring potrafiłby sobie sam poradzić, 
ale jako świadomi programiści zdecydowaliśmy się nie wpuszczać adnotacji @Component do naszej domeny. Dodatkową wadą takich deklaracji oprócz tego, 
że w dużym projekcie zajmują setki a nawet tysiące linii kodu jest fakt, że kiedy dodajemy w klasie nowy parametr konstruktora, konieczne okazuje się 
również dodanie go w klasie z konfiguracją. A o tym programista często przypomina sobie dopiero w momencie kompilowania projektu.

Jak żyć zatem? Czy trzeba kupić wygodną mechaniczną klawiaturę i pisać setki linii kodu z konfiguracją? Otóż nie! Od wersji 5, Spring daje alternatywę w formie 
dodatkowych metod do rejestrowania beanów. Dokumentacja pokazuje przykład definicji beana za pomocą takiej metody:

{% highlight java %}
GenericApplicationContext context = new GenericApplicationContext();
context.registerBean(Foo.class);
context.registerBean(Bar.class, () -> new
    Bar(context.getBean(Foo.class))
);
{% endhighlight %}

Wystarczy podać jakiej klasy beana rejestrujemy, a resztą zajmie się Spring. Nie musimy jawnie przekazywać zależności. No chyba że chcemy, wtedy również mamy taką możliwość. 
Proste, a zarazem potężne narzędzie. Otwiera to nowe możliwości pisania kodu w paradygmacie funkcyjnym. Połączenie tego podejścia i języka takiego jak na przykład Kotlin, 
zaowocowało powstaniem DSL-a do rejestrowania beanów. Osobiście uważam, że jest to najbardziej efektywny sposób pracy z Springiem. Ale co gdy z jakichkolwiek przyczyn nie możemy użyć Kotlina? 
W takim przypadku pozostaje użycie funkcyjnego sposobu deklaracji beanów z poziomu Javy, co również pozwoli usunąć bardzo dużo kodu konfiguracji. Mogą to być nawet dziesiątki tysięcy linii kodu, 
jak w naszym przypadku. Chciałbym skupić się teraz na takim właśnie przypadku i pokazać jak wpleść funkcyjny sposób konfiguracji z istniejącym projektem Spring Boot w Javie.

Zacznijmy od zebrania komponentów, które chcemy zarejestrować w formie jakiegoś rejestru. Dobrze jest w jakiś sposób pogrupować te deklaracje od razu. Ja przyjmę grupowanie layer-first, 
ale nic nie stoi na przeszkodzie, żeby użyć feature-first. Klasa z definicjami beanów z poprzedniego przykładu będzie wyglądać tak:

{% highlight java %}
public class EventFactoryBeans {
    static List<Class<?>> eventFactories = Arrays.asList(
        AccountsBalancesSynchronizationEventFactory.class,
        AccountSynchronizationEventFactory.class,
        StopAccountDispositionSynchronizationEventFactory.class,
        AccountBalancesSynchronizationEventFactory.class
    )
}
{% endhighlight %}

Czyż nie jest to czytelniejszy zapis? Większość kodu stało się niepotrzebne. Warto zauważyć, że podaję listę konkretnych klas, a nie np. interfejsów. 
Mimo to nadal jest możliwość wstrzykiwania za pomocą interfejsu, który klasa implementuje. 
Żeby jednak ta lista klas została zamieniona na beany Springa w runtime potrzebuję jeszcze wywołać gdzieś metodę registerBean. 
W tym celu dodam implementację interfejsu ApplicationContextInitializer:

{% highlight java %}
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.support.GenericApplicationContext;

public class BeanRegistrationContextInitializer implements 
    ApplicationContextInitializer<GenericApplicationContext> {
    
    private Set<Class<?>> allBeans = Arrays.asList(
        EventFactoryBeans.eventFactories
        // miejsce na reszte deklaracji komponentów
    );

    @Override
    public void initialize(GenericApplicationContext context) {
          allBeans.forEach(bean -> context.registerBean(bean));
    }

    protected void overrideBean(Class<?> oldBean, Class<?> newBean) {
        allBeans.remove(oldBean);
        allBeans.add(newBean);
    }

    protected void register(Class<?> newBean) {
        allBeans.add(newBean);
    }
}
{% endhighlight %}

Jak widać rejestracja beanów sprowadza się do wywołania metody registerBean. Spring nie musi już skanować pakietów w poszukiwaniu adnotacji, przekazujemy mu jawnie jakie komponenty ma utworzyć. 
Pozwoli to również zmniejszyć czas uruchamiania aplikacji. Na koniec pozostaje jeszcze kwestia, dlaczego w powyższym kodzie pojawiły się metody overrideBean oraz register? 
Ułatwią one nam życie, w momencie kiedy tworzymy produkt dla kilku różnych klientów i pracujemy na jednym branchu. W taki przypadku kiedy dla jakiegoś klienta (nazwijmy go roboczo X) 
pojawia się konieczność wprowadzenia customizacji możemy stworzyć dodatkowy initializer:

{% highlight java %}
public class BeanRegistrationContextInitializerX extends
    BeanRegistrationContextInitializer {
    
    public BeanRegistrationContextInitializerX() {
        overrideBean(AccountFileRepository.class, AccountFileRepositoryX.class);

        register(CustomerTypeToCustomerConverterX.class);
        register(CustomerHttpEndpointX.class);
    }
    
}
{% endhighlight %}

Jak widać możemy dowolne fragmenty procesów biznesowych nadpisywać specyficznym dla klienta kodem. Pozostaje jeszcze pytanie jak powiadomić Spring Boota o tym, że ma uruchomić initializer:

{% highlight java %}
public class Application {
    public static void main(final String[] args) {
        new SpringApplicationBuilder(Application.class)
                .initializers(new BeanRegistrationContextInitializer())
                .run(args);
    }
}
{% endhighlight %}

Natomiast w testach integracyjnych możemy skorzystać z adnotacji: @ContextConfiguration(initializers = BeanRegistrationContextInitializer.class)

Dodatkową zaletą powyższego sposobu konfiguracji jest to, że możemy wprowadzać go stopniowo. Kod zawierający konfigurację z wykorzystaniem adnotacji @Bean będzie nadal uruchamiany 
i możemy ciągle z niego korzystać. Jeżeli w Twoim projekcie nadal jest dużo kodu, w którym deklarujesz komponenty z wykorzystaniem adnotacji @Bean, zachęcam Cię do spróbowania alternatywy 
w formie DSLa w języku Kotlin, bądź też w formie prostego mechanizmu jaki przedstawiłem.