# Claude Documentation pour Herbie

## Vue d'ensemble du projet

Herbie est un package Python sophistiqué permettant de télécharger et de manipuler des données de modèles de prévision météorologique numérique (NWP). Il offre un accès simplifié à plus de 15 modèles météorologiques provenant de diverses sources (NOAA, ECMWF, etc.).

### Informations clés
- **Nom du package**: `herbie-data` (PyPI)
- **Version Python**: ≥ 3.10
- **Licence**: MIT
- **Auteur**: Brian K. Blaylock
- **Documentation**: https://herbie.readthedocs.io/
- **Repository**: https://github.com/blaylockbk/Herbie

## Architecture du projet

### Structure des répertoires

```
Herbie/
├── src/herbie/              # Code source principal
│   ├── __init__.py         # Point d'entrée du package
│   ├── core.py             # Classe Herbie principale (54KB, ~1300 lignes)
│   ├── cli.py              # Interface en ligne de commande
│   ├── accessors.py        # Extensions xarray personnalisées
│   ├── fast.py             # Fonctions optimisées pour téléchargements multiples
│   ├── models/             # Définitions spécifiques aux modèles
│   │   ├── hrrr.py        # High Resolution Rapid Refresh
│   │   ├── gfs.py         # Global Forecast System
│   │   ├── ecmwf.py       # Modèles ECMWF (IFS, AIFS)
│   │   ├── gefs.py        # Global Ensemble Forecast System
│   │   ├── rap.py         # Rapid Refresh
│   │   ├── nbm.py         # National Blend of Models
│   │   ├── rrfs.py        # Rapid Refresh Forecast System
│   │   └── ...            # Autres modèles (NAM, HAFS, CFS, etc.)
│   ├── paint/             # Outils de visualisation et cartographie
│   ├── toolbox/           # Utilitaires et fonctions helper
│   ├── crs.py             # Systèmes de coordonnées de référence
│   ├── wgrib2.py          # Interface avec wgrib2
│   ├── pick_points.py     # Extraction de données à des points spécifiques
│   └── hrrr_zarr.py       # Support pour archive HRRR en format Zarr
├── tests/                  # Tests unitaires et d'intégration
├── docs/                   # Documentation Sphinx
├── ci/                     # Scripts d'intégration continue
├── images/                 # Ressources graphiques (logo, etc.)
└── pyproject.toml         # Configuration du projet (PEP 621)
```

### Composants principaux

#### 1. **Classe `Herbie` (core.py)**
La classe centrale qui gère:
- Recherche de fichiers GRIB2 sur plusieurs sources de données
- Téléchargement complet ou partiel (subset) des fichiers
- Lecture des données avec xarray
- Gestion de l'inventaire des variables
- Cache local des fichiers téléchargés

**Attributs clés**:
- `date`: Date/heure de l'initialisation du modèle
- `model`: Nom du modèle (hrrr, gfs, etc.)
- `product`: Type de produit (sfc, prs, nat, etc.)
- `fxx`: Heure de prévision (forecast hour)
- `grib`: Chemin vers le fichier GRIB2
- `IDX_SUFFIX`: Extensions possibles pour les fichiers d'index

**Méthodes principales**:
- `inventory()`: Affiche toutes les variables disponibles
- `download()`: Télécharge le fichier (complet ou subset)
- `xarray()`: Charge les données dans un Dataset xarray
- `read_idx()`: Lit le fichier d'index pour connaître le contenu

#### 2. **Système de modèles (models/)**
Chaque modèle a son propre fichier définissant:
- Structure des URLs pour différentes sources
- Disponibilité temporelle des données
- Produits disponibles
- Patterns de nommage des fichiers

**Pattern utilisé**: Chaque modèle retourne un dictionnaire de templates avec:
```python
{
    'source_name': {
        'help': 'Description',
        'url': 'Template URL avec {date}, {fxx}, etc.',
        'file': 'Pattern du nom de fichier',
        'filter': Fonction de filtrage optionnelle
    }
}
```

#### 3. **Interface CLI (cli.py)**
Commandes disponibles:
- `herbie data`: Affiche l'URL d'un fichier
- `herbie download`: Télécharge des fichiers
- `herbie inventory`: Liste les variables disponibles
- `herbie xarray`: Charge les données (retourne métadonnées JSON)

#### 4. **Accessors xarray (accessors.py)**
Extensions personnalisées pour les Datasets xarray:
- `ds.herbie.crs`: Obtient le système de coordonnées
- `ds.herbie.polygon`: Crée un polygone de la zone couverte
- Méthodes de visualisation intégrées

