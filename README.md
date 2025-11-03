# Yulu Bike Sharing - Analyse Prédictive de la Demande

## TL;DR

Modèle prédictif XGBoost avec **89% de précision (R² = 0.8883)** pour anticiper la demande de vélos. Identification des facteurs clés via SHAP : l'heure de la journée (90.73), les heures de pointe (39.93) et la température (35.21) sont les variables les plus impactantes. Tests statistiques rigoureux confirmant l'effet significatif de la saison (p < 0.001, ε² = 0.0640, 6.4% de variance) et de la météo (p < 0.001, ε² = 0.0186, 1.86% de variance). 

**Recommandations clés**: Augmenter la capacité aux heures de pointe (7h-9h, 17h-19h), ajuster selon météo, optimiser par saison et implémenter une tarification dynamique basée sur les prédictions.

---

## Vue d'ensemble

**Dataset**: 10,886 observations sur les locations de vélos (2011)

**Approche méthodologique**:
- Tests d'hypothèses non-paramétriques (données non normales)
- Modélisation comparative (Linear Regression, Random Forest, XGBoost)
- Optimisation hyperparamètres (RandomizedSearchCV, 40 configurations)
- Interprétabilité SHAP

**Résultat final**: Modèle XGBoost avec Test R² = **0.8883** | MAE = **41.49 vélos** | RMSE = **61.71 vélos**

## Insights clés

### Facteurs d'influence (SHAP)

Top 5 des variables les plus impactantes:

1. **hour** (heure): 90.73 - *Facteur dominant*
2. **is_rush_hour** (heure de pointe): 39.93
3. **temp** (température): 35.21
4. **workingday** (jour ouvré): 19.82
5. **humidity** (humidité): 18.05

### Résultats statistiques

| Hypothèse | Test | p-value | Effet | Variance expliquée |
|-----------|------|---------|-------|-------------------|
| **Saison** | Kruskal-Wallis | < 0.001 | **FORT** (ε² = 0.064) | 6.4% |
| **Météo** | Kruskal-Wallis | < 0.001 | MOYEN (ε² = 0.0186) | 1.86% |
| Jour ouvré | Mann-Whitney | 0.9679 | Négligeable (r = -0.0005) | - |
| Jour férié | Mann-Whitney | 0.8646 | Négligeable (r = -0.0057) | - |

### Patterns identifiés

- **Heures de pointe**: 17:00, 18:00, 8:00 (pic à >450 vélos)
- **Meilleure saison**: Automne (234.42 vélos/h en moyenne vs 116.34 au printemps)
- **Météo optimale**: Temps clair (205.24 vélos/h vs 118.85 pluie/neige légère)
- **Corrélations Spearman**: Température (ρ = 0.400), Humidité (ρ = -0.194), Vent (ρ = -0.134)

## Recommandations

### 1. Gestion de la flotte
- Augmenter la capacité durant les heures de pointe (7h-9h, 17h-19h)
- Ajuster la disponibilité selon les conditions météorologiques (temps clair favorise +73% de demande vs pluie/neige)
- Optimiser la distribution saisonnière (+101% de demande en automne vs printemps)

### 2. Stratégie de tarification
- Tarification dynamique basée sur la demande prédite
- Tarifs promotionnels durant les périodes creuses (nuit, conditions météo défavorables)

---

## Méthodologie détaillée

## Analyses statistiques réalisées

### Tests d'hypothèses (tests non-paramétriques)

**H1 - Effet de la saison** (Test de Kruskal-Wallis)
- **Résultat**: Effet significatif (p < 0.001)
- **Taille d'effet**: ε² = 0.0640 (FORT)
- **Interprétation**: La saison explique 6.4% de la variance de la demande
- **Saison optimale**: Automne (Fall) avec une demande moyenne de 234.42 vélos
- **Comparaisons significatives (post-hoc Bonferroni, α = 0.0083)**: 
  - Printemps vs Été: p < 0.001, d = -0.6091
  - Printemps vs Automne: p < 0.001, d = -0.7136
  - Printemps vs Hiver: p < 0.001, d = -0.5369
  - Été vs Automne: p < 0.001, d = -0.0985
  - Automne vs Hiver: p < 0.001, d = 0.1888

**H2 - Effet de la météo** (Test de Kruskal-Wallis)
- **Résultat**: Effet significatif (p < 0.001)
- **Taille d'effet**: ε² = 0.0186 (MOYEN)
- **Interprétation**: La météo explique 1.86% de la variance
- **Condition optimale**: Temps clair (Clear) - demande moyenne de 205.24 vélos
- **Comparaisons significatives (post-hoc Bonferroni, α = 0.0083)**:
  - Clair vs Brume: p < 0.001, d = 0.1439
  - Clair vs Pluie/Neige légère: p < 0.001, d = 0.4712
  - Brume vs Pluie/Neige légère: p < 0.001, d = 0.3712

**H3 - Effet du jour ouvrable** (Test de Mann-Whitney U)
- **Résultat**: Effet non significatif (p = 0.9679)
- **Taille d'effet**: r = -0.0005 (NÉGLIGEABLE)
- **Différence**: +4.51 vélos (+2.4%) entre jours ouvrés et non ouvrés
- **Moyennes**: Jours non ouvrables: 188.51 | Jours ouvrables: 193.01

**H4 - Effet des jours fériés** (Test de Mann-Whitney U)
- **Résultat**: Effet non significatif (p = 0.8646)
- **Taille d'effet**: r = -0.0057 (NÉGLIGEABLE)
- **Différence**: -5.86 vélos (-3.1%) entre jours fériés et non fériés
- **Moyennes**: Jours non fériés: 191.74 | Jours fériés: 185.88

## Modélisation prédictive

### Modèles testés

| Modèle | CV R² | CV MAE | CV RMSE | Gap Train/CV |
|--------|-------|---------|---------|--------------|
| **XGBoost** | **0.8691** | **44.069** | **65.161** | 0.0792 |
| Random Forest | 0.8572 | 44.800 | 68.056 | 0.1235 |
| Linear Regression | 0.5140 | 93.007 | 125.557 | 0.0023 |

### Modèle final (après optimisation)

**XGBoost optimisé**
- **Hyperparamètres**:
  - n_estimators: 300
  - max_depth: 7
  - learning_rate: 0.05
  - subsample: 0.8
  - colsample_bytree: 1.0
  - reg_alpha: 0
  - reg_lambda: 0.1

**Performance**:
- Test R²: **0.8883**
- Test MAE: **41.49 vélos**
- Test RMSE: **61.71 vélos**
- Généralisation: **Excellente** (différence CV/Test = 0.0118)

## Technologies utilisées

- **Python 3.x**
- **Analyse**: pandas, numpy, scipy
- **Visualisation**: matplotlib, seaborn
- **Modélisation**: scikit-learn, XGBoost
- **Interprétabilité**: SHAP

---
