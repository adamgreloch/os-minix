# System płatności w MINIX-ie

Celem zadania jest umożliwienie procesom w systemie MINIX posiadania pieniędzy i
dokonywania przelewów wzajemnych.

Każdy proces otrzymuje na start `INIT_BALANCE` jednostek waluty. Następnie
procesy mogą wzajemnie przelewać sobie pieniądze, tj. proces `P` może zlecić
przelew `n` jednostek waluty procesowi Q. Aby przelew się udał, proces `P` musi mieć
na swoim koncie przynajmniej `n` jednostek waluty (stan konta procesu nie może być
ujemny), a proces `Q` musi mieć na swoim koncie nie więcej niż `MAX_BALANCE` - `n`
jednostek waluty. Ponadto, jako podstawowe zabezpieczenie przed praniem brudnych
pieniędzy, wymagamy, żeby procesy `P` i `Q` nie były w relacji potomek-przodek.
Jeśli przelew się udaje, to stan konta procesu `P` zmniejsza się o `n` jednostek
waluty, zaś stan konta procesu `Q` zwiększa się o `n` jednostek waluty.

Pieniądze procesów nie są dziedziczone - kiedy proces kończy działanie,
zgromadzone przez niego jednostki waluty znikają.

Uwaga: przyznawanie każdemu procesowi na start nowych jednostek waluty
nieuchronnie prowadzi do inflacji pieniądza, ale ten problem pozostawmy
ekonomistom. Nowe wywołanie systemowe

Zadanie polega na dodaniu wywołania systemowego `PM_TRANSFER_MONEY` oraz funkcji
bibliotecznej `int transfermoney(pid_t recipient, int amount)`. Funkcja powinna
być zadeklarowana w pliku `unistd.h`. Stałe `INIT_BALANCE = 100` i `MAX_BALANCE =
1000` powinny być zdefiniowane w pliku `minix/config.h`.

Wywołanie funkcji `int transfermoney(pid_t recipient, int amount)` powinno
zrealizować przelew `amount` jednostek waluty z konta procesu wywołującego funkcję
na konto procesu o identyfikatorze recipient. W przypadku powodzenia wykonania
przelewu funkcja zwraca stan konta procesu, który ją wywołał, po wykonaniu tego
przelewu.

Uwaga: proces może sprawdzić swój stan konta, np. przelewając sobie samemu `0`
jednostek waluty.

W przypadku kiedy przelew nie udaje się, funkcja `transfermoney` zwraca w wyniku
`-1` i ustawia `errno` na odpowiedni kod błędu:

* jeśli `recipient` nie jest identyfikatorem aktualnie działającego procesu, to
  errno zostaje ustawione na `ESRCH`;
* jeśli `recipient` jest identyfikatorem procesu będącego potomkiem lub
  przodkiem procesu wywołującego funkcję `transfermoney`, to `errno` zostaje
  ustawione na `EPERM`;
* jeśli wartość `amount` jest ujemna lub proces wywołujący funkcję
  `transfermoney` ma na koncie mniej niż `amount` jednostek waluty lub proces o
  identyfikatorze `recipient` ma więcej niż `MAX_BALANCE` - `amount` jednostek
  waluty, to `errno` zostaje ustawione na `EINVAL`.

Działanie funkcji `transfermoney()` powinno polegać na użyciu nowego wywołania
systemowego `PM_TRANSFER_MONEY`, które należy dodać do serwera PM. Do przekazania
parametrów należy zdefiniować własny typ komunikatu.
