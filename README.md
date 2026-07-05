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

## Design Decision

One design decision made for this project was to keep seat availability logic inside the reservation flow rather than in the frontend. This ensures that seat counts are validated consistently on the server side, preventing overbooking even if multiple clients submit reservations at the same time.

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
2. Create and activate a virtual environment based on your operating system.

### Windows

```bash
python -m venv .venv
.venv\Scripts\activate
```

### Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Ubuntu

```bash
sudo apt update
sudo apt install python3-venv python3-pip
python3 -m venv .venv
source .venv/bin/activate
```

3. Install the project dependencies from the requirements file.

```bash
pip install -r requirements.txt
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

## Postman Testing Checklist Guide

Here is the step-by-step walkthrough to check off every single item on your testing checklist using Postman.

### 1. Create an Event

- **Method:** `POST`
- **URL:** `http://127.0.0.1:8000/api/events/`
- **Body:** Select **raw** → change dropdown to **JSON**.
- **Payload:**

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

- **Verify:** Hit **Send**. Look for status `201 Created` and note down the `"id"` assigned to it (for example, `1`).

### 2. List All Events

- **Method:** `GET`
- **URL:** `http://127.0.0.1:8000/api/events/`
- **Body:** Select **none**.
- **Verify:** Hit **Send**. You should get a `200 OK` status with a JSON array listing the event you just created.

### 3. Filter Events by Status

- **Method:** `GET`
- **URL:** `http://127.0.0.1:8000/api/events/?status=upcoming`
- **Alternative Setup via Postman UI:** Instead of typing it in the URL bar, you can click the **Params** tab, type `status` under **Key**, and `upcoming` under **Value**.
- **Verify:** Hit **Send**. Ensure only events with `"status": "upcoming"` show up. Change it to `completed` to see an empty list.

### 4. Filter Events by Venue

- **Method:** `GET`
- **URL:** `http://127.0.0.1:8000/api/events/?venue=Bangalore`
- **Verify:** Hit **Send**. Because the API uses `venue__icontains`, searching for `Bangalore` will find the event even though the full text is `"NIMHANS Convention Centre, Bangalore"`.

### 5. Create a Reservation (Seats Deducted)

- **Method:** `POST`
- **URL:** `http://127.0.0.1:8000/api/reservations/`
- **Body:** **raw** → **JSON**
- **Payload:**

```json
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

- **Verify:** Hit **Send**. You should receive `201 Created`.
- **The Seat Deduction Check:** Go back to your **List All Events** request and hit send. Notice that `available_seats` has dropped from `500` to `498`.

### 6. Overbooking Attempt Returns 400

To test this easily, try to reserve more seats than remain.

- **Method:** `POST`
- **URL:** `http://127.0.0.1:8000/api/reservations/`
- **Body:** **raw** → **JSON**
- **Payload:**

```json
{
  "event": 1,
  "attendee_name": "Late Booker",
  "attendee_email": "late@example.com",
  "seats_reserved": 600
}
```

- **Verify:** Hit **Send**. The response status code should be `400 Bad Request`, and the body will read: `["Only 498 seat(s) available."]`.

### 7. Cancel a Reservation (Seats Restored)

Let's cancel the reservation created in step 5 (which should have an ID of `1`).

- **Method:** `POST`
- **URL:** `http://127.0.0.1:8000/api/reservations/1/cancel/`
- **Body:** Select **none**.
- **Verify:** Hit **Send**. You will get a `200 OK` status with `"status": "cancelled"`.
- **The Seat Restoration Check:** Call your **List All Events** endpoint once more. The `available_seats` field will bounce back up to `500`.

### 8. Filter Reservations by Event ID

- **Method:** `GET`
- **URL:** `http://127.0.0.1:8000/api/reservations/?event_id=1`
- **Verify:** Hit **Send**. This returns an array containing only the reservation objects attached to Event #1.
