## Zachętka

Cześć. Chciałem opowiedzieć historię, która przydarzyła mi się 2 lata temu. Miałem zająć się projektem, który nasz klient rozwijał u siebie od jakiegoś czasu. Była to aplikacja AngularJS z backendem w WebAPI. Wszystko brzmiało dość fajnie. Architektura nie była zbyt skomplikowana ponieważ aplikacja robiła proste rządania do API po HTTP. Pierwsza rzecz, która mnie zainteresowała to w jaki sposób to API jest zabezpieczone. Nigdy nie zajmowałem się podobnymi sprawami i uznałem, że to dobra okazja do nauki. Właściwie nie wiedziałem czego się spodziewać ale to co zastałem nie wydało mi się prawidłowe. Bezpieczeństwo API opierało się o GUID zapisany w bazie danych. Ta sama wartość była zapisana w kodzie aplikacji javascriptowej i dołączana do każdego requesta w nagłówku Authorization.

## Problem - Czy to jest prawidłowe rozwiązanie?

Po chwili zastanowienia od razu widać, że nie potrzeba wiele aby złamać takie zabezpieczenie. Cały kod aplikacji jest dostępny w przeglądarce. Mimo, że programiści zapewnili minifikację to przy niewielkim wysiłku można było wydobyć tą wartość z kodu js. 

## Problem - W jaki sposób to poprawić? 

Najprościej byłoby stworzyć bazę danych przechowującą nazwę użytkownika i jego hasło. Ale w jaki sposób decydowałbym czy dany request może być przetworzony? Musiałbym przesyłać login i hasło z każdym rządaniem? To na pewno nie jest dobrze żeby tak wrażliwe dane latały sobie po sieci. To może baza danych powinna być po stronie aplikacji klienckiej? Ale w przypadku kiedy więcej aplikacji chciałoby korzystać z mojego api to każda musiałaby mieć osobną bazę userów. Miałem więc 2 problemy:
 - jak zabezpieczyć API,
 - jak i gdzie przechowywać dane użytkowników.

Po krótkich konsultacjach z profesorem Google dowiedziałem się, że takie giganty jak Google czy Facebook korzystają ze standardu Oauth2. To pojęcie obiło mi się o uszy kilka razy więc postanowiłem dowiedzieć się o nim więcej.

## Problem - Co trzeba wiedzieć o Oauth2, żeby z niego korzystać?

Oauth to standard służący do autoryzacji. Kiedy czytałem o zabezpieczeniach często pojawiały się dwa określenia: authorization i authentication. Wydawało mi się, że to jest to samo. Okazuje się, że nie. Autoryzacja to proces, który pozwala odpowiedzieć na pytanie "co ktoś może zrobić?". Uwierzytelnianie to proces, który pozwala odpowiedzieć na pytanie "kim ktoś jest?". Oauth2 zajmuje się tylko i wyłącznie tą pierwszą częścią. Świetnie! To rozwiązanie mojego pierwszego problemu! Ale zaraz zaraz Oauth wymaga wyboru czegoś co jest nazwane w specyfikacji FLOW. 

## Problem - Co to jest FLow w Outh.

Flow to scenariusz interakcji pomiędzy aplikacją kliencką a API. Żeby wybrać prawidłowe flow trzeba wiedzieć jakie elementy współpracują ze sobą w takim scenariuszu. 

1. Pierwszy z nich to Użytkownik. Jest właścicielem zasobów i komunikuje się z aplikacją kliencką przez przeglądarkę.
2. Aplikacja kliencka - aplikacja, która wykonuje rządania w imieniu użytkownika i prezentuje wyniki ich przetwarzania.
3. Serwer z zasobami - nasze API. Posiada ono informacje, których właścicielem jest użytkownik.
4. Serwer autoryzacyjny - posiada informacje o użytkownikach i wydaje poświadczenia stanowiące podstawę autoryzacji.

Podstawą działania tego mechanizmu są wspomnniane poświadczenia. Czym jest poświadczenie? Większość z Was na pewno słyszała pojęcie token. Token jest właśnie poświadczeniem czyli dowodem autoryzacji zakończonej powodzeniem. Więc jeśli request zawiera taki token to znaczy, że jego właściciel został zautoryzowany do wykonania danej operacji. Oauth wykorzystuje coś co nazywa się Bearer Token. Bearer token to typ token'a, który autoryzuje posiadacza tokena. To znaczy, że ktokolwiek wszedł w posiadanie tokena i go przedstawi, zostanie zautoryzowany.

