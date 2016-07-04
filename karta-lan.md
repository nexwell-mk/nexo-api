# Karta lan

## Zasada i parametry komunikacji

Komunikacja z kartą lan odbywa się za pomocą poleceń tekstowych wysyłanych w standardzie ASCII. Karta lan pełni w komunikacji rolę serwera. Klient (np. komputer PC lub smartfon) wysyła do karty lan dane w dowolnym momencie. Do odebrania danych z karty lan wymagana jest jej odpytanie odpowiednią komendą. 

Warstwa fizyczna komunikacji:
 - Ethernet

Transmisja TCP/IP:
 - port: 1024

Kodowanie znaków:
 - ASCII

Timeout odpowiedzi:
 - 20 ms

## Przykład komunikacji

Testowy skrypt umożliwiający zalogowanie się do systemu Nexo, wykonanie jednego polecenia i odpytanie o odpowiedź.

```
#!/bin/bash

TCP_HOST=192.168.0.100
TCP_PORT=1024
NEXOTALK_PASSWORD="password"
NEXOTALK_COMMAND="system command system"
SLEEP_TIME=0.1
HEX_PASSWORD=`(echo 0:; echo -n $NEXOTALK_PASSWORD | md5) | xxd -r`

exec 3< /dev/tcp/$TCP_HOST/$TCP_PORT             ; sleep $SLEEP_TIME
cat <&3 &
echo "plain" >&3                       ; echo "" ; sleep $SLEEP_TIME
echo $HEX_PASSWORD >&3                 ; echo "" ; sleep $SLEEP_TIME
echo "@00000000:ping" >&3              ; echo "" ; sleep $SLEEP_TIME
echo "@00000000:$NEXOTALK_COMMAND" >&3 ; echo "" ; sleep $SLEEP_TIME
echo "@00000000:get" >&3               ; echo "" ; sleep $SLEEP_TIME
exec 3<&-
exec 3>&-
kill %1
```

Pakiety można podejrzeć bezpośrednio przy użyciu narzędzi takich jak np. whireshark, tcpdump, ngrep.
```
# ngrep -x tcp and port 1024
interface: en0 (192.168.0.0/255.255.255.0)
filter: (ip or ip6) and ( tcp and port 1024 )
##
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AS]
  00 00                                                 ..              
##
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  57 65 6c 63 6f 6d 65 20    74 6f 20 4e 65 78 6f 21    Welcome to Nexo!
##
T 192.168.0.101:51005 -> 192.168.0.100:1024 [AP]
  70 6c 61 69 6e 0a                                     plain.          
#
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  4e 4f 20 75 53 53 4c                                  NO uSSL         
##
T 192.168.0.101:51007 -> 192.168.0.100:1024 [AP]
  5f 4d cc 3b 5a a7 65 d6    1d 83 27 de b8 82 cf 99    _M.;Z.e...'.....
  0a                                                    .               
#
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  4c 4f 47 49 4e 20 4f 4b                               LOGIN OK        
##
T 192.168.0.101:51005 -> 192.168.0.100:1024 [AP]
  40 30 30 30 30 30 30 30    30 3a 70 69 6e 67 0a       @00000000:ping. 
#
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  7e 30 30 30 30 30 30 30    30 3a 70 6f 6e 67          ~00000000:pong  
##
T 192.168.0.101:51005 -> 192.168.0.100:1024 [AP]
  40 30 30 30 30 30 30 30    30 3a 73 79 73 74 65 6d    @00000000:system
  20 63 6f 6d 6d 61 6e 64    20 73 79 73 74 65 6d 0a     command system.
#
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  43 4d 44 20 4f 4b                                     CMD OK          
##
T 192.168.0.101:51005 -> 192.168.0.100:1024 [AP]
  40 30 30 30 30 30 30 30    30 3a 67 65 74 0a          @00000000:get.  
#
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AP]
  7e 30 30 30 30 30 30 30    30 3a 4e 65 78 6f 20 35    ~00000000:Nexo 5
  2e 33 35 20 52 31 20 58    32 2e 20 43 7a 61 73 20    .35 R1 X2. Czas 
  64 7a 69 61 6c 61 6e 69    61 3a 20 30 20 64 6e 2e    dzialania: 0 dn.
  20 35 20 67 6f 64 7a 2e    20 32 33 20 6d 69 6e 2e     5 godz. 23 min.
###
T 192.168.0.100:1024 -> 192.168.0.101:51005 [AF]
  00 00 00 00 00 00                                     ......          
#
```
