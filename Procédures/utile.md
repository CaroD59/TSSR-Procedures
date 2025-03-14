## Classless et Classful

Les adresses IP sont divisées en 5 classes, mais seulement 3 sont utilisables :

| Classe | Description |
|--------|-------------|
| A      | Un seul octet pour la partie réseau |
| B      | 2 octets pour la partie réseau |
| C      | 3 octets pour la partie réseau |
| D      | Réservé aux protocoles |
| E      | Réservé aux tests pour l'ICANN |

### Plages des classes :

- **Classe A** : de `1.0.0.0` à `126.0.0.0`
- **Classe B** : de `128.0.0.0` à `191.0.0.0`
- **Classe C** : de `192.0.0.0` à `223.0.0.0`
- **Classe D** : de `224.0.0.0` à `239.0.0.0`
- **Classe E** : de `240.0.0.0` à `255.0.0.0`

### Adresses spéciales :
- `0.0.0.0` : signifie *ce réseau*.
- `255.255.255.255` : est l'adresse de broadcast.
- `127.0.0.1` : adresse de loopback (utilisée pour pointer vers soi-même).
- `169.254.0.0` à `169.254.255.255` : adresses APIPA, utilisées lorsqu'un appareil ne peut pas obtenir une adresse via DHCP.

### Adresses privées :
- **Classe A** : de `10.0.0.0` à `10.255.255.255`.
- **Classe B** : de `172.16.0.0` à `172.31.255.255`.
- **Classe C** : de `192.168.0.0` à `192.168.255.255`.

## Liste des protocoles les plus utilisés avec leurs ports :

| Protocole | Type | Port(s) |
|-----------|------|---------|
| FTP       | TCP  | 20/21   |
| SSH       | TCP  | 22     |
| Telnet    | TCP  | 23     |
| SMTP      | TCP  | 25     |
| DNS       | TCP/UDP | 53    |
| TFTP      | UDP  | 69     |
| HTTP      | TCP  | 80     |
| HTTPS     | TCP  | 443    |
| POP3      | TCP  | 110    |
| IMAP      | TCP  | 143    |
