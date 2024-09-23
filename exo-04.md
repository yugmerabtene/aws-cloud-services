### Mise en place de l'AWS CLI et des permissions IAM pour travailler avec AWS Lambda

### Étape 1 : Installation et Configuration de l'AWS CLI

#### 1.1 Installer l'AWS CLI

Si vous n'avez pas encore installé l'AWS CLI, voici comment procéder :

- **Linux/macOS** :
  ```bash
  curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
  sudo installer -pkg AWSCLIV2.pkg -target /
  ```

- **Windows** : Téléchargez et installez à partir de [AWS CLI Installer](https://awscli.amazonaws.com/AWSCLIV2.msi).

- **Vérification de l'installation** :
  Pour vérifier que l'installation s'est bien passée, exécutez la commande suivante :
  ```bash
  aws --version
  ```
  Vous devriez obtenir quelque chose comme `aws-cli/2.x.x`.

#### 1.2 Configurer l'AWS CLI

Vous devez configurer l'AWS CLI avec vos **credentials AWS** (clé d'accès et clé secrète) afin de pouvoir interagir avec les services AWS.

- Exécutez cette commande pour démarrer la configuration :
  ```bash
  aws configure
  ```
  Vous devrez fournir les informations suivantes :
  - **AWS Access Key ID** : Votre clé d'accès.
  - **AWS Secret Access Key** : Votre clé secrète.
  - **Default region name** : La région AWS où vous souhaitez travailler (exemple : `us-west-2`).
  - **Default output format** : Le format de sortie par défaut (exemple : `json`).

---

### Étape 2 : Création et Configuration des Rôles et Permissions IAM

Pour exécuter et déployer des fonctions Lambda, vous avez besoin d'un rôle IAM avec les permissions appropriées. Voici comment créer un rôle avec les bonnes permissions pour votre fonction Lambda.

#### 2.1 Créer un Rôle IAM pour AWS Lambda

1. **Accéder à la console IAM** :
   - Allez sur la [console IAM](https://console.aws.amazon.com/iam).

2. **Créer un Rôle IAM** :
   - Dans le menu de gauche, cliquez sur **Rôles**, puis sur **Créer un rôle**.
   - Sélectionnez **AWS Service** et choisissez **Lambda** comme type de service qui utilisera ce rôle.
   - Cliquez sur **Suivant** pour attribuer des permissions au rôle.

3. **Ajouter les Permissions Nécessaires** :
   - Sélectionnez les politiques suivantes pour accorder les permissions minimales à votre fonction Lambda :
     - **AWSLambdaBasicExecutionRole** : Pour permettre à Lambda de créer des logs dans CloudWatch.
     - **AWSLambdaS3Policy** (si vous utilisez S3 comme source d'événements) : Pour permettre à Lambda de lire les fichiers S3.
     - **AWSLambdaDynamoDBExecutionRole** (si vous interagissez avec DynamoDB).

4. **Configurer le Rôle** :
   - Donnez un nom au rôle, par exemple : `lambda-execution-role`.
   - Cliquez sur **Créer un rôle**.

#### 2.2 Attacher le Rôle IAM à la Fonction Lambda

Lorsque vous créez une fonction Lambda via l'AWS CLI ou la console, vous devez associer ce rôle IAM au moment de la création de la fonction.

---

### Étape 3 : Utilisation de l'AWS CLI pour Créer et Gérer les Fonctions Lambda

Avec l'AWS CLI configurée et le rôle IAM en place, vous pouvez commencer à gérer vos fonctions Lambda.

#### 3.1 Créer une Fonction Lambda avec AWS CLI

Voici un exemple de création d'une fonction Lambda en utilisant l'AWS CLI.

1. **Préparer le Code Lambda** :
   Créez un fichier nommé `lambda_function.py` avec le code suivant :
   ```python
   import json

   def lambda_handler(event, context):
       return {
           'statusCode': 200,
           'body': json.dumps('Hello from Lambda!')
       }
   ```

2. **Créer un Package ZIP** :
   Vous devez zipper le fichier Python avant de le déployer :
   ```bash
   zip function.zip lambda_function.py
   ```

3. **Créer la Fonction Lambda** :
   Utilisez la commande suivante pour créer la fonction Lambda en spécifiant le rôle IAM que vous avez créé précédemment :
   ```bash
   aws lambda create-function \
   --function-name MyFirstLambdaFunction \
   --runtime python3.8 \
   --role arn:aws:iam::<votre-account-id>:role/lambda-execution-role \
   --handler lambda_function.lambda_handler \
   --zip-file fileb://function.zip
   ```

4. **Tester la Fonction Lambda** :
   Vous pouvez tester la fonction en exécutant la commande suivante :
   ```bash
   aws lambda invoke --function-name MyFirstLambdaFunction output.txt
   ```

   Cette commande invoque la fonction Lambda et enregistre la sortie dans le fichier `output.txt`.

---

### Étape 4 : Utilisation des Layers dans AWS Lambda avec l'AWS CLI

#### 4.1 Créer et Déployer un Layer via l'AWS CLI

Si vous souhaitez déployer des bibliothèques partagées via un **Layer**, voici comment le faire via l'AWS CLI.

1. **Préparer les Dépendances** :
   Supposons que vous souhaitiez partager la bibliothèque `requests` entre plusieurs fonctions.

   - Créez une structure de fichiers :
     ```bash
     mkdir python
     pip install requests -t python/
     ```

2. **Empaqueter et Créer le Layer** :
   - Zippez le répertoire `python/` :
     ```bash
     zip -r requests-layer.zip python/
     ```

   - Utilisez l'AWS CLI pour créer le layer :
     ```bash
     aws lambda publish-layer-version \
     --layer-name requests-layer \
     --description "Layer contenant la bibliothèque requests" \
     --zip-file fileb://requests-layer.zip \
     --compatible-runtimes python3.8
     ```

#### 4.2 Ajouter le Layer à une Fonction Lambda

Pour utiliser le Layer que vous venez de créer dans une fonction Lambda existante, procédez comme suit :

1. **Récupérer l'ARN du Layer** :
   Après la création du layer, vous recevrez un ARN (Amazon Resource Name) pour ce layer. Il ressemble à ceci :
   ```
   arn:aws:lambda:<region>:<account-id>:layer:<layer-name>:<version>
   ```

2. **Attacher le Layer à la Fonction Lambda** :
   Utilisez la commande suivante pour mettre à jour la fonction Lambda et ajouter le layer :
   ```bash
   aws lambda update-function-configuration \
   --function-name MyFirstLambdaFunction \
   --layers arn:aws:lambda:<region>:<account-id>:layer:requests-layer:<version>
   ```

3. **Tester la Fonction avec le Layer** :
   Mettez à jour votre code Lambda pour utiliser la bibliothèque `requests` et testez-le comme précédemment.

---

### Conclusion :
Vous êtes maintenant prêt à utiliser **l'AWS CLI** pour déployer et gérer des fonctions Lambda, ainsi que pour utiliser des **AWS Lambda Layers** pour partager des dépendances. Vous avez également appris à configurer les rôles **IAM** nécessaires pour exécuter les fonctions Lambda en toute sécurité.
