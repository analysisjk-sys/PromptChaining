# Scenariusze BDD — zmiana PIN-u do aplikacji

## Zakres i warunki wykonania

Wersja końcowa, 2026-11-11. Źródło wymagań: [Kryteria_Akceptacji.md](Kryteria_Akceptacji.md). Obowiązuje PIN 6-cyfrowy i jeden formularz z trzema polami. Każdy scenariusz oraz każdy wariant danych uruchamia się niezależnie, z odtworzonym stanem początkowym.

Domyślnie: konto A jest zalogowane, sesja aktywna, brak blokady, licznik 0, aktualny PIN `583026`, nowy PIN i powtórzenie `729461`. Inne dane wskazuje Given. Sukcesy uwierzytelnienia i odrzucenia PIN-ów sprawdza się w niezależnych próbach, bez wpływu liczników logowania. Obserwacja licznika, momentu zatwierdzenia, unieważnienia sesji i trwałego stanu korzysta z instrumentacji środowiska testowego, bez ujawniania PIN-ów w produkcyjnych logach.

Wielowariantowe scenariusze są szablonami: każdy wymieniony wariant to oddzielny test. Kontrolowane awarie, zegar, synchronizacja żądań, proxy i audyt magazynów są elementami środowiska testowego. Automatyczne mapowanie KA→BDD oznacza pokrycie specyfikacją, nie dowód wykonania testów ani kompletnego bezpieczeństwa aplikacji.

## BDD-01: Otwarcie formularza

**Kryteria:** KA-01, KA-02.

- **Given:** Użytkownik jest na ekranie głównym.
- **When:** Wybiera Profil → Ustawienia → Bezpieczeństwo → Zmień PIN.
- **Then:** Widoczne są trzy wymagane pola o etykietach z KA-02 i przycisk „Zmień PIN”; wpisane wartości każdego pola są maskowane.

## BDD-02: Brak autoryzacji i cudze konto

**Kryteria:** KA-03.

- **Given:** Przygotowano niezależne warianty: brak tokenu, nieważny token, token konta A i wskazanie konta B; znane są PIN-y obu kont.
- **When:** Klient bezpośrednio wywołuje endpoint zmiany z poprawnymi składniowo danymi.
- **Then:** Każde żądanie jest odrzucone; PIN-y obu kont pozostają niezmienione; żadna chroniona informacja o koncie B nie jest zwrócona.

## BDD-03: Wyjście i anulowanie przed wysłaniem

**Kryteria:** KA-04, KA-22, KA-26.

- **Given:** W formularzu wpisano trzy PIN-y, żądania jeszcze nie wysłano.
- **When:** Użytkownik w niezależnych wykonaniach anuluje lub opuszcza formularz, następnie otwiera go ponownie.
- **Then:** Nie wysłano żądania zmiany, obowiązuje stary PIN, wszystkie pola są puste i robocze wartości usunięte.

## BDD-04: Puste pola

**Kryteria:** KA-05, KA-12, KA-13.

- **Given:** Pozostałe pola są poprawne; w każdym wykonaniu puste jest jedno z trzech pól, a dodatkowo wykonywany jest wariant wszystkich pustych pól.
- **When:** Użytkownik zatwierdza formularz.
- **Then:** Każde puste pole ma komunikat „Pole jest wymagane.”; brak zapisu i weryfikacji aktualnego PIN-u; licznik nie zmienia się.

## BDD-05: Długość i niedozwolone znaki

**Kryteria:** KA-06, KA-12, KA-13.

- **Given:** Nowy PIN i jego powtórzenie mają wartość z tabeli BDD-05; pozostałe dane są poprawne.
- **When:** W każdym niezależnym wykonaniu użytkownik zatwierdza jedną wartość.
- **Then:** Zmiana jest odrzucona z komunikatem „PIN musi składać się z 6 cyfr od 0 do 9.”; nie następuje obcięcie ani normalizacja; PIN i licznik pozostają bez zmian.

## BDD-06: Akceptowane PIN-y i zera wiodące

**Kryteria:** KA-06, KA-09, KA-18.

- **Given:** Nowy PIN i powtórzenie mają jedną z wartości: 072946, 729461, 135790, 789012; pozostałe dane są poprawne.
- **When:** W każdym niezależnym wykonaniu użytkownik zatwierdza formularz i serwer zapisuje zmianę.
- **Then:** Występuje potwierdzony sukces; uwierzytelnienie dokładną nową wartością jest poprawne, poprzednią niepoprawne. Dla 072946 zero wiodące zostaje zachowane.

