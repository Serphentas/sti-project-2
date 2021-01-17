# STI - projet 2

Étudiants: Moïn DANAI, Laurent THOENY

## Table des matières

- [Introduction](#introduction)
- [Rappel technique]()
  - [Vulnérabilités](#Vulnérabilités)
  - [Exposition](#Exposition)
  - [Menaces](#Menaces)
  - [Risques et dommages](#Risques-et-dommages)
- [Description du système]()
  - [Objectifs]()
  - [Exigeances]()
  - [Composition]()
- [Sources de menaces]()
- [Scénarios d'attaque]()
- [Correctifs proposés]()
- [Conclusion]()

## 1 Introduction

Le présent document a pour but de tracer en détails l'étude de menaces effectuée pour le [codebase](https://github.com/STI-Flask-FTW/sti-project-1) tiré du projet 1 de STI. Il s'agit de déterminer les menaces potentielles pour l'application donnée avec le contexte dans lequel elle est déployée, et y apporter des correctifs de sécurité dans la mesure du possible.

Les différents aspects sécuritaires abordés et étudiés sont individuellement chapitrés pour faciliter la navigation dans le contenu qui suit.

Enfin, une conclusion apporte des appréciations qualitatives sur le codebase de départ ainsi que la version amendée par les correctifs.

## 2 Rappel technique

Par souci de clarté, nous rappelons quelques définitions, termes, et concepts se rapportant à l'étude de menaces et la sécurité numérique en général.

Pour une entitée donnée, il peut exister des vulnérabilités sécuritaires qui pourraient être à découvert et donc potentiellement exploitées par une entité malveillante (ou plus) si elle le désire afin de réaliser un certain objectif, voire des cas de force majeure (actes de Dieu, ex. tremblement de terre) sans réel dessein néfaste.

### 2.1 Vulnérabilités

Dans le cadre de services numériques, les vulnérabilités sont généralement des failles logicielles, matérielles, ou structurelles dont la nature peut varier (ex. circuit défectueux, chiffrement faible, redondance inexistance).

Leur présence est souvent due à l'erreur humaine, qu'elle soit volontaire ou non. Dans certains cas, la malveillance mène à l'instauration de portes dérobées qui sont alors connues par une poignée de gens (qui en ont connaissance ou qui la découvrent par chance).

### 2.2 Exposition

Celles-ci se rapportent souvent à des services mis en réseau, et seraient donc exposés à Internet ou un réseau informatique qui y aura affaire d'une manière directe ou indirecte.

Les connexions entre réseaux et services numériques grandissant d'année en année, il devient difficile de mesurer la portée réelle quand on expose un certain service (et donc ses potentielles vulnérabilités). Il n'est pas trop optimiste de supposer qu'en pratique tout Internet peut atteindre les services les plus ouverts (i.e. utilisés par le grand nombre) tandis que ceux plus spécifiques comme des bases de données vivront sur des réseaux restreints.

### 2.3 Menaces

Internet et les services qu'il contient étant faits à l'image du monde réel, on y trouve de manière homologue toute une série d'acteurs malveillants avec des objectifs divers et variés. Certains convoitent l'argent et donc tenteront l'extorsion (ex. rançon), tandis que certaines agences gouvernementales seront plutôt intéressées par de la propriété intellectuelle et des informations de renseignements (_intelligence_) et opteront pour des infiltrations (comprendre: avec discrétion) de longue durée pour siphonner des données.

### 2.4 Risques et dommages

Le danger, ou risque, existe lorsqu'une menace peut exploiter des vulnérabilités dans son intérêt. Cela se fait généralement avec un type de dommage appelé conséquence.

On classe cela en trois catégories:

- Confidentialité (C),
- Intégrité (I),
- Disponibilité (D),

avec un facteur contextuel:

- direct - destruction de données, mise à l'arrêt de services,
- indirect - pertes financières, nuisances à l'image de marque,

que l'on mesurera en termes d'impact pour l'entité victime.

## 3 Description du système

### 3.1 Objectifs

La messagerie doit offrir les fonctionnalités suivantes:
- possibilité de créer un compte dans le service (contrôle d'accès),
  - contrôle d'accès avec un système de login username/password,
- possibilité de rédiger des messages avec champs: De, À, Message (messagerie),
- possibilité de modifier ses propres données,
- possibilité d'administrer les utilisateurs avec un compte admin (mot de passe, désactivation, rôle).

### 3.2 Exigeances

La messagerie doit illustrer les trois propriétés suivantes:
- Disponibilité: être accessible un maximum de temps,
- Fiabilité: ne pas perdre les données de ses utilisateurs,
- Confidentialité: garantir que les messages échangés entre deux utilisateurs ne sont accessibles par personne d'autre que l'administrateur et eux-mêmes.

### 3.3 Composition

On dénombre deux entités majeurs dans le système:
1. Comptes (normaux & admin),
2. Le couple webapp-database.

Leurs relations sont représentées comme suit:

![DFD](img/dfd.jpg)

## 4 Sources de menaces

### 4.1 Sources d'attaques

Nous pouvons supposer les différents acteurs malicieux suivants:
- utilisateur malintentionné,
- ennemi de l'entité où la messagerie est déployée (ex. concurrent),
- hacker "random" cherchant un gain (a priori monétaire),

### 4.2 Cibles potentielles

Naturellement, il suit que les entités suivantes sont à risques:
- n'importe quel utilisateur,
- un administrateur,
- les entités qui dépendent des propriétés sécuritaires fournies par l'application (ex. bouts de propriété intellectuelle dans un message -> entreprise).

### 4.3 Motivations

Selon les deux derniers chapitres, nous pouvons émettre les motivations suivantes:
- déni de service,
- accès à des informations privées,
- accès aux messages internes/privés,
- pivot pour une attaque de plus grande envergure.

## 5 Scénarios d'attaque

Nous considérons des attaques à l'encontre de l'application sous différents angles, notamment:

- Mapping the application
- Attacking authentication
- Attacking session management
- Attacking access controls
- Attacking data stores
- Attacking users: Cross-site scripting
- Attacking the application server

et suivant les vecteurs d'attaque que nous avons trouvés:

- CSRF manquant dans tous les formulaires
- pas d'usage de CORS
- pas d'usage de X-Frame-Option

nous proposons les scénarios d'attaque ci-dessous.

### 5.1 Déni de service

L'application ne fournit aucune fonctionnalité de type "throttling", "quota management", et "robot check". Par conséquent, il est possible que:
- des créations de comptes en masse se fassent sans restriction,
- des messages soient envoyés sans fin, dans la limite de la taille des messages (200 caractères),
- des actions (ex. envoi de message, suppression de comptes) soient déclenchées par l'abus du manque de CSRF,
- des requêtes vers la boîte mail soient faites en masse et, supposant une quantité de messages importante, que la bande passante soit fortement sollicitée.

### 5.2 Fuite de PII

Comme nous pouvons inclure le site en Iframe sans restriction, il serait possible

### 5.3 Fuite de messages



### 5.4 Pivot



## 6 Correctifs proposés


## 7 Conclusion
