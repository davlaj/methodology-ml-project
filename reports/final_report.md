# Rapport – Prévision des ventes

## Question 1 : Préparation des données  
**Enjeux de qualité identifiés :**  
1. **Colonnes inutiles et redondantes** : suppression de `Country`, `Category`, `Row ID`, `Order ID`, `Customer Name`, `Quantity` (risque de data leakage).  
2. **Dates et temporalité** : conversion en datetime (`Order Date`, `Ship Date`) et création de variables (`Order_Year`, `Order_Month`, `Order_DayOfWeek`, `Order_Quarter`, `Is_EndOfYear`, `Delivery_Days`).  
3. **Valeurs extrêmes** : outliers observés via boxplots (`Sales`, `Discount`). Décision : conserver mais créer un indicateur `High_Discount` pour que le modèle apprenne l’effet des rabais élevés.  

**Visuels :**  
- Voir `reports/eda_report.html`
- Voir les visuels EDA dans le notebook

**Outputs :**  
- `data/interim/stores_sales_forecasting_interim.csv` : dataset nettoyé.  
- `data/processed/stores_sales_forecasting_processed.csv` : dataset enrichi avec features avancés (cyclicité, moyennes par état/catégorie, indicateur discount).  

---

## Question 2 : Insights et interprétation  
- **35% des ventes ne génèrent pas de profits** 
- **Rabais, facteurs géographiques & produits dominent** dominent pour expliquer la profitabilité.  
- **Temporalité** : tendance et saisonalité détectée, mais impact limités sur le modèle retenu.

**Décision d'opter pour un modèle en deux étapes:**
  1. Classification pour prédire la profitabilité des ventes
  2. Régression pour prédire le montant des ventes profitables

**Valeur pour les gestionnaires :**  
- Identifier **à l’avance les commandes non profitables** → ajuster la stratégie de rabais.  
- Estimer le **montant des ventes attendues** sur les commandes profitables → mieux gérer l’inventaire et la logistique.  

**Use cases concrets :**  
- Optimiser politiques de rabais.  
- Cibler régions/produits rentables.  
- Décider des ressources logistiques à mobiliser.  

---

## Question 3 : Solution ML
**Pipeline en deux étapes :**  
1. **Classification (XGBoostClassifier)**  
   - Objectif : prédire si une commande est profitable.  
   - Score : précision ≈ **90%**.  

2. **Régression conditionnelle (XGBoostRegressor)**  
   - Objectif : estimer le montant des ventes sur les commandes profitables.  
   - Résultats (cross-validation, échelle réelle) :  
     - **RMSE ≈ 419$**  
     - **MAE ≈ 197$ par commande**  
   - Comparaison : nettement meilleur que le baseline (prédiction par la moyenne).  
   - Early stopping utilisé pour limiter le surapprentissage.  

**Choix de l’approche :**  
- XGBoost = robuste, efficace sur données tabulaires hétérogènes.  
- Séparer classification & régression = plus clair pour l’explicabilité métier.  

---

## Question 4 : Dégradation de la performance  
**Risque de dérive des données** (distribution des rabais, évolution des canaux logistiques, croissance du marché ou changement de comportement des clients).  

**Approche MLOps (AWS) :**  
- **Détection** : Amazon SageMaker Model Monitor pour surveiller la dérive des distributions des features.  
- **Réentraînement** : déclencher un pipeline SageMaker (avec par exemple Step Functions) quand un seuil est dépassé.  
- **Versioning** : stocker modèles/datasets dans **S3** et **Model Registry** pour audit et rollback.  
- **Monitoring continu** : alertes CloudWatch sur métriques (ex. RMSE > seuil ou data drift) pour déclencher le pipeline de réentraînement.  

---

## Question 5 : Intégration de l’IA générative  
**Architecture proposée :**  
- **ML backend** (SageMaker endpoints) : classification + régression déployées.  
- **Orchestration** : Step Functions pour automatiser ingestion → prédiction → interprétation.  
- **Couche LLM** : via **AWS Bedrock** (Claude, Llama2) pour interface naturelle.  
- **Intégration dans un système interne de gestion**  pour évaluer la profitabilité, prévoir inventaire, rabais, et ce, par zones géographiques et produits.

**Exemple de prompt utilisateur :**  
> *« Explique-moi pourquoi les ventes prévues en Californie pour le produit X sont inférieures à la moyenne et propose des actions concrètes pour améliorer la profitabilité. »*  

---

## Résultats globaux  
- **Classification** : 90% de précision sur la profitabilité.  
- **Régression** : RMSE ~419$, MAE ~197$ → prédictions raisonnables compte tenu de la variabilité des ventes.  
- **Apport business** : workflow en 2 étapes déjà actionnable pour orienter les décisions stratégiques (rabais, inventaire, logistique).  