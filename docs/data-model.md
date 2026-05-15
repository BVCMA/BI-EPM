# Modèle de Données EPM

## 📊 Schema PostgreSQL Complet

### 1. Tables d'Organisation & Authentification

```sql
CREATE TABLE organizations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    industry VARCHAR(50),
    country VARCHAR(2),
    accounting_standard VARCHAR(50) DEFAULT 'PCG', -- PCG, IFRS, GAAP
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    org_id INTEGER NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    role VARCHAR(20) DEFAULT 'user', -- admin, manager, user, viewer
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_org_id ON users(org_id);
CREATE INDEX idx_users_email ON users(email);
```

### 2. Tables d'Import

```sql
CREATE TABLE imports (
    id SERIAL PRIMARY KEY,
    org_id INTEGER NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id),
    name VARCHAR(255) NOT NULL, -- "Balance comptable Janvier 2026"
    description TEXT,
    file_path VARCHAR(500),
    file_size INTEGER, -- bytes
    status VARCHAR(50) DEFAULT 'draft', -- draft, processing, validated, error
    error_message TEXT,
    rows_count INTEGER,
    rows_valid INTEGER,
    rows_invalid INTEGER,
    metadata JSONB, -- separator, decimal, encoding
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    processed_at TIMESTAMP
);

CREATE TABLE import_rows (
    id SERIAL PRIMARY KEY,
    import_id INTEGER NOT NULL REFERENCES imports(id) ON DELETE CASCADE,
    row_number INTEGER,
    account_code VARCHAR(20) NOT NULL,
    account_label VARCHAR(255),
    debit DECIMAL(15, 2) DEFAULT 0,
    credit DECIMAL(15, 2) DEFAULT 0,
    is_valid BOOLEAN DEFAULT true,
    validation_errors TEXT, -- JSON array of errors
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_import_rows_import_id ON import_rows(import_id);
CREATE INDEX idx_import_rows_account ON import_rows(account_code);
```

### 3. Tables de Configuration Comptable

```sql
CREATE TABLE accounting_standards (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL, -- PCG, IFRS, GAAP, CUSTOM
    description TEXT,
    is_system BOOLEAN DEFAULT true, -- System or custom
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE account_classes (
    id SERIAL PRIMARY KEY,
    standard_id INTEGER NOT NULL REFERENCES accounting_standards(id),
    code VARCHAR(10), -- 1, 2, 3, 4, 5, 6, 7
    name VARCHAR(100), -- Assets, Liabilities, Equity, Expenses, Revenue
    side VARCHAR(10), -- debit, credit
    is_balance_sheet BOOLEAN,
    is_income_statement BOOLEAN,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE account_categories (
    id SERIAL PRIMARY KEY,
    standard_id INTEGER NOT NULL REFERENCES accounting_standards(id),
    class_id INTEGER NOT NULL REFERENCES account_classes(id),
    name VARCHAR(100), -- Immobilisations, Stocks, Clients, etc.
    pattern VARCHAR(50), -- Regex pattern like "^1[0-9]{2}"
    min_range INTEGER,
    max_range INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE account_mappings (
    id SERIAL PRIMARY KEY,
    org_id INTEGER NOT NULL REFERENCES organizations(id),
    account_code VARCHAR(20) UNIQUE NOT NULL,
    account_label VARCHAR(255),
    standard_category_id INTEGER REFERENCES account_categories(id),
    custom_category VARCHAR(100),
    is_active BOOLEAN DEFAULT true,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_account_mappings_org ON account_mappings(org_id);
CREATE INDEX idx_account_mappings_code ON account_mappings(account_code);
```

### 4. Tables de Périodes & Données Financières

