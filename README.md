# Atelier de Préparation de Données Images — IA Tri des Déchets

Ce projet implémente un pipeline complet d'ingénierie des données (*Data Engineering*) appliqué à la Vision par Ordinateur. L'objectif est de nettoyer, auditer et normaliser un jeu d'images hétérogènes afin de construire un dataset propre, équilibré et prêt pour l'entraînement de modèles de Machine Learning ou Deep Learning.

---

## Contexte du Projet
Une entreprise souhaite automatiser le tri des déchets à l'aide d'une Intelligence Artificielle. Le modèle doit classer les photographies de détritus parmi **6 catégories** :
* `cardboard` (cartons)
* `glass` (verre)
* `metal` (canettes, boîtes)
* `paper` (papiers, journaux)
* `plastic` (bouteilles, emballages)
* `trash` (déchets résiduels)

**Le problème initial :** Les images brutes proviennent de sources multiples et présentent de nombreuses anomalies visuelles et techniques (fichiers corrompus, images vides, disparités de résolution, doublons, classes très déséquilibrées).

---

## Structure de l'Arborescence
Le projet respecte scrupuleusement l'architecture standardisée suivante :

```text
atelier_prepa_donnees_images/
├── notebooks/
│   └── atelier_prepa_donnees_images.ipynb   # Pipeline de traitement pas à pas
├── reports/
│   └── audit_images.csv                     # Rapport d'audit quantitatif final
└── data/
    ├── raw/                                 # Images brutes d'origine (Lecture seule)
    │   └── [cardboard, glass, metal, paper, plastic, trash]/
    ├── cleaned/                             # Images saines, uniques, RGB et carrées
    │   └── [cardboard, glass, metal, paper, plastic, trash]/
    ├── train/                               # 70% des données propres (avec Data Augmentation)
    ├── val/                                 # 15% des données propres (Contrôle continu)
    └── test/                                # 15% des données propres (Évaluation finale)
```

---

## Étapes Clés du Pipeline (Jalons Réalisés)

### 1. Audit Technique et Exploration (`Parties 1 à 8`)
On a analysé chaque fichier brut à l'aide de **Pillow** et **Numpy** pour intercepter les anomalies :
* **Images corrompues :** Détectées via une structure de fichier cassée (`img.verify()`).
* **Images vides ou uniformes :** Identifiées automatiquement lorsque la variance/écart-type des pixels est inférieur à un seuil strict (`std < 2.0`).
* **Résolutions & Canaux :** Détection des formats trop petits (exclus si `< 64x64 px`), et des modes de couleurs hétérogènes (`RGB`, `RGBA`, `P`, `L`).
* **Doublons stricts :** Éliminés par hachage cryptographique **MD5** pour éviter les fuites de données (*Data Leakage*).
* **Contrôle visuel :** Affichage d'échantillons avec **Matplotlib** pour valider la cohérence des dossiers.

### 2. Transformation et Nettoyage (`Parties 9 à 11`)
* **Redimensionnement intelligent :** Toutes les images sont ramenées au format standard **224 × 224 pixels** avec conservation du ratio d'origine et ajout de *padding* (bandes noires).
* **Uniformisation des canaux :** Conversion forcée de tous les fichiers en mode **RGB pur (3 canaux)** en aplatissant les transparences.
* **Mise à l'échelle (Normalisation) :** Préparation des pixels en valeurs décimales comprises entre **0.0 et 1.0**.

### 3. Stratégie d'Entraînement (`Parties 12 et 13`)
* **Découpage Stratifié :** Séparation robuste via `splitfolders` en sous-ensembles **Train (70%)**, **Validation (15%)** et **Test (15%)** en conservant la distribution des classes.
* **Data Augmentation (Keras) :** Application ciblée de transformations aléatoires (rotations, zooms, translations, luminosité, contraste) **uniquement sur le dossier Train de la classe minoritaire (`trash`)** pour corriger le fort déséquilibre détecté sans fausser l'évaluation finale.

---

## 🚀 Fonctionnalités Bonus (Expertise Terrain)
* **Détection du Flou (OpenCV) :** Utilisation de la **variance du Laplacien** pour mesurer automatiquement la netteté des contours et exclure les clichés trop flous.
* **Normalisation des Étiquettes Textuelles (Regex) :** Pipeline de traitement de texte automatique pour nettoyer la ponctuation, corriger la casse et mapper les synonymes des métadonnées vers les étiquettes officielles de l'IA.

---

## Dépendances Requises
Pour exécuter le notebook, installez les paquets suivants :
```bash
pip install pillow numpy pandas matplotlib tensorflow split-folders opencv-python
```