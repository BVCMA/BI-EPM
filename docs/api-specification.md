# API Specification - EPM Backend

## 🚀 Base URL

```
https://api.bi-epm.com/v1
```

## 🔐 Authentication

Tous les endpoints requièrent un token JWT dans le header:

```bash
Authorization: Bearer <JWT_TOKEN>
```

## 📋 Endpoints

### 1. Authentication

#### POST /auth/register

Créer un nouveau compte utilisateur.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "first_name": "John",
  "last_name": "Doe",
  "organization_name": "My Company"
}
```

**Response:** 201 Created
```json
{
  "user_id": 123,
  "email": "user@example.com",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "organization_id": 456
}
```

#### POST /auth/login

Authentifier un utilisateur.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123"
}
```

**Response:** 200 OK
```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "user_id": 123,
  "organization_id": 456,
  "role": "admin"
}
```

### 2. Imports

#### POST /imports/upload

Uploader un fichier CSV de balance comptable.

**Request:** multipart/form-data
```
file: <binary file>
name: "Balance comptable janvier 2026"
```

**Response:** 201 Created
```json
{
  "import_id": 789,
  "status": "processing",
  "file_name": "balance-01-2026.csv",
  "rows_count": 1250,
  "created_at": "2026-05-15T10:30:00Z"
}
```

#### GET /imports/{import_id}

Récupérer les détails d'un import.

**Response:** 200 OK
```json
{
  "import_id": 789,
  "name": "Balance comptable janvier 2026",
  "status": "validated",
  "rows_count": 1250,
  "rows_valid": 1248,
  "rows_invalid": 2,
  "metadata": {
    "separator": ",",
    "decimal": ".",
    "encoding": "utf-8"
  },
  "errors": [
    {
      "row": 245,
      "message": "Debit and Credit cannot both be non-zero"
    }
  ],
  "created_at": "2026-05-15T10:30:00Z",
  "processed_at": "2026-05-15T10:31:45Z"
}
```

#### GET /imports

Lister tous les imports.

**Query Parameters:**
- `page`: 1
- `per_page`: 20
- `status`: validated, draft, error

**Response:** 200 OK
```json
{
  "data": [
    {
      "import_id": 789,
      "name": "Balance comptable janvier 2026",
      "status": "validated",
      "rows_count": 1250,
      "created_at": "2026-05-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 42
  }
}
```

#### POST /imports/{import_id}/validate

Valider et traiter un import.

**Response:** 200 OK
```json
{
  "import_id": 789,
  "status": "validated",
  "period_created": "2026-01",
  "rows_processed": 1248,
  "financial_statements_generated": true
}
```

### 3. Accounting Configuration

#### GET /accounting/standards

Récupérer les standards comptables disponibles.

**Response:** 200 OK
```json
{
  "standards": [
    {
      "id": 1,
      "name": "PCG",
      "description": "Plan Comptable Général (France)",
      "is_system": true,
      "classes": 8 // number of account classes
    },
    {
      "id": 2,
      "name": "IFRS",
      "description": "International Financial Reporting Standards",
      "is_system": true
    }
  ]
}
```

#### GET /accounting/mappings

Récupérer les mappings comptables de l'organisation.

**Query Parameters:**
- `standard_id`: 1
- `page`: 1
- `per_page`: 100

**Response:** 200 OK
```json
{
  "data": [
    {
      "account_code": "601000",
      "account_label": "Achats de matières premières",
      "category": "Cost of Goods Sold",
      "class": "6",
      "type": "expense"
    }
  ],
  "pagination": {"page": 1, "total": 324}
}
```

#### PUT /accounting/mappings/{mapping_id}

Modifier un mapping comptable.

**Request:**
```json
{
  "custom_category": "Manufacturing Costs",
  "notes": "Updated category for better reporting"
}
```

**Response:** 200 OK
```json
{
  "mapping_id": 101,
  "account_code": "601000",
  "custom_category": "Manufacturing Costs",
  "updated_at": "2026-05-15T11:00:00Z"
}
```

### 4. Financial Periods

#### POST /periods

Créer une nouvelle période financière.

