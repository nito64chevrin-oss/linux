# Partie 1 – Prise en main et sécurisation

## 1. Accès à l’interface

### 1.1 Connection à l'interface web

![alt text](image-36.png)
![alt text](image-37.png)

### 1.2 Questions

#### Quelle est l’adresse IP du LAN  ?

- Par défaut il s'agit de 192.168.1.1 donc ici on la configure pour correspondre à la nôtre donc ici 192.168.170.2

#### Quelle est l’adresse IP du WAN ?

- 10.0.2.15 ici 2 car il s'agit de mon deuxième adaptateur
![alt text](image-38.png)

#### Pourquoi utilise-t-on HTTPS ?

- Elle est utilisée pour des raisons de sécurité, elle évite d'avoir les échanges entre pfsense et le client clairs pour les pirates

#### Pourquoi faut-il changer les identifiants par défaut sur un pare-feu ?

- Il faut les changer car admin/pense sont publiquement partagés et donc exposent notre pare-feu

---

## 2. Sécurisation de l’accès administrateur

### 2.1 Modifier compte admin

![alt text](image-39.png)

### 2.2 Questions

#### Où se gèrent les utilisateurs ?

![alt text](image-40.png)

#### Qu’est-ce qu’un mot de passe robuste ?

- Il s'agit d'un mot de passe qui ne se trouve pas par la devinette ou par le dictionnaire, c'est-à-dire :
- des caractères spéciaux "/,#,!"
- pas de mots complets présents dans le dico
- long mdp +12 ou 14 caractères

#### Pourquoi sécuriser en priorité l’accès admin sur un équipement réseau ?

- L'admin est le plus haut niveau de privilège et donc permet de tout modifier, dont la protection et les accès

# Partie 2 – Comprendre les interfaces réseau

## 3. érification des interfaces

### 3.1 Affectation WAN / LAN

![alt text](image-41.png)
![alt text](image-42.png)

### 3.2 Questions

#### Quelle interface permet l’accès Internet ?

- Il s'agit du WAN (Wide Area Network)

#### Quelle interface correspond au réseau interne ?

- Il s'agit du LAN (Local Area Network)

#### Que se passerait-il si les interfaces étaient inversées ?

- Il y aurait plusieurs problèmes liés à la sécurité avec un filtrage du pare-feu inefficace, admin exposé, etc. Mais aussi la perte de la connexion à internet, routage NAT perturbé, etc

# Partie 3 – Configuration des services réseau

## 4. DHCP

### 4.1 Configurer serveur DHCP

![alt text](image-43.png)

### 4.2 Questions

#### Pourquoi utiliser DHCP plutôt qu’une IP fixe ?

- DHCP est un gain de temps et évite les erreurs car il assigne automatiquement les adresses IP et évite les conflits plus probables avec des IP fixes qui sont appliqués manuellement

#### Quelle plage d’adresses choisir ?

- Le mieux serait entre 192.168.170.100 et 192.168.170.200 car l'adresse de notre pfsense est 192.168.170.2

#### Quelles adresses faut-il éviter d’inclure dans la plage ?

- Il faut éviter l'adresse de sous-réseau (192.168.170.0),
- l'adresse de podcast (192.168.170.255),
- l'adresse pfseense (192.168.170.2),
- Les adresses rajoutées manuellement

### 4.3 Vérification

- Oui, si le serveur DHCP est activé sur la machine avec Ubuntu et que le DHCP Server est bien configuré sur pfSense

![alt text](image-51.png)

Ici on a obtenue 192.168.170.100

## 5. DNS

### 5.1 Activer et configurer le résolveur DNS
- Activé
![alt text](image-44.png)
- Configurer
![alt text](image-45.png)

### 5.2 Questions

#### Pourquoi un pare-feu peut-il jouer le rôle de serveur DNS ?

