# UnitySupport

Smart Resource Allocation and Volunteer Coordination portal for NGOs.

UnitySupport is a Django-based platform designed to help NGOs capture community needs, coordinate volunteers, and improve task assignment quality with explainable and fairness-aware AI support.

## Current Status

This repository is currently scaffolded only.

- Folder and file structure is in place.
- No business logic has been implemented yet.
- Python files contain module-level placeholders.
- HTML files contain minimal placeholder markup.

## Tech Stack

- Backend: Django 5, Django REST Framework
- Database: Firebase Firestore (via firebase-admin SDK)
- Authentication: Firebase Auth (roles: Admin, NGO, Volunteer)
- AI Summarization: Anthropic Claude API
- Matching Engine: scikit-learn
- Explainability: SHAP
- Fairness Auditing: Fairlearn
- Frontend: Django Templates, Tailwind CSS, Alpine.js

## Key Architecture Notes

- Firestore is the primary data store.
- Traditional Django ORM models are intentionally not used.
- App boundaries are role- and domain-oriented under apps/.
- Environment-specific Django settings live in unity_support/settings/.

## Project Layout

High-level structure:

- unity_support/: Django project root
- unity_support/unity_support/: Django config package (settings, URLs, ASGI, WSGI)
- unity_support/apps/: Domain apps (core, accounts, portals, needs, matching, api)
- unity_support/templates/: Shared templates and public landing page
- unity_support/static/: CSS, JS, and image assets
- unity_support/firebase/: Firestore rules and service account template
- unity_support/ml/: ML notebooks and sample matching data
- unity_support/docs/: Architecture, schema, API, deployment docs

## App Overview

- core: Shared utilities, Firebase integration, auth middleware/backends, decorators
- accounts: Login, registration, and role selection flows
- admin_portal: Admin dashboard, NGO verification, reporting, map views
- ngo_portal: Need creation and task management for NGOs
- volunteer_portal: Volunteer task browsing, profile, and impact views
- needs: Community need workflows and survey summarization services
- matching: Volunteer-task matching, explanations, and fairness checks
- api: API root, versioning entry, and health/version endpoints

## Local Setup

1. Create and activate a virtual environment.
2. Install dependencies from requirements.txt.
3. Copy .env.example to .env and fill required values.
4. Run Django migrations and startup commands when implementation is added.

Example commands:

```bash
python -m venv .venv
# Windows PowerShell
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

## Environment Variables

Set these in .env (names can be adjusted during implementation):

- DJANGO_SECRET_KEY
- DJANGO_DEBUG
- DJANGO_ALLOWED_HOSTS
- FIREBASE_PROJECT_ID
- FIREBASE_CLIENT_EMAIL
- FIREBASE_PRIVATE_KEY
- FIREBASE_DATABASE_URL
- ANTHROPIC_API_KEY

## Firebase Notes

- Keep real Firebase credentials out of source control.
- Use firebase/serviceAccountKey.json.example as a template reference only.
- Store sensitive values in environment variables or secure secret management.

## Documentation

- docs/architecture.md
- docs/firebase_schema.md
- docs/api_endpoints.md
- docs/deployment.md

## Development Roadmap

1. Finalize settings and environment management.
2. Implement Firebase initialization and auth verification in core.
3. Build role-based portal routes and baseline views.
4. Add Firestore repository/service layer patterns.
5. Implement needs summarization and matching services.
6. Add DRF endpoints and API documentation.
7. Add tests, CI checks, and deployment pipeline.

## License

TBD
