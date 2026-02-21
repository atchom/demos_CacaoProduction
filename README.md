# demos_CacaoProduction
## 📊 ARCHITECTURE RECOMMANDÉE POUR VOS PACKAGES STAGING

```
📁 Projet SSIS: CacaoETL_Staging
│
├── 📁 Packages (20 packages au total)
│   │
│   ├── 📁 01_PostgreSQL (6 packages)
│   │   ├── STG_PG_Fournisseurs.dtsx
│   │   ├── STG_PG_CategoriesMateriel.dtsx
│   │   ├── STG_PG_CommandesFournisseurs.dtsx
│   │   ├── STG_PG_InventaireLogistique.dtsx
│   │   ├── STG_PG_MouvementsStock.dtsx
│   │   └── STG_PG_MaintenanceEquipements.dtsx
│   │
│   ├── 📁 02_SQLServer_CoerMetier (8 packages)
│   │   ├── STG_SS_Plantations.dtsx
│   │   ├── STG_SS_Agriculteurs.dtsx
│   │   ├── STG_SS_Recoltes.dtsx
│   │   ├── STG_SS_Exportations.dtsx
│   │   ├── STG_SS_Clients.dtsx
│   │   ├── STG_SS_Cooperatives.dtsx
│   │   ├── STG_SS_Parametres.dtsx
│   │   └── STG_SS_StockFeves.dtsx   ✅ (PACKAGE DE DEPART)
│   │
│   ├── 📁 03_SQLServer_Tracabilite (3 packages)
│   │   ├── STG_SS_UtilisationIntrants.dtsx
│   │   ├── STG_SS_AffectationEquipement.dtsx
│   │   └── STG_SS_ControleQualite.dtsx
│   │
│   └── 📁 04_FichiersCSV (6 packages)
│       ├── STG_CSV_Fermentation.dtsx
│       ├── STG_CSV_Formations.dtsx
│       ├── STG_CSV_TransactionsLocales.dtsx
│       ├── STG_CSV_ObservationsChamp.dtsx
│       ├── STG_CSV_CommandesFournisseurs.dtsx
│       └── STG_CSV_DonneesMeteo.dtsx
│
├── 📁 GestionnairesConnexions (connexions partagées)
│   ├── PostgreSQL_CACAO.conmgr
│   ├── SQLServer_CacaoProduction.conmgr
│   ├── SQLServer_CacaoStaging.conmgr
│   └── CSV_FileConnection.conmgr
│
└── 📁 MasterPackages (pour orchestration)
    ├── MASTER_Staging_PostgreSQL.dtsx
    ├── MASTER_Staging_SQLServer.dtsx
    ├── MASTER_Staging_CSV.dtsx
    └── MASTER_Staging_Complete.dtsx (tous)

```
## 📦 FLUX SSIS : STG_SS_Recoltes.dtsx
### 📋 STRUCTURE DU FLUX
```
[Source OLE DB] 
    ↓
[Derived Column 1] (Corrections de base)
    ↓
[Derived Column 2] (Calculs et enrichissements)
    ↓
[Conditional Split] (Séparation données valides / erreurs)
    ├── [Valides] → [Sort] → [Destination Net]
    └── [Erreurs] → [Destination Erreurs]
```
### 📥 ÉTAPE 1 : SOURCE OLE DB
```
-- Source: Recoltes (table originale)
SELECT 
    RecolteID,
    PlantationID,
    AgriculteurID,
    DateRecolte,
    Saison,
    PoidsCabosses,
    PoidsFevesFraiches,
    TauxExtraction,
    PrixAchatKG,
    ModePaiement,
    StatutPaiement,
    Observations
FROM Recoltes;
```
#### Configuration :

- **ADO.NET Source Editor**
- **ADO.NET Connection Manager**: `ATCHOM.CacaoProductionDB.sa`
- **Data access mode**: `SQL command`
- **Columns**: Toutes les colonnes sélectionnées

<img src="https://github.com/atchom/demos_CacaoProduction/raw/eb1bbece9c6e5fec47ce439eea02349125eb45cc/Images/connection%20manager.png" width="600" alt="Connection Manager">

