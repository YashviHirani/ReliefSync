# AGENTS.md - UnitySupport: Smart Resource Allocation Portal

## Project Purpose

UnitySupport is a Django 5 web application that aggregates scattered community needs data (paper surveys, field reports) and uses AI to surface urgent local needs and match available volunteers to tasks. It has three roles: Admin, NGO, and Volunteer.

---

## Architecture Rules (Never Deviate)

- No Django ORM. There is no models.py with database models. All data reads/writes go through Firestore via apps/core/firebase.py. Never run makemigrations or migrate for app models.
- Firebase Auth only. User authentication is handled by Firebase Auth client-side (ID token) plus verified server-side in apps/core/auth_backends.py. Never use Django's built-in User model or django.contrib.auth session login.
- Role enforcement via decorator. All role-protected views must use @role_required('admin'), @role_required('ngo'), or @role_required('volunteer') from apps/core/decorators.py. Never check roles inline in views.
- AI logic is isolated. All Claude API calls live exclusively in apps/needs/services/claude_summarizer.py. All scikit-learn/SHAP/Fairlearn logic lives in apps/matching/services/. Views call these service functions; they do not contain ML or API logic directly.
- DRF serializers validate Firestore data. Even though data comes from Firestore (not ORM), use DRF serializers in each app's serializers.py to validate and structure incoming/outgoing data.

---

## Firestore Collections (Source of Truth)

users/        {uid, role, name, email, created_at}
ngos/         {uid, name, verified, location, contact}
needs/        {id, ngo_id, title, description, urgency_score, status, skills_required, location, created_at}
tasks/        {id, need_id, ngo_id, title, volunteer_slots, filled_slots, status}
applications/ {id, task_id, volunteer_uid, status, applied_at}
volunteers/   {uid, skills[], availability, location, hours_logged, badges[]}
surveys/      {id, ngo_id, raw_text, claude_summary, processed_at}
matches/      {id, task_id, volunteer_uid, match_score, shap_values, created_at}

## Coding Conventions

- Python: PEP 8.
- Functions in services/ are pure functions (no side effects, no direct request access).
- All Firestore reads return plain Python dicts. Never pass Firestore document references between layers.
- URL namespaces: admin_portal, ngo_portal, volunteer_portal, accounts, api.
- Template inheritance: all templates extend templates/base.html. Role-specific layouts include the correct sidebar partial.
- Environment variables: all secrets (Firebase key path, Anthropic API key) must come from .env via python-dotenv. Never hardcode keys.
- Claude API calls must always include a system prompt that instructs the model to return structured JSON.

## Role-Based Access Map

| URL Prefix | Role Required | App |
| --- | --- | --- |
| /admin-portal/* | admin | admin_portal |
| /ngo/* | ngo | ngo_portal |
| /volunteer/* | volunteer | volunteer_portal |
| /api/v1/* | any (token) | api |
| /accounts/* | public | accounts |
| / | public landing page | landing page |

## AI Agent Behavior Rules

- Stay in scope. Only modify files within the app relevant to the current task. Do not refactor unrelated apps.
- No logic in templates. Django templates may use template tags and filters. No business logic and no direct Firestore calls inside templates; all data must come from the view context.
- Service functions are unit-testable. Write every service function so it can be called with plain Python inputs (no request, no Django context). This is mandatory for the ML matching and Claude summarization layers.
- Never skip role checks. Every view that is not on the public landing page or accounts URLs must be decorated with @role_required.
- Firestore schema is locked. Do not add new top-level collections without updating docs/firebase_schema.md first and noting it in your PR description.
- Claude API JSON output. When calling the Claude API for survey summarization, the system prompt must explicitly say: "Respond only with a valid JSON object. No preamble, no markdown, no explanation." Parse the response with json.loads() wrapped in try/except.
- SHAP explanations are always generated alongside match scores. Never return a match score from apps/matching/services/matcher.py without also returning the corresponding SHAP values for transparency.
- Fairness audit runs on batch operations only. fairness.py is called only when matching is run for 10 or more volunteers at once, not on single lookups.

## Common Pitfalls to Avoid

- Do not call firebase_admin.initialize_app() more than once. It is initialized once in apps/core/firebase.py using Django's AppConfig.ready().
- Do not store Firebase service account key in version control. Use the path from .env -> FIREBASE_CREDENTIALS_PATH.
- Do not use request.user for role checks. Use the decoded Firebase token from request.firebase_user (set by middleware).
- Do not create Django admin (/django-admin/) routes. Use the custom admin_portal app instead.

## Task Execution Order (for fresh setup)

1. Configure .env with FIREBASE_CREDENTIALS_PATH, ANTHROPIC_API_KEY, DJANGO_SECRET_KEY, and DEBUG.
2. Run pip install -r requirements.txt.
3. Set Firebase Firestore rules from firebase/firestore.rules.
4. Run python manage.py runserver (no migrations needed).
5. Create the first admin user directly in Firebase Console and set role: admin in Firestore users/ collection.
