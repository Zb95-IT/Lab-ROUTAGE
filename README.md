# Challenge Packet Tracer — Routage Statique

> **Formation :** TSSR — Wild Code School  
> **Auteur :** Zinedine Balamane  
> **GitHub :** [Zb95-IT](https://github.com/Zb95-IT)  
> **Objectif :** Configurer l'adressage IPv4 statique d'un réseau multi-routeurs et choisir le type de routage statique adapté à chaque situation.

---

## Topologie du réseau

![Topologie complète](screenshots/topologie.png)

Le réseau est composé de **4 sous-réseaux**, **3 routeurs**, **1 PC** et **1 serveur** interconnectés en ligne.  
Chaque routeur joue un rôle distinct selon sa position dans la chaîne.

---

## Plan d'adressage

| Équipement | Interface          | Adresse IP     | Masque          | Passerelle    |
|------------|--------------------|----------------|-----------------|---------------|
| PC0        | NIC                | 192.168.1.10   | 255.255.255.0   | 192.168.1.1   |
| R1         | GigabitEthernet0/0 | 192.168.1.1    | 255.255.255.0   | —             |
| R1         | GigabitEthernet0/1 | 192.168.2.1    | 255.255.255.0   | —             |
| R2         | GigabitEthernet0/0 | 192.168.2.2    | 255.255.255.0   | —             |
| R2         | GigabitEthernet0/1 | 192.168.3.1    | 255.255.255.0   | —             |
| R3         | GigabitEthernet0/0 | 192.168.3.2    | 255.255.255.0   | —             |
| R3         | GigabitEthernet0/1 | 192.168.4.1    | 255.255.255.0   | —             |
| Server0    | NIC                | 192.168.4.10   | 255.255.255.0   | 192.168.4.1   |

---

## Logique du routage statique

| Routeur | Type de route       | Justification                                                                 |
|---------|---------------------|-------------------------------------------------------------------------------|
| R1      | Route par défaut    | Routeur en bout de chaîne — tout ce qui n'est pas le Réseau A va vers R2.     |
| R2      | Routes spécifiques  | Routeur central — doit savoir précisément où envoyer le trafic dans chaque direction. |
| R3      | Route par défaut    | Routeur en bout de chaîne — tout ce qui n'est pas le Réseau D va vers R2.     |

---

## Tables de routage

### R1 — `show ip route`

![R1 show ip route](screenshots/r1_show_ip_route.png)

R1 est connecté directement aux réseaux A (`192.168.1.0/24`) et B (`192.168.2.0/24`).  
Une **route par défaut** (`S* 0.0.0.0/0`) pointe vers R2 (`192.168.2.2`), ce qui lui permet d'atteindre tous les réseaux au-delà sans avoir à les connaître individuellement.

---

### R2 — `show ip route`

![R2 show ip route](screenshots/r2_show_ip_route.png)

R2 est le routeur central, connecté aux réseaux B et C.  
Il dispose de deux **routes statiques spécifiques** :
- `192.168.1.0/24` via R1 (`192.168.2.1`) → pour atteindre le Réseau A (PC0)
- `192.168.4.0/24` via R3 (`192.168.3.2`) → pour atteindre le Réseau D (Server0)

---

### R3 — `show ip route`

![R3 show ip route](screenshots/r3_show_ip_route.png)

R3 est connecté directement aux réseaux C (`192.168.3.0/24`) et D (`192.168.4.0/24`).  
Une **route par défaut** (`S* 0.0.0.0/0`) pointe vers R2 (`192.168.3.1`), lui permettant d'atteindre tous les réseaux en amont via R2.

---

## Tests de connectivité

### Ping depuis PC0 vers R3 (192.168.4.1)

![Ping PC0 vers 192.168.4.1](screenshots/ping_pc0_vers_r3.png)

Le ping depuis PC0 (`192.168.1.10`) vers l'interface de R3 (`192.168.4.1`) est **100% réussi** (0% perte).  
Le paquet traverse R1 → R2 → R3, validant la cohérence du routage de bout en bout.

---

### Ping depuis Server0 vers R1 (192.168.1.1)

![Ping Server0 vers 192.168.1.1](screenshots/ping_server0_vers_r1.png)

Le ping depuis Server0 (`192.168.4.10`) vers l'interface de R1 (`192.168.1.1`) est **100% réussi** (0% perte).  
Le chemin retour R3 → R2 → R1 fonctionne correctement, confirmant que le routage est bidirectionnel.

---

## Commandes clés utilisées

```bash
# Entrer en mode de configuration
enable
configure terminal

# Nommer le routeur
hostname R1

# Configurer une interface
interface GigabitEthernet0/0
ip address 192.168.1.1 255.255.255.0
no shutdown                            # Activer l'interface (éteinte par défaut sur Cisco)
exit

# Route par défaut (R1 et R3)
ip route 0.0.0.0 0.0.0.0 192.168.2.2

# Route spécifique (R2)
ip route 192.168.1.0 255.255.255.0 192.168.2.1
ip route 192.168.4.0 255.255.255.0 192.168.3.2

# Sauvegarder la configuration
write memory

# Vérifier la table de routage
show ip route

# Vérifier l'état des interfaces
show ip interface brief
```

---

## Structure du dépôt

```
challenge-routage-statique/
├── README.md
└── screenshots/
    ├── topologie.png
    ├── r1_show_ip_route.png
    ├── r2_show_ip_route.png
    ├── r3_show_ip_route.png
    ├── ping_pc0_vers_r3.png
    └── ping_server0_vers_r1.png
```
