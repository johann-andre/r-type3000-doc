# 🧩 R-TYPE — Documentation du système ECS

## 📘 Sommaire
1. [Introduction](#-introduction)  
2. [Structure du projet ECS](#-structure-du-projet-ecs)  
3. [Principe général](#-principe-général)  
4. [Les Composants (`Component`)](#-les-composants-component)  
5. [Les Entités (`Entity`)](#-les-entités-entity)  
6. [Le Registre (`Registry`)](#-le-registre-registry)  
7. [Les Systèmes (`System`)](#-les-systèmes-system)  
8. [Cycle de jeu (Game Loop)](#-cycle-de-jeu-game-loop)  
9. [Ajout d’un nouveau composant : exemple `CollisionInfo`](#-ajout-dun-nouveau-composant-exemple-collisioninfo)  
10. [Schéma du flux ECS](#-schéma-du-flux-ecs)  
11. [Bonnes pratiques](#-bonnes-pratiques)

---

## 🚀 Introduction

Le moteur du jeu **R-TYPE** repose sur une architecture **ECS (Entity Component System)**, un modèle orienté **composition** plutôt qu’héritage.

Ce modèle rend le code :
- **Flexible** : on peut combiner des composants sans dépendances fortes.  
- **Performant** : chaque système traite uniquement les entités qui possèdent les composants dont il a besoin.  
- **Modulaire** : facile à ajouter de nouveaux comportements sans modifier les anciens.

---

## 🧱 Structure du projet ECS

```
ECS/
├── Component.hpp     # Définitions des composants (Position, Velocity, Health, etc.)
├── Entity.hpp        # Représentation d'une entité
├── Registry.hpp      # Gestionnaire central des entités et composants
├── SparseArray.hpp   # Stockage optimisé des composants
├── System.hpp        # Ensemble des systèmes (render, movement, collision, etc.)
├── GameLogic.hpp     # Logique gameplay (Player, Ennemis, Missiles)
└── Libs.hpp          # Librairies externes (SFML, standard libs, etc.)
```

---

## ⚙️ Principe général

L’**ECS** décompose le jeu en trois grandes familles d’objets :

| Élément | Rôle |
|----------|------|
| **Entity** | Identifiant unique représentant un objet du jeu (joueur, missile, ennemi, etc.) |
| **Component** | Structure de données pure (ex: position, santé, dégâts, sprite, tag, etc.) |
| **System** | Code qui agit sur un ensemble d’entités possédant certains composants |

Exemple concret :  
👉 un missile est une entité qui possède :
- `Position`
- `Hitbox`
- `Object` (sprite SFML)
- `Tag("missile")`
- `Damage`
- `Shooter`

Et il sera traité par :
- `MissileSys` pour le déplacement  
- `CollisionSystem` pour la détection d’impact  
- `Renderring` pour l’affichage  

---

## 🧩 Les Composants (`Component`)

Chaque composant est une **structure légère** contenant uniquement des données.

### Composants de base
| Nom | Description |
|------|-------------|
| `Position` | Coordonnées `(x, y)` d’une entité |
| `Velocity` | Vitesse de déplacement `(vx, vy)` |
| `Health` | Points de vie |
| `Damage` | Dégâts infligés |
| `Tag` | Chaîne identifiant le type d’entité (`player`, `enemy`, `missile`, `background`) |
| `Object` | Sprite SFML associé à l’entité |
| `Hitbox` | Dimensions de collision `(width, height)` |
| `Level` | Niveau de difficulté d’un ennemi |
| `Shooter` | Identifiant du joueur ayant tiré un missile |
| `CollisionInfo` | Données d’une collision : `shooter_id`, `missile_id`, `target_id` |

### Exemple :
```cpp
namespace Component {

    struct Position {
        float x, y;
    };

    struct Health {
        int health;
    };

    struct Damage {
        int damage;
    };

    struct Tag {
        std::string tag;
    };

    struct Shooter {
        int player_id;
    };

    struct CollisionInfo {
        int shooter_id;
        int missile_id;
        int target_id;
    };

}
```

---

## 🧬 Les Entités (`Entity`)

Une **entité** est un simple identifiant entier (`id`) géré par le `registry`.

Création :
```cpp
Entity missile = reg.spawn_entity();
reg.add_component<Component::Position>(missile, {x, y});
reg.add_component<Component::Tag>(missile, {"missile"});
```

Destruction :
```cpp
reg.kill_entity(missile);
```

---

## 🗂️ Le Registre (`Registry`)

Le `registry` centralise :
- les entités
- les composants
- les systèmes

Il permet de :
- enregistrer de nouveaux types de composants (`register_component`)
- ajouter/enlever des composants à une entité (`add_component`)
- accéder à tous les composants d’un type (`get_components`)

Initialisation :
```cpp
registry reg = init_register();
```

---

## ⚙️ Les Systèmes (`System`)

Les **systèmes** appliquent une logique sur un groupe d’entités ayant les bons composants.

| Système | Rôle |
|----------|------|
| `InputSys` | Récupère les entrées clavier |
| `MouvementSys` | Déplace le joueur selon l’entrée |
| `MissileSys` | Fait avancer les missiles et les détruit hors de l’écran |
| `EnemyMovementSys` | Déplace les ennemis vers la gauche |
| `CollisionSystem` | Détecte et gère les collisions entre entités |
| `BackgroundSystem` | Gère le défilement du fond |
| `Renderring` | Dessine tous les sprites sur la fenêtre SFML |
| `CollisionEventSystem` *(optionnel)* | Informe le serveur des collisions via `CollisionInfo` |

---

## 🔄 Cycle de jeu (Game Loop)

Le moteur tourne en boucle :

```cpp
while (window.isOpen()) {
    float dt = clock.restart().asSeconds();

    input = InputSys();
    MouvementSys(reg, input, player, dt);
    BackgroundSystem(reg, dt);
    EnemyMovementSys(reg, dt, 1920);
    MissileSys(reg, dt, 1920);
    CollisionSystem(reg);
    CollisionEventSystem(reg, client); // Envoi au serveur

    window.clear();
    Renderring(reg, window);
    window.display();
}
```

---

## 🆕 Ajout d’un nouveau composant : exemple `CollisionInfo`

### 1️⃣ Définir le composant
```cpp
struct CollisionInfo {
    int shooter_id;
    int missile_id;
    int target_id;
};
```

### 2️⃣ L’enregistrer dans le `registry`
```cpp
reg.register_component<Component::CollisionInfo>();
```

### 3️⃣ Le remplir dans `CollisionSystem`
```cpp
if (tag1 == "missile" && tag2 == "enemy") {
    int shooterId = shooters[i] ? shooters[i]->player_id : -1;
    Entity collision = reg.spawn_entity();
    reg.add_component<Component::CollisionInfo>(collision, {shooterId, (int)i, (int)j});
}
```

### 4️⃣ Le traiter dans `CollisionEventSystem`
```cpp
for (auto &col : reg.get_components<Component::CollisionInfo>()) {
    if (col) {
        client.sendMessage("COLLISION " +
            std::to_string(col->shooter_id) + " " +
            std::to_string(col->missile_id) + " " +
            std::to_string(col->target_id));
    }
}
```

---

## 🧠 Schéma du flux ECS

```
      ┌──────────────┐
      │   InputSys   │ ←─ clavier
      └──────┬───────┘
             │
             ▼
     ┌───────────────┐
     │ MouvementSys  │
     └───────────────┘
             │
             ▼
     ┌───────────────┐
     │ MissileSys    │
     └───────────────┘
             │
             ▼
     ┌───────────────┐
     │ RenderingSys  │──▶ affichage SFML
     └───────────────┘
```

---

## 💡 Bonnes pratiques

- **Pas de logique dans les composants** → uniquement des données.  
- **Un système = une responsabilité.**  
- **Toujours nettoyer les entités mortes** pour éviter les crashs ou doublons.  
- **Éviter les références croisées** entre entités, préférer les `id`.  
- **Déboguer les collisions** avec des `std::cout` + rectangles rouges (déjà intégrés).  

---

## 📄 Auteurs
Projet développé dans le cadre de **R-TYPE (Epitech 2025)**  
Architecture ECS conçue et implémentée par **Hs Darren**
