# TP3 - Cryptographie avec OpenSSL

**Étudiant :** [Votre Nom]  
**Date :** 13 février 2026  
**Système :** Windows avec OpenSSL 3.6.1

---

## A. Base64

### 1. Génération d'un fichier binaire

**Commande :**
```bash
openssl rand -out data.bin 102400
```

**Vérification de la taille :**
```bash
dir data.bin
```

**Résultat :** Fichier de 102 400 octets (100 Ko) créé avec succès.

---

### 2. Encodage

**Commande :**
```bash
openssl base64 -e -in data.bin -out data.b64
```

**Affichage du contenu :**
```bash
type data.b64 | more
```

**Comparaison des tailles :**
```bash
dir data.bin data.b64
```

**Résultat :**
- `data.bin` : 102 400 octets
- `data.b64` : 136 536 octets
- Augmentation : environ 33,33%

---

### 3. Décodage

**Commande :**
```bash
openssl base64 -d -in data.b64 -out data_restored.bin
```

**Vérification de l'identité :**
```bash
fc /b data.bin data_restored.bin
```

**Résultat :** Les fichiers sont strictement identiques (aucune différence).

**Vérification par hash SHA-256 :**
```bash
openssl dgst -sha256 data.bin
openssl dgst -sha256 data_restored.bin
```

---

### 4. Réponses aux questions

**1. Base64 est-il un chiffrement ? Pourquoi ?**

Non, Base64 n'est pas un chiffrement mais un encodage. La différence fondamentale :
- **Encodage** : transformation réversible sans clé secrète, accessible à tous
- **Chiffrement** : transformation nécessitant une clé secrète pour le décodage

Base64 transforme des données binaires en caractères ASCII imprimables pour faciliter leur transmission via des protocoles texte, mais n'offre aucune sécurité.

**2. Pourquoi la taille du fichier change-t-elle après encodage ?**

Base64 utilise 64 caractères (A-Z, a-z, 0-9, +, /) pour représenter des données binaires qui peuvent avoir 256 valeurs possibles (0-255). Cette limitation impose une expansion :
- 3 octets binaires (24 bits) → 4 caractères Base64 (24 bits)
- Ratio : 4/3

**3. Quel est approximativement le pourcentage d'augmentation ?**

Le pourcentage d'augmentation est de **33,33%** (4/3 ≈ 1,333).

Calcul : (136 536 - 102 400) / 102 400 × 100 = 33,33%

**4. Quelle méthode permet de vérifier rigoureusement que deux fichiers sont identiques ?**

Trois méthodes principales :
- **Comparaison binaire** : `fc /b fichier1 fichier2` (Windows) ou `diff` (Linux)
- **Hash cryptographique** : Comparer les empreintes SHA-256 des deux fichiers
- **Checksum** : Utiliser MD5 ou SHA-1 (moins sûr que SHA-256)

La méthode la plus fiable est la comparaison des hash SHA-256, car elle garantit mathématiquement l'identité des fichiers.

---

## B. Chiffrement symétrique - AES

### 1. Création d'un message

**Commande :**
```bash
echo Nom: Baptiste > confidentiel.txt
echo Date: 13/02/2026 >> confidentiel.txt
echo Ligne 1: Message confidentiel pour TP cryptographie >> confidentiel.txt
echo Ligne 2: Demonstration du chiffrement symetrique AES-256 >> confidentiel.txt
echo Ligne 3: Utilisation d'OpenSSL version 3.6.1 >> confidentiel.txt
echo Ligne 4: Protocole de securisation des donnees >> confidentiel.txt
echo Ligne 5: Fin du document confidentiel >> confidentiel.txt
```

**Vérification :**
```bash
type confidentiel.txt
```

---

### 2. Chiffrement

**Commande :**
```bash
openssl enc -e -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel.enc
```

**Mot de passe utilisé :** [votre mot de passe]

**Vérification du caractère binaire :**
```bash
type confidentiel.enc
```

