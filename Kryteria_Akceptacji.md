# Kryteria akceptacji — zmiana PIN-u do aplikacji

## User Story

Jako zarejestrowany użytkownik aplikacji bankowości mobilnej chcę zmienić swój PIN do aplikacji w ustawieniach profilu, aby podnieść bezpieczeństwo mojego konta.

## Status i zakres

Wersja robocza po doprecyzowaniu kryteriów. Wartości liczbowe, reguły, ścieżka nawigacji i komunikaty są przyjętymi założeniami do zatwierdzenia przez właściciela produktu oraz osoby odpowiedzialne za bezpieczeństwo. Dokument dotyczy PIN-u do logowania do aplikacji, a nie PIN-u karty płatniczej.

## 1. Dostępność i przebieg zmiany

- **KA-01:** Funkcja jest dostępna dla zarejestrowanego, zalogowanego użytkownika z aktywną sesją pod ścieżką: Profil → Ustawienia → Bezpieczeństwo → Zmień PIN.
- **KA-02:** Formularz zawiera trzy wymagane pola: „Aktualny PIN”, „Nowy PIN”, „Powtórz nowy PIN” oraz przycisk „Zmień PIN”. Wartości PIN-ów są maskowane.
- **KA-03:** Wybranie „Zmień PIN” uruchamia walidację wymaganych pól, reguł nowego PIN-u, zgodności obu wpisów nowego PIN-u oraz poprawności aktualnego PIN-u.
- **KA-04:** Zmiana zostaje zapisana tylko wtedy, gdy wszystkie warunki walidacji są spełnione, sesja nadal jest aktywna i nie obowiązuje blokada zmiany PIN-u.
- **KA-05:** Po zapisaniu zmiany każde kolejne uwierzytelnienie wymagające PIN-u akceptuje nowy PIN i odrzuca poprzedni. Niepowodzenie walidacji nie zmienia obowiązującego PIN-u.
- **KA-06:** Po wygaśnięciu sesji system nie zapisuje zmiany i wyświetla „Sesja wygasła. Zaloguj się ponownie.”.

## 2. Reguły nowego PIN-u i ograniczenie prób

- **KA-07:** Nowy PIN składa się dokładnie z 6 cyfr ASCII od 0 do 9. Zera na początku są dozwolone. Litery, spacje i pozostałe znaki są odrzucane.
- **KA-08:** Nowy PIN różni się od aktualnego PIN-u.
- **KA-09:** System odrzuca PIN złożony z sześciu identycznych cyfr, np. `111111`.
- **KA-10:** System odrzuca PIN złożony z sześciu kolejnych cyfr rosnących albo malejących. Różnica między każdymi sąsiednimi cyframi wynosi odpowiednio +1 albo −1. Sekwencje nie zawijają się między 9 a 0; przykłady odrzucane: `012345`, `123456`, `654321`, `987654`.
- **KA-11:** Wartości w polach „Nowy PIN” i „Powtórz nowy PIN” muszą być identyczne.
- **KA-12:** Każda zakończona weryfikacja z niepoprawnym aktualnym PIN-em zwiększa licznik błędnych prób o 1. Błędy formatu nowego PIN-u, niezgodność jego powtórzenia i błędy techniczne nie zwiększają licznika.
- **KA-13:** Piąta kolejna błędna próba podania aktualnego PIN-u powoduje blokadę zmiany PIN-u na 15 minut od chwili tej próby. Blokada i licznik dotyczą konta użytkownika na wszystkich urządzeniach i sesjach; ponowne logowanie ich nie zeruje.
- **KA-14:** Podczas blokady system odrzuca żądania zmiany PIN-u i nie przedłuża czasu blokady. Po upływie 15 minut blokada ustaje, a licznik wynosi 0. Udana zmiana PIN-u również zeruje licznik.
- **KA-15:** Blokada opisana w KA-13 dotyczy funkcji zmiany PIN-u. Zasady blokowania logowania stanowią odrębne wymagania.

## 3. Informowanie o wyniku

- **KA-16:** System wyświetla „PIN został zmieniony” na ekranie zmiany PIN-u dopiero po potwierdzeniu zapisania zmiany przez system.
- **KA-17:** Błędy walidacji są prezentowane przy odpowiednich polach, zgodnie z tabelą:

| Warunek | Komunikat |
| --- | --- |
| Puste wymagane pole | „Pole jest wymagane.” |
| Niepoprawny aktualny PIN, gdy próba nie uruchamia blokady | „Aktualny PIN jest niepoprawny.” |
| Nowy PIN nie składa się dokładnie z 6 cyfr od 0 do 9 | „PIN musi składać się z 6 cyfr od 0 do 9.” |
| Nowy PIN jest identyczny z aktualnym | „Nowy PIN musi różnić się od aktualnego.” |
| Sześć identycznych cyfr lub sekwencja zabroniona w KA-10 | „PIN nie może składać się z identycznych ani kolejnych cyfr.” |
| Niezgodne powtórzenie nowego PIN-u | „Podane nowe PIN-y nie są identyczne.” |

- **KA-18:** Po uruchomieniu blokady i przy próbie zmiany podczas blokady system wyświetla na ekranie „Zmiana PIN-u jest zablokowana. Spróbuj ponownie po zakończeniu blokady.” oraz czas jej zakończenia w lokalnej strefie czasowej urządzenia.
- **KA-19:** Gdy system potwierdzi błąd techniczny i brak zapisu zmiany, wyświetla „Nie udało się zmienić PIN-u. Dotychczasowy PIN pozostaje aktywny. Spróbuj ponownie.”.
- **KA-20:** Utrata połączenia lub brak odpowiedzi po wysłaniu żądania nie są traktowane jako potwierdzenie niepowodzenia. System weryfikuje wynik tego samego żądania. Do ustalenia wyniku wyświetla „Nie można jeszcze potwierdzić zmiany PIN-u. Trwa weryfikacja.”. Po odzyskaniu połączenia weryfikacja jest wznawiana, również po ponownym otwarciu aplikacji.
- **KA-21:** Po ustaleniu wyniku system wyświetla komunikat sukcesu zgodny z KA-16 albo potwierdzonego niepowodzenia zgodny z KA-19. Ponowienie transmisji tego samego żądania nie powoduje dodatkowej zmiany PIN-u.

## 4. Czas odpowiedzi i oczekiwanie

- **KA-22:** Dla co najmniej 95% poprawnych operacji potwierdzenie zapisanej zmiany pojawia się w ciągu 2 sekund. Pomiar rozpoczyna się w chwili wybrania „Zmień PIN” po prawidłowym wypełnieniu formularza, a kończy w chwili wyświetlenia komunikatu „PIN został zmieniony”.
- **KA-23:** Pomiar KA-22 obejmuje co najmniej 1000 poprawnych operacji dla każdej konfiguracji urządzenia i systemu operacyjnego wskazanej w zatwierdzonej macierzy testowej. Warunki: opóźnienie sieci RTT ≤100 ms, brak utraty pakietów, przepustowość co najmniej 10 Mb/s w obu kierunkach i 100 równocześnie przetwarzanych żądań zmiany PIN-u z różnych kont.
- **KA-24:** Od wysłania żądania do ustalenia wyniku aplikacja pokazuje wskaźnik postępu i uniemożliwia ponowne wysłanie formularza.
- **KA-25:** Jeżeli w ciągu 10 sekund od wybrania „Zmień PIN” nie uzyskano potwierdzonego wyniku, aplikacja wyświetla komunikat weryfikacji z KA-20. Przekroczenie tego czasu nie oznacza automatycznego anulowania operacji.

## Decyzje do zatwierdzenia przed wdrożeniem

1. Długość PIN-u i katalog zabronionych wzorców.
2. Próg, zakres i czas blokady oraz zasady zerowania licznika.
3. Ścieżka nawigacji, etykiety pól i treści komunikatów.
4. Próg czasu odpowiedzi, warunki obciążenia i profil sieci.
5. Macierz wspieranych urządzeń i wersji systemów operacyjnych — należy wskazać jej konkretną wersję przed pomiarem KA-23.

Przyjęte reguły są propozycją dla tego User Story; nie stanowią deklaracji kompletnej polityki bezpieczeństwa aplikacji bankowej.