## Dépendances principales

### Essentielles
- **cfgrib** (≥0.9.15): Lecture des fichiers GRIB2 via ecCodes
- **eccodes** (≥2.37.0): Bibliothèque C++ pour GRIB2
- **xarray** (≥2025.1.1): Manipulation de données multidimensionnelles
- **pandas** (≥2.1): DataFrames pour les index
- **numpy** (≥1.24): Calculs numériques
- **requests** (≥2.23.3): Requêtes HTTP
- **pyproj** (≥3.7.0): Projections cartographiques

### Optionnelles
- **cartopy**: Visualisation cartographique
- **matplotlib**: Graphiques
- **metpy**: Calculs météorologiques
- **pygrib**: Lecture alternative des GRIB2
- **scikit-learn**: Machine learning

## Workflow typique

### 1. Création d'un objet Herbie
```python
from herbie import Herbie

H = Herbie(
    date='2021-01-01 12:00',  # Date/heure d'initialisation
    model='hrrr',              # Modèle à utiliser
    product='sfc',             # Type de produit
    fxx=6                      # Heure de prévision
)
```

### 2. Recherche de données
Herbie cherche automatiquement dans plusieurs sources:
1. Cache local (`~/Library/Caches/herbie/` sur Mac)
2. NOMADS (serveurs NOAA)
3. AWS (via NODD)
4. Google Cloud Platform
5. Azure
6. Autres archives (Pando, etc.)

### 3. Téléchargement
```python
# Téléchargement complet
H.download()

# Téléchargement d'un subset (économise bande passante)
H.download(":TMP:2 m")  # Température à 2m
H.download(":500 mb")   # Tous les champs à 500 mb
```

### 4. Lecture des données
```python
ds = H.xarray("TMP:2 m")  # Retourne un xarray.Dataset
```

## Conventions de développement

### Style de code
- **Linter**: Ruff (configuré dans pyproject.toml)
- **Convention docstring**: NumPy style
- **Python version**: Compatible 3.10+
- **Type hints**: Encouragés (fichier `py.typed` présent)

### Tests
- Framework: **pytest**
- Localisation: `tests/`
- Coverage: Activé via pytest-cov
- Exécution: `pytest tests/` ou `make test`

### Documentation
- Outil: **Sphinx**
- Format: ReStructuredText (.rst) et MyST (Markdown)
- Build: `cd docs && make html`
- Extensions:
  - autodocsumm
  - nbsphinx (pour Jupyter notebooks)
  - sphinx-copybutton
  - sphinxcontrib-mermaid

### Gestion des versions
- Utilise **hatch-vcs** pour versioning depuis Git tags
- Version automatiquement générée dans `src/herbie/_version.py`
- Format: Semantic Versioning (MAJOR.MINOR.PATCH)

## Environnements de développement

### Installation pour développement
```bash
# Cloner le repo
git clone https://github.com/blaylockbk/Herbie.git
cd Herbie

# Créer environnement conda
conda env create -f environment-dev.yml
conda activate herbie-dev

# Installation en mode éditable
pip install -e .

# Ou avec uv (plus rapide)
uv pip install -e .
```

### Environnements disponibles
- **environment.yml**: Environnement minimal pour utilisation
- **environment-dev.yml**: Développement complet (avec docs, tests)
- **environment-test.yml**: Tests CI/CD uniquement

## Patterns de code importants

### 1. Gestion des erreurs réseau
Le code utilise des retry mechanisms pour les requêtes HTTP:
- Tentatives multiples avec backoff
- Gestion des timeouts
- Fallback vers sources alternatives

### 2. Cache des fichiers
```python
# Localisation du cache
Path.home() / ".cache" / "herbie"  # Linux
Path.home() / "Library" / "Caches" / "herbie"  # macOS
```

### 3. Parsing des index GRIB2
Les fichiers .idx contiennent:
- Byte ranges pour chaque variable
- Métadonnées (niveau, paramètre, heure)
- Permet téléchargements partiels via HTTP Range requests

### 4. Templating des URLs
Utilise `.format()` avec des variables nommées:
```python
url_template = "https://example.com/{YYYY}/{MM}/{DD}/file.grib2"
url = url_template.format(
    YYYY=date.year,
    MM=f"{date.month:02d}",
    DD=f"{date.day:02d}"
)
```

## Sources de données