**Résultat :** Le fichier affiche des caractères illisibles (données binaires).

---

### 3. Déchiffrement

**Commande :**
```bash
openssl enc -d -aes-256-cbc -pbkdf2 -md sha256 -in confidentiel.enc -out confidentiel_dechiffre.txt
```

**Vérification :**
```bash
type confidentiel_dechiffre.txt
fc confidentiel.txt confidentiel_dechiffre.txt
```

**Résultat :** Les fichiers sont identiques, le déchiffrement a réussi.

---

### 4. Analyse

**Deuxième chiffrement :**
```bash
openssl enc -e -aes-256-cbc -salt -pbkdf2 -md sha256 -in confidentiel.txt -out confidentiel2.enc
```

**Comparaison :**
```bash
fc /b confidentiel.enc confidentiel2.enc
```

**Résultat :** Les deux fichiers chiffrés sont différents malgré :
- Le même fichier source
- Le même mot de passe
- Les mêmes options de chiffrement

---

### 5. Réponses aux questions

**1. Pourquoi les deux fichiers chiffrés sont-ils différents ?**

Le sel (salt) est généré aléatoirement à chaque chiffrement. Même avec le même mot de passe et le même fichier source, le sel différent produit :
- Une clé de chiffrement différente (via PBKDF2)
- Un vecteur d'initialisation différent
- Un cryptogramme totalement différent

**2. Quel est le rôle du sel ?**

Le sel a trois rôles principaux :
- **Protection contre les rainbow tables** : Les tables précalculées de hash deviennent inutiles
- **Unicité des chiffrements** : Deux chiffrements identiques produisent des résultats différents
- **Protection contre les attaques parallèles** : Chaque fichier doit être attaqué individuellement

**3. Que se passe-t-il si une option change lors du déchiffrement ?**

Le déchiffrement échoue ou produit des données corrompues. Toutes les options doivent correspondre :
- Algorithme de chiffrement (`-aes-256-cbc`)
- Méthode de dérivation de clé (`-pbkdf2`)
- Fonction de hachage (`-md sha256`)

**4. Pourquoi utilise-t-on PBKDF2 ?**

PBKDF2 (Password-Based Key Derivation Function 2) :
- Dérive une clé cryptographique robuste à partir d'un mot de passe faible
- Applique de nombreuses itérations (par défaut 10 000+) pour ralentir les attaques
- Augmente considérablement le temps nécessaire pour une attaque par force brute
- Transforme un mot de passe simple en clé de 256 bits utilisable par AES

**5. Quelle est la différence entre encodage et chiffrement ?**

| Critère | Encodage | Chiffrement |
|---------|----------|-------------|
| **Objectif** | Transformation de format | Protection des données |
| **Clé secrète** | Non requise | Obligatoire |
| **Réversibilité** | Publique (tout le monde peut décoder) | Nécessite la clé |
| **Sécurité** | Aucune | Forte (si bien implémenté) |
| **Exemples** | Base64, UTF-8, URL encoding | AES, RSA, ChaCha20 |

---

## C. Cryptographie asymétrique - RSA

### 1. Génération de clés

**Génération de la clé privée (protégée par mot de passe) :**
```bash
openssl genrsa -aes256 -out rsa_private.pem 2048
```

**Mot de passe de protection :** [votre mot de passe]

**Extraction de la clé publique :**
```bash
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```

**Affichage des paramètres détaillés de la clé privée :**
```bash
openssl rsa -in rsa_private.pem -text -noout
```

**Paramètres observés :**
- **Modulus (n)** : 2048 bits
- **Public Exponent (e)** : 65537 (0x10001)
- **Private Exponent (d)** : 2048 bits
- **Prime1 (p)** et **Prime2 (q)** : facteurs premiers du modulus
- **Exponent1, Exponent2, Coefficient** : valeurs précalculées pour optimisation

**Affichage des paramètres de la clé publique :**
```bash
openssl rsa -in rsa_public.pem -pubin -text -noout
```

