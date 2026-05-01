# LAB 1 — Mise en place du lab Mobile Pentesting

## Objectif

Mettre en place un environnement de test complet pour l'analyse d'applications Android, composé de :
- **Mobexler** : VM Linux préconfigurée avec les outils de pentesting mobile
- **Genymotion** : émulateur Android rapide et léger
- **ADB** : pont de communication entre la machine hôte et le device Android

---

## Étape 1 — Téléchargement de Mobexler

Télécharger l'image OVA depuis le lien officiel :

```
https://drive.google.com/file/d/1rd8g3bmK_XMTtb6PlcfIwjyoJ-mEhAk5/view?usp=sharing
```

### Vérification de l'intégrité

**Windows (PowerShell) :**
```powershell
Get-FileHash .\Mobexler.ova -Algorithm SHA256
```

**Linux / macOS :**
```bash
sha256sum Mobexler.ova
```

Comparer le hash obtenu avec celui fourni sur la page officielle.

---

## Étape 2 — Import de l'OVA dans VirtualBox

1. Ouvrir VirtualBox → **File → Import Appliance**
2. Sélectionner le fichier `.ova` → **Import**
3. Une fois importée, aller dans **VM → Settings → Network** et configurer :

| Adaptateur | Mode |
|---|---|
| Adapter 1 | NAT |
| Adapter 2 | Host-Only Adapter |

> **NAT** : fournit un accès Internet stable à la VM (mises à jour, outils).  
> **Host-Only** : réseau isolé pour communiquer avec la cible Android.

Si l'adaptateur Host-Only n'apparaît pas, le créer via :  
**VirtualBox → Tools → Network Manager → Host-Only Networks → Create**

---

## Étape 3 — Premier démarrage et connexion

1. Démarrer la VM Mobexler
2. Se connecter avec les identifiants par défaut :

```
username : mobexler
password : mobexler
```

> Si l'écran de login affiche un utilisateur différent, utiliser celui proposé ou consulter le README officiel du projet Mobexler.

---

## Étape 4 — Vérification du réseau

Ouvrir un terminal dans Mobexler et exécuter les commandes suivantes.

### Vérifier les interfaces réseau
```bash
ip a
```
Identifier :
- Interface NAT (`10.x.x.x` ou `192.168.x.x`)
- Interface Host-Only (`192.168.56.x`)

### Vérifier la route par défaut
```bash
ip route
```

### Tester la connectivité Internet
```bash
ping -c 2 8.8.8.8
ping -c 2 google.com
```

| Résultat | Cause probable |
|---|---|
| Ping IP OK mais DNS KO | Problème de configuration DNS (`/etc/resolv.conf`) |
| Pas de route par défaut | NAT mal configuré dans VirtualBox |

---

## Étape 5 — Snapshot propre

Avant toute manipulation, prendre un snapshot de la VM dans son état propre.

**VirtualBox :** VM → Snapshots → Take Snapshot

Nommer le snapshot de façon explicite, par exemple : `clean-install`.

> Le snapshot permet de restaurer l'environnement à son état initial après chaque TP (proxy, certificats, modifications système, etc.).

---

## Étape 6 — Connexion de la cible Android (Genymotion)

### B1 — Démarrer le device Genymotion

Lancer Genymotion sur la machine hôte et démarrer le device souhaité (ex: Google Pixel 3).

L'IP du device s'affiche en haut de la fenêtre Genymotion, généralement :
```
192.168.56.101
```

### B2 — Connecter ADB

```powershell
adb connect 192.168.56.101:5555
adb devices
```

Résultat attendu :
```
List of devices attached
192.168.56.101:5555     device
```

### Accéder au shell Android

```powershell
adb -s 192.168.56.101:5555 shell
```

Invite de commande attendue :
```
vbox86p:/ $
```

> Si plusieurs devices sont connectés simultanément, toujours préciser le device cible avec le flag `-s` pour éviter les conflits.

---

## Récapitulatif des points de contrôle

| Étape | Point de contrôle |
|---|---|
| 1 | Fichier `.ova` présent, taille cohérente, hash vérifié |
| 2 | 2 cartes réseau configurées : NAT + Host-Only |
| 3 | Accès au bureau ou terminal Mobexler |
| 4 | Internet fonctionnel (ping IP + ping DNS) |
| 5 | Snapshot visible dans VirtualBox |
| 6 | Device Android visible dans `adb devices` |

---

## Environnement final

| Composant | Rôle |
|---|---|
| Mobexler (VM VirtualBox) | Machine de pentesting avec outils préconfigurés |
| Genymotion | Émulateur Android cible |
| ADB | Communication hôte ↔ device Android |
| NAT | Accès Internet de la VM |
| Host-Only | Réseau isolé VM ↔ émulateur |
