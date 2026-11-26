# Module 03 – Behaviour Event Tracking & Data Ingestion

A runnable Django implementation of the uploaded Module 03 assignment for the AI-Based Product Recommendation System.

## Included
- Event taxonomy: impression, view, click, wishlist add/remove, cart add/remove, purchase
- JSON schema validation and timezone-aware timestamps
- Demo session consent checks
- Product/session reference validation
- Unique event IDs and retry-safe duplicate handling
- Single event API and bounded batch API with per-row outcomes
- Basic per-IP rate limiting and request-size limits
- Ingestion error records and staff-only summary endpoint
- Django admin, demo dashboard, sample payloads and automated tests

## Requirements
Python 3.12+ (your Python 3.14 installation is suitable) and pip.

## Windows PowerShell setup
Open PowerShell in the extracted `Module_03_Event_Tracking_Project` folder:

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python manage.py migrate
python manage.py seed_demo
python manage.py createsuperuser
python manage.py test
python manage.py runserver
```

If `py -3.14` is not recognized but `py` works, use `py -m venv .venv`. If PowerShell blocks activation, run:
`Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`
then activate again.

Open:
- Dashboard: http://127.0.0.1:8000/
- Admin: http://127.0.0.1:8000/admin/
- Single event: POST http://127.0.0.1:8000/api/v1/events
- Batch events: POST http://127.0.0.1:8000/api/v1/events/batch
- Ingestion summary: GET http://127.0.0.1:8000/api/v1/admin/events/ingestion-summary (staff login required)

## Test the API
Use Postman or the dashboard buttons. In Postman select POST, Body > raw > JSON, and paste `sample_payloads/single_event.json`.
For batch, use `sample_payloads/batch_events.json`. A repeated event_id returns `duplicate: true` and does not create a second row.

## Event contract
Required fields: `event_id` (UUID), `event_type`, `occurred_at` (ISO-8601 timestamp with timezone), `session_id`, `consent` (boolean).
Product events also require `product_id`. Optional fields: `pseudonymous_user_id`, `placement_id`, `request_id`, `properties` (JSON object).

The server checks both the event's consent flag and the stored session consent. Create/update sessions and product references in Django admin. The included demo seed creates:
- session: `demo-session-001` with consent granted
- products: `SKU-1001`, `SKU-1002`

## Important implementation notes
- SQLite and the local-memory cache make this easy to run locally. For production, use PostgreSQL and a shared Redis cache.
- The summary endpoint is restricted to authenticated Django staff users.
- This is the Module 03 event-ingestion module, not the complete multi-module recommendation engine. Identity, catalogue and recommendation services are represented by local reference models so the module can run independently.
- The purchase event is accepted as an authorized event type after session consent and reference validation; integrate real order authorization with the commerce/order module before production.
- Do not put real personal data, credentials or payment details in event properties.

## Load the real dataset (data/events.csv)
Stop the server (Ctrl+C), then push the Retailrocket events into the database and restart:
```powershell
python manage.py load_events_dataset              # first 100,000 rows
python manage.py load_events_dataset --limit 0    # all ~2.75M rows
python manage.py load_events_dataset --reset      # clear old events/errors first
python manage.py runserver
```
Refresh http://127.0.0.1:8000/ - the four cards now show the dataset counts and update live when you click the demo buttons.
Mapping: view -> view, addtocart -> cart_add, transaction -> purchase.
