# Comment tester le jeu — Guide détaillé

Ce guide décrit **tout** ce qu’il faut pour tester correctement le projet **R‑TYPE** : prérequis matériels/logiciels, installation, lancement, scénarios de test, critères d’acceptation, réseau, performance et dépannage.

---

## 🖥️ Configuration minimale & recommandée

> **Important : pas de Core i3 pour les tests officiels** (risque de saccades et latences réseau).

### Minimum (acceptable pour test basique local)
- **CPU** : Intel Core i5 (4 cœurs) / AMD Ryzen 5 équivalent
- **RAM** : 8 Go
- **GPU** : Intégré OK (Intel UHD) ou carte dédiée d’entrée de gamme
- **Stockage** : 1 Go libre
- **Réseau** : Ethernet ou Wi‑Fi stable (5 GHz recommandé)

### Recommandé (multijoueur fluide et captures)
- **CPU** : Intel Core i7 / AMD Ryzen 7 (≥ 6 cœurs)
- **RAM** : 16 Go
- **GPU** : NVIDIA/AMD dédiée (≥ 2 Go VRAM)
- **Écran** : 1920×1080
- **Réseau** : Ethernet filaire (latence < 20 ms sur LAN)

### Systèmes d’exploitation supportés
- **Windows 10/11** (MSVC, MinGW possible)
- **Linux** (Ubuntu 22.04+ / Debian 12 / Arch) — *gcc 11+ ou clang 14+*
- **macOS** (Apple Silicon/Intel) — *Clang*, SFML via Homebrew

> Conseil : éviter les VM pour les tests réseau temps réel (latence et horloges instables).

---

## 📦 Dépendances & outils

- **CMake ≥ 3.15**
- **Compilateur C++20** : gcc 11+/clang 14+/MSVC 19.3+
- **SFML 2.5+** (2.6.x recommandé)
- Gestionnaire de paquets (optionnel) : **vcpkg** ou **Conan**
- **Git** pour récupérer le dépôt

### Installation rapide des libs
- **Ubuntu/Debian**
  ```bash
  sudo apt update
  sudo apt install -y build-essential cmake git libsfml-dev
  ```
- **Arch**
  ```bash
  sudo pacman -S --needed base-devel cmake git sfml
  ```
- **macOS (Homebrew)**
  ```bash
  brew install cmake git sfml
  ```
- **Windows (vcpkg)**
  ```powershell
  git clone https://github.com/microsoft/vcpkg.git
  .cpkgootstrap-vcpkg.bat
  .cpkgcpkg install sfml
  ```

---

## 🧪 Étapes de test (build & run)

### 1) Cloner le dépôt
```bash
git clone <repo-link>
cd r-type
```

### 2) Générer & compiler (mode Release conseillé)
```bash
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . -j
```

> Sur Windows avec vcpkg :
> ```powershell
> cmake -B build -S . -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=C:\path	ocpkg.cmake
> cmake --build build --config Release -j
> ```

### 3) Lancer le serveur
```bash
./r-type_server [port]
# Exemple
./r-type_server 4242
```
- Le serveur écoute par défaut en **UDP** sur le port choisi.  
- Ouvrir le port dans le firewall si nécessaire.

### 4) Lancer un ou plusieurs clients
```bash
# Même machine (localhost) :
./r-type_client 127.0.0.1 4242

# Client distant (réseau local) :
./r-type_client <IP_SERVEUR_LAN> 4242
```
> Plusieurs clients peuvent être lancés sur **une même machine** (fenêtres multiples) ou sur des machines différentes.

### 5) Contrôles par défaut
- **Flèches** : déplacement
- **Espace** : tirer
- **Échap** : quitter / retour menu

---

## 🌐 Notes réseau (LAN/Internet)

- **LAN** : privilégier Ethernet pour réduire la latence et éviter le *packet loss*.
- **Wi‑Fi** : 5 GHz recommandé ; éviter le 2.4 GHz saturé.
- **NAT/Internet** :
  - Faire un **port forwarding UDP** (ex. 4242) sur la box du serveur.
  - Vérifier l’IP publique (serveur) et les règles du pare‑feu.