```sql
CREATE TABLE financial_periods (
    id SERIAL PRIMARY KEY,
    org_id INTEGER NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    import_id INTEGER REFERENCES imports(id),
    period_name VARCHAR(50) NOT NULL, -- "2026-01", "Janvier 2026"
    period_type VARCHAR(20) DEFAULT 'monthly', -- monthly, quarterly, annual
    period_start DATE,
    period_end DATE,
    status VARCHAR(50) DEFAULT 'draft', -- draft, open, closed, archived
    is_consolidated BOOLEAN DEFAULT false,
    parent_period_id INTEGER REFERENCES financial_periods(id),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE journal_entries (
    id SERIAL PRIMARY KEY,
    period_id INTEGER NOT NULL REFERENCES financial_periods(id) ON DELETE CASCADE,
    import_row_id INTEGER REFERENCES import_rows(id),
    account_code VARCHAR(20) NOT NULL,
    account_label VARCHAR(255),
    debit DECIMAL(15, 2) DEFAULT 0,
    credit DECIMAL(15, 2) DEFAULT 0,
    amount DECIMAL(15, 2), -- debit - credit (signed)
    account_category_id INTEGER REFERENCES account_categories(id),
    is_consolidated BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_journal_entries_period ON journal_entries(period_id);
CREATE INDEX idx_journal_entries_account ON journal_entries(account_code);
CREATE INDEX idx_journal_entries_category ON journal_entries(account_category_id);
```

### 5. Tables d'États Financiers

```sql
CREATE TABLE financial_statements (
    id SERIAL PRIMARY KEY,
    period_id INTEGER NOT NULL REFERENCES financial_periods(id) ON DELETE CASCADE,
    statement_type VARCHAR(50) NOT NULL, -- income_statement, balance_sheet, cashflow
    status VARCHAR(50) DEFAULT 'draft', -- draft, ready, locked
    data JSONB NOT NULL, -- Structure détaillée de l'état
    calculated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(period_id, statement_type)
);

CREATE TABLE statement_line_items (
    id SERIAL PRIMARY KEY,
    statement_id INTEGER NOT NULL REFERENCES financial_statements(id) ON DELETE CASCADE,
    line_code VARCHAR(50),
    line_label VARCHAR(255),
    line_level INTEGER, -- 0 = section, 1 = subsection, 2 = line
    value DECIMAL(15, 2),
    percentage DECIMAL(5, 2), -- % of total
    sequence INTEGER,
    parent_line_id INTEGER REFERENCES statement_line_items(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_statement_line_items_statement ON statement_line_items(statement_id);
```

### 6. Tables de KPIs & Métriques

```sql
CREATE TABLE kpi_definitions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL, -- "Gross Margin %"
    code VARCHAR(50) UNIQUE NOT NULL, -- "GROSS_MARGIN_PCT"
    description TEXT,
    formula TEXT, -- Expression calculée
    category VARCHAR(50), -- profitability, liquidity, efficiency
    is_system BOOLEAN DEFAULT true,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE kpis (
    id SERIAL PRIMARY KEY,
    period_id INTEGER NOT NULL REFERENCES financial_periods(id) ON DELETE CASCADE,
    kpi_def_id INTEGER NOT NULL REFERENCES kpi_definitions(id),
    value DECIMAL(15, 2),
    unit VARCHAR(20), -- %, currency, ratio, days
    target_value DECIMAL(15, 2),
    variance DECIMAL(15, 2), -- actual - target
    trend VARCHAR(20), -- up, down, neutral
    calculated_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE kpi_history (
    id SERIAL PRIMARY KEY,
    kpi_id INTEGER NOT NULL REFERENCES kpis(id),
    period_id INTEGER NOT NULL REFERENCES financial_periods(id),
    value DECIMAL(15, 2),
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_kpis_period ON kpis(period_id);
CREATE INDEX idx_kpi_history_kpi ON kpi_history(kpi_id);
```

### 7. Tables d'Anomalies & Insights