### NOAA Open Data Dissemination (NODD)
- **AWS**: `s3://noaa-*` buckets
- **Google**: `gs://noaa-*` buckets
- **Azure**: Azure Blob Storage
- Accès public via HTTPS

### Archives universitaires
- **Pando** (University of Utah): Archive historique HRRR
- Format Zarr pour accès optimisé

### NOMADS
- Serveurs NOAA en temps réel
- Latence faible, mais rétention limitée (quelques jours)

## Contribution et développement

### Workflow Git
1. Fork le repository
2. Créer une branche feature: `git checkout -b feature/nom-feature`
3. Commit: `git commit -m "Description claire"`
4. Push: `git push origin feature/nom-feature`
5. Créer une Pull Request

### Checklist PR
- [ ] Tests passent (`pytest`)
- [ ] Code linté (`ruff check`)
- [ ] Formatting respecté (`ruff format`)
- [ ] Documentation mise à jour si nécessaire
- [ ] Exemples ajoutés si nouvelle fonctionnalité
- [ ] CHANGELOG.md mis à jour

### Ajout d'un nouveau modèle
1. Créer `src/herbie/models/nom_modele.py`
2. Définir la fonction `template()`
3. Ajouter la documentation dans `docs/gallery/`
4. Ajouter des tests dans `tests/`
5. Mettre à jour la liste des modèles dans README.md

## Debugging et troubleshooting

### Activer les logs détaillés
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Problèmes courants

**1. Fichier GRIB2 non trouvé**
- Vérifier la date (disponibilité des données)
- Essayer une source différente: `H.download(source='aws')`
- Vérifier les logs pour voir les URLs essayées

**2. Erreur cfgrib/eccodes**
- S'assurer que eccodes est bien installé
- Sur conda: `conda install -c conda-forge eccodes`
- Problème fréquent: version incompatible

**3. Projection/CRS incorrecte**
- Certains modèles ont des CRS complexes
- Utiliser `ds.herbie.crs` pour obtenir le CRS correct
- Référence: `src/herbie/crs.py`

**4. Mémoire insuffisante**
- Utiliser des subsets au lieu de fichiers complets
- Charger seulement les variables nécessaires
- Utiliser dask pour lazy loading (xarray le supporte)

## Commandes utiles

### Make targets
```bash
make test          # Exécute les tests
make lint          # Vérifie le style du code
make format        # Formate le code avec ruff
make docs          # Génère la documentation
make clean         # Nettoie les fichiers temporaires
```

### Tests spécifiques
```bash
pytest tests/test_core.py           # Test un fichier
pytest tests/test_core.py::test_nom # Test une fonction
pytest -v                            # Mode verbose
pytest --cov                         # Avec couverture
```

## Ressources additionnelles

### Documentation externe
- [GRIB2 Format](https://www.nco.ncep.noaa.gov/pmb/docs/grib2/)
- [cfgrib Documentation](https://github.com/ecmwf/cfgrib)
- [xarray Documentation](https://docs.xarray.dev/)
- [NOAA Model Documentation](https://www.ncei.noaa.gov/products/weather-climate-models)

### Projets connexes (même auteur)
- **goes2go**: Téléchargement de données satellite GOES
- **SynopticPy**: API Synoptic pour données mesonet
- **Carpenter Workshop**: Outils d'analyse météorologique
- **pandas-rose**: Roses des vents avec pandas

### Communauté
- **GitHub Discussions**: Questions et discussions
- **Issues**: Rapporter bugs et demander features
- **Show and Tell**: Partager vos utilisations

## Notes pour Claude

### Lors de modifications du code:
1. Toujours lire le fichier avant modification
2. Respecter le style NumPy pour les docstrings
3. Ajouter des tests pour les nouvelles fonctionnalités
4. Vérifier la compatibilité Python 3.10+
5. Mettre à jour la documentation si nécessaire

### Lors de l'ajout de fonctionnalités:
1. Vérifier si une fonctionnalité similaire existe déjà
2. Privilégier l'extension des classes existantes
3. Utiliser les accessors xarray pour les fonctionnalités Dataset
4. Documenter avec des exemples dans les docstrings

### Lors du debugging:
1. Vérifier les logs en mode DEBUG
2. Examiner le fichier d'index (.idx) pour comprendre la structure
3. Tester avec différentes sources de données
4. Valider les byte ranges pour les subsets

---

**Version du document**: 1.0
**Dernière mise à jour**: 2025-11-20
**Maintenu par**: Équipe de développement Herbie
