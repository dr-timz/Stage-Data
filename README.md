# Extraction et évaluation d'annotations médicamenteuses

Ce projet regroupe un pipeline simple pour comparer des annotations **manuelles** issues de Doccano avec des annotations **automatiques** produites à partir d'un fichier Medkit/Talha. Le flux de travail comprend quatre étapes : extraction du gold standard, extraction des annotations automatiques, normalisation des sorties tabulaires, puis évaluation des performances.

## Structure du projet

| Fichier | Rôle |
|---|---|
| `ExtractTableauCsvDoccano_v2.ipynb` | Extrait les médicaments et attributs depuis `doccanno0904.jsonl`, puis génère `gold_standard.csv`. |
| `ExtractTableauCsvMedkit_v2.ipynb` | Extrait les médicaments et attributs depuis `pour_talha.json`, puis génère `annot_medkit.csv`. |
| `normalisation.ipynb` | Harmonise les valeurs textuelles et génère `gold_standard_norm.csv` et `annot_medkit_norm.csv`. |
| `evaluation.ipynb` | Compare les fichiers normalisés et calcule les métriques globales et par attribut. |

## Données d'entrée

Le notebook Doccano lit un export JSONL contenant des entités et des relations, puis reconstruit les mentions médicamenteuses à partir des offsets de texte. Le notebook Medkit lit `pour_talha.json`, sépare les objets de type `Entity` et `Relation`, puis rattache les attributs aux médicaments via les identifiants source et cible.

## Étape 1 — Extraction Doccano

Le script Doccano identifie les entités de type `Med` et `Classe`, puis conserve uniquement les relations d'intérêt comme `Refer_to`, `Start`, `Stop`, `Augmentation` et `Duration_presc`. Pour chaque médicament, il rassemble les attributs liés (`Dosage`, `Frequence`, `Voie_administration`, `Date`, `Date_relative`, `Contexte`) et génère une ligne par combinaison d'attributs au moyen d'un produit cartésien.

## Étape 2 — Extraction Medkit

Le script Medkit applique la même logique métier sur `pour_talha.json`, avec une étape préalable de séparation entre entités et relations dans `content["anns"]`. Il conserve également le score de confiance lorsqu'il est présent dans les attributs de l'entité, puis exporte les résultats vers `annot_medkit.csv`.

## Étape 3 — Normalisation

Le notebook de normalisation standardise les chaînes de caractères afin de réduire les variations de forme entre les deux sources. Il supprime notamment les accents, homogénéise la casse, nettoie les espaces et applique un mapping sur certaines voies d'administration comme `iv`, `sc` ou `per os`.

## Étape 4 — Évaluation

Le notebook d'évaluation fusionne les fichiers normalisés sur `Medicament_norm` et `Position_texte`, ce qui permet d'évaluer les correspondances entre gold standard et annotations automatiques. Il calcule une métrique globale sur les lignes alignées, puis des métriques par attribut : `Dosage`, `Frequence`, `Voie_administration`, `Date`, `Date_relative` et `Contexte`.


## Sorties produites

| Fichier de sortie | Description |
|---|---|
| `gold_standard.csv` | Extraction structurée des annotations Doccano. |
| `annot_medkit.csv` | Extraction structurée des annotations automatiques Medkit. |
| `gold_standard_norm.csv` | Version normalisée du gold standard. |
| `annot_medkit_norm.csv` | Version normalisée des annotations automatiques. |
| `output/metrics_summary_corrected.csv` | Tableau récapitulatif des métriques globales et par attribut. |

## Tech Stack

- **Language:** Python
- **Libraries:** medkit, pandas
- **Annotation Tool:** doccano
- **Formats:** CSV, JSONL

## Objectif

L'objectif de ce dépôt est de mesurer dans quelle mesure un système automatique retrouve les médicaments et leurs attributs cliniques par rapport à une annotation de référence construite manuellement.
