# Day 3: Your Own Django App 🐍

> **Duration:** ~4 hours (hands-on pair coding session)  
> **Objective:** Build a complete Task Manager application from scratch  
> **Prerequisites:** Basic Python knowledge, command line familiarity

---

## 📁 Repository Structure

This repository contains pre-built templates and static files. You will create the Django project structure by running commands during the session.

```
django_workshop/
├── README.md                   # This file (instructions)
├── templates/                  # Pre-built HTML templates
│   └── tasks/
│       ├── base.html
│       ├── home.html
│       ├── task_list.html
│       ├── task_detail.html
│       ├── task_form.html
│       └── task_confirm_delete.html
└── static/                     # Pre-built static files
    └── tasks/
        ├── css/
        │   └── styles.css
        └── js/
            └── main.js
```

**After completing the Django setup, you will copy these to:**
```
django_workshop/
├── tasks/                      # Your Django app (created during session)
│   ├── templates/tasks/        # Copy templates here
│   └── static/tasks/           # Copy static files here
└── ...
```

---

## Table of Contents

1. [Environment Setup](#1-environment-setup)
2. [Django Project Initialization](#2-django-project-initialization)
3. [Models, Migrations & Admin](#3-models-migrations--admin)
4. [Views & URL Routing](#4-views--url-routing)
5. [Templates & Static Files](#5-templates--static-files)
6. [Local Development & Testing](#6-local-development--testing)
7. [📝 Take-Home Assignment: PostgreSQL with Docker](#7-take-home-assignment-postgresql-with-docker)

---

## 1. Environment Setup

### 1.1 Clone This Repository

```bash
# Clone the repository
cd ~
git clone <repository-url> django_workshop
cd django_workshop
```

### 1.2 Create a Conda Environment

```bash
# Create a new conda environment with Python 3.12
conda create -n django_intern python=3.12 -y

# Activate the environment
conda activate django_intern

# Verify Python version
python --version
```

**Expected output:**
```
Python 3.12.x
```

### 1.3 Install Django and Dependencies

```bash
# Install Django and related packages
pip install django==5.1.4

# Verify Django installation
django-admin --version
```

**Expected output:**
```
5.1.4
```

---

## 2. Django Project Initialization

### 2.1 Create a New Django Project

```bash
# Create the Django project (note the . at the end - creates in current directory)
django-admin startproject taskmanager .

# View the project structure
ls -la
```

**Project Structure (after this step):**
```
django_workshop/
├── README.md
├── manage.py               # Command-line utility for Django
├── templates/              # Pre-built templates (to be moved later)
├── static/                 # Pre-built static files (to be moved later)
└── taskmanager/            # Project configuration directory
    ├── __init__.py         # Python package marker
    ├── asgi.py             # ASGI config (async servers)
    ├── settings.py         # Project settings
    ├── urls.py             # URL declarations (root)
    └── wsgi.py             # WSGI config (traditional servers)
```

### 2.2 Understanding the Files

| File | Purpose |
|------|---------|
| `manage.py` | Entry point for running Django commands |
| `settings.py` | Database config, installed apps, middleware, etc. |
| `urls.py` | Maps URLs to views (routing) |
| `wsgi.py` | Web Server Gateway Interface for deployment |
| `asgi.py` | Async Server Gateway Interface for async features |

### 2.3 Create Your First App

Django projects contain multiple **apps**. Each app handles a specific feature.

```bash
# Create the tasks app
python manage.py startapp tasks
```

**Updated Project Structure:**
```
django_workshop/
├── README.md
├── manage.py
├── templates/              # Pre-built (to be moved)
├── static/                 # Pre-built (to be moved)
├── taskmanager/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── tasks/                  # Our new app!
    ├── __init__.py
    ├── admin.py            # Admin interface config
    ├── apps.py             # App configuration
    ├── migrations/         # Database migrations
    │   └── __init__.py
    ├── models.py           # Database models
    ├── tests.py            # Unit tests
    └── views.py            # Request handlers
```

### 2.4 Register the App

Open `taskmanager/settings.py` and add the app to `INSTALLED_APPS`:

```python
# taskmanager/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Our apps
    'tasks',  # Add this line
]
```

### 2.5 Run the Development Server

```bash
# Run the server
python manage.py runserver
```

**Expected output:**
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

🎉 **Open your browser and visit:** http://127.0.0.1:8000/

You should see the Django welcome page!

---

## 3. Models, Migrations & Admin

### 3.1 Define the Task Model

Django uses an **ORM (Object-Relational Mapping)** to interact with the database using Python classes.

Edit `tasks/models.py`:

```python
# tasks/models.py

from django.db import models
from django.utils import timezone


class Category(models.Model):
    """Category for organizing tasks."""
    
    name = models.CharField(max_length=100, unique=True)
    color = models.CharField(max_length=7, default='#3498db')  # Hex color
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name_plural = 'Categories'
        ordering = ['name']
    
    def __str__(self):
        return self.name


class Task(models.Model):
    """A task item in the task manager."""
    
    # Status choices
    class Status(models.TextChoices):
        TODO = 'TODO', 'To Do'
        IN_PROGRESS = 'IN_PROGRESS', 'In Progress'
        DONE = 'DONE', 'Done'
    
    # Priority choices
    class Priority(models.IntegerChoices):
        LOW = 1, 'Low'
        MEDIUM = 2, 'Medium'
        HIGH = 3, 'High'
    
    # Fields
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True, null=True)
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.TODO
    )
    priority = models.IntegerField(
        choices=Priority.choices,
        default=Priority.MEDIUM
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='tasks'
    )
    due_date = models.DateField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-priority', 'due_date', '-created_at']
    
    def __str__(self):
        return f'{self.title} ({self.get_status_display()})'
    
    @property
    def is_overdue(self):
        """Check if the task is overdue."""
        if self.due_date and self.status != self.Status.DONE:
            return self.due_date < timezone.now().date()
        return False
```

### 3.2 Create and Apply Migrations

Migrations are Django's way of propagating model changes to the database.

```bash
# Create migration files
python manage.py makemigrations tasks

# View the SQL that will be executed
python manage.py sqlmigrate tasks 0001

# Apply all migrations
python manage.py migrate
```

**Expected output for makemigrations:**
```
Migrations for 'tasks':
  tasks/migrations/0001_initial.py
    - Create model Category
    - Create model Task
```

### 3.3 Configure the Admin Interface

Django provides a powerful admin interface out of the box.

Edit `tasks/admin.py`:

```python
# tasks/admin.py

from django.contrib import admin
from .models import Category, Task


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'color', 'task_count', 'created_at']
    search_fields = ['name']
    
    def task_count(self, obj):
        return obj.tasks.count()
    task_count.short_description = 'Tasks'


@admin.register(Task)
class TaskAdmin(admin.ModelAdmin):
    list_display = ['title', 'status', 'priority', 'category', 'due_date', 'is_overdue']
    list_filter = ['status', 'priority', 'category', 'created_at']
    search_fields = ['title', 'description']
    list_editable = ['status', 'priority']
    date_hierarchy = 'created_at'
    
    fieldsets = (
        ('Task Information', {
            'fields': ('title', 'description', 'category')
        }),
        ('Status & Priority', {
            'fields': ('status', 'priority', 'due_date')
        }),
    )
    
    def is_overdue(self, obj):
        return obj.is_overdue
    is_overdue.boolean = True
    is_overdue.short_description = 'Overdue?'
```

### 3.4 Create a Superuser

```bash
# Create an admin user
python manage.py createsuperuser
```

Enter the following when prompted:
- **Username:** admin
- **Email:** admin@example.com
- **Password:** adminpass123 (or any password you prefer)

### 3.5 Access the Admin Interface

```bash
# Start the server if not running
python manage.py runserver
```

🔗 **Visit:** http://127.0.0.1:8000/admin/

Log in with your superuser credentials and explore the admin interface!

**Try these:**
1. Create 2-3 Categories (e.g., "Work", "Personal", "Learning")
2. Create 5-6 Tasks with different statuses and priorities
3. Explore the filtering and search features

---

## 4. Views & URL Routing

### 4.1 Create Views

We'll create both **function-based views (FBVs)** and **class-based views (CBVs)**.

Edit `tasks/views.py`:

```python
# tasks/views.py

from django.shortcuts import render, get_object_or_404, redirect
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from django.contrib import messages
from django.db.models import Count, Q
from django.http import JsonResponse

from .models import Task, Category


# ============================================
# Function-Based Views (FBVs)
# ============================================

def home(request):
    """Home page with dashboard statistics."""
    # Get statistics
    total_tasks = Task.objects.count()
    todo_count = Task.objects.filter(status=Task.Status.TODO).count()
    in_progress_count = Task.objects.filter(status=Task.Status.IN_PROGRESS).count()
    done_count = Task.objects.filter(status=Task.Status.DONE).count()
    
    # Get recent tasks
    recent_tasks = Task.objects.all()[:5]
    
    # Get categories with task counts
    categories = Category.objects.annotate(task_count=Count('tasks'))
    
    context = {
        'total_tasks': total_tasks,
        'todo_count': todo_count,
        'in_progress_count': in_progress_count,
        'done_count': done_count,
        'recent_tasks': recent_tasks,
        'categories': categories,
    }
    return render(request, 'tasks/home.html', context)


def task_list(request):
    """List all tasks with filtering."""
    tasks = Task.objects.select_related('category').all()
    
    # Filter by status
    status = request.GET.get('status')
    if status:
        tasks = tasks.filter(status=status)
    
    # Filter by category
    category_id = request.GET.get('category')
    if category_id:
        tasks = tasks.filter(category_id=category_id)
    
    # Filter by priority
    priority = request.GET.get('priority')
    if priority:
        tasks = tasks.filter(priority=priority)
    
    # Search
    search = request.GET.get('search')
    if search:
        tasks = tasks.filter(
            Q(title__icontains=search) | Q(description__icontains=search)
        )
    
    categories = Category.objects.all()
    
    context = {
        'tasks': tasks,
        'categories': categories,
        'current_status': status,
        'current_category': category_id,
        'current_priority': priority,
        'search_query': search or '',
    }
    return render(request, 'tasks/task_list.html', context)


def task_detail(request, pk):
    """View a single task."""
    task = get_object_or_404(Task, pk=pk)
    return render(request, 'tasks/task_detail.html', {'task': task})


def task_create(request):
    """Create a new task."""
    if request.method == 'POST':
        # Get form data
        title = request.POST.get('title')
        description = request.POST.get('description')
        status = request.POST.get('status', Task.Status.TODO)
        priority = request.POST.get('priority', Task.Priority.MEDIUM)
        category_id = request.POST.get('category')
        due_date = request.POST.get('due_date') or None
        
        # Create the task
        task = Task.objects.create(
            title=title,
            description=description,
            status=status,
            priority=priority,
            category_id=category_id if category_id else None,
            due_date=due_date,
        )
        
        messages.success(request, f'Task "{task.title}" created successfully!')
        return redirect('tasks:task_detail', pk=task.pk)
    
    # GET request - show form
    categories = Category.objects.all()
    context = {
        'categories': categories,
        'status_choices': Task.Status.choices,
        'priority_choices': Task.Priority.choices,
    }
    return render(request, 'tasks/task_form.html', context)


def task_update(request, pk):
    """Update an existing task."""
    task = get_object_or_404(Task, pk=pk)
    
    if request.method == 'POST':
        task.title = request.POST.get('title')
        task.description = request.POST.get('description')
        task.status = request.POST.get('status')
        task.priority = request.POST.get('priority')
        category_id = request.POST.get('category')
        task.category_id = category_id if category_id else None
        task.due_date = request.POST.get('due_date') or None
        task.save()
        
        messages.success(request, f'Task "{task.title}" updated successfully!')
        return redirect('tasks:task_detail', pk=task.pk)
    
    categories = Category.objects.all()
    context = {
        'task': task,
        'categories': categories,
        'status_choices': Task.Status.choices,
        'priority_choices': Task.Priority.choices,
        'is_update': True,
    }
    return render(request, 'tasks/task_form.html', context)


def task_delete(request, pk):
    """Delete a task."""
    task = get_object_or_404(Task, pk=pk)
    
    if request.method == 'POST':
        title = task.title
        task.delete()
        messages.success(request, f'Task "{title}" deleted successfully!')
        return redirect('tasks:task_list')
    
    return render(request, 'tasks/task_confirm_delete.html', {'task': task})


def toggle_task_status(request, pk):
    """Toggle task status via AJAX."""
    if request.method == 'POST':
        task = get_object_or_404(Task, pk=pk)
        
        # Cycle through statuses
        status_order = [Task.Status.TODO, Task.Status.IN_PROGRESS, Task.Status.DONE]
        current_index = status_order.index(task.status)
        next_index = (current_index + 1) % len(status_order)
        task.status = status_order[next_index]
        task.save()
        
        return JsonResponse({
            'status': task.status,
            'status_display': task.get_status_display(),
        })
    
    return JsonResponse({'error': 'Invalid request'}, status=400)


# ============================================
# Class-Based Views (CBVs) - Alternative
# ============================================

class TaskListView(ListView):
    """Class-based view for listing tasks."""
    model = Task
    template_name = 'tasks/task_list_cbv.html'
    context_object_name = 'tasks'
    paginate_by = 10


class TaskDetailView(DetailView):
    """Class-based view for task detail."""
    model = Task
    template_name = 'tasks/task_detail.html'
    context_object_name = 'task'
```

### 4.2 Create URL Patterns

Create a new file `tasks/urls.py`:

```python
# tasks/urls.py

from django.urls import path
from . import views

app_name = 'tasks'  # Namespace for URL names

urlpatterns = [
    # Home/Dashboard
    path('', views.home, name='home'),
    
    # Task CRUD - Function-Based Views
    path('tasks/', views.task_list, name='task_list'),
    path('tasks/create/', views.task_create, name='task_create'),
    path('tasks/<int:pk>/', views.task_detail, name='task_detail'),
    path('tasks/<int:pk>/update/', views.task_update, name='task_update'),
    path('tasks/<int:pk>/delete/', views.task_delete, name='task_delete'),
    
    # API endpoint
    path('tasks/<int:pk>/toggle/', views.toggle_task_status, name='task_toggle'),
    
    # Class-Based Views (alternative)
    path('tasks-cbv/', views.TaskListView.as_view(), name='task_list_cbv'),
]
```

### 4.3 Include App URLs in Project

Edit `taskmanager/urls.py`:

```python
# taskmanager/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tasks.urls')),  # Include tasks app URLs
]
```

---

## 5. Templates & Static Files

### 5.1 Copy Pre-built Templates and Static Files

Now it's time to use the pre-built templates and static files from this repository!

```bash
# Create the directories inside the tasks app
mkdir -p tasks/templates
mkdir -p tasks/static

# Copy the pre-built templates to the tasks app
cp -r templates/tasks tasks/templates/

# Copy the pre-built static files to the tasks app
cp -r static/tasks tasks/static/

# Verify the structure
ls -la tasks/templates/tasks/
ls -la tasks/static/tasks/
```

**Your tasks app should now look like:**
```
tasks/
├── __init__.py
├── admin.py
├── apps.py
├── migrations/
├── models.py
├── tests.py
├── urls.py
├── views.py
├── templates/
│   └── tasks/
│       ├── base.html
│       ├── home.html
│       ├── task_list.html
│       ├── task_detail.html
│       ├── task_form.html
│       └── task_confirm_delete.html
└── static/
    └── tasks/
        ├── css/
        │   └── styles.css
        └── js/
            └── main.js
```

### 5.2 Understanding the Templates

| Template | Purpose |
|----------|---------|
| `base.html` | Base layout with navigation, footer, and CSS/JS includes |
| `home.html` | Dashboard with statistics and recent tasks |
| `task_list.html` | List all tasks with filters and search |
| `task_detail.html` | View a single task's details |
| `task_form.html` | Create/edit task form |
| `task_confirm_delete.html` | Confirm task deletion |

### 5.3 Template Syntax Quick Reference

```html
<!-- Variables -->
{{ variable }}
{{ task.title }}

<!-- Tags -->
{% if condition %}...{% endif %}
{% for item in list %}...{% endfor %}
{% url 'tasks:home' %}
{% static 'tasks/css/styles.css' %}

<!-- Filters -->
{{ text|truncatewords:20 }}
{{ date|date:"F d, Y" }}
{{ task.status|lower }}

<!-- Template Inheritance -->
{% extends 'tasks/base.html' %}
{% block content %}...{% endblock %}
```

---

## 6. Local Development & Testing

### 6.1 Run the Server

```bash
# Make sure you're in the project directory
cd ~/django_workshop

# Activate the conda environment
conda activate django_intern

# Run migrations if not done
python manage.py migrate

# Start the development server
python manage.py runserver
```

### 6.2 Test the Application

Open your browser and test all the features:

1. **Dashboard:** http://127.0.0.1:8000/
2. **Task List:** http://127.0.0.1:8000/tasks/
3. **Create Task:** http://127.0.0.1:8000/tasks/create/
4. **Admin Panel:** http://127.0.0.1:8000/admin/

### 6.3 Django Shell

The Django shell is great for debugging and testing:

```bash
# Open Django shell
python manage.py shell
```

```python
# In the shell:
from tasks.models import Task, Category

# Create a category
cat = Category.objects.create(name="Urgent", color="#ff0000")

# Create a task
task = Task.objects.create(
    title="Test Task",
    description="This is a test",
    priority=3,
    category=cat
)

# Query tasks
Task.objects.all()
Task.objects.filter(status='TODO')
Task.objects.filter(priority__gte=2)

# Exit shell
exit()
```

### 6.4 Writing Unit Tests

Edit `tasks/tests.py`:

```python
# tasks/tests.py

from django.test import TestCase, Client
from django.urls import reverse
from .models import Task, Category


class CategoryModelTest(TestCase):
    """Tests for the Category model."""
    
    def test_create_category(self):
        """Test creating a category."""
        category = Category.objects.create(
            name="Work",
            color="#3498db"
        )
        self.assertEqual(str(category), "Work")
        self.assertEqual(category.color, "#3498db")


class TaskModelTest(TestCase):
    """Tests for the Task model."""
    
    def setUp(self):
        """Set up test data."""
        self.category = Category.objects.create(name="Test Category")
        self.task = Task.objects.create(
            title="Test Task",
            description="Test Description",
            priority=Task.Priority.HIGH,
            category=self.category
        )
    
    def test_task_creation(self):
        """Test task is created correctly."""
        self.assertEqual(self.task.title, "Test Task")
        self.assertEqual(self.task.status, Task.Status.TODO)
        self.assertEqual(self.task.priority, Task.Priority.HIGH)
    
    def test_task_str_representation(self):
        """Test task string representation."""
        self.assertIn("Test Task", str(self.task))
    
    def test_task_ordering(self):
        """Test tasks are ordered by priority."""
        low_task = Task.objects.create(
            title="Low Priority",
            priority=Task.Priority.LOW
        )
        tasks = Task.objects.all()
        # High priority should come first
        self.assertEqual(tasks[0], self.task)


class TaskViewsTest(TestCase):
    """Tests for task views."""
    
    def setUp(self):
        """Set up test client and data."""
        self.client = Client()
        self.task = Task.objects.create(
            title="Test Task",
            description="Test Description"
        )
    
    def test_home_view(self):
        """Test home page loads correctly."""
        response = self.client.get(reverse('tasks:home'))
        self.assertEqual(response.status_code, 200)
        self.assertTemplateUsed(response, 'tasks/home.html')
    
    def test_task_list_view(self):
        """Test task list page loads correctly."""
        response = self.client.get(reverse('tasks:task_list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Task")
    
    def test_task_detail_view(self):
        """Test task detail page loads correctly."""
        response = self.client.get(
            reverse('tasks:task_detail', args=[self.task.pk])
        )
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Task")
    
    def test_task_create_view(self):
        """Test creating a new task."""
        response = self.client.post(
            reverse('tasks:task_create'),
            {
                'title': 'New Task',
                'description': 'New Description',
                'status': 'TODO',
                'priority': 2,
            }
        )
        self.assertEqual(Task.objects.count(), 2)
        new_task = Task.objects.get(title='New Task')
        self.assertRedirects(
            response,
            reverse('tasks:task_detail', args=[new_task.pk])
        )
    
    def test_task_update_view(self):
        """Test updating a task."""
        response = self.client.post(
            reverse('tasks:task_update', args=[self.task.pk]),
            {
                'title': 'Updated Task',
                'description': 'Updated Description',
                'status': 'IN_PROGRESS',
                'priority': 3,
            }
        )
        self.task.refresh_from_db()
        self.assertEqual(self.task.title, 'Updated Task')
        self.assertEqual(self.task.status, Task.Status.IN_PROGRESS)
    
    def test_task_delete_view(self):
        """Test deleting a task."""
        response = self.client.post(
            reverse('tasks:task_delete', args=[self.task.pk])
        )
        self.assertEqual(Task.objects.count(), 0)
        self.assertRedirects(response, reverse('tasks:task_list'))
```

### 6.5 Run Tests

```bash
# Run all tests
python manage.py test

# Run tests with verbose output
python manage.py test -v 2

# Run specific test class
python manage.py test tasks.tests.TaskModelTest

# Run specific test
python manage.py test tasks.tests.TaskViewsTest.test_home_view
```

**Expected output:**
```
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
..........
----------------------------------------------------------------------
Ran 10 tests in 0.XXXs

OK
Destroying test database for alias 'default'...
```

### 6.6 Common Debugging Tips

```bash
# Check for issues
python manage.py check

# View SQL for a migration
python manage.py sqlmigrate tasks 0001

# Show migration status
python manage.py showmigrations

# Reset database (development only!)
rm db.sqlite3
python manage.py migrate
python manage.py createsuperuser
```

---

## 7. Take-Home Assignment: PostgreSQL with Docker 🏠

### Objective

Convert your Django application from SQLite to PostgreSQL using Docker, ensuring data persistence and implementing a simple REST API.

### Prerequisites

- Docker Desktop installed ([Download here](https://www.docker.com/products/docker-desktop/))
- Basic understanding of Docker concepts

### Part 1: Set Up PostgreSQL with Docker

#### 1.1 Create a Docker Compose File

Create `docker-compose.yml` in your project root (`~/django_workshop/`):

```yaml
# docker-compose.yml
version: '3.9'

services:
  db:
    image: postgres:16-alpine
    container_name: taskflow_postgres
    environment:
      POSTGRES_DB: taskflow_db
      POSTGRES_USER: taskflow_user
      POSTGRES_PASSWORD: taskflow_secret_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U taskflow_user -d taskflow_db"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    name: taskflow_postgres_data
```

#### 1.2 Start the PostgreSQL Container

```bash
# Start the database
docker compose up -d

# Verify it's running
docker compose ps

# View logs
docker compose logs db
```

### Part 2: Configure Django for PostgreSQL

#### 2.1 Install PostgreSQL Adapter

```bash
# Install psycopg2 (PostgreSQL adapter)
pip install psycopg2-binary
```

#### 2.2 Update Django Settings

Edit `taskmanager/settings.py`:

```python
# taskmanager/settings.py

# Replace the DATABASES configuration with:
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'taskflow_db',
        'USER': 'taskflow_user',
        'PASSWORD': 'taskflow_secret_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

#### 2.3 Apply Migrations to PostgreSQL

```bash
# Apply migrations to the new database
python manage.py migrate

# Create a new superuser
python manage.py createsuperuser
```

### Part 3: Create a REST API

#### 3.1 Install Django REST Framework

```bash
pip install djangorestframework
```

#### 3.2 Update Settings

Add to `taskmanager/settings.py`:

```python
INSTALLED_APPS = [
    # ... existing apps ...
    'rest_framework',
]

# REST Framework settings
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

#### 3.3 Create Serializers

Create `tasks/serializers.py`:

```python
# tasks/serializers.py

from rest_framework import serializers
from .models import Task, Category


class CategorySerializer(serializers.ModelSerializer):
    task_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Category
        fields = ['id', 'name', 'color', 'task_count', 'created_at']
    
    def get_task_count(self, obj):
        return obj.tasks.count()


class TaskSerializer(serializers.ModelSerializer):
    category_name = serializers.CharField(source='category.name', read_only=True)
    status_display = serializers.CharField(source='get_status_display', read_only=True)
    priority_display = serializers.CharField(source='get_priority_display', read_only=True)
    is_overdue = serializers.BooleanField(read_only=True)
    
    class Meta:
        model = Task
        fields = [
            'id', 'title', 'description', 'status', 'status_display',
            'priority', 'priority_display', 'category', 'category_name',
            'due_date', 'is_overdue', 'created_at', 'updated_at'
        ]


class TaskCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ['title', 'description', 'status', 'priority', 'category', 'due_date']
```

#### 3.4 Create API Views

Create `tasks/api_views.py`:

```python
# tasks/api_views.py

from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django.db.models import Count

from .models import Task, Category
from .serializers import TaskSerializer, TaskCreateSerializer, CategorySerializer


class CategoryViewSet(viewsets.ModelViewSet):
    """API endpoint for categories."""
    queryset = Category.objects.annotate(task_count=Count('tasks'))
    serializer_class = CategorySerializer


class TaskViewSet(viewsets.ModelViewSet):
    """API endpoint for tasks."""
    queryset = Task.objects.select_related('category').all()
    
    def get_serializer_class(self):
        if self.action in ['create', 'update', 'partial_update']:
            return TaskCreateSerializer
        return TaskSerializer
    
    def get_queryset(self):
        queryset = super().get_queryset()
        
        # Filter by status
        status_param = self.request.query_params.get('status')
        if status_param:
            queryset = queryset.filter(status=status_param)
        
        # Filter by priority
        priority = self.request.query_params.get('priority')
        if priority:
            queryset = queryset.filter(priority=priority)
        
        # Filter by category
        category = self.request.query_params.get('category')
        if category:
            queryset = queryset.filter(category_id=category)
        
        return queryset
    
    @action(detail=True, methods=['post'])
    def toggle_status(self, request, pk=None):
        """Toggle the task status."""
        task = self.get_object()
        
        status_order = [Task.Status.TODO, Task.Status.IN_PROGRESS, Task.Status.DONE]
        current_index = status_order.index(task.status)
        next_index = (current_index + 1) % len(status_order)
        task.status = status_order[next_index]
        task.save()
        
        return Response({
            'id': task.id,
            'status': task.status,
            'status_display': task.get_status_display()
        })
    
    @action(detail=False, methods=['get'])
    def statistics(self, request):
        """Get task statistics."""
        total = Task.objects.count()
        by_status = Task.objects.values('status').annotate(count=Count('id'))
        by_priority = Task.objects.values('priority').annotate(count=Count('id'))
        
        return Response({
            'total': total,
            'by_status': {item['status']: item['count'] for item in by_status},
            'by_priority': {item['priority']: item['count'] for item in by_priority},
        })
```

#### 3.5 Create API URLs

Create `tasks/api_urls.py`:

```python
# tasks/api_urls.py

from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .api_views import TaskViewSet, CategoryViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)
router.register(r'categories', CategoryViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

#### 3.6 Update Main URLs

Edit `taskmanager/urls.py`:

```python
# taskmanager/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('tasks.urls')),
    path('api/', include('tasks.api_urls')),  # Add API URLs
]
```

### Part 4: Test the API

#### 4.1 Start the Server

```bash
python manage.py runserver
```

#### 4.2 Test with curl (or use Postman/Insomnia)

**List all tasks:**
```bash
curl http://127.0.0.1:8000/api/tasks/
```

**Create a category:**
```bash
curl -X POST http://127.0.0.1:8000/api/categories/ \
  -H "Content-Type: application/json" \
  -d '{"name": "Work", "color": "#3498db"}'
```

**Create a task:**
```bash
curl -X POST http://127.0.0.1:8000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Complete Django Tutorial",
    "description": "Finish the take-home assignment",
    "priority": 3,
    "status": "TODO"
  }'
```

**Get task statistics:**
```bash
curl http://127.0.0.1:8000/api/tasks/statistics/
```

**Toggle task status:**
```bash
curl -X POST http://127.0.0.1:8000/api/tasks/1/toggle_status/
```

**Update a task:**
```bash
curl -X PATCH http://127.0.0.1:8000/api/tasks/1/ \
  -H "Content-Type: application/json" \
  -d '{"status": "DONE"}'
```

**Delete a task:**
```bash
curl -X DELETE http://127.0.0.1:8000/api/tasks/1/
```

### Part 5: Verify Data Persistence

```bash
# Stop the containers
docker compose down

# Start them again
docker compose up -d

# Check your data is still there
curl http://127.0.0.1:8000/api/tasks/
```

Your data should persist because we're using a Docker volume (`postgres_data`).

### Bonus Challenges 🌟

1. **Add authentication:** Implement token-based authentication for the API
2. **Add filtering:** Use `django-filter` for advanced filtering in the API
3. **Add Swagger docs:** Use `drf-spectacular` to generate API documentation
4. **Containerize Django:** Create a Dockerfile for the Django app and add it to docker-compose
5. **Add tests:** Write API tests using Django REST Framework's test client

### Deliverables Checklist

- [ ] PostgreSQL running in Docker with data persistence
- [ ] Django connected to PostgreSQL
- [ ] REST API endpoints working for CRUD operations
- [ ] Sample data created via API
- [ ] Data persists after container restart
- [ ] Screenshot of API responses (optional)

---

## Quick Reference Commands

```bash
# Environment
conda activate django_intern

# Django commands
python manage.py runserver              # Start server
python manage.py makemigrations         # Create migrations
python manage.py migrate                # Apply migrations
python manage.py createsuperuser        # Create admin user
python manage.py shell                  # Django shell
python manage.py test                   # Run tests

# Docker commands
docker compose up -d                    # Start containers
docker compose down                     # Stop containers
docker compose logs db                  # View logs
docker compose ps                       # List containers

# Copy templates (after creating tasks app)
cp -r templates/tasks tasks/templates/
cp -r static/tasks tasks/static/
```

---

## Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)

---

**🎉 Congratulations!** You've built a complete Django application with:
- ✅ Database models and migrations
- ✅ Admin interface
- ✅ Web views and templates
- ✅ URL routing
- ✅ Static files
- ✅ Unit tests
- ✅ REST API (take-home)
- ✅ PostgreSQL with Docker (take-home)

Happy coding! 🚀

