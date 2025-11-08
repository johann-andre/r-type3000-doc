# 🖥️ README_SERVER — Version Développeur Complète (V3)

## 🧠 Description générale

Le **serveur R-Type** est un composant fondamental du projet, développé en **C++17** avec la bibliothèque **SFML 2.5+**. Il gère la **communication réseau**, la **simulation du monde de jeu**, la **synchronisation des entités ECS (Entity Component System)** et la **coordination multijoueur en temps réel**.

Le serveur agit comme un **simulateur central** : il reçoit les entrées des clients (mouvements, tirs, collisions), met à jour la logique ECS via le moteur de jeu, puis diffuse les états synchronisés vers tous les clients connectés.

---

## 🧱 Objectifs principaux

- Fournir une architecture réseau fiable, extensible et modulaire.  
- Gérer efficacement plusieurs clients simultanés via **UDP**.  
- Maintenir une simulation ECS centralisée, synchronisée avec les clients.  
- Minimiser la latence et optimiser les performances via le **multithreading**.  
- Garantir une gestion robuste des erreurs réseau et des déconnexions.  

---

## 🧩 Structure du dossier

```
SERVER/
├── CMakeLists.txt
├── main.cpp
├── Newmain.cpp
├── Server.hpp
├── Network.hpp
├── NewNetwork.hpp
├── ClientManager.hpp
├── NewClientManager.hpp
├── GameEngine.hpp
├── NewGameEngine.hpp
├── Error.hpp
├── UdpSocket.hpp
└── ../ECS/ (partagé avec le client)
```

---

## ⚙️ Détails des fichiers

### 🧱 1. **CMakeLists.txt**
Configure la compilation du binaire `r_type_server` et lie les bibliothèques **SFML** nécessaires (graphics, window, system, audio, network).  
Inclut également la logique ECS commune (`../ECS`).

Exemple de build :
```bash
mkdir build && cd build
cmake ..
make
./r_type_server
```

💡 **Astuce** : pour un développement rapide, ajouter une option `-j` à `make` pour paralléliser la compilation (`make -j4`).

---

### 🧱 2. **main.cpp / Newmain.cpp**
Point d’entrée du serveur.  

- Initialise la connexion réseau (port, socket, gestionnaire clients).  
- Démarre le moteur de jeu ECS.  
- Lance la boucle principale (écoute, traitement, diffusion).  

Exemple minimal :
```cpp
int main() {
    try {
        Server server(4242);
        server.run();
    } catch (const std::exception &e) {
        std::cerr << e.what() << std::endl;
    }
}
```

---

### 🧱 3. **Server.hpp**
Composant central du serveur. Gère :
- Les sockets UDP via `UdpSocket`.  
- Les connexions clients (via `ClientManager`).  
- La simulation du monde de jeu (via `NewGameEngine`).  
- L’envoi et la réception des paquets réseau.  

```cpp
class Server {
private:
    UdpSocket _socket;
    unsigned short _port;
    ClientManager _clients;
    NewGameEngine _engine;
    std::atomic<bool> _running;

public:
    explicit Server(unsigned short port);
    void run();
    void handlePacket(sf::Packet &packet, sf::IpAddress sender, unsigned short port);
    void broadcast(sf::Packet &packet);
    void stop();
};
```

💡 **Optimisation** : la méthode `run()` peut être exécutée dans un thread dédié pour le réseau, tandis que le moteur ECS s’exécute dans un autre thread pour réduire la latence.

---

### 🧱 4. **Network.hpp / NewNetwork.hpp**
Implémente la couche de communication réseau.  
C’est le pont entre le moteur ECS et la couche transport (UDP).

Principales fonctions :
```cpp
void sendToAll(sf::Packet &packet);
void receive();
void processIncomingPackets();
```

- Utilise `UdpSocket` pour l’envoi/réception.  
- Sérialise les entités ECS pour les transmettre aux clients.  
- Détecte les déconnexions et nettoie la liste des clients.  

🔹 **Note technique** : la version `NewNetwork` inclut un traitement asynchrone (thread de réception séparé) et un buffer circulaire pour traiter les paquets entrants sans bloquer la boucle principale.

---

### 🧱 5. **ClientManager.hpp / NewClientManager.hpp**
Gère tous les clients connectés au serveur.  
Attribution d’un identifiant unique et maintenance de leur état réseau.

