# demo-api-first
An API first application using Spring Boot and OpenAPI 3

- Article: https://medium.com/xgeeks/api-first-using-openapi-and-spring-boot-2602c04bb0d3

- OpenAPI mock server with random data generation : https://github.com/sanogotech/openapi-mock

# **API First avec OpenAPI et Spring Boot**

### Introduction

Dans le développement logiciel moderne, il existe plusieurs approches permettant de créer des API efficaces et maintenables. L’une des meilleures pratiques est l'approche **API First**, qui place la conception de l’API au cœur du développement, avant même l'implémentation des services backend. Une méthode populaire pour définir des API est **OpenAPI** (anciennement Swagger), qui permet de définir de manière structurée et lisible les spécifications des API REST. Dans ce cours, nous explorerons comment l'approche **API First** peut être mise en œuvre avec **OpenAPI** et **Spring Boot**, en générant automatiquement les objets DTO, les interfaces REST, et en facilitant la collaboration entre les équipes de développement backend et frontend.

### Objectifs du Cours

1. **Comprendre le concept d'API First et son importance**.
2. **Découvrir comment utiliser OpenAPI pour définir une API de manière déclarative**.
3. **Apprendre à générer automatiquement des DTO et des interfaces REST avec OpenAPI et Spring Boot**.
4. **Explorer l'intégration de OpenAPI avec Spring Boot pour automatiser la documentation et les tests**.
5. **Faciliter la collaboration entre les équipes grâce à une spécification commune et des outils de génération automatiques de clients**.

### Prérequis

- Connaissances de base en **Java** et **Spring Boot**.
- Familiarité avec les **APIs RESTful**.
- Compréhension de base de l'outil **Swagger/OpenAPI**.
- Installation de **Maven** ou **Gradle** pour la gestion des dépendances.

---

### Partie 1 : Introduction à OpenAPI

OpenAPI est une spécification qui permet de définir de manière standardisée les interfaces d'APIs RESTful. Elle offre une représentation lisible et compréhensible pour les humains, mais aussi pour les machines, permettant ainsi de générer automatiquement des clients, des tests et de la documentation.

#### 1.1 Définition et But de l'OpenAPI

L'OpenAPI Specification (OAS) définit une interface standard pour les APIs REST. Cela permet aux développeurs de concevoir et de partager des API avant même leur implémentation. Une API correctement définie avec OpenAPI permet à d'autres équipes ou applications de comprendre comment interagir avec les services sans avoir à examiner le code source.

#### 1.2 Structure d’un Fichier OpenAPI

Un fichier OpenAPI peut être écrit en **YAML** ou en **JSON** et contient les éléments suivants :

- **Paths** : Les différents points d'entrée de l'API, comme `/users` ou `/users/{id}`.
- **Methods HTTP** : Comme GET, POST, PUT, DELETE, etc., utilisés pour interagir avec chaque path.
- **Paramètres** : Les paramètres passés dans l'URL, dans le corps de la requête ou les en-têtes.
- **Composants** : Définition des objets réutilisables, tels que les DTOs, les réponses standardisées, les schémas de données.

#### 1.3 Exemple de Spécification OpenAPI

Voici un exemple d'un fichier OpenAPI pour une API simple de gestion d'utilisateurs :

```yaml
openapi: 3.0.0
info:
  title: API de gestion des utilisateurs
  description: API pour la gestion des utilisateurs dans le système
  version: 1.0.0
paths:
  /users:
    get:
      summary: Récupère la liste des utilisateurs
      responses:
        '200':
          description: Liste des utilisateurs
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
    post:
      summary: Crée un nouvel utilisateur
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/User'
      responses:
        '201':
          description: Utilisateur créé
  /users/{id}:
    get:
      summary: Récupère un utilisateur par son identifiant
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Informations de l'utilisateur
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: Utilisateur non trouvé

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
```

---

### Partie 2 : API First avec Spring Boot

#### 2.1 Architecture d’un Projet API First

Dans un projet API First, nous avons une **spécification OpenAPI** qui définit l’API, puis nous générons automatiquement les objets DTO et les interfaces REST. L'implémentation de l’API elle-même se fait dans un autre module qui dépend de la spécification générée.

Voici un exemple d'architecture de projet basée sur OpenAPI et Spring Boot :

