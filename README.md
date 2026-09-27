# Prévision de la consommation électrique française

Projet de prévision de séries temporelles comparant une approche statistique classique et une approche de machine learning, réalisé dans le cadre de mon Master 2 (Ingénieur Mathématique pour la Science de Données).

## Objectif

Prédire la consommation électrique nationale (données éCO2mix, RTE) à partir de son historique, en comparant deux approches :
- Un modèle statistique classique (lissage exponentiel Holt-Winters)
- Un modèle de machine learning (Random Forest) avec des features temporelles (heure, jour de la semaine, valeurs passées)

## Données

Données ouvertes RTE éCO2mix (consommation nationale au pas de 15 minutes) : https://odre.opendatasoft.com/explore/dataset/eco2mix-national-tr/

## Méthodologie

1. Chargement et nettoyage des données
2. Séparation train/test (3 derniers jours en test)
3. Modèle Holt-Winters (statsmodels) avec saisonnalité journalière
4. Modèle Random Forest avec features : heure, jour de la semaine, valeur à J-1, J-1 (même heure), semaine précédente (même heure)
5. Comparaison des performances (MAE, RMSE)

## Résultats

| Modèle | MAE (MW) | RMSE (MW) |
|---|---|---|
| Holt-Winters | 17 230 | 20 491 |
| Random Forest | 341 | 458 |

Le Random Forest surpasse largement le modèle statistique classique : l'ajout de features temporelles simples (lags, heure, jour de la semaine) permet de capturer bien plus finement la saisonnalité journalière de la consommation électrique.

## Outils

Python, pandas, statsmodels, scikit-learn, matplotlib

## Reproduire

Ouvrir `analyse_conso_electrique.ipynb` dans Google Colab (bouton "Open in Colab" en haut du notebook) et exécuter les cellules dans l'ordre.
