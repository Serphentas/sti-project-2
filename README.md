# STI - projet 2

Étudiants: Moïn DANAI, Laurent THOENY

## Table des matières

- [Introduction](#introduction)
- [Rappel technique]()
  - [Vulnérabilités](#Vulnérabilités)
  - [Exposition](#Exposition)
  - [Menaces](#Menaces)
  - [Risques et dommages](#Risques-et-dommages)
- [Buts visés]()
- [Analyse de menaces]()
  - [Topic 1]()
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