**Request:**
```json
{
  "import_id": 789,
  "period_name": "2026-01",
  "period_type": "monthly",
  "period_start": "2026-01-01",
  "period_end": "2026-01-31"
}
```

**Response:** 201 Created
```json
{
  "period_id": 234,
  "period_name": "2026-01",
  "status": "open",
  "created_at": "2026-05-15T10:35:00Z"
}
```

#### GET /periods

Lister les périodes financières.

**Response:** 200 OK
```json
{
  "data": [
    {
      "period_id": 234,
      "period_name": "2026-01",
      "period_type": "monthly",
      "status": "open",
      "journal_entries_count": 1248,
      "created_at": "2026-05-15T10:35:00Z"
    }
  ]
}
```

#### GET /periods/{period_id}

Récupérer les détails d'une période.

**Response:** 200 OK
```json
{
  "period_id": 234,
  "period_name": "2026-01",
  "status": "open",
  "journal_entries_count": 1248,
  "financial_statements_ready": true,
  "kpis_calculated": true,
  "anomalies_detected": 3
}
```

### 5. Financial Statements

#### GET /periods/{period_id}/statements/income-statement

Récupérer le compte de résultat.

**Response:** 200 OK
```json
{
  "statement_type": "income_statement",
  "period": "2026-01",
  "lines": [
    {
      "code": "REV",
      "label": "Total Revenues",
      "value": 125000.00,
      "percentage": 100.0,
      "level": 0,
      "lines": [
        {
          "code": "REV-PROD",
          "label": "Product Sales",
          "value": 95000.00,
          "percentage": 76.0,
          "level": 1
        },
        {
          "code": "REV-SERV",
          "label": "Service Revenue",
          "value": 30000.00,
          "percentage": 24.0,
          "level": 1
        }
      ]
    },
    {
      "code": "COGS",
      "label": "Cost of Goods Sold",
      "value": -45000.00,
      "percentage": 36.0,
      "level": 0
    },
    {
      "code": "GP",
      "label": "Gross Profit",
      "value": 80000.00,
      "percentage": 64.0,
      "level": 0
    },
    {
      "code": "OPEX",
      "label": "Operating Expenses",
      "value": -25000.00,
      "percentage": 20.0,
      "level": 0
    },
    {
      "code": "EBITDA",
      "label": "EBITDA",
      "value": 55000.00,
      "percentage": 44.0,
      "level": 0
    }
  ]
}
```

#### GET /periods/{period_id}/statements/balance-sheet

Récupérer le bilan.

**Response:** 200 OK
```json
{
  "statement_type": "balance_sheet",
  "period": "2026-01",
  "assets": {
    "current_assets": 150000.00,
    "fixed_assets": 250000.00,
    "total_assets": 400000.00
  },
  "liabilities": {
    "current_liabilities": 80000.00,
    "long_term_liabilities": 120000.00,
    "total_liabilities": 200000.00
  },
  "equity": 200000.00
}
```

### 6. KPIs & Analytics

#### GET /periods/{period_id}/kpis

Récupérer tous les KPIs d'une période.

**Response:** 200 OK
```json
{
  "period": "2026-01",
  "kpis": [
    {
      "name": "Gross Margin %",
      "code": "GROSS_MARGIN_PCT",
      "value": 64.0,
      "unit": "%",
      "target": 60.0,
      "variance": 4.0,
      "trend": "up",
      "category": "profitability"
    },
    {
      "name": "EBITDA %",
      "code": "EBITDA_PCT",
      "value": 44.0,
      "unit": "%",
      "target": 40.0,
      "variance": 4.0,
      "trend": "up"
    },
    {
      "name": "Current Ratio",
      "code": "CURRENT_RATIO",
      "value": 1.87,
      "unit": "ratio",
      "target": 1.5,
      "variance": 0.37,
      "trend": "up",
      "category": "liquidity"
    }
  ]
}
```

#### GET /periods/{period_id}/kpis/trends

Récupérer les tendances historiques des KPIs.

**Query Parameters:**
- `kpi_code`: GROSS_MARGIN_PCT
- `periods`: 6 (dernières 6 périodes)

