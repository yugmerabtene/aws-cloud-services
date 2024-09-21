### **IaaS (Infrastructure as a Service)**

1. **EC2 (Elastic Compute Cloud)**  
   - **Signification** : Service de calcul à la demande.  
   - **Explication** : Fournit des machines virtuelles (instances) pour exécuter des applications.

2. **S3 (Simple Storage Service)**  
   - **Signification** : Stockage d'objets.  
   - **Explication** : Stockage sécurisé pour des données volumineuses.

3. **EBS (Elastic Block Store)**  
   - **Signification** : Stockage en bloc élastique.  
   - **Explication** : Disques durs virtuels pour les instances EC2.

4. **VPC (Virtual Private Cloud)**  
   - **Signification** : Cloud privé virtuel.  
   - **Explication** : Réseau isolé pour héberger des ressources cloud.

5. **RDS (Relational Database Service)**  
   - **Signification** : Base de données relationnelle.  
   - **Explication** : Gère des bases de données comme MySQL, PostgreSQL sans gestion de serveur.

### **PaaS (Platform as a Service)**

1. **Elastic Beanstalk**  
   - **Signification** : Plateforme de déploiement automatique.  
   - **Explication** : Déploiement d'applications sans gestion de l'infrastructure sous-jacente.

2. **Lambda**  
   - **Signification** : Calcul sans serveur (Serverless).  
   - **Explication** : Exécution de code sans provisionnement de serveurs.

3. **Fargate**  
   - **Signification** : Calcul sans serveur pour containers.  
   - **Explication** : Exécute des conteneurs sans gérer les serveurs ou clusters.

### **SaaS (Software as a Service)**

1. **WorkMail**  
   - **Signification** : Service de messagerie professionnelle.  
   - **Explication** : Hébergement d'emails et calendriers pour entreprises.

2. **Chime**  
   - **Signification** : Service de communication unifiée.  
   - **Explication** : Messagerie, appels vidéo et conférences pour entreprises.

3. **WorkDocs**  
   - **Signification** : Stockage et collaboration de documents.  
   - **Explication** : Plateforme pour partager et collaborer sur des documents en ligne.

### **IaC (Infrastructure as Code)**

1. **CloudFormation**  
   - **Signification** : Modèles d'infrastructure.  
   - **Explication** : Provisionnement d'infrastructure AWS via des fichiers de configuration.

2. **CDK (Cloud Development Kit)**  
   - **Signification** : Kit de développement d'infrastructure.  
   - **Explication** : Utilisation de langages de programmation pour définir des ressources AWS.

3. **OpsWorks**  
   - **Signification** : Gestion de configuration.  
   - **Explication** : Automatisation de la configuration des serveurs et applications avec Chef/Puppet.

### **LaaS (Logging as a Service)**

1. **CloudWatch**  
   - **Signification** : Service de surveillance et de logs.  
   - **Explication** : Surveillance des métriques, journaux et événements AWS.

2. **AWS X-Ray**  
   - **Signification** : Analyse de performance d’application.  
   - **Explication** : Suivi et analyse des requêtes à travers une application distribuée.

### **BaaS (Backend as a Service)**

1. **AWS Amplify**  
   - **Signification** : Service de développement d’applications.  
   - **Explication** : Fournit un backend complet pour des applications web et mobiles (API, authentification, stockage).

2. **AWS Cognito**  
   - **Signification** : Service d'authentification et d'autorisation.  
   - **Explication** : Gestion des utilisateurs et de l'authentification pour les applications.

3. **AWS AppSync**  
   - **Signification** : GraphQL en tant que service.  
   - **Explication** : Facilite la création d’API GraphQL pour synchroniser les données entre le backend et les applications.

### **CaaS (Container as a Service)**

1. **ECS (Elastic Container Service)**  
   - **Signification** : Service de gestion de conteneurs.  
   - **Explication** : Gère et exécute des conteneurs Docker sur AWS, avec intégration avec EC2 et Fargate.

2. **EKS (Elastic Kubernetes Service)**  
   - **Signification** : Service de Kubernetes managé.  
   - **Explication** : Exécute et gère des clusters Kubernetes pour orchestrer les conteneurs.

3. **Fargate**  
   - **Signification** : Exécution de conteneurs sans gestion de serveurs.  
   - **Explication** : Permet d'exécuter des conteneurs sans avoir à gérer l'infrastructure sous-jacente.

### **FaaS (Function as a Service)**

1. **Lambda**  
   - **Signification** : Calcul sans serveur.  
   - **Explication** : Permet d’exécuter des fonctions sans gestion de serveurs, facturation basée sur l'exécution du code uniquement.

2. **Step Functions**  
   - **Signification** : Orchestration de services.  
   - **Explication** : Permet de coordonner des fonctions Lambda et d'autres services AWS en créant des workflows.

3. **API Gateway**  
   - **Signification** : Passerelle d'API.  
   - **Explication** : Permet de créer et gérer des API RESTful ou WebSocket, souvent utilisé avec Lambda pour des architectures FaaS.

### **Résumé par catégories** :

- **IaaS** : EC2, S3, EBS, VPC, RDS
- **PaaS** : Elastic Beanstalk, Lambda, Fargate
- **SaaS** : WorkMail, Chime, WorkDocs
- **IaC** : CloudFormation, CDK, OpsWorks
- **LaaS** : CloudWatch, AWS X-Ray
- **BaaS** : AWS Amplify, AWS Cognito, AWS AppSync
- **CaaS** : ECS, EKS, Fargate
- **FaaS** : Lambda, Step Functions, API Gateway
