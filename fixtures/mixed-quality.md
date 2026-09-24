# CLAUDE.md

Guidance for Claude Code working in this repository — a Python/Django REST API with a Postgres backend.

## Commands

```bash
# Run the dev server (auto-reloads)
make dev

# Run tests (pytest, with coverage)
make test

# Apply migrations + seed local data
make setup

# Lint + format (ruff + black)
make lint
```

## Environment

Copy `.env.example` to `.env` and fill in:

- `DATABASE_URL` — required, Postgres connection string
- `STRIPE_SECRET_KEY` — required for payments, use test key in dev
- `REDIS_URL` — required for Celery task queue

The dev server **will not start without `DATABASE_URL`** — it fails silently with a confusing connection error instead of a clear message. If you see `connection refused` on boot, check this first.

## Architecture decisions

- We use **class-based views** (DRF generic views) everywhere, **not** function-based views. This was decided in Q2 and there's a migration plan to standardize the old FBVs.
- All serializers live in `serializers.py` per-app, not inline with views.
- Permissions are class-based, defined in `permissions.py`, referenced by name in view `permission_classes`.

## Django REST Framework notes

Django REST Framework (DRF) is a powerful and flexible toolkit for building Web APIs in Django. It provides features like serialization, authentication, permission classes, pagination, and filtering out of the box. DRF builds on Django's class-based views and adds API-specific functionality.

To create a simple API endpoint with DRF, you define a view, a serializer, and wire it in URLs:

```python
from rest_framework import generics, serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = '__all__'

class ProductList(generics.ListCreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

Then register the view in your `urls.py` using `path('products/', ProductList.as_view())`. DRF will handle GET (list) and POST (create) automatically.

## Database gotchas

- **Never use `bulk_create` on models with signals.** Our `Order` model has a `post_save` signal that updates inventory — `bulk_create` skips signals, so inventory silently drifts. Always use a loop with `.save()` for orders, even though it's slower.
- **Migrations must be reversible.** Every migration needs a `reverse_code`. The deploy system runs `migrate` on rollback too — non-reversible migrations will corrupt the DB on a failed deploy.
- **The `User.email` field is NOT unique at the DB level**, only at the form validation level. Direct ORM writes can create duplicates. Always go through serializers for user creation.

## Project structure

The project is organized into several Django apps:

- `accounts/` — User authentication, registration, profile management. Contains views for login, logout, password reset, and email verification. Uses Django's built-in auth system with JWT tokens for API auth.
- `products/` — Product catalog, categories, inventory tracking. Models include Product, Category, and InventoryRecord.
- `orders/` — Order processing, checkout, payment integration with Stripe.
- `marketing/` — Coupons, promotions, email campaigns.

## Celery tasks

Background tasks use Celery with Redis as broker. Common patterns:

- Long-running operations (PDF generation, report building) go in tasks, never in views.
- Always pass IDs to tasks, not model instances — serialized instances go stale.
- The `@shared_task` decorator is used instead of `@app.task` so tasks work across multiple Django apps.

Celery is an asynchronous task queue/job queue based on distributed message passing. It allows you to run time-consuming operations in the background, outside the request-response cycle. Celery requires a broker (we use Redis) to pass messages between your Django application and the worker processes.

## Code style

- Write clean, readable, self-documenting code
- Every function should have a single responsibility
- Handle errors gracefully and never swallow exceptions silently
- Follow PEP 8 for all Python code

## Testing

Tests use pytest with pytest-django. The test database is created fresh on each run — do not rely on persistent test data. Fixtures live in `tests/conftest.py`. Use `pytest.mark.django_db` to grant DB access to a test.