## BDD-07: PIN identyczny z aktualnym

**Kryteria:** KA-07, KA-12.

- **Given:** Oba nowe pola zawierają 583026.
- **When:** Użytkownik zatwierdza formularz.
- **Then:** Przy nowym PIN-ie widoczny jest komunikat „Nowy PIN musi różnić się od aktualnego.”; nie ma zapisu.

## BDD-08: Wszystkie identyczne cyfry

**Kryteria:** KA-08, KA-12.

- **Given:** Oba nowe pola zawierają kolejno w niezależnych wykonaniach: 000000, 111111, 222222, 333333, 444444, 555555, 666666, 777777, 888888, 999999.
- **When:** Użytkownik zatwierdza formularz.
- **Then:** Każda wartość jest odrzucona z komunikatem „PIN nie może składać się z identycznych ani kolejnych cyfr.”; PIN i licznik pozostają bez zmian.

## BDD-09: Wszystkie zabronione sekwencje

**Kryteria:** KA-09, KA-12.

- **Given:** Oba nowe pola zawierają kolejno w niezależnych wykonaniach: 012345, 123456, 234567, 345678, 456789, 987654, 876543, 765432, 654321, 543210.
- **When:** Użytkownik zatwierdza formularz.
- **Then:** Każda wartość jest odrzucona z komunikatem „PIN nie może składać się z identycznych ani kolejnych cyfr.”; PIN i licznik pozostają bez zmian.

## BDD-10: Niezgodne powtórzenie

**Kryteria:** KA-10, KA-12, KA-13.

- **Given:** Nowy PIN to 729461, powtórzenie to 729462; licznik wynosi 2.
- **When:** Użytkownik zatwierdza formularz.
- **Then:** Przy powtórzeniu widnieje „Podane nowe PIN-y nie są identyczne.”; brak zapisu i weryfikacji aktualnego PIN-u; licznik nadal wynosi 2.

## BDD-11: Niepoprawny aktualny PIN

**Kryteria:** KA-11, KA-13.

- **Given:** Aktualny PIN wpisany w formularzu to 583027, pozostałe dane są poprawne; licznik wynosi 0.
- **When:** Serwer kończy weryfikację.
- **Then:** Przy aktualnym PIN-ie widnieje „Aktualny PIN jest niepoprawny.”; licznik wynosi 1; stary PIN nadal obowiązuje.

## BDD-12: Walidacja serwerowa i kilka błędów

**Kryteria:** KA-05, KA-06, KA-07, KA-08, KA-09, KA-10, KA-11, KA-12, KA-13.

- **Given:** Klient omija UI. Niezależne żądania obejmują wszystkie dane niepoprawne z BDD-04–BDD-11 oraz wariant: aktualny PIN 583027, nowy PIN 12a, powtórzenie 999999; licznik początkowy 0.
- **When:** Żądania trafiają bezpośrednio do serwera.
- **Then:** Każde zostaje odrzucone bez zapisu. W wariancie kilku błędów odpowiedź wskazuje błędny format nowego PIN-u i niezgodne powtórzenie, nie weryfikuje aktualnego PIN-u; licznik pozostaje 0. Dla izolowanego błędnego aktualnego PIN-u licznik rośnie o 1. UI mapuje błędy na komunikaty i pola KA-05–KA-11.

## BDD-13: Granica czwartej i piątej próby

**Kryteria:** KA-11, KA-13, KA-14, KA-17.

- **Given:** Licznik wynosi 3, sesja jest aktywna.
- **When:** Dwie różne operacje kolejno kończą weryfikację błędnego aktualnego PIN-u; druga w chwili serwera T.
- **Then:** Po pierwszej licznik wynosi 4, brak blokady i komunikat KA-11. Po drugiej licznik wynosi 5, blokada kończy się T+900 s; widoczny jest komunikat KA-17 i czas zakończenia.

## BDD-14: Licznik wspólny i trwały

**Kryteria:** KA-13, KA-14.

- **Given:** Na urządzeniu A naliczono 4 błędne weryfikacje.
- **When:** Użytkownik ponownie loguje się, reinstaluje klienta i z urządzenia B wykonuje piątą błędną weryfikację.
- **Then:** Licznik nie został wyzerowany; piąta próba uruchamia blokadę widoczną również na A.

