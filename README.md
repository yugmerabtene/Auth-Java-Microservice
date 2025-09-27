# Cahier des charges – Projet Java Spring (Java 21 OpenJDK) Microservices Dockerisés avec Kafka, Nginx et CI/CD GitHub

## 1. Contexte et objectifs

Le projet consiste à développer une application en **Java Spring Boot** basée sur une architecture **microservices**.
La stack technique inclura :

* **Java 21 (OpenJDK LTS)** → gratuit en production (aucune licence à payer).
* **Docker** pour la conteneurisation.
* **Kafka** comme broker de messages.
* **Nginx** comme reverse proxy pour HTTPS.
* **GitHub Actions** pour CI/CD.

Le système sera déployé sur un **VPS Ubuntu Server 22.04**, durci avec des mesures de sécurité (SSH clés, UFW, Fail2ban).

Le pipeline CI/CD doit garantir que **si les tests échouent, aucune image n’est publiée ni déployée sur le VPS**.

---

## 2. Périmètre fonctionnel

### 2.1 Microservice Authentification (Auth Service)

* Gestion utilisateurs (inscription, connexion, suppression, modification).
* Authentification avec **JWT**.
* Gestion des rôles et permissions (**RBAC**).
* Base MySQL dédiée (`users`, `roles`, `tokens`).
* Publication d’événements dans Kafka (ex. `UserCreated`).
* **Tests unitaires et d’intégration obligatoires**.

### 2.2 Microservice Notifications

* Réception d’événements Kafka (`UserCreated`, `PasswordReset`).
* Envoi de notifications (e-mail, SMS, push en V2).
* Base MySQL dédiée (`notifications`, `status`).
* API REST pour consulter l’historique des notifications.

### 2.3 API Gateway

* Implémentée avec **Spring Cloud Gateway**.
* Point d’entrée unique vers les microservices.
* Validation des JWT.
* Routage dynamique.
* Gestion des timeouts et erreurs.

### 2.4 Frontend (Bootstrap)

* Interface responsive (HTML5, CSS3, JS ES6, Bootstrap 5).
* Pages : Inscription / Connexion, Dashboard utilisateur, Liste des notifications.
* Communication via l’API Gateway.

### 2.5 Bases de données

* **Une instance MySQL par microservice** (isolation complète).
* Migration avec **Flyway** (via Maven).

---

## 3. Architecture technique

* **Langage** : Java 21 (OpenJDK LTS, sans licence payante).
* **Framework** : Spring Boot 3.x + Spring Cloud.
* **Build** : Maven.
* **Broker** : Apache Kafka (dockerisé).
* **Bases de données** : MySQL (1 par service, dockerisée).
* **Frontend** : Bootstrap 5.
* **Documentation API** : Swagger/OpenAPI.
* **Proxy** : Nginx (terminaison TLS + reverse proxy).
* **Conteneurisation** : Docker + Docker Compose.
* **CI/CD** : GitHub Actions.
* **Serveur** : VPS Ubuntu Server 22.04.

---

## 4. Exigences CI/CD (GitHub Actions)

### 4.1 Stratégie de branches

* **`dev`** :

  * Build Maven.
  * Tests unitaires + tests d’intégration (Kafka, MySQL via Testcontainers).
  * Si échec ❌ → blocage merge.
  * Si succès ✅ → PR possible vers `main`.

* **`main`** :

  * Branch protégée (pas de push direct).
  * Merge via PR validée.
  * Pipeline complet :

    1. Build Maven
    2. Tests unitaires
    3. Tests d’intégration
    4. Build images Docker
    5. Push images vers registre privé (Docker Hub ou GHCR)
    6. Déploiement auto sur VPS (pull + restart via Docker Compose)

---

## 5. Sécurité

### 5.1 Applicative

* JWT pour l’authentification.
* Mots de passe hashés avec **bcrypt**.
* Secrets (DB, JWT keys, Kafka) dans **GitHub Secrets**.
* Communication sécurisée (TLS).

### 5.2 VPS Ubuntu Server

* **Accès SSH par clé uniquement** (désactivation mot de passe).
* **UFW** : ports autorisés 22 (SSH), 80 (HTTP), 443 (HTTPS), Kafka interne.
* **Fail2ban** : bloque brute force SSH.
* Mise à jour **manuelle et contrôlée** des paquets (via `apt update && apt upgrade`).
* Utilisateur non-root avec `sudo`.
* Logs surveillés (`journalctl`, logs Nginx).

---

## 6. Environnement et outils

* **IDE** : IntelliJ IDEA / VS Code.
* **VCS** : Git + GitHub.
* **CI/CD** : GitHub Actions.
* **Tests** : JUnit 5, Mockito, Testcontainers.
* **Proxy** : Nginx.
* **Conteneurisation** : Docker, Docker Compose.
* **Déploiement** : VPS Ubuntu Server 22.04.
* **Monitoring (V2)** : Prometheus + Grafana.

---

## 7. Livrables attendus

1. Code source complet (Auth, Notifications, API Gateway, Frontend).
2. Fichiers Dockerfile + `docker-compose.yml`.
3. Pipeline CI/CD (`.github/workflows/ci-cd.yml`).
4. Scripts Flyway (migrations BDD).
5. Rapport tests unitaires + couverture.
6. Tests d’intégration Kafka/MySQL.
7. Documentation technique et guide de déploiement.
8. Guide de sécurisation du VPS (SSH, UFW, Fail2ban).
