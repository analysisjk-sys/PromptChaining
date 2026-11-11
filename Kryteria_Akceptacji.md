# Kryteria akceptacji — zmiana PIN-u do aplikacji

## User Story

Jako zarejestrowany użytkownik aplikacji bankowości mobilnej chcę zmienić swój PIN do aplikacji w ustawieniach profilu, aby podnieść bezpieczeństwo mojego konta.

## Status i zakres

Wersja końcowa po ocenie kompletności, 2026-11-11. Obejmuje poprawki zaakceptowane do włączenia przez użytkownika: PIN 6-cyfrowy, jeden formularz z trzema polami, blokadę 5 prób / 15 minut, unieważnienie sesji i odzyskanie nieznanego wyniku. Zastępuje wcześniejsze warianty z PIN-em 4-cyfrowym i przepływem trzyetapowym. Dotyczy PIN-u do aplikacji, nie PIN-u karty płatniczej.

Dokument określa zakres tego User Story, nie całą politykę bezpieczeństwa banku. Scenariusze i mapowanie pokrycia znajdują się w [ScenariuszeBDD.md](ScenariuszeBDD.md). Macierz wspieranych urządzeń jest wejściem do wykonania testów wydajności; jej konkretny identyfikator musi znaleźć się w raporcie, nie jest tu arbitralnie wymyślany.

## 1. Dostęp i formularz

- **KA-01:** Funkcja jest dostępna dla zarejestrowanego, zalogowanego użytkownika z aktywną sesją pod ścieżką Profil → Ustawienia → Bezpieczeństwo → Zmień PIN.
- **KA-02:** Formularz zawiera trzy wymagane pola: „Aktualny PIN”, „Nowy PIN”, „Powtórz nowy PIN” oraz przycisk „Zmień PIN”. Wartości wszystkich pól PIN są maskowane.
- **KA-03:** Serwer autoryzuje każde żądanie zmiany. Brak ważnej sesji lub próba zmiany PIN-u innego konta powoduje odrzucenie bez zmiany PIN-u któregokolwiek konta. Bezpośrednie wywołanie endpointu nie omija kontroli dostępu; sama ścieżka nawigacji nie stanowi zabezpieczenia.
- **KA-04:** Opuszczenie formularza przed wysłaniem żądania usuwa wpisane PIN-y. Ponowne otwarcie formularza wyświetla puste pola.

## 2. Walidacja

- **KA-05:** Puste wymagane pole blokuje zapis i otrzymuje komunikat „Pole jest wymagane.”.
- **KA-06:** Nowy PIN składa się dokładnie z 6 cyfr ASCII 0–9. Zera wiodące są dozwolone. Inna długość i pozostałe znaki są odrzucane bez obcinania, usuwania spacji lub normalizacji znaków. Komunikat: „PIN musi składać się z 6 cyfr od 0 do 9.”.
- **KA-07:** Nowy PIN różni się od aktualnego. W przeciwnym razie system odrzuca zmianę i wyświetla przy nowym PIN-ie „Nowy PIN musi różnić się od aktualnego.”.
- **KA-08:** System odrzuca każdy PIN złożony z sześciu identycznych cyfr. Komunikat przy nowym PIN-ie: „PIN nie może składać się z identycznych ani kolejnych cyfr.”.
- **KA-09:** System odrzuca PIN, gdy każda z pięciu różnic między sąsiednimi cyframi wynosi +1 albo każda wynosi −1. Sekwencje nie zawijają się między 9 a 0. Pełny zbiór zabronionych sekwencji: 012345, 123456, 234567, 345678, 456789, 987654, 876543, 765432, 654321, 543210. Komunikat jak w KA-08. Wartości 135790 i 789012 są dozwolone, jeśli spełniają pozostałe reguły.
- **KA-10:** Wartości „Nowy PIN” i „Powtórz nowy PIN” są identyczne. Niezgodność blokuje zapis i wyświetla przy powtórzeniu „Podane nowe PIN-y nie są identyczne.”.
- **KA-11:** Niepoprawny aktualny PIN blokuje zapis. Jeśli próba nie uruchamia blokady, komunikat przy aktualnym PIN-ie brzmi „Aktualny PIN jest niepoprawny.”.
- **KA-12:** Serwer egzekwuje wszystkie reguły walidacji także po pominięciu interfejsu. Odrzucona walidacja nie zmienia obowiązującego PIN-u. Kolejność kontroli: autoryzacja i blokada, wymagane pola i format, zgodność powtórzenia i wzorce, weryfikacja aktualnego PIN-u, porównanie nowego z aktualnym. Gdy występuje kilka błędów lokalnych, każde błędne pole otrzymuje odpowiedni komunikat; weryfikacja aktualnego PIN-u nie jest wtedy wykonywana.

## 3. Błędne próby i blokada