```cpp
class ClientManager {
public:
    struct ClientInfo {
        sf::IpAddress address;
        unsigned short port;
        int id;
        sf::Clock lastPacketTime;
    };

    void addClient(sf::IpAddress addr, unsigned short port);
    void removeClient(int id);
    void sendToClient(int id, sf::Packet &packet);
    void broadcast(sf::Packet &packet);
    void pruneInactiveClients(float timeoutSec);
};
```

💡 **Bonne pratique** : `pruneInactiveClients()` permet de déconnecter automatiquement les clients inactifs depuis plus de X secondes.

---

### 🧱 6. **GameEngine.hpp / NewGameEngine.hpp**
Le moteur ECS du serveur. Il gère la simulation du monde, sans affichage graphique.  
`NewGameEngine` est une version étendue du moteur de base, incluant :
- Support multithreading.  
- Gestion dynamique du deltaTime.  
- Synchronisation automatique des entités ECS vers les clients.  
- Détection des collisions et mise à jour des scores.  

```cpp
class NewGameEngine {
private:
    Registry _registry;
    float _lastUpdate;
    bool _running;

public:
    void initEntities();
    void update(float deltaTime);
    void handleEvents();
    void broadcastState(ClientManager &clients);
    void stop();
};
```

📊 **Cycle de mise à jour** :
1. Calcul du deltaTime.  
2. Application des systèmes ECS (mouvement, collisions, tirs).  
3. Synchronisation du monde vers les clients.  
4. Réinitialisation du timer.  

---

### 🧱 7. **UdpSocket.hpp**
Encapsule la socket UDP SFML pour simplifier son usage et gérer les erreurs.

```cpp
class UdpSocket {
private:
    sf::UdpSocket _socket;
    bool _nonBlocking;

public:
    UdpSocket();
    void bind(unsigned short port);
    void setBlocking(bool blocking);
    bool send(sf::Packet &packet, const sf::IpAddress &ip, unsigned short port);
    bool receive(sf::Packet &packet, sf::IpAddress &ip, unsigned short &port);
};
```

- Mode non bloquant activé par défaut.  
- Timeout configurable.  
- Vérification automatique des erreurs réseau.  
- Logging des paquets échoués pour le débogage.

---

### 🧱 8. **Error.hpp**
Gère les exceptions personnalisées pour le réseau et le moteur ECS.

```cpp
class NetworkError : public std::runtime_error {
public:
    explicit NetworkError(const std::string &msg) : std::runtime_error(msg) {}
};
```

Utilisé dans `UdpSocket` et `Server` pour capturer les anomalies (timeout, port occupé, erreur d’envoi...).

---

## 🔁 Cycle complet du serveur

```
1️⃣ Initialisation du moteur ECS et des sockets
2️⃣ Attente des connexions clients
3️⃣ Réception et interprétation des paquets
4️⃣ Mise à jour du moteur ECS (NewGameEngine::update)
5️⃣ Broadcast des états synchronisés vers les clients
6️⃣ Nettoyage des clients inactifs
7️⃣ Boucle continue jusqu’à arrêt manuel
```

---

## ⚙️ Optimisations recommandées

- Séparer la logique ECS et la gestion réseau en **threads indépendants**.  
- Utiliser un **tickrate fixe (60Hz)** pour stabiliser la simulation.  
- Implémenter un **buffer de paquets** (FIFO) pour la réception réseau.  
- Logguer tous les envois via un système d’horodatage.  
- Prévoir une **commande d’arrêt distant** (`Server::stop()`) pour le debug.

---

## 📈 Avantages techniques

| Domaine | Atout |
|----------|-------|
| **Performance** | Réseau UDP non bloquant + multithreading |
| **Architecture** | Séparation claire entre réseau et ECS |
| **Robustesse** | Gestion d’erreurs centralisée et automatique |
| **Scalabilité** | Support multi-clients simultanés |
| **Interopérabilité** | Compatible avec le client ECS SFML |

---

## 🧰 Pistes d’amélioration futures

- Implémenter une **compression réseau (zlib / lz4)** pour réduire la bande passante.  
- Ajouter une **file de logs réseau** détaillée pour chaque client.  
- Support des **commandes admin** (kick, shutdown, restart).  
- Ajouter un **système de ping/pong** pour mesurer la latence par client.  
- Intégrer une **interface REST** pour monitorer les sessions serveur.  

---

## 👤 Auteur
Projet serveur développé par **des étudiants d’Epitech**.  
> Langage : C++17  
> Framework : SFML (Network, System, Audio, Graphics)  
> Architecture : ECS + Serveur UDP multithread  
> Outil de build : CMake  
> Objectif : offrir une infrastructure serveur performante, modulaire et extensible pour R-Type.
