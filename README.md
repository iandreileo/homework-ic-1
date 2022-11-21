# Tema 1 IC - ILIE ANDREI-LEONARD 342C4

## Cerinta 1

Pas 1 si 2. Server-ul are urmatoarele functionalitati:

- obtinerea unui token de autentificare pentru user-ul `Anonymous`
- autentificarea pe baza unui token

Generarea token-ului se relealizeaza folosind algoritmul de criptare AES ECB imbunatatit cu un sistem rudimentar de integritate a mesajelor prin atasarea unui tag. Formula de obtinere a token-ului poate fi exprimata astfel:

```
token = XOR(AES_E(KEY, IV), user) +
	    SERVER_PUBLIC_BANNER +
	    SUBSTRING(AES_E(KEY, PADD(user)), INTEGRITY_LEN) # tag integritate

XOR, SUBSTRING - self-explainatory
AES_E- criptare AES ECB
PADD - padding cu bytes 00

KEY, IV - siruri de caractere random, generate la runtime
SERVER_PUBLIC_BANNER, INTEGRITY_LEN - constante necunoscute
```

Un token valid va fi de forma `token = encrypted user + server banner + tag`. Un token valid respecta urmatoarea proprietate:

```
SUBSTRING(AES_E(key, PADD(user)), INTEGRITY_LEN) = tag si
server banner = SERVER_PUBLIC_BANNER, unde

user = XOR(AES_E(KEY, IV), encrypted user)
```

Pas 3 si 4.

Cum nu cunoastem valoarea lui `INTEGRITY_LEN` si nici lungimea lui `SERVER_PUBLIC_BANNER`, va trebui sa executam atacul pentru toate lungimile posibile. De asemenea, stim ca `LENGTH(SERVER_PUBLIC_BANNER) + INTEGRITY_LEN = LENGTH(token) - LENGTH(encrypted user)`, iar `LENGTH(token) = 16` pe server (valoare obtinuta interogand server-ul pentru un token anonim).

In urma brute-force-ului efectuat pe server pentru valoarea lui `INTEGRITY_LEN`, am obtinut:

```
SERVER_PUBLIC_BANNER = b'\x01su\xa7\xe5\xf9'
INTEGRITY_LEN = 1
```

Cunoastem deja valoarea `encrypted user = XOR(AES_E(KEY, IV), GUEST_USER)` deoarce a fost obtinuta de la server. Putem calcula `AES_E(KEY, IV) = XOR(encrypted user, GUEST_USER)`. Dupa obtinerea cheii, putem pregati un token care sa contina user-ul tinta, anume `Ephvuln`.

Deoarece nu cunoastem valoarea `AES_E(KEY, PADD("Ephvuln"))`, va trebui sa incercam toate tag-urile posibile pana vom gasi un token valid. Deci vom incerca toate token-urile de forma:

```
token = XOR(AES_E(KEY, IV), "Ephvuln") + "\x01su\xa7\xe5\xf9" + <tag oarecare>
```

Deoarece `INTEGRITY_LEN = 1`, sunt doar 255 de tag-uri posibile, deci atacul ar trebui sa dureze cateva secunde.

Pas 5.

Codul atacului se afla in fisierul `exploit.py`. In urma executiei am obtinut: `CTF{Ez_T4g_Cr4ftyng}`.