```sql
CREATE TABLE anomalies (
    id SERIAL PRIMARY KEY,
    period_id INTEGER NOT NULL REFERENCES financial_periods(id) ON DELETE CASCADE,
    anomaly_type VARCHAR(50), -- variance, outlier, missing_data, unbalanced
    severity VARCHAR(20) DEFAULT 'medium', -- low, medium, high, critical
    account_code VARCHAR(20),
    metric_name VARCHAR(100),
    expected_value DECIMAL(15, 2),
    actual_value DECIMAL(15, 2),
    variance_pct DECIMAL(5, 2),
    description TEXT,
    is_acknowledged BOOLEAN DEFAULT false,
    acknowledgment_by INTEGER REFERENCES users(id),
    acknowledgment_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE insights (
    id SERIAL PRIMARY KEY,
    period_id INTEGER NOT NULL REFERENCES financial_periods(id) ON DELETE CASCADE,
    insight_type VARCHAR(50), -- trend, comparison, alert, recommendation
    category VARCHAR(50), -- revenue, costs, cash, profitability
    title VARCHAR(255),
    content TEXT,
    is_automated BOOLEAN DEFAULT true,
    severity VARCHAR(20), -- low, medium, high
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_anomalies_period ON anomalies(period_id);
CREATE INDEX idx_insights_period ON insights(period_id);
```

### 8. Tables d'Audit & Historique

```sql
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    org_id INTEGER NOT NULL REFERENCES organizations(id),
    action VARCHAR(100), -- create, update, delete, import, export
    entity_type VARCHAR(50), -- import, mapping, period, etc
    entity_id INTEGER,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_org ON audit_logs(org_id);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at);
```

## 📈 Données Exemple (Sample Data)

### Import de Balance Comptable

```csv
Compte,Debit,Credit
101000,50000.00,0.00
201000,25000.00,0.00
311000,15000.00,0.00
401000,0.00,8000.00
512000,12000.00,0.00
601000,3000.00,0.00
602000,2000.00,0.00
701000,0.00,45000.00
702000,0.00,8000.00
```

### Structure JSON - Income Statement

```json
{
  "period": "2026-01",
  "lines": [
    {
      "code": "REV",
      "label": "Revenues",
      "level": 0,
      "value": 53000.00,
      "lines": [
        {
          "code": "REV-PROD",
          "label": "Product Sales",
          "level": 1,
          "value": 45000.00
        },
        {
          "code": "REV-SERV",
          "label": "Service Revenue",
          "level": 1,
          "value": 8000.00
        }
      ]
    },
    {
      "code": "COGS",
      "label": "Cost of Goods Sold",
      "level": 0,
      "value": 0.00
    },
    {
      "code": "GP",
      "label": "Gross Profit",
      "level": 0,
      "value": 53000.00,
      "percentage": 100.0
    },
    {
      "code": "OPEX",
      "label": "Operating Expenses",
      "level": 0,
      "value": 5000.00,
      "lines": [
        {
          "code": "OPEX-RENT",
          "label": "Rent & Facilities",
          "level": 1,
          "value": 3000.00
        },
        {
          "code": "OPEX-ADMIN",
          "label": "Admin & Other",
          "level": 1,
          "value": 2000.00
        }
      ]
    },
    {
      "code": "EBITDA",
      "label": "EBITDA",
      "level": 0,
      "value": 48000.00,
      "percentage": 90.6
    },
    {
      "code": "NI",
      "label": "Net Income",
      "level": 0,
      "value": 48000.00,
      "percentage": 90.6
    }
  ]
}
```

## 🔑 Relations Principales

```
organizations
    ├── users
    ├── accounting_standards → account_classes → account_categories
    ├── account_mappings
    └── financial_periods
        ├── imports → import_rows
        ├── journal_entries
        ├── financial_statements → statement_line_items
        ├── kpis → kpi_history
        ├── anomalies
        └── insights
```

## 🚀 Optimisations

**Indexes clés:**
- org_id, period_id sur tables fréquemment interrogées
- account_code pour les recherches de comptes
- created_at pour les tris chronologiques
- Composite indexes pour les queries complexes

**Partitioning (future):**
- Partitionner journal_entries par period_id pour les grands volumes
- Archiver les old periods dans une partition séparée

**Materialized Views (future):**
- V_PERIOD_SUMMARIES: Résumés pré-calculés par période
- V_TREND_ANALYSIS: Données historiques pré-agrégées
