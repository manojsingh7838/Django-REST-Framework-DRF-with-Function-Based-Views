
## Django REST Framework (DRF) with Function-Based Views

### Prerequisites
- Python (>=3.6)
- pip (Python package installer)
- Basic knowledge of Django

### Step 1: Setting Up the Environment

#### 1.1. Install Python and pip
Ensure Python and pip are installed on your system. You can download Python from the official [Python website](https://www.python.org/downloads/).

#### 1.2. Create a Virtual Environment
It's good practice to create a virtual environment for your project to manage dependencies.

```bash
python -m venv myenv
source myenv/bin/activate  # On Windows use `myenv\Scripts\activate`
```

#### 1.3. Install Django and Django REST Framework

```bash
pip install django djangorestframework
```

### Step 2: Creating a Django Project

#### 2.1. Create a New Project

```bash
django-admin startproject myproject
cd myproject
```

#### 2.2. Start the Development Server

```bash
python manage.py runserver
```

You should see a "Congratulations!" page at `http://127.0.0.1:8000/`.

### Step 3: Creating a Django App

#### 3.1. Create a New App

```bash
python manage.py startapp myapp
```

#### 3.2. Add the App to `INSTALLED_APPS`
Edit `myproject/settings.py`:

```python
INSTALLED_APPS = [
    ...
    'myapp',
    'rest_framework',
]
```

### Step 4: Creating Models

#### 4.1. Define a Model
Edit `myapp/models.py`:

```python
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.name
```

#### 4.2. Create and Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### Step 5: Creating Serializers

#### 5.1. Define a Serializer
Create a file `myapp/serializers.py`:

```python
from rest_framework import serializers
from .models import Item

class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = '__all__'
```

### Step 6: Creating Function-Based Views

#### 6.1. Define Views
Edit `myapp/views.py`:

```python
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated
from .models import Item
from .serializers import ItemSerializer

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def item_list_create(request):
    if request.method == 'GET':
        items = Item.objects.all()
        serializer = ItemSerializer(items, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = ItemSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
@permission_classes([IsAuthenticated])
def item_detail(request, pk):
    try:
        item = Item.objects.get(pk=pk)
    except Item.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = ItemSerializer(item)
        return Response(serializer.data)
    elif request.method == 'PUT':
        serializer = ItemSerializer(item, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        item.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Step 7: Configuring URLs

#### 7.1. Define App URLs
Create a file `myapp/urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('items/', views.item_list_create, name='item-list-create'),
    path('items/<int:pk>/', views.item_detail, name='item-detail'),
]
```

#### 7.2. Include App URLs in Project URLs
Edit `myproject/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('myapp.urls')),
]
```

### Step 8: Testing the API

#### 8.1. Run the Server

```bash
python manage.py runserver
```

#### 8.2. Test Endpoints
- List/Create Items: `http://127.0.0.1:8000/api/items/`
- Retrieve/Update/Delete Item: `http://127.0.0.1:8000/api/items/<id>/`

You can use tools like Postman or cURL to interact with these endpoints.

### Step 9: Adding Authentication

#### 9.1. Install Django REST Framework Auth

```bash
pip install djangorestframework-simplejwt
```

#### 9.2. Update Settings
Edit `myproject/settings.py`:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}
```

#### 9.3. Define Auth URLs
Edit `myproject/urls.py`:

```python
from django.urls import path, include
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    ...
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

#### 9.4. Protect Your Views
Edit `myapp/views.py` to include permission classes:

```python
from rest_framework.permissions import IsAuthenticated

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def item_list_create(request):
    ...

@api_view(['GET', 'PUT', 'DELETE'])
@permission_classes([IsAuthenticated])
def item_detail(request, pk):
    ...
```

#### 9.5. Obtain and Use Tokens
- Obtain Token: `POST http://127.0.0.1:8000/api/token/` with body `{"username": "yourusername", "password": "yourpassword"}`
- Refresh Token: `POST http://127.0.0.1:8000/api/token/refresh/` with body `{"refresh": "yourrefreshtoken"}`

Use the obtained access token in the Authorization header of your requests: `Authorization: Bearer youraccesstoken`.

### Step 10: Adding Pagination

#### 10.1. Update Settings for Pagination
Edit `myproject/settings.py`:

```python
REST_FRAMEWORK = {
    ...
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

### Step 11: Adding Filtering and Search

#### 11.1. Install Django Filter

```bash
pip install django-filter
```

#### 11.2. Update Settings
Edit `myproject/settings.py`:

```python
REST_FRAMEWORK = {
    ...
    'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend'],
}
```

#### 11.3. Define Filters
Edit `myapp/views.py`:

```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def item_list_create(request):
    if request.method == 'GET':
        items = Item.objects.all()
        filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
        filterset_fields = ['name']
        search_fields = ['name', 'description']
        ordering_fields = ['created_at']
        # Apply filters, search, and ordering
        for backend in filter_backends:
            items = backend().filter_queryset(request, items, view=None)
        serializer = ItemSerializer(items, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = ItemSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
@permission_classes([IsAuthenticated])
def item_detail(request, pk):
    ...
```

### Conclusion
This guide covers setting up a basic Django REST Framework application with function-based views (FBVs) and essential features like CRUD operations, authentication, pagination, filtering, and search. From here, you can explore more advanced topics like permissions, viewsets, routers, and integrating with frontend frameworks.

Feel free to expand this project by adding more models, serializers, views, and tests to match your application's requirements. Happy coding!