**Response:** 200 OK
```json
{
  "kpi_code": "GROSS_MARGIN_PCT",
  "kpi_name": "Gross Margin %",
  "history": [
    {"period": "2025-08", "value": 58.5},
    {"period": "2025-09", "value": 60.2},
    {"period": "2025-10", "value": 61.5},
    {"period": "2025-11", "value": 62.0},
    {"period": "2025-12", "value": 63.1},
    {"period": "2026-01", "value": 64.0}
  ]
}
```

### 7. Anomalies & Insights

#### GET /periods/{period_id}/anomalies

Récupérer les anomalies détectées.

**Query Parameters:**
- `severity`: critical, high, medium, low
- `type`: variance, outlier, missing_data

**Response:** 200 OK
```json
{
  "anomalies": [
    {
      "anomaly_id": 501,
      "type": "variance",
      "severity": "high",
      "account_code": "602000",
      "metric_name": "Marketing Expenses",
      "expected_value": 5000.00,
      "actual_value": 8500.00,
      "variance_pct": 70.0,
      "description": "Marketing expenses increased by 70% compared to last month",
      "created_at": "2026-05-15T10:45:00Z"
    }
  ],
  "total_count": 1,
  "acknowledged_count": 0
}
```

#### GET /periods/{period_id}/insights

Récupérer les insights générés automatiquement.

**Query Parameters:**
- `type`: trend, comparison, alert, recommendation
- `category`: revenue, costs, cash, profitability

**Response:** 200 OK
```json
{
  "insights": [
    {
      "insight_id": 601,
      "type": "trend",
      "category": "revenue",
      "title": "Revenue growing strong",
      "content": "Total revenue increased from €110k (Dec) to €125k (Jan), a growth of +13.6%. Product sales remain the main driver with €95k.",
      "severity": "low",
      "is_automated": true,
      "created_at": "2026-05-15T11:00:00Z"
    },
    {
      "insight_id": 602,
      "type": "alert",
      "category": "costs",
      "title": "Operating expenses increased",
      "content": "Operating expenses jumped to €25k, up from €20k last month. This is primarily due to increased marketing spend (€3.5k vs €2k).",
      "severity": "medium",
      "is_automated": true
    }
  ]
}
```

### 8. Analytics & Comparisons

#### GET /analytics/periods/compare

Comparer deux périodes.

**Query Parameters:**
- `period_1`: 2026-01
- `period_2`: 2025-12
- `metrics`: revenue, expenses, ebitda, margin

**Response:** 200 OK
```json
{
  "comparison": {
    "periods": ["2025-12", "2026-01"],
    "metrics": [
      {
        "name": "Revenue",
        "values": [110000.00, 125000.00],
        "change": 15000.00,
        "change_pct": 13.6,
        "trend": "up"
      },
      {
        "name": "Operating Expenses",
        "values": [20000.00, 25000.00],
        "change": 5000.00,
        "change_pct": 25.0,
        "trend": "down"
      },
      {
        "name": "EBITDA",
        "values": [50000.00, 55000.00],
        "change": 5000.00,
        "change_pct": 10.0,
        "trend": "up"
      }
    ]
  }
}
```

### 9. Export & Reports

#### GET /periods/{period_id}/export/pdf

Exporter les états financiers en PDF.

**Query Parameters:**
- `statements`: income_statement, balance_sheet, cashflow (comma-separated)
- `include_kpis`: true
- `include_insights`: true

**Response:** 200 OK (binary PDF)

#### GET /periods/{period_id}/export/xlsx

Exporter en Excel.

**Response:** 200 OK (binary XLSX)

## 🔄 Error Responses

Tous les erreurs suivent ce format:

```json
{
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {
    "field": "additional_info"
  }
}
```

**Codes d'erreur communs:**
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden
- `404`: Not Found
- `422`: Unprocessable Entity (validation error)
- `500`: Internal Server Error

## 📊 Rate Limiting

- 100 requests par minute pour les endpoints publics
- 500 requests par minute pour les utilisateurs authentifiés

Headers de réponse:
```
X-RateLimit-Limit: 500
X-RateLimit-Remaining: 487
X-RateLimit-Reset: 1726234800
```
