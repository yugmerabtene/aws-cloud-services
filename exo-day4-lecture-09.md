### Exercice sur le Développement et le Test d’une Application Serverless avec AWS et Serverless Framework

#### **Énoncé :**
Vous allez développer une petite application **serverless** qui gère des utilisateurs, avec des fonctions Lambda déployées sur AWS. Votre objectif est de créer une API REST qui permet :
- L’ajout d’un utilisateur.
- La récupération des informations d’un utilisateur.
- Le tout en utilisant DynamoDB comme base de données.
- Tester votre application à la fois **localement** et **sur le cloud**.

L'exercice comprendra les étapes suivantes :
1. Créer et configurer un projet **Serverless Framework**.
2. Développer deux fonctions Lambda : `addUser` et `getUser`.
3. Tester les fonctions localement avec **serverless-offline**.
4. Déployer et tester sur AWS.
5. Implémenter les tests unitaires et d'intégration avec **Jest** et **aws-sdk-client-mock**.
6. Évaluer la performance avec **Artillery**.

#### **Partie 1 : Mise en place du projet**
1. **Initialiser le projet Serverless Framework**
   - Installez le framework si ce n’est pas déjà fait :
     ```bash
     npm install -g serverless
     ```
   - Créez un nouveau projet :
     ```bash
     serverless create --template aws-nodejs --path projet-serverless
     cd projet-serverless
     ```

2. **Configuration du fichier `serverless.yml`**
   - Modifiez le fichier `serverless.yml` pour définir les ressources DynamoDB et les fonctions Lambda.
   
   ```yaml
   service: gestion-utilisateurs

   provider:
     name: aws
     runtime: nodejs14.x
     region: us-east-1
     environment:
       USERS_TABLE: usersTable

   resources:
     Resources:
       UsersTable:
         Type: AWS::DynamoDB::Table
         Properties:
           TableName: usersTable
           AttributeDefinitions:
             - AttributeName: id
               AttributeType: S
           KeySchema:
             - AttributeName: id
               KeyType: HASH
           BillingMode: PAY_PER_REQUEST

   functions:
     addUser:
       handler: handler.addUser
       events:
         - http:
             path: users
             method: post
     getUser:
       handler: handler.getUser
       events:
         - http:
             path: users/{id}
             method: get
   ```

#### **Partie 2 : Développement des fonctions Lambda**
Créez un fichier `handler.js` pour implémenter les fonctions `addUser` et `getUser`.

1. **Ajouter un utilisateur (`addUser`)**
   Cette fonction va recevoir un objet utilisateur (nom et email) via une requête POST et enregistrer cet utilisateur dans DynamoDB.

   ```javascript
   const { v4: uuidv4 } = require('uuid');
   const AWS = require('aws-sdk');
   const dynamoDb = new AWS.DynamoDB.DocumentClient();

   module.exports.addUser = async (event) => {
     const data = JSON.parse(event.body);
     const userId = uuidv4();

     const params = {
       TableName: process.env.USERS_TABLE,
       Item: {
         id: userId,
         name: data.name,
         email: data.email
       },
     };

     try {
       await dynamoDb.put(params).promise();
       return {
         statusCode: 200,
         body: JSON.stringify({ message: "Utilisateur ajouté avec succès", id: userId }),
       };
     } catch (error) {
       return {
         statusCode: 500,
         body: JSON.stringify({ error: "Impossible d'ajouter l'utilisateur" }),
       };
     }
   };
   ```

2. **Récupérer un utilisateur (`getUser`)**
   Cette fonction va récupérer les informations d'un utilisateur via une requête GET en utilisant l'ID de l'utilisateur.

   ```javascript
   module.exports.getUser = async (event) => {
     const userId = event.pathParameters.id;

     const params = {
       TableName: process.env.USERS_TABLE,
       Key: {
         id: userId,
       },
     };

     try {
       const result = await dynamoDb.get(params).promise();
       if (result.Item) {
         return {
           statusCode: 200,
           body: JSON.stringify(result.Item),
         };
       } else {
         return {
           statusCode: 404,
           body: JSON.stringify({ error: "Utilisateur non trouvé" }),
         };
       }
     } catch (error) {
       return {
         statusCode: 500,
         body: JSON.stringify({ error: "Erreur lors de la récupération de l'utilisateur" }),
       };
     }
   };
   ```

