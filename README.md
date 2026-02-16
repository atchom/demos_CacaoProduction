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

