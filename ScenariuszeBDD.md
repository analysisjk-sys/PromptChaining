# Scenariusze BDD — zmiana kodu PIN

## Zakres i założenia

Scenariusze dotyczą pięciu kryteriów podanych w wiadomości: PIN-u 4-cyfrowego i przepływu trzyetapowego.

Plik [Kryteria_Akceptacji.md](Kryteria_Akceptacji.md) w momencie analizy zawierał inne ustalenia: PIN 6-cyfrowy, formularz z trzema polami i SLA dla 95% operacji. Poniższe scenariusze odzwierciedlają dane z wiadomości, a nie pełną treść tego pliku.

Każdy scenariusz jest niezależny. Użytkownik jest zalogowany, ma aktywną sesję, a aktualny PIN wynosi `5830`, o ile nie wskazano inaczej.

## 1. Dostępność i nawigacja

### Scenariusz 1: Otwarcie formularza przez wskazaną ścieżkę

- **Given:** Użytkownik znajduje się na ekranie głównym aplikacji.
- **When:** Wybiera kolejno „Ustawienia” → „Bezpieczeństwo” → „Zmiana kodu PIN”.
- **Then:** Aplikacja wyświetla Krok 1 umożliwiający wprowadzenie aktualnego PIN-u.

### Scenariusz 2: Brak dostępu inną drogą

- **Given:** Użytkownik nie przeszedł wskazanej ścieżki nawigacji.
- **When:** Próbuje otworzyć formularz przez alternatywny punkt wejścia objęty testem, np. bezpośredni link.
- **Then:** Formularz zmiany PIN-u nie zostaje otwarty.

Ten scenariusz należy wykonać dla każdego alternatywnego punktu wejścia wskazanego w inwentaryzacji aplikacji.

## 2. Uwierzytelnienie i kolejność kroków

### Scenariusz 3: Poprawny aktualny PIN

- **Given:** Użytkownik znajduje się w Kroku 1.
- **When:** Wprowadza `5830` i zatwierdza.
- **Then:** System pozytywnie weryfikuje aktualny PIN i wyświetla Krok 2.

### Scenariusz 4: Niepoprawny aktualny PIN

- **Given:** Użytkownik znajduje się w Kroku 1.
- **When:** Wprowadza `5820` i zatwierdza.
- **Then:** Pozostaje w Kroku 1, Krok 2 jest niedostępny, a system wyświetla dokładnie „Podany kod PIN jest nieprawidłowy”. Aktualny PIN pozostaje niezmieniony.

### Scenariusz 5: Poprawny nowy PIN

- **Given:** Użytkownik pomyślnie zakończył Krok 1 w bieżącym przepływie.
- **When:** W Kroku 2 wprowadza `7294` i zatwierdza.
- **Then:** System wyświetla Krok 3. Aktualny PIN nadal wynosi `5830`.

### Scenariusz 6: Próba pominięcia kroku

- **Given:** W bieżącym przepływie użytkownik nie zakończył pomyślnie kroku poprzedzającego.
- **When:** Próbuje przejść do Kroku 2 albo Kroku 3.
- **Then:** System odmawia przejścia do wskazanego kroku i nie zmienia aktualnego PIN-u.

## 3. Reguły złożoności nowego PIN-u

### Scenariusz 7: Niepoprawna długość lub niedozwolone znaki

- **Given:** Użytkownik znajduje się w Kroku 2.
- **When:** Wprowadza kolejno, w niezależnych wykonaniach, jedną z wartości: pustą wartość, `7`, `729`, `72945`, `72a4`, `72 4`, `-729`.
- **Then:** System odrzuca wartość, wyświetla komunikat błędu walidacji, nie umożliwia przejścia do Kroku 3 i nie zmienia aktualnego PIN-u.

### Scenariusz 8: Cztery identyczne cyfry

- **Given:** Użytkownik znajduje się w Kroku 2.
- **When:** Wprowadza dowolną wartość ze zbioru `{0000, 1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999}` i zatwierdza.
- **Then:** System odrzuca PIN, wyświetla komunikat błędu walidacji, pozostaje w Kroku 2 i nie zmienia aktualnego PIN-u.

### Scenariusz 9: Kolejne cyfry rosnące lub malejące

- **Given:** Użytkownik znajduje się w Kroku 2.
- **When:** Wprowadza dowolną wartość ze zbioru `{1234, 2345, 3456, 4567, 5678, 6789, 9876, 8765, 7654, 6543, 5432, 4321, 3210}` i zatwierdza.
- **Then:** System odrzuca PIN, wyświetla komunikat błędu walidacji, pozostaje w Kroku 2 i nie zmienia aktualnego PIN-u.

### Scenariusz 10: Nowy PIN identyczny z aktualnym

- **Given:** Użytkownik znajduje się w Kroku 2, a aktualny PIN wynosi `5830`.
- **When:** Wprowadza `5830` jako nowy PIN i zatwierdza.
- **Then:** System odrzuca PIN, wyświetla komunikat błędu walidacji, pozostaje w Kroku 2 i nie zmienia aktualnego PIN-u.

## 4. Potwierdzenie i wynik operacji

### Scenariusz 11: Niezgodne potwierdzenie nowego PIN-u

- **Given:** W Kroku 2 użytkownik podał poprawny nowy PIN `7294` i przeszedł do Kroku 3.
- **When:** Wprowadza `7295` jako potwierdzenie i zatwierdza.
- **Then:** System wyświetla pod polem potwierdzenia dokładnie „Wprowadzone kody PIN różnią się od siebie”, nie zapisuje zmiany i pozostawia aktywny PIN `5830`.

### Scenariusz 12: Pomyślna zmiana PIN-u

- **Given:** Użytkownik pomyślnie zakończył Krok 1 i podał `7294` w Kroku 2.
- **When:** W Kroku 3 wprowadza `7294`, zatwierdza operację, a serwer potwierdza zapis.
- **Then:** Aplikacja wyświetla dedykowany ekran potwierdzenia z widocznym przyciskiem powrotu. W kolejnych niezależnych próbach uwierzytelnienia nowy PIN `7294` jest akceptowany, a poprzedni PIN `5830` jest odrzucany.

### Scenariusz 13: Brak przedwczesnego potwierdzenia sukcesu

- **Given:** Wszystkie trzy kroki zostały poprawnie wykonane, a żądanie zmiany wysłano do serwera.
- **When:** Serwer nie potwierdził jeszcze zapisania zmiany.
- **Then:** Aplikacja nie wyświetla ekranu sukcesu.

## 5. Wydajność i interfejs podczas oczekiwania

### Scenariusz 14: Potwierdzenie operacji w limicie czasu

- **Given:** Wszystkie trzy kroki zostały poprawnie wykonane, a pomiar działa we wskazanym środowisku testowym.
- **When:** Aplikacja wysyła żądanie zapisu zmiany w chwili `t0` i odbiera potwierdzenie serwera w chwili `t1`.
- **Then:** Serwer potwierdza zapis, a `t1 − t0 ≤ 2000 ms`. Wynik większy niż `2000 ms` oznacza FAIL.

### Scenariusz 15: Blokada interakcji i spinner

- **Given:** Aplikacja wysłała poprawne żądanie zmiany PIN-u, a odpowiedź serwera jest kontrolowanie opóźniona.
- **When:** Użytkownik próbuje edytować pola, ponownie zatwierdzić operację lub uruchomić nawigację wewnątrz aplikacji.
- **Then:** Żadna z tych interakcji nie wykonuje swojej akcji, nie zostaje wysłane kolejne żądanie zmiany PIN-u, a animowany spinner pozostaje widoczny przez cały okres oczekiwania.

## Decyzje wymagane do pełnej testowalności

- Definicja sekwencji, w tym sposób traktowania `1357` i `7890`.
- Dopuszczenie zer wiodących i cyfr spoza ASCII.
- Docelowy ekran przycisku powrotu.
- Warunki pomiaru SLA.
- Zatwierdzenie granic pomiaru w scenariuszu 14: od wysłania żądania do odebrania potwierdzenia.
- Uzgodnienie rozbieżności między danymi wejściowymi a plikiem Kryteria_Akceptacji.md.

Bez tych ustaleń nie można potwierdzić 100% weryfikowalności wymagań.
