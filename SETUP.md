# STI - projet 2

Étudiants: Moïn DANAI, Laurent THOENY

## Table des matières

- [1 Introduction](#1-Introduction)
- [2 Utilisation](#2-Utilisation)
    - [2.1 Automatique](#2.1-Automatique)
    - [2.2 Manuel](#2.2-Manuel)

## 1 Introduction

Ce document couvre le lancement du code corrigé de STI. Il est relativement similaire au [readme précédent](site/README.md).

## 2 Utilisation

Il existe deux manières de lancer le projet:
1. [automatique](#2.1-Automatique), avec `docker-compose` qui facilite la tâche,
2. [manuel](#2.2-Manuel), comme son nom l'indique avec tout en manuel.

### 2.1 Automatique

Ces trois commandes suffisent:

```
cd site
docker-compose up -d
docker exec -it sti_uwsgi python /app/scripts/devseed.py
```

**NOTE:** attendre la fin du lancement de MariaDB avant de lancer la commande finale.

Sont ensuite disponibles:
- [STI Messenger](http://localhost:12321), login admin:admin
- [adminer](http://localhost:45654), login sti:stipass

### 2.2 Manuel

Se référer au [readme précédent](site/README.md), section 'Usage Local'.