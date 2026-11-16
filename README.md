# AutoLoc — Plateforme de gestion de location de véhicules multi-agences

Étude de cas fil rouge du cours **UP ASI — Architecture des Systèmes d'Information** (Esprit, ASI 26-27).
Back-end **Spring Boot** exposant, à terme, une API REST pour numériser le processus de location :
réservation, contractualisation, facturation, suivi de flotte et relances automatiques.

## Objectifs du projet

- Construire pas à pas une architecture en couches (monolithe) : domaine, persistance, métier, présentation.
- Mettre en pratique Spring Boot, Maven, Spring Data JPA, Lombok, puis les patrons de conception, SOLID, Spring MVC, AOP…

## Acteurs identifiés (Séance 1)

| Acteur | Rôle | Cas d'utilisation principaux |
|---|---|---|
| **Client** | Particulier ou professionnel souhaitant louer un véhicule | Consulter les véhicules disponibles, créer / annuler une réservation, consulter ses contrats |
| **Agent d'agence** | Employé en charge de la gestion opérationnelle d'une agence | Gérer les véhicules, valider une réservation, établir un contrat, enregistrer un paiement |
| **Responsable d'agence (Manager)** | Supervise une agence et son personnel | Droits de l'agent + gestion des employés, statistiques de l'agence |
| **Administrateur** | Administre la plateforme | Gérer les agences et les catégories de véhicules, statistiques globales, configuration |

## Stack technique

| Catégorie | Choix |
|---|---|
| Langage / Build | Java 17, Maven (embarqué dans IntelliJ IDEA) |
| Framework | Spring Boot 3.5.16, Spring Web, Spring Data JPA, Validation, DevTools |
| Base de données | MySQL 8.x (`autoloc_db`) |
| Productivité | Lombok |
| Outillage | IntelliJ IDEA, Git/GitHub, Postman |

## Structure du dépôt

```
AutoLoc
├── README.md
├── docs/modele-ea-autoloc.png          Modèle Entités/Associations de l'étude de cas
└── autoloc-api                          Projet Spring Boot / Maven
    ├── pom.xml
    └── src/main
        ├── java/tn/esprit/autoloc
        │   ├── AutolocApiApplication.java
        │   ├── domain                   Entités JPA et énumérations
        │   ├── repository               Interfaces Spring Data JPA (Atelier 4)
        │   ├── service                  Couche métier (Atelier 4)
        │   └── web
        │       ├── controller           Contrôleurs REST (Atelier 5)
        │       └── dto                  DTO (Atelier 6)
        └── resources/application.properties
```

## Modèle de domaine (Atelier 1)

Neuf entités JPA, sans association pour l'instant (elles sont ajoutées à l'Atelier 2 avec les stratégies de cascade et de fetch).
Toutes utilisent `@GeneratedValue(strategy = GenerationType.IDENTITY)` et Lombok (`@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`).

| Entité (table) | Attributs |
|---|---|
| `Vehicule` (`vehicule`) | idVehicule, immatriculation, marque, modele, categorie, tarifJournalier, statut |
| `Agence` (`agence`) | idAgence, nom, ville, adresse, telephone |
| `Client` (`client`) | idClient, nom, prenom, email, telephone, numPermis, dateInscription |
| `Employe` (`employe`) | idEmploye, nom, prenom, role |
| `Equipement` (`equipement`) | idEquipement, libelle |
| `Reservation` (`reservation`) | idReservation, dateDebut, dateFin, statut |
| `Contrat` (`contrat`) | idContrat, dateSignature, montantTotal, valide |
| `Paiement` (`paiement`) | idPaiement, montant, datePaiement, modePaiement |
| `Maintenance` (`maintenance`) | idMaintenance, dateDebut, dateFin, description |

Énumérations (stockées en `String` via `@Enumerated(EnumType.STRING)`) :
`StatutVehicule` (DISPONIBLE, LOUE, MAINTENANCE), `CategorieVehicule` (CITADINE, BERLINE, SUV, UTILITAIRE),
`RoleEmploye` (AGENT, MANAGER), `StatutReservation` (EN_ATTENTE, CONFIRMEE, ANNULEE, TERMINEE),
`ModePaiement` (CARTE, ESPECES, VIREMENT).

## Lancer le projet

**Prérequis :** JDK 17+, IntelliJ IDEA (annotation processing activé + plugin Lombok), MySQL démarré sur le port 3306.

1. Cloner le dépôt et ouvrir le dossier `autoloc-api` dans IntelliJ IDEA (*Open as Project*), puis attendre l'import Maven.
2. Si le compte MySQL a un mot de passe, le fournir par variable d'environnement (aucun mot de passe n'est versionné) :
   `DB_USERNAME` (défaut : `root`) et `DB_PASSWORD` (défaut : vide).
3. Lancer `AutolocApiApplication` : la base `autoloc_db` est créée si elle n'existe pas (`createDatabaseIfNotExist=true`)
   et Hibernate génère les 9 tables (`spring.jpa.hibernate.ddl-auto=update`).
4. Vérifier dans les logs la ligne `Started AutolocApiApplication in … seconds` et les requêtes `create table …`,
   puis contrôler les tables avec MySQL Workbench / DBeaver / phpMyAdmin.

> `ddl-auto=update` et `show-sql=true` sont des réglages de **développement** : ils ne doivent pas être utilisés tels quels en production.

## Commandes Maven (Activité Maven)

Depuis le panneau *Maven* d'IntelliJ (ou en ligne de commande dans `autoloc-api`) :

```bash
mvn clean compile package install
```

Les tests ne sont pas exécutés par défaut (`skipTests=true` dans le `pom.xml`) car le test de contexte Spring nécessite une base MySQL démarrée ;
les tests unitaires sont vus en 5ᵉ année (module DevOps). Ne pas lancer les phases `test` ni `deploy`.

## Avancement

- [x] Atelier 0 : environnement (JDK 17, IntelliJ, MySQL, Postman, Git), README v0
- [x] Atelier 1 : projet Spring Boot + Maven, connexion MySQL, 9 entités JPA, génération du schéma
- [ ] Atelier 2 : associations, cascade et fetch
