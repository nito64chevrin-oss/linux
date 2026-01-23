# TP Réseau - Exploration locale en solo

## I. Affichage d'informations sur la pile TCP/IP locale

### Configuration réseau actuelle

| Paramètre | Valeur |
|-----------|--------|
| **Carte réseau** | Wi-Fi |
| **Adresse IP** | `10.33.69.69` |
| **Masque de sous-réseau** | `255.255.240.0` |
| **Adresse MAC** | `E8:BF:B8:64:93:49` |
| **Adresse réseau** | `10.33.64.0/20` |
| **Broadcast** | `10.33.79.255` |

### Comment afficher les informations réseau sous Windows

**Méthode 1 : Interface graphique**
1. Panneau de configuration → Réseau et Internet → Centre Réseau et partage
2. Cliquer sur votre connexion → Détails
3. Ou : Paramètres → Réseau et Internet → WiFi → Propriétés

**Méthode 2 : Ligne de commande**
```powershell
ipconfig /all
```

![Configuration réseau](images/image.png)
![Gateway](image.png)

### 🔍 Rôle de la gateway dans le réseau Ingésup

La **gateway** (passerelle) joue un rôle crucial :
- Elle permet aux machines du réseau Ingésup de communiquer avec Internet
- C'est le routeur qui fait le lien entre le réseau local (LAN) et le réseau externe (WAN/Internet)
- Sans gateway configurée, on ne peut communiquer qu'avec les machines de notre réseau local

---

## II. Modifications des informations réseau

### A. Calcul des adresses disponibles

**Calcul du nombre d'adresses :**
```
Réseau : 10.33.64.0/20
Nombre d'adresses utilisables : 2^(32-20) - 2 = 4094 adresses
```

**Plage d'adresses :**
- **Première adresse utilisable** : `10.33.64.1`
- **Dernière adresse utilisable** : `10.33.79.254`

> ⚠️ **À exclure :** l'adresse réseau (`10.33.64.0`), l'adresse broadcast (`10.33.79.255`) et la gateway

### Configuration manuelle de l'IP sous Windows

1. **Accéder aux paramètres réseau**
   - Paramètres → Réseau et Internet → WiFi → Propriétés du matériel

   ![Propriétés réseau](image-2.png)

2. **Passer en mode manuel**
   - Désactiver le mode automatique (DHCP)
   - Sélectionner IPv4 en mode manuel

   ![Configuration manuelle](image-3.png)

3. **Saisir les informations**
   - Entrer la nouvelle adresse IP (qui ne doit pas être déjà utilisée)
   - Masque de sous-réseau
   - Gateway

---

### B. Scan réseau avec nmap

#### Scan des hôtes actifs
```bash
nmap -sn 10.33.64.0/20
```

![Scan nmap](image-4.png)

**Résultat :** Liste de toutes les adresses IP utilisées et leurs hôtes associés.
> 💡 **Note :** "Unknown" signifie que nmap n'a pas pu identifier le nom d'hôte.

**Exemples d'adresses IP libres détectées :**
- `10.33.73.197`
- `10.33.73.199`
- `10.33.73.205`

#### Liste complète des adresses (avec résolution DNS)
```bash
nmap -sL 10.33.64.0/20
```

![Liste nmap](image-5.png)

---

### C. Modification de l'adresse IP via ligne de commande

#### Changement d'adresse IP
```powershell
netsh interface ip set address "Wi-Fi" static 10.33.73.197 255.255.240.0 10.33.79.254
```

Cette commande configure :
- **Nouvelle IP** : `10.33.73.197`
- **Masque** : `255.255.240.0`
- **Gateway** : `10.33.79.254`

#### Configuration du DNS
```powershell
netsh interface ip set dns "Wi-Fi" static 8.8.8.8
```

Cette commande permet d'avoir accès à Internet en configurant le serveur DNS de Google (`8.8.8.8`).

> 🌐 **Alternatives DNS :**
> - Google : `8.8.8.8` / `8.8.4.4`
> - Cloudflare : `1.1.1.1` / `1.0.0.1`

---

## 📝 Vérification de la configuration

```powershell
# Afficher la nouvelle configuration
ipconfig /all

# Tester la connectivité
ping 8.8.8.8          # Test IP
ping google.com       # Test DNS
```