**Paramètres observés :**
- **Modulus (n)** : identique à la clé privée
- **Public Exponent (e)** : 65537

**Comparaison :**

La clé publique contient uniquement :
- Le modulus (n)
- L'exposant public (e)

La clé privée contient en plus :
- L'exposant privé (d)
- Les facteurs premiers (p et q)
- Les valeurs d'optimisation

---

### 2. Chiffrement asymétrique

**Création du fichier secret :**
```bash
echo Ceci est mon message ultra confidentiel pour le TP3 > secret.txt
```

**Chiffrement avec la clé publique :**
```bash
openssl pkeyutl -encrypt -in secret.txt -pubin -inkey rsa_public.pem -out secret.enc
```

**Vérification du fichier chiffré :**
```bash
type secret.enc
```

**Résultat :** Données binaires illisibles.

**Déchiffrement avec la clé privée :**
```bash
openssl pkeyutl -decrypt -in secret.enc -inkey rsa_private.pem -out secret_dechiffre.txt
```

**Vérification :**
```bash
type secret_dechiffre.txt
fc secret.txt secret_dechiffre.txt
```

**Résultat :** Les fichiers sont identiques.

---

### 3. Réponses aux questions

**1. Pourquoi la clé privée ne doit-elle jamais être partagée ?**

La clé privée permet de :
- **Déchiffrer** tous les messages chiffrés avec la clé publique correspondante
- **Signer numériquement** des documents en votre nom
- **Usurper votre identité** dans les systèmes d'authentification

Si la clé privée est compromise :
- Toute la sécurité du système est perdue
- L'attaquant peut se faire passer pour vous
- Les messages passés et futurs peuvent être déchiffrés

**2. Pourquoi RSA n'est-il pas adapté au chiffrement de gros fichiers ?**

Trois raisons principales :
- **Performance** : RSA est 100 à 1000 fois plus lent que AES en raison des calculs mathématiques complexes sur de grands nombres
- **Limitation de taille** : RSA ne peut chiffrer que des données de taille inférieure à la clé moins le padding (RSA 2048 : max ~245 octets)
- **Expansion des données** : Le cryptogramme fait toujours la taille de la clé (256 octets pour RSA 2048), même pour 1 octet de données

**3. Quelles différences observe-t-on entre les paramètres d'une clé publique et d'une clé privée ?**

**Clé publique (partageable) :**
- Modulus (n) : produit de deux grands nombres premiers
- Exposant public (e) : généralement 65537

**Clé privée (secrète) :**
- Tous les paramètres de la clé publique
- Exposant privé (d) : inverse de e modulo φ(n)
- Prime1 (p) et Prime2 (q) : facteurs premiers de n
- Exponent1, Exponent2, Coefficient : valeurs précalculées pour le théorème des restes chinois (optimisation)

**4. Quel est le rôle du modulus dans RSA ?**

Le modulus (n) :
- Définit l'espace mathématique des opérations RSA
- Est le produit de deux grands nombres premiers (n = p × q)
- Détermine la taille de la clé (2048 bits → n est un nombre de 617 chiffres décimaux)
- Sa factorisation est le fondement de la sécurité RSA : si on peut factoriser n en p et q, on peut calculer la clé privée

La sécurité repose sur le fait qu'il est computationnellement impossible de factoriser de très grands nombres en temps raisonnable.

**5. Pourquoi utilise-t-on souvent RSA pour chiffrer une clé AES plutôt qu'un document entier ?**

Cette approche appelée **chiffrement hybride** combine les avantages des deux :

**RSA** :
- Utilisé uniquement pour chiffrer une clé AES aléatoire (256 bits = 32 octets)
- Rapide pour cette petite quantité de données
- Permet l'échange sécurisé de la clé symétrique

**AES** :
- Utilisé pour chiffrer le document complet
- Très rapide (débit de plusieurs Go/s)
- Pas de limitation de taille

