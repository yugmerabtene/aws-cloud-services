### **Exercice 1 : Fonction Lambda de base avec AWS Lambda et Framework Serverless**

#### **Énoncé :**
Vous devez créer une fonction simple "Hello World" en utilisant AWS Lambda via le Framework Serverless. La fonction doit être déclenchée lorsqu'une requête HTTP GET est effectuée sur l'endpoint `/hello`.

---

### **Correction :**

1. **Installation du Framework Serverless :**
   - Installer le Framework Serverless globalement sur votre machine :
     ```bash
     npm install -g serverless
     ```
   - Créer un nouveau projet Serverless :
     ```bash
     serverless create --template aws-python --path hello-world-service
     cd hello-world-service
     ```

2. **Configuration du fichier `serverless.yml` :**
   Modifiez le fichier `serverless.yml` pour définir une fonction Lambda qui répond aux requêtes HTTP GET :
   ```yaml
   service: hello-world-service
   provider:
     name: aws
     runtime: python3.8
     region: us-east-1
   functions:
     hello:
       handler: handler.hello
       events:
         - http:
             path: hello
             method: get
   ```

3. **Écriture du gestionnaire Lambda :**
   Dans le fichier `handler.py`, ajoutez le code suivant pour gérer l'événement :
   ```python
   def hello(event, context):
       response = {
           "statusCode": 200,
           "body": "Bonjour, le monde !"
       }
       return response
   ```

4. **Déploiement de la fonction :**
   Déployez la fonction sur AWS :
   ```bash
   serverless deploy
   ```

5. **Invocation de la fonction localement :**
   Testez la fonction localement :
   ```bash
   serverless invoke local --function hello
   ```

6. **Résultat attendu :**
   La fonction Lambda doit renvoyer :
   ```json
   {
     "statusCode": 200,
     "body": "Bonjour, le monde !"
   }
   ```

---

### **Exercice 2 : Ajouter des ressources AWS (S3, DynamoDB) à une application Serverless**

#### **Énoncé :**
Améliorez l’exercice précédent en ajoutant un bucket S3 et une table DynamoDB. La fonction doit permettre de télécharger un objet dans S3 et de stocker des métadonnées dans DynamoDB lorsqu’une requête HTTP POST est effectuée.

---

### **Correction :**

1. **Mise à jour du fichier `serverless.yml` :**
   Ajoutez des ressources S3 et DynamoDB dans le fichier `serverless.yml` :
   ```yaml
   service: resource-handler-service
   provider:
     name: aws
     runtime: python3.8
     region: us-east-1

   functions:
     saveData:
       handler: handler.save_data
       events:
         - http:
             path: upload
             method: post

   resources:
     Resources:
       S3Bucket:
         Type: AWS::S3::Bucket
         Properties:
           BucketName: resource-bucket
       DynamoDBTable:
         Type: AWS::DynamoDB::Table
         Properties:
           TableName: ResourceMetadata
           AttributeDefinitions:
             - AttributeName: id
               AttributeType: S
           KeySchema:
             - AttributeName: id
               KeyType: HASH
           ProvisionedThroughput:
             ReadCapacityUnits: 1
             WriteCapacityUnits: 1
   ```

2. **Écriture de la fonction Lambda :**
   Modifiez le fichier `handler.py` pour ajouter une fonction qui permet d’envoyer des fichiers à S3 et de sauvegarder des métadonnées dans DynamoDB :
   ```python
   import boto3
   import json
   import uuid

   s3 = boto3.client('s3')
   dynamodb = boto3.resource('dynamodb')

   def save_data(event, context):
       data = json.loads(event['body'])
       file_content = data['fileContent']
       file_name = str(uuid.uuid4())

       # Téléchargement vers S3
       s3.put_object(Bucket='resource-bucket', Key=file_name, Body=file_content)

       # Sauvegarde des métadonnées dans DynamoDB
       table = dynamodb.Table('ResourceMetadata')
       table.put_item(Item={'id': file_name, 'content': file_content})

       return {
           "statusCode": 200,
           "body": json.dumps({"message": "Données enregistrées avec succès !", "fileId": file_name})
       }
   ```

3. **Déploiement et Test :**
   - Déployez la fonction avec la commande suivante :
     ```bash
     serverless deploy
     ```
   - Testez la fonction en envoyant une requête POST :
     ```bash
     curl -X POST https://<API-ENDPOINT>/upload -d '{"fileContent": "Exemple de contenu"}'
     ```

4. **Vérification des données :**
   Après avoir invoqué la fonction, vérifiez que le fichier est bien présent dans le bucket S3 et que les métadonnées sont enregistrées dans la table DynamoDB.

---

### **Exercice 3 : Développement et Test local avec les plugins du Framework Serverless**

#### **Énoncé :**
Vous devez simuler un environnement AWS local pour développer et tester des fonctions serverless sans déployer sur AWS. Utilisez le plugin `serverless-offline` et simulez un événement S3 localement.

---

### **Correction :**

1. **Installation du plugin `serverless-offline` :**
   Installez le plugin `serverless-offline` pour exécuter API Gateway localement :
   ```bash
   serverless plugin install -n serverless-offline
   ```

2. **Configuration du fichier `serverless.yml` pour le développement local :**
   Modifiez le fichier `serverless.yml` pour inclure le plugin `serverless-offline` et simuler S3 :
   ```yaml
   plugins:
     - serverless-offline

   custom:
     serverless-offline:
       httpPort: 4000
     s3:
       host: localhost
       directory: /tmp
   ```

3. **Lancement du serveur local :**
   Démarrez le serveur local pour simuler l’API Gateway AWS :
   ```bash
   serverless offline start
   ```

4. **Simulation d’un événement S3 localement :**
   Utilisez AWS CLI pour simuler un événement S3 en téléchargeant un fichier dans un bucket S3 local :
   ```bash
   aws --endpoint http://localhost:4569 s3 cp data.json s3://local-bucket/data.json
   ```

5. **Invocation de la fonction localement :**
   Utilisez la commande `serverless invoke local` pour tester la fonction localement avec un événement simulé :
   ```bash
   serverless invoke local --function save_data --path event.json
   ```

6. **Vérification des logs et des sorties :**
   Vérifiez que la fonction fonctionne comme prévu et que les logs sont générés correctement.

