# Tuto pour installer symfony, le dockerisé et ce connecter à ça database avec beekeeper

# Prérequis
  - composer
  - version de php surperieur à 8.1
  - node.js (optionnel mais conseiller)

# Installtion

  - 1er étape : créer l'appli symfony de base

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

  - 2eme étape : configuration de l'environement

    - aller dans le fichier .env et commenter la ligne DATABASE_URL
    - ![image](https://github.com/user-attachments/assets/00724a9d-b36c-47c1-96dd-3d3f97b5cf65)

