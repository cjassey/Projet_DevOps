# README

## Introduction

This project provides a Docker setup for a PostgreSQL database along with Adminer for database management. Below are the detailed steps to create and manage your PostgreSQL Docker container, set up networking, and ensure data persistence.

## Prerequisites

- Docker installed on your machine
- Terminal or command prompt access

## Step-by-Step Guide

### 1. Create the Dockerfile

Create a `Dockerfile` with the following content:

```dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

### 2. Build the Docker Image

Open a terminal and navigate to the directory containing your `Dockerfile`. Run the following command to build the Docker image:

```sh
docker build -t my_postgres .
```

This command will build the image and tag it as `my_postgres`.

### 3. Create a Docker Network

Create a Docker network named `app-network`:

```sh
docker network create app-network
```

### 4. Run the PostgreSQL Container

Run a container from the image you just built. Use the following command to start the container and bind a port on your host to the container's port 5432:

```sh
docker run --name mypostgres -p 5432:5432 -d --network app-network my_postgres
```

To verify that the container is running, use:

```sh
docker ps
```

You should see output similar to this:

```
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS              PORTS                  NAMES
971af71c88a0   my_postgres   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp mypostgres
```

### 5. Run the Adminer Container

Run an Adminer container for database management:

```sh
docker run --name my_adminer -p 8080:8080 --network app-network -d adminer
```

You can now access Adminer by navigating to `http://localhost:8080` in your web browser.

### 6. Prepare SQL Files

Create a directory named `data` in the same location as your `Dockerfile`. Place your SQL files inside this `data` directory.

01-CreateScheme.sql:


CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);

02-InsertData.sql :

INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');


INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');

put sql files in /docker-entrypoint-initdb.d on docker;
When rerun, it deletes the files because they are not permanent.

### 7. Use Volumes to Persist Data

To persist data and ensure it is not lost when the container is removed, use volumes. This will map a directory on your host machine to the container:

```sh
docker run --name mypostgres -d --network app-network -v ./data:/docker-entrypoint-initdb.d my_postgres
```

### 8. Verify SQL Files in Container

Ensure your SQL files are located in the `./data` directory on your host. These files will be copied to the container's `/docker-entrypoint-initdb.d` directory when the container starts. You can verify this by checking the contents of the directory within the container.

### 9. Recreate the PostgreSQL Container

If you need to make changes to the SQL files and want them to be reloaded, remove the existing container and recreate it:

```sh
docker rm -f mypostgres
docker run --name mypostgres -d --network app-network -v ./data:/docker-entrypoint-initdb.d my_postgres
```

### 10. Updated Dockerfile with COPY Instruction

Ensure your `Dockerfile` includes the instruction to copy the SQL files:

```dockerfile
FROM postgres:14.1-alpine

# Copy the SQL scripts into the container
COPY ./data /docker-entrypoint-initdb.d

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

### 11. Add SQL Files to Entrypoint

Ensure your SQL files are correctly added to the entrypoint directory `/docker-entrypoint-initdb.d`. This ensures they are executed when the container is started.

### 12. Verify Container and Data Persistence

After recreating the container, verify that the SQL files have been correctly copied into the container and are persistent across container restarts. You can check the container's `/docker-entrypoint-initdb.d` directory.

## Conclusion

You now have a PostgreSQL database running in a Docker container, connected to an Adminer interface for easy database management. The use of Docker volumes ensures that your data persists across container restarts and recreations.


### Backend API

#### Étape 1: Exécution d'une Classe Java Hello World

**1. Création du fichier `Main.java`:**

```java
public class Main {
   public static void main(String[] args) {
       System.out.println("Hello World!");
   }
}
```

**2. Compilation avec la version Java cible:** `javac Main.java`.
Si besoin il faut réinstaller java avec une version plus récente. Dans mon cas, java avait du mal à se mettre sur mon pc.

**3. Création du Dockerfile:**

```Dockerfile
# Utilisation d'une image Java pour exécuter un environnement Java runtime
FROM openjdk:17-rc-oraclelinux8

# Création d'un répertoire de travail
WORKDIR /app

# Copie du fichier .class compilé dans le conteneur
COPY Main.class /app

