# Méthodes ensemblistes pour la régression en C++ (arbres, bagging, gradient boosting)

Projet du M1 CHPS (UVSQ – Université Paris-Saclay), réalisé en équipe de quatre. Implémentation from scratch en C++17 d'un arbre de régression, du bagging et du gradient boosting pour prédire le temps d'exécution d'une factorisation LU (DGETRF) à partir de ses paramètres, puis optimisation (histogrammes, SoA, OpenMP).

## Contenu

- Arbre de régression (critère MSE), bagging (bootstrap) et gradient boosting (résidus de la perte L2)
- Tests unitaires GTest, build CMake, analyse et graphes en Python
- Rapports : `rapport-partie1.pdf` (semestre 1), `rapport-partie2.pdf` (semestre 2)

## Résultats

| Optimisation | Gain mesuré |
|---|---|
| Filtrage des valeurs aberrantes (jeu 15k_hvs) | RMSE de 3,30 à 0,097 |
| Calcul du MSE par la formule de Koenig-Huygens | entraînement de 18 s à 0,4 s |
| Histogrammes (256 bins) et AoS vers SoA | ×18,8 sur l'arbre, ×5 sur le bagging, ×24 sur le boosting (moyennes géométriques) |
| OpenMP, bagging (N=100, 74 719 lignes) | 38,7 s (naïf), 2,33 s (1 thread), 1,40 s (2 threads) |
| OpenMP, boosting (N=100) | 12,0 s (naïf), 0,44 s (1 thread), 0,24 s (4 threads) |

Scalabilité testée sur un nœud 128 cœurs (MesoNET Zen) : speedup d'environ ×3 jusqu'à 4 threads, puis dégradation (bande passante mémoire, overhead OpenMP). Étude des compilateurs (GCC, Clang, ICX, AOCC) et analyse perf (IPC, cache).

## Comparaison avec les bibliothèques (74 719 lignes)

| Modèle | RMSE | Temps (s) |
|---|---|---|
| Notre bagging (N=100, D=12, 2 threads) | 0,0347 | 1,40 |
| scikit-learn RandomForest | 0,0163 | 1,88 |
| Notre boosting (N=100, D=4, 4 threads) | 0,0666 | 0,247 |
| XGBoost (hist) | 0,0309 | 0,096 |
| scikit-learn (hist) | 0,0318 | 0,126 |
| LightGBM | 0,0312 | 0,362 |

## Limites

Notre implémentation est plus rapide que scikit-learn (bagging) et LightGBM (boosting) sur ces mesures, mais moins précise (binning uniforme simplifié). Le RMSE du boosting optimisé augmente anormalement à 100 estimateurs, et la scalabilité OpenMP est limitée à environ 4 threads. Pistes : GPU, gestion mémoire fine, parallélisation hybride.
