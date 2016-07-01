# NexoTalk

NexoTalk jest protokołem umożliwiającym komunikację z centalami systemu Nexo.
Do komunikacji w określonym standardzie wymagane jest urządzenie pośredniczące:

 - RS-485 - karta komend
 - Ethernet - karta lan

Specyfika komunikacji przy wykorzystaniu danej karty jest opisana
w osobnych plikach.

# Ramka komunikacyjna

Ramka polecenia od urządzenia nadrzędnego ma postać:
```
@<adres-karty>:<dane><NUL>
```
`<adres-karty>` - 8-cyfrowa liczba szesnastkowa (cyfry: 0-9 oraz a-f lub A-F) identyfikująca kartę (standardowo: `00000000`);

`<dane>` - dane tekstowe wysyłane do karty, zawierające komendą i argumenty; ilość danych nie może przekroczyć 240 znaków

`<NUL>` - znak ASCII o kodzie 0, stanowiący znacznik końca transmisji

Ramka odpowiedzi karty ma postać:
```
~<adres-karty>:<dane><NUL>
```
`<adres-karty>` - identycznie jak w przypadku ramki polecenia

`<dane>` - dane tekstowe wysyłane od karty, o maksymalnej długości 64 znaków

`<NUL>` - znak ASCII o kodzie 0, stanowiący znacznik końca transmisji


# Komendy

Pole `<dane>` w ramce polecenia przyjmuje postać:
```
<komenda> <argument>
```
Zestawienie formatu komend:
```
komenda         argument      
======================================================
system          polecenie do centrali i jego parametr
get             -brak-
ping            -brak-
```
## Komenda `system`

Przesyła `<argument>` jako dane do centrali, składające się z polecenia oraz jego parametru w formacie:
```
<polecenie> <parametr>
```
Zestawienie formatu poleceń:
```
polecenie      parametr   
======================================================
info           tekst
warning        tekst
error          tekst
message        tekst
logic          komenda zewnetrzna tekstowa (do 7 znaków)
L              komenda zewnetrzna tekstowa (do 7 znaków)
command        komenda systemowa tekstowa
C              komenda systemowa liczbowa
```
Opis dostępnych komend systemowych znajduje się w sekcji Komendy systemowe.

## Komenda `get`

Odpytuje kartę o obecność danych od centrali. W przypadku ich braku karta wysyła ramkę odpowiedzi, w której pole `<dane>` jest puste. W przypadku obecności danych pole `<dane>` przyjmuje zawsze wartość nadaną przez użytkownika podczas konfiguracji działania systemu Nexo lub stanowi tekst odpowiedzi na komendę pytająca o stan zasobu systemu.

## Komenda `ping`

Służy sprawdzeniu stanu komunikacji z kartą. Po komendzie ping karta odsyła w ciągu 20 ms ramkę odpowiedzi, w której pole `<dane>` ma wartość `pong`. Komenda ta obsługiwana jest bez udziału centrali systemu.


# Komendy systemowe

## Komendy systemowe tekstowe

Sposób formatowania komend oraz odpowiedzi zwrotnych jest identyczny z formatem poleceń SMS. Komendy sterujące zasobami nie generują informacji zwrotnych, za wyjątkiem sytuacji w których wystąpi błąd - otrzymane zostanie nieznane polecenie lub polecenie będzie dotyczyło nieistniejącego zasobu lub zasobu, którego dana komenda nie dotyczy. W takich przypadkach po komendzie `get` zostanie zwrócony komunikat z opisem błędu. Komendy odpytujące o status zasobów zwracają informacje w postaci komunikatu tekstowego o określonym formacie.

Dostępne polecenia systemowe:
```
składnia                                 działanie
=====================================================================================
uzbroj <hasło> <nazwa_partycji>          uzbraja partycję za pomocą hasła użytkownika
rozbroj <hasło> <nazwa_partycji>         rozbraja partycję za pomocą hasła użytkownika
podnies <nazwa_rolety>                   podnosi roletę
opusc <nazwa_rolety>                     opuszcza roletę
wlacz <nazwa_wyjscia>                    włącza wyjście przekaźnikowe, OC lub oświetleniowe
wylacz <nazwa_wyjscia>                   wyłącza wyjście przekaźnikowe, OC lub oświetleniowe
otworz                                   otwiera drzwi wideodomofonu
ustaw <temperatura> <nazwa_termostatu>   ustawia próg termostatu na zadaną wartość
                                         (liczba całkowita ze znakiem)
stan <nazwa_zasobu>                      odpytuje system o stan danego zasobu (partycji,
                                         wejścia lub wyjścia); format odpowiedzi
                                         w formacie zależnym od typu zasobu
```
Nazwy zasobów zawierające spację należy podawać w apostrofach.

## Komendy systemowe liczbowe


# Przykłady

W poniższych przykładach założono, że karta jest skonfigurowana w systemie na obsługę adresu 0.0.0.0 (szesnastkowo: 0x00.0x00.0x00.0x00). Każdą z podanych ramek należy ponadto zakończyć znakiem ASCII o kodzie 0.

## (1) Pingowanie

Po wysłaniu przez urządzenie nadrzędne ramki:
```
@00000000:ping
```
karta odeśle:
```
~00000000:pong
```
## (2) Wyświetlanie komunikatów tekstowych na panelu LCD

Po wysłaniu następujących poleceń:
```
@00000000:system info Hello World!
@00000000:system warning Hello World!
@00000000:system error Hello World!
```
na panelu LCD systemu pojawią się komunikaty odpowiadające swym priorytetem, kolejno, informacji, ostrzeżeniu i komunikatowi błędu.

## (3) Ustawianie zasobów systemu

Przykładowe użycie poleceń systemowych:
```
@00000000:system command wlacz swiatlo
@00000000:system command wylacz 'swiatlo w kuchni'
@00000000:system command uzbroj 5678 Dom
@00000000:system command ustaw +21 termostat-przedpokoj
```
## (4) Odpytanie o stan partycji
```
@00000000:system command stan nazwa_partycji
```
Po wysłaniu tego komunikatu w buforze karty po upływie od kilku do kilkudziesięciu milisekund (w zależności od aktualnego obciążenia centrali zadaniami) pojawi się odpowiedź od systemu. Można ją pobrać za pomocą komendy 'get':
```
@00000000:get
```
Karta lan w przeciągu 20 ms odpowie ramką odpowiedzi:
```
~00000000:nazwa_partycji jest uzbrojona
```
## (5) Użycie poleceń w logice systemu

W konfiguracji logiki systemu Nexo możliwe jest zarówno sprawdzenie czy od karty przyszła określona komenda tekstowa jak i wysyłanie określonych tekstów. Po skonfigurowaniu logiki o następującej postaci:

Warunek#1: Komenda zewnętrzna: "Test"
Akcja#1: Wyślij wiadomość do karty: "Test OK"
Tabela prawdy: 2

będzie możliwa następująca wymiana ramek komunikacyjnych:
```
@00000000:system logic Test
@00000000:get
~00000000:Test OK
```
