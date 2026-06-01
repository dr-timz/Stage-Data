# Extraction et évaluation d'annotations médicamenteuses

Ce projet regroupe un pipeline simple pour comparer des annotations **manuelles** issues de Doccano avec des annotations **automatiques** produites à partir d'un fichier Medkit/Talha. Le flux de travail comprend quatre étapes : extraction du gold standard, extraction des annotations automatiques, normalisation des sorties tabulaires, puis évaluation des performances.[1][2][3][4]

## Structure du projet

| Fichier | Rôle |
|---|---|
| `ExtractTableauCsvDoccano_v2.ipynb` | Extrait les médicaments et attributs depuis `doccanno0904.jsonl`, puis génère `gold_standard.csv`.[1] |
| `ExtractTableauCsvMedkit_v2.ipynb` | Extrait les médicaments et attributs depuis `pour_talha.json`, puis génère `annot_medkit.csv`.[2] |
| `normalisation.ipynb` | Harmonise les valeurs textuelles et génère `gold_standard_norm.csv` et `annot_medkit_norm.csv`.[3] |
| `evaluation.ipynb` | Compare les fichiers normalisés et calcule les métriques globales et par attribut.[4] |

## Données d'entrée

Le notebook Doccano lit un export JSONL contenant des entités et des relations, puis reconstruit les mentions médicamenteuses à partir des offsets de texte.[1] Le notebook Medkit lit `pour_talha.json`, sépare les objets de type `Entity` et `Relation`, puis rattache les attributs aux médicaments via les identifiants source et cible.[2]

## Étape 1 — Extraction Doccano

Le script Doccano identifie les entités de type `Med` et `Classe`, puis conserve uniquement les relations d'intérêt comme `Refer_to`, `Start`, `Stop`, `Augmentation` et `Duration_presc`.[1] Pour chaque médicament, il rassemble les attributs liés (`Dosage`, `Frequence`, `Voie_administration`, `Date`, `Date_relative`, `Contexte`) et génère une ligne par combinaison d'attributs au moyen d'un produit cartésien.[1]

## Étape 2 — Extraction Medkit

Le script Medkit applique la même logique métier sur `pour_talha.json`, avec une étape préalable de séparation entre entités et relations dans `content["anns"]`.[2] Il conserve également le score de confiance lorsqu'il est présent dans les attributs de l'entité, puis exporte les résultats vers `annot_medkit.csv`.[2]

## Étape 3 — Normalisation

Le notebook de normalisation standardise les chaînes de caractères afin de réduire les variations de forme entre les deux sources.[3] Il supprime notamment les accents, homogénéise la casse, nettoie les espaces et applique un mapping sur certaines voies d'administration comme `iv`, `sc` ou `per os`.[3]

## Étape 4 — Évaluation

Le notebook d'évaluation fusionne les fichiers normalisés sur `Medicament_norm` et `Position_texte`, ce qui permet d'évaluer les correspondances entre gold standard et annotations automatiques.[4] Il calcule une métrique globale sur les lignes alignées, puis des métriques par attribut : `Dosage`, `Frequence`, `Voie_administration`, `Date`, `Date_relative` et `Contexte`.[4]

## Ordre d'exécution

1. Exécuter `ExtractTableauCsvDoccano_v2.ipynb` pour produire `gold_standard.csv`.[1]
2. Exécuter `ExtractTableauCsvMedkit_v2.ipynb` pour produire `annot_medkit.csv`.[2]
3. Exécuter `normalisation.ipynb` pour produire les deux fichiers normalisés.[3]
4. Exécuter `evaluation.ipynb` pour générer le tableau de métriques et le fichier `output/metrics_summary_corrected.csv`.[4]

## Sorties produites

| Fichier de sortie | Description |
|---|---|
| `gold_standard.csv` | Extraction structurée des annotations Doccano.[1] |
| `annot_medkit.csv` | Extraction structurée des annotations automatiques Medkit.[2] |
| `gold_standard_norm.csv` | Version normalisée du gold standard.[3] |
| `annot_medkit_norm.csv` | Version normalisée des annotations automatiques.[3] |
| `output/metrics_summary_corrected.csv` | Tableau récapitulatif des métriques globales et par attribut.[4] |

## Dépendances

Le projet utilise principalement `pandas`, `itertools`, `json`, `re`, `unicodedata` et `pathlib`, selon les notebooks concernés.[1][2][3][4] L'environnement d'exécution observé dans les notebooks est Python 3 avec noyau Jupyter `ipykernel`.[1][2][3][4]

## Objectif

L'objectif de ce dépôt est de mesurer dans quelle mesure un système automatique retrouve les médicaments et leurs attributs cliniques par rapport à une annotation de référence construite manuellement.[1][2][4]
