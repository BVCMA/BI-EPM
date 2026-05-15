# BI-EPM: Enterprise Performance Management Software

> Transformez votre balance comptable CSV en plateforme EPM sophistiquée

## 🎯 Vision

Créer une application SaaS EPM moderne et intuitive, capable de transformer automatiquement une balance comptable (CSV) en tableaux financiers dynamiques, KPIs et analyses de pilotage - inspirée par Pigment, Anaplan et Workday.

## 📋 Fonctionnalités MVP

### Phase 1: Import & Validation
- ✅ Upload CSV avec détection automatique des colonnes
- ✅ Support des séparateurs (`;` et `,`)
- ✅ Formats européens (. et , pour décimales)
- ✅ Validation des données avec messages d'erreur clairs

### Phase 2: Mapping Comptable
- ✅ Classification automatique des comptes (6xx→Charges, 7xx→Revenus, etc.)
- ✅ Mapping configurable par utilisateur
- ✅ Support des plans comptables standards (français, IFRS)

### Phase 3: États Financiers
- ✅ Compte de résultat
- ✅ Bilan
- ✅ Calcul EBITDA
- ✅ Résultat net
- ✅ Position de trésorerie

### Phase 4: KPIs & Analytics
- ✅ Marge brute
- ✅ EBITDA %
- ✅ Burn rate
- ✅ Cash runway
- ✅ Croissance CA
- ✅ Ratio charges/revenus

### Phase 5: Dashboard Interactif
- ✅ UI moderne minimaliste
- ✅ Graphiques interactifs (Recharts)
- ✅ Drill-down par compte
- ✅ Filtres et sélecteurs de période

### Phase 6: Intelligence Financière
- ✅ Détection d'anomalies
- ✅ Variations inhabituelles
- ✅ Commentaires auto-générés
- ✅ Insights financiers

## 🏗️ Stack Technique

**Frontend:**
- React 18 + TypeScript
- Recharts pour les graphiques
- TailwindCSS pour le design
- React Query pour la gestion d'état

**Backend:**
- Python FastAPI
- Pydantic pour la validation
- SQLAlchemy ORM

**Database:**
- PostgreSQL
- Redis (cache & jobs)

**Infrastructure:**
- Docker
- GitHub Actions CI/CD
- Vercel/Railway (deployment)

## 📁 Structure du Projet

```
BI-EPM/
├── docs/
│   ├── architecture.md
│   ├── api-specification.md
│   ├── data-model.md
│   ├── user-flows.md
│   └── roadmap.md
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── models/
│   │   ├── services/
│   │   └── schemas/
│   ├── tests/
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   └── types/
│   ├── public/
│   └── package.json
└── docker-compose.yml
```

## 🚀 Quick Start

### Prérequis
- Docker & Docker Compose
- Node.js 18+
- Python 3.11+
- PostgreSQL 15+

### Installation

```bash
# Clone
git clone https://github.com/BVCMA/BI-EPM.git
cd BI-EPM

# Backend
cd backend
pip install -r requirements.txt
python -m uvicorn app.main:app --reload

# Frontend (new terminal)
cd frontend
npm install
npm run dev
```

## 📊 Roadmap

### MVP (Q2 2026)
- Import CSV & validation
- Mapping comptable automatique
- États financiers de base
- Dashboard simple

### v1.0 (Q3 2026)
- KPIs avancés
- Intelligence financière
- UI polished

### v2.0 (Q4 2026)
- Prévisions budgétaires
- Consolidation multi-entités
- Scénarios & simulations

### v3.0 (2027)
- Planning collaboratif
- Forecast AI
- Reporting temps réel

## 📖 Documentation

- [Architecture complète](docs/architecture.md)
- [Spécification API](docs/api-specification.md)
- [Modèle de données](docs/data-model.md)
- [User Flows](docs/user-flows.md)
- [Roadmap détaillée](docs/roadmap.md)

## 🤝 Contributing

Les contributions sont les bienvenues! Consultez [CONTRIBUTING.md](CONTRIBUTING.md)

## 📄 License

MIT License - Voir [LICENSE](LICENSE)

## 👤 Auteur

BVCMA - Créateur du projet
