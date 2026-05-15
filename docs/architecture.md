# Architecture EPM

## 🏗️ Vue d'ensemble

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND (React)                         │
│  Dashboard | Import | Mapping | Analysis | Settings             │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                    REST API (FastAPI)
                           │
┌──────────────────────────┴──────────────────────────────────────┐
│                    BACKEND (Python)                              │
│  ┌────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │
│  │CSV Parser  │ │Accounting│ │Financial │ │Analytics Engine  │ │
│  │& Validator │ │ Mapper   │ │Calculator│ │& AI Insights     │ │
│  └────────────┘ └──────────┘ └──────────┘ └──────────────────┘ │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────┴──────────────────────────────────────┐
│                      DATA LAYER                                  │
│  ┌────────────────┐ ┌──────────┐ ┌──────────────────────────┐  │
│  │   PostgreSQL   │ │  Redis   │ │  File Storage (S3/Local)  │  │
│  │  (Primary DB)  │ │ (Cache)  │ │  (CSV uploads)            │  │
│  └────────────────┘ └──────────┘ └──────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

## 📦 Composants Principaux

### 1. CSV Parser & Validator

**Responsabilités:**
- Détecter les séparateurs (`,` ou `;`)
- Détecter les décimales (`.` ou `,`)
- Valider les colonnes requises: Compte, Debit, Credit
- Nettoyer et normaliser les données
- Gérer les erreurs avec rapports détaillés

**Input:**
```
Compte,Debit,Credit
401001,1000.00,0.00
702001,0.00,5000.00
```

**Output:**
```json
{
  "status": "success",
  "rows": 1000,
  "data": [...],
  "metadata": {
    "separator": ",",
    "decimal": ".",
    "encoding": "utf-8"
  }
}
```

### 2. Accounting Mapper

**Classification automatique basée sur le numéro de compte:**

| Plage | Catégorie | Classe | Flux |
|-------|-----------|--------|------|
| 1xx | Immobilisations | Actif | Balance Sheet |
| 2xx | Stocks | Actif | Balance Sheet |
| 3xx | Clients | Actif | Balance Sheet |
| 4xx | Tiers divers | Actif/Passif | Balance Sheet |
| 5xx | Trésorerie | Actif | Balance Sheet |
| 6xx | Charges d'exploitation | Expense | P&L |
| 7xx | Produits d'exploitation | Revenue | P&L |
| 75 | Charges financières | Expense | P&L |
| 76 | Produits financiers | Revenue | P&L |
| 84 | Charges exceptionnelles | Expense | P&L |
| 85 | Produits exceptionnels | Revenue | P&L |

**Mapping configurable par plan comptable:**
- Français (PCG)
- IFRS
- Américain (GAAP)
- Personnalisé (user-defined)

### 3. Financial Calculator

**Formules de calcul:**

```python
# Income Statement
Revenue = SUM(comptes 7xx)
OperatingExpenses = SUM(comptes 6xx)
EBITDA = Revenue - OperatingExpenses
InterestExpenses = SUM(comptes 75)
Taxes = (EBITDA - InterestExpenses) * TaxRate
NetIncome = EBITDA - InterestExpenses - Taxes

# Balance Sheet
TotalAssets = SUM(comptes 1xx-5xx positif)
TotalLiabilities = SUM(comptes 4xx-5xx négatif)
Equity = TotalAssets - TotalLiabilities

# KPIs
GrossMargin% = (Revenue - COGS) / Revenue * 100
EBITDA% = EBITDA / Revenue * 100
CashRunway_months = CashPosition / BurnRate
Growth% = (CurrentRevenue - PreviousRevenue) / PreviousRevenue * 100
```

### 4. Analytics Engine

**Détection d'anomalies:**
- Écarts vs période précédente (seuil: 20%+)
- Valeurs extrêmes (Z-score > 2.5)
- Ratios hors normes
- Variations inhabituelles

**Génération d'insights:**
- Comparaisons MoM / YoY
- Tendances
- Alertes
- Recommendations

### 5. Frontend Components

**Pages principales:**
- `/` - Dashboard
- `/import` - CSV Upload
- `/mapping` - Accounting Configuration
- `/financials` - Financial Statements
- `/analytics` - KPIs & Analysis
- `/settings` - Configuration

## 🔄 Data Flow

### Import Workflow

```
1. User uploads CSV file
   ↓
2. Backend receives file
   ↓
3. CSV Parser:
   - Détecte séparateur
   - Détecte décimales
   - Valide colonnes
   - Nettoie données
   ↓
4. Validation:
   - Vérifie format
   - Vérifie équilibrage (Debit = Credit)
   - Détecte doublons
   ↓
5. Store in PostgreSQL:
   - imports table
   - import_rows table
   ↓
6. User confirms import
   ↓
7. Data processing:
   - Apply mapping rules
   - Generate financial statements
   - Calculate KPIs
   - Store in analytics tables
   ↓
8. Frontend receives summary
```

### Analysis Workflow

```
1. User selects period/filters
   ↓
2. Frontend requests data via API
   ↓
3. Backend queries database:
   - Financial data
   - KPIs
   - Anomalies
   - Insights
   ↓
4. Analytics engine:
   - Calculates comparisons
   - Detects trends
   - Generates comments
   ↓
5. Return JSON response
   ↓
6. Frontend renders charts/tables
```

## 🗄️ Database Schema (Simplified)

```sql
-- Users & Organizations
users (id, email, password, org_id, created_at)
organizations (id, name, settings, created_at)

-- Imports
imports (id, org_id, name, status, rows_count, created_at)
import_rows (id, import_id, account, debit, credit, validated)

-- Accounting Configuration
account_mappings (id, org_id, account_code, category, class, type)
accounting_plans (id, name, is_default, mappings_json)

-- Financial Data
journal_entries (id, import_id, account_code, debit, credit, account_category)
financial_periods (id, org_id, import_id, period_name, status, created_at)

-- Analysis & KPIs
financial_statements (id, period_id, type, data_json) -- P&L, Balance Sheet, etc
kpis (id, period_id, metric_name, value, period, calculated_at)
anomalies (id, period_id, type, description, severity, created_at)
insights (id, period_id, content, category, generated_at)
```

## 🔐 Security

- **Authentication:** JWT tokens
- **Authorization:** Role-based access control (RBAC)
- **Data:** Encrypted at rest & in transit
- **Audit:** Logging of all data changes
- **Input Validation:** Server-side validation

## 🚀 Scalability

**Stratégies:**
- Redis caching pour les calculs fréquents
- Async tasks pour l'import de gros fichiers
- Database indexing sur account_code, period_id
- API pagination
- CDN pour assets statiques

## 📈 Performance Targets

- CSV import: < 5s pour 10,000 lignes
- Dashboard load: < 2s
- Financial calculation: < 1s
- API response time: < 500ms (p95)