- **KA-13:** Każda zakończona weryfikacja niepoprawnego aktualnego PIN-u zwiększa licznik konta o 1. Odrzucenia wcześniejszych walidacji, błędy techniczne i ponowienia tego samego żądania nie zwiększają licznika. Licznik jest aktualizowany atomowo również przy żądaniach równoległych. Poprawna weryfikacja bez udanej zmiany nie zeruje licznika.
- **KA-14:** Piąta błędna weryfikacja od ostatniego wyzerowania licznika blokuje zmianę PIN-u na 15 minut od czasu serwera tej próby. Licznik i blokada dotyczą konta we wszystkich sesjach i na wszystkich urządzeniach. Ponowne logowanie ani reinstalacja aplikacji ich nie zerują.
- **KA-15:** W trakcie blokady każde nowe żądanie zmiany jest odrzucane, również z poprawnym aktualnym PIN-em. Próba podczas blokady nie zwiększa licznika i nie przedłuża blokady. Odczyt wyniku wcześniejszej operacji pozostaje możliwy po autoryzacji.
- **KA-16:** Dla czasu serwera t < koniec blokady blokada obowiązuje; dla t ≥ koniec blokady ustaje i licznik wynosi 0 przed obsługą nowej próby. Udana zmiana PIN-u również zeruje licznik.
- **KA-17:** Uruchomienie blokady i próba zmiany w jej trakcie wyświetlają „Zmiana PIN-u jest zablokowana. Spróbuj ponownie po zakończeniu blokady.” oraz czas zakończenia zgodny z czasem serwera, przeliczony na lokalną strefę urządzenia z oznaczeniem strefy/offsetu. Zmiana zegara urządzenia nie skraca blokady. Blokada dotyczy zmiany PIN-u; reguły blokowania logowania są odrębne.

## 4. Zapis i równoległość

- **KA-18:** Zapis następuje tylko po spełnieniu wszystkich walidacji, przy aktywnej sesji i braku blokady w chwili zatwierdzenia transakcji. Po sukcesie nowe uwierzytelnienie PIN-em akceptuje nowy PIN i odrzuca poprzedni. Komunikat „PIN został zmieniony” pojawia się wyłącznie po potwierdzeniu zapisu.
- **KA-19:** Zapis jest atomowy i trwały: awaria przed zatwierdzeniem pozostawia poprzedni PIN, a awaria po zatwierdzeniu pozostawia nowy. Nigdy nie występuje stan z dwoma aktywnymi PIN-ami ani bez aktywnego PIN-u.
- **KA-20:** Każda operacja ma identyfikator przypisany do konta. Powtórzenie identycznego żądania z tym samym identyfikatorem zwraca wynik tej samej operacji bez ponownego zapisu i naliczania błędnych prób. Użycie identyfikatora z innymi danymi jest odrzucane bez zmiany PIN-u. Odczyt wyniku wymaga autoryzacji właściciela konta.
- **KA-21:** Dla dwóch równoległych operacji z różnymi identyfikatorami, poprawną sesją i tym samym aktualnym PIN-em najwyżej jedna kończy się sukcesem. Druga nie nadpisuje pierwszej i wymaga ponownego uwierzytelnienia; aplikacja wyświetla „Dane uwierzytelniające uległy zmianie. Zaloguj się ponownie.”.
- **KA-22:** Anulowanie przed wysłaniem żądania pozostawia PIN bez zmian i usuwa wartości pól. Zamknięcie aplikacji po wysłaniu nie anuluje operacji; jej wynik podlega weryfikacji zgodnie z KA-29–KA-31.

## 5. Sesja i ochrona danych

- **KA-23:** Wygaśnięcie sesji przed zatwierdzeniem transakcji odrzuca zmianę z komunikatem „Sesja wygasła. Zaloguj się ponownie.” i usuwa wartości pól. Wygaśnięcie po zatwierdzeniu nie cofa zapisu. Kontrola ważności sesji i zapis nie mogą pozostawiać okna umożliwiającego zapis po wygaśnięciu sesji.
- **KA-24:** Udana zmiana unieważnia wszystkie wcześniejsze sesje i tokeny odświeżania konta, w tym sesję bieżącą, jako część zatwierdzenia zmiany. Każde chronione żądanie rozpoczęte po zatwierdzeniu ze starymi danymi sesji jest odrzucane. Po odebraniu wyniku aplikacja pokazuje „PIN został zmieniony” na ekranie potwierdzenia z przyciskiem „Przejdź do logowania”, prowadzącym do ekranu logowania. Po utracie odpowiedzi wynik jest dostępny po ponownym uwierzytelnieniu; mechanizm odzyskiwania dostępu nie wymaga zachowania starej sesji.
- **KA-25:** PIN-y nie występują w URL-ach, logach klienta i serwera, analityce, raportach awarii ani powiadomieniach; nie są zapisywane w postaci jawnej w trwałej pamięci klienta lub serwera. Weryfikatory nie pozwalają na odczyt PIN-u w postaci jawnej. Transmisja wymaga szyfrowanego połączenia i poprawnej weryfikacji certyfikatu; nieważny lub niezaufany certyfikat blokuje wysłanie PIN-ów. Identyfikator operacji nie zawiera PIN-u.
- **KA-26:** Po zakończeniu operacji lub opuszczeniu formularza pola i robocze wartości PIN-ów są usuwane z aplikacji. Podgląd aplikacji w przełączniku zadań nie ujawnia PIN-ów. Wznowienie po przejściu do tła wyświetla puste pola; dla wysłanej operacji zachowany jest tylko identyfikator potrzebny do odzyskania wyniku, bez wartości PIN-ów.

