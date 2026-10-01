# Préparation de données cliniques multi-sources (hépatologie)

Fusion et harmonisation de deux fichiers de données cliniques (CSV et JSON) de patients hépatiques, puis classification binaire (maladie hépatique / pas de maladie). Projet académique réalisé en binôme à l'ISIS Castres (INSA).

## Ce que montre ce projet

Un pipeline de préparation **sans fuite de données**, et une évaluation qui ne survend pas ses résultats.

## Démarche

1. **Harmonisation des deux sources**
   - Contrôle des échelles par comparaison des médianes : la bilirubine du CSV est ≈ 10 fois celle du JSON, elle est ramenée à la même échelle.
   - Protéines totales converties de mg/dL en g/dL ; tranches d'âge (« 41 to 60 yo ») converties en valeur numérique (milieu de tranche).
   - Renommage des colonnes, concaténation, suppression des doublons, y compris 18 doublons détectables seulement après harmonisation des formats (589 patients au final).
2. **Nettoyage** : les 0 biologiquement impossibles sont remplacés par des valeurs manquantes.
3. **Simplification** : suppression de `Direct_Bilirubin` (corrélation de 0,88 avec la bilirubine totale) et de `Albumin_and_Globulin_Ratio` (dérivé d'autres colonnes, ≈ 20 % de valeurs manquantes).
4. **Pipeline strict** : split stratifié 80/20 → imputation KNN → suppression des outliers (train) → StandardScaler → SMOTE (train). Tout paramètre appris l'est sur le train uniquement.
5. **Classification** : Random Forest.
6. **Validation croisée 5 plis** avec un pipeline complet (imputation, normalisation, SMOTE) réappris dans chaque pli.

## Résultats

| Évaluation | Résultat |
|---|---|
| Test (118 patients) : F1 macro | 0,655 |
| Test : rappel classe « maladie » | 0,718 (61 / 85) |
| Test : rappel classe « pas de maladie » | 0,636 (21 / 33) |
| Validation croisée 5 plis : F1 macro | 0,621 ± 0,068 |

Ces performances sont modestes et l'estimation est instable (écart-type élevé, test de 118 patients). Environ 3 patients malades sur 10 ne sont pas détectés sur le test : le modèle n'est pas utilisable en l'état pour un dépistage.

## Limites

- 589 patients seulement, dont 33 « sans maladie » dans le test.
- Âge connu par tranches uniquement (perte de précision).
- Le facteur 10 sur la bilirubine est déduit des données, pas d'une documentation.
- Le projet valide la méthode de préparation, pas un modèle clinique.

## Stack

Python · Pandas · scikit-learn · imbalanced-learn · SciPy · Matplotlib · Seaborn

## Structure du projet

```
├── clinical_data_preparation_liver.ipynb
├── README.md
├── requirements.txt
├── .gitignore
└── LICENSE
```

## Reproduire

Les fichiers `liver.csv` et `liver.json` fournis par le cours ne sont pas versionnés. Pour exécuter le notebook, placer ces deux fichiers dans le dossier indiqué par `BASE_DIR` (première cellule), puis :

```bash
pip install -r requirements.txt
```

Le notebook détecte automatiquement s'il est exécuté sous Google Colab (montage de Drive) ou en local.

## Auteur

**Mathieu Jonniaux** — ISIS Castres (partenaire INSA), FIE4 DSIA
[github.com/zeyglitch](https://github.com/zeyglitch) · [LinkedIn](https://www.linkedin.com/in/mathieu-jonniaux)