#### **Partie 3 : Tester localement avec `serverless-offline`**
1. **Installer `serverless-offline` :**
   ```bash
   npm install serverless-offline --save-dev
   ```

2. **Configurer le plugin dans `serverless.yml` :**
   ```yaml
   plugins:
     - serverless-offline
   ```

3. **Lancer le service localement :**
   ```bash
   serverless offline start
   ```
   Cela démarre un serveur local sur `http://localhost:3000`. Vous pouvez tester vos endpoints :

   - Ajouter un utilisateur :
     ```bash
     curl -X POST http://localhost:3000/dev/users -d '{"name":"John Doe", "email":"john@example.com"}'
     ```

   - Récupérer un utilisateur :
     ```bash
     curl http://localhost:3000/dev/users/<id>
     ```

#### **Partie 4 : Déploiement sur AWS**
1. **Déployer l'application sur AWS :**
   ```bash
   serverless deploy
   ```

2. **Tester en ligne** :
   Utilisez l'URL générée par AWS API Gateway pour tester les fonctions :
   ```bash
   curl -X POST https://<api-id>.execute-api.us-east-1.amazonaws.com/dev/users -d '{"name":"John Doe", "email":"john@example.com"}'
   ```

#### **Partie 5 : Tests unitaires avec Jest et `aws-sdk-client-mock`**
1. **Installer les dépendances nécessaires :**
   ```bash
   npm install jest aws-sdk-client-mock --save-dev
   ```

2. **Écrire des tests pour `addUser` :**
   Créez un fichier `handler.test.js` et écrivez vos tests avec **Jest** et le mock du SDK AWS.

   ```javascript
   const { mockClient } = require('aws-sdk-client-mock');
   const { DynamoDBClient, PutItemCommand } = require('@aws-sdk/client-dynamodb');
   const { addUser } = require('./handler');

   const dynamoMock = mockClient(DynamoDBClient);

   describe('addUser', () => {
     beforeEach(() => {
       dynamoMock.reset();
     });

     it('devrait ajouter un utilisateur avec succès', async () => {
       dynamoMock.on(PutItemCommand).resolves({});
       const event = {
         body: JSON.stringify({ name: 'John Doe', email: 'john@example.com' }),
       };
       const result = await addUser(event);
       expect(result.statusCode).toBe(200);
     });
   });
   ```

3. **Lancer les tests unitaires :**
   ```bash
   npx jest
   ```

#### **Partie 6 : Tests de performance avec Artillery**
1. **Installer Artillery :**
   ```bash
   npm install -g artillery
   ```

2. **Créer un fichier de test de charge :** `artillery-config.yml`
   ```yaml
   config:
     target: 'https://<api-id>.execute-api.us-east-1.amazonaws.com/dev'
     phases:
       - duration: 60
         arrivalRate: 10
   scenarios:
     - flow:
         - post:
             url: "/users"
             json:
               name: "John Doe"
               email: "john@example.com"
   ```

3. **Lancer le test de charge :**
   ```bash
   artillery run artillery-config.yml
   ```

Cela va générer des requêtes pour tester la capacité de votre application à gérer plusieurs utilisateurs ajoutés simultanément.

---

### **Correction attendue :**

1. **Mise en place du projet :** Vous devez avoir un projet `Serverless Framework` bien structuré avec deux fonctions Lambda (`addUser` et `getUser`), et une table DynamoDB configurée pour stocker les utilisateurs.
   
2. **Test local :** Utilisation de `serverless-offline` pour simuler un environnement AWS localement. Vous devez être capable d'ajouter un utilisateur via une requête POST et de récupérer ses informations via une requête GET.

3. **Déploiement sur AWS :** L'application doit être déployée sur AWS et accessible via API Gateway. Les fonctions Lambda doivent interagir correctement avec DynamoDB.

4. **Tests unitaires :** Les tests unitaires doivent être implémentés avec Jest et simuler DynamoDB à l’aide de `aws-sdk-client-mock`.

5. **Performance :** Le test de performance avec Artillery doit montrer la capacité de l'application à gérer plusieurs requêtes simultanées.