# Exécution de l'application Java
CMD ["java", "Main"]
```

**Explication:** Dans cette étape, nous avons écrit une classe Java simple qui affiche "Hello World!" sur la console. Ensuite, nous l'avons compilée en utilisant le compilateur Java (`javac`) pour générer un fichier de classe exécutable. Ensuite, nous avons créé un Dockerfile qui utilise une image OpenJDK pour exécuter notre application Java Hello World dans un conteneur Docker.

**4. Construction de l'image Docker:**

```bash
docker build -t java-hello-world .
```

**5. Exécution du conteneur Docker:**

```bash
docker run --name java-hello-world-container --network app-network java-hello-world
```

**Explication:** Nous avons construit une image Docker à partir du Dockerfile et ensuite exécuté un conteneur Docker à partir de cette image. Le message "Hello World!" devrait être affiché dans la sortie de la console.

#### Étape 2: Construction d'une Application Spring Boot avec une API Simple

**1. Création d'une application Spring Boot avec Spring Initializr:**

Nous avons utilisé Spring Initializr pour générer une nouvelle application Spring Boot avec les dépendances nécessaires pour créer une API Web simple.

**2. Implémentation du contrôleur `GreetingController`:**

Nous avons créé une classe `GreetingController` qui définit un point de terminaison REST simple qui renvoie une salutation avec un paramètre optionnel pour le nom.

package fr.takima.training.simpleapi.controller;

import org.springframework.web.bind.annotation.*;

import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {

   private static final String template = "Hello, %s!";
   private final AtomicLong counter = new AtomicLong();

   @GetMapping("/")
   public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
       return new Greeting(counter.incrementAndGet(), String.format(template, name));
   }

   record Greeting(long id, String content) {}

}


**3. Création du Dockerfile pour l'application Spring Boot:**

```Dockerfile
# Étape de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Étape d'exécution
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

**Explication:** Dans ce Dockerfile, nous utilisons une image Maven pour compiler notre application Spring Boot. Nous copions d'abord le fichier `pom.xml` et le répertoire `src` contenant notre code source. Ensuite, nous exécutons la commande Maven `mvn package` pour construire le jar exécutable de notre application. Enfin, nous utilisons une image Amazon Corretto pour exécuter notre application Spring Boot.

**4. Construction de l'image Docker:**

```bash
docker build -f Dockerfile -t simpleapi .
```

**5. Exécution du conteneur Docker:**

```bash
docker run -p 80:8080 simpleapi
```

#### Étape 3: Connexion à la Base de Données

**1. Ajustement de la configuration dans `simple-api/src/main/resources/application.yml`:**

Nous avons ajusté le fichier `application.yml` pour configurer la connexion à la base de données. Nous avons fourni l'URL, le nom d'utilisateur et le mot de passe pour se connecter à la base de données PostgreSQL.

**2. Exécution du conteneur Docker:**

Nous avons exécuté le conteneur Docker avec les modifications apportées à la configuration pour nous assurer que l'application est correctement liée à la base de données. Une fois que tout est correctement lié, nous devrions pouvoir accéder à notre API d'application, par exemple sur : `/departments/IRC/students`.

```json
[
  {
    "id": 1,
    "firstname": "Eli",
    "lastname": "Copter",
    "department": {
      "id": 1,
      "name": "IRC"
    }
  }
]
```

**Explication:** Pour que notre application Spring Boot puisse accéder à la base de données, nous devons configurer les paramètres de connexion dans le fichier `application.yml`. Ensuite, en exécutant le conteneur Docker, nous nous assurons que tout est correctement lié et que notre API fonctionne correctement.


### Http Server

#### Basics

**1. Choose an appropriate base image:**

We'll start by selecting an appropriate base image for our HTTP server. In this case, we'll use the `httpd:2.4-alpine` image, which provides an Apache HTTP server.

**2. Create a simple landing page: `index.html`**

We'll create a simple HTML landing page named `index.html` and place it inside our container. Here's the content of the `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Welcome to My Httpd Docker Container!</title>
</head>

<body>
  <header>
    <h1>Welcome to My Httpd Docker Container!</h1>
  </header>

  <main>
    <p>This is a simple landing page served by the Httpd (Apache) server inside a Docker container.</p>
  </main>
</body>

</html>
```

**3. Dockerfile:**

```Dockerfile
FROM httpd:2.4-alpine

COPY index.html /usr/local/apache2/htdocs/
```

**4. Build the Docker image:**

```bash
docker build -t my-httpd-image .
```

**5. Run the container:**

```bash
docker run --name my-httpd-container -p 82:80 -d my-httpd-image
```

**6. Verify everything is working:**

You can check that everything is working as expected using the following commands:

```bash
docker stats
docker inspect my-httpd-container
docker logs my-httpd-container
```

#### Reverse Proxy

**1. Retrieve the default configuration from the running container:**

We can use `docker exec` to retrieve the default configuration file from the running container at `/usr/local/apache2/conf/httpd.conf`.

```bash
docker exec -it my-httpd-container cat /usr/local/apache2/conf/httpd.conf
```

**2. Modify the default configuration:**

Add the following configuration in  /usr/local/apache2/conf/httpd.conf to enable reverse proxy:

```apache

<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://simple-api-container-student:8080/
ProxyPassReverse / http://simple-api-container-student:8080/
</VirtualHost>
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

```

**3. Update the Dockerfile to include the modified configuration:**

```Dockerfile
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
```

#### Tip: Why do we need a reverse proxy?

A reverse proxy can be useful for several reasons, including:

- Serving front-end applications.
- Configuring SSL termination.
- Handling load balancing.

#### Notes

