# Projet R-Type (Serveur & Client Core)

Ce projet est une implémentation du jeu **R-Type** en **C++17**, utilisant **SFML** pour l’affichage, l’audio et le réseau, ainsi qu’un système personnalisé de communication **client/serveur basé sur UDP**.

---

## 📂 Structure du projet

- **main.cpp**  
  Point d’entrée du serveur. Initialise le moteur et lance la boucle principale du jeu.

- **Server.hpp**  
  Définit la logique du serveur de jeu (connexions des joueurs, gestion des salles, traitement des messages).

- **ClientManager.hpp**  
  Gère les clients connectés, leurs états et leur communication avec le serveur.

- **GameEngine.hpp**  
  Moteur principal qui gère la boucle de jeu, les mises à jour et les interactions entre entités.

- **UdpSocket.hpp**  
  Encapsulation de la communication UDP (basée sur SFML).

- **Error.hpp**  
  Gestion centralisée des erreurs et exceptions.

- **CMakeLists.txt**  
  Configuration de compilation avec CMake. Recherche SFML (via Conan si configuré) et lie les bibliothèques nécessaires.

---

## ⚙️ Dépendances

- **C++17 ou plus récent**
- **SFML 2.5+** (graphics, window, system, audio, network)
- **CMake 3.15+**
- (Optionnel) **Conan** pour la gestion des dépendances

---

## 🔨 Instructions de compilation

```bash
# Cloner le projet
git clone <repo_url>
cd r_type_project

# Créer un dossier de build
mkdir build && cd build

# Configurer avec CMake
cmake ..

# Compiler
cmake --build .
```

Cela génère l’exécutable `r_type_server`.

---

## 🚀 Exécution

Lancer le serveur :

```bash
./r_type_server
```

Par défaut, il écoute les connexions UDP entrantes et gère les salles de jeu.

---

## 🧩 Protocole (Vue d’ensemble)

Le serveur et les clients communiquent via des **MessageType (enum)** définis dans le code.  
Exemples :

- `CONNECT`, `DISCONNECT`, `JOIN_ROOM`, `INPUT` (client → serveur)  
- `OK`, `ROOM_CREATED`, `GAME_START`, `UPDATE`, `PLAYER_CONNECTED` (serveur → client)

Cela permet de synchroniser l’état du jeu entre tous les joueurs.

---

## 👩‍💻 Notes pour développeurs

- Voir `Server.hpp` et `ClientManager.hpp` pour la gestion réseau et des clients.  
- `GameEngine.hpp` contient la logique de boucle principale et doit être étendu pour de nouvelles fonctionnalités de gameplay.  
- `UdpSocket.hpp` encapsule le code bas-niveau UDP de SFML.  
- Utiliser `Error.hpp` pour assurer une gestion homogène des erreurs.  

---

## 📌 Roadmap

- Implémenter un vrai ECS (Entity Component System) pour le gameplay.  
- Ajouter des fonctionnalités (armes, ennemis, scoring).  
- Améliorer la gestion des erreurs et le logging.  
