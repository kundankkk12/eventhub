# EventHub API

EventHub is a Django REST Framework project for managing events and seat reservations. It supports event creation, reservation handling, seat availability validation, cancellation flow, and request logging.

## Features

- Create and list events
- Filter events by status or venue
- Reserve seats for an event with availability checks
- Cancel reservations and restore seats automatically
- Log incoming request details through custom middleware

## Tech Stack

- Python 3.10+
- Django
- Django REST Framework
- SQLite (default database)

## Project Structure

```text
eventhub/
├── manage.py
├── eventhub/
│   ├── settings.py
│   └── urls.py
└── events/
    ├── models.py
    ├── serializers.py
    ├── views.py
    ├── urls.py
    ├── middleware.py
    └── migrations/
```

## Installation

1. Clone or enter the project directory.
2. Create and activate a virtual environment.

```bash
python -m venv .venv
.venv\Scripts\activate
```

3. Install the required packages.

```bash
pip install django djangorestframework
```

4. Apply database migrations.

```bash
python manage.py migrate
```

5. Start the development server.

```bash
python manage.py runserver
```

The API will be available at http://127.0.0.1:8000/api/.

## API Endpoints

### Events

- `GET /api/events/` - List all events
- `POST /api/events/` - Create a new event
- `GET /api/events/?status=upcoming` - Filter by status
- `GET /api/events/?venue=bangalore` - Filter by venue (case-insensitive)

Example event payload:

```json
{
  "title": "PyCon India 2025",
  "venue": "NIMHANS Convention Centre, Bangalore",
  "date": "2025-09-20",
  "total_seats": 500,
  "available_seats": 500,
  "status": "upcoming"
}
```

### Reservations

- `GET /api/reservations/` - List reservations
- `POST /api/reservations/` - Create a reservation
- `POST /api/reservations/{id}/cancel/` - Cancel a reservation and restore seats
- `GET /api/reservations/?event_id=1` - Filter reservations by event ID

Example reservation payload:

```json
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

## Middleware

The project includes a custom request logging middleware in [events/middleware.py](events/middleware.py). It prints request details such as method, path, status code, and execution time to the console during development.

Example log output:

```text
INFO 2026-07-05 10:14:22,104 events.middleware POST /api/events/ - 201 - 0.04s
INFO 2026-07-05 10:15:01,837 events.middleware GET /api/events/?status=upcoming - 200 - 0.02s
```