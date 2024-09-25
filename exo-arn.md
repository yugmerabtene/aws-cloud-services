### Exercice : Création d'une Table DynamoDB avec AWS SAM et Récupération de son ARN

#### Énoncé :

Tu es chargé de déployer une application serverless qui utilise DynamoDB comme base de données. Ton objectif est de créer une table DynamoDB en utilisant un fichier de configuration AWS SAM au format YAML et d'afficher son ARN après la création.

**Étapes :**
1. Crée une table DynamoDB avec un attribut de partition nommé `UserId` (type `String`).
2. Configure la table pour utiliser le mode de facturation "Pay-per-request".
3. Dans la section `Outputs` du fichier YAML, exporte l'ARN de la table afin qu'il puisse être utilisé dans d'autres stacks ou services.
4. Une fois la stack déployée, récupère l'ARN de la table via AWS CLI.

**Contrainte :**
- La table doit avoir le nom `UsersTable`.
- Utilise AWS SAM pour déployer la table.
- Utilise la commande AWS CLI pour obtenir l'ARN après le déploiement.

#### Fichier YAML attendu (AWS SAM) :

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Resources:
  UsersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: UsersTable
      AttributeDefinitions:
        - AttributeName: UserId
          AttributeType: S
      KeySchema:
        - AttributeName: UserId
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

Outputs:
  UsersTableARN:
    Value: !GetAtt UsersTable.Arn
    Export:
      Name: UsersTableARN
```

#### Étapes de correction :

1. **Création de la table** :
   Dans le fichier YAML, on utilise la ressource `AWS::DynamoDB::Table` pour créer la table DynamoDB. On définit un attribut de partition (`UserId`) et le mode de facturation `PAY_PER_REQUEST` pour éviter de spécifier manuellement la capacité.

2. **Exportation de l'ARN** :
   L'ARN de la table est obtenu avec la fonction **`!GetAtt UsersTable.Arn`** et exporté dans la section **Outputs**. Cela permet d'utiliser cet ARN ailleurs dans d'autres stacks ou applications.

3. **Déploiement** :
   Après avoir défini le fichier YAML, tu peux déployer la stack avec les commandes suivantes :

   ```bash
   sam build
   sam deploy --guided
   ```

   Pendant la phase de déploiement, AWS SAM créera la table DynamoDB et affichera le nom et l'ARN dans les sorties.

4. **Récupération de l'ARN via AWS CLI** :
   Une fois la stack déployée, tu peux récupérer l'ARN directement en utilisant la commande suivante :

   ```bash
   aws dynamodb describe-table --table-name UsersTable --query "Table.TableArn"
   ```

   Cette commande affichera l'ARN de la table, que tu peux utiliser dans d'autres services ou pour des besoins de configuration.

#### Résultat attendu :
La table `UsersTable` sera créée dans DynamoDB, et son ARN sera disponible à la fois dans la section **Outputs** de SAM et via la commande AWS CLI.