At this point, we have a working HTTP server serving a simple landing page, and we've configured it as a reverse proxy to forward requests to a backend application.

### Docker-Compose Setup and Workflow

#### Basics of Docker-Compose

Docker-Compose is a tool that allows you to define and manage multi-container Docker applications. It uses a YAML file to configure the application’s services, networks, and volumes. In our case, we are using Docker-Compose to orchestrate the backend API, database, and HTTP server containers.

#### Setting Up Docker-Compose

**1. Install Docker-Compose:**

If the `docker-compose` command is not available on your system, you need to install Docker-Compose. Follow the installation instructions provided in the Docker documentation for your operating system.

**2. Create `docker-compose.yml`:**

We created a `docker-compose.yml` file to define our multi-container application. Below is the content of our Docker-Compose configuration:

```yaml
version: '3.7'

services:
  backend:
    build:
      context: "C:\\Users\\Dell\\OneDrive - Fondation EPF\\Documents\\4A semestre 2 EPF\\DevOps\\backend api\\simple-api-student-main"
    container_name: "simple-api-container-student2"
    networks:
      - app-network
    depends_on:
      - database
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://mypostgres2:5432/db
      - SPRING_DATASOURCE_USERNAME=usr
      - SPRING_DATASOURCE_PASSWORD=pwd
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
      - SPRING_JPA_DATABASE_PLATFORM=org.hibernate.dialect.PostgreSQLDialect

  database:
    build:
      context: "C:\\Users\\Dell\\OneDrive - Fondation EPF\\Documents\\4A semestre 2 EPF\\DevOps\\postgres"
    container_name: "mypostgres2"
    networks:
      - app-network
    environment:
      - POSTGRES_DB=db
      - POSTGRES_USER=usr
      - POSTGRES_PASSWORD=pwd

  httpd:
    build:
      context: "C:\\Users\\Dell\\OneDrive - Fondation EPF\\Documents\\4A semestre 2 EPF\\DevOps\\http"
    ports:
      - "82:80"
    networks:
      - app-network
    depends_on:
      - backend
      - database

networks:
  app-network:
```

#### Explanation of the `docker-compose.yml` File

- **Backend Service:**
  - **Build Context:** Specifies the path to the backend API code.
  - **Container Name:** Names the container `simple-api-container-student2`.
  - **Networks:** Connects to `app-network`.
  - **Depends On:** Specifies that the backend service depends on the `database` service.
  - **Environment Variables:** Configures the Spring Boot application to connect to the PostgreSQL database.

- **Database Service:**
  - **Build Context:** Specifies the path to the PostgreSQL Dockerfile.
  - **Container Name:** Names the container `mypostgres2`.
  - **Networks:** Connects to `app-network`.
  - **Environment Variables:** Sets the database name, user, and password.

- **HTTPD Service:**
  - **Build Context:** Specifies the path to the HTTP server configuration.
  - **Ports:** Maps port 82 on the host to port 80 in the container.
  - **Networks:** Connects to `app-network`.
  - **Depends On:** Specifies that the HTTPD service depends on both `backend` and `database` services.

- **Network:**
  - **app-network:** Creates a custom network for inter-container communication.

#### Running the Application with Docker-Compose

To start the application using Docker-Compose, run the following command:

```bash
docker-compose up
```

This command will build and start all the services defined in the `docker-compose.yml` file.

#### Publishing Docker Images to Docker Hub

**1. Tag the Image:**

For this we need a account dockerhub.
To publish the Docker image to Docker Hub, we first tagged the image with a meaningful version:

```bash
docker tag devops-database justinesmmt/my-database:1.0
```
"justinesmmt" is the account username.
"devops-database" is the name of the local Docker image you built earlier with dosker-compose.

**2. Push the Image:**

Next, we pushed the tagged image to Docker Hub:

```bash
docker push justinesmmt/my-database:1.0
```

After pushing the image, it is available in the Docker Hub repository under our account.


#### Documentation for Docker Hub

It's important to provide documentation for your Docker images on Docker Hub. This documentation should include details about the image, usage instructions, and any relevant configuration information. This makes it easier for team members or others to use the images effectively.

#### Why Use Docker-Compose?

Docker-Compose simplifies the process of managing multi-container applications. It allows you to define, build, and run all services with a single command, making it easier to manage dependencies and configurations.

#### Why Publish Docker Images?

Publishing Docker images to an online repository like Docker Hub makes it easy to share and distribute images. This allows team members to pull and run the images on their machines, ensuring consistency across different environments. It also facilitates continuous integration and deployment workflows.

By following these steps, we have successfully set up and managed a multi-container application using Docker-Compose, and published our Docker images to Docker Hub for easy distribution and use.

### TP2 



#### Steps to Set Up CI with GitHub Actions

