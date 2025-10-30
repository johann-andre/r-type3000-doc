# Comment tester le jeu

## Prérequis
- Un compilateur compatible **C++20**.  
- **CMake** installé.  
- La bibliothèque **SFML** disponible.  

## Étapes de test
1. Cloner le dépôt :
   ```bash
   git clone <repo-link>
   cd r-type
   ```
2. Configurer et compiler avec CMake :
   ```bash
   cmake .
   make
   ```
3. Lancer le **Serveur** :
   ```bash
   ./r-type_server
   ```
4. Lancer un ou plusieurs **Clients** :
   ```bash
   ./r-type_client
   ```
5. Tester le mode multijoueur et les fonctionnalités principales.  

## Résultats attendus
- Plusieurs clients peuvent se connecter au serveur.  
- Chaque joueur contrôle un vaisseau, peut tirer et interagir avec les ennemis.  
- Les échanges réseau sont synchronisés en temps réel.  