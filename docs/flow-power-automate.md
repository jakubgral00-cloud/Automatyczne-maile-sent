# Flow „SENT-przypomnienia" w Power Automate

Zasada: za „pierwszy mail" dla numeru SENT uznajemy **najwcześniej odebrany** mail z tym numerem
w skrzynce ogólnej. Dni liczymy od jego daty odebrania (czas Europe/Warsaw, w dobach kalendarzowych).
Przypomnienie idzie 8. dnia (zostają 2 dni) i 9. dnia (zostaje 1 dzień).
Kod `[SENT_xxx]` w temacie nie jest używany.

Zmienna `tz` = `Central European Standard Time`.

## Kroki

1. **Recurrence**: codziennie 07:30, strefa „(UTC+01:00) Sarajevo, Skopje, Warsaw, Zagreb".
2. **Initialize variable** `dzisiaj` (String):
   `formatDateTime(convertTimeZone(utcNow(),'UTC','Central European Standard Time'),'yyyy-MM-dd')`
3. **Get emails (V3)** (Office 365 Outlook):
   - Original Mailbox Address: `sent@biesterfeld.com.pl`
   - Folder: Inbox, From: `sent@mf.gov.pl`, Subject Filter: `Komunikat: Nr SENT`
   - Top: 500, Include Attachments: Nie, Only with Attachments: Nie
4. **Filter array** `kandydaci`: z `body('Get_emails_(V3)')?['value']`, warunek (tryb zaawansowany):
   ```
   @or(
     equals(div(sub(ticks(variables('dzisiaj')), ticks(formatDateTime(convertTimeZone(item()?['receivedDateTime'],'UTC','Central European Standard Time'),'yyyy-MM-dd'))), 864000000000), 8),
     equals(div(sub(ticks(variables('dzisiaj')), ticks(formatDateTime(convertTimeZone(item()?['receivedDateTime'],'UTC','Central European Standard Time'),'yyyy-MM-dd'))), 864000000000), 9)
   )
   ```
5. **Apply to each** po `body('kandydaci')`:
   1. **Compose** `numer`: `first(split(last(split(items('Apply_to_each')?['subject'],'Nr ')),' '))`
   2. **Get emails (V3)** `maile_tego_sent`: ta sama skrzynka, Subject Filter: `outputs('numer')`, Top 50.
   3. **Compose** `najwczesniejszy`: `first(sort(body('maile_tego_sent')?['value'],'receivedDateTime'))`
   4. **Condition**: `outputs('najwczesniejszy')?['id']` jest równe `items('Apply_to_each')?['id']`.
      Jeśli nie, to nie jest pierwszy mail i pomijamy (dzięki temu kolejne maile do tego samego SENT nie dają dubli).
   5. Gałąź „Tak":
      - **Compose** `dni_do_konca`: `if(equals(div(sub(ticks(variables('dzisiaj')), ticks(formatDateTime(convertTimeZone(items('Apply_to_each')?['receivedDateTime'],'UTC','Central European Standard Time'),'yyyy-MM-dd'))), 864000000000), 8), 2, 1)`
      - **Compose** `odbiorca` (To + CC bez skrzynki ogólnej):
        `replace(replace(toLower(concat(coalesce(items('Apply_to_each')?['toRecipients'],''),';',coalesce(items('Apply_to_each')?['ccRecipients'],''))),'sent@biesterfeld.com.pl',''),';;',';')`
        Jeśli wynik po usunięciu średników jest pusty, adresatem jest `sent@biesterfeld.com.pl`.
      - **Send an email from a shared mailbox (V2)**: skrzynka `sent@biesterfeld.com.pl`,
        Do: `odbiorca`, DW: `sent@biesterfeld.com.pl`,
        Temat: `Przypomnienie: zamknij SENT @{outputs('numer')} – zostało @{outputs('dni_do_konca')} dni`
        (dla 1 dnia: „został 1 dzień"; można zrobić `if`),
        Treść: szablon z `docs/szablony/przypomnienie-8d.html` lub `-9d.html`, podstawić `{{NUMER_SENT}}`,
        `{{DATA_ZALOZENIA}}` (data odebrania pierwszego maila), `{{TERMIN}}` (data odebrania + 10 dni).

## Testy
- Wysłać na skrzynkę testową maile z tematem `Komunikat: Nr SENT20260101000001 [SENT_112], Id dokumentu: 1 / 2`
  i ręcznie ustawić okno (np. tymczasowo 0 dni zamiast 8/9 w kroku 4).
- Sprawdzić: drugi mail z tym samym numerem nie wysyła drugiego przypomnienia, adresat bez skrzynki ogólnej, DW na skrzynkę ogólną.

## Ograniczenia
- Brak informacji o zamknięciu SENT: przypomnienie przyjdzie także dla zamkniętych zgłoszeń (treść mówi, żeby wtedy zignorować).
  Jeśli przy zamknięciu przychodzi osobny mail z gov, można dodać krok pomijający numery z takim mailem.
- Skrzynka ogólna ma mieć uprawnienie „Send as" dla konta uruchamiającego flow.
