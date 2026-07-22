# ELD Trip Planner — Backend

Django + Django REST Framework backend for a full-stack ELD Trip Planner application.

The API accepts trip details, calculates route information, generates driver duty-status timelines, and creates auto-filled **ELD daily log sheets** based on FMCSA Hours-of-Service (HOS) rules for property-carrying drivers (70hr/8day cycle).

The React frontend is maintained as a separate project and consumes this API for trip planning, route display, and ELD visualization.

## Domain References

- [HOS Rules](documentation/hos-rules.md)
- [ELD Log Format](documentation/eld-log-format.md)

---


## 👨‍💻 Author


### Abdullah Omer
**Computer Science Student | Backend Developer | Full Stack AI Developer**

- 📧 **Email:** [abdullahhhomer@gmail.com](mailto:abdullahhhomer@gmail.com)
- 🐙 **GitHub:** [github.com/abdullahhhomer](https://github.com/abdullahhhomer)
- 💼 **LinkedIn:** [linkedin.com](https://www.linkedin.com/in/abdullah-omer-84b991181/)
---

# Stack

- Python 3.14
- Django 6.0
- Django REST Framework
- SQLite (development)
- PostgreSQL support for production deployments
- Free map services:
  - OpenStreetMap Nominatim for geocoding and reverse geocoding
  - Routing service abstraction for route calculations
- `timezonefinder` for offline timezone resolution from coordinates
- Gunicorn + WhiteNoise for production serving
- Railway deployment with automated migrations and health checks

---

# Project Structure

```
eld-trip-planner-api/

├─ config/
│  ├─ settings.py
│  ├─ urls.py
│  ├─ wsgi.py
│  └─ asgi.py
│
├─ apps/
│  └─ trips/
│     ├─ models.py
│     ├─ serializers.py
│     ├─ views.py
│     ├─ urls.py
│     ├─ admin.py
│     │
│     ├─ tests/
│     │
│     └─ services/
│        ├─ routing.py
│        ├─ timezone.py
│        ├─ geo.py
│        ├─ hos.py
│        └─ eld.py
│
├─ documentation/
│
├─ requirements.txt
├─ railway.toml
├─ manage.py
└─ .env.example
```

---

# Architecture Overview

The backend follows a service-based architecture:

### Routing Service

`services/routing.py`

Handles:

- Location geocoding
- Reverse geocoding
- Route calculations
- External API communication

### HOS Service

`services/hos.py`

Implements FMCSA Hours-of-Service simulation:

- 11-hour driving limit
- 14-hour duty window
- 30-minute break requirement
- 70 hours / 8 days cycle

### ELD Service

`services/eld.py`

Transforms duty timelines into daily ELD log sheets:

- 24-hour RODS format
- Duty status segments
- Daily totals
- Location tracking

### Timezone Service

`services/timezone.py`

Uses coordinates to determine the trip timezone and keeps the ELD timeline timezone-aware.

---

# Design Decisions

## Service Separation

Business logic is separated from external dependencies.

- HOS calculations are pure Python.
- ELD generation is independent from database/network calls.
- External API calls are isolated inside routing services.
- Database persistence happens through API views.

This keeps the system easier to test and maintain.

---

## Centralized HOS Configuration

FMCSA limits are stored in:

```
config/settings.py
```

Rules should not be duplicated throughout the codebase.

---

## Timezone-Aware Timeline

Trip timelines preserve timezone information to correctly handle:

- Daily log boundaries
- Midnight splits
- Local driver time calculations

---

## Error Handling

External API failures return clean API responses:

- User-friendly `400` responses
- Internal provider errors remain logged
- No raw third-party errors are exposed

---

# Setup Instructions

## Clone Repository

```bash
git clone <repository-url>

cd eld-trip-planner-api
```

---

## Create Virtual Environment

Windows:

```bash
python -m venv .venv

.venv\Scripts\activate
```

macOS/Linux:

```bash
python -m venv .venv

source .venv/bin/activate
```

---

## Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Configure Environment Variables

Create:

```
.env
```

from:

```
.env.example
```

Example:

```env
SECRET_KEY=your_secret_key

DEBUG=True

NOMINATIM_BASE_URL=https://nominatim.openstreetmap.org

GEOCODER_USER_AGENT=eld-trip-planner/1.0 your_email@example.com
```

---

## Database Setup

Run migrations:

```bash
python manage.py migrate
```

Optional admin user:

```bash
python manage.py createsuperuser
```

---

## Run Development Server

```bash
python manage.py runserver
```

Backend will run at:

```
http://127.0.0.1:8000/
```

---

# API Documentation

## Health Check

### GET

```
/api/health/
```

Example response:

```json
{
    "status": "ok"
}
```

---

## Geocode Location

### GET

```
/api/geocode/?q=San Francisco, CA
```

Returns location suggestions and coordinates.

---

## Plan Trip

### POST

```
/api/trips/
```

Example request:

```json
{
    "current_location": "Chicago, IL",
    "pickup_location": "Des Moines, IA",
    "dropoff_location": "Denver, CO",
    "current_cycle_used_hours": 10
}
```

---

Response includes:

- Resolved coordinates
- Route distance
- Route duration
- Route geometry
- Trip timezone
- Stops
- Duty-status timeline
- Generated ELD daily sheets

---

## Retrieve Trips

### GET

```
/api/trips/
```

Returns all planned trips.

---

### GET

```
/api/trips/{id}/
```

Returns:

- Trip details
- Route information
- ELD logs

---

# Testing

Run:

```bash
python manage.py test
```

Tests cover:

- HOS calculations
- ELD generation
- Routing services
- Timezone handling
- API workflows

External API calls are mocked to keep tests fast and reliable.

---

# Deployment (Railway)

This backend is configured for Railway deployment using:

```
railway.toml
```

Production uses:

- Gunicorn
- WhiteNoise
- Automatic migrations
- Static file collection
- Health checks

Railway start command:

```bash
python manage.py migrate --noinput &&
python manage.py collectstatic --noinput &&
gunicorn config.wsgi --bind 0.0.0.0:$PORT --workers 3
```

---

## Railway Environment Variables

Add these in Railway:

```env
SECRET_KEY=your_secret_key

DEBUG=False

NOMINATIM_BASE_URL=https://nominatim.openstreetmap.org

GEOCODER_USER_AGENT=eld-trip-planner/1.0 your_email@example.com
```

Railway automatically exposes the application through a public URL.

Health check:

```
/api/health/
```

---

# Frontend

The frontend is developed separately using:

- React
- TypeScript
- Tailwind CSS

It communicates with this backend through the REST API.

Frontend repository:

```
(Add frontend repository link here)
```

---

# Notes

- The project uses FMCSA HOS rules for property-carrying drivers.
- Sleeper berth splits and short-haul exceptions are outside the current scope.
- The planner assumes the driver starts the trip at planning time.
- ELD sheets are generated as complete 24-hour records with off-duty padding.
- Geocoding uses OpenStreetMap Nominatim and does not require paid API keys.
- Production deployments should use PostgreSQL instead of SQLite for persistent storage.

---


# License

This project was developed as a full-stack engineering project demonstrating:

- Django REST API development
- Route planning
- HOS simulation
- ELD generation
- Production deployment workflows
