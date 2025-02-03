# Tuto pour installer symfony, le dockerisé et ce connecter à ça database avec beekeeper

# Prérequis
  - composer
  - version de php surperieur à 8.1
  - node.js (optionnel, mais conseiller)

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

    attention on vous demandera si vous voulez utilisé docker, vous dever répondre non.

    une fois, ceci fait, vous pouvez ouvrir votre projet dans un IDE

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
          && docker-php-ext-install pdo pdo_mysql zip
      
      WORKDIR /app
      COPY . /app
      ```

    - Créer un fichier php.ini et ajouter y le code ci-dessous
      ```bash
      extension=pdo_mysql
      ```

    - Créer un fichier docker-compose.yml et ajouter y le code ci-dessous
      ```bash
      services:
        app:
          build: .
          container_name: symfony2_app
          ports:
            - "8080:8000"
          volumes:
            - .:/app
            - ./php.ini:/usr/local/etc/php/php.ini
          depends_on:
            - database
      
        database:
          image: mysql:8.0.32
          container_name: symfony2_mysql
          environment:
            MYSQL_ROOT_PASSWORD: symfony_root_password
            MYSQL_DATABASE: symfony2_db
            MYSQL_USER: symfony2_user
            MYSQL_PASSWORD: symfony2_password
          ports:
            - "3306:3306"
          volumes:
            - db_data:/var/lib/mysql
      
        nginx:
          image: nginx:latest
          container_name: symfony2_nginx
          ports:
            - "8000:80"
          volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf:ro
            - ./snippets:/etc/nginx/snippets:ro 
            - .:/app
          depends_on:
            - app
      
      volumes:
        db_data:
      ```

      n'oublier pas de remplacer tous les endroit ou il y'a symfony2 pas le nom de votre application ou autre, cela vous permettera de mieux reconnaitre votre conteneur, etc...
      et pensez à changer les ports si nécessaire

    - Créer un fichier nginx.conf et ajouter y le code ci-dessous
      ```bash
      worker_processes auto;

      events {
          worker_connections 1024;
      }
      
      http {
          include       /etc/nginx/mime.types;
          default_type  application/octet-stream;

          server {
              listen 80;
              server_name localhost;
      
              root /app/public;
              index index.php index.html;
      
              location / {
                  try_files $uri /index.php$is_args$args;
              }
      
              location ~ ^/index\.php(/|$) {
                  fastcgi_pass symfony2_app:9000;
                  fastcgi_param SCRIPT_FILENAME /app/public/index.php;
                  include fastcgi_params;
              }
      
              location ~ \.php$ {
                  include /etc/nginx/snippets/fastcgi-php.conf;
                  fastcgi_pass symfony2_app:9000;
                  fastcgi_param SCRIPT_FILENAME /app/public$fastcgi_script_name;
              }
          }
      }
      ```

      pareil ici n'oubliez pas de remplacer symfony2 par le nom de votre application

    - Créer un dossier snippets et à l'intérieur créer un fichier fastcgi-php.conf et ajouter y le code ci-dessous
      ```bash
      fastcgi_split_path_info ^(.+\.php)(/.+)$;
      fastcgi_index index.php;
      include fastcgi_params;
      ```

    - Créer un fichier .env.local et ajouter y le code ci-dessous
      ```bash
      DATABASE_URL="mysql://symfony2_user:symfony2_password@database:3306/symfony2_db?serverVersion=8.0.32&charset=utf8mb4"
      ```

      ici aussi n'oubliez pas de remplacer symfony2_user etc... par ce que vous avez dans votre fichier docker-compose.yml

    - et pour finir avec cette 2eme étape exécuter la commande suivante
      ```bash
      docker-compose up -d
      ```

  - 3eme étape : Créer un utilisateur MySQL

    - Dans votre terminal taper la commande suivante
      ```bash
      docker exec -it symfony2_mysql mysql -u root -p
      ```
      ici symfony_mysql et le nom du container dans le docker-compose.yml pense à le changer si vous l'avez modifier dans votre fichier.
      ensuite un mot de passe vous seras demander, le mot de passe et dans le fichier docker-compose.yml, ici c'est 'symfony_root_password'
      ![image](https://github.com/user-attachments/assets/21858610-4657-4f66-b173-562cd00246e8)

    - Une fois connecter vous devez créer un utilisateur MySQL en éxécutant les commande suivante
      ```bash
      CREATE USER 'symfony2_user'@'%' IDENTIFIED BY 'symfony2_password';
      ```
      
      ```bash
      GRANT ALL PRIVILEGES ON symfony2_db.* TO 'symfony2_user'@'%';
      ```
      
      ```bash
      FLUSH PRIVILEGES;
      ```

      n'oubliez pas de remplacer ce qui est entre les guillemet par ce que vous avez dans votre docker-compose.yml

      - une fois fini vous pouvez taper exit pour sortir et taper la commande suivante pour tester de créer la base de données
        ```bash
        docker exec -it symfony2_app php bin/console doctrine:database:create
        ```
      si tous fonctionne bien vous devrez avoir un message indiquant la création de la BDD comme ci-dessous
      ![image](https://github.com/user-attachments/assets/5e983a4a-e530-4415-b6d0-2343a827612e)
      

  - 4eme étape : aller sur ce lien pour vérifier http://localhost:8000/ que votre application symfony se lance correctement

  - 5eme étape : se connecter à un client de base de données pour pouvoir intéragir avec notre BDD (ici j'utiliserais beekeeper studio)

    - choisissez une nouvelle connexion MySQL
      ![image](https://github.com/user-attachments/assets/830e79a0-a574-4c53-a9b0-3440ed34dfd6)

    - remplisser les champs nécessaire avec ce que vous avez dans votre fichier docker-compose.yml
      ![image](https://github.com/user-attachments/assets/83494c05-023a-44af-910f-3c06aa0c3deb)

    - et pour finir il suffit de vous connecter pour avoir accès à votre BDD
   
  # Conclusion

  j'espère que cela vous à aider, si vous avez une question n'hésitez pas à me contacter à l'adresse mail suivante : rebouletlucas@gmail.com
