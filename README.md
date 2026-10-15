# Automatyczne przypomnienia o zamknięciu SENT (Biesterfeld)

Gdy system gov wysyła powiadomienie o założeniu SENT, osoba, która go założyła, dostaje przypomnienie
8 i 9 dni później (2 i 1 dzień przed upływem 10 dni na zamknięcie).

Działa jako zaplanowany flow w Power Automate (Microsoft 365), bez własnego serwera.
Logika bezstanowa: raz dziennie flow szuka w skrzynce ogólnej powiadomień odebranych 8 i 9 dni temu
i wysyła przypomnienie do osoby z oryginalnego maila (skrzynka ogólna w kopii).

- `docs/konfiguracja.md`: parametry do uzupełnienia
- `docs/szablony/`: treść przypomnień (`{{NUMER_SENT}}`, `{{DATA_ZALOZENIA}}`, `{{TERMIN}}`)
- `flow/`: eksport flow (po zbudowaniu)

Status: szablony i dokumentacja gotowe, flow czeka na przykładowy mail z powiadomieniem.
