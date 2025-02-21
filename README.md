# voip
A brief introduction and how to for VoIP

# VOIP Dai Ni Keitai

1\. Installation des dépendances

Pour installer les paquets nécessaires, exécutez la commande suivante :

    sudo apt install build-essential wget libncurses5-dev libssl-dev libxml2-dev libsqlite3-dev uuid-dev perl libwww-perl sox mpg123
    

* * *

2\. Installation et configuration d'Asterisk
--------------------------------------------

### 2.1. Téléchargement et compilation d'Asterisk

Accédez au répertoire /usr/src​, téléchargez l'archive d'Asterisk, puis décompressez-la :

    cd /usr/src
    sudo wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-22-current.tar.gz
    sudo tar xvf asterisk-22-current.tar.gz
    cd asterisk-22.2.0
    

Procédez ensuite à la configuration, compilation et installation d'Asterisk :

    sudo ./configure
    sudo make
    sudo make install
    sudo make samples
    sudo make config
    

* * *

3\. Configuration de PJSIP
--------------------------

### 3.1. Modification du fichier pjsip.conf​

Ouvrez le fichier de configuration :

    sudo nano /etc/asterisk/pjsip.conf
    

Ajoutez ou modifiez les paramètres suivants :

    ; ---------------- TRANSPORT UDP ----------------
    [transport-udp]
    type=transport
    protocol=udp
    bind=0.0.0.0:5060
    
    ; ---------------- TRANSPORT TLS ----------------
    [transport-tls]
    type=transport
    protocol=tls
    bind=0.0.0.0:5061
    cert_file=/etc/asterisk/keys/certificate.pem
    priv_key_file=/etc/asterisk/keys/private.key
    method=tlsv1_2
    
    ; ---------------- TEMPLATE DES ENDPOINTS ----------------
    [endpoint_internal](!)
    type=endpoint
    context=from-internal
    disallow=all
    allow=ulaw
    transport=transport-tls
    
    [auth_userpass](!)
    type=auth
    auth_type=userpass
    
    [aor_dynamic](!)
    type=aor
    max_contacts=10
    
    ; ---------------- CONFIGURATION DES UTILISATEURS ----------------
    [alice](endpoint_internal)
    auth=alice
    aors=alice
    [alice](auth_userpass)
    password=bonjour
    username=alice
    [alice](aor_dynamic)
    
    [bob](endpoint_internal)
    auth=bob
    aors=bob
    [bob](auth_userpass)
    password=bonjour
    username=bob
    [bob](aor_dynamic)
    
    [julien](endpoint_internal)
    auth=julien
    aors=julien
    [julien](auth_userpass)
    password=bonjour
    username=julien
    [julien](aor_dynamic)
    

### 3.2. Génération des certificats TLS

Exécutez la commande suivante pour générer les certificats :

    cd /etc/asterisk/keys
    openssl req -x509 -newkey rsa:2048 -keyout private.key -out certificate.pem -days 365 -nodes
    

* * *

4\. Configuration des extensions
--------------------------------

### 4.1. Modification du fichier extensions.conf​

Ouvrez le fichier :

    sudo nano /etc/asterisk/extensions.conf
    

Ajoutez les règles d'appel suivantes :

    [from-internal]
    ; Extensions directes des utilisateurs
    exten => 6001,1,Dial(PJSIP/alice,10)
    same => n,VoiceMail(6001)
    same => n,Hangup()
    
    exten => 6002,1,Dial(PJSIP/bob,10)
    same => n,VoiceMail(6002)
    same => n,Hangup()
    
    exten => 6004,1,Dial(PJSIP/julien,10)
    same => n,VoiceMail(6004)
    same => n,Hangup()
    
    ; IVR principal (Menu interactif)
    exten => 6003,1,Answer()
    same => n,Set(TIMEOUT(response)=10)
    same => n,agi(googletts.agi,"Bonjour, et bienvenue à la plateforme",fr)
    same => n,agi(googletts.agi,"Appuyez sur 1 pour joindre le service compte",fr)
    same => n,agi(googletts.agi,"Appuyez sur 2 pour joindre le service RH.",fr)
    same => n,agi(googletts.agi,"Appuyez sur 3 pour joindre le service informatique",fr)
    same => n,WaitExten(5)
    
    ; Options de l'IVR
    exten => 1,1,Goto(from-internal,6001,1)
    exten => 2,1,Goto(from-internal,6002,1)
    exten => 3,1,Goto(from-internal,6004,1)
    
    ; Gestion des entrées invalides
    exten => i,1,agi(googletts.agi,"Option non valide.",fr)
    same => n,Goto(6003,1)
    
    ; Gestion du timeout
    exten => t,1,agi(googletts.agi,"Vous n'avez pas saisi d'option.",fr)
    same => n,Goto(6003,1)
    
    ; Boîte vocale générale
    exten => 6099,1,VoiceMailMain()
    same => n,Hangup()
    

* * *

5\. Configuration de la messagerie vocale
-----------------------------------------

### 5.1. Modification du fichier voicemail.conf​

Ouvrez le fichier :

    sudo nano /etc/asterisk/voicemail.conf
    

Ajoutez les paramètres suivants :

    [general]
    format=wav49|gsm|wav|ulaw
    
    [default]
    6001 => 1234, alice
    6002 => 1234, bob
    6004 => 1234, julien
    

* * *

6\. Installation de GoogleTTS
-----------------------------

    wget -O GoogleTTS.tar.gz http://github.com/zaf/asterisk-googletts/tarball/master --no-check-certificate
    tar -xvf GoogleTTS.tar.gz
    cd zaf-asterisk-googletts-5566ddc
    tcp googletts.agi /var/lib/asterisk/agi-bin/
    

* * *

7\. Redémarrage d'Asterisk
--------------------------

Appliquez les modifications en redémarrant le service Asterisk :

    sudo systemctl restart asterisk.service
