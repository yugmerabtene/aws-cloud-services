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
   - Accédez à la **Console AWS**.
   - Allez dans **S3** et cliquez sur "Créer un Bucket".
   - Donnez un nom unique à votre bucket (exemple : `my-lambda-trigger-bucket`) et choisissez la région.
   - Configurez les options par défaut ou personnalisez selon vos besoins, puis cliquez sur **Créer**.

#### 2. Créer la Fonction Lambda :
   - Allez dans **AWS Lambda** via la console.
   - Cliquez sur **Créer une fonction**.
   - Sélectionnez "Créer une fonction à partir de zéro" :
     - Nom de la fonction : `fileProcessorFunction`.
     - Langage : Choisissez un langage supporté comme **Node.js** ou **Python**.
     - Rôle : Utilisez un rôle existant avec les permissions S3 et CloudWatch, ou créez un nouveau rôle avec ces permissions.
   - Cliquez sur **Créer une fonction**.

   Exemple de code pour une fonction Lambda en **Python** :

   ```python
   import json
   import boto3
   import logging

   s3 = boto3.client('s3')
   logger = logging.getLogger()
   logger.setLevel(logging.INFO)

   def lambda_handler(event, context):
       # Obtenez les informations sur l'objet S3
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = event['Records'][0]['s3']['object']['key']

       try:
           # Téléchargez le fichier depuis S3
           response = s3.get_object(Bucket=bucket, Key=key)
           content = response['Body'].read().decode('utf-8')

           # Loggez le contenu du fichier
           logger.info(f"Contenu du fichier {key} : {content}")
           return {
               'statusCode': 200,
               'body': json.dumps(f"Fichier {key} traité avec succès.")
           }
       except Exception as e:
           logger.error(f"Erreur lors du traitement du fichier {key} : {str(e)}")
           raise e
   ```

   Ce code lit le fichier depuis S3 et enregistre son contenu dans les logs CloudWatch.

#### 3. Configurer le Déclencheur d'Événements S3 :
   - Une fois la fonction Lambda créée, allez dans la section **Configuration** de la fonction.
   - Sous **Déclencheurs**, cliquez sur **Ajouter un déclencheur**.
   - Sélectionnez **S3** comme source de l'événement.
   - Sélectionnez le **bucket S3** que vous avez créé.
   - Choisissez **ObjectCreated (Put)** comme type d'événement, afin que la fonction soit déclenchée lorsqu'un fichier est ajouté au bucket.
   - Cliquez sur **Ajouter** pour sauvegarder.

#### 4. Ajouter la Gestion des Erreurs :
   - Dans la configuration de votre fonction Lambda, vous pouvez définir une **Dead Letter Queue (DLQ)** pour enregistrer les messages échoués. Cela permet de traiter les erreurs qui persistent après les tentatives de retry.
   - Allez dans la section **Configuration** > **Avancé** et configurez une file d'attente DLQ à l’aide de **SQS** ou **SNS**.
   - Cela garantira que les messages qui échouent après plusieurs tentatives seront stockés pour un traitement ultérieur.

#### 5. S'assurer de l'Idempotence :
   - L’idempotence permet d’éviter le traitement multiple d’un même fichier en cas de répétition des événements.
   - Exemple d'approche :
     - Utiliser **DynamoDB** pour garder une trace des fichiers déjà traités.
     - Avant de traiter un fichier, vérifier dans DynamoDB si le fichier a déjà été traité.
     - Si le fichier a été traité, ignorer l'exécution.

   Exemple d’ajout de l’idempotence au code :

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

#### 6. Journalisation et Surveillance :
   - Les logs de votre fonction Lambda sont automatiquement envoyés dans **CloudWatch Logs**.
   - Allez dans **CloudWatch** > **Logs** pour voir les logs d'exécution.
   - Utilisez également **AWS X-Ray** pour tracer les appels et diagnostiquer les performances.

#### Résultat attendu :
- Chaque fois qu’un fichier est téléchargé dans le bucket S3, la fonction Lambda est déclenchée, lit le fichier, enregistre son contenu dans CloudWatch, et marque le fichier comme traité.
- En cas d’erreurs, les messages sont envoyés dans une DLQ pour analyse future.
- La fonction traite chaque fichier de manière idempotente, garantissant qu’un fichier n’est pas traité plusieurs fois par accident.