## BDD-15: Blokada nie jest przedłużana

**Kryteria:** KA-15, KA-17.

- **Given:** Blokada kończy się o T, licznik wynosi 5.
- **When:** Przed T wysyłane są nowe żądania z poprawnym i błędnym aktualnym PIN-em; zegar urządzenia zostaje przesunięty o godzinę.
- **Then:** Wszystkie żądania są odrzucone komunikatem KA-17; koniec nadal T według serwera, licznik nadal 5; PIN bez zmian.

## BDD-16: Granica końca blokady

**Kryteria:** KA-15, KA-16.

- **Given:** Kontrolowany zegar serwera, koniec blokady T; każde wykonanie ma niezależny stan.
- **When:** Poprawne żądanie jest rozpatrywane kolejno w T−1 ms, T i T+1 ms.
- **Then:** W T−1 ms jest odrzucone przez blokadę. W T oraz T+1 ms blokada już nie obowiązuje, licznik jest wyzerowany przed próbą i poprawna zmiana kończy się sukcesem.

## BDD-17: Zerowanie i zachowanie licznika

**Kryteria:** KA-13, KA-16.

- **Given:** Licznik wynosi 3; w niezależnych wykonaniach kontroluje się wynik zapisu.
- **When:** Poprawna weryfikacja aktualnego PIN-u kończy się: a) udaną zmianą, b) potwierdzonym błędem technicznym przed zapisem.
- **Then:** W a) licznik wynosi 0; w b) pozostaje 3. Błąd techniczny nie nalicza nowej próby.

## BDD-18: Atomowe naliczanie prób

**Kryteria:** KA-13, KA-14, KA-15.

- **Given:** Licznik wynosi 3; dwa różne żądania błędnej weryfikacji są synchronizowane na serwerze.
- **When:** Obie weryfikacje dochodzą do atomowej aktualizacji licznika.
- **Then:** Nie ma utraconego zwiększenia: licznik osiąga 5 i uruchamia jedną blokadę; dalsze próby są odrzucane bez przedłużenia.

## BDD-19: Sukces i brak przedwczesnego potwierdzenia

**Kryteria:** KA-18, KA-24.

- **Given:** Poprawny formularz; zapis i odpowiedź są kontrolowanie wstrzymane.
- **When:** Użytkownik zatwierdza, następnie serwer zatwierdza zapis i dostarcza wynik.
- **Then:** Przed potwierdzeniem aplikacja nie pokazuje sukcesu. Po wyniku pokazuje dokładnie „PIN został zmieniony”; nowy PIN działa, poprzedni nie; widoczny przycisk „Przejdź do logowania” otwiera logowanie.

## BDD-20: Awaria przed i po zatwierdzeniu

**Kryteria:** KA-18, KA-19, KA-23.

- **Given:** Przygotowano awarie: a) przed zatwierdzeniem transakcji, b) po zatwierdzeniu, przed odpowiedzią; zapis obejmuje unieważnienie sesji.
- **When:** W każdym wykonaniu następuje awaria i restart serwera.
- **Then:** W a) działa tylko stary PIN; w b) działa tylko nowy PIN i stare sesje są nieważne. Brak stanu dwóch aktywnych PIN-ów albo żadnego. Odczyt wyniku po autoryzacji jest zgodny z trwałym stanem.

## BDD-21: Duplikat udanej operacji

**Kryteria:** KA-20, KA-24, KA-31.

- **Given:** Operacja X zakończyła się sukcesem; stara sesja jest unieważniona.
- **When:** Klient ponawia żądanie ze starą sesją, a następnie po ponownym uwierzytelnieniu odczytuje lub ponawia identyczną operację X.
- **Then:** Stara sesja zostaje odrzucona; autoryzowany odczyt/duplikat zwraca pierwotny sukces X bez nowej zmiany i bez zmiany licznika.

## BDD-22: Duplikat błędnej próby i zmiana treści

**Kryteria:** KA-13, KA-20.

- **Given:** Błędna weryfikacja operacji X podniosła licznik z 1 do 2.
- **When:** Klient ponawia identyczne X, następnie X z innym nowym PIN-em.
- **Then:** Duplikat zwraca poprzednie odrzucenie i licznik pozostaje 2; zmieniona treść z tym samym identyfikatorem jest odrzucona bez zapisu i dodatkowego naliczania.

