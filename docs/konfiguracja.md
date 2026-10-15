# Konfiguracja

Na podstawie przykładowego maila (FW od koleżanki, oryginał przychodzi bezpośrednio z gov).

| Parametr | Wartość |
|---|---|
| Skrzynka ogólna (źródło) | `sent@biesterfeld.com.pl` |
| Nadawca powiadomień gov | `sent@mf.gov.pl` |
| Temat | `Komunikat: Nr SENT<RRRRMMDD><nr> [SENT_<kod>], Id dokumentu: <id1> / <id2>` |
| Numer SENT | z tematu, wzorzec `SENT\d{14}` (np. `SENT20261009001084`; w numerze jest data RRRRMMDD) |
| Adres osoby zakładającej | To oryginalnego maila, z pominięciem `sent@biesterfeld.com.pl` |
| Załączniki | `.xml` i `.xml.html` ze szczegółami (nie są potrzebne do przypomnienia) |
| Uruchomienie | codziennie 7:30, Europe/Warsaw |
| Temat przypomnienia | `Przypomnienie: zamknij SENT {numer} – zostało {2\|1} dni` |

## Do ustalenia
- Który kod `[SENT_xxx]` oznacza założenie zgłoszenia (to jest „pierwszy mail", od którego liczymy dni). Przykład miał `SENT_112`.
- Który kod oznacza zamknięcie zgłoszenia. Po nim flow pomija przypomnienia dla tego numeru.
- Czy 10 dni liczymy od daty pierwszego maila, czy od daty zakodowanej w numerze SENT.
