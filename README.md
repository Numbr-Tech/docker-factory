# 🏭 Docker Factory – Numbr-Tech

Ce dépôt héberge une **usine CI/CD centralisée** pour la **construction et le déploiement d’images Docker** des projets de l’organisation **Numbr-Tech**.

---

🎯 **Objectif** : Centraliser, sécuriser et automatiser la gestion des images Docker pour l’ensemble des projets Numbr-Tech 🚀

---

## 🧠 Philosophie

Plutôt qu’un CI par projet dispersé, ce dépôt propose une approche **industrielle et modulaire** :
- Un dépôt unique pour tous les workflows Docker
- Un fichier YAML **par projet** pour plus de clarté et de personnalisation
- Sécurité, évolutivité et maintenance simplifiée

---

## 🔧 Projets intégrés

Chaque projet dispose de son propre fichier workflow dans `.github/workflows/`.  
👉 **Exemple** : `neo.yml` pour Neo

---

## 📦 Checkout de dépôt distant

Dans certains cas, les images Docker ne sont **pas génériques** ou ne sont pas construites à partir du dépôt central.  
👉 Ces projets nécessitent un **checkout d’un dépôt distant** pour récupérer leurs sources avant compilation :

- Le workflow utilise une étape dédiée avec `actions/checkout` configuré sur le dépôt distant concerné.
- L’accès est sécurisé via une **GitHub App** et un **token d’installation dynamique** généré au runtime.
- Cette méthode permet de centraliser la factory tout en gardant les projets modulaires et indépendants.

> 🛠 Exemple : Le projet **Neo** utilise cette approche pour builder ses images depuis `Numbr-Tech/Neo`.  
> ⚠️ _Note : Neo est un cas particulier — les images sont construites avec **Bazel** et non via `docker build`. La logique CI reste identique, mais le mécanisme de build diffère._


---

## 🚀 Déclencheurs CI

Les workflows s’exécutent :
- à chaque **push sur la branche `develop`**
- automatiquement **chaque dimanche à 07h30 UTC** via le scheduler GitHub Actions

---

## 🐳 Build & Push des images

Les images Docker sont :
- construites via `docker build`
- taggées avec le tag `latest`
- poussées vers un **Azure Container Registry (ACR)** en fonction de la branche :
  - `main` → registre **de production**
  - autres branches → registre **de préproduction**

Chaque projet définit son propre matrix ou configuration dans son fichier.

---

## 🔐 Authentification GitHub App

Certains jobs nécessitent un accès GitHub authentifié via une **GitHub App** :
- Génération d’un JWT signé
- Obtention d’un token d’accès GitHub via l’API
- Accès sécurisé aux dépôts privés

---

## ☁️ Authentification Azure (ACR)

Connexion sécurisée à Azure via **Federated Identity Credentials** :

- Pas de secrets statiques
- Utilisation du module `azure/login@v2`
- Push automatisé des images vers ACR

---

## 🧩 Ajouter un nouveau projet dans la Factory

Pour intégrer un projet :
1. Créer un fichier `.github/workflows/nom-du-projet.yml`
2. Définir les déclencheurs (`push`, `schedule`) dans ce fichier
3. Configurer les variables `REGISTRY`, `PROJECT`, `DOCKERFILE_ROOT`, etc.
4. Ajouter les steps nécessaires à la construction et au push
5. Effectuer un push sur `develop` pour valider le bon fonctionnement du workflow, puis merger sur `main` une fois la configuration testée et approuvée  
   > ⚠️ Le scheduler ne s’exécute que sur `main` : aucune exécution planifiée n’aura lieu tant que les modifications restent sur `develop`.


> 💡 Le workflow Neo peut servir de base pour les nouveaux projets. Dupliquer, adapter, tester !

---

## 🛠️ Technologies utilisées

| Outil / Service             | Description                                       |
|-----------------------------|---------------------------------------------------|
| GitHub Actions              | Orchestration des workflows CI/CD                |
| Docker                      | Build et gestion des images                      |
| Azure Container Registry    | Registry privé hébergé sur Azure                 |
| GitHub App                  | Authentification avancée via JWT & token         |
| Federated Identity (Azure) | Connexion sécurisée sans secrets statiques       |
| jq + curl                   | Communication fluide avec l'API GitHub           |