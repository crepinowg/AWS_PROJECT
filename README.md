# AWS_PROJECT
# Auteur: CREPIN TOVIAWOU
# Filière: Master 1 IABD
# Projet AWS - Déploiement d’une infrastructure avec CloudFormation

## Description

Ce projet a été fait pour automatiser la création d’une petite infrastructure sur AWS, en utilisant CloudFormation. L’idée, c’était de tout déployer d’un coup : une instance EC2, un bucket S3, une fonction Lambda, et une base DynamoDB.

En gros, dès qu’on ajoute un fichier dans le bucket, une Lambda se déclenche et enregistre des infos (comme le nom du fichier) dans DynamoDB.

Ce système est surtout pensé pour du test ou du développement, pas encore pour de la prod.

Ce déploiement est adapté pour des environnements de test ou de développement.

---

## Composants AWS utilisés

- **EC2** : machine virtuelle Ubuntu pour tester les opérations.
- **S3** : stockage d’objets, avec déclencheur d’événement.
- **Lambda** : fonction sans serveur pour gérer les événements S3.
- **DynamoDB** : base de données NoSQL pour stocker les métadonnées.
- **IAM** : rôle Lambda (déjà existant) permettant l’accès à DynamoDB.
- **CloudFormation** : service pour déployer l’ensemble automatiquement.

---

### Paramètres personnalisables

Quelques paramètres peuvent être modifiés au moment du déploiement :

| Nom         | À quoi ça sert                          | Valeur par défaut              |
|-------------|------------------------------------------|-------------------------------|
| `EnvName`   | Nom de l’environnement (dev, prod, etc.) | `dev`                         |
| `VPcId`     | ID de la VPC pour placer l’EC2           | `vpc-0b5f3f9b482b7b7a4`       |

---

## Ce que fait exactement le template

- **Instance EC2** :
  - Démarre avec Ubuntu 22.04.
  - Clone automatiquement un dépôt GitHub (le projet AWS).
  - Est accessible en SSH uniquement depuis une IP précise (196.169.11.120/24).
  - Utilise une clé SSH appelée `iabdkey`.

- **S3** :
  - Crée un bucket nommé `dev-file-metadata-bucket` (ou selon l’`EnvName`).
  - Déclenche une fonction Lambda dès qu’un fichier est uploadé.

- **Lambda** :
  - Écrite en Python (dans le template lui-même).
  - Récupère le nom du fichier et le bucket.
  - Insère ces infos dans DynamoDB.

- **DynamoDB** :
  - Crée une table avec une clé primaire `FileName`.
  - Enregistre les fichiers uploadés dans le S3.

---

## Commande de déploiement

```bash
aws cloudformation deploy \
  --template-file template-s3-trigger.yaml \
  --stack-name devoir-iabd-crepintoviawou-stack \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides EnvName=dev VPcId=vpc-0b5f3f9b482b7b7a4