```
- parent-project
  - openapi-spec (module pour la spécification OpenAPI)
    - openapi.yaml
  - api-impl (module pour l'implémentation de l'API)
    - UserApi.java
    - UserApiController.java
    - UserService.java
  - pom.xml (fichier de gestion des dépendances)
```

#### 2.2 Génération Automatique des DTO et Interfaces avec OpenAPI Generator

OpenAPI Generator est un outil qui permet de générer automatiquement des clients, des serveurs, des DTOs, et des interfaces à partir d'une spécification OpenAPI. Pour utiliser ce générateur avec **Maven**, nous ajoutons le plugin suivant dans le `pom.xml` :

```xml
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>5.0.0</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <inputSpec>${project.basedir}/openapi-spec/openapi.yaml</inputSpec>
        <generatorName>spring</generatorName>
        <output>${project.build.directory}/generated-sources</output>
    </configuration>
</plugin>
```

Cela génère automatiquement les fichiers Java nécessaires à l'implémentation de l’API, comme les interfaces et les modèles de données.

#### 2.3 Implémentation avec Spring Boot

Une fois que les interfaces et les DTOs sont générés, il suffit de les utiliser dans votre code Spring Boot. Par exemple, dans le cas de l’API utilisateur, vous devrez implémenter l'interface `UserApi` générée :

```java
@RestController
public class UserApiController implements UserApi {

    @Autowired
    private UserService userService;

    @Override
    public ResponseEntity<User> getUserById(String id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }

    @Override
    public ResponseEntity<List<User>> getUsers() {
        List<User> users = userService.findAll();
        return ResponseEntity.ok(users);
    }

    @Override
    public ResponseEntity<Void> createUser(User user) {
        userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

---

### Partie 3 : Collaboration Frontend et Backend

L'une des grandes forces de l'approche **API First** est la possibilité de faire avancer simultanément les équipes **frontend** et **backend**. Le frontend peut commencer à travailler sur les clients API et l'intégration de l'interface utilisateur sans attendre que l'API soit entièrement implémentée. Voici quelques outils qui facilitent cette collaboration :

#### 3.1 Génération Automatique de Clients Frontend

Les développeurs frontend peuvent utiliser des générateurs de clients à partir de la spécification OpenAPI pour générer des appels API dans leur code, ce qui garantit la cohérence avec le backend.

Voici un exemple avec un générateur de client pour **React** utilisant **Swagger Codegen** :

```bash
swagger-codegen generate -i openapi.yaml -l javascript -o ./generated-client
```

Cela génère un client API React avec les méthodes nécessaires pour interagir avec les endpoints de l’API définie.

#### 3.2 Serveur Mock pour Tester les Endpoints

Pendant que le backend n'est pas encore complètement implémenté, le frontend peut utiliser un **serveur mock** pour tester les appels API avec des données aléatoires générées selon les schémas de la spécification OpenAPI. Un outil populaire est **MockServer**, qui permet de simuler des réponses HTTP basées sur le fichier OpenAPI.

---

### Partie 4 : Avantages de l'Approche API First

#### 4.1 Cohérence et Collaboration

L’approche **API First** garantit que toutes les équipes (frontend, backend, et autres parties prenantes) travaillent avec la même spécification. Cela réduit les risques de divergences dans les attentes et améliore la collaboration.

#### 4.2 Test et Validation Automatiques

Grâce à des outils comme OpenAPI Generator et MockServer, il est possible de générer des tests et de valider rapidement les comportements des API, même avant que l’implémentation ne soit complète.

#### 4.3 Rapidité et Flexibilité

L’approche API First permet aux équipes de commencer à travailler sur leurs tâches respectives dès que la spécification est définie, sans attendre l’implémentation complète de l’API.

---

### Conclusion

L'approche **API First** avec **OpenAPI** et **Spring Boot** offre une méthode puissante et flexible pour concevoir, implémenter et tester des APIs. En permettant une collaboration fluide entre les équipes de développement backend et frontend, elle réduit les risques de mauvaises implémentations et accélère le développement des applications. L’utilisation de générateurs automatiques pour les clients et les serveurs, combinée à un serveur de tests mock, permet d'avancer rapidement et efficacement dans le développement de systèmes distribués.
