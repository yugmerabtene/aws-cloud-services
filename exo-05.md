### Thème 1 : **Stockage des données dans des applications serverless**

#### Exercice 1 : Comprendre le concept de l'architecture stateless

**Énoncé :**  
Tu as une application web qui doit gérer des sessions utilisateurs et stocker des données pour plusieurs requêtes. Ton application fonctionne avec AWS Lambda, qui est par nature stateless.  

1. Explique pourquoi AWS Lambda est considéré comme stateless.  
2. Propose une solution pour gérer les sessions utilisateurs dans une architecture serverless, en utilisant des services AWS.
3. Si l’application a besoin de stocker des données persistantes, quels services AWS recommandes-tu ? Explique les avantages de chacun.

---

**Correction :**

1. **Pourquoi AWS Lambda est stateless :**  
   AWS Lambda est un service *stateless* car il ne conserve aucune information de session ou d'état entre les invocations. Chaque appel est exécuté indépendamment des précédents, ce qui permet une scalabilité simple et flexible, car Lambda n’a pas besoin de suivre les connexions utilisateurs ou d'autres données entre les requêtes【7†source】.

2. **Gestion des sessions utilisateurs dans une architecture serverless :**  
   Pour gérer les sessions utilisateurs dans un contexte *stateless*, une solution est d’utiliser **Amazon DynamoDB** pour stocker les informations de session avec une clé unique (comme un jeton ou un ID de session). Alternativement, les informations de session peuvent être stockées dans un token JWT (JSON Web Token), transmis avec chaque requête côté client【7†source】.

3. **Services AWS pour le stockage de données persistantes :**  
   - **Amazon S3** : Idéal pour le stockage de fichiers et objets. Il offre une forte disponibilité et scalabilité, avec une cohérence immédiate pour les opérations PUT de nouveaux objets【7†source】.
   - **Amazon DynamoDB** : Base de données NoSQL avec des options de lecture à forte ou éventuelle cohérence, conçue pour des requêtes à faible latence【7†source】.
   - **Amazon RDS** (ou Aurora Serverless) : Fournit des bases de données relationnelles conformes ACID pour des transactions nécessitant une forte cohérence【7†source】.

---

### Thème 2 : **Sécurité et gestion des identités (IAM)**

#### Exercice 2 : IAM et sécurité des applications serverless

**Énoncé :**  
Dans une application serverless, une fonction Lambda doit accéder à des objets dans un bucket S3 pour lire des fichiers. Pour cela, tu dois utiliser AWS Identity and Access Management (IAM) pour configurer les rôles et permissions.

1. Crée une politique IAM qui autorise une fonction Lambda à lire uniquement des objets dans un bucket S3 spécifique.
2. Qu'est-ce que le **principe du moindre privilège**, et pourquoi est-il important dans cette situation ?
3. Explique comment gérer les secrets (comme les clés API ou les informations d'identification) dans une application Lambda en évitant de les hardcoder dans le code source.

---

**Correction :**

1. **Politique IAM pour accès S3 :**  
   Voici un exemple de politique JSON IAM qui autorise la fonction Lambda à lire les objets dans un bucket S3 spécifique :

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::mon_bucket/*"
       }
     ]
   }
   ```
   Cette politique accorde l’autorisation à la fonction Lambda d’effectuer l’action `s3:GetObject` sur tous les objets du bucket spécifié【6†source】.

2. **Principe du moindre privilège :**  
   Le principe du moindre privilège consiste à n'accorder à un utilisateur ou un service que les permissions strictement nécessaires pour accomplir ses tâches. Cela réduit les risques de sécurité en limitant les actions possibles en cas de compromission du service ou de l’utilisateur. Dans ce cas, il est essentiel que la fonction Lambda n’ait accès qu’aux objets S3 requis et pas à d'autres ressources non nécessaires【6†source】.

3. **Gestion des secrets dans Lambda :**  
   Pour gérer les secrets en toute sécurité dans AWS Lambda, il est recommandé d’utiliser **AWS Secrets Manager** ou **AWS Systems Manager Parameter Store**. Ces services permettent de stocker les secrets de manière sécurisée et de les récupérer à l'exécution via des appels API, au lieu de les coder en dur dans le code. En outre, ils permettent la rotation automatique des secrets et l’application de politiques IAM pour restreindre l'accès【6†source】.

   Voici un exemple de code en Node.js pour récupérer un secret depuis Secrets Manager dans une fonction Lambda :

   ```javascript
   const AWS = require('aws-sdk');
   const secretsManager = new AWS.SecretsManager();

   exports.handler = async (event) => {
     const secretId = 'mon-secret-id';
     const secret = await secretsManager.getSecretValue({ SecretId: secretId }).promise();
     // Utilisation du secret récupéré
   };
   ```
