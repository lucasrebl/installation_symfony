# Tuto pour installer symfony, le dockerisé et ce connecter à ça database avec beekeeper

# Prérequis
  - composer
  - version de php surperieur à 8.1
  - node.js (optionnel mais conseiller)

# Installtion

  - 1er étape : Création l'appli symfony de base

    ```bash
    composer create-project symfony/skeleton:"7.2.x" my_project_directory
    ```

    ```bash
    cd my_project_directory
    ```

    ```bash
    composer require webapp
    ```

    attention on vous demanderas si vous voulez utilisé docker, vous dever répondre non.

    une fois ceci fait vous pouvez ouvrir votre projet dans un IDE

  - 2eme étape : Configuration de l'environement

    - Aller dans le fichier .env et commenter la ligne DATABASE_URL
      ![image](https://github.com/user-attachments/assets/00724a9d-b36c-47c1-96dd-3d3f97b5cf65)
      
    - Créer un fichier Dockerfile et ajouter y le code ci-dessous
      ```bash
      FROM php:8.2-fpm

      # Installer les extensions nécessaires
      RUN apt-get update && apt-get install -y \
          libpq-dev \
          libzip-dev \
          unzip \
          && docker-php-ext-install pdo pdo_mysql
      
      WORKDIR /app
      COPY . /app
      ```

    - Créer un fichier docker-compose.yml et ajouter y le code ci-dessous

    
