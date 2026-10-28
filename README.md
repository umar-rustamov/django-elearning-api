# Educa: E-learning Platform with a REST API

A Django e-learning platform where instructors build courses from modules and mixed content, students enroll and study, and a Django REST Framework API exposes subjects and courses.

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-4.1-092E20?logo=django&logoColor=white)
![DRF](https://img.shields.io/badge/Django%20REST%20Framework-3.14-A30000?logo=django&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-cache-DC382D?logo=redis&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-3-003B57?logo=sqlite&logoColor=white)

## Features

**Instructors (content management)**
- Create, edit and delete their own courses; access is limited to the course owner and controlled by Django model permissions.
- Add and edit course modules with an inline formset.
- Attach different content types to a module (text, image, file, video) through a generic relation (`ContentType` + `GenericForeignKey`).
- Reorder modules and contents with drag and drop (html5sortable); the new order is saved through JSON views.
- A custom `OrderField` that numbers modules within a course and contents within a module automatically.

**Students**
- Sign up and get logged in right away.
- Enroll in courses and see the list of enrolled courses.
- Study a course module by module; videos are embedded with `django-embed-video`.

**Catalog**
- Public course list, filterable by subject, with module counts per course.
- Subject and course lists are cached in Redis; student course pages use `cache_page` (15 minutes).

**REST API (Django REST Framework)**
- Read-only endpoints for subjects and courses (courses are served by a `ReadOnlyModelViewSet` registered on a router).
- Enroll in a course with `POST`, using HTTP Basic authentication.
- Get a course's full contents, available only to enrolled students through a custom `IsEnrolled` permission.

**Developer tools**
- Django Debug Toolbar and `django-redisboard` for inspecting Redis from the admin.
- A fixture with sample subjects (Mathematics, Music, Physics, Programming).

## API endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/subjects/` | List subjects | none |
| GET | `/api/subjects/<id>/` | Subject detail | none |
| GET | `/api/courses/` | List courses with their modules | none |
| GET | `/api/courses/<id>/` | Course detail | none |
| POST | `/api/courses/<id>/enroll/` | Enroll the current user | Basic auth |
| GET | `/api/courses/<id>/contents/` | Course with all module contents | Basic auth, enrolled students only |

Example:

```bash
curl -X POST -u student:password http://127.0.0.1:8000/api/courses/1/enroll/
```

## Tech stack

| Area | Technology |
|------|------------|
| Backend | Python, Django 4.1 (class-based views, formsets, generic relations) |
| API | Django REST Framework |
| Cache | Redis (`django.core.cache.backends.redis.RedisCache`) |
| Database | SQLite |
| Libraries | django-braces, django-embed-video, django-debug-toolbar, django-redisboard |
| Frontend | Django templates, html5sortable |

## Project structure

```
API/
├── educa/                # Project settings and root URLs
├── courses/
│   ├── api/              # Serializers, views, permissions and router for the REST API
│   ├── fixtures/         # subjects.json sample data
│   ├── templates/        # Course management, catalog and content templates
│   ├── templatetags/     # Template filter for content model names
│   ├── fields.py         # Custom OrderField
│   ├── models.py         # Subject, Course, Module, Content, Text/File/Image/Video
│   └── views.py          # Instructor CMS and public catalog views
├── students/             # Registration, enrollment and student course views
├── manage.py
└── requirements.txt
```

## Getting started

Requirements: Python 3.10+ and a Redis server on `127.0.0.1:6379` (used as the cache backend), for example `docker run -p 6379:6379 redis`.

```bash
git clone https://github.com/Umar37-max/API.git
cd API

python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate

pip install -r requirements.txt

export DJANGO_SECRET_KEY="<your-secret-key>"   # see .env.example

python manage.py migrate
python manage.py loaddata subjects.json
python manage.py createsuperuser
python manage.py runserver
```

Open http://127.0.0.1:8000/. To let a user manage courses, add them in the admin (`/admin/`) to a group that has the `courses` add/change/delete permissions.