## BDD-23: Równoległa zmiana z dwóch urządzeń

**Kryteria:** KA-21, KA-24.

- **Given:** Dwie ważne sesje A i B, różne identyfikatory; oba żądania używają 583026, nowe PIN-y to 729461 i 829461.
- **When:** Serwer otrzymuje żądania równolegle; harmonogram testu pozwala jednemu zatwierdzić zmianę jako pierwszemu.
- **Then:** Dokładnie jedno żądanie zapisuje zmianę; drugie nie nadpisuje wyniku i wymaga ponownego uwierzytelnienia z komunikatem KA-21. Działa wyłącznie zwycięski PIN.

## BDD-24: Zamknięcie po wysłaniu

**Kryteria:** KA-22, KA-29, KA-30.

- **Given:** Wysłano operację X, odpowiedź jest opóźniona.
- **When:** Użytkownik zamyka aplikację, serwer zapisuje zmianę, użytkownik otwiera aplikację.
- **Then:** Operacja nie jest anulowana; klient zachował identyfikator bez PIN-ów, po autoryzacji weryfikuje X i pokazuje sukces; nie uruchamia nowej zmiany.

## BDD-25: Wygaśnięcie sesji na granicy zapisu

**Kryteria:** KA-23.

- **Given:** Zegar serwera jest kontrolowany; sesja wygasa w T.
- **When:** W niezależnych wykonaniach zatwierdzenie próbuje nastąpić w T−1 ms, T, T+1 ms.
- **Then:** W T−1 ms poprawna operacja może zostać zatwierdzona i wygaśnięcie jej nie cofa. W T i T+1 ms zapis jest odrzucony: „Sesja wygasła. Zaloguj się ponownie.”, pola puste, stary PIN nadal obowiązuje.

## BDD-26: Unieważnienie wszystkich sesji i tokenów

**Kryteria:** KA-24.

- **Given:** Konto ma sesje A i B oraz tokeny odświeżania na dwóch urządzeniach.
- **When:** A zatwierdza zmianę; po zatwierdzeniu A i B wywołują chronione endpointy oraz próbują odświeżyć sesje.
- **Then:** Każde takie użycie wcześniejszych sesji i tokenów jest odrzucone; potwierdzenie wyniku dla inicjującego żądania nie tworzy nowej sesji; nowe logowanie wymaga nowego PIN-u.

## BDD-27: Brak PIN-ów w danych trwałych i telemetrii

**Kryteria:** KA-25, KA-26.

- **Given:** Włączono przechwytywanie URL-i, logów klienta/serwera, analityki, raportów awarii, powiadomień i audyt pamięci trwałej; używane są rozpoznawalne PIN-y testowe.
- **When:** W niezależnych wykonaniach następuje sukces, odrzucenie, błąd techniczny, timeout i kontrolowana awaria klienta.
- **Then:** Żadne z obserwowanych wyjść ani identyfikator operacji nie zawiera PIN-u. Audyt magazynów i konfiguracji potwierdza brak jawnego zapisu i odwracalnie odczytywalnego weryfikatora PIN-u; po opuszczeniu formularza brak roboczych wartości w stanie klienta.

## BDD-28: Certyfikat i szyfrowanie

**Kryteria:** KA-25.

- **Given:** Proxy testowe przedstawia kolejno certyfikat wygasły, dla innej nazwy hosta i niezaufany; osobny wariant oferuje połączenie nieszyfrowane.
- **When:** Klient próbuje wysłać zmianę.
- **Then:** Każde połączenie jest odrzucone przed wysłaniem PIN-ów; nie ma obejścia weryfikacji certyfikatu ani transmisji jawnej. W wariancie kontrolnym poprawny certyfikat umożliwia szyfrowaną transmisję.

## BDD-29: Tło i podgląd aplikacji

**Kryteria:** KA-26, KA-30.

- **Given:** Pola są wypełnione; osobny wariant ma już wysłaną operację X.
- **When:** Użytkownik przenosi aplikację do tła, otwiera przełącznik zadań i wraca.
- **Then:** Podgląd nie ujawnia PIN-ów; po powrocie pola są puste. Dla X klient zachowuje identyfikator bez PIN-ów i wznawia ustalanie wyniku.

## BDD-30: Brak internetu przed wysłaniem

**Kryteria:** KA-27.