**Avantages** :
- Sécurité de RSA pour l'échange de clé
- Performance de AES pour les données volumineuses
- Solution utilisée dans SSL/TLS, PGP, etc.

---

## D. Signature numérique

### 1. Création et signature

**Création du fichier :**
```bash
echo Contrat de prestation entre parties prenantes > contrat.txt
echo Article 1: Objet du contrat >> contrat.txt
echo Article 2: Duree du contrat >> contrat.txt
echo Article 3: Modalites de paiement >> contrat.txt
```

**Génération de l'empreinte (hash) :**
```bash
openssl dgst -sha256 contrat.txt
```

**Résultat :** 
```
SHA256(contrat.txt)= [hash hexadécimal de 64 caractères]
```

**Signature du fichier avec la clé privée :**
```bash
openssl dgst -sha256 -sign rsa_private.pem -out contrat.sig contrat.txt
```

**Vérification de la création du fichier de signature :**
```bash
dir contrat.sig
```

---

### 2. Vérification

**Vérification de la signature avec la clé publique :**
```bash
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

**Résultat :** `Verified OK`

**Modification du fichier :**
```bash
echo Article 4: Clause additionnelle >> contrat.txt
```

**Nouvelle vérification :**
```bash
openssl dgst -sha256 -verify rsa_public.pem -signature contrat.sig contrat.txt
```

**Résultat :** `Verification Failure`

---

### 3. Réponses aux questions

**1. Que se passe-t-il après modification du fichier ?**

La vérification de signature échoue avec le message "Verification Failure". La signature n'est plus valide pour le fichier modifié.

**2. Pourquoi ?**

Le processus de signature fonctionne ainsi :
1. Calcul du hash du fichier original
2. Chiffrement du hash avec la clé privée → signature
3. Lors de la vérification :
   - Calcul du hash du fichier actuel
   - Déchiffrement de la signature avec la clé publique
   - Comparaison des deux hash

Après modification, le hash du fichier change, donc la comparaison échoue.

**3. Quel est le rôle du hachage dans le mécanisme de signature ?**

Le hachage a plusieurs rôles essentiels :
- **Intégrité** : Toute modification du fichier change complètement le hash
- **Taille fixe** : Un document de 1 Mo ou 1 Go produit toujours un hash de 256 bits (SHA-256)
- **Performance** : Signer un hash de 32 octets est bien plus rapide que signer un document entier
- **Unicité** : Deux documents différents ont statistiquement des hash différents

**4. Quelle différence entre signature numérique et chiffrement ?**

| Critère | Signature numérique | Chiffrement |
|---------|-------------------|-------------|
| **Objectif** | Authentifier et garantir l'intégrité | Assurer la confidentialité |
| **Clé utilisée pour créer** | Clé privée | Clé publique (RSA) ou secrète (AES) |
| **Clé pour vérifier/déchiffrer** | Clé publique | Clé privée (RSA) ou secrète (AES) |
| **Visibilité des données** | Document en clair + signature | Document chiffré illisible |
| **Protection contre** | Falsification, usurpation | Interception, espionnage |
| **Propriété garantie** | Authenticité + Intégrité | Confidentialité |

**Schéma des processus :**

Signature :
```
Document → Hash → Chiffrement (clé privée) → Signature
Vérification : Document → Hash → Comparaison ← Déchiffrement (clé publique) ← Signature
```

Chiffrement :
```
Document → Chiffrement (clé publique) → Cryptogramme
Déchiffrement : Cryptogramme → Déchiffrement (clé privée) → Document
```

---

## Bonus : Chiffrement hybride complet

### Principe du chiffrement hybride

Le chiffrement hybride combine les avantages du chiffrement symétrique (AES) et asymétrique (RSA) :
- **AES** : rapide, pour les données volumineuses
- **RSA** : pour échanger la clé AES de manière sécurisée

### Étape 1 : Génération d'une clé AES aléatoire

```bash
openssl rand -out aes.key 32
```

**Explication :** Génère 32 octets (256 bits) de données aléatoires pour la clé AES-256.

**Vérification :**
```bash
dir aes.key
```

---

### Étape 2 : Chiffrement du fichier volumineux avec AES

**Création d'un fichier de test :**
```bash
openssl rand -out document_volumineux.txt 1048576
```

**Chiffrement avec la clé AES :**
```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -md sha256 -in document_volumineux.txt -out document_volumineux.enc -pass file:aes.key
```

**Paramètres :**
- `-pass file:aes.key` : utilise le contenu du fichier comme mot de passe
- Le fichier est chiffré rapidement grâce à AES

---

### Étape 3 : Chiffrement de la clé AES avec RSA

```bash
openssl pkeyutl -encrypt -in aes.key -pubin -inkey rsa_public.pem -out aes.key.enc
```

**Explication :**
- La clé AES (32 octets) est chiffrée avec RSA
- Seul le détenteur de la clé privée RSA pourra déchiffrer la clé AES
- Le fichier `aes.key.enc` peut être transmis en toute sécurité

---

### Étape 4 : Déchiffrement (processus inverse)

**Déchiffrement de la clé AES avec la clé privée RSA :**
```bash
openssl pkeyutl -decrypt -in aes.key.enc -inkey rsa_private.pem -out aes.key.recovered
```

**Déchiffrement du document avec la clé AES récupérée :**
```bash
openssl enc -d -aes-256-cbc -pbkdf2 -md sha256 -in document_volumineux.enc -out document_volumineux.recovered.txt -pass file:aes.key.recovered
```

**Vérification :**
```bash
fc /b document_volumineux.txt document_volumineux.recovered.txt
```

---

### Explication précise de chaque étape

**1. Génération de la clé AES aléatoire**
- Utilise le générateur de nombres pseudo-aléatoires cryptographiquement sûr d'OpenSSL
- 256 bits = niveau de sécurité maximal pour AES
- Cette clé est unique pour chaque session de chiffrement

**2. Chiffrement du fichier avec AES**
- AES-256-CBC : algorithme symétrique très rapide
- Débit typique : plusieurs Go/s sur un processeur moderne
- Adapté aux fichiers de toute taille
- Le sel rend chaque chiffrement unique

**3. Chiffrement de la clé AES avec RSA**
- RSA 2048 peut chiffrer jusqu'à 245 octets (avec padding)
- La clé AES (32 octets) rentre largement
- Seul le destinataire avec la clé privée peut déchiffrer
- Permet l'échange sécurisé de la clé symétrique

**4. Avantages du chiffrement hybride**
- **Performance** : AES rapide pour les données volumineuses
- **Sécurité** : RSA pour l'échange de clé
- **Scalabilité** : Un même document peut être chiffré pour plusieurs destinataires en chiffrant la clé AES avec la clé publique de chacun
- **Flexibilité** : Utilisé dans tous les protocoles modernes (TLS, PGP, S/MIME)

**5. Applications réelles**
- **HTTPS/TLS** : Échange de clé symétrique via RSA ou ECDHE
- **PGP/GPG** : Chiffrement d'emails
- **VPN** : Établissement de tunnels sécurisés
- **SSH** : Authentification et chiffrement de session

---

## Conclusion

Ce TP a permis d'explorer les fondamentaux de la cryptographie moderne :
- **Encodage Base64** : transformation de données binaires en texte
- **Chiffrement symétrique AES** : rapide et efficace pour les données volumineuses
- **Chiffrement asymétrique RSA** : échange de clés et signatures numériques
- **Signatures numériques** : garantie d'authenticité et d'intégrité
- **Chiffrement hybride** : combinaison optimale des deux approches

Ces techniques sont au cœur de la sécurité informatique moderne et sont utilisées quotidiennement dans les protocoles HTTPS, SSH, VPN, et applications de messagerie sécurisée.
