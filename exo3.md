Traitement des événements S3 avec AWS Lambda et gestion des erreurs

#### Objectif :
Mettre en place un traitement d'événements basé sur S3 avec AWS Lambda, et implémenter des mécanismes de gestion des erreurs, y compris l'utilisation d'une **Dead Letter Queue (DLQ)** pour gérer les échecs d'exécution.

Cet exercice couvre les concepts suivants :
- Déclenchement de Lambda par un événement S3.
- Gestion des erreurs et des échecs avec DLQ.
- Utilisation de CloudWatch pour surveiller les erreurs.

---

### Correction détaillée, étape par étape :

#### 1. Créer un Bucket S3 pour Déclencher des Événements

##### Étape 1.1 : Créer un Bucket S3
- Accédez à la **console AWS**.
- Allez dans **S3** et cliquez sur **Créer un bucket**.
- Donnez un nom unique à votre bucket, par exemple `bucket-evenement-lambda`.
- Configurez les options par défaut et cliquez sur **Créer**.

#### 2. Créer une Fonction Lambda qui Traite les Fichiers S3

##### Étape 2.1 : Créer la Fonction Lambda
- Allez dans **AWS Lambda**.
- Créez une nouvelle fonction :
  - Nom : `S3EventProcessor`
  - Runtime : **Python 3.x** ou un autre runtime que vous préférez.
  - Rôle : Utilisez un rôle existant ou créez-en un nouveau avec des permissions pour accéder à S3 et CloudWatch.

##### Étape 2.2 : Code de la Fonction Lambda
Voici un exemple de fonction qui est déclenchée par l'ajout d'un fichier dans le bucket S3. La fonction tente de lire le fichier et d'enregistrer son contenu dans CloudWatch Logs.

```python
import json
import boto3
import logging

# Configurer les logs
logger = logging.getLogger()
logger.setLevel(logging.INFO)

s3 = boto3.client('s3')

def lambda_handler(event, context):
    try:
        # Récupérer les informations de l'événement S3
        bucket = event['Records'][0]['s3']['bucket']['name']
        key = event['Records'][0]['s3']['object']['key']
        
        logger.info(f"Traitement du fichier : {key} depuis le bucket : {bucket}")

        # Lire le contenu du fichier
        response = s3.get_object(Bucket=bucket, Key=key)
        file_content = response['Body'].read().decode('utf-8')
        
        # Logguer le contenu du fichier
        logger.info(f"Contenu du fichier {key} : {file_content}")
        
        return {
            'statusCode': 200,
            'body': json.dumps('Fichier traité avec succès')
        }

    except Exception as e:
        logger.error(f"Erreur lors du traitement du fichier {key} : {str(e)}")
        raise e
```

##### Explication :
- **event['Records']** : C'est l'objet transmis à la fonction Lambda par l'événement S3. Il contient des informations sur le fichier ajouté.
- **s3.get_object** : Cette commande lit le fichier depuis le bucket S3.
- **Gestion des erreurs** : Si une erreur se produit, elle est loggée, et l'exception est levée pour permettre à Lambda de réessayer ou d'envoyer l'échec à la Dead Letter Queue (DLQ).

#### 3. Configurer le Déclencheur d'Événements S3 pour la Fonction Lambda

##### Étape 3.1 : Ajouter un Déclencheur S3
- Allez dans la section **Déclencheurs** de la fonction Lambda.
- Cliquez sur **Ajouter un déclencheur**.
- Choisissez **S3** comme source de l'événement.
- Sélectionnez le bucket que vous avez créé précédemment (`bucket-evenement-lambda`).
- Choisissez l'événement **PutObject** pour déclencher la fonction lorsqu'un nouveau fichier est ajouté au bucket.
- Cliquez sur **Ajouter**.

#### 4. Configurer une Dead Letter Queue (DLQ)

##### Étape 4.1 : Créer une File SQS pour DLQ
- Allez dans la **console SQS** (Simple Queue Service).
- Créez une nouvelle file d'attente nommée `LambdaDLQ`.
- Acceptez les paramètres par défaut et enregistrez la file.

##### Étape 4.2 : Associer la DLQ à la Fonction Lambda
- Dans la configuration de votre fonction Lambda, allez dans l'onglet **Configuration**.
- Sélectionnez **Destinations** et configurez une **Dead Letter Queue** pour que les erreurs de traitement soient envoyées à la file SQS que vous avez créée.
- Cela permettra de capturer tous les échecs de traitement que Lambda ne parvient pas à résoudre après plusieurs tentatives.

#### 5. Ajouter la Gestion des Erreurs et Réessayer en Cas d'Échec

##### Étape 5.1 : Configurer des Tentatives Automatiques
- Allez dans la configuration de la fonction Lambda et sous l'onglet **Asynchronous invocation**, configurez les tentatives automatiques (par défaut, Lambda réessaiera deux fois).
- Cela garantit que Lambda réessayera de traiter le fichier en cas d'échec.

#### 6. Tester la Solution

##### Étape 6.1 : Télécharger un Fichier dans le Bucket S3
- Allez dans le bucket S3 (`bucket-evenement-lambda`) et téléchargez un fichier texte.
- Cela déclenchera la fonction Lambda. Vous pouvez consulter les logs dans **CloudWatch Logs** pour voir si le fichier a été correctement traité.

##### Étape 6.2 : Provoquer une Erreur pour Tester la DLQ
- Modifiez la fonction Lambda pour provoquer une erreur intentionnelle (par exemple, en supprimant la permission d'accès à S3).
- Téléchargez un fichier dans le bucket S3 pour déclencher l'erreur.
- Consultez les logs dans **CloudWatch** et la file d'attente SQS **LambdaDLQ** pour voir l'enregistrement des erreurs.

---

### Résultat attendu :
- Vous verrez dans **CloudWatch Logs** les journaux de traitement des fichiers avec succès.
- Si une erreur se produit, elle sera loggée dans **CloudWatch**, et après plusieurs tentatives échouées, l'échec sera capturé dans la **Dead Letter Queue (DLQ)**.
- Ce mécanisme vous aide à gérer les erreurs de manière plus robuste en évitant de perdre des événements importants.