- **Given:** Sieć jest niedostępna, a instrumentacja potwierdza brak wysłania jakichkolwiek danych żądania.
- **When:** Użytkownik zatwierdza poprawny formularz.
- **Then:** Widoczny komunikat „Brak połączenia. Żądanie zmiany PIN-u nie zostało wysłane.”; brak sukcesu, zapisu i zmiany licznika.

## BDD-31: Potwierdzony błąd techniczny

**Kryteria:** KA-28, KA-31.

- **Given:** Serwer kontrolowanie odrzuca operację przed zapisem i potwierdza brak zatwierdzenia.
- **When:** Użytkownik zatwierdza poprawny formularz i otrzymuje wynik.
- **Then:** Widoczny dokładny komunikat KA-28; działa stary PIN, licznik nie rośnie; nie pojawia się sukces.

## BDD-32: Utrata połączenia po wysłaniu

**Kryteria:** KA-29, KA-30, KA-31.

- **Given:** Żądanie X wysłano; połączenie zostaje przerwane. W wariancie a) serwer zapisuje zmianę, w b) potwierdza błąd przed zapisem.
- **When:** Klient wykrywa utratę sieci, następnie połączenie wraca i uprawniony użytkownik odzyskuje wynik X.
- **Then:** Do rozstrzygnięcia widoczny jest dokładny komunikat KA-29, bez deklaracji aktywnego PIN-u. Po autoryzowanej weryfikacji a) pokazuje sukces, b) komunikat KA-28; brak ponownego zapisu.

## BDD-33: Błąd HTTP bez dowodu wyniku

**Kryteria:** KA-29, KA-33.

- **Given:** Po zapisie odpowiedź zostaje zastąpiona przez gateway błędem HTTP 500/502, w oddzielnych wykonaniach.
- **When:** Klient odbiera błąd bez wiarygodnego potwierdzenia braku zapisu.
- **Then:** Przechodzi do weryfikacji, nie wyświetla komunikatu o pozostawieniu starego PIN-u; późniejszy odczyt X potwierdza sukces.

## BDD-34: Ochrona wyniku i wznowienie po restarcie

**Kryteria:** KA-03, KA-15, KA-20, KA-30.

- **Given:** Operacja X konta A ma nieznany wynik; aplikację uruchomiono ponownie. Identyfikator zna też konto B; dla A osobny wariant ma blokadę zmiany.
- **When:** Bez sesji oraz z sesją B następuje odczyt X; następnie A uwierzytelnia się i odczytuje X.
- **Then:** Nieuprawnione odczyty są odrzucone bez ujawnienia wyniku. A uzyskuje wynik także podczas blokady zmiany. Do rozstrzygnięcia klient nie pozwala uruchomić nowej operacji.

## BDD-35: Historyczny sukces po kolejnej zmianie

**Kryteria:** KA-20, KA-31.

- **Given:** X zmieniło PIN na 729461, potem Y poprawnie zmieniło go na 829461.
- **When:** Właściciel po autoryzacji odczytuje wynik X i próbuje uwierzytelnienia PIN-ami w niezależnych kontrolowanych próbach.
- **Then:** X nadal ma wynik sukcesu, nie wykonuje nowego zapisu; działa tylko 829461. Klient nie przedstawia historycznego PIN-u jako nadal obowiązującego.

## BDD-36: Oczekiwanie i wielokrotne kliknięcie

**Kryteria:** KA-32.

- **Given:** Odpowiedź poprawnej operacji jest opóźniona.
- **When:** Użytkownik wielokrotnie naciska zatwierdzenie, próbuje edytować wysłane dane, następnie opuszcza ekran.
- **Then:** Wysłano jedną operację; przed wyjściem widoczny jest stan oczekiwania, wysłane dane nie są edytowalne; wyjście jest możliwe i nie anuluje operacji.

## BDD-37: Granica timeoutu

**Kryteria:** KA-29, KA-32, KA-33.

- **Given:** Brak odpowiedzi i brak wcześniej wykrytej utraty sieci; t0 to wybranie „Zmień PIN”.
- **When:** Zegar testowy osiąga t0+9999 ms, następnie t0+10000 ms i t0+10001 ms.
- **Then:** Przy 9999 ms trwa oczekiwanie; przy 10000 ms i później widoczny jest komunikat KA-29. Nie następuje anulowanie ani nowe żądanie zapisu. Późniejszy sukces jest prawidłowo wyświetlany.

## BDD-38: Pomiar wydajności i granica 2000 ms

**Kryteria:** KA-34, KA-35.

- **Given:** Spełniono wszystkie warunki KA-35, użyto wersjonowanej macierzy; N≥1000 dla każdej jej konfiguracji, operacje z poprawnymi danymi i bez blokad.
- **When:** Dla każdej operacji mierzy się od kliknięcia do pierwszego potwierdzenia w UI; błędy i brak potwierdzenia pozostają w N.
- **Then:** Wynik 2000 ms jest terminowy, 2001 ms nieterminowy. Każda konfiguracja ma osobny wynik: PASS wyłącznie gdy terminowe/N≥0,95; dla N=1000 potrzeba co najmniej 950. Raport zawiera wszystkie pola KA-35.

## BDD-39: Weryfikacja reguły raportowania SLA

**Kryteria:** KA-34, KA-35.

- **Given:** Przygotowano syntetyczne wyniki N=1000: a) 950 terminowych, b) 949 terminowych, c) 940 terminowych, 40 opóźnionych i 20 błędów/braków odpowiedzi; d) brak wersji macierzy lub niespełniona sieć.
- **When:** Mechanizm raportowania ocenia dane.
- **Then:** a) PASS, b) FAIL, c) FAIL z mianownikiem 1000, d) pomiar nieważny bez deklaracji PASS. Test syntetyczny sprawdza ocenę; nie zastępuje pomiaru aplikacji.

### Dane BDD-05 — każdy wiersz osobno

| Klasa | Wartości |
| --- | --- |
| Długość 1, 4, 5, 7 | `7`, `7294`, `72946`, `7294618` |
| Litery i znaki specjalne | `72a461`, `-29461`, `72946!` |
| Spacje i znaki sterujące | `72 461`, ` 729461`, `729461 `, `729461` + nowa linia |
| Cyfry spoza ASCII | `٧٢٩٤٦١`, `７２９４６１` |
| Nadmierna długość | 10 000 znaków `7` przesłanych bezpośrednio do API; odrzucenie bez zapisu, bez obcięcia do 6 cyfr |

## Macierz pokrycia

| Kryterium | Scenariusze |
| --- | --- |
| KA-01 | BDD-01 |
| KA-02 | BDD-01 |
| KA-03 | BDD-02, BDD-34 |
| KA-04 | BDD-03 |
| KA-05 | BDD-04, BDD-12 |
| KA-06 | BDD-05, BDD-06, BDD-12 |
| KA-07 | BDD-07, BDD-12 |
| KA-08 | BDD-08, BDD-12 |
| KA-09 | BDD-06, BDD-09, BDD-12 |
| KA-10 | BDD-10, BDD-12 |
| KA-11 | BDD-11, BDD-12, BDD-13 |
| KA-12 | BDD-04, BDD-05, BDD-07, BDD-08, BDD-09, BDD-10, BDD-12 |
| KA-13 | BDD-04, BDD-05, BDD-10, BDD-11, BDD-12, BDD-13, BDD-14, BDD-17, BDD-18, BDD-22 |
| KA-14 | BDD-13, BDD-14, BDD-18 |
| KA-15 | BDD-15, BDD-16, BDD-18, BDD-34 |
| KA-16 | BDD-16, BDD-17 |
| KA-17 | BDD-13, BDD-15 |
| KA-18 | BDD-06, BDD-19, BDD-20 |
| KA-19 | BDD-20 |
| KA-20 | BDD-21, BDD-22, BDD-34, BDD-35 |
| KA-21 | BDD-23 |
| KA-22 | BDD-03, BDD-24 |
| KA-23 | BDD-20, BDD-25 |
| KA-24 | BDD-19, BDD-21, BDD-23, BDD-26 |
| KA-25 | BDD-27, BDD-28 |
| KA-26 | BDD-03, BDD-27, BDD-29 |
| KA-27 | BDD-30 |
| KA-28 | BDD-31 |
| KA-29 | BDD-24, BDD-32, BDD-33, BDD-37 |
| KA-30 | BDD-24, BDD-29, BDD-32, BDD-34 |
| KA-31 | BDD-21, BDD-31, BDD-32, BDD-35 |
| KA-32 | BDD-36, BDD-37 |
| KA-33 | BDD-33, BDD-37 |
| KA-34 | BDD-38, BDD-39 |
| KA-35 | BDD-38, BDD-39 |
