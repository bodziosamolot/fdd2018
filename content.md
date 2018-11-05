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