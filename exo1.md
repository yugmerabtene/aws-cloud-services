### Exercice : Création d'un système de traitement de fichiers avec AWS Lambda déclenché par S3

#### Objectif :
Créer un système de traitement de fichiers serverless en utilisant AWS Lambda, déclenché par des événements d'un bucket S3.

#### Exigences :
1. **Créer un Bucket S3** :
   - Ce bucket servira de source pour déclencher la fonction Lambda chaque fois qu'un fichier y est téléchargé.
   
2. **Créer une Fonction Lambda** :
   - La fonction devra traiter les fichiers téléchargés dans le bucket S3.
   - Par exemple, si le fichier est un fichier `.txt`, la fonction doit lire son contenu et l'enregistrer dans les logs de CloudWatch.
   
3. **Configurer le Déclencheur d'Événements** :
   - Configurer S3 comme source d'événements pour la fonction Lambda, de manière à ce qu'elle soit déclenchée automatiquement lors de l'ajout d'un nouveau fichier dans le bucket.
   
4. **Gestion des Erreurs** :
   - Implémenter des stratégies de gestion des erreurs, comme l'utilisation d'une Dead Letter Queue (DLQ) ou des mécanismes de retry, pour gérer les échecs efficacement.
   
5. **Idempotence** :
   - S'assurer que la fonction Lambda est idempotente, c'est-à-dire que même si la fonction est déclenchée plusieurs fois pour le même fichier, elle le traite correctement sans effets secondaires indésirables.
   
6. **Journalisation et Surveillance** :
   - Utiliser AWS CloudWatch pour surveiller et enregistrer l'exécution de la fonction Lambda.
   - Optionnellement, utiliser AWS X-Ray pour tracer et déboguer l'exécution de la fonction.

---
### Correction :

#### 1. Créer un Bucket S3 :
   - **Étape** : Accédez à la **Console AWS**.
   - **S3** : Allez dans la section S3 et cliquez sur **Créer un Bucket**.
   - **Nom** : Donnez un nom unique à votre bucket (par exemple : `my-lambda-trigger-bucket`) et choisissez la région.
   - **Options** : Laissez les options par défaut ou personnalisez-les selon vos besoins.
   - **Création** : Cliquez sur **Créer**.

#### 2. Créer la Fonction Lambda avec les droits appropriés :

1. **Étape 1 : Accéder à AWS Lambda**  
   - Dans la console AWS, allez à la section **Lambda** et cliquez sur **Créer une fonction**.
   
2. **Étape 2 : Créer la fonction Lambda**  
   - **Nom** : `fileProcessorFunction`.
   - **Langage** : Choisissez un langage comme **Python** ou **Node.js**.
   - **Rôle IAM** : Créez un rôle avec les permissions appropriées pour S3 et CloudWatch, ou utilisez un rôle existant.