1. **Create a New GitHub Repository and Push Your Project**:

    1.1 **Create a New Repository**:
    - Go to [GitHub](https://github.com/) and create a new repository named `Devops`.

    1.2 **Add Your Project to the Repository**:
    - Open your terminal and navigate to your project directory.
    - Initialize the Git repository if it's not already done:
      ```bash
      git init
      ```
    - Add all your project files to the repository:
      ```bash
      git add .
      ```
    - Make an initial commit:
      ```bash
      git commit -m "Initial commit"
      ```
    - Add the remote repository:
      ```bash
      git remote add origin https://github.com/justinesmmt/Devops.git
      ```
    - Push your project to the GitHub repository:
      ```bash
      git push -u origin main
      ```

### Detailed Instructions for Setting Up and Using Maven

To build and test your Java application using Maven, follow these steps:

#### Step 1: Download and Install Maven

1. **Download Maven**:
   - Go to the [Maven download page](https://maven.apache.org/download.cgi).
   - Download the latest binary zip archive (e.g., `apache-maven-3.9.7-bin.zip`).

2. **Extract Maven**:
   - Extract the downloaded zip file to a directory of your choice, for example, `C:\apache-maven-3.9.7`.

3. **Set Up Environment Variables**:
   - Add `C:\apache-maven-3.9.7\bin` to your system's `PATH` environment variable:
     1. Open the **Start Menu** and search for "Environment Variables".
     2. Select "Edit the system environment variables".
     3. In the System Properties window, click on the **Environment Variables** button.
     4. In the Environment Variables window, under **System variables**, find the `Path` variable and select it. Click **Edit**.
     5. In the Edit Environment Variable dialog, click **New** and add the path to the Maven `bin` directory (`C:\apache-maven-3.9.7\bin`).
     6. Click **OK** to close all dialogs.

4. **Verify Maven Installation**:
   - Open a new command prompt and run:
     ```bash
     mvn -v
     ```
   - This should display the Maven version and other environment details, confirming that Maven is installed correctly.

#### Step 2: Build and Test Your Application

1. **Navigate to Your Project Directory**:
   - Open a command prompt and change to the directory containing your `pom.xml` file:
     ```bash
     cd path\to\your\project
     ```

2. **Run Maven Build and Tests**:
   - Execute the following command to clean, build, and test your application:
     ```bash
     mvn clean verify
     ```
   - If your `pom.xml` is not in the current directory, you can specify the path to it:
     ```bash
     mvn clean verify --file /path/to/pom.xml
     ```

#### Explanation of `mvn clean verify`

- **`mvn clean`**:
  - This command removes all files generated by the previous build. This includes compiled classes, JAR files, and other build artifacts. Cleaning ensures that any remnants from previous builds are removed, preventing unexpected behavior due to stale artifacts.

- **`mvn verify`**:
  - This command executes a full build of the project. It compiles the source code, processes resources, packages the compiled code into JARs or other artifacts, and runs the tests.
  - **Unit Tests**: These are small, isolated tests that verify the functionality of individual units of code (e.g., methods or classes). They are usually fast and cover edge cases.
  - **Integration Tests (Component Tests)**: These tests check the interactions between different parts of the application. They ensure that components work together as expected and often involve a larger scope, such as database interactions or external service calls.

#### 1. What are Testcontainers?

**Testcontainers** are Java libraries that allow developers to run Docker containers within their tests. They provide lightweight, throwaway instances of databases, message brokers, and other services, making integration testing more reliable and easier to set up. For example, in our project, we use the PostgreSQL Testcontainer to run a PostgreSQL instance during tests.



#### 2. Setting Up Continuous Integration with GitHub Actions

##### Step 1: Create Your `main.yml` File

First, create a `.github/workflows` directory in your project repository. This directory will contain the configuration files for GitHub Actions.

```plaintext
mkdir -p .github/workflows
```

Create a file named `main.yml` inside the `.github/workflows` directory with the following content:

```yaml
name: CI devops 2024

on:
  # To begin you want to launch this job in main and develop
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      # Checkout your GitHub code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

      # Setup JDK 17 using actions/setup-java@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'

      # Build and test with Maven
      - name: Build and test with Maven
        run: mvn clean install
        working-directory: ./backend api/simple-api-student-main
```

##### Step 2: Verify CI Workflow on GitHub

1. Push the `main.yml` file to your GitHub repository:
   
    ```bash
    git add .github/workflows/main.yml
    git commit -m "Add GitHub Actions CI workflow"
    git push origin main
    ```

2. Go to the "Actions" tab in your GitHub repository. You should see your workflow running. If everything is set up correctly, the workflow will complete successfully, indicated by a green checkmark.

### Step-by-Step Guide: Setting Up Continuous Delivery with GitHub Actions and Docker Hub

#### Adding Docker Hub Credentials to GitHub Secrets

1. **Go to Your GitHub Repository Settings**:
   - Navigate to your GitHub repository and click on the "Settings" tab.

2. **Access Secrets and Variables**:
   - In the left-hand menu, click on "Secrets and variables".

3. **Add a New Repository Secret**:
   - Click on the "New repository secret" button.

4. **Enter Your Docker Hub Username**:
   - In the "Name" field, enter `DOCKERHUB_USERNAME`.
   - In the "Value" field, enter your Docker Hub username.
   - Click on the "Add secret" button to save the secret.

5. **Enter Your Docker Hub Password**:
   - Click on the "New repository secret" button again.
   - In the "Name" field, enter `DOCKERHUB_PASSWORD`.
   - In the "Value" field, enter your Docker Hub password.
   - Click on the "Add secret" button to save the secret.

### Setting Up Continuous Delivery with GitHub Actions: Building and Pushing Docker Images

#### Adding Docker Hub Credentials to GitHub Secrets

1. **Go to Your GitHub Repository Settings**:
   - Navigate to your GitHub repository and click on the "Settings" tab.

2. **Access Secrets and Variables**:
   - In the left-hand menu, click on "Secrets and variables".

3. **Add a New Repository Secret**:
   - Click on the "New repository secret" button.

4. **Enter Your Docker Hub Username**:
   - In the "Name" field, enter `DOCKERHUB_USERNAME`.
   - In the "Value" field, enter your Docker Hub username.
   - Click on the "Add secret" button to save the secret.

5. **Enter Your Docker Hub Password**:
   - Click on the "New repository secret" button again.
   - In the "Name" field, enter `DOCKERHUB_PASSWORD`.
   - In the "Value" field, enter your Docker Hub password.
   - Click on the "Add secret" button to save the secret.

### Configuring Continuous Delivery with GitHub Actions

#### Step 1: Update Your `main.yml` File

Enhance your `main.yml` file to include steps for logging in to Docker Hub, building Docker images, and pushing them to Docker Hub. Here’s how you can do it:

```yaml
name: CI/CD devops 2024

on:
  push:
    branches:
      - main
      - develop
  pull_request:

jobs:
  test-backend:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'
      
      - name: Build and test with Maven
        run: mvn clean install
        working-directory: ./backend api/simple-api-student-main

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v2.5.0

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

      - name: Build and push backend image
        uses: docker/build-push-action@v3
        with:
          context: ./backend api/simple-api-student-main
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push database image
        uses: docker/build-push-action@v3
        with:
          context: ./postgres
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build and push httpd image
        uses: docker/build-push-action@v3
        with:
          context: ./http
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-httpd:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```

### Explanation of the Workflow Configuration

1. **Triggers (`on`)**:
   - `push`: The workflow triggers on pushes to `main` and `develop` branches.
   - `pull_request`: The workflow triggers on pull request events.

2. **Jobs**:
   - **test-backend**:
     - **runs-on**: Specifies the runner, in this case, `ubuntu-22.04`.
     - **steps**:
       - **actions/checkout@v2.5.0**: Checks out the code from the repository.
       - **actions/setup-java@v3**: Sets up JDK 17.
       - **Build and test with Maven**: Runs `mvn clean install` to build and test the backend.

   - **build-and-push-docker-image**:
     - **needs**: Specifies that this job depends on the successful completion of `test-backend`.
     - **runs-on**: Specifies the runner, in this case, `ubuntu-22.04`.
     - **steps**:
       - **actions/checkout@v2.5.0**: Checks out the code from the repository.
       - **Log in to Docker Hub**: Logs in to Docker Hub using the secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`.
       - **Build and push backend image**: Builds and pushes the backend Docker image to Docker Hub.
       - **Build and push database image**: Builds and pushes the database Docker image to Docker Hub.
       - **Build and push httpd image**: Builds and pushes the httpd Docker image to Docker Hub.

### Setting Up Continuous Delivery and Quality Gate with GitHub Actions

In this step, you will learn how to configure your GitHub Actions pipeline to build and publish Docker images to Docker Hub on every commit to the main branch, as well as set up SonarCloud for quality gate analysis.

### 3- Publish Your Docker Images on Main Branch Commit

#### Updating `main.yml` for Docker Image Build and Push

To publish Docker images, you need to ensure you log in to Docker Hub and push the images only on commits to the main branch. Here’s the updated `main.yml` configuration:

```yaml
name: CI/CD devops 2024

on:
  push:
    branches: 
      - main
      - develop
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'adopt'

      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=justine-smmt -Dsonar.organization=sammut-justine -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file pom.xml
        working-directory: ./backend api/simple-api-student-main

  build-and-push-docker-image:
    needs: test-backend
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          context: ./backend api/simple-api-student-main
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api-backend:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          context: ./postgres
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api-database:latest
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          context: ./http
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/tp-devops-simple-api-httpd:latest
          push: ${{ github.ref == 'refs/heads/main' }}
```

### Explanation

1. **Docker Login**:
   - The `Log in to Docker Hub` step uses `docker/login-action@v2` to log in to Docker Hub using the credentials stored in GitHub Secrets.

2. **Build and Push Docker Images**:
   - Each `docker/build-push-action@v3` step builds and pushes a Docker image.
   - The `context` specifies the location of the Dockerfile.
   - The `tags` specify the Docker image tag, and the `push` parameter ensures the image is pushed only when the commit is on the `main` branch.

### Verifying Docker Images

1. **Push the Configuration**:
   ```bash
   git add .github/workflows/main.yml
   git commit -m "Add Docker build and push steps to GitHub Actions"
   git push origin main
   ```

2. **Check GitHub Actions**:
   - Navigate to the "Actions" tab in your GitHub repository to verify the workflow execution.
   - Ensure the workflow builds and pushes Docker images to Docker Hub on commits to the `main` branch.

### Setting Up Quality Gate with SonarCloud

#### Register and Configure SonarCloud

1. **Create a SonarCloud Account**:
   - Go to [SonarCloud](https://sonarcloud.io) and create a free-tier account.
   - Create an organization and a project. Note down the `project key` and `organization key`.

2. **Add SonarCloud Token to GitHub Secrets**:
   - In your GitHub repository settings, add a new secret named `SONAR_TOKEN` and paste your SonarCloud token.

#### Updating `main.yml` for SonarCloud Analysis

Modify the Maven build step to include SonarCloud analysis:

```yaml
      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=devops-2024 -Dsonar.organization=devops-school -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./simple-api/pom.xml
        working-directory: ./backend api/simple-api-student-main
```

### Explanation

- The `Build and test with Maven` step now includes parameters for SonarCloud analysis.
  - `-Dsonar.projectKey`: Your SonarCloud project key.
  - `-Dsonar.organization`: Your SonarCloud organization key.
  - `-Dsonar.host.url`: The SonarCloud URL.
  - `-Dsonar.login`: Your SonarCloud token stored in GitHub Secrets.

### Verifying SonarCloud Analysis

1. **Push the Configuration**:
   ```bash
   git add .github/workflows/main.yml
   git commit -m "Add SonarCloud analysis to GitHub Actions"
   git push origin main
   ```

2. **Check GitHub Actions**:
   - Navigate to the "Actions" tab in your GitHub repository to verify the workflow execution.
   - Ensure the workflow performs SonarCloud analysis during the build process.

3. **Check SonarCloud**:
   - Go to your SonarCloud dashboard to view the analysis report for your project.

### Summary

By following these steps, you have configured a CI/CD pipeline that:
- Builds and pushes Docker images to Docker Hub on commits to the `main` branch.
- Integrates SonarCloud for code quality analysis, ensuring a maintainable and secure codebase.

## TD3

## Introduction

This guide will walk you through the installation and configuration of Ansible on a machine running Windows Linux Subsystem (WSL). You will learn how to use Ansible to manage and provision servers, as well as run simple commands to check the connectivity and configuration of your server.

## Prerequisites

1. **Windows Linux Subsystem (WSL)**: Ensure that WSL is installed and configured on your Windows machine.
2. **Ansible**: We will install Ansible in WSL.
3. **SSH Private Key**: You will need your private key to connect to your remote server.

## Installation and Configuration Steps

### 1. Install Ansible

Start by installing Ansible in your WSL environment:

```sh
pip3 install ansible
```

### 2. Verify Ansible Installation

Verify the installation by displaying the Ansible version:

```sh
ansible --version
```

### 3. Configure Private Key Permissions

To use your private key, you need to restrict its permissions:

```sh
chmod 400 '/mnt/c/Users/Dell/OneDrive - Fondation EPF/documents/4a semestre 2 EPF/Devops/id_rsa'
```

### 4. Connect to the Remote Server via SSH

Connect to your remote server using SSH and the private key:

```sh
cd ~/.ssh
sudo chmod 400 id_rsa
ssh -i id_rsa centos@justine.sammut.takima.cloud
```

After verifying the connection, you can exit the SSH session:

```sh
exit
```

### 5. Update and Install Packages on the Remote Server

Once connected to your server, update the packages and install the necessary tools:

```sh
sudo yum update
sudo yum install vim -y
sudo yum install epel-release
sudo yum install ansible
```

### 6. Configure Ansible Hosts

Back in your WSL environment, create and edit the Ansible hosts file:

```sh
sudo vim /etc/ansible/hosts
```

Add your server to the `/etc/ansible/hosts` file:

```sh
justine.sammut.takima.cloud
```

### 7. Test Configuration with Ansible Ping Command

Test the connection and Ansible configuration with the following command:

```sh
ansible all -m ping --private-key=~/.ssh/id_rsa -u centos
```

You should receive a response like:

```json
justine.sammut.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

## TP3

## Introduction

This guide will walk you through setting up a project-specific inventory for Ansible and executing some basic commands to interact with your remote server. You will learn how to create the necessary directories and files, configure your inventory, and use Ansible modules to gather information and manage your server.

## Prerequisites

1. **Windows Linux Subsystem (WSL)**: Ensure that WSL is installed and configured on your Windows machine.
2. **Ansible**: We will install Ansible in WSL.
3. **SSH Private Key**: You will need your private key to connect to your remote server.

## Steps

### 1. Create Project Directories and Files

Start by creating the required directories and files for your Ansible project:

```sh
mkdir -p /mnt/c/Users/Dell/OneDrive\ -\ Fondation\ EPF/documents/4a\ semestre\ 2\ EPF/Devops/my-project/ansible/inventories
```

Navigate to your project directory:

```sh
cd /mnt/c/Users/Dell/OneDrive\ -\ Fondation\ EPF/documents/4a\ semestre\ 2\ EPF/Devops/my-project
```

Create the `ansible` directory:

```sh
mkdir ansible
cd ansible
```

Create the `inventories` directory:

```sh
mkdir inventories
cd inventories
```

Create the `setup.yml` file:

```sh
touch setup.yml
```

### 2. Configure the Inventory

Edit the `setup.yml` file and add the following configuration:

```yaml
all:
  vars:
    ansible_user: centos
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts: justine.sammut.takima.cloud
```

### 3. Test the Inventory with the Ping Command

Use the `ping` module to test the connectivity with your server:

```sh
ansible all -i inventories/setup.yml -m ping
```

Expected output:

```json
justine.sammut.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

### 4. Gather Facts About the Host

Use the `setup` module to gather information about your host:

```sh
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

Expected output:

```json
justine.sammut.takima.cloud | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "CentOS",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/redhat-release",
        "ansible_distribution_file_variety": "RedHat",
        "ansible_distribution_major_version": "7",
        "ansible_distribution_release": "Core",
        "ansible_distribution_version": "7.9",
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false
}
```



## Steps

### 1. Create a Basic Playbook

Start by creating a simple playbook to test the connection to your server.

Create the `playbook.yml` file:

```sh
cd /mnt/c/Users/Dell/OneDrive\ -\ Fondation\ EPF/documents/4a\ semestre\ 2\ EPF/Devops/my-project/ansible
touch playbook.yml
```

Edit the `playbook.yml` file and add the following content:

```yaml
- hosts: all
  gather_facts: false
  become: true

  tasks:
   - name: Test connection
     ping:
```

Execute the playbook:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

Expected output:

```plaintext
PLAY [all] **************************************************************************************************************************

TASK [Test connection] **************************************************************************************************************
ok: [justine.sammut.takima.cloud]

PLAY RECAP **************************************************************************************************************************
justine.sammut.takima.cloud : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 2. Create an Advanced Playbook to Install Docker

Edit the `playbook.yml` file to install Docker on your server:

```sh
nano playbook.yml
```

Replace the existing content with:

```yaml
- hosts: all
  gather_facts: false
  become: true

  tasks:
    - name: Install device-mapper-persistent-data
      yum:
        name: device-mapper-persistent-data
        state: latest

    - name: Install lvm2
      yum:
        name: lvm2
        state: latest

    - name: Add Docker repository
      command:
        cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

    - name: Install Docker
      yum:
        name: docker-ce
        state: present

    - name: Install python3
      yum:
        name: python3
        state: present

    - name: Install Docker with Python 3
      pip:
        name: docker
        executable: pip3
      vars:
        ansible_python_interpreter: /usr/bin/python3

    - name: Make sure Docker is running
      service:
        name: docker
        state: started
      tags: docker
```

Execute the playbook:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

Expected output:

```plaintext
PLAY [all] **************************************************************************************************************************

TASK [Install device-mapper-persistent-data] ****************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Install lvm2] *****************************************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Add Docker repository] ********************************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Install Docker] ***************************************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Install python3] **************************************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Install Docker with Python 3] *************************************************************************************************
changed: [justine.sammut.takima.cloud]

