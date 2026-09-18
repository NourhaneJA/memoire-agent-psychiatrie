# Assistant conversationnel à double rôle pour la psychiatrie professionnelle

Mémoire de master (Smart Systems) — ENSI, Université de la Manouba,
laboratoire RIADI. Encadrante~: Dr. Fadoua Ouamani.

Ce dépôt contient l'ensemble du code utilisé pour construire et évaluer
le système, dans l'ordre chronologique où il a été développé.

## Vue d'ensemble du pipeline

```
Données brutes → Nettoyage → Base de connaissances (RAG)
→ Comparaison de backbones → Évaluation du système → Validation à 102 cas
```

---

## Architecture

```
memoire-agent-psychiatrie/
├── README.md
├── requirements.txt
│
├── 01_donnees/
│   ├── 01_preparation_shifaa.ipynb
│   └── 02_preparation_kanakmi.ipynb
│
├── 02_base_connaissances/
│   ├── 01_construction_rag_guidelines_kanakmi_shifaa.ipynb
│   └── 02_correction_documents_shifaa_indexes.ipynb
│
├── 03_comparaison_backbones/
│   ├── 01_correction_juge_llm.ipynb
│   └── 02_analyse_comparaison_backbones.ipynb
│
├── 04_evaluation_systeme/
│   └── 01_evaluation_architecture_complete.ipynb
│
└── 05_validation_102_cas/
    ├── cas_cliniques_R1.csv
    ├── cas_cliniques_R2.csv
    └── 01_test_102_cas.ipynb
```


## Description détaillée de chaque fichier

### 01_donnees/

**`01_preparation_shifaa.ipynb`** --- Pipeline complet en 11 étapes~:
chargement brut, suppression des valeurs manquantes, dédoublonnage
conjoint (question + réponse + diagnostic), vérification qualité,
normalisation arabe, filtre de langue par ratio de caractères arabes,
statistiques de longueur, filtre hybride (plancher métier + plafond
P99), récapitulatif visuel en entonnoir, construction du document RAG
au format Markdown, sauvegarde finale (CSV + métadonnées + config
justificative). Produit les 34 912 lignes retenues sur 35 648 initiales.

**`02_preparation_kanakmi.ipynb`** --- Télécharge et nettoie
`Kanakmi/mental-disorders` (581 217 → 15 000 lignes après équilibrage à
3000 exemples par pathologie). Documente explicitement dans sa config
que ce dataset couvre BPD, Bipolar, Depression, Anxiety et
Schizophrenia --- pas le PTSD, couvert par ailleurs (Shifaa, guidelines).

### 02_base_connaissances/

**`01_construction_rag_guidelines_kanakmi_shifaa.ipynb`** --- Construit
la base finale~: détection automatique langue/pathologie/rôle depuis
l'arborescence des guidelines, nettoyage du bruit récurrent (copyright,
pagination), chunking par section Markdown plutôt que par nombre brut de
tokens, fusion avec Kanakmi et Shifaa déjà préparés, génération des
embeddings bge-m3, stockage dans ChromaDB persistant sur Drive, test de
récupération. Contient aussi les toutes premières briques de la
comparaison de backbones (fonction de récupération partagée, jeu de
test, chargement des modèles) ce qui explique le chevauchement avec
le dossier suivant.

**`02_correction_documents_shifaa_indexes.ipynb`** --- Corrige des
documents Shifaa déjà indexés dans ChromaDB (sans tout ré-indexer)~:
connexion à la collection existante, diagnostic, reconstruction des
documents nettoyés, suppression puis ré-ajout, vérification finale.

### 03_comparaison_backbones/

**`01_correction_juge_llm.ipynb`** --- Notebook autonome isolant la
correction apportée au modèle-juge (Phi-3-mini-4k-instruct) utilisé pour
noter les réponses des backbones comparés.

**`02_analyse_comparaison_backbones.ipynb`** --- Analyse les résultats
déjà générés pour comparer Qwen3-8B, Llama-3.1-8B et OpenBioLLM-8B~:
rejet des scores hors échelle, test de Wilcoxon sur toutes les paires,
grille de décision corrigée, figures B/C/D/F (performance par langue,
cas de sécurité critiques, intervalles de confiance bootstrap, stabilité
du classement).

### 04_evaluation_systeme/

**`01_evaluation_architecture_complete.ipynb`** :
jeux de test étiquetés (rôle, langue, crise, hors-scope), évaluation
bout-en-bout avec trace complète par scénario, simulation de la boucle
de clarification, évaluation à grande échelle sur 75 questions,
évaluation mixte R1+R2, visualisations, et un guide de rédaction pour
le chapitre expérimental.

### 05_validation_102_cas/

**`01_test_102_cas.ipynb`** test à 102 cas~: installation, chargement des modèles,
portage du pipeline, chargement des cas, exécution R1 (51 cas),
exécution R2 (51 cas), statistiques de synthèse, export final.

---

## Comment reproduire

1. `01_donnees/` --- régénérer les corpus nettoyés, dans l'ordre.
2. `02_base_connaissances/01_...`  construire la base ChromaDB;
   exécuter `02_correction_...` seulement si une correction ultérieure
   des documents Shifaa est nécessaire.
3. `03_comparaison_backbones/`
4. `04_evaluation_systeme/` et `05_validation_102_cas/` --- reproduisent
   l'ensemble des résultats du système.


