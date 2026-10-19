# GeOMEES

**Scripts de préparation, de publication et de moissonnage des ressources de l’OMEES.**

GeOMEES est l’infrastructure de données géographiques de l’OMEES. Elle permet de réunir dans un même portail des ressources produites dans le cadre de l’observatoire et des ressources issues de catalogues externes, en publiant leurs métadonnées et, lorsque cela est prévu, leurs données géographiques.

Ce dépôt contient les données de travail, les scripts R et les configurations nécessaires à son alimentation. Il s’organise autour de **quatre cas d’usage**, regroupés en deux démarches : le moissonnage de ressources externes et la publication de nouvelles ressources.

- **Portail GeOMEES en production :** [Datahub OMEES](https://geodata.inrae.fr/geonetwork/OMEES/datahub/news)
- **Portail de test :** [Datahub OMEES — bac à sable](https://geodata.bac-a-sable.inrae.fr/geonetwork/PROJET_OMEES/datahub/news)
- **Dépôt de codes :** [ptaconet/geomees](https://github.com/ptaconet/geomees)

## 1. Socle technique

GeOMEES s’appuie sur la plateforme **Geodata INRAE**, fondée sur [geOrchestra](https://www.georchestra.org/fr/), et sur une base de données PostGIS de l’OMEES.

| Composant | Fonction |
| --- | --- |
| **PostgreSQL/PostGIS** | Stocker les données géographiques préparées pour l’OMEES. |
| **GeoNetwork** | Cataloguer les ressources à travers des fiches de métadonnées ISO 19115/19139. |
| **GeoServer** | Publier les données géographiques sous forme de services, notamment WMS pour leur visualisation et WFS pour l’accès aux entités. |
| **Datahub OMEES** | Proposer un point d’entrée pour découvrir et consulter les ressources de l’observatoire. |
| **MapStore** | Fournir les fonctions de visualisation cartographique de la plateforme. |

Le mot-clé **`omees`** dans les fiches de métadonnées permet leur référencement dans le Datahub OMEES. Il doit être conservé ou ajouté lors de la création des fiches.

## 2. Organisation du dépôt

| Dossier | Contenu | Usage |
| --- | --- | --- |
| [`ressources_omees/`](ressources_omees/) | Jeux de données propres à l’OMEES et scripts R pour les préparer avant publication : atlas écosanté et cartographie SfN de Lunaret. | Préparation en amont des cas 3 et 4. |
| [`publication_ressources_omees/`](publication_ressources_omees/) | Chaînes **geoflow**, configurations et tableaux de description des ressources et des contacts. | Cas 3 et 4. |
| [`moissonnage_ressources_externes/`](moissonnage_ressources_externes/) | Sélection des notices externes, fonctions de récupération/conversion et workflow de moissonnage. | Cas 1 et 2. |

### Fichiers de travail principaux

| Fichier ou sous-dossier | Rôle |
| --- | --- |
| `publication_ressources_omees/geoflow_entities/entities_metadata_only.xlsx` | Décrire les ressources à publier sans données spatiales associées, dont les catalogues externes. |
| `publication_ressources_omees/geoflow_entities/entities_metadata_with_data.xlsx` | Décrire les ressources à publier avec des données spatiales associées. |
| `publication_ressources_omees/geoflow_entities/contacts.xlsx` | Décrire les contacts associés aux ressources. |
| `publication_ressources_omees/geoflow_config_files/` | Configurer les sources et les services cibles des workflows geoflow. |
| `moissonnage_ressources_externes/catalogues_exterieurs_fiches_a_moissonner.csv` | Sélectionner les notices à moissonner dans les catalogues externes. |
| `moissonnage_ressources_externes/catalogues_exterieurs_fonctions_moissonnage/` | Rassembler les fonctions adaptées aux formats et interfaces des catalogues sources. |

## 3. Choisir le bon cas d’usage

| Cas | Besoin | Chaîne à utiliser | Résultat attendu |
| --- | --- | --- | --- |
| **1** | Référencer une ressource déjà décrite dans un catalogue externe. | Moissonnage R. | Fiche de métadonnées dans GeoNetwork, avec référence à la source. |
| **2** | Moissonner une notice et publier les données géographiques correspondantes du catalogue Montpellier Méditerranée Métropole (3M). | Moissonnage R avec fonctions DKAN. | Fiche GeoNetwork et données publiées dans GeoServer. |
| **3** | Créer une nouvelle fiche, sans publier de données spatiales associées. | geoflow « metadata only ». | Nouvelle fiche de métadonnées dans GeoNetwork. |
| **4** | Publier une nouvelle ressource géographique et sa fiche de métadonnées. | geoflow « metadata with data ». | Fiche GeoNetwork et services géographiques GeoServer. |

**Le cas 2 est actuellement limité au catalogue open data de la 3M.** Pour les autres catalogues externes, la chaîne décrite ici porte sur le moissonnage des métadonnées.

## 4. Préparer une session de travail

### Récupérer les codes

```bash
git clone https://github.com/ptaconet/geomees.git
cd geomees
```

Ouvrir le projet `geomees.Rproj` dans RStudio et travailler depuis la racine du dépôt. Les commandes R présentées ci-dessous supposent ce répertoire de travail.

### Disposer des paquets et accès nécessaires

| Traitement | Principaux paquets R utilisés |
| --- | --- |
| Préparation des données | `sf`, `tidyverse`, `readxl`, `DBI`, `RPostgres`, `janitor`, `jsonlite`. |
| Publication geoflow | `geoflow`, `jsonlite`, avec les composants de publication GeoNetwork et GeoServer utilisés par le workflow. |
| Moissonnage | `geometa`, `geonapi`, `geosapi`, `ows4R`, `rjson`, `sf`, `tidyverse`, `readxl` ; selon la source : `httr`, `xml2`, `XML`, `textutils`, `jsonlite`. |

Les traitements nécessitent des accès en écriture aux services cibles et, pour les données stockées dans PostGIS, à la base de données concernée.

Les scripts principaux chargent un fichier local **`credentials_geomees.R`**, à placer à la racine du projet. Ce fichier n’est pas fourni dans le dépôt examiné. Il doit définir les variables de connexion attendues :

- PostGIS : `geomees_db_host`, `geomees_db_port`, `geomees_db_dbname`, `geomees_db_user`, `geomees_db_password`.
- GeoNetwork : `geomees_gn_prod_url` et `geomees_gn_bacasable_url`.
- GeoServer : `geomees_gs_prod_url` et `geomees_gs_bacasable_url`.
- Comptes des services : `geomees_gn_gs_prod_user`, `geomees_gn_gs_prod_pwd`, `geomees_gn_gs_bacasable_user`, `geomees_gn_gs_bacasable_pwd`.

### Choisir l’environnement et adapter les chemins

Dans chacun des trois scripts principaux de publication/moissonnage, vérifier la variable :

```r
geomees_prod_or_bacasable <- "bacasable"  # ou "prod"
```

**Dans la version examinée, les trois scripts ciblent par défaut `"prod"`.** Modifier cette valeur dans le script lui-même pour effectuer un essai sur le bac à sable ; une valeur définie uniquement dans la console serait réaffectée par le script.

Adapter également les chemins absolus `/home/ptaconet/geomees/...` présents dans les scripts et les configurations JSON à l’emplacement du clone local. Les connexions PostGIS reposent sur les variables `geomees_db_*` : le sélecteur ci-dessus ne choisit pas une autre base de données.

Créer les dossiers de sortie si nécessaire :

```r
dir.create("xml_metadata", recursive = TRUE, showWarnings = FALSE)
dir.create("moissonnage_ressources_externes/data",
           recursive = TRUE, showWarnings = FALSE)
dir.create("publication_ressources_omees/geoflow_logs",
           recursive = TRUE, showWarnings = FALSE)
```

> Avant de lancer les commandes des sections suivantes, consulter les points d’adaptation de la section 10. Certains chemins du code actuel ne correspondent pas encore à la nouvelle arborescence.

## 5. Cas 1 — Moissonner des métadonnées externes

**Objectif :** récupérer des notices déjà présentes dans des catalogues externes et les intégrer à GeOMEES, en reprenant ou en convertissant leurs métadonnées au format ISO 19115/19139.

### Étape 1 — Décrire le catalogue source

Renseigner la fiche du catalogue dans :

- `publication_ressources_omees/geoflow_entities/entities_metadata_only.xlsx` ;
- `publication_ressources_omees/geoflow_entities/contacts.xlsx` pour les contacts associés.

Le workflow sélectionne les entités dont `Type` vaut **`directories`**. Il utilise leur `Identifier`, leur contact (`Creator`), leur `catalogue_metadata_standard` et leur champ `Relation`, notamment l’entrée « API catalogue » qui fournit l’URL de l’interface source.

### Étape 2 — Sélectionner les notices à moissonner

Compléter `moissonnage_ressources_externes/catalogues_exterieurs_fiches_a_moissonner.csv` :

| Colonne | Contenu attendu |
| --- | --- |
| `catalogue_identifier` | Identifiant du catalogue source, correspondant à son `Identifier` dans le tableau des entités. |
| `url` | Adresse de la page originale de la ressource. |
| `data_identifier` | Identifiant unique attendu par le catalogue source : UUID, DOI ou autre identifiant propre au portail. |
| `topics` | Catégories thématiques ISO à ajouter à la notice. Pour plusieurs valeurs, conserver le séparateur underscore suivi d’un saut de ligne utilisé dans les exemples du CSV. |

### Étape 3 — Vérifier la fonction de conversion

Utiliser la fonction existante pour le type de catalogue concerné. Pour un nouveau format ou une nouvelle interface, développer une fonction adaptée, l’enregistrer dans `catalogues_exterieurs_fonctions_moissonnage/` et ajouter son appel au workflow.

| Famille de catalogue | Valeur utilisée par le workflow | Fonction / fichier |
| --- | --- | --- |
| DKAN — portail open data 3M | `dkan` | `fonctions_moissonnage_dkan.R` |
| Dataverse — notamment IRD et Recherche Data Gouv | `dataverse` | `dataverse_create_iso19115()` dans `fonctions_moissonnage_dataverse.R` |
| GeoNetwork par API | `geonetwork_api` | `geonetwork_create_iso19115()` dans `fonctions_moissonnage_geonetwork.R` |
| GeoNetwork par CSW, schéma ISO 19115-3 | `geonetwork_csw_19115_3` | Même fonction. |
| Opendatasoft par CSW | `opendatasoft` | Même fonction. |
| DDI — PROGEDO | `ddi` | `ddi_create_iso19115()` dans `fonctions_moissonnage_ddi.R` |

Le support du comité technique cite notamment OREME, Copernicus, IGN, ODISSE, Atmo Occitanie, Data Hérault et PICTO-Occitanie parmi les catalogues concernés par les interfaces GeoNetwork/Opendatasoft. La valeur de configuration doit correspondre à l’interface réellement utilisée pour chaque source.

### Étape 4 — Lancer le moissonnage

Après configuration des accès, de l’environnement et des fichiers d’entrée :

```r
source("moissonnage_ressources_externes/catalogues_exterieurs_workflow_moissonnage.R")
```

Le workflow récupère les métadonnées, les enrichit (provenance, catalogue parent, mots-clés, etc.), ajoute le préfixe `omees_` à l’identifiant de fiche, sauvegarde les XML dans `xml_metadata/`, puis insère les notices dans GeoNetwork.

**À contrôler :** présence de la fiche, lien vers la source originale, rattachement au catalogue parent, catégories thématiques et mot-clé `omees`.

## 6. Cas 2 — Moissonner des métadonnées et des données spatiales de la 3M

**Objectif :** intégrer les métadonnées du catalogue open data de Montpellier Méditerranée Métropole et publier également les données géographiques associées dans GeoServer.

Suivre les étapes du cas 1 avec les notices du catalogue 3M et la valeur `dkan`. Le même script principal pilote les deux opérations :

```r
source("moissonnage_ressources_externes/catalogues_exterieurs_workflow_moissonnage.R")
```

Les fonctions du fichier `fonctions_moissonnage_dkan.R` recherchent une ressource ZIP, la téléchargent dans `moissonnage_ressources_externes/data/`, puis publient le shapefile dans GeoServer lorsqu’une ressource compatible est trouvée. Elles construisent également la fiche ISO et ses liens vers les services géographiques.

**Résultat attendu :** une fiche GeoNetwork et une couche GeoServer avec ses liens WMS/WFS. La publication des données spatiales dépend donc de la présence et de la compatibilité du fichier ZIP ; elle n’est pas garantie pour toute notice 3M.

Exemple présenté au comité technique : [Cours d’eau de Montpellier](https://geodata.inrae.fr/geonetwork/OMEES/datahub/dataset/omees_cours-deau-de-montpellier).

## 7. Cas 3 — Publier de nouvelles métadonnées seules

**Objectif :** créer une fiche pour une nouvelle ressource, sans publier de données spatiales associées dans GeoServer.

1. Décrire la ressource dans `publication_ressources_omees/geoflow_entities/entities_metadata_only.xlsx`.
2. **Laisser la colonne `Data` vide.**
3. Renseigner les contacts dans `contacts.xlsx` et leurs références dans le tableau des entités.
4. Ajouter le mot-clé `omees` et vérifier la configuration `config_geomees_metadata_only.json`.
5. Après adaptation des chemins et choix de l’environnement, exécuter :

```r
source("publication_ressources_omees/geoflow_omees_metadata_only.R")
```

Le script met à jour les paramètres de connexion du JSON, initialise le workflow et l’exécute. La configuration active la création des métadonnées ISO et leur publication dans GeoNetwork, avec les vignettes.

**À contrôler :** contenu de la fiche, contacts, liens, mot-clé `omees` et visibilité dans le Datahub. Ce parcours peut aussi servir à publier les fiches de catalogues décrites pour le cas 1.

## 8. Cas 4 — Publier de nouvelles métadonnées avec des données spatiales

**Objectif :** publier une nouvelle ressource géographique dans GeoServer et sa fiche de métadonnées dans GeoNetwork.

1. Préparer les données et les rendre accessibles dans la base PostGIS utilisée par le workflow. Pour les jeux propres à l’OMEES, voir la section 9.
2. Décrire la ressource dans `publication_ressources_omees/geoflow_entities/entities_metadata_with_data.xlsx`, en renseignant la référence aux données dans `Data` selon le modèle du classeur.
3. Renseigner les contacts dans `contacts.xlsx` et ajouter le mot-clé `omees`.
4. Vérifier `config_geomees_metadata_with_data.json`, les connexions PostGIS/GeoServer/GeoNetwork et les chemins du script.
5. Après ces adaptations, exécuter :

```r
source("publication_ressources_omees/geoflow_omees_metadata_with_data.R")
```

Le script complète la configuration avec les paramètres de connexion, puis lance geoflow pour publier les métadonnées et les services géographiques associés.

**À contrôler :** présence de la fiche GeoNetwork et de la couche GeoServer, géométrie et projection, affichage cartographique et fonctionnement des liens de services. Les journaux geoflow sont dirigés vers `publication_ressources_omees/geoflow_logs/`.

## 9. Préparer les jeux de données OMEES

La préparation des données est distincte de leur publication : les scripts ci-dessous construisent les tables géographiques utilisées ensuite dans le cas 4.

### Atlas écosanté

**Dossier :** `ressources_omees/atlas_ecosante/`.

Le script [`atlas_ecosante.R`](ressources_omees/atlas_ecosante/atlas_ecosante.R) prépare :

- les projets à partir de `projets/data.xlsx`, des correspondances thématiques de `projets/thematiques_omees.xlsx` et des géométries de `projets/sites_geo_wkt.csv` ;
- les acteurs à partir des exports RNSR et des fichiers de préparation/géocodage du sous-dossier `acteurs/`.

Il écrit les tables PostGIS **`projets_omees`** et **`acteurs_omees`**. La partie acteurs contient des reprises de fichiers intermédiaires et des corrections manuelles : examiner et exécuter le script par sections avant une mise à jour complète.

Le sous-dossier `atlas_ecosante_to_geonetwork/` contient aussi une fonction de création de métadonnées par projet et le script `iso19115_from_projets.R`. Ce traitement complémentaire nécessite encore l’adaptation des anciens chemins et l’initialisation de la connexion GeoNetwork `GN`.

### Cartographie SfN du Grand Parc de Lunaret

**Dossier :** `ressources_omees/datanat_lunaret/`.

Le script [`datanat_lunaret.R`](ressources_omees/datanat_lunaret/datanat_lunaret.R) lit les shapefiles `Point SfN.shp`, `Linéaire SfN.shp` et `Surface SfN.shp`, regroupe leurs géométries par identifiant, puis joint les attributs de `Grille SfN.xlsx`.

Après vérification des données et de la connexion PostGIS :

```r
source("ressources_omees/datanat_lunaret/datanat_lunaret.R")
```

Le résultat est écrit dans la table PostGIS **`datanat_lunaret`**. Le dossier contient aussi le projet QGIS, des GeoPackages et des styles SLD. Les exemples de publication directe GeoServer en fin de script sont commentés et ne sont pas exécutés par cet appel.

## 10. Points à adapter dans le code actuel

Les procédures ci-dessus reprennent les quatre cas d’usage du support du comité technique. La lecture du dépôt le 19 octobre 2026 fait apparaître les points suivants à régler avant une exécution sur un autre poste :

| Élément | Adaptation nécessaire |
| --- | --- |
| Chemins absolus | Remplacer les références à `/home/ptaconet/geomees/` dans les scripts et JSON. |
| Workflow geoflow avec données | Les appels finaux à `initWorkflow()` et `executeWorkflow()` pointent vers `publication_ressources_omees/geoflow/geoflow_config_files/`. Retirer le niveau `geoflow/` supplémentaire pour viser le dossier présent dans le dépôt. |
| JSON « avec données » | Le fichier examiné comporte des valeurs non conformes à la syntaxe JSON. Le corriger avant lancement ; `jsonlite::fromJSON()` doit pouvoir le lire avant même l’injection des paramètres de connexion. |
| Préparation des acteurs | Une lecture utilise encore `donnees_omees/atlas_ecosante/...` au lieu de `ressources_omees/atlas_ecosante/...`. Contrôler également les coordonnées, les corrections manuelles et les sites web. |
| Fiches individuelles des projets | Adapter les chemins hérités de l’ancien dépôt et initialiser les objets/paquets nécessaires à la publication. |
| Essai GBIF | `meta_harvest_gbif.R` est un essai séparé, non appelé par le workflow principal ; il dépend de fichiers locaux, dont une feuille XSLT absente du dépôt. |

### Effets des traitements

- Les scripts de préparation écrivent dans PostGIS avec `append = FALSE` ; contrôler les tables cibles avant une relance.
- Le moissonnage insère les métadonnées avec l’option `OVERWRITE` et configure les privilèges GeoNetwork, y compris ceux du groupe `all`.
- Les scripts geoflow réécrivent leurs JSON avec les paramètres de connexion. Conserver ces paramètres localement et vérifier les fichiers avant tout partage.

## 11. Compléments de documentation

Les prochaines précisions utiles sont : le détail des colonnes et de la syntaxe des classeurs geoflow, un exemple minimal validé pour chacun des quatre cas, les versions de R et des paquets, ainsi que les règles de choix des mots-clés pour catégoriser les ressources.

---

Documentation préparée à partir du support « GeOMEES — Copil technique n°1 — 21 octobre 2026 » et de la lecture du dépôt `ptaconet/geomees` le 19 octobre 2026. Les traitements n’ont pas été exécutés dans le cadre de cette rédaction.
