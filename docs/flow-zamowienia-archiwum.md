# Flow „Zamowienia-archiwum": archiwum potwierdzeń zamówień do audytu

## Cel
Handlowcy wysyłają klientom potwierdzenia zamówień ze swoich skrzynek. W mailu jest załącznik PDF o nazwie
`<cyfry>_<nr zamówienia>_<coś>.pdf`, np. `22576_0050435169_2202351608.pdf` albo `24470_0050437994_e-mail.pdf`.
Środkowy segment (10 cyfr zaczynających się od `005`) to numer zamówienia. Flow zapisuje każdy taki mail
i jego załącznik w SharePoint, w folderze o numerze zamówienia, żeby na audycie dało się szybko znaleźć wszystko.

Flow nie może czytać skrzynek handlowców. Maile zbiera do wspólnej skrzynki reguła Exchange (punkt 1).

## 1. Prośba do IT: reguła Exchange i wspólna skrzynka
Gotowa wiadomość:

> Dzień dobry,
> prosimy o:
> 1. Utworzenie wspólnej skrzynki (shared mailbox) `archiwum-zamowien@biesterfeld.com.pl`.
> 2. Utworzenie reguły przepływu poczty (Exchange transport rule) o nazwie „Archiwum potwierdzeń zamówień":
>    - warunek: nazwa dowolnego załącznika pasuje do wzorca `^\d+_005\d{7}_.*\.pdf$`, nadawca znajduje się w organizacji,
>    - akcja: dodaj odbiorcę w UDW: `archiwum-zamowien@biesterfeld.com.pl`,
>    - wyjątek: wiadomości, których jedynym adresatem jest ta skrzynka.
> 3. Nadanie kontu j.gral@biesterfeld.com.pl uprawnień Full Access do tej skrzynki.
>
> Cel: automatyczne archiwum potwierdzeń zamówień na potrzeby audytu. Prosimy o potwierdzenie, jak długo
> przechowujemy takie kopie (okres retencji) i kto ma dostęp.

Test reguły: wyślij mail z załącznikiem o nazwie `99999_0050000001_test.pdf`. Kopia ma pojawić się w skrzynce archiwum.

## 2. SharePoint
- Witryna (np. dział sprzedaży) z biblioteką dokumentów **Zamówienia**. Audytorzy: odczyt. Konto flow: zapis.
- Struktura: `Zamówienia / <numer 005xxxxxxx> / <data-godzina>_<nazwa>`.
- Opcjonalnie lista **Rejestr zamówień** (numer zamówienia, data, nadawca, odbiorcy, temat, link do pliku).

## 3. Flow (wyzwalany przy każdym nowym mailu we wspólnej skrzynce)
1. **Wyzwalacz:** „Gdy nadejdzie nowa wiadomość e-mail do skrzynki współdzielonej (V2)".
   Skrzynka `archiwum-zamowien@biesterfeld.com.pl`, folder Inbox, tylko z załącznikami: Tak, dołącz załączniki: Tak.
2. **Zastosuj do każdego**, nazwa `zalacznik`, na liście załączników (`attachments` z wyzwalacza).
3. **Warunek** (tryb zaawansowany) na nazwę załącznika:

```
@and(greater(length(split(items('zalacznik')?['name'],'_')),2),equals(length(split(items('zalacznik')?['name'],'_')[1]),10),isInt(split(items('zalacznik')?['name'],'_')[1]),startsWith(split(items('zalacznik')?['name'],'_')[1],'005'))
```

4. W gałęzi **Tak**:
   1. **Redaguj** `numer_zam`: `split(items('zalacznik')?['name'],'_')[1]`
   2. **Redaguj** `znacznik`: `concat(formatDateTime(triggerOutputs()?['body/receivedDateTime'],'yyyyMMdd-HHmmss'),'_',substring(guid(),0,8))`
   3. **SharePoint: Utwórz nowy folder** `/Zamowienia/<numer_zam>`. Jeśli folder już istnieje, ta akcja może dać błąd.
      Włóż ją w **Zakres** (Scope) i ustaw następną akcję „Uruchom po" na: powodzenie i niepowodzenie.
   4. **Outlook: Eksportuj wiadomość e-mail (V2)**: Id wiadomości z wyzwalacza, skrzynka archiwum.
   5. **SharePoint: Utwórz plik**: folder `/Zamowienia/<numer_zam>`, nazwa `concat(outputs('znacznik'),'.eml')`, zawartość z eksportu.
   6. **SharePoint: Utwórz plik**: ten sam folder, nazwa `concat(outputs('znacznik'),'_',items('zalacznik')?['name'])`,
      zawartość `base64ToBinary(items('zalacznik')?['contentBytes'])`.
   7. Opcjonalnie: **Utwórz element** w liście Rejestr zamówień.
5. Mail bez pasującego załącznika: flow nic nie robi.

Data i losowy końcówka w nazwach pliku zapobiegają nadpisywaniu przy kilku mailach do tego samego zamówienia.

## 4. Testy
| Test | Wynik oczekiwany |
|---|---|
| Mail z `99999_0050000001_test.pdf` | folder `0050000001`, plik .eml i PDF |
| Drugi mail do tego samego numeru | dwa komplety plików w tym samym folderze, bez błędu |
| Mail z `faktura.pdf` | nic się nie zapisuje |
| Mail z dwoma załącznikami o różnych numerach | mail zapisany w dwóch folderach |
| Wyszukanie numeru w SharePoint | zwraca wszystkie maile i załączniki |

## 5. Uwagi
- Dane klientów trafiają do wspólnego miejsca: ustalić z IT i inspektorem danych dostęp i retencję.
- Reguła i flow działają tylko dla wzorca `005` + 7 cyfr. Inna nazwa załącznika nie zostanie złapana.
- Maile sprzed uruchomienia reguły nie trafią do archiwum. Zebranie historii to osobna operacja po stronie IT (eDiscovery).
