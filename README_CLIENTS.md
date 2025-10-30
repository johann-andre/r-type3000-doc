# R-Type — Game Client

Projet académique visant à recréer une version multijoueur simplifiée du jeu **R-Type** en C++, en utilisant une architecture **ECS (Entity-Component-System)** et un protocole réseau **UDP**.

Ce README fournit toutes les informations nécessaires pour comprendre, compiler et exécuter le **client** du projet, même pour un développeur qui ne connaît pas le code à l’avance.

---

## 📂 Structure du client

### 🎮 Core du moteur (ECS)
- **GameEngine.hpp** : cœur du moteur de jeu. Il initialise les systèmes, met à jour les entités (joueurs, ennemis, projectiles) et gère la logique de la boucle de jeu.
- **GraphisManager.hpp** : responsable du rendu graphique. Il charge les sprites (joueur, ennemis, arrière-plan) et affiche à l’écran l’état du jeu fourni par l’ECS.

### 🌐 Réseau
- **Network.hpp** : définit les structures de données utilisées pour la communication réseau (messages, enums, gestion des paquets).
- **UDPclient.hpp** : encapsule la communication UDP côté client. Il gère l’envoi des inputs (actions joueur) et la réception des snapshots envoyés par le serveur.

### 🏁 Entrée principale
- **main.cpp** : point d’entrée du client. Il :
  1. Crée une fenêtre graphique.
  2. Connecte le client au serveur.
  3. Lance la boucle de jeu (lecture des inputs, envoi réseau, mise à jour ECS, rendu).

---

## ⚙️ Compilation

Le projet utilise **CMake** et **Conan** pour gérer ses dépendances.

### Étapes :

```bash
# 1. Cloner le projet
git clone https://github.com/mon-compte/r-type.git
cd r-type

# 2. Installer les dépendances avec Conan
conan install . --build=missing

# 3. Compiler avec CMake
mkdir build && cd build
cmake ..
make -j
```

Le binaire `r-type_client` sera généré dans le dossier `build`.

---

## ▶️ Lancer le client

Exécuter le client en lui donnant l’adresse IP et le port du serveur :

```bash
./r-type_client [ip] [port]
```

Exemple :

```bash
./r-type_client 127.0.0.1 4242
```

---

## 🕹️ Contrôles par défaut

- **Flèches directionnelles** : déplacer le vaisseau
- **Espace** : tirer
- **Échap** : quitter le jeu

---

## 📡 Fonctionnement réseau (côté client)

1. **Connexion** : le client envoie un message `CONNECT` au serveur et attend un `OK` avec un `client_id` assigné.
2. **Lobby** : le joueur peut créer ou rejoindre une room (messages `ROOM_NAME` / `JOIN_ROOM`).
3. **Prêt** : le joueur envoie `READY`. Quand tous les joueurs sont prêts, le serveur envoie `GAME_START` avec un `start_tick` synchronisé.
4. **In-Game** :
   - Le client envoie régulièrement ses **inputs** (`INPUT`) au serveur (20–60 fois/seconde).
   - Le serveur renvoie des **snapshots** (`UPDATE`, `MOVE`, `ENEMY_SPAWNED`, etc.), que le client applique à son ECS.
   - Le rendu est mis à jour à partir de l’état ECS.

5. **Quitter** : `QUIT_GAME` ou fermeture de la fenêtre envoie un `DISCONNECT`.

---

## 🔄 Cycle de la boucle de jeu (client)

1. **Lecture des entrées clavier** → transformation en `input_bits`.
2. **Envoi des inputs** au serveur via `UDPclient`.
3. **Réception des snapshots** du serveur → mise à jour des entités ECS.
4. **Mise à jour du moteur ECS** (collisions, logiques locales).
5. **Rendu graphique** avec `GraphisManager`.

---

## 📡 Protocole réseau (résumé)

- Transport : **UDP** (binaire).
- Client → Serveur :
  - `CONNECT`, `DISCONNECT`, `ROOM_NAME`, `JOIN_ROOM`, `READY`, `INPUT`, `QUIT_GAME`
- Serveur → Client :
  - `OK`, `REFUSED`, `ROOM_CREATED`, `ROOM_JOINED`, `GAME_START`
  - `UPDATE`, `MOVE`, `COLLISION`, `SHOOT`, `ENEMY_SPAWNED`
  - `PLAYER_CONNECTED`, `PLAYER_DISCONNECTED`

📖 Pour plus de détails, voir [`PROTOCOL.md`](./PROTOCOL.md).

---

## 👥 Équipe (à compléter)

- Développement ECS : Darren HOUNSA, Hilary ZOCLI
- Gestion Graphique :Wakil KARIMOU, Darren HOUNSA
- Réseau Client : Johann-André MAFORIKAN, Emeric AMEGAH
- Réseau Serveur : Emeric AMEGAH

---

## 📜 Licence

Projet académique — utilisation libre à des fins pédagogiques.