Od czego zależy flow? Głównie od typu aplikacji klienckiej. Mamy 2 typy aplikacji klienckich:
 - aplikacje poufne - takie, które działają na serwerze, a do przeglądarki wysyłają jedynie wygenerowany widok, np MVC. Takie aplikacje potrafią ukrywać informacje przed użytkownikiem ponieważ ich logika wykonuje się na innej maszynie niż komputer, z którego korzysta użytkownik.
 - aplikacje publiczne - aplikacje, które nie są w stanie niczego ukryć przed użytkownikiem. To cecha aplikacji javascript lub natywnych aplikacji mobilnych. W przypadku js jest tak poneiważ całość kodu aplikacji jest ściągana do przeglądarki i się w niej wykonuje. 

 ## Problem - Czy mogę wykorzystać Resource Owner Flow?

 Wydaje się być ok ponieważ w tym flow moja aplikacja pobiera nazwę użytkownika i hasło, przesyła je i w zamian dostaje token.
 Niestety nie mogę go wykorzystać. Ten flow jest dedykowany dla aplikacji poufnych. Poza tym dużym atutem Oauth2 jest wykorzystanie serwera autoryzacyjnego w taki sposób, że aplikacja kliencka nigdy nie widzi nazwy użytkownika i hasła. Ten flow został dodany z powodu kompatybilności ze starszymi rozwiązaniami i nie jest zalecany.

 ## Problem - To może Client Credentials?

 Też dość prosty flow, w którym aplikacja kliencka podaje swoje clientId i secret, a w zamian dostaje token. To rozwiązanie jest przeznaczone dla aplikacji, które nie mają użytkowników. Tutaj clientId i secret jest loginem i hasłem całej aplikacji. To powoduje, że clientId i secret nie mogą wyciekać w żadnym momencie do użytkownika aplikacji. Eliminuje to kleintów publicznych, czyli niestety takich jak mój. 

 ## Problem - Może AuthCode?

 Auth code wydaje się mieć wszystko czego potrzebuję. Użytkownik jest przekierowywany na stronę logowania serwera autoryzacyjnego gdzie podaje swój login i hasło. W ten sposób aplikacja kliencka nigdy go nie poznaje. Po zalogowaniu do serwera autoryzacyjnego zostaje przekierowany spowrotem do aplikacji klienckiej z kodem autoryzacyjnym. Kod autoryzacyjny to nie token. Aplikacja kliencka dołącza clientId i secret do kodu autoryzacyjnego i wymienia je z serwerem autoryzacyjnym na token. Ta operacja powinna dziać się bez interakcji z przeglądarką. ClientId i secret muszą być podobnie jak poprzednio ukryte przed użytkownikiem. Nasza aplikacja nie jest w stanie tego zapewnić więc ten flow również nie jest dla nas.

 ## Problem - Czy Implicit Flow jest ok?

 Tak! Ten flow jest prostszy niż poprzedni ponieważ omija konieczność przekazania kodu autoryzacyjnego. Ponieważ tutaj nie jesteśmy w stanie ukryć niczego przed użytkownikiem, Serwer autoryzacyjny po zalogowaniu zwraca token bezpośrednio do przeglądarki. Takie rozwiązanie jest konieczne ale mniej bezpieczne. To powoduje, że token powinen mieć krótszy czas przydatności.

 ## Problem - Przeniosłem magazyn użytkowników do serwera autoryzacyjnego. W jaki sposób mam uzyskać o nich informacje?

 W ten sposób dotarliśmy do konieczności uwierzytelniania. Ten problem rozwiązuje standard rozwijający protokół Oauth2 czyli OpenId Connect. Pozwala uzyskać dodatkowy token zawierający informacje o użytkowniku. Mamy teraz do czynienia z dwoma tokenami:

 - access_token - służący do uzyskania dostępu do API. Ten token jest przekazywany przez naszą aplikację do requestów lecących do api.
 - id_token - token stanowiący dowód uwierzytelnienia użytkownika. Ten token jest przeznaczony do przekazania informacji o użytkowniku do naszego programu.

 ## Problem - Czemu nie upakować wszystkiego do jednego tokena? 

 Można tak zrobić i niektórzy stosują takie rozwiązania. Nie jest to jednak zgodne ze standardem, niesie ryzyko wyciekania danych w przypadku interakcji z dużą ilośćią API.

 ## Problem - To wszystko jest bardzo skomplikowane. Ile czasu zajmie zaprogramowanie tak skomplikowanych protokołów?

 Ten problem rozwiązuje IdentityServer. To modularny serwer autoryzacyjny. W wersji 4 oparty o .NET Core. Jest dostarczany jako paczka Nuget i wpina się do aplikacji ASP.NET Core jako jeden z middleware'ów. Pozwala w elastyczny sposób podpiąć magazyn użytkowników (np ASP.NET Identity lub stworzyć swój własny), łatwo definiować aplikacje klienckie i różne flow dla każdej z nich. Autorzy przygotowali również paczki umożliwiające łatwe wykorzystanie w API oraz aplikacjach klienckich tworzonych w różnych technologiach, np oidc-client dla javascript.