## 6. Sieć, błędy i odzyskanie wyniku

- **KA-27:** Jeśli aplikacja potwierdza, że żądanie nie zostało wysłane z powodu braku połączenia, wyświetla „Brak połączenia. Żądanie zmiany PIN-u nie zostało wysłane.”. Nie prezentuje sukcesu i nie zmienia PIN-u. Gdy brak wysłania nie jest pewny, stosuje KA-29.
- **KA-28:** Potwierdzony przez system błąd techniczny bez zapisu skutkuje komunikatem „Nie udało się zmienić PIN-u. Dotychczasowy PIN pozostaje aktywny. Spróbuj ponownie.”. Licznik błędnych prób nie rośnie.
- **KA-29:** Utrata połączenia po wysłaniu lub brak odpowiedzi oznacza nieznany wynik. Aplikacja nie deklaruje sukcesu ani aktywności starego PIN-u; wyświetla „Nie można jeszcze potwierdzić zmiany PIN-u. Trwa weryfikacja.”. Sam błąd HTTP lub timeout nie stanowi dowodu braku zapisu.
- **KA-30:** Po odzyskaniu połączenia system wznawia ustalanie wyniku tej samej operacji, również po zamknięciu i ponownym otwarciu aplikacji. W razie braku ważnej sesji wymaga ponownego uwierzytelnienia właściciela konta. Zapisany identyfikator operacji nie uprawnia samodzielnie do odczytu wyniku. Do rozstrzygnięcia nie można uruchomić nowej operacji zmiany w tym kliencie.
- **KA-31:** Po ustaleniu wyniku aplikacja wyświetla potwierdzony sukces z KA-18 albo potwierdzony brak zapisu z KA-28; rozpoznane odrzucenia walidacji, blokady i sesji zachowują właściwe komunikaty. Wynik zatwierdzonej operacji pozostaje sukcesem także po późniejszej zmianie PIN-u; nie oznacza to, że PIN z historycznej operacji jest nadal aktualny.

## 7. Oczekiwanie i wydajność

- **KA-32:** Od wysłania żądania do ustalenia wyniku aplikacja pokazuje stan oczekiwania i uniemożliwia ponowne wysłanie formularza oraz edycję wysłanych danych. Można opuścić ekran lub zamknąć aplikację bez anulowania operacji. Po przejściu do stanu weryfikacji komunikat KA-29 zastępuje sam spinner.
- **KA-33:** Jeżeli po 10 000 ms od wybrania „Zmień PIN” nie ma potwierdzonego wyniku, aplikacja przechodzi do stanu weryfikacji z KA-29. Granica jest włącznie: przy 10 000 ms brak wyniku oznacza stan weryfikacji. Timeout nie anuluje operacji. Wcześniejsze wykrycie utraty połączenia może uruchomić ten stan wcześniej.
- **KA-34:** Dla co najmniej 95% poprawnych operacji komunikat „PIN został zmieniony” pojawia się w czasie ≤2000 ms. t0 to wybranie „Zmień PIN” z prawidłowymi danymi; t1 to pierwsze wyświetlenie potwierdzenia zapisu. PASS: liczba operacji z t1−t0 ≤2000 ms / liczba wszystkich poprawnie zainicjowanych operacji próby ≥0,95. Błędy i brak potwierdzenia w próbie nie są usuwane z mianownika i nie są wynikami terminowymi.
- **KA-35:** Pomiar KA-34 obejmuje co najmniej 1000 operacji na każdą konfigurację urządzenia i systemu z wersjonowanej macierzy wspieranych urządzeń produktu. Warunki: RTT ≤100 ms, brak utraty pakietów, przepustowość ≥10 Mb/s w obu kierunkach i 100 równolegle przetwarzanych żądań z różnych kont. Raport zapisuje model urządzenia, wersję OS i aplikacji, wersję macierzy, warunki sieci, obciążenie, N i liczbę wyników terminowych. Brak macierzy lub niespełnienie warunków oznacza pomiar nieważny, a nie PASS.
