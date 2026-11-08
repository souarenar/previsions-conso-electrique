# Prévision de la consommation électrique française

Projet de prévision de séries temporelles comparant des approches statistiques classiques et une approche de machine learning, réalisé dans le cadre de mon Master 2 Ingénierie Mathématique pour la Science des Données (Université de Lorraine).

Le projet a été **revu en novembre 2026** pour corriger deux biais d'évaluation identifiés dans la première version (voir la section « Ce que j'ai corrigé »).

## Objectif

Prévoir la consommation électrique nationale française (données éCO2mix, RTE) à partir de son historique, et comparer de façon **équitable** :
- une référence naïve (même heure, semaine précédente) ;
- un modèle statistique classique : le lissage exponentiel de Holt-Winters ;
- un modèle de machine learning : le Random Forest, avec des variables temporelles.

## Données

Données ouvertes RTE éCO2mix, consommation nationale en temps réel au pas de 15 minutes :
https://odre.opendatasoft.com/explore/dataset/eco2mix-national-tr/

- Période utilisée : de début septembre au 8 novembre 2026.
- Les lignes sans consommation mesurée (heures à venir, pour lesquelles seules les prévisions RTE sont renseignées) sont supprimées.
- Les dates sont converties en UTC puis dans le fuseau Europe/Paris, pour gérer le passage à l'heure d'hiver.
- Un instantané des données est enregistré (`eco2mix_snapshot.csv`) pour rendre les résultats reproductibles.

## Méthodologie

1. Chargement, nettoyage et mise au pas régulier de 15 minutes.
2. Séparation chronologique : les **3 derniers jours** (288 pas de temps) servent de jeu de test.
3. Modèles comparés **au même horizon de 3 jours** :
   - **Naïf** : valeur observée à la même heure la semaine précédente.
   - **Holt-Winters** : saisonnalité additive hebdomadaire (672 pas de 15 minutes), sans tendance.
   - **Random Forest** : heure, minute, jour de la semaine et valeur à la même heure la semaine précédente. Les lags plus courts sont exclus, car ils ne sont pas connus 3 jours à l'avance.
4. À titre de comparaison, un **Random Forest à horizon 15 minutes**, qui utilise en plus la valeur observée au pas précédent et la veille à la même heure.
5. Évaluation par MAE, RMSE et MAPE.

## Résultats

Jeu de test : 5 au 8 novembre 2026.

| Modèle | Horizon | MAE (MW) | MAPE |
|---|---|---|---|
| Naïf (semaine précédente) | 3 jours | 6 433 | 11,6 % |
| **Holt-Winters hebdomadaire** | 3 jours | **2 065** | **3,8 %** |
| Random Forest | 3 jours | 5 199 | 9,3 % |
| Random Forest | 15 minutes | 596 | 1,1 % |

## Analyse

- **À horizon égal (3 jours), Holt-Winters avec saisonnalité hebdomadaire est le meilleur modèle**, et le seul à nettement battre la référence naïve.
- **Le Random Forest extrapole mal.** L'entraînement couvre septembre et octobre, alors que le test tombe début novembre, quand le chauffage fait monter la consommation au-delà des niveaux observés. Un Random Forest ne peut pas prédire au-delà des valeurs vues à l'entraînement, alors que Holt-Winters ajuste son niveau.
- **La saisonnalité hebdomadaire est essentielle** : un Holt-Winters avec une saisonnalité seulement journalière et une tendance additive dérive fortement (MAPE de 46 %).
- **La prévision à 15 minutes est un exercice beaucoup plus facile** : le modèle connaît la valeur observée juste avant. Ses 1,1 % d'erreur ne sont pas comparables aux résultats à 3 jours.

## Ce que j'ai corrigé par rapport à la première version

1. **Horizons différents.** La première version comparait un Random Forest à 15 minutes à un Holt-Winters à 3 jours, ce qui donnait un avantage artificiel au Random Forest. Tous les modèles sont maintenant comparés au même horizon.
2. **Valeurs futures vides.** Le jeu de données temps réel contient des lignes futures sans consommation mesurée. Elles étaient remplies par interpolation, ce qui créait une fin de série artificiellement plate dans le jeu de test. Elles sont maintenant supprimées.
3. **Saisonnalité.** Ajout de la saisonnalité hebdomadaire à Holt-Winters et d'une référence naïve.

## Limites et pistes d'amélioration

- Évaluation sur une seule période de test de 3 jours : il faudrait une évaluation sur plusieurs périodes, avec des origines de prévision glissantes (backtesting).
- L'optimisation de Holt-Winters hebdomadaire signale un problème de convergence : le résultat est à confirmer.
- Aucune variable météo : la température est pourtant déterminante pour la consommation en France, à cause du chauffage électrique.
- Ajouter une référence de persistance (dernière valeur observée) pour l'horizon de 15 minutes.

## Outils

Python, pandas, NumPy, statsmodels, scikit-learn, matplotlib

## Reproduire

Ouvrir `analyse_conso_electrique.ipynb` dans Google Colab (bouton « Open in Colab » en haut du notebook), puis exécuter les cellules dans l'ordre. Les données étant téléchargées en temps réel, les chiffres varient selon la date d'exécution ; utiliser `eco2mix_snapshot.csv` pour retrouver exactement les résultats ci-dessus.