### 📝 ÉTAPE 2 : DERIVED COLUMN 1 - CORRECTIONS DE BASE
|Derived Column Name | Derived Column | Expression|
|---------------------|----------------|------------|
|PoidsFevesFraiches | Replace 'PoidsFevesFraiches' | (PoidsFevesFraiches < 0) ? 0 : PoidsFevesFraiches|
|PoidsCabosses | Replace 'PoidsCabosses' | (PoidsCabosses < 0) ? 0 : PoidsCabosses|
|TauxExtraction | Replace 'TauxExtraction' | (TauxExtraction <= 0 || ISNULL(TauxExtraction)) ? (PoidsCabosses > 0 ? (PoidsFevesFraiches / PoidsCabosses) * 100 : 0) : TauxExtraction|
|PrixAchatKG | Replace 'PrixAchatKG' | (PrixAchatKG < 0) ? 0 : PrixAchatKG|
|Saison | Replace 'Saison' | TRIM((ISNULL(Saison) || (Saison != "Grande" && Saison != "Petite")) ? "Non spécifiée" : Saison)|
|ModePaiement | Replace 'ModePaiement' | TRIM((ISNULL(ModePaiement) || ModePaiement == "") ? "Non spécifié" : ModePaiement)|
|StatutPaiement | Replace 'StatutPaiement' | TRIM((ISNULL(StatutPaiement) || StatutPaiement == "") ? "En attente" : StatutPaiement)|
|Observations | Replace 'Observations' | (ISNULL(Observations)) ? "Aucune observation" : Observations|
#### Appercu
<img src="https://github.com/atchom/demos_CacaoProduction/blob/db0f6f0e20700d9fd39bddaac04457930d93d95f/Images/derive_column.png" width="600" alt="Connection Manager">

### 📊 ÉTAPE 3 : DERIVED COLUMN 2 - CALCULS ET ENRICHISSEMENTS
|Derived Column Name | Derived Column | Expression|
|---------------------|----------------|------------|
|TauxPerte | Add as new column | (PoidsCabosses > 0) ? (((PoidsCabosses - PoidsFevesFraiches) / PoidsCabosses) * 100) : 0|
|MontantTotal | Add as new column | PoidsFevesFraiches * PrixAchatKG|
|AnneeRecolte | Add as new column | YEAR(DateRecolte)|
|MoisRecolte | Add as new column | MONTH(DateRecolte)|
|DateControle | Add as new column | GETDATE()|
|QualiteDonnee | Add as new column | (PoidsFevesFraiches <= 0 || PoidsCabosses <= 0) ? "Invalide" : (TauxExtraction < 20 ? "Faible" : (TauxExtraction > 30 ? "Élevé" : "Normal"))|

### 📊 ÉTAPE 4 :DERIVED COLUMN 2 - DERIVED_MOTIFREJET
Column Name: MotifRejet
```
Expression 
   (PoidsFevesFraiches <= 0 ? "Poids négatif ou nul, " : "") +
   (PoidsCabosses <= 0 ? "Poids cabosses nul, " : "") +
   (TauxExtraction <= 0 ? "Taux extraction invalide, " : "") +
   (TauxExtraction > 100 ? "Taux extraction > 100%, " : "") +
   (PlantationID <= 0 ? "PlantationID invalide, " : "") +
   (AgriculteurID <= 0 ? "AgriculteurID invalide, " : "") +
   (ISNULL(DateRecolte) ? "Date récolte manquante, " : "")
```
### 🔀 ÉTAPE 4 : CONDITIONAL SPLIT
<img src="https://github.com/atchom/demos_CacaoProduction/blob/40f2cbdcc5384d8ff5a4d1c4d7b1533e2ed4df50/Images/Motif_rejet.png" width="600" alt="Connection Manager">

### 🔄 ÉTAPE 5 : DERIVED SORT
Colonnes de tri :
| Input Column    | Sort Type   | Sort Order |
|-----------------|-------------|------------|
| RecolteID       | ascending   | 1          |

☑ Remove rows with duplicate sort values  ← COCHER
#### Appercu
<img src="https://github.com/atchom/demos_CacaoProduction/blob/960151be6be13f1e94b1fe7b0945eb4587925cea/Images/delete%20doublons2.png" width="600" alt="Connection Manager">