TASK [Make sure Docker is running] **************************************************************************************************
changed: [justine.sammut.takima.cloud]

PLAY RECAP **************************************************************************************************************************
justine.sammut.takima.cloud : ok=7    changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 3. Refactor Using Roles

Create a Docker role for cleaner task management:

```sh
ansible-galaxy init roles/docker
```

Move the Docker installation tasks to the role:

Edit the `tasks/main.yml` file in the `roles/docker` directory and add the following content:

```yaml
---
# tasks file for roles/docker

- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: Add Docker repository
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install python3
  yum:
    name: python3
    state: present

- name: Install Docker with Python 3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Make sure Docker is running
  service:
    name: docker
    state: started
  tags: docker
```

Edit the `handlers/main.yml` file in the `roles/docker` directory and add the following content:

```yaml
---
# handlers file for roles/docker

- name: Restart Docker
  systemd:
    name: docker
    state: restarted
    enabled: yes
```

### 4. Update Playbook to Use Role

Edit the `playbook.yml` file to use the Docker role:

```yaml
- hosts: all
  gather_facts: false
  become: true

  roles:
    - role: docker
```

Execute the playbook to apply the role:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

## Conclusion

You have now successfully created and executed both simple and advanced Ansible playbooks, tested server connectivity, installed Docker, and refactored your tasks into roles for better organization. This setup will help you efficiently manage and provision your servers using Ansible.


