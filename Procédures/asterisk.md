# Asterisk Server Configuration

## Installation et Configuration de Asterisk

### Configuration du Serveur Asterisk
- Mettre en NAT
- Éditer Virtual Network Editor :
  - VMnet8 > Change Settings > Décocher DHCP

### Installation Asterisk
```sh
apt update
apt upgrade
apt install perl libwww-perl sox mpg123 flac
```

### Installation IVR
```sh
cd /var/lib/asterisk/agi-bin
wget -4 https://raw.github.com/zaf/asterisk-googletts/master/googletts.agi
chmod +x googletts.agi
```

### Vérification des Modules
```sh
asterisk -rvvv
module show
```

## Configuration SIP
```sh
cp /etc/asterisk/sip.conf /etc/asterisk/sip.conf.original
> /etc/asterisk/sip.conf
nano /etc/asterisk/sip.conf
```
### Contenu du fichier `sip.conf`
```ini
[general]
language=fr
bindaddr=0.0.0.0
dtmfmode=rfc2833
disallow=all
allow=ulaw
allow=alaw
hassip=yes
hasiax=no
callwaiting=yes
transfer=yes
canpark=yes
hasvoicemail=yes
deny=0.0.0.0/0.0.0.0
permit=10.125.14.0/255.255.255.0
qualify=yes

[100]
secret=1234
callerid='Ben' <100>
mailbox=100@default
type=friend
host=dynamic
nat=no
context=local

[101]
secret=1234
callerid='Caro' <101>
mailbox=101@default
type=friend
host=dynamic
nat=no
context=local

[102]
secret=1234
callerid='Nico' <102>
mailbox=102@default
type=friend
host=dynamic
nat=no
context=local

[200]
secret=1234
callerid='Arthur' <200>
mailbox=200@default
type=friend
host=dynamic
nat=no
context=test
```

## Configuration des Extensions
```sh
cp /etc/asterisk/extensions.conf /etc/asterisk/extensions.conf.original
> /etc/asterisk/extensions.conf
nano /etc/asterisk/extensions.conf
```
### Contenu du fichier `extensions.conf`
```ini
[default]
[local]
; Appels normaux dans le contexte local
exten => _1XX,1,NoOp(Appel de ${CALLERID(num)} vers ${EXTEN})
exten => _1XX,n,GotoIf($["${CALLERID(num)}" = "100" & "${EXTEN}" = "101"]?transfert102)
exten => _1XX,n,Dial(SIP/${EXTEN},10)
exten => _1XX,n,VoiceMail(${EXTEN}@default,u)
exten => _1XX,n,Hangup()

; Gestion du transfert pour 100 qui appelle 101
exten => 101,n(transfert102),NoOp(🔀 Transfert automatique 100 → 102)
exten => 101,n,Wait(1)
exten => 101,n,Transfer(SIP/102)
exten => 101,n,Hangup()

exten => 11,1,Goto(IVR,11,1)

[local]
; Accès à la messagerie vocale pour 888
exten => 888,1,VoiceMailMain(${CALLERID(num)})

; Appels en même temps entre 100, 101, et 102
exten => 2000,1,Answer()
exten => 2000,n,Dial(SIP/100&SIP/101&SIP/102,15)
exten => 2000,n,Hangup()

[test]
; Contexte test pour 200
exten => _1XX,1,NoOp(Appel entrant de ${CALLERID(num)} vers ${EXTEN})
exten => _1XX,n,Dial(SIP/${EXTEN},10)
exten => _1XX,n,Hangup()

; Faire sonner 100, 101 et 102 en même temps quand 200 appelle
exten => 2000,1,Answer()
exten => 2000,n,Dial(SIP/100&SIP/101&SIP/102,15)
exten => 2000,n,Hangup()

exten => 777, 1, VoiceMailMain(${CALLERID(num)})

exten => _1XX, 1, Goto(local,${EXTEN},1)

[IVR]
exten => 11,1,Answer()
exten => 11,2,Set(TIMEOUT(response)=10)
exten => 11,3,agi(googletts.agi,"Bienvenue dans la formation TSSR 2025",fr,any)
exten => 11,4,agi(googletts.agi,"Qui souhaitez vous joindre?",fr,any)
exten => 11,5,agi(googletts.agi,"Pour 100 tapez 1",fr,any)
exten => 11,6,agi(googletts.agi,"Pour 101 tapez 2",fr,any)
exten => 11,7,agi(googletts.agi,"Pour 102 tapez 3",fr,any)
exten => 11,8,agi(googletts.agi,"Appuyez sur dièse si vous souhaitez réécouter ce message",fr,any)
exten => 11,9,WaitExten()

exten => 1,1,Goto(local,100,1)
exten => 2,1,Goto(local,101,1)
exten => 3,1,Goto(local,102,1)
exten => _[3-9#],1,Goto(IVR,11,3)
exten => t,1,Goto(local,100,3)
```

## Vérification et Redémarrage
```sh
asterisk -rvv
reload
sip show users
systemctl restart asterisk
```

## Configuration de la Messagerie
```sh
cp /etc/asterisk/voicemail.conf /etc/asterisk/voicemail.conf.original
> /etc/asterisk/voicemail.conf
nano /etc/asterisk/voicemail.conf
```
### Contenu du fichier `voicemail.conf`
```ini
[general]
format=ulaw|alaw
attach=yes
serveremail=test@default
emailsubject=Nouveau message de ${VM_CIDNAME}
emailbody=Bonjour ${VM_NAME},\n\nTu as un message de ${VM_CIDNAME} d'une durée de {VM_DUR}
maxmsg=20
maxsecs=30
minsecs=2
maxlogins=3
review=yes
saycid=yes

[default]
100 = 1234, Ben, 100@default
101 = 1234, Caro, 101@default
102 = 1234, Nico, 102@default
200 = 1234, Arthur, 200@default
```

## Vérification Finale
```sh
asterisk -rvv
reload
```

------------------------------------------------------------------

# Client Asterisk :
**En NAT**  

*Ne rien changer dans les adresses IP, nom de machine, etc...*  

### Installation de MicroSip :  
1. Ajouter un compte  
2. Renseigner les informations suivantes :  
   - **Nom du compte** : `101`  
   - **Server SIP** : `192.168.75.133`  
   - **Proxy SIP** : `192.168.75.133`  
   - **Nom d'utilisateur** : `101`  
   - **Domaine** : `m2i.local`  
   - **Login** : `101`  
   - **Mot de passe** : `1234`  
   - **Transport** : `UDP + TCP`  

------------------------------------------------------------------

# Sur notre machine :  
### Installation de MicroSip :  
1. Ajouter un compte  
2. Renseigner les informations suivantes :  
   - **Nom du compte** : `100`  
   - **Server SIP** : `192.168.75.133`  
   - **Proxy SIP** : `192.168.75.133`  
   - **Nom d'utilisateur** : `100`  
   - **Domaine** : `m2i.local`  
   - **Login** : `100`  
   - **Mot de passe** : `1234`  
   - **Transport** : `UDP + TCP`  
