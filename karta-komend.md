# Karta komend

# Zasada i parametry komunikacji

Komunikacja z kartą komend odbywa się za pomocą poleceń tekstowych wysyłanych w standardzie ASCII. Karta komend pełni w komunikacji rolę urządzenia podrzędnego. Urządzenie nadrzędne (np. komputer PC) wysyła do karty komend dane w dowolnym momencie. Do odebrania danych z karty komend wymagana jest jej odpytanie odpowiednią komendą. Karta komend ma 20 ms na wysłanie odpowiedzi i w tym czasie urządzenie nadrzędne nie powinno nadawać, ale nasłuchiwać odpowiedzi. Możliwe jest podłączenie kilku kart komend we wspólną sieć nadzorowaną przez jedno urządzenie nadrzędne.

Warstwa fizyczna komunikacji:
 - RS485

Transmisja UART:
 - 115200 bps
 - ilość bitów danych: 8
 - parzystość: bit parzystości (Odd)
 - ilość bitów stopu: 1

Kodowanie znaków:
 - ASCII

Timeout odpowiedzi:
 - 20 ms

Podłączenie karty do komputera PC wymaga użycia konwertera RS485<->RS232 (w przypadku gdy komputer jest wyposażony w port COM) lub RS485<->USB (w przypadku gdy komputer jest wyposażony w port USB).
