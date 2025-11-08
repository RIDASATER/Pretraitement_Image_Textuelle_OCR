# Prétraitement d'images pour OCR

**Objectif**

Ce projet a pour but de fournir un pipeline de prétraitement d'images permettant d'obtenir des images prêtes à être analysées par un modèle OCR (Reconnaissance Optique de Caractères) en utilisant **OpenCV** et **NumPy**. Le README présente l'architecture du projet, les étapes de prétraitement recommandées, un exemple de code, des conseils pour améliorer la qualité OCR et des méthodes de debug.

---

## Table des matières

1. Présentation
2. Prérequis
3. Installation
4. Structure du projet
5. Pipeline de prétraitement (détail étape par étape)
7. Paramètres et conseils pratiques
8. Tests et validation
9. Dépannage (troubleshooting)
10. Contributions
11. Licence

---

## 1. Présentation

Ce README explique comment transformer une image brute (photo ou scan) en une image binaire/contraste optimisé et géométriquement corrigée, minimisant le bruit et les artefacts afin d'améliorer le taux de réussite d'un OCR (ex. Tesseract, modèles ML). Le pipeline est conçu pour être robuste et paramétrable.

## 2. Prérequis

* Python 3.12 (recommandé)
* Bibliothèques Python :
  * `opencv-python`: 
  * `numpy`: 2.2.x

Exemple d'installation :

```bash
pip install opencv-python 
```


## 3. Installation

1. Clonez le dépôt :

```bash
git clone <url-du-projet>
cd <nom-du-projet>
```

2. Créez un environnement virtuel (optionnel) et installez les dépendances.

## 4. Structure du projet

```
ocr-preprocess/
├─ README.md
├─ requirements.txt
├─ preprocess.ipynb     # script principal (exemples d'usage ci-dessous)
├─ data                 # contient l'image originale
├─ new_data             # Continet les images gnere lors de la phase de pretraitement
└─ outputs/             # images traitées
```

## 5. Pipeline de prétraitement (détail étape par étape)

Le pipeline suivant est une recommandation générale — adaptez les paramètres selon la qualité et la nature des images.

1. **Chargement**

   * Lire l'image avec OpenCV (`cv2.imread`) en couleur ou en niveaux de gris.

2. **Conversion en niveaux de gris**

   * `cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)`

3. **Normalisation / amélioration du contraste**

   * Histogram equalization : `cv2.equalizeHist(gray)` ou CLAHE pour éviter la sur-amplification.

4. **Réduction du bruit**

   * `cv2.medianBlur(gray, ksize)` pour enlever le bruit salé/poivre.
   * Opérations morphologiques pour supprimer petits points : `cv2.morphologyEx(img, cv2.MORPH_OPEN, kernel)` et `cv2.MORPH_CLOSE` pour combler petits trous.

5. **Binarisation**

   * Otsu: `_, th = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)`
   * Ou `cv2.adaptiveThreshold` (utile pour éclairage non uniforme).

6. **Suppression des petits artefacts**

   * Trouver contours et supprimer composants connexes petits par aire (`cv2.findContours` + `cv2.contourArea`).

7. **Amélioration des bordures de caractères**

   * `cv2.morphologyEx(binary, cv2.MORPH_CLOSE, kernel)` pour reconnecter traits cassés.
   * Attention aux tailles de kernel : trop grandes —> fusion de caractères.

8. **Deskew (correction d'inclinaison)**

   * Estimer l'angle via `cv2.findContours` ou `skimage` (Hough / moments) et appliquer une rotation inverse.

9. **Redimensionnement**

   * Redimensionner à une résolution adaptée à l'OCR (ex. hauteur du texte entre 30–60 px selon moteur).

10. **Finalisation**

    * Dilatation légère si les traits sont fins : `cv2.dilate` avec kernel petit.
    * Sauvegarder l'image traitée (`cv2.imwrite`).

---

### Utilisation

```bash
python preprocess.ipynb 
```

## 7. Paramètres et conseils pratiques

* **Taille de kernel** : commencez par `2x2` ou `3x3`. Pour documents imprimés propres, kernel plus petit ; pour photos bruitées, augmenter progressivement.
* **MedianBlur ksize** : valeurs impaires (3,5,7). 3 pour texte clair, 5/7 si bruit fort.
* **Binarisation adaptative** : utile pour pages prises en photo avec éclairage non uniforme.
* **Redimensionnement** : conservez l’aspect ratio. OCRs classiques exigent texte lisible: augmenter la résolution si le texte est trop petit.
* **Éviter** : sur-prétraitement qui efface ou fusionne les caractères.

## 8. Tests et validation

* Testez avec plusieurs images représentatives (scan propre, photo éclairage faible, photo avec mouchetures). Mesurez la qualité OCR (ex. taux de reconnaissance / distance de Levenshtein avec texte attendu).
* Sauvegardez les images intermédiaires (gray, enhanced, binary) pour debug.

## 9. Dépannage (troubleshooting)

* **Textes trop fins après binarisation** : appliquez une légère dilatation.
* **Caractères collés** : réduisez la taille du kernel de fermeture ou appliquez une érosion légère après dilatation.
* **Bruit autour des bords** : crop ou supprimer composants connexes de petite aire.
* **Rotation non corrigée** : vérifier la méthode d'estimation d'angle (Hough, moments, minAreaRect).

## 10. Contributions

Les contributions sont les bienvenues :

* Ouvrir une issue pour des bugs/idéés
* Faire une pull request pour des améliorations (ajout de nouveaux filtres, pipeline paramétrable, tests)

## 11. Licence

Ce projet est distribué sous la licence **MIT** — voir le fichier LICENSE pour les détails.

---

© Projet Prétraitement OCR — (OpenCV + NumPy)
