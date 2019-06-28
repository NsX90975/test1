## Plan prezentacji
1. [Wprowadzenie do standardu IHE XDS.b oraz omówienie wybranych transakcji](#ihe)
2. [Architektura SGZ EDM w P1](#arch)
3. [Wprowadzenie do IPF](#ipf)
4. [Wprowadzenie do Apache Camel](#camel)
5. [Wykorzystanie języka Groovy w projekcie SGZ EDM](#groovy)
6. [Omówienie przetwarzania komunikatu na przykładzie transakcji ITI-42 i ITI-18](#iti)
7. [Omówienie nowego podejścia do testów SoapUI oraz omówienie narzędzi zewnętrznych wspomagających testy SGZ EDM](#testy)

<a name="ihe"></a>
## Wprowadzenie do standardu IHE XDS.b oraz omówienie wybranych transakcji

#### Podstawy standardu IHE XDS.b
 ######IHE – Integrating the Health Enterprise 
Działania IHE są prowadzone zgodnie ze zdefiniowanym procesem: 
1. Eksperci z dziedzin klinicznych i profesjonaliści IT wspólnie definiują kluczowe przypadki użycia dla wymiany informacji medycznej. 
2. Aby zrealizować zdefiniowane przypadki użycia tworzone są profile integracji (IHE profiles), przy użyciu powszechnie używanych standardów (w tym standardów interoperacyjności) i dobrych praktyk. 
3. Profile integracji są implementowane w systemach informatycznych. 
4. IHE testuje tworzone rozwiązania pod kątem zgodności z profilami integracji poprzez organizowanie cyklu starannie planowanych i nadzorowanych spotkań.

 ######Profile integracyjne IHE
![profile_ihe](obrazki/profile_ihe.png)
 ######Profile integracji IHE zawierają:
 - definicję aktorów i przypadków użycia wymiany informacji medycznej w kontekście określonej domeny danych medycznych, 
 - specyfikację interfejsów komponentów systemu, 
 - określenie poszczególnych standardów interoperacyjności lub specyfikacji technicznych wybranych dla implementacji danego scenariusz komunikacji.
 ######Profil treści IHE jest doprecyzowaniem danego standardu wymiany danych poprzez:
 - określenie dodatkowych wymagań i ograniczeń struktury obiektów, dokumentów lub komunikatów 
 - zdefiniowanie słowników terminologicznych i zbiorów wartości na ich podstawie
 ######XDS - Cross-Enterprise Document Sharing 
 - Specyfikacja umożliwiająca powszechną wymianę dokumentów medycznych w formie elektronicznej pomiędzy różnymi organizacjami, których systemy udostępniają elektroniczny rekord medyczny pacjenta. 
 - Wyróżnione są dwa główne komponenty uczestniczące w wymianie dokumentów 
 - Repozytorium dokumentów (Document Repository) – odpowiedzialne za przechowywanie instancji dokumentów w bezpieczny i trwały sposób. 
 - Rejestr dokumentów (Document Registry) – odpowiedzialny za przechowywanie informacji o dokumentach, umożliwiającej ich wyszukiwanie oraz pobranie ich zawartości bez względu na repozytorium, w którym się znajdują. 
 - Dokumenty są udostępniane przez jedno lub wiele źródeł dokumentów (Document Sources). 
 - Dokumenty są wyszukiwane i pobierane przez konsumentów dokumentów (Document Consumers).
######HL7 CDA (Clinical Document Architecture) 
 - Standard przeznaczony do tworzenia i wymiany dokumentów medycznych w postaci elektronicznej.
 - Zawiera rozbudowany model danych umożliwiający zapisanie: 
   1. elementów nagłówka dokumentu (np. dane pacjenta, usługodawcy, wystawcy dokumentu) 
   2. treść dokumentu z podziałem na poszczególne sekcje (prezentowana odbiorcy dokumentu w formie tekstowej)
   3. złożonych wyrażeń klinicznych , które mogą być jednoznacznie interpretowane przez systemy informatyczne. 
######HL7 FHIR (Fast Helathcare Interoperability Resources) 
- Standard służący do wymiany danych medycznych między systemami.
- Tworzony z myślą o twórcach oprogramowania oraz łatwości implementacji z zastosowaniem powszechnie stosowanych technologii. 
- Definiuje zasoby – podstawowe atomy wymiany informacji w zakresie najczęstszych przypadków użycia w ochronie zdrowia
- Weryfikacja zgodności polega na walidacji danego zasobu z wykorzystaniem: 
  1. podstawowej definicji struktury dla tego zasobu, 
  2. definicji profilu dla zasobu, określającego dodatkowe wymagania i ograniczenia dla jego struktury. 
 - Funkcjonalność walidacji zasobów jest elementem każdej referencyjnej implementacji standardu HL7 FHIR dla różnych środowisk programistycznych, 
 komponentem realizującym walidację zasobów jest serwer FHIR.
![wymiana_dokumentow_standard](obrazki/ihe_wymiana_dokumentow_standard.png)

######Aktorzy, biorący udział w wymianie dokumentów medycznych (opisane strzałki oznaczają kierunek wykonania transakcji, tj. wywołania operacji WebService):
- Document Source – źródło dokumentów, tj. system usługodawcy, w ramach którego dokumenty wystawiono
- Document Repository – repozytorium dokumentów, przy czym objęcie dodatkową ramką aktora wraz z Document Source oznacza, że repozytorium jest z punktu widzenia P1 zintegrowane z usługodawcą wystawiającym dokument i to niezależnie czy ten usługodawca wykorzystuje do prowadzenia repozytorium własny system informatyczny, system regionalny, czy też komercyjny system zewnętrzny. 
- Document Registry – rejestr dokumentów prowadzony w P1, jedna z funkcjonalności Systemu P1. Do zaindeksowania dokumentu w P1 służy **operacja zapisu indeksów** (rozszerzona o dane zdarzenia medycznego) „**RegisterDocumentSet**” oznaczona w specyfikacji profilu symbolem **ITI-42**; 
- Document Consumer – system pracownika usługodawcy wyszukującego i pobierającego dokumenty medyczne. Do **wyszukania indeksów w rejestrze** służy operacja „**RegistryStoredQuery**” oznaczona w specyfikacji profilu symbolem **ITI-18**, przy czym nazwa ta oznacza, że rejestr posiada predefiniowane zapytania rozróżniane identyfikatorem, zawierające z góry określoną liczbę atrybutów wyszukiwania. W wyniku wyszukiwania pracownik usługodawcy otrzymuje indeksy, do których posiada prawo dostępu, wraz z informacją o statusie dostępności poszczególnych dokumentów medycznych. 
- DocumentAdministrator – aktor pominięty w powyższym standardowym diagramie, implementowany w systemach źródłowych oraz w systemie zarządzającym rejestrem dokumentów, w tym implementowany w P1. Administrowanie w polskim rozwiązaniu ogranicza się do implementacji operacji „**UpdateDocumentSet**” oznaczonej w specyfikacji profilu symbolem **ITI-57**, wywoływanej na danych rejestru celem **modyfikacji zawartości indeksu** w kontekście jednego pacjenta. 
- Record Audit Event (ATNA) – Ta transakcja służy do **zgłaszania zdarzeń audytowych** zapisywanych w repozytorium audytu. W specyfikacji profilu jest oznaczona symbolem **ITI-20**.

<a name="arch"></a>
## Architektura SGZ EDM w P1

Na wczesnym etapie prac związanych z analizą oraz architekturą, został opracowany diagram przedstawiający wizję docelowej wersji systemu obsługującego: 
* wymianę elektronicznej dokumentacji medycznej, 
* operacje związane ze zdarzeniami medycznymi. 

![propozycja_architektury_docelowej](obrazki/p1_propozycja_architektury_docelowej.png)

W kontekście EDM należy wyróżnić następujące komponenty P1:
* Rejestr XDS.b P1
    * Udostępnia usługi do rejestrowania oraz aktualizacji indeksu dokumentacji (ITI-42 i ITI-57)
    * Udostępnia usługę do wyszukiwania indeksu dokumentacji (ITI-18)
    * Rejestruje zdarzenia audytu zgodnie z ATNA (ITI-20)
    * Korzysta z SAZ w celu: 
        * depersonalizacji oraz personalizacji danych usługobiorcy 
        * ustalenia czy może przekazać systemowi zewnętrznemu informacje o indeksie dokumentacji 
    * Korzysta z SOR w celu weryfikacji poprawności danych
* SGS / SGR
    * Przechowuje dokumentację medyczną
    * Udostępnia usługę do pobrania dokumentacji medycznej (ITI-43)
    * Rejestruje zdarzenia audytu zgodnie z ATNA (ITI-20)
    * Korzysta z SAZ w celu: 
        * personalizacji danych usługobiorcy 
        * ustalenia czy może przekazać systemowi zewnętrznemu dokumentację medyczną 
* IKP 
    * Umożliwia usługobiorcy wyszukanie indeksów (swoich oraz podopiecznego/mocodawcy)
    * Umożliwia pobranie dokumentacji medycznej na podstawie indeksu
    * Umożliwia uzyskanie informacji o tym kto i kiedy wyszukał indeksy EDM, które jego dotyczą
    * Umożliwia zarządzanie zgodami
* SAZ
    * Umożliwia depersonalizację oraz personalizację danych usługobiorcy
    * Przechowuje zgody oraz umożliwia ustalenie czy można przekazać systemowi zewnętrznemu informacje o indeksie oraz dokumentację medyczną
* SAU
    * Przechowuje logi zdarzeń zarejestrowanych zgodnie z ATNA
    * Przechowuje logi UDO    

<a name="ipf"></a>
## Wprowadzenie do IPF
### Opis
IPF (Open eHealth Integration Platform) jest platformą dostarczającą usługi umożliwiające integrację rozwiązań informatycznych w obszarze ochrony zdrowia. W szczególności posiada interfejsy dla transakcji określonych w profilach IHE.

### Wsparcie IHE
W kontekście IHE można wyróżnić następujące pojęcia:
* Aktor - komponent lub system informacyjny, który tworzy lub wykorzystuje informacje w celu realizacji działań
* Transakcja - interakcja między aktorami, która polega na przekazywaniu wymaganych informacji w ustandaryzowany sposób
* Profil - zestaw aktorów oraz transakcji, który umożliwia realizację rzeczywistych celów (procesów biznesowych)

W kontekście IPF:
* Aktorem jest komponent udostęniający usługę (Apache Camel endpoint) oraz komponent/system, który jest konsumentem tej usługi
* Za przetwarzanie transakcji odpowiada komponent Apache Camel, który odpowiada za przetworzenie komunikatu żądania oraz odpowiedzi zgodnie ze standardem
* Profil to grupa komponentów Apache Camel

Więcej informacji można znaleźć pod [adresem](https://oehf.github.io/ipf-docs/docs/ihe/) .

### Wsparcie transakcji
IPF wspiera kilkadziesiąt transakcji. W szczególności interesujące są dla te wymienione poniżej:


| Transakcja | Profil | Opis | Komponent IPF | Transport oraz format komunikatu  | Moduł IPF |
|---|---|---|---|---|---|
| ITI-18 | XDS | Registry Stored Query | xds-iti18 | SOAP/HTTP(S), ebXML | ipf-platform-camel-ihe-xds |
| ITI-20 | ATNA | Record Audit Event | n/a | Syslog/TLS, DICOM/IHE Audit Message | ipf-platform-camel-ihe-atna |
| ITI-42 | XDS | Register Document Set | xds-iti42 | SOAP/HTTP(S), ebXML | ipf-platform-camel-ihe-xds |
| ITI-57 | XDS | Update Document Set | xds-iti57 | SOAP/HTTP(S), ebXML | ipf-platform-camel-ihe-xds |

Więcej informacji można znaleźć pod [adresem](https://oehf.github.io/ipf-docs/docs/ihe/) .

### Spring Boot
IPF dostarcza moduły Spring Boot starters, które umożliwiają autokonfigurację podczas uruchamiania aplikacji.

M.in. dostępne są moduły:
* ipf-atna-spring-boot-starter
* ipf-xds-spring-boot-starter

Więcej informacji można znaleźć pod [adresem](https://oehf.github.io/ipf-docs/docs/boot/) .

Konfiguracja dostarczana przez w/w moduły może być sterowana przy pomocy zdefiniowanych parametrów.

##### ipf-atna-spring-boot-starter

| Parametr (prefiks *ipf.atna*) | Opis | Domyślna wartość |
|---|---|---|
| audit-enabled | Whether auditing is enabled | false |
| audit-repository-host | Host of the ATNA repository to send the events to | localhost |
| audit-repository-port | Port of the ATNA repository to send the events to | 514 |
| audit-repository-transport | Wire transport format (UDP, TLS) | UDP |
| audit-source-id | Source ID for ATNA events | ${spring.application.name} |
| audit-enterprise-site-id | Enterprise Site ID for ATNA events | |
| include-participants-from-response | Whether to include (patient) participants from responses as well | false |
| audit-source-type | Type of Audit Source | 4 (ApplicationServerProcess) |
| audit-queue-class | Queue implementation for auditing | org.openehealth.ipf.commons.audit.queue.SynchronousAuditMessageQueue |
| audit-sender-class | ATNA sender implementation | as indicated by audit-repository-transport |
| audit-exception-handler-class | Exception handler implementation | org.openehealth.ipf.commons.audit.handler.LoggingAuditExceptionHandler |
| audit-value-if-missing | Value used for mandatory audit attributes that are not set | UNKNOWN |

Więcej informacji można znaleźć pod [adresem](https://oehf.github.io/ipf-docs/docs/boot-atna/) .

##### ipf-xds-spring-boot-starter

| Parametr (prefiks *ipf.xds*) | Opis | Domyślna wartość |
|---|---|---|
| caching | Whether to set up a cache for Asynchronous Web Service exchange | false |
Więcej informacji można znaleźć pod [adresem](https://oehf.github.io/ipf-docs/docs/boot-xds/) .

<br/>
Dodatkowo moduł zależy od modułu cxf-spring-boot-starter-jaxws, który można konfigurować przy pomocy parametrów:

| Parametr (prefiks *cxf*) | Opis | Domyślna wartość |
|---|---|---|
| path | Path that serves as the base URI for the services | /services |
| servlet.init | empty map | Optional servlet init parameters |
| servlet.load-on-startup | -1 | Startup order |

Więcej informacji można znaleźć pod [adresem](https://cxf.apache.org/docs/springboot.html) .

### Rejestracja komponentów Apache Camel
Komponenty Apache Camel, które są odpowiedzialne m.in. za realizację obsługi komunikatów transakcji, są rejestrowane automatycznie.

Jest to możliwe, ponieważ Apache Camel wykorzystuje mechanizm Service Provider Interface (znany także jako Service Provider pattern lub Service Provider framework), który umożliwia pisanie rozszerzalnych aplikacji.

Ważnymi składowymi SPI są:
* interfejs, np.

```
package org.apache.camel;
...
public interface Component extends CamelContextAware {

    Endpoint createEndpoint(String uri) throws Exception;
    ...   
}
```

* implementacja, np. `org.openehealth.ipf.platform.camel.ihe.xds.iti42.Iti42Component`
* rejestracja implementacji, np.

![rejestracja_komponentow](obrazki/ipf_xds_spi_rejestracja_komponentow.png)

```
# Camel registration for the xds-iti42 component
 
class=org.openehealth.ipf.platform.camel.ihe.xds.iti42.Iti42Component
```

* wykorzystanie implementacji - realizowane przy pomocy klasy `java.util.ServiceLoader`


### Routing
IPF dostarcza gotowe usługi sieciowe zgodne ze standardami IHE. 

Nie określa jednak w jaki sposób dokładnie powinny być obsługiwane komunikaty dla danych transakcji. Sposób obsługi komunikatów musi zostać określony przez komponent wykorzystujący IPF (w szczególności dotyczy to SGZ). 

Określa się go przy pomocy definicji tras (routing'ów). 

Apache Camel umożliwia definiowanie routingu przy pomocy Domain Specific Language (np. Java DSL, Groovy DSL - informacje nt. innych języków można znaleźć pod [adresem](https://camel.apache.org/dsl.html)).
 
W celu zdefiniowania routingu, należy utworzyć własny komponent, który będzie odpowiedzialny za konfigurację routingu, np.

```
@Component
class Iti42RouteBuilder extends RouteBuilder {

    @Override
    void configure() {
        from('xds-iti42:xds-iti42').
                .process(myProcessor)
        ...    
    }
}
```

Więcej informacji można znaleźć w [tutorialu dla XDS](https://oehf.github.io/ipf/ipf-tutorials-xds/index.html) .

### Konwersja

W kontekście IHE XDS.b, IPF posiada trzy rodzaje modeli:
* model transportowy ebXML
* model logiczny XDS
* model logiczny ebXML

Obecnie realizowane są dwa typy dwustronnej konwersji:
* model transportowy ebXML <-> model logiczny XDS
* model logiczny XDS <-> model logiczny ebXML

Pierwsza z nich jest realizowana niejawnie. W tutorialu, w przykładzie konfiguracji routingu dla transakcji ITI-42, można znaleźć fragment:

```
        from('xds-iti42:xds-iti42')
            ...
            .transform ( {exchange, type ->
                [ 'req': exchange.in.getBody(RegisterDocumentSet.class), 'uuidMap': [:] ]} as Expression
            )
```
przy wywołaniu: `exchange.in.getBody(RegisterDocumentSet.class)` mechanizm na podstawie klasy parametru oraz typu zwrotnego ustala, że powinien wywołać metodę konwertującą:

`org.openehealth.ipf.platform.camel.ihe.xds.core.converters.EbXML30Converters.convert(org.openehealth.ipf.commons.ihe.xds.core.stub.ebrs30.lcm.SubmitObjectsRequest)`

Wszystkie metody `EbXML30Converters` są oznaczone adnotacją `@org.apache.camel.Converter`. A sama klasa `EbXML30Converters` jest rejestrowana przy pomocy mechanizmu SPI, który został omówiony wcześniej.

![rejestracja_konwertera](obrazki/ipf_xds_spi_rejestracja_konwerterow.png)

```
org.openehealth.ipf.platform.camel.ihe.xds.core.converters.EbXML30Converters
```

Drugi mechanizm konwersji jest wywoływany jawnie. Składa się z zestawu klas konwerterów:

![konwertery_xds_ebxml](obrazki/ipf_xds_ebxml_konwertery.png)

Wykorzystują one:
* interfejs fabryki obiektów ebXML 
* interfejsy reprezentujące model logiczny ebXML

Takie podejście pozwala na wykorzystanie istniejących konwerterów dla różnych implementacji modelu ebXML.

### Walidacja 
IPF posiada zestaw walidatorów, które umożliwiają weryfikację poprawności struktury komunikatów żądań i odpowiedzi (model transportowy ebXML).

Do zestawu należą zarówno: 
* walidatory proste, które weryfikują pojedyncze elementy
* walidatory złożone, które weryfikują np. obiekty documentEntry
* walidatory żądań i odpowiedzi, które weryfikują komunikaty żądań i odpowiedzi przekazywane w ramach transakcji

### Utrwalanie danych
IPF nie posiada mechanizmu utrwalania przekazywanych informacji. 

W przykładzie, można zauważyć, że dane pochodzące z żądań zapisywane są w strukturach, przechowywanych w pamięci. 

Zatem konieczne jest utworzenie modelu danych, implementacji warstwy utrwalającej i odczytującej dane oraz instalacja oraz konfiguracja zewnętrznej bazy danych.

Z uwagi na brak narzuconego modelu oraz sposobu utrwalania danych, istnieje możliwość zrealizowania własnego w oparciu o zarówno relacyjną bazę danych, jak i bazę danych NoSQL.

<a name="camel"></a>
## Wprowadzenie do Apache Camel
Bla bla

<a name="groovy"></a>
## Wykorzystanie języka Groovy w projekcie SGZ EDM

#### Podstawy
##### Wstęp
Groovy (właściwie Apache Groovy) to obiektowy, skryptowy język programowania działający na maszynie wirtualnej Javy 
(jego kod kompiluje się do kodu bajtowego Javy). Tak jak inne języki oparte o JVM, Groovy może być używany zamiennie z Javą 
oraz wykorzystywać jej biblioteki. 

##### Podstawowe cechy języka Groovy
* **Kompatybilny z Javą** - kod napisany w Groovy może korzystać z bibliotek napisanych w Javie, jest również kompatybilny z takimi frameworkami jak np. Spring.

* **Z założenia wygodniejszy niż Java** – dla osoby już zaznajomionej z językiem kod napisany w Groovy jest mniej rozwlekły, 
prostszy oraz czytelniejszy od analogicznego kodu napisanego w Javie. Mnogość dodatkowych konstrukcji składniowych pozwala pisać krótszy i bardziej zwięzły kod.

* **Opcjonalnie typowany** – wspiera typowanie dynamiczne jak i statyczne. W czasie kompilacji sprawdzanie typu wykonywane jest domyślnie na minimalnym poziomie. 
Patrząc na to od strony praktycznej, w większości przypadków możemy np. pozbyć się rzutowania, 
podczas deklarowania typu zmiennej lub metody posłużyć słowem kluczowym „def” bądź zupełnie pominąć typy parametrów np. w sygnaturach metod lub skryptach.  

* **Wykorzystywany jako język skryptowy** – w przeciwieństwie do Javy, nieskompilowany wcześniej plik kodu źródłowego 
(nawet bez utworzenia w nim klasy) może być wykonany jako skrypt. Kompiler Groovy sam skompiluje i wytworzy odpowiednie struktury kodu podczas jego uruchomienia.

>Przykład metody:
>(jednocześnie kod ten jest przykładem wykonywalnego skryptu)
 ```groovy
     def calculateSum(a, b) {
         a + b
     }
     def sum = calculateSum 3, 2
     assert 5 == sum​
 ```
 
##### Typowe komercyjne zastosowanie: 
Ze względu na swoje właściwości Groovy wykorzystywany jest najczęściej do pisania skryptów automatyzujących prace (Jenkins), 
rozszerzania funkcjonalności bibliotek oraz narzędzi (SoapUI), pisania testów (Spock) oraz klas „utilowych”.

#### Wykorzystanie języka Groovy w SGZ EDM
W przypadku komponentu SGZ język Groovy wykorzystywany jest szczególnie w łączności z Apache Camel. 
Zastosowany został m.in. w konfiguracji, rozszerzaniu oraz testowaniu routingu opisującego obsługę żądań XSD.b.

> Aby móc wykorzystać w pełni możliwości języka Groovy oferowane przez Apache Camel należy do projektu dodać zależność „camel-groovy”.

##### Groovy w konfiguracji routingu Apache Camel
Domyślnie do konfiguracji routingu w Apache Camel wykorzystywany jest oparty na Javie mechanizm DSL zaprojektowany w stylu fluentowego buildera. 
Camel dostarcza implementację tego mechanizmu również w innych językach w tym m.in. w Groovy. 

Zaletami wykorzystania Grooviego w tym wypadku są:

 * **Czytelniejszy kod** – poprzez prostszy dostęp do zmiennych obiektów oraz łatwiejszą implementację anonimowych bloków kodu.

> Przykład kodu konfiguracji routingu w języku Java:
 ```Java
     ...
        from("direct:test")
           .transform(new Expression() {
              @Override
              public Object evaluate(Exchange e) {
                 return new StringBuffer(e.getIn().getBody().toString()).reverse().toString();
              }
           })
           .process(new Processor() {
              @Override
              public void process(Exchange e) {
                System.out.println(e.getIn().getBody());
              }
           });
     ...
 ```

> Analogiczny kod napisany w Groovy:
 ```groovy
    ...
       from('direct:test')
          .transform { it.in.body.reverse() }
          .process { println it.in.body }
    ...
 ```

 * **Wykorzystanie Grooviowych domknięć (colsures)** - które posłużyć mogą jako implementacje elementów konfiguracji routingu takich jak: 
Processor, Expression, Predicate czy Aggregation Strategy.

> Przykład wykorzystania Groovy Closures:
 ```groovy
    ...
       def someValue
       // Closure stored in a variable
       def predicate = { Exchange e -> e.in.body != someValue }
    ...
       from('direct:test')
          .filter(predicate)
    ...
 ```

 * **Możliwość rozbudowania mechanizmu DSL** – rozszerzanie go o dodatkowe metody dodawane w czasie działania programu. 
Jest to możliwe dzięki istniejącemu w Groovy modułowi rozszerzenia (extension module), który pozwala definiować metody rozszerzeń na istniejących klasach.

> Przykład wykorzystania rozszerzeń w SGZ EDM - deskryptor umieszczony w META-INF/services:
 ```properties
    moduleName=RoutingRozszerzenia
    moduleVersion=2.5
    extensionClasses=pl.gov.csioz.p1.sgz.edm.routing.RoutingRozszerzenia
    staticExtensionClasses=
 ```
> Rozszerzone metody procesowania routingu:
 ```groovy
    static ProcessorDefinition zapisz(ProcessorDefinition self) {
        self.process {
            repozytorium().zapisz(it.in.body.entries)
        }
    }

    static ProcessorDefinition rozdzielElementy(ProcessorDefinition self, entriesClosure) {
        self.transform({ exchange, type ->
            def body = exchange.in.body
            ['entries': entriesClosure(body)]
        } as Expression)
    }
 ```   
> Wykorzystanie metod w konfiguracji routingu
 ```groovy
    from(STORE_DOCUMENT_ENTRIES)
            .rozdzielElementy { it.req.documentEntries }
            .konwertujDocumentEntries()
            .to(STORE)

    from(STORE)
            .zapisz()
 ```

* **Wykorzystanie rozszerzeń języka Groovy** – takich jak specjalna obsługa przetwarzania XML czy wzbogacone łańcuchy znaków GString. 

> Przykład operacji na XML:
 ```groovy
    ...
       from('direct:test1')
          .unmarshal().gnode() 
          // message body is now of type groovy.util.Node
          ... // some logic
          .marshal().gnode()
    ...
 ```

##### Testy routingu za pomocą Groovy oraz Spock Testing Framework
Kolejnym przykładem wykorzystywania języka Groovy w SGZ są testy jednostkowe routingu wraz z wykorzystaniem frameworku do testowania o nazwie Spock. 
Framework Spock można nazwać zbiorem funkcjonalności popularnych i sprawdzonych bibliotek wykorzystywanych do pisania testów w Javie (takich jak JUnit, jMock, Mockito) 
wzbogaconym o zalety języka Groovy.

Wśród korzyści zastosowania języka Groovy wraz ze Spock wymienić można:
 * **Spójna i czytelna konwencja nazewnicza** – w przeciwieństwie do Javy metody w Groovy mogą mieć postać stringów. 
Umożliwia to dosłownie opisanie testu zdaniem. Umożliwia również parametryzowanie nazw testu, co pozwala lepiej opisywać sprawdzane przypadki testowe. 

> Przykład - zmienna *#opis* zostanie zamieniona na "włączonym" oraz "wyłączonym" 

 ```groovy
    def "Wywołaj metodę loguj z #opis debugiem i sprawdź czy wywolano logowanie"() {
        // ...
    }
 ```

 * **Zastosowanie rozszerzonej konwencji GivenWhenThen** – konwencja rozdzielająca fragmenty kodu testu, znana dotychczas pod postacią komentarzy, 
została w Spocku rozbudowana. Bloki (given, when, then oraz inne) są w Spocku domyślnym sposobem na dzielenie faz naszego testu za pomocą znanych z Javy etykiet (labels).
 Przykładowy test może składać się m.in. z następujących bloków:
    - ***given*** (zamiennie setup) - w bloku tym ustawiamy wartości do naszego testu
    - ***when*** - definiujemy nasze akcje, których wyniki będziemy sprawdzać w teście
    - ***then*** - jest blokiem, w którym sprawdzamy wynik z bloku when. Składać się on może nie tylko z asercji ale także z z samych warunków logicznych jak na przykładzie.
    - ***where*** – jest blokiem w którym możemy ustawiać parametry naszego testu

> Przykład: 
 ```groovy
    def "Should get an index out of bounds when removing a non-existent item"() {
        given:
            def list = [1, 2, 3, 4]
        when:
            list.remove(20)
        then:
            thrown(IndexOutOfBoundsException)
            list.size() == 4
    }
 ```

 * **Mechanizmy parametryzowania testów** – W Spocku testy parametryzować można na dwa sposoby za pomocą „data tables” oraz „data pipes”
   - ***Data tables*** - w tym przypadku przygotowuje się tabelę, w której nagłówki to pola, które chcemy uzupełnić, natomiast pod nagłówkami umieszczamy wartości tych pól. Pola rozdzielamy znakiem pipe „|” lub „||”
   
> Przykład parametryzowania testów - *Data tables*:
```groovy
    def "Numbers to the power of two"() {
      expect:
          Math.pow(a, b) == c 
      where:
          a | b | c
          1 | 2 | 1
          2 | 2 | 4
          3 | 2 | 9
    }
```
   
   - ***Data pipes*** - analogiczny w działaniu do poprzedniego mechanizmu wykorzystuje zbiór danych, wskazany przez operator przesunięcia w lewo (<<), tworząc dostawcę danych (data provider). Dostawca danych przechowuje wszystkie wartości w zmiennej, po jednej na iterację. 

> Przykład parametryzowania testów - *Data pipes*:
```groovy
    def 'Get the bigger number'() {
 
        expect: 'Should return the bigger number'
            Math.max(number1, number2) == expectedBiggerNumber
        where:
            number1 << [1, 2]
            number2 << [0, 3]
            expectedBiggerNumber << [1, 3]
    }
```

 * **Wbudowany mechanizm mockowania** – Spock ma własny wbudowany mechanizm mockowania, który także oparty jest o rozwiązaniach języka Groovy. 

> Przykład
```groovy
    def setup() {
        itemProvider = Stub(ItemProvider)
        eventPublisher = Mock(EventPublisher)
        loggingEventPublisher = Spy(LoggingEventPublisher)
        itemService = ItemService(itemProvider, eventPublisher)
    }
```

#### Jak wykorzystujemy Groovy w SoapUI (element nowego podejścia do testów) 
W narzędziu SoapUI możemy korzystać ze skryptów napisanych w języku Groovy. Samo narzędzie udostępnia z miejsca szereg metod ułatwiających operowanie na danych. 
Warto wyróżnić tutaj klasy takie jak GroovyUtils czy XmlHolder. 

Z ich użyciem skryptów w SoapUI możemy na przykład:
 - Napisać logikę wykonującą się przed i po lub w trakcie wykonania się kolejnych kroków scenariusza 
 
> Przykład dynamicznego wczytywania danych z mapy JSON, które bedą następnie dodane do żądania

```groovy
    def setAdditionalDynamicData = {
        def additionalDynamicData = context.expand('${#TestCase#AdditionalDynamicData}')
        if (additionalDynamicData) {
            log.info "Additional dynamic data: $additionalDynamicData"
            def additionalDynamicDataMap = new groovy.json.JsonSlurper().parseText(additionalDynamicData)
            additionalDynamicDataMap.each {
                key, value ->
                    testRunner.testCase.setPropertyValue(key, value)
            }
        }
    }
    ... //some logic
    setAdditionalDynamicData.call()
```
 
 - Stworzyć zaawansowane asercje dostosowane do naszego przypadku testowego. 
 
 > Przykład wczytania zewnętrznego skryptu w asercji
 ```groovy
     // Location of script file relative to SOAPUI project file.
     def scriptPath = new File( context.expand( '${projectDir}')).getParentFile().getAbsolutePath() + "/" + "Skrypty"
     
     // Create Groovy Script Engine to run the script.
     def gse = new GroovyScriptEngine(scriptPath)
     
     // Run the Groovy Script file
     gse.run("comparisonSoapUi.groovy", binding)
 ```
 - W wygodny sposób operować na przesyłanych żądaniach (strukturach XML) oraz samych danych 
 - W szerokim stopniu manipulować ustawieniami i zmiennymi programu (znajdującymi się zarówno na poziomie przypadku testowego jak i całego projektu)  

#### Przykłady ciekawych konstrukcji składniowych w języku Groovy

- Linie kodu nie muszą być zakończone średnikiem, nawiasy przy wywołaniu funkcji są zbędne, a słowo kluczowe return jest opcjonalne
    ```groovy
    def calculateSum(a, b) {
        a + b
    }
    def sum = calculateSum 3, 2
    assert 5 == sum​
    ```
 
 - GString oraz interpolacja tekstu - łatwy sposób na tworzenie teksu w którym zawarte są zmienne
     ```groovy
     def groovyToString(String name, Integer age) {
         "User $name is $age years old."
     }
        
     try{
        throw new Exception()
     }catch(Exception e){
        println "Error during operation. Cause: ${e}"
     }
     ```
 
-  Łatwe tworzenie list i map
    ```groovy
     def personMap = [name: 'Ala', animal: 'kot', age: 15]
     def samleList = ["item1", "item2", "item3", "item4"]
     
     assert person.getClass()​​​​​​​​​​​ == ​LinkedHashMap.class
     assert person.get('name') == 'Ala'
     assert person.animal == 'kot'
     ```

- Domknięcia (closures) - zachowują się jak metody, możemy przypisać je do zmiennej i wykorzystywać
    ```groovy
     def timesTwo = { it -> it*2 }
     
     assert timesTwo(4) == 8
     assert timesTwo('abc') == 'abcabc'​​​​​​​​​​​​
    ```

- Elvis operator
  ```groovy
  //Java way
    String ternaryOutput = (sampleText != null) ? sampleText : "Bye Java!";
  //Groovy way
    def elvisOutput = sampleText ?: 'Hello Groovy!'
  ```
- Pętle w Groovy
  ```groovy
  0.upto(4) {print "$it,"}        // Will print 0,1,2,3,4,
  5.times{print "$it,"}           // equivalent output
  (0..4).each {print "$it,"}      // equivalent output
  
  for (int x = 0; x <= 5; x=x+2) {  // In Java
        System.out.println(x);
  }
  0.step(7,2){println "$it"}        // In Groovy
  ```
  
- Wszystkie składowe obiektu są domyślnie publiczne, a Groovy sam generuje settery, gettery i konstruktory
  ```groovy 
  class Person {
    int id
    String name, surname
    //let Groovy compiler generate all setters and getters!
  }
  
  //named constructor parameters, only for default constructor
  new Person(id: 1, name: 'John', surname: 'Doe')
  ```
  
- Oraz wiele wiele innych  
  http://groovy-lang.org/documentation.html
  
<a name="iti"></a>
## Omówienie przetwarzania komunikatu na przykładzie transakcji ITI-42 i ITI-18
#### Konfiguracja routingu
Konfiguracje routingu polega na skonfigurowaniu w apache camel jednego z dostepnych w biblitece IPF interfejsu transakcji
oraz wykorzystaniu camelowego DSL (domain specific language) czyli języka programowania dedykowanego do rozwiązywania określonej
dziedziny problemów a w tym przypadku ułatwienia w konfiguracji i wsparcia wykorzystywania wzorców EIP.
Podczas implementacji język dedykowany dostępny w camelu został rozszerzony o dodatkowe metody mające na celu ułatwienie pracy oraz implementacji,
opis jednej z nich zostanie przedstawiony w dalszej części warsztatu. 

##### Przykład wraz z opisem implementacji routingu transakcji ITI-18

![omówienie_przetwarzania_komunikatu_opis_iti18](obrazki/omowienie_przetwarzania_komunikatu_opis_iti18.png)

1. Udostępnienie routingu gotowego komponentu iti18 na zewnątrz i ustalenie endpointu pod który możemy uderzać z zewnątrz (mapuje się bezpośrednio na ***http://host:port/services/xds-iti18***)
2. Jedna z metod w zaimplementowanym rozszerzonym DSL`u mająca na celu logowanie ciała przesłanego w requescie po uwczesnej konwersji go na klase *QueryRegistry*
3. Podstawowa metoda udostępniana przez apache camel mająca na celu działanie na przesłanej wiadomości (w tym przypadku uruchamiany jest dostępny z pudełka walidator requestu iti18)
4. Tutaj widzimy transformacje przesłanej wiadomości w formacie ebXML do formatu XDS a następnie wrzucenie do mapy pod kluczem 'req' oraz stworzenie juz obiektu odpowiedzi i wrzucenie go pod klucz *'resp'*
5. Blok warunkowy, w przypadku przesłania zapytania typu **FindDocuments** routing przekierowywany jest do odpowiedzialnej za wyszukiwanie dokumentu częsci implementacji, w przypadku nieobsługiwanego typu zwracany jest błąd
6. Kolejny blok warunkowy odpowiedzialny za zwracanie referencji do obiektu gdy zapytanie będzie tego wymagało
7. Transformacja mająca na celu wyciągniecie z mapy odpowiedzi znajdującej się pod kluczem *'resp'*
8. Lokalny route zajmujący się wyszukiwaniem dokumentu dla typu **FindDocuments**
9. Lokalny route zajmujący się konwersją obiektów na referencję
10. Konwersja zapytania w formacie **XDS** na obiekt transportowy **DTO** ułatwiający wyszukiwanie (motoda dodana w rozszerzonym DSL)
11. Wyszukanie dokumentu w repozytorium (motoda dodana w rozszerzonym DSL)
12. Konwertuje dokumenty na ich referencje

##### Przykład wraz z opisem jednej z metod rozszerzonego DSL`a

![omówienie_przetwarzania_komunikatu_opis_metody_rozszerzenia.PNG](obrazki/omowienie_przetwarzania_komunikatu_opis_metody_rozszerzenia.png)

1. Deklaracja metody *konwertujAtrybutyWyszukiwaniaFindDocuments*
2. Wywołanie interfejsu procesora dzięki któremu możemy modyfikować przesyłaną wiadomość ([apache_camel_processor](https://camel.apache.org/processor.html))
3. Blok kodu odpowiadający za pobranie z obrabianej wiadomośći potrzebnych informacji do stworzenia obiektu transportowego **DTO**
4. Blok kodu tworzący obiekt transportowy **DTO** wraz z potrzebnymi do wyszukiwania dokumentów parametrami wyszukiwania
5. Dodanie do ciała wiadomości obiektu stworzonego w kroku 4

##### Przydatne informacje/odnośniki

|Nazwa|Opis|Odnośnik|
|-----|-----|-----|
|ITI-42|Implementacja routingu ITI-42 (przygotowana także na ITI-57)|*pl.gov.csioz.p1.sgz.edm.routing.Iti4257RouteBuilder*|
|ITI-18|Implementacja routingu ITI-18 |*pl.gov.csioz.p1.sgz.edm.routing.Iti18RouteBuilder*|
|DSL|Klasa rozszerzająca DSL dostępny w apache camel o dodatkowe metody|*pl.gov.csioz.p1.sgz.edm.routing.RoutingRozszerzenia*|

#### Konwersja ebXML (MT) - XDS - ebXML (ML)

**ebXML**- Standard ebXML Registry Information Model (ebRIM) do budowy Rejestru XDS.b

**XDS**- Profil IHE XDS.b użytu do rejestrowania i wymiany informacji o dokumentach medycznych

**MT**- model transportowy

**ML**- model logiczny

Aby przejść od odebrania requestu do zapisu/odczytu danych w bazie musimy przejść pewną konwersję pomiędzy różnymi standardami/profilami oraz poszczególnymi warstawmi aplikacji.

W poniższej tabeli zostały przedstawione poszczególne kroki podczas wywołania transakcji ITI-18 oraz odbywające się konwersje od przyjęcia requestu wyszukania do zwrócenia oczekiwanego obiektu.

|Lp.|Typ przed konwersją|Obiekt przed konwersją|Typ po konwersji|Obiekt po konwersji|Opis operacji/działania|Pakiet|Klasa|Metoda|Linia kodu|
|---|:---:|:---:|:---:|:---:|---|---|---|---|---|
|1|ebXML(MT)|AdhocQueryRequest| - | - | Przyjście requestu wywołane przesłaniem przez system zewnętrzny (początek przetwarzania) |  *pl.gov.csioz.p1.sgz.edm.routing* | Iti18RouteBuilder | configure |33|
|2|ebXML(MT)|AdhocQueryRequest| XDS | QueryRegistry | Konwersja requestu na typ XSD |  *pl.gov.csioz.p1.sgz.edm.routing* | Iti18RouteBuilder | configure |36|
|3|XSD|QueryRegistry| DTO | FindDocumentsDto | Utworzenie DTO z obiektu XSD w celu ułatwienia budowania zapytania |  *pl.gov.csioz.p1.sgz.edm.routing* | Iti18RouteBuilder | configure |49|
|4|DTO|FindDocumentsDto| ebXML(ML) | EbXmlExtrinsicObjectJpa | Wyszukanie obiektu z bazy danych oraz utworzenie obiektu typu ebXML(ML) |  *pl.gov.csioz.p1.sgz.edm.routing* | Iti18RouteBuilder | configure |50|
|5|ebXML(ML)|EbXmlExtrinsicObjectJpa| XDS | QueryResponse | Konwersja obiektu ebXml modelu logicznego na XDS |  *pl.gov.csioz.p1.sgz.edm.routing* | RoutingRozszerzenia | wyszukajDokument |52-53|


#### Fasada repozytorium

W projekcie posiadamy klase reprezentującą fasade repozytorium (*pl.gov.csioz.p1.sgz.edm.model.common.RepozytoriumFasada*) jest to klasa która ma na celu
stworzenie sposobnośći rozszerzenia implementacji w zależnośći o technologię utrwalania danych. Metody w klasie fasady powinny przyjmowac i zwracać obiekty będące
instancjami typów prostych lub interfejsów IPF i taką konwencje należy utrzymywać. W tej chwili zdecydowaliśmy się na utrwalanie danych w relacyjnej bazie danych
lecz gdy się zdecydujemy na zmiane z realacyjnej na nierelacyjną baze danych (np. po testach wydajnościowych) to nic nie stoi na przeszkodzie. 

Model realcyjnej bazy danych jest dostępny w sub-module edm/model/jpa modelu edm-model i też tam znajdziemy konkretną implementacje naszej fasady (*pl.gov.csioz.p1.sgz.edm.model.jpa.fasada.RepozytoriumRelacyjnejBazyDanych*)
na potrzeby zapisu(ITI-42) oraz odczytu(ITI-18) danych. Także tutaj znajdziemy wszystkie pliki encji/konfiguracji/konwersji/wrapperów EbXml czy też repozytoriów.

##### Przydatne informacje/odnośniki

|Nazwa|Opis|Odnośnik|
|-----|-----|-----|
|RepozytoriumFasada|Abstrakcyjna klasa fasady repozytorium|*pl.gov.csioz.p1.sgz.edm.model.common.RepozytoriumFasada*|
|RepozytoriumRelacyjnejBazyDanych|Implementacja fasady oparta na modelu relacyjnym  |*pl.gov.csioz.p1.sgz.edm.model.jpa.fasada.RepozytoriumRelacyjnejBazyDanych*|

#### Podsumowanie

W tym rozdziale omówiliśmy przetwarzanie komuniaktu na przykładzie transakcji ITI-42 i ITI-18, przede wszystkim skupiono się na transakcji ITI-18 ponieważ w niej
zachodzi więcej konwersji, ITI-42 bazuje na tych samych pryncypiach co przestawione na podstawie przykładów/opisów ITI-18.

<a name="testy"></a>
## Omówienie nowego podejścia do testów SoapUI oraz omówienie narzędzi zewnętrznych wspomagających testy SGZ EDM 
Bla bla