# Documentation for Deploying Your Dockerized Application with Ansible

## Introduction

This guide walks through the steps to deploy a dockerized application using Ansible. We will create specific roles to handle different parts of the deployment process, including installing Docker, creating a network, launching the database, the application, and the proxy. Each part will use the `docker_container` module to start the respective Docker containers.

## Roles Overview

- **Install Docker**: Install Docker on the managed server.
- **Create Network**: Create a Docker network for the containers to communicate.
- **Launch Database**: Pull and run the database container.
- **Launch App**: Pull and run the application container.
- **Launch Proxy**: Pull and run the proxy container.

## Creating the Roles

1. **Create the roles using `ansible-galaxy`:**

```sh
ansible-galaxy init roles/network
ansible-galaxy init roles/database
ansible-galaxy init roles/launch_app
ansible-galaxy init roles/launch_proxy
```

## Playbook Structure

### Playbook

The main playbook, `playbook.yml`, which includes all the roles:

```yaml
- hosts: all
  gather_facts: false
  become: true

  roles:
    - docker
    - network
    - database
    - launch_app
    - launch_proxy
```

### Role: Docker

**Tasks file (`roles/docker/tasks/main.yml`):**

```yaml
---
# tasks file for roles/docker

- name: Install device-mapper-persistent-data
  yum:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  yum:
    name: lvm2
    state: latest

- name: add repo docker
  command:
    cmd: sudo yum-config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  yum:
    name: docker-ce
    state: present

- name: Install python3
  yum:
    name: python3
    state: present

- name: Install docker with Python 3
  pip:
    name: docker
    executable: pip3
  vars:
    ansible_python_interpreter: /usr/bin/python3

- name: Make sure Docker is running
  service:
    name: docker
    state: started
  tags: docker

- name: Log in to Docker Hub
  docker_login:
    username: justinesmmt
    password: 9Xdqn4A55jj,,Ai
    reauthorize: yes
```

