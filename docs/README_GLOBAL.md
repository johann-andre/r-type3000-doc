# 🚀 Projet R-Type — Implémentation C++ (Client / Serveur / ECS)

Projet académique visant à recréer une version multijoueur simplifiée du jeu **R-Type** en **C++17**, avec une architecture **ECS (Entity-Component-System)** et un protocole réseau **UDP** basé sur **SFML**.

---

## 📂 Arborescence du projet

```
Project R-Type
├── Assets/                  # Ressources graphiques et sons
│   ├── background.jpg
│   ├── enemy.png
│   ├── fire.png
│   ├── font.ttf
│   ├── heart.png
│   ├── Montserrat-VariableFont_wght.ttf
│   ├── player.png
│   ├── player_up.png
│   ├── player_down.png
│   ├── r-typesheet1-ezgif.com-crop.gif
│   ├── r-typesheet7-ezgif.com-crop.gif
│   └── sprites.zip
│
├── build.sh                 # Script de compilation
│
├── clients/                 # Code source client
│   ├── CMakeLists.txt
│   ├── Network.hpp
│   └── UDPclient.hpp
│
├── CMakeLists.txt           # Configuration globale du projet
├── conanfile.txt            # Dépendances (Conan)
│
├── config/                  # Fichiers de configuration du gameplay
│   ├── bullets.conf
│   ├── enemie_1.conf
│   ├── enemie_2.conf
│   ├── enemie_3.conf
│   └── player.conf
│
├── ECS/                     # Core ECS (Entity-Component-System)
│   ├── Component.hpp
│   ├── Entity.hpp
│   ├── GameEngine.hpp
│   ├── GameLogic.cpp
│   ├── GameLogic.hpp
│   ├── Libs.hpp
│   ├── Network.hpp
│   ├── Parsing.cpp
│   ├── Parsing.hpp
│   ├── README_ECS.md
│   ├── Registry.hpp
│   ├── SparseArray.hpp
│   ├── System.cpp
│   └── System.hpp
│
├── main.cpp                 # Entrée principale
│
├── NETWORK/                 # Code serveur et outils réseau
│   ├── ClientManager.hpp
│   ├── Error.hpp
│   ├── main.cpp
│   ├── Server.hpp
│   ├── UdpSocket.cpp
│   └── UdpSocket.hpp
│
├── reseau_rtype/            # Dossier build + binaires
│   ├── build/
│   │   ├── r-type_client
│   │   └── r-type_server
│   ├── client/
│   │   └── main.cpp
│   ├── CMakeLists.txt
│   ├── serialization.cpp
│   └── server2/
│       └── main.cpp
│
└── utils.hpp                # Fonctions utilitaires
```

---

## ⚙️ Dépendances

- **C++17 ou plus récent**
- **SFML 2.5+** (graphics, window, system, audio, network)
- **CMake 3.15+**
- (Optionnel) **Conan** pour la gestion des dépendances

---

## 🔨 Compilation

### Via CMake et Conan

```bash
# Cloner le projet
git clone <repo_url>
cd r-type

# Installer les dépendances avec Conan
conan install . --build=missing

# Créer un dossier build et compiler
mkdir build && cd build
cmake ..
make -j
```

L’exécutable `r-type_server` et `r-type_client` seront générés.

---

## ▶️ Exécution

### Lancer le serveur
```bash
./r-type_server
```

### Lancer un client
```bash
./r-type_client [ip] [port]
```

Exemple :

```bash
./r-type_client 127.0.0.1 4242
```

---

## 🕹️ Contrôles par défaut (client)

- **Flèches directionnelles** : déplacer le vaisseau
- **Espace** : tirer
- **Échap** : quitter le jeu

---

## 🧩 Architecture du projet

### 🎮 ECS (Entity-Component-System)
- **Entity** : identifiant unique d’un objet du jeu (joueur, ennemi, missile).  
- **Component** : données associées (Position, Velocity, Health, Tag, Sprite, etc.).  
- **System** : logique appliquée aux entités (déplacement, collisions, rendu, input).  

📖 Documentation détaillée : [`ECS/README_ECS.md`](./ECS/README_ECS.md)

### 🌐 Réseau
- **Protocol UDP** (binaire) basé sur `MessageType`.  
- **Client → Serveur** : `CONNECT`, `DISCONNECT`, `JOIN_ROOM`, `INPUT`, etc.  
- **Serveur → Client** : `OK`, `ROOM_CREATED`, `GAME_START`, `UPDATE`, `MOVE`, `COLLISION`, etc.  
- La communication est gérée via `UdpSocket` (serveur) et `UDPclient` (client).

### 🖥️ Organisation
- **NETWORK/** : logique serveur (`Server.hpp`, `ClientManager.hpp`, `UdpSocket`).  
- **clients/** : logique client (connexion au serveur, envoi des inputs, réception des snapshots).  
- **ECS/** : cœur du moteur de jeu (systèmes, entités, composants).  
- **config/** : paramètres de gameplay (vitesse, HP, patterns ennemis).  
- **Assets/** : ressources graphiques et sonores.  

---

## 🔄 Cycle de jeu

1. **Client** lit les inputs clavier → envoie `INPUT` au serveur.  
2. **Serveur** reçoit les actions → met à jour l’état global du jeu via l’ECS.  
3. **Serveur** envoie des `UPDATE`/`MOVE` aux clients.  
4. **Clients** appliquent les snapshots → affichent avec SFML.  
5. Boucle continue jusqu’à `QUIT_GAME` / fermeture.

---

## 📌 Roadmap

- Améliorer la synchronisation réseau (lag compensation, interpolation).  
- Ajouter de nouveaux ennemis et patterns.  
- Implémenter un vrai système ECS complet (séparer encore plus données/logique).  
- Système de scoring et d’UI.  

---

## 👥 Équipe (à compléter)

- Développement ECS : Darren HOUNSA, Hilary ZOCLI
- Gestion Graphique :Wakil KARIMOU, Darren HOUNSA
- Réseau Client : Johann-André MAFORIKAN, Emeric AMEGAH
- Réseau Serveur : Emeric AMEGAH
---

## 📜 Licence

Projet académique — utilisation libre à des fins pédagogiques.