3. **Étape 3 : Permissions spécifiques à attribuer (important)**

   - **Politique IAM pour accéder à S3 (lecture uniquement)** :
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::my-lambda-trigger-bucket/*"
         }
       ]
     }
     ```
     - Cette politique permet à la fonction Lambda de **lire** les objets dans le bucket S3 spécifié.

   - **Politique IAM pour CloudWatch (permissions d'écriture)** :
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Action": [
             "logs:CreateLogGroup",
             "logs:CreateLogStream",
             "logs:PutLogEvents"
           ],
           "Resource": "*"
         }
       ]
     }
     ```
     - Cette politique donne à Lambda l'autorisation de **créer des groupes et des flux de logs** et de **publier des événements de logs** dans CloudWatch Logs.

4. **Étape 4 : Exemple de code pour la fonction Lambda (Python)** :
   ```python
   import json
   import boto3
   import logging

   s3 = boto3.client('s3')
   logger = logging.getLogger()
   logger.setLevel(logging.INFO)

   def lambda_handler(event, context):
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = event['Records'][0]['s3']['object']['key']

       try:
           # Téléchargez le fichier depuis S3
           response = s3.get_object(Bucket=bucket, Key=key)
           content = response['Body'].read().decode('utf-8')

           # Enregistrer le contenu dans CloudWatch Logs
           logger.info(f"Contenu du fichier {key} : {content}")
           return {
               'statusCode': 200,
               'body': json.dumps(f"Fichier {key} traité avec succès.")
           }
       except Exception as e:
           logger.error(f"Erreur lors du traitement du fichier {key} : {str(e)}")
           raise e
   ```

---

#### 3. Configurer le Déclencheur d'Événements S3 :
   - **Étape 1** : Dans la console Lambda, allez dans la section **Configuration** de la fonction Lambda.
   - **Étape 2** : Sous **Déclencheurs**, cliquez sur **Ajouter un déclencheur**.
   - **Étape 3** : Sélectionnez **S3** comme source de l'événement.
   - **Étape 4** : Choisissez le **bucket S3** créé (`my-lambda-trigger-bucket`).
   - **Étape 5** : Choisissez **ObjectCreated (Put)** comme type d'événement (pour déclencher la fonction lorsqu'un fichier est ajouté au bucket).
   - **Étape 6** : Cliquez sur **Ajouter** pour finaliser.

---

#### 4. Ajouter la Gestion des Erreurs :
   - **Étape 1** : Utilisez une **Dead Letter Queue (DLQ)** pour enregistrer les messages échoués dans le cas où Lambda échouerait après plusieurs tentatives.
   - **Étape 2** : Allez dans la section **Configuration > Avancé** de Lambda et configurez une file DLQ en utilisant **SQS** ou **SNS**.
   - **Étape 3** : Cela garantira que les messages qui échouent seront stockés pour un traitement ultérieur.

---

#### 5. S'assurer de l'Idempotence :
   - L’idempotence est cruciale pour éviter de traiter un fichier plusieurs fois par erreur.
   
1. **Étape 1 : Approche d'idempotence** :
   - Utilisez **DynamoDB** pour stocker les informations des fichiers déjà traités.
   - Avant de traiter un fichier, vérifiez dans DynamoDB si le fichier a déjà été traité.
   - Si le fichier a déjà été traité, ignorez l'exécution.

2. **Étape 2 : Exemple de code avec idempotence** :
   ```python
   import boto3
   dynamodb = boto3.resource('dynamodb')
   table = dynamodb.Table('ProcessedFiles')

   def lambda_handler(event, context):
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = event['Records'][0]['s3']['object']['key']

       # Vérifiez si le fichier a déjà été traité
       response = table.get_item(Key={'filename': key})
       if 'Item' in response:
           logger.info(f"Fichier {key} déjà traité.")
           return {
               'statusCode': 200,
               'body': json.dumps(f"Fichier {key} déjà traité.")
           }

       # Traitement du fichier (comme dans l'exemple précédent)

       # Enregistrer dans DynamoDB que le fichier a été traité
       table.put_item(Item={'filename': key})
   ```

---

#### 6. Journalisation et Surveillance :
   - **Logs** : Les logs de votre fonction Lambda sont envoyés automatiquement dans **CloudWatch Logs**.
   - **Surveillance** : Allez dans **CloudWatch > Logs** pour voir les logs d'exécution.
   - **Option X-Ray** : Utilisez **AWS X-Ray** pour tracer l'exécution des fonctions et diagnostiquer les performances si nécessaire.

---

#### Résultat attendu :
- À chaque fois qu’un fichier est téléchargé dans le bucket S3, la fonction Lambda se déclenche automatiquement, lit le fichier, enregistre son contenu dans **CloudWatch Logs**, et note le fichier comme "traité" dans **DynamoDB** pour assurer l'idempotence.
- En cas d'erreurs, une **DLQ** capture les messages échoués pour un traitement ultérieur.
- La fonction Lambda reste idempotente, garantissant qu’un fichier n’est pas traité plusieurs fois par erreur.