### Role: Network

**Tasks file (`roles/network/tasks/main.yml`):**

```yaml
---
# tasks file for roles/network

- name: Create a network
  community.docker.docker_network:
    name: app-network
```

### Role: Database

**Tasks file (`roles/database/tasks/main.yml`):**

```yaml
---
# tasks file for roles/database

- name: Pull the BDD image
  docker_image:
    name: justinesmmt/tp-devops-simple-api-database
    tag: latest
    source: pull

- name: Run BDD
  docker_container:
    state: started
    name: mypostgres
    image: justinesmmt/tp-devops-simple-api-database
    networks: 
      - name: "app-network"
```

### Role: Launch App

**Tasks file (`roles/launch_app/tasks/main.yml`):**

```yaml
---
# tasks file for roles/launch_app

- name: Pull the API Image
  docker_image:
    name: justinesmmt/tp-devops-simple-api-backend
    tag: latest
    source: pull

- name: Run API
  docker_container:
    name: simple-api-container-student2
    image: justinesmmt/tp-devops-simple-api-backend:latest
    networks: 
      - name: "app-network"
    state: started
```

### Role: Launch Proxy

**Tasks file (`roles/launch_proxy/tasks/main.yml`):**

```yaml
---
# tasks file for roles/launch_proxy

- name: Pull the proxy container
  docker_image:
    name: justinesmmt/tp-devops-simple-api-httpd
    tag: latest
    source: pull
 
- name: Run the proxy container
  docker_container:
    name: httpd-1
    image: justinesmmt/tp-devops-simple-api-httpd
    ports:
      - "80:80"
    networks:
      - name: "app-network"
```

## Executing the Playbook

Run the playbook to deploy the application:

```sh
ansible-playbook -i inventories/setup.yml playbook.yml
```

### Verifying the Deployment

Once the playbook runs successfully, verify the deployment by accessing the API endpoints:

- Check the base endpoint:

  ```sh
  http://justine.sammut.takima.cloud
  ```

  Expected response:

  ```json
  {
    "id": 4,
    "content": "Hello, World!"
  }
  ```

- Check the departments endpoint:

  ```sh
  http://justine.sammut.takima.cloud/departments/IRC/students
  ```

  Expected response:

  ```json
  [
    {
      "id": 1,
      "firstname": "Eli",
      "lastname": "Copter",
      "department": {
        "id": 1,
        "name": "IRC"
      }
    }
  ]
  ```

## Conclusion

You have now successfully set up and deployed your dockerized application using Ansible. This guide covers the creation of roles, configuring tasks for Docker, and running the playbook to deploy the application on a managed server. This structured approach ensures a clean and maintainable deployment process.