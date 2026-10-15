# Wdrożenie krok po kroku (laptop służbowy)

Wszystko robi się w przeglądarce, na koncie służbowym Microsoft 365. Nic nie trzeba instalować.

## 0. Uzgodnić z IT (jednorazowo)
- Konto, na którym powstanie flow, ma mieć do `sent@biesterfeld.com.pl` uprawnienia **Full Access** (czytanie) i **Send As** (wysyłka jako skrzynka ogólna).
- Sprawdzić, czy Power Automate nie jest zablokowany polityką DLP.
- Najlepiej użyć konta serwisowego, żeby flow nie przestał działać po odejściu pracownika.

## 1. Utworzyć flow
1. Wejść na make.powerautomate.com i zalogować się kontem służbowym.
2. Utwórz → **Zaplanowany przepływ w chmurze**, nazwa `SENT-przypomnienia`, powtarzaj co 1 dzień.
3. W kroku Recurrence (opcje zaawansowane): strefa czasowa Warszawa, godzina 7, minuta 30.

## 2. Dodać akcje
Dokładnie według `flow-power-automate.md`. Nazwy akcji w wyrażeniach muszą się zgadzać z nazwami w flow
(spacje zamieniają się na `_`). Po wklejeniu wyrażeń sprawdzić, czy nie ma błędów.

## 3. Test przed uruchomieniem
1. Tymczasowo w kroku Filter array zamienić 8 i 9 na 0 i ustawić adresata na własny adres.
2. **Test → Ręcznie**. Sprawdzić: przypomnienie przyszło, numer SENT i liczba dni się zgadzają, skrzynka ogólna jest w DW.
3. Uruchomić drugi raz: nie może przyjść duplikat dla numeru z kilkoma mailami.
4. Przywrócić 8 i 9 oraz właściwego adresata.

## 4. Włączyć i pilnować
- Włączyć flow, dodać współwłaściciela, przez pierwsze dni sprawdzać „Historię uruchomień".
- Włączyć powiadomienia o błędach.
- Po zamknięciu pierwszego SENT sprawdzić, czy przyszedł mail o zamknięciu. Jeśli tak, trzeba dopisać pomijanie zamkniętych.