- **Tests croisés** : tester 2 clients sur 2 PC différents + 1 client sur la machine serveur.

---

## ✅ Scénarios de test & critères d’acceptation

### S1 — Connexion multijoueur (base)
1. Lancer le serveur (4242).
2. Lancer 2 clients (localhost et/ou LAN).
3. **Attendu** : les deux joueurs rejoignent la partie, IDs distincts, positions initiales correctes.

### S2 — Déplacement & tirs
1. Chaque joueur se déplace et tire.
2. **Attendu** : affichage fluide, tirs visibles par tous, pas de désynchro.

### S3 — Collisions & ennemis
1. Faire apparaître des ennemis, tirer jusqu’à destruction.
2. **Attendu** : collisions détectées, score/état mis à jour, cohérent côté clients.

### S4 — Déconnexion client
1. Fermer brutalement un client.
2. **Attendu** : le serveur détecte la déconnexion, l’état global reste stable, les autres clients continuent sans freeze.

### S5 — Stabilité (stress court)
1. 4 clients pendant 10 minutes (mouvements & tirs continus).
2. **Attendu** : pas de crash, pas de fuite mémoire visible, latence stable (< 60 ms en LAN), CPU < 60% sur machine recommandée.

---

## 📊 Performance & qualité (checklist rapide)

- [ ] FPS stable (objectif 60 en local, 30 mini).
- [ ] Latence faible en LAN (< 60 ms).
- [ ] Aucun freeze lors des pics de projectiles.
- [ ] Pas de fuite mémoire évidente (observer l’usage RAM sur 10–15 min).
- [ ] Logs réseau/jeu lisibles en niveau **INFO/DEBUG**.

---

## 🛠️ Dépannage (erreurs fréquentes)

### Le client ne rejoint pas la partie
- Vérifier IP/port.
- Firewall Windows/macOS/Linux → autoriser l’exécutable en **UDP**.
- Port déjà utilisé → changer de port (ex. 4243).

### Erreur SFML non trouvée
- Sous Linux : `sudo apt install libsfml-dev`.
- Sous Windows (vcpkg) : `vcpkg install sfml` puis ajoute `-DCMAKE_TOOLCHAIN_FILE=...vcpkg.cmake`.
- Sous macOS : `brew install sfml`.

### Saccades / latence élevée
- Éviter Wi‑Fi 2.4 GHz ; passer en 5 GHz / Ethernet.
- Fermer les applications lourdes (Chrome 50 onglets…).
- **Interdit pour demo** : machines **Core i3** ou CPU 2 cœurs.

### Builds qui échouent
- Mettre à jour CMake et le compilateur (**C++20** requis).
- Supprimer/recréer le dossier `build/`.
- Regarder la première erreur dans la sortie CMake (dépendance manquante).

---

## 🧾 Journal de test (template)

Copier/coller ce tableau dans votre rapport :

| Scénario | Date | Environnement | Résultat | Remarques |
|---|---|---|---|---|
| S1 Connexion | 2025‑10‑30 | 2 clients LAN | ✅ | IDs OK |
| S2 Déplacements & tirs | 2025‑10‑30 | 2 clients | ✅ | Fluide |
| S3 Collisions | 2025‑10‑30 | 3 clients | ⚠️ | Lag ponctuel |
| S4 Déconnexion | 2025‑10‑30 | 2 clients | ✅ | RAS |
| S5 Stabilité 10 min | 2025‑10‑30 | 4 clients | ✅ | 55 FPS moyen |

---

## 📎 Annexe — Commandes utiles

```bash
# Build (Release)
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . -j

# Lancer serveur
./r-type_server 4242

# Lancer client (localhost)
./r-type_client 127.0.0.1 4242

# Lancer client (LAN)
./r-type_client 192.168.1.50 4242
```

---

Bon test ! Si vous avez besoin d’un **script de lancement** (serveur + N clients) ou d’un **fichier .bat/.sh** prêt à l’emploi, indiquez votre OS et je le fournis.