- Dans un réseau, il en est le point de passage obligatoire entre le LAN et le WAN, il connaît l'ensemble des segments réseau et est la passerelle par défaut des machines clientes
-  En termes de sécurité, il filtre les requêtes DNS et protège des des domaines malveillants, il active DNSSEC qui vérifie les réponses et protège contre le DNS spoofing et protège du DNS tunneling (exfiltrer des données via requêtes DNS)
- DNS Spoofing : Il s'agit d'une méthode pour envoyer des fausses réponses dans le cache DNS et mettre à disposition des utilisateurs des liens pour qu'ils soient dirigés vers de mauvais sites web

#### Que se passe-t-il si le DNS ne fonctionne pas mais que le ping vers 8.8.8.8 fonctionne ?

- Ce qui va arriver, c'est que les navigateurs web affichent des messages d'erreur, mais en revanche ceux qui utilisent les adresses IP fonctionnent. Donc, un ping google.com ne fonctionne pas, mais un ping 8.8.8.8 oui

# Partie 4 – Autoriser l’accès Internet

## 6. Règles de pare-feu

### 6.1 Configuration des règles pour accéder à internet

![alt text](image-46.png)
![alt text](image-47.png)

### 6.2 Questions

#### Quelle doit être la source ?

- La source est l'origine du trafic et donc la machine qui initient la connexion. Ce que l'on choisit est LAN net qui représente toutes les adresses IP qui appartiennent au sous-réseau LAN

#### Quelle doit être la destination ?

- La destination est où le trafic se dirige et généralement on choisit any car on ne peut pas prévoir ce que les utilisateurs veulent visiter comme site web

#### Faut-il autoriser tous les protocoles ?

- Non, en terme de sécurité cela serait une mauvaise idée, il vaut mieux autoriser strictement le nécessaire

### 6.3 Tests 

#### Ping pfsense

![alt text](image-48.png)

#### ping 8.8.8.8

![alt text](image-49.png)

#### Test DNS

![alt text](image-50.png)

#### Accès web

![alt text](image-52.png)

## 7. NAT

### 7.1 Verification NAT

![alt text](image-53.png)
![alt text](image-54.png)
![alt text](image-55.png)

### 7.2 Questions

#### Pourquoi le NAT est-il nécessaire avec une interface WAN en NAT ?

- Sur internet, seules les adresses IP publiques sont routables. Si une adresse n'est pas publique, elle est ignorée et donc la NAT va traduire cette adresse et convertir les adresses IPv4

#### Quelle est la différence entre NAT automatique et manuel ?

- La différence est que la NAT automatique va gérer les règles NAT lui-même alors que manuellement on les définit à la main

#### Comment vérifier qu’une traduction d’adresse a lieu ?

![alt text](image-56.png)

# Partie 5 – Filtrage

## 8. Blocage d’un site spécifique

### 8.1 Bloquer l'accès à un site

![alt text](image-57.png)

![alt text](image-58.png)

### 8.2 Questions

#### Faut-il bloquer par IP ou par nom de domaine ?

- Par nom de domaine, elle est plus lisible et bloque tous les IP associés au nom de domaine mais elle est simple à contourner avec un DNS externe
- Par IP, difficile à contourner, mais il faut souvent l'actualiser car l'IP peut être changée et une adresse IP peut couvrir plusieurs sites et donc provoquer des blocages involontaires

#### Que se passe-t-il si le site utilise HTTPS ?

- Les trafics sont chiffrés et donc nécessitent une inspection SSL/TLS

#### Pourquoi le blocage par IP peut-il être contourné ?

- On peut utiliser un VPN, utiliser un proxy, le site change d'IP fréquent chez les grands sites comme Facebook et certains navigateurs comme Google et Firefox utilisent DNS over HTTPS qui va chiffrer les requêtes DNS

### 8.3 Tester

![alt text](image-59.png)

## 9. Blocage d’une catégorie de sites (jeux d’argent)

### 9.1 Bloquer plusieurs sites

![alt text](image-60.png)

![alt text](image-61.png)

![alt text](image-62.png)

### 9.2 Questions

#### Pourquoi ne pas créer une règle par site ?

- Car si l'on veut gérer plusieurs cela va vite prendre beaucoup de temps ainsi que devenir illisible et provoquer une perte de performance

#### Où se créent les alias ?

![alt text](image-63.png)
![alt text](image-64.png)

#### Comment vérifier qu’une règle bloque réellement le trafic ?

![alt text](image-65.png)
- Si une règle est appliquée, elle est cochée et n'est pas égale à 0/0 MB. Si c'est le cas, cela signifie que la règle n'est pas atteinte et donc le site n'est pas bloqué

# Partie 6 – Aller plus loin (partie plus tendue)

## 10. Blocage par catégorie (réseaux sociaux)

### 10.1 Régle de blocage

![alt text](image-66.png)



### Question

#### Que se passe-t-il si la règle est placée sous une règle "Pass Any" ?

- La règle de blocage serait complètement ignorée et donc n'aurait aucun effet

## 11. Règles horaires

### 11.1 Créer un horaire

![alt text](image-67.png)

![alt text](image-68.png)

### 11.2 Question

#### Pourquoi les règles horaires sont-elles utiles en entreprise ?

- Car elles peuvent empêcher les employés de procrastiner pendant les horaires de travail

## 12. Serveur web local

### 12.1 Serveur web sur Unbuntu

![alt text](image-70.png)

![alt text](image-69.png)

![alt text](image-71.png)
- Ici rajouter les IP des machines autorisées. Ensuite, faire un curl http://192.168.170.100 sur une machine. Si l'adresse de la machine est dans la règle, alors elle peut y accéder, sinon l'accès est refusé

### 12.2 Questions

#### Filtrer par IP source ?

- Elle permet de limiter l'accès de façon précise

#### Filtrer par port ?

- Par port ce sera les services

#### Pourquoi le pare-feu protège-t-il le LAN même en réseau interne ?

- Car les principaux dangers peuvent venir de l'intérieur via le wi-fi ou par branchement entre deux machines

## 13. Logs et analyse

### 13.1 Activer la journalisation

![alt text](image-72.png)

### 13.2 Questions

#### Différence entre paquet bloqué et autorisé

- Un paquet autorisé nommé Pass traverse le pare-feu et atteint sa destination
- Un paquet bloqué nommé Block, le paquet est abandonné par pfsense et n'atteint jamais sa destination

#### Identifier quelle règle a déclenché le blocage

- Dans les logs, on peut voir à droite le nom donné à la règle qui provoque le blocage, ou alors par son numéro dans l'ordre de création de la règle (s'il s'agit de la première règle créée, alors son numéro est 1)

#### Comprendre le sens du trafic

- Traffic sortant LAN --> WAN
- Traffic entrant WAN --> LAN
- Traffic interne LAN --> LAN

## 14. DMZ

### 14.1 Mise en place DMZ

### 14.2 Questions

#### Qu'est ce qu'une DMZ ?

- 

#### Pourquoi isoler un serveur ?

- 

#### Une machine en DMZ peut-elle accéder au LAN ?

- 

#### Le LAN peut-il accéder librement à la DMZ ?

## 15. Filtrage MAC

### 15.1 Filtrage MAC

### 15.2 Questions

#### Le filtrage MAC est-il réellement sécurisé ?

- 

#### Pourquoi est-il facilement contournable ?

- 

## 16. Portail captif

### 16.1 Implémenter portail captif

### 16.2 Questions

#### Dans quels contextes utilise-t-on cela ?

- 

#### Quelle(s) avantage(s) avec une simple règle de pare-feu ?

- 

## 17. Sauvegarde / restauration

### 17.1 

### 17.2 Question

#### Pourquoi la sauvegarde régulière est-elle essentielle en production ?

- 