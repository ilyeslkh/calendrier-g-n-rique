# BUT3A - Systèmes de gestion de rendez-vous en ligne

Par Nathan Hallez et Ilyès Lekehal De Percin

## Description

Ce projet consiste à réaliser un site internet de gestion de rendez-vous multi-utilisateurs.\
Le site permet d’une part de montrer aux utilisateurs les créneaux libres, d’autre part de permettre
aux utilisateurs de saisir et gérer leurs rendez-vous.

À la configuration de l'application, on précise quelques paramètres qui permettent d'adapter le planning à toutes les situations.
Par exemple le planning de réservation des créneaux de piscine (avec la contrainte “pas plus de 30 personnes par heure”)
ou le planning de réservation de créneaux chez le médecin (avec la contrainte “pas plus d’1 personne toutes les 15mn”).

## Bien démarrer

### Frontend

Le frontend est écrit en [vue.js](https://fr.vuejs.org/), assurez-vous d'avoir nodejs et npm d'installé pour pouvoir
lancer l'application.\
Le moyen le plus simple sur Debian est d'exécuter cette commande :

```shell
sudo apt update && sudo apt install nodejs npm
```

Allez dans le dossier frontend du projet, et installez les dépendances

```shell
npm i
```

Vous pouvez ensuite démarrer un serveur pour le développement

```shell
npm run dev
```

### Backend

Vous devez avoir maven et java d'installé (on va supposer que c'est déjà le cas).

Dans le dossier backend, exécutez cette commande

```shell
mvn clean spring-boot:run
```

Maintenant que le frontend et le backend sont lancés, vous pouvez aller sur http://localhost:5173/ depuis votre navigateur.

## Choix des scénarios

Pour pouvoir générer aléatoirement des rendez-vous durant une longue période, nous avons créé des scénarios en Java
plutôt que dans des `import.sql` (ce choix sera justifié dans [cette partie](#générer-un-jeu-de-données)).

Vous trouverez dans le fichier `application.properties` la variable `app.scenario.name`. Deux scénarios sont disponibles : `medecin` et `piscine`.

Si toutefois vous voulez quand même exécuter un des deux scripts `import.sql`, vous devez mettre les champs comme ceci

```
app.scenario.name=
spring.jpa.hibernate.ddl-auto=create
```

## Technos utilisées

### Frontend

- Vue.js : Pour créer des interfaces utilisateurs avec des composants. Nous utilisons certains modules supplémentaires
  (pinia pour le partage d'information entre les pages, et Vue Router pour faire une single page application).
- TailwindCSS : Comparable à Bootstrap. Il permet de se libérer des feuilles CSS, il ne fournit pas de style par défaut
  comme Bootstrap : Bien plus pratique quand on a un style qui s'éloigne du style par défaut.
- Dicebear : Librairie qui fait des appels à une API pour générer des photos de profil par défaut.
- Axios : Utilisé pour faire des appels à l'API REST
- Moment : Améliore le formattage des dates
- Lucide-vue : Pack d'icônes

### Backend

- Spring Boot : Le framework qui englobe tout le projet
- H2 : Utilisée pour de la base de donnée in-memory
- Lombok : Annotation pour les modèles
- Spring Boot Security : Pour tout ce qui est sécurité (authentification, autorisations)
- OAuth2-jose : Pour la gestion des tokens JWT
- Spring Boot Starter Mail : Pour l'envoi de mail
- Spring Doc API Starter WebMVC UI : Pour avoir swagger
- ICal4J : Pour envoyer des informations iCal dans les emails de réservations

## Difficultés rencontrées

### Choix des technologies

Tout d’abord, nous avons fait le choix de séparer la partie front et back en deux.
En effet, cela nous a beaucoup aidé par la suite du projet, mais aussi, nous avons dû faire face à d’autres
types de problèmes suite à ce choix.

On devait gérer en plus les erreurs liées à l'API REST, la charge de travail était augmentée surtout
pour faire le transfert d'information entre le front et le back.

Enfin, la bonne mise en pratique et la compréhension de Vue était une difficulté pour les premières semaines,
bien que par la suite, ce framework nous ont bien aidé pour la réalisation du front.

### Gestion des créneaux hebdomadaires

Dès le départ du projet, nous voulions que les créneaux hebdomadaires soient visible sur tout le calendrier, pas seulement après
un an, 10 ans, etc. Quand on crée un rendez-vous, si le créneau associé à un créneau hebdomadaire n'existe pas, on le crée aussi.
Cela complique certaines requêtes d'affichage des créneaux, puisqu'on doit prendre en compte des créneaux qui n'existent pas encore.

Ces deux requêtes en question sont dans le [TimeSlotsRepository](backend/src/main/java/fr/univlille/iut/gestionnaireplanning/repositories/TimeSlotsRepository.java),
il y a peut-être moyen de les optimiser, mais il ne faut pas oublier de prendre en compte certains cas spécifique.
Par exemple, quand toute une journée a des créneaux annulés, il faut bien mettre la journée en rouge dans le calendrier.

Plus globalement, la gestion des dates était un problème assez récurrent et nous a bien posé des complications notamment car il y a plein de types différents à gérer (LocalDate, LocalTime, Calendar, sql.Date), qui demande parfois de faire des conversions.

### Générer un jeu de données

Pour pouvoir faire une démonstration, nous voulions avoir un jeu de donnée qui puisse s'étendre à plusieurs mois.\
En SQL, cela devient très compliqué de devoir générer un grand jeu de donnée. De plus, dans H2, il n'y a ni les
generate_series, ni les requêtes récursives comme dans Postgresql.

Pour régler ce problème, nous avons préféré créer une classe Java qui en fonction d'une date de départ et d'une date de fin
nous génère aléatoirement des rendez-vous. Certains peuvent même être annulés aléatoirement.

Pour la forme, nous avons quand même crée deux `import.sql` pour le scénario piscine et médecin qui contiennent moins de données.

### Spring Security

La mise en place d'un système de token JWT a aussi été une partie complexe lors du projet.
En effet, l'authentification et la sécurisation des sessions utilisateurs via des tokens représentaient un défi technique.
Cela impliquait la génération, la distribution, et la validation de tokens pour sécuriser les échanges entre le client et le serveur.

Une fois bien configuré, Spring Boot gère toutes ces étapes tout seul, mais encore faut-il savoir le configurer !\
La documentation change régulièrement, ne dis parfois pas tout. Quand on fait des recherches sur internet, certaines choses
ne sont pas à jour.

Pour le token JWT, il y a surtout beaucoup de tutoriels sur internet qui réinventent la roue au lieu d'utiliser des
fonctionnalités déjà existantes sur Spring Security. En croisant certains guides, nous avons réussi à trouver une solution
qui est assez concise.

Malgré tout, ce système de token nous permet plus de flexibilité, comme l'ajout d'une date d'expiration.

### Afficher la liste des réservations

Nous avons décidé d’afficher la liste des réservations par ordre décroissant.
Les réservations doivent être regroupées sur le jour, car il peut y avoir plusieurs réservations sur une journée.
On ne peut pas faire de GROUP BY dans une requête SQL, parce qu'il faudrait obligatoirement faire de l'aggrégation.
Pour simplifier l'affichage, on a préféré transformer le résultat en une map où la clé est une date
et la valeur une liste des réservations.

## Améliorations potentielles

- Permettre à l'administrateur d'annuler plusieurs créneaux en même temps, en donnant un outil qui permet de sélectionner
  plusieurs créneaux par exemple. Actuellement, il est obligé de supprimer les créneaux un à un
- Créer un système de pagination pour l'affichage des réservations courantes et passées.
  Actuellement, on limite l'affichage à 50 créneaux.

## Nos impressions

- Ilyès :

Tout d’abord, j’ai trouvé ce sujet très intéressant d’un point de vue pédagogique, mais aussi technique.

J’ai appris de nouvelles technologies telles que Vue, Tailwind et Spring Security.
Ce projet m’a permis de mieux comprendre le cours de web back en Spring que je suivais en parallèle.

De plus, Nathan m’a aussi montré une nouvelle méthode de travail avec des nouveaux outils comme GitLens, Prettier et surtout IntelliJ IDEA.

D’autre part, j’ai approfondi mes connaissances vis-à-vis de Spring. J’ai mieux compris le principe de Composant et de Controller.

Enfin, je trouve que faire le point chaque semaine de projet est une bonne idée, car cela nous donne idée de notre avancée au fur et à mesure des semaines.

- Nathan :

Ce projet était très intéressant. En plus de cela, nous avons fait le projet avec un framework Javascript que nous ne connaissions pas.
C'est une agréable découverte. Cependant, il est vrai qu'il aurait été plus simple par moment d'avoir un projet uniquement en Spring.
De plus, nous avons fait une single page application, ce qui complexifie parfois l'implémentation par rapport à un site rendu côté serveur.

Nous pensons quand même que l'utilisation d'un tel framework Javascript nous a permis d'aller plus loin côté frontend. C'était l'occasion aussi pour nous
d'apprendre à utiliser TailwindCSS, qui est très agréable à utiliser.
