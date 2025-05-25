# Django REST Framework - Function-Based Views Complete Cheatsheet

## Table of Contents
1. [Basic Setup and Configuration](#basic-setup-and-configuration)
2. [Simple API View](#simple-api-view)
3. [HTTP Method Handling](#http-method-handling)
4. [Serializers with FBVs](#serializers-with-fbvs)
5. [Request and Response Objects](#request-and-response-objects)
6. [Status Codes](#status-codes)
7. [URL Parameters and Query Strings](#url-parameters-and-query-strings)
8. [CRUD Operations](#crud-operations)
9. [Error Handling](#error-handling)
10. [Authentication](#authentication)
11. [Permissions](#permissions)
12. [Pagination](#pagination)
13. [Filtering and Searching](#filtering-and-searching)
14. [File Uploads](#file-uploads)
15. [Custom Response Formats](#custom-response-formats)
16. [Throttling](#throttling)
17. [Versioning](#versioning)
18. [Content Negotiation](#content-negotiation)

---

## Basic Setup and Configuration

### Concept
Before creating FBVs, you need to properly configure Django and DRF in your project.

### Code Example
```python
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'myapp',  # Your app
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}

# models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13, unique=True)
    published_date = models.DateField()
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

### Explanation
- **INSTALLED_APPS**: Includes `rest_framework` and your app
- **REST_FRAMEWORK**: Global DRF configuration dictionary
- **DEFAULT_AUTHENTICATION_CLASSES**: Defines how users authenticate
- **DEFAULT_PERMISSION_CLASSES**: Sets default permissions for all views
- **DEFAULT_PAGINATION_CLASS**: Configures automatic pagination
- **Book model**: Sample model we'll use throughout examples

---

## Simple API View

### Concept
The most basic FBV that returns JSON data using DRF's Response class.

### Code Example
```python
# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(['GET'])
def hello_world(request):
    """
    Simple API view that returns a greeting message.
    """
    data = {
        'message': 'Hello, World!',
        'status': 'success',
        'timestamp': '2024-01-01T12:00:00Z'
    }
    return Response(data)

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello_world, name='hello-world'),
]
```

### Explanation
- **@api_view(['GET'])**: Decorator that converts a regular function into a DRF API view
- **['GET']**: List of HTTP methods this view accepts (GET, POST, PUT, DELETE, etc.)
- **request**: DRF Request object (enhanced version of Django's HttpRequest)
- **Response(data)**: DRF Response object that automatically handles JSON serialization
- **data**: Python dictionary that gets converted to JSON
- The decorator handles content negotiation, parsing, and authentication automatically

---

## HTTP Method Handling

### Concept
Handle different HTTP methods (GET, POST, PUT, DELETE) in a single FBV.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET', 'POST', 'PUT', 'DELETE'])
def book_operations(request):
    """
    Handle multiple HTTP methods in one view.
    """
    if request.method == 'GET':
        # Handle GET request
        data = {
            'message': 'This is a GET request',
            'books_count': 150
        }
        return Response(data)
    
    elif request.method == 'POST':
        # Handle POST request
        received_data = request.data
        response_data = {
            'message': 'Book created successfully',
            'received_data': received_data,
            'method': 'POST'
        }
        return Response(response_data, status=status.HTTP_201_CREATED)
    
    elif request.method == 'PUT':
        # Handle PUT request
        updated_data = request.data
        response_data = {
            'message': 'Book updated successfully',
            'updated_data': updated_data,
            'method': 'PUT'
        }
        return Response(response_data)
    
    elif request.method == 'DELETE':
        # Handle DELETE request
        response_data = {
            'message': 'Book deleted successfully',
            'method': 'DELETE'
        }
        return Response(response_data, status=status.HTTP_204_NO_CONTENT)
```

### Explanation
- **Multiple methods**: The decorator accepts a list of HTTP methods
- **request.method**: String indicating the HTTP method used
- **request.data**: Parsed request body data (works with JSON, form data, etc.)
- **status parameter**: Explicitly sets HTTP status code for the response
- **Conditional logic**: Use if/elif to handle different methods appropriately
- Each method can return different data and status codes

---

## Serializers with FBVs

### Concept
Serializers convert model instances to JSON and validate incoming data.

### Code Example
```python
# serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'published_date', 'pages', 'price']
        
    def validate_isbn(self, value):
        """Custom validation for ISBN field."""
        if len(value) != 13:
            raise serializers.ValidationError("ISBN must be 13 characters long.")
        return value

# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Book
from .serializers import BookSerializer

@api_view(['GET', 'POST'])
def book_list(request):
    """
    List all books or create a new book.
    """
    if request.method == 'GET':
        # Serialize queryset to JSON
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
    elif request.method == 'POST':
        # Deserialize JSON to create new book
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def book_detail(request, pk):
    """
    Retrieve, update or delete a book instance.
    """
    try:
        book = Book.objects.get(pk=pk)
    except Book.DoesNotExist:
        return Response({'error': 'Book not found'}, 
                       status=status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        serializer = BookSerializer(book)
        return Response(serializer.data)
    
    elif request.method == 'PUT':
        serializer = BookSerializer(book, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        book.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### Explanation
- **BookSerializer**: ModelSerializer automatically creates fields based on the model
- **fields**: List of model fields to include in serialization
- **validate_isbn()**: Custom validation method for specific fields
- **many=True**: Used when serializing multiple objects (querysets)
- **serializer.data**: Contains the serialized data (Python dict/list)
- **serializer.is_valid()**: Validates incoming data against serializer rules
- **serializer.save()**: Creates or updates model instance
- **serializer.errors**: Contains validation errors if data is invalid
- **try/except**: Handles cases where requested object doesn't exist

---

## Request and Response Objects

### Concept
DRF provides enhanced Request and Response objects with additional functionality.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['GET', 'POST'])
def request_info(request):
    """
    Demonstrate DRF Request and Response objects.
    """
    if request.method == 'GET':
        # Accessing request information
        request_info = {
            'method': request.method,
            'path': request.path,
            'query_params': dict(request.query_params),
            'headers': dict(request.headers),
            'user': str(request.user),
            'auth': str(request.auth),
            'content_type': request.content_type,
            'accepted_renderer': str(request.accepted_renderer),
            'accepted_media_type': request.accepted_media_type,
        }
        
        # Creating custom Response
        response = Response(
            data=request_info,
            status=status.HTTP_200_OK,
            headers={'Custom-Header': 'Custom-Value'}
        )
        return response
    
    elif request.method == 'POST':
        # Accessing POST data
        post_info = {
            'data': request.data,          # Parsed content
            'files': list(request.FILES.keys()),  # Uploaded files
            'form_data': dict(request.POST),       # Form data
            'json_data': request.data if request.content_type == 'application/json' else None,
        }
        
        return Response(
            data=post_info,
            status=status.HTTP_201_CREATED,
            headers={
                'Location': '/api/books/1/',
                'X-Custom-Header': 'Post-Response'
            }
        )

@api_view(['GET'])
def content_negotiation_demo(request):
    """
    Demonstrate content negotiation features.
    """
    data = {'message': 'This response adapts to requested format'}
    
    # Response will be formatted based on Accept header
    # application/json -> JSON response
    # application/xml -> XML response (if renderer available)
    # text/html -> Browsable API HTML
    
    return Response(data)
```

### Explanation
- **request.query_params**: Dictionary-like object for URL query parameters
- **request.headers**: Access to HTTP headers
- **request.user**: Current authenticated user
- **request.auth**: Authentication credentials
- **request.content_type**: MIME type of request content
- **request.data**: Parsed request body (handles JSON, form data, files)
- **request.FILES**: Uploaded files
- **Response headers**: Custom headers can be added to responses
- **Content negotiation**: DRF automatically formats response based on Accept header

---

## Status Codes

### Concept
DRF provides constants for HTTP status codes to make code more readable and maintainable.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from .models import Book
from .serializers import BookSerializer

@api_view(['GET', 'POST', 'PUT', 'DELETE'])
def status_code_examples(request, pk=None):
    """
    Demonstrate proper use of HTTP status codes.
    """
    
    if request.method == 'GET':
        if pk:
            # Get single book
            try:
                book = Book.objects.get(pk=pk)
                serializer = BookSerializer(book)
                return Response(serializer.data, status=status.HTTP_200_OK)
            except Book.DoesNotExist:
                return Response(
                    {'error': 'Book not found'}, 
                    status=status.HTTP_404_NOT_FOUND
                )
        else:
            # Get all books
            books = Book.objects.all()
            if books.exists():
                serializer = BookSerializer(books, many=True)
                return Response(serializer.data, status=status.HTTP_200_OK)
            else:
                return Response([], status=status.HTTP_200_OK)
    
    elif request.method == 'POST':
        # Create new book
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'PUT':
        # Update existing book
        if not pk:
            return Response(
                {'error': 'Book ID required for update'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        try:
            book = Book.objects.get(pk=pk)
        except Book.DoesNotExist:
            return Response(
                {'error': 'Book not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )
        
        serializer = BookSerializer(book, data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_200_OK)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        if not pk:
            return Response(
                {'error': 'Book ID required for deletion'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        try:
            book = Book.objects.get(pk=pk)
            book.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)
        except Book.DoesNotExist:
            return Response(
                {'error': 'Book not found'}, 
                status=status.HTTP_404_NOT_FOUND
            )

# Common status codes reference
@api_view(['GET'])
def status_codes_reference(request):
    """
    Reference for common HTTP status codes in DRF.
    """
    codes = {
        'success_codes': {
            'HTTP_200_OK': 'Standard success response',
            'HTTP_201_CREATED': 'Resource created successfully',
            'HTTP_204_NO_CONTENT': 'Success with no content (often for DELETE)',
        },
        'client_error_codes': {
            'HTTP_400_BAD_REQUEST': 'Invalid request data',
            'HTTP_401_UNAUTHORIZED': 'Authentication required',
            'HTTP_403_FORBIDDEN': 'Permission denied',
            'HTTP_404_NOT_FOUND': 'Resource not found',
            'HTTP_405_METHOD_NOT_ALLOWED': 'HTTP method not allowed',
            'HTTP_409_CONFLICT': 'Resource conflict',
            'HTTP_422_UNPROCESSABLE_ENTITY': 'Validation error',
        },
        'server_error_codes': {
            'HTTP_500_INTERNAL_SERVER_ERROR': 'Server error',
            'HTTP_503_SERVICE_UNAVAILABLE': 'Service temporarily unavailable',
        }
    }
    return Response(codes)
```

### Explanation
- **status.HTTP_200_OK**: Standard success response (200)
- **status.HTTP_201_CREATED**: Resource created successfully (201)
- **status.HTTP_204_NO_CONTENT**: Success with no content, often used for DELETE (204)
- **status.HTTP_400_BAD_REQUEST**: Invalid request data or validation errors (400)
- **status.HTTP_404_NOT_FOUND**: Requested resource doesn't exist (404)
- **partial=True**: Allows partial updates in PUT requests
- **Using constants**: Makes code more readable and prevents typos in status codes
- **Appropriate status codes**: Each operation returns semantically correct status

---

## URL Parameters and Query Strings

### Concept
Handle URL parameters (path parameters) and query string parameters in FBVs.

### Code Example
```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('books/', views.book_list, name='book-list'),
    path('books/<int:pk>/', views.book_detail, name='book-detail'),
    path('books/by-author/<str:author_name>/', views.books_by_author, name='books-by-author'),
    path('search/', views.book_search, name='book-search'),
]

# views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.db.models import Q
from .models import Book
from .serializers import BookSerializer

@api_view(['GET'])
def book_detail(request, pk):
    """
    Get book by ID (URL parameter).
    """
    try:
        book = Book.objects.get(pk=pk)
        serializer = BookSerializer(book)
        return Response(serializer.data)
    except Book.DoesNotExist:
        return Response(
            {'error': f'Book with ID {pk} not found'}, 
            status=status.HTTP_404_NOT_FOUND
        )

@api_view(['GET'])
def books_by_author(request, author_name):
    """
    Get books by author name (string URL parameter).
    """
    books = Book.objects.filter(author__icontains=author_name)
    
    if not books.exists():
        return Response(
            {'message': f'No books found by author: {author_name}'}, 
            status=status.HTTP_404_NOT_FOUND
        )
    
    serializer = BookSerializer(books, many=True)
    return Response({
        'author': author_name,
        'count': books.count(),
        'books': serializer.data
    })

@api_view(['GET'])
def book_search(request):
    """
    Search books using query parameters.
    URL: /search/?title=django&author=smith&min_price=10&max_price=50&ordering=title
    """
    # Get query parameters
    title = request.query_params.get('title', None)
    author = request.query_params.get('author', None)
    min_price = request.query_params.get('min_price', None)
    max_price = request.query_params.get('max_price', None)
    ordering = request.query_params.get('ordering', 'title')
    page_size = request.query_params.get('page_size', 10)
    
    # Build queryset based on parameters
    queryset = Book.objects.all()
    
    if title:
        queryset = queryset.filter(title__icontains=title)
    
    if author:
        queryset = queryset.filter(author__icontains=author)
    
    if min_price:
        try:
            min_price = float(min_price)
            queryset = queryset.filter(price__gte=min_price)
        except ValueError:
            return Response(
                {'error': 'Invalid min_price value'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    if max_price:
        try:
            max_price = float(max_price)
            queryset = queryset.filter(price__lte=max_price)
        except ValueError:
            return Response(
                {'error': 'Invalid max_price value'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    # Apply ordering
    valid_orderings = ['title', '-title', 'author', '-author', 'price', '-price', 'published_date', '-published_date']
    if ordering in valid_orderings:
        queryset = queryset.order_by(ordering)
    
    # Limit results
    try:
        page_size = int(page_size)
        if page_size > 100:  # Prevent excessive results
            page_size = 100
        queryset = queryset[:page_size]
    except ValueError:
        page_size = 10
        queryset = queryset[:page_size]
    
    # Serialize and return
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'search_parameters': {
            'title': title,
            'author': author,
            'min_price': min_price,
            'max_price': max_price,
            'ordering': ordering,
            'page_size': page_size,
        },
        'results_count': len(serializer.data),
        'results': serializer.data
    })

@api_view(['GET'])
def advanced_search(request):
    """
    Advanced search with multiple query parameters and complex filtering.
    """
    # Get all query parameters
    params = request.query_params
    
    # Complex search with Q objects
    search_query = request.query_params.get('q', '')
    if search_query:
        queryset = Book.objects.filter(
            Q(title__icontains=search_query) | 
            Q(author__icontains=search_query) |
            Q(isbn__icontains=search_query)
        )
    else:
        queryset = Book.objects.all()
    
    # Date range filtering
    date_from = params.get('date_from')
    date_to = params.get('date_to')
    
    if date_from:
        queryset = queryset.filter(published_date__gte=date_from)
    if date_to:
        queryset = queryset.filter(published_date__lte=date_to)
    
    # Multiple value filtering (e.g., ?authors=smith,jones,brown)
    authors = params.get('authors')
    if authors:
        author_list = [author.strip() for author in authors.split(',')]
        queryset = queryset.filter(author__in=author_list)
    
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'query_parameters': dict(params),
        'results': serializer.data
    })
```

### Explanation
- **URL parameters**: Captured in URL patterns using `<int:pk>` or `<str:author_name>`
- **Function parameters**: URL parameters are passed as function arguments
- **request.query_params**: Dictionary-like object for query string parameters
- **.get('param', default)**: Safely get query parameter with optional default
- **Query building**: Dynamically build Django queries based on parameters
- **Validation**: Always validate query parameters before using them
- **Q objects**: Enable complex OR queries and filtering
- **Parameter sanitization**: Prevent SQL injection and validate data types
- **Reasonable limits**: Implement limits to prevent abuse (like max page size)

---

## CRUD Operations

### Concept
Complete Create, Read, Update, Delete operations using FBVs.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.shortcuts import get_object_or_404
from .models import Book
from .serializers import BookSerializer

# CREATE - POST
@api_view(['POST'])
def create_book(request):
    """
    Create a new book.
    """
    serializer = BookSerializer(data=request.data)
    
    if serializer.is_valid():
        book = serializer.save()
        response_data = {
            'message': 'Book created successfully',
            'book': serializer.data
        }
        return Response(response_data, status=status.HTTP_201_CREATED)
    
    return Response({
        'message': 'Failed to create book',
        'errors': serializer.errors
    }, status=status.HTTP_400_BAD_REQUEST)

# READ - GET (List)
@api_view(['GET'])
def list_books(request):
    """
    Retrieve all books with optional filtering.
    """
    # Optional filtering
    author = request.query_params.get('author')
    min_price = request.query_params.get('min_price')
    
    queryset = Book.objects.all()
    
    if author:
        queryset = queryset.filter(author__icontains=author)
    
    if min_price:
        try:
            queryset = queryset.filter(price__gte=float(min_price))
        except ValueError:
            return Response(
                {'error': 'Invalid min_price value'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    # Order by creation date (newest first)
    queryset = queryset.order_by('-created_at')
    
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'count': queryset.count(),
        'books': serializer.data
    })

# READ - GET (Detail)
@api_view(['GET'])
def retrieve_book(request, pk):
    """
    Retrieve a single book by ID.
    """
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book)
    
    return Response(serializer.data)

# UPDATE - PUT (Full Update)
@api_view(['PUT'])
def update_book(request, pk):
    """
    Update a book completely (all fields required).
    """
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book, data=request.data)
    
    if serializer.is_valid():
        serializer.save()
        response_data = {
            'message': 'Book updated successfully',
            'book': serializer.data
        }
        return Response(response_data)
    
    return Response({
        'message': 'Failed to update book',
        'errors': serializer.errors
    }, status=status.HTTP_400_BAD_REQUEST)

# UPDATE - PATCH (Partial Update)
@api_view(['PATCH'])
def partial_update_book(request, pk):
    """
    Partially update a book (only provided fields).
    """
    book = get_object_or_404(Book, pk=pk)
    serializer = BookSerializer(book, data=request.data, partial=True)
    
    if serializer.is_valid():
        serializer.save()
        response_data = {
            'message': 'Book partially updated successfully',
            'book': serializer.data,
            'updated_fields': list(request.data.keys())
        }
        return Response(response_data)
    
    return Response({
        'message': 'Failed to update book',
        'errors': serializer.errors
    }, status=status.HTTP_400_BAD_REQUEST)

# DELETE
@api_view(['DELETE'])
def delete_book(request, pk):
    """
    Delete a book.
    """
    book = get_object_or_404(Book, pk=pk)
    book_title = book.title  # Store for response message
    book.delete()
    
    return Response({
        'message': f'Book "{book_title}" deleted successfully'
    }, status=status.HTTP_204_NO_CONTENT)

# Combined CRUD view
@api_view(['GET', 'POST', 'PUT', 'PATCH', 'DELETE'])
def book_crud(request, pk=None):
    """
    Combined CRUD operations in a single view.
    """
    
    # CREATE
    if request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    
    # For operations that require an existing book
    if pk is None and request.method != 'POST':
        return Response(
            {'error': 'Book ID required for this operation'}, 
            status=status.HTTP_400_BAD_REQUEST
        )
    
    if pk is not None:
        book = get_object_or_404(Book, pk=pk)
        
        # READ
        if request.method == 'GET':
            serializer = BookSerializer(book)
            return Response(serializer.data)
        
        # UPDATE (Full)
        elif request.method == 'PUT':
            serializer = BookSerializer(book, data=request.data)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
        # UPDATE (Partial)
        elif request.method == 'PATCH':
            serializer = BookSerializer(book, data=request.data, partial=True)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
        # DELETE
        elif request.method == 'DELETE':
            book.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)
    
    # LIST (GET without pk)
    elif request.method == 'GET':
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

# Bulk operations
@api_view(['POST'])
def bulk_create_books(request):
    """
    Create multiple books at once.
    """
    if not isinstance(request.data, list):
        return Response(
            {'error': 'Expected a list of books'}, 
            status=status.HTTP_400_BAD_REQUEST
        )
    
    serializer = BookSerializer(data=request.data, many=True)
    
    if serializer.is_valid():
        books = serializer.save()
        return Response({
            'message': f'{len(books)} books created successfully',
            'books': serializer.data
        }, status=status.HTTP_201_CREATED)
    
    return Response({
        'message': 'Failed to create books',
        'errors': serializer.errors
    }, status=status.HTTP_400_BAD_REQUEST)

@api_view(['DELETE'])
def bulk_delete_books(request):
    """
    Delete multiple books by IDs.
    Expected data: {"book_ids": [1, 2, 3, 4]}
    """
    book_ids = request.data.get('book_ids', [])
    
    if not book_ids:
        return Response(
            {'error': 'No book IDs provided'}, 
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # Validate that all IDs exist
    existing_books = Book.objects.filter(id__in=book_ids)
    existing_ids = list(existing_books.values_list('id', flat=True))
    missing_ids = [id for id in book_ids if id not in existing_ids]
    
    if missing_ids:
        return Response({
            'error': f'Books with IDs {missing_ids} not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    # Delete books
    deleted_count = existing_books.delete()[0]
    
    return Response({
        'message': f'{deleted_count} books deleted successfully',
        'deleted_ids': existing_ids
    }, status=status.HTTP_204_NO_CONTENT)
```

### Explanation
- **get_object_or_404()**: Django shortcut that returns object or raises 404 error
- **partial=True**: Allows partial updates in PATCH requests (only provided fields)
- **Combined views**: Single view handling multiple HTTP methods with conditional logic
- **Bulk operations**: Handle multiple records in single request for efficiency
- **Validation**: Always validate IDs exist before performing operations
- **Response messages**: Provide clear feedback about what operation was performed
- **Status codes**: Use appropriate HTTP status codes for each operation type
- **Error handling**: Comprehensive error handling for edge cases

---

## Error Handling

### Concept
Proper error handling and custom exception responses in DRF FBVs.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from rest_framework.exceptions import ValidationError, NotFound, PermissionDenied
from django.db import IntegrityError, transaction
from django.core.exceptions import ObjectDoesNotExist
import logging

# Set up logging
logger = logging.getLogger(__name__)

@api_view(['POST'])
def create_book_with_error_handling(request):
    """
    Create book with comprehensive error handling.
    """
    try:
        # Validate required fields manually
        required_fields = ['title', 'author', 'isbn']
        missing_fields = [field for field in required_fields if not request.data.get(field)]
        
        if missing_fields:
            return Response({
                'error': 'Missing required fields',
                'missing_fields': missing_fields,
                'required_fields': required_fields
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Custom business logic validation
        isbn = request.data.get('isbn')
        if len(isbn) != 13:
            return Response({
                'error': 'Invalid ISBN format',
                'message': 'ISBN must be exactly 13 characters long',
                'provided_isbn': isbn,
                'isbn_length': len(isbn)
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Check for duplicate ISBN
        if Book.objects.filter(isbn=isbn).exists():
            return Response({
                'error': 'Duplicate ISBN',
                'message': f'Book with ISBN {isbn} already exists',
                'isbn': isbn
            }, status=status.HTTP_409_CONFLICT)
        
        # Use database transaction
        with transaction.atomic():
            serializer = BookSerializer(data=request.data)
            
            if serializer.is_valid():
                book = serializer.save()
                logger.info(f'Book created successfully: {book.title} (ID: {book.id})')
                
                return Response({
                    'message': 'Book created successfully',
                    'book': serializer.data
                }, status=status.HTTP_201_CREATED)
            else:
                return Response({
                    'error': 'Validation failed',
                    'details': serializer.errors
                }, status=status.HTTP_400_BAD_REQUEST)
    
    except IntegrityError as e:
        logger.error(f'Database integrity error: {str(e)}')
        return Response({
            'error': 'Database constraint violation',
            'message': 'This operation violates database constraints',
            'details': str(e)
        }, status=status.HTTP_400_BAD_REQUEST)
    
    except Exception as e:
        logger.error(f'Unexpected error in create_book: {str(e)}', exc_info=True)
        return Response({
            'error': 'Internal server error',
            'message': 'An unexpected error occurred'
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

@api_view(['GET'])
def get_book_with_error_handling(request, pk):
    """
    Retrieve book with detailed error handling.
    """
    try:
        # Validate PK format
        try:
            pk = int(pk)
            if pk <= 0:
                raise ValueError("ID must be positive")
        except ValueError:
            return Response({
                'error': 'Invalid book ID',
                'message': 'Book ID must be a positive integer',
                'provided_id': pk
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Attempt to retrieve book
        try:
            book = Book.objects.get(pk=pk)
        except Book.DoesNotExist:
            return Response({
                'error': 'Book not found',
                'message': f'No book exists with ID {pk}',
                'book_id': pk
            }, status=status.HTTP_404_NOT_FOUND)
        
        # Check permissions (example)
        if hasattr(book, 'is_private') and book.is_private and not request.user.is_staff:
            return Response({
                'error': 'Access denied',
                'message': 'You do not have permission to view this book'
            }, status=status.HTTP_403_FORBIDDEN)
        
        serializer = BookSerializer(book)
        return Response(serializer.data)
    
    except Exception as e:
        logger.error(f'Unexpected error in get_book: {str(e)}', exc_info=True)
        return Response({
            'error': 'Internal server error',
            'message': 'An unexpected error occurred while retrieving the book'
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

@api_view(['PUT', 'PATCH'])
def update_book_with_error_handling(request, pk):
    """
    Update book with comprehensive error handling.
    """
    try:
        # Get the book
        try:
            book = Book.objects.get(pk=pk)
        except Book.DoesNotExist:
            return Response({
                'error': 'Book not found',
                'message': f'No book exists with ID {pk}'
            }, status=status.HTTP_404_NOT_FOUND)
        
        # Store original data for rollback reference
        original_data = BookSerializer(book).data
        
        # Handle partial vs full update
        partial = request.method == 'PATCH'
        
        # Validate data
        serializer = BookSerializer(book, data=request.data, partial=partial)
        
        if not serializer.is_valid():
            return Response({
                'error': 'Validation failed',
                'details': serializer.errors,
                'original_data': original_data
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Check for ISBN conflicts (if ISBN is being updated)
        new_isbn = request.data.get('isbn')
        if new_isbn and new_isbn != book.isbn:
            if Book.objects.filter(isbn=new_isbn).exclude(pk=pk).exists():
                return Response({
                    'error': 'ISBN conflict',
                    'message': f'Another book already has ISBN {new_isbn}',
                    'conflicting_isbn': new_isbn
                }, status=status.HTTP_409_CONFLICT)
        
        # Perform update in transaction
        with transaction.atomic():
            updated_book = serializer.save()
            logger.info(f'Book updated successfully: {updated_book.title} (ID: {updated_book.id})')
            
            return Response({
                'message': 'Book updated successfully',
                'book': serializer.data,
                'update_type': 'partial' if partial else 'full'
            })
    
    except IntegrityError as e:
        logger.error(f'Database integrity error during update: {str(e)}')
        return Response({
            'error': 'Database constraint violation',
            'message': 'Update violates database constraints'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    except Exception as e:
        logger.error(f'Unexpected error in update_book: {str(e)}', exc_info=True)
        return Response({
            'error': 'Internal server error',
            'message': 'An unexpected error occurred during update'
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

# Custom error handler decorator
def handle_exceptions(func):
    """
    Decorator for consistent error handling across views.
    """
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except ValidationError as e:
            return Response({
                'error': 'Validation error',
                'details': e.detail
            }, status=status.HTTP_400_BAD_REQUEST)
        except NotFound as e:
            return Response({
                'error': 'Not found',
                'message': str(e)
            }, status=status.HTTP_404_NOT_FOUND)
        except PermissionDenied as e:
            return Response({
                'error': 'Access denied',
                'message': str(e)
            }, status=status.HTTP_403_FORBIDDEN)
        except Exception as e:
            logger.error(f'Unexpected error in {func.__name__}: {str(e)}', exc_info=True)
            return Response({
                'error': 'Internal server error',
                'message': 'An unexpected error occurred'
            }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
    return wrapper

@api_view(['GET'])
@handle_exceptions
def book_list_with_decorator(request):
    """
    Example of using the error handling decorator.
    """
    books = Book.objects.all()
    serializer = BookSerializer(books, many=True)
    return Response(serializer.data)

# Error response utility function
def error_response(error_type, message, details=None, status_code=status.HTTP_400_BAD_REQUEST):
    """
    Utility function to create consistent error responses.
    """
    response_data = {
        'error': error_type,
        'message': message,
        'timestamp': timezone.now().isoformat()
    }
    
    if details:
        response_data['details'] = details
    
    return Response(response_data, status=status_code)

@api_view(['POST'])
def create_book_with_utility(request):
    """
    Example using error response utility.
    """
    serializer = BookSerializer(data=request.data)
    
    if not serializer.is_valid():
        return error_response(
            'Validation failed',
            'The provided data is invalid',
            serializer.errors,
            status.HTTP_400_BAD_REQUEST
        )
    
    try:
        book = serializer.save()
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    except IntegrityError:
        return error_response(
            'Duplicate entry',
            'A book with this ISBN already exists',
            status_code=status.HTTP_409_CONFLICT
        )
```

### Explanation
- **try/except blocks**: Catch specific exceptions and handle them appropriately
- **Logging**: Log errors for debugging and monitoring purposes
- **Validation**: Validate input data before processing
- **Business logic validation**: Custom validation beyond serializer validation
- **Database transactions**: Use transactions to ensure data consistency
- **Specific error types**: Handle different exceptions with appropriate responses
- **Error response structure**: Consistent error response format
- **Decorator pattern**: Reusable error handling logic
- **Utility functions**: Helper functions for consistent error responses
- **Status codes**: Use appropriate HTTP status codes for different error types

---

## Authentication

### Concept
Implement various authentication methods in DRF FBVs.

### Code Example
```python
from rest_framework.decorators import api_view, authentication_classes, permission_classes
from rest_framework.authentication import TokenAuthentication, SessionAuthentication, BasicAuthentication
from rest_framework.permissions import IsAuthenticated, AllowAny
from rest_framework.response import Response
from rest_framework.authtoken.models import Token
from rest_framework import status
from django.contrib.auth import authenticate
from django.contrib.auth.models import User

# Token Authentication Example
@api_view(['POST'])
@authentication_classes([])  # No authentication required for login
@permission_classes([AllowAny])
def login(request):
    """
    Login endpoint that returns authentication token.
    """
    username = request.data.get('username')
    password = request.data.get('password')
    
    if not username or not password:
        return Response({
            'error': 'Username and password required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Authenticate user
    user = authenticate(username=username, password=password)
    
    if user:
        # Get or create token
        token, created = Token.objects.get_or_create(user=user)
        
        return Response({
            'message': 'Login successful',
            'token': token.key,
            'user_id': user.id,
            'username': user.username,
            'email': user.email
        })
    else:
        return Response({
            'error': 'Invalid credentials'
        }, status=status.HTTP_401_UNAUTHORIZED)

@api_view(['POST'])
@authentication_classes([TokenAuthentication])
@permission_classes([IsAuthenticated])
def logout(request):
    """
    Logout endpoint that deletes the authentication token.
    """
    try:
        # Delete the user's token
        request.user.auth_token.delete()
        return Response({
            'message': 'Successfully logged out'
        })
    except Token.DoesNotExist:
        return Response({
            'error': 'Token not found'
        }, status=status.HTTP_400_BAD_REQUEST)

# Different authentication methods
@api_view(['GET'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated])
def protected_view(request):
    """
    View that requires authentication (Token or Session).
    """
    return Response({
        'message': f'Hello, {request.user.username}!',
        'user_id': request.user.id,
        'authentication_method': type(request.auth).__name__ if request.auth else 'Session',
        'is_staff': request.user.is_staff,
        'is_superuser': request.user.is_superuser
    })

@api_view(['GET'])
@authentication_classes([BasicAuthentication])
@permission_classes([IsAuthenticated])
def basic_auth_view(request):
    """
    View that uses HTTP Basic Authentication.
    """
    return Response({
        'message': 'Authenticated with Basic Auth',
        'user': request.user.username
    })

@api_view(['GET'])
@authentication_classes([])
@permission_classes([AllowAny])
def public_view(request):
    """
    Public view that doesn't require authentication.
    """
    return Response({
        'message': 'This is a public endpoint',
        'authenticated': request.user.is_authenticated,
        'user': request.user.username if request.user.is_authenticated else 'Anonymous'
    })

# User profile management
@api_view(['GET', 'PUT'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated])
def user_profile(request):
    """
    Get or update user profile.
    """
    if request.method == 'GET':
        return Response({
            'id': request.user.id,
            'username': request.user.username,
            'email': request.user.email,
            'first_name': request.user.first_name,
            'last_name': request.user.last_name,
            'date_joined': request.user.date_joined,
            'is_staff': request.user.is_staff
        })
    
    elif request.method == 'PUT':
        user = request.user
        
        # Update allowed fields
        user.first_name = request.data.get('first_name', user.first_name)
        user.last_name = request.data.get('last_name', user.last_name)
        user.email = request.data.get('email', user.email)
        
        # Validate email uniqueness
        email = request.data.get('email')
        if email and User.objects.filter(email=email).exclude(id=user.id).exists():
            return Response({
                'error': 'Email already in use'
            }, status=status.HTTP_400_BAD_REQUEST)
        
        user.save()
        
        return Response({
            'message': 'Profile updated successfully',
            'user': {
                'id': user.id,
                'username': user.username,
                'email': user.email,
                'first_name': user.first_name,
                'last_name': user.last_name
            }
        })

# Password change
@api_view(['POST'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated])
def change_password(request):
    """
    Change user password.
    """
    current_password = request.data.get('current_password')
    new_password = request.data.get('new_password')
    confirm_password = request.data.get('confirm_password')
    
    if not all([current_password, new_password, confirm_password]):
        return Response({
            'error': 'All password fields are required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Verify current password
    if not request.user.check_password(current_password):
        return Response({
            'error': 'Current password is incorrect'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Check password confirmation
    if new_password != confirm_password:
        return Response({
            'error': 'New passwords do not match'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Basic password validation
    if len(new_password) < 8:
        return Response({
            'error': 'Password must be at least 8 characters long'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Set new password
    request.user.set_password(new_password)
    request.user.save()
    
    # Invalidate current token if using token auth
    if hasattr(request.user, 'auth_token'):
        request.user.auth_token.delete()
        # Create new token
        new_token = Token.objects.create(user=request.user)
        
        return Response({
            'message': 'Password changed successfully',
            'new_token': new_token.key
        })
    
    return Response({
        'message': 'Password changed successfully'
    })

# Custom authentication check
@api_view(['GET'])
def check_authentication(request):
    """
    Check current authentication status without requiring auth.
    """
    if request.user.is_authenticated:
        return Response({
            'authenticated': True,
            'user': {
                'id': request.user.id,
                'username': request.user.username,
                'email': request.user.email
            },
            'auth_method': type(request.auth).__name__ if request.auth else 'Session'
        })
    else:
        return Response({
            'authenticated': False,
            'message': 'Not authenticated'
        })

# Registration endpoint
@api_view(['POST'])
@authentication_classes([])
@permission_classes([AllowAny])
def register(request):
    """
    User registration endpoint.
    """
    username = request.data.get('username')
    email = request.data.get('email')
    password = request.data.get('password')
    confirm_password = request.data.get('confirm_password')
    
    # Validation
    if not all([username, email, password, confirm_password]):
        return Response({
            'error': 'All fields are required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    if password != confirm_password:
        return Response({
            'error': 'Passwords do not match'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    if User.objects.filter(username=username).exists():
        return Response({
            'error': 'Username already exists'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    if User.objects.filter(email=email).exists():
        return Response({
            'error': 'Email already exists'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Create user
    try:
        user = User.objects.create_user(
            username=username,
            email=email,
            password=password
        )
        
        # Create token
        token = Token.objects.create(user=user)
        
        return Response({
            'message': 'User created successfully',
            'user': {
                'id': user.id,
                'username': user.username,
                'email': user.email
            },
            'token': token.key
        }, status=status.HTTP_201_CREATED)
        
    except Exception as e:
        return Response({
            'error': 'Failed to create user',
            'details': str(e)
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
```

### Explanation
- **@authentication_classes**: Specifies which authentication methods to use
- **@permission_classes**: Defines permission requirements for the view
- **TokenAuthentication**: Uses tokens in Authorization header
- **SessionAuthentication**: Uses Django's session framework
- **BasicAuthentication**: Uses HTTP Basic Auth (username:password in header)
- **IsAuthenticated**: Requires user to be logged in
- **AllowAny**: No authentication required
- **request.user**: Access to authenticated user object
- **request.auth**: Authentication credentials (token, etc.)
- **Token.objects.get_or_create()**: Get existing token or create new one
- **authenticate()**: Django's built-in authentication function
- **User management**: Registration, profile updates, password changes

---

## Permissions

### Concept
Control access to views and data using DRF's permission system.

### Code Example
```python
from rest_framework.decorators import api_view, permission_classes, authentication_classes
from rest_framework.permissions import IsAuthenticated, IsAdminUser, AllowAny, BasePermission
from rest_framework.authentication import TokenAuthentication, SessionAuthentication
from rest_framework.response import Response
from rest_framework import status
from django.contrib.auth.models import User
from .models import Book

# Custom Permission Classes
class IsOwnerOrReadOnly(BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions for any request (GET, HEAD, OPTIONS)
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        
        # Write permissions only to the owner
        return obj.owner == request.user

class IsAuthorOrReadOnly(BasePermission):
    """
    Permission for book authors to edit their books.
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions for everyone
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        
        # Write permissions only for book authors
        return obj.created_by == request.user

class IsStaffOrReadOnly(BasePermission):
    """
    Permission for staff users to edit, others can only read.
    """
    def has_permission(self, request, view):
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        return request.user.is_staff

# Basic permission examples
@api_view(['GET'])
@permission_classes([AllowAny])
def public_books(request):
    """
    Public endpoint - no authentication required.
    """
    books = Book.objects.all()[:10]  # Limit for demo
    return Response({
        'message': 'Public book list',
        'books': [{'id': b.id, 'title': b.title} for b in books],
        'user_authenticated': request.user.is_authenticated
    })

@api_view(['GET'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated])
def authenticated_books(request):
    """
    Requires authentication - any logged-in user can access.
    """
    books = Book.objects.all()
    return Response({
        'message': f'Hello {request.user.username}!',
        'books_count': books.count(),
        'user_permissions': {
            'is_staff': request.user.is_staff,
            'is_superuser': request.user.is_superuser
        }
    })

@api_view(['GET', 'POST', 'DELETE'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAdminUser])
def admin_only_books(request):
    """
    Admin-only endpoint - requires staff status.
    """
    if request.method == 'GET':
        books = Book.objects.all()
        return Response({
            'message': 'Admin book management',
            'total_books': books.count(),
            'admin_user': request.user.username
        })
    
    elif request.method == 'POST':
        # Admin can create books
        title = request.data.get('title')
        if title:
            book = Book.objects.create(
                title=title,
                author=request.data.get('author', 'Unknown'),
                isbn=request.data.get('isbn', '0000000000000'),
                published_date=request.data.get('published_date', '2024-01-01'),
                pages=request.data.get('pages', 100),
                price=request.data.get('price', 0.00)
            )
            return Response({
                'message': 'Book created by admin',
                'book': {'id': book.id, 'title': book.title}
            }, status=status.HTTP_201_CREATED)
        
        return Response({
            'error': 'Title is required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    elif request.method == 'DELETE':
        # Admin can delete all books (dangerous!)
        count = Book.objects.count()
        Book.objects.all().delete()
        return Response({
            'message': f'All {count} books deleted by admin {request.user.username}'
        })

# Custom permission usage
@api_view(['GET', 'PUT', 'DELETE'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated, IsStaffOrReadOnly])
def staff_managed_books(request, pk=None):
    """
    Staff can edit, others can only read.
    """
    if request.method == 'GET':
        if pk:
            try:
                book = Book.objects.get(pk=pk)
                return Response({
                    'id': book.id,
                    'title': book.title,
                    'author': book.author,
                    'can_edit': request.user.is_staff
                })
            except Book.DoesNotExist:
                return Response({
                    'error': 'Book not found'
                }, status=status.HTTP_404_NOT_FOUND)
        else:
            books = Book.objects.all()
            return Response({
                'books': [{'id': b.id, 'title': b.title} for b in books],
                'can_edit': request.user.is_staff
            })
    
    # PUT and DELETE require staff permission (handled by IsStaffOrReadOnly)
    elif request.method == 'PUT':
        try:
            book = Book.objects.get(pk=pk)
            book.title = request.data.get('title', book.title)
            book.author = request.data.get('author', book.author)
            book.save()
            
            return Response({
                'message': 'Book updated by staff',
                'book': {'id': book.id, 'title': book.title}
            })
        except Book.DoesNotExist:
            return Response({
                'error': 'Book not found'
            }, status=status.HTTP_404_NOT_FOUND)
    
    elif request.method == 'DELETE':
        try:
            book = Book.objects.get(pk=pk)
            book_title = book.title
            book.delete()
            
            return Response({
                'message': f'Book "{book_title}" deleted by staff'
            })
        except Book.DoesNotExist:
            return Response({
                'error': 'Book not found'
            }, status=status.HTTP_404_NOT_FOUND)

# Permission checking function
def check_book_permission(user, book, action):
    """
    Helper function to check permissions for book operations.
    """
    if action == 'read':
        return True  # Anyone can read
    
    if action in ['create', 'update', 'delete']:
        if user.is_staff:
            return True
        if hasattr(book, 'created_by') and book.created_by == user:
            return True
        return False
    
    return False

@api_view(['GET', 'PUT', 'DELETE'])
@authentication_classes([TokenAuthentication, SessionAuthentication])  
@permission_classes([IsAuthenticated])
def book_with_custom_permission_check(request, pk):
    """
    Manual permission checking within the view.
    """
    try:
        book = Book.objects.get(pk=pk)
    except Book.DoesNotExist:
        return Response({
            'error': 'Book not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    if request.method == 'GET':
        # Anyone authenticated can read
        if not check_book_permission(request.user, book, 'read'):
            return Response({
                'error': 'Permission denied'
            }, status=status.HTTP_403_FORBIDDEN)
        
        return Response({
            'id': book.id,
            'title': book.title,
            'author': book.author,
            'permissions': {
                'can_update': check_book_permission(request.user, book, 'update'),
                'can_delete': check_book_permission(request.user, book, 'delete')
            }
        })
    
    elif request.method == 'PUT':
        if not check_book_permission(request.user, book, 'update'):
            return Response({
                'error': 'You do not have permission to update this book'
            }, status=status.HTTP_403_FORBIDDEN)
        
        book.title = request.data.get('title', book.title)
        book.save()
        
        return Response({
            'message': 'Book updated successfully',
            'book': {'id': book.id, 'title': book.title}
        })
    
    elif request.method == 'DELETE':
        if not check_book_permission(request.user, book, 'delete'):
            return Response({
                'error': 'You do not have permission to delete this book'
            }, status=status.HTTP_403_FORBIDDEN)
        
        book_title = book.title
        book.delete()
        
        return Response({
            'message': f'Book "{book_title}" deleted successfully'
        })

# Multiple permission classes
@api_view(['POST'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated, IsStaffOrReadOnly])
def multiple_permissions_example(request):
    """
    Example with multiple permission classes.
    All permission classes must pass for access to be granted.
    """
    return Response({
        'message': 'You have passed all permission checks!',
        'user': request.user.username,
        'is_staff': request.user.is_staff
    })

# Conditional permissions
@api_view(['GET', 'POST'])
@authentication_classes([TokenAuthentication, SessionAuthentication])
def conditional_permissions(request):
    """
    Apply different permissions based on HTTP method.
    """
    if request.method == 'GET':
        # Anyone can read (no permission check needed)
        return Response({
            'message': 'Public data',
            'books_count': Book.objects.count()
        })
    
    elif request.method == 'POST':
        # Only authenticated users can create
        if not request.user.is_authenticated:
            return Response({
                'error': 'Authentication required for creating books'
            }, status=status.HTTP_401_UNAUTHORIZED)
        
        # Only staff can create books
        if not request.user.is_staff:
            return Response({
                'error': 'Staff privileges required for creating books'
            }, status=status.HTTP_403_FORBIDDEN)
        
        title = request.data.get('title')
        if title:
            book = Book.objects.create(
                title=title,
                author=request.data.get('author', 'Unknown'),
                isbn=request.data.get('isbn', '0000000000000'),
                published_date=request.data.get('published_date', '2024-01-01'),
                pages=request.data.get('pages', 100),
                price=request.data.get('price', 0.00)
            )
            return Response({
                'message': 'Book created successfully',
                'book': {'id': book.id, 'title': book.title}
            }, status=status.HTTP_201_CREATED)
        
        return Response({
            'error': 'Title is required'
        }, status=status.HTTP_400_BAD_REQUEST)
```

### Explanation
- **BasePermission**: Base class for creating custom permissions
- **has_permission()**: Check view-level permissions
- **has_object_permission()**: Check object-level permissions
- **IsAuthenticated**: User must be logged in
- **IsAdminUser**: User must have is_staff=True
- **AllowAny**: No restrictions (default for views without @permission_classes)
- **Multiple permissions**: All permission classes must pass
- **Custom permission logic**: Implement business-specific access control
- **Permission checking functions**: Reusable permission logic
- **Conditional permissions**: Different permissions based on HTTP method or other factors
- **403 vs 401**: 403 for permission denied, 401 for authentication required

---

## Pagination

### Concept
Implement pagination to handle large datasets efficiently in FBVs.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination
from rest_framework import status
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.db.models import Q
from .models import Book
from .serializers import BookSerializer

# Custom pagination class
class CustomPageNumberPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100
    page_query_param = 'page'

# Manual pagination with Django's Paginator
@api_view(['GET'])
def books_with_manual_pagination(request):
    """
    Manual pagination using Django's Paginator.
    """
    # Get all books
    books_queryset = Book.objects.all().order_by('id')
    
    # Get pagination parameters
    page_number = request.query_params.get('page', 1)
    page_size = request.query_params.get('page_size', 10)
    
    # Validate page_size
    try:
        page_size = int(page_size)
        if page_size > 100:
            page_size = 100
        elif page_size < 1:
            page_size = 10
    except ValueError:
        page_size = 10
    
    # Create paginator
    paginator = Paginator(books_queryset, page_size)
    
    try:
        books_page = paginator.page(page_number)
    except PageNotAnInteger:
        books_page = paginator.page(1)
    except EmptyPage:
        books_page = paginator.page(paginator.num_pages)
    
    # Serialize the books
    serializer = BookSerializer(books_page.object_list, many=True)
    
    # Build pagination info
    pagination_info = {
        'count': paginator.count,
        'num_pages': paginator.num_pages,
        'current_page': books_page.number,
        'page_size': page_size,
        'has_next': books_page.has_next(),
        'has_previous': books_page.has_previous(),
        'next_page': books_page.next_page_number() if books_page.has_next() else None,
        'previous_page': books_page.previous_page_number() if books_page.has_previous() else None,
    }
    
    return Response({
        'pagination': pagination_info,
        'results': serializer.data
    })

# DRF Pagination with PageNumberPagination
@api_view(['GET'])
def books_with_drf_pagination(request):
    """
    Using DRF's built-in pagination.
    """
    books_queryset = Book.objects.all().order_by('id')
    
    # Create paginator instance
    paginator = CustomPageNumberPagination()
    
    # Paginate the queryset
    paginated_books = paginator.paginate_queryset(books_queryset, request)
    
    # Serialize the paginated data
    serializer = BookSerializer(paginated_books, many=True)
    
    # Return paginated response
    return paginator.get_paginated_response(serializer.data)

# LimitOffsetPagination example
class CustomLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 20
    limit_query_param = 'limit'
    offset_query_param = 'offset'
    max_limit = 100

@api_view(['GET'])
def books_with_limit_offset_pagination(request):
    """
    Using limit/offset pagination.
    """
    books_queryset = Book.objects.all().order_by('id')
    
    # Create paginator instance
    paginator = CustomLimitOffsetPagination()
    
    # Paginate the queryset
    paginated_books = paginator.paginate_queryset(books_queryset, request)
    
    # Serialize the paginated data
    serializer = BookSerializer(paginated_books, many=True)
    
    # Return paginated response
    return paginator.get_paginated_response(serializer.data)

# Search with pagination
@api_view(['GET'])
def search_books_with_pagination(request):
    """
    Search books with pagination support.
    """
    # Get search parameters
    search_query = request.query_params.get('q', '')
    author = request.query_params.get('author', '')
    min_price = request.query_params.get('min_price')
    max_price = request.query_params.get('max_price')
    
    # Build base queryset
    queryset = Book.objects.all()
    
    # Apply search filters
    if search_query:
        queryset = queryset.filter(
            Q(title__icontains=search_query) |
            Q(author__icontains=search_query) |
            Q(isbn__icontains=search_query)
        )
    
    if author:
        queryset = queryset.filter(author__icontains=author)
    
    if min_price:
        try:
            queryset = queryset.filter(price__gte=float(min_price))
        except ValueError:
            return Response({
                'error': 'Invalid min_price value'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    if max_price:
        try:
            queryset = queryset.filter(price__lte=float(max_price))
        except ValueError:
            return Response({
                'error': 'Invalid max_price value'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    # Order results
    ordering = request.query_params.get('ordering', 'id')
    valid_orderings = ['id', '-id', 'title', '-title', 'author', '-author', 'price', '-price']
    if ordering in valid_orderings:
        queryset = queryset.order_by(ordering)
    
    # Apply pagination
    paginator = CustomPageNumberPagination()
    paginated_queryset = paginator.paginate_queryset(queryset, request)
    
    # Serialize
    serializer = BookSerializer(paginated_queryset, many=True)
    
    # Add search info to response
    response = paginator.get_paginated_response(serializer.data)
    response.data['search_info'] = {
        'query': search_query,
        'author': author,
        'min_price': min_price,
        'max_price': max_price,
        'ordering': ordering,
        'total_matches': queryset.count()
    }
    
    return response

# Custom pagination response format
@api_view(['GET'])
def books_with_custom_pagination_format(request):
    """
    Custom pagination response format.
    """
    books_queryset = Book.objects.all().order_by('id')
    
    # Get pagination parameters
    page = int(request.query_params.get('page', 1))
    per_page = int(request.query_params.get('per_page', 15))
    
    # Limit per_page
    if per_page > 50:
        per_page = 50
    
    # Create paginator
    paginator = Paginator(books_queryset, per_page)
    
    try:
        books_page = paginator.page(page)
    except PageNotAnInteger:
        books_page = paginator.page(1)
    except EmptyPage:
        books_page = paginator.page(paginator.num_pages)
    
    # Serialize
    serializer = BookSerializer(books_page.object_list, many=True)
    
    # Custom response format
    return Response({
        'data': serializer.data,
        'meta': {
            'pagination': {
                'page': books_page.number,
                'per_page': per_page,
                'total': paginator.count,
                'pages': paginator.num_pages,
                'has_next': books_page.has_next(),
                'has_prev': books_page.has_previous(),
                'next_page': books_page.next_page_number() if books_page.has_next() else None,
                'prev_page': books_page.previous_page_number() if books_page.has_previous() else None,
            }
        },
        'links': {
            'first': f"?page=1&per_page={per_page}",
            'last': f"?page={paginator.num_pages}&per_page={per_page}",
            'next': f"?page={books_page.next_page_number()}&per_page={per_page}" if books_page.has_next() else None,
            'prev': f"?page={books_page.previous_page_number()}&per_page={per_page}" if books_page.has_previous() else None,
        }
    })

# Cursor pagination for large datasets
@api_view(['GET'])
def books_with_cursor_pagination(request):
    """
    Simple cursor-based pagination for large datasets.
    """
    # Get cursor parameter (last seen ID)
    cursor = request.query_params.get('cursor')
    limit = int(request.query_params.get('limit', 20))
    
    # Limit the limit
    if limit > 100:
        limit = 100
    
    # Build queryset
    queryset = Book.objects.all().order_by('id')
    
    if cursor:
        try:
            cursor_id = int(cursor)
            queryset = queryset.filter(id__gt=cursor_id)
        except ValueError:
            return Response({
                'error': 'Invalid cursor value'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    # Get one extra to check if there's a next page
    books = list(queryset[:limit + 1])
    
    has_next = len(books) > limit
    if has_next:
        books = books[:limit]  # Remove the extra item
    
    # Serialize
    serializer = BookSerializer(books, many=True)
    
    # Get next cursor
    next_cursor = books[-1].id if books and has_next else None
    
    return Response({
        'results': serializer.data,
        'pagination': {
            'limit': limit,
            'has_next': has_next,
            'next_cursor': next_cursor,
            'count': len(books)
        }
    })

# Pagination utility function
def paginate_queryset(queryset, request, page_size=20):
    """
    Utility function for consistent pagination across views.
    """
    page = int(request.query_params.get('page', 1))
    size = int(request.query_params.get('page_size', page_size))
    
    # Limit page size
    if size > 100:
        size = 100
    elif size < 1:
        size = page_size
    
    paginator = Paginator(queryset, size)
    
    try:
        page_obj = paginator.page(page)
    except PageNotAnInteger:
        page_obj = paginator.page(1)
    except EmptyPage:
        page_obj = paginator.page(paginator.num_pages)
    
    return {
        'objects': page_obj.object_list,
        'pagination': {
            'count': paginator.count,
            'num_pages': paginator.num_pages,
            'current_page': page_obj.number,
            'page_size': size,
            'has_next': page_obj.has_next(),
            'has_previous': page_obj.has_previous(),
        }
    }

@api_view(['GET'])
def books_with_utility_pagination(request):
    """
    Using the pagination utility function.
    """
    books_queryset = Book.objects.all().order_by('title')
    paginated_data = paginate_queryset(books_queryset, request)
    
    serializer = BookSerializer(paginated_data['objects'], many=True)
    
    return Response({
        'results': serializer.data,
        'pagination': paginated_data['pagination']
    })
```

### Explanation
- **Paginator**: Django's built-in pagination class
- **PageNumberPagination**: DRF's page-based pagination
- **LimitOffsetPagination**: DRF's limit/offset pagination
- **page_size_query_param**: URL parameter for page size
- **max_page_size**: Maximum allowed page size
- **paginate_queryset()**: Applies pagination to queryset
- **get_paginated_response()**: Returns formatted paginated response
- **Cursor pagination**: Efficient for large datasets using ID-based cursors
- **Custom formats**: Create custom pagination response structures
- **Search + pagination**: Combine filtering with pagination
- **Utility functions**: Reusable pagination logic

---

## Filtering and Searching

### Concept
Implement filtering and searching capabilities in FBVs for better data retrieval.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.db.models import Q, Count, Avg, Min, Max
from django.db.models.functions import Lower
from datetime import datetime, timedelta
from .models import Book
from .serializers import BookSerializer

# Basic filtering
@api_view(['GET'])
def filter_books(request):
    """
    Basic filtering with query parameters.
    """
    queryset = Book.objects.all()
    
    # Single field filters
    title = request.query_params.get('title')
    author = request.query_params.get('author')
    isbn = request.query_params.get('isbn')
    
    if title:
        queryset = queryset.filter(title__icontains=title)
    
    if author:
        queryset = queryset.filter(author__icontains=author)
    
    if isbn:
        queryset = queryset.filter(isbn=isbn)
    
    # Price range filtering
    min_price = request.query_params.get('min_price')
    max_price = request.query_params.get('max_price')
    
    if min_price:
        try:
            queryset = queryset.filter(price__gte=float(min_price))
        except ValueError:
            return Response({
                'error': 'Invalid min_price format'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    if max_price:
        try:
            queryset = queryset.filter(price__lte=float(max_price))
        except ValueError:
            return Response({
                'error': 'Invalid max_price format'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    # Date range filtering
    published_after = request.query_params.get('published_after')
    published_before = request.query_params.get('published_before')
    
    if published_after:
        try:
            date_after = datetime.strptime(published_after, '%Y-%m-%d').date()
            queryset = queryset.filter(published_date__gte=date_after)
        except ValueError:
            return Response({
                'error': 'Invalid published_after format. Use YYYY-MM-DD'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    if published_before:
        try:
            date_before = datetime.strptime(published_before, '%Y-%m-%d').date()
            queryset = queryset.filter(published_date__lte=date_before)
        except ValueError:
            return Response({
                'error': 'Invalid published_before format. Use YYYY-MM-DD'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    # Pages range filtering
    min_pages = request.query_params.get('min_pages')
    max_pages = request.query_params.get('max_pages')
    
    if min_pages:
        try:
            queryset = queryset.filter(pages__gte=int(min_pages))
        except ValueError:
            return Response({
                'error': 'Invalid min_pages format'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    if max_pages:
        try:
            queryset = queryset.filter(pages__lte=int(max_pages))
        except ValueError:
            return Response({
                'error': 'Invalid max_pages format'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    # Ordering
    ordering = request.query_params.get('ordering', 'id')
    valid_orderings = [
        'id', '-id', 'title', '-title', 'author', '-author', 
        'price', '-price', 'published_date', '-published_date', 'pages', '-pages'
    ]
    
    if ordering in valid_orderings:
        queryset = queryset.order_by(ordering)
    
    # Serialize and return
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'count': queryset.count(),
        'filters_applied': {
            'title': title,
            'author': author,
            'isbn': isbn,
            'min_price': min_price,
            'max_price': max_price,
            'published_after': published_after,
            'published_before': published_before,
            'min_pages': min_pages,
            'max_pages': max_pages,
            'ordering': ordering
        },
        'results': serializer.data
    })

# Advanced search with Q objects
@api_view(['GET'])
def search_books(request):
    """
    Advanced search using Q objects for complex queries.
    """
    query = request.query_params.get('q', '').strip()
    
    if not query:
        return Response({
            'error': 'Search query parameter "q" is required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Split query into terms
    search_terms = query.split()
    
    # Build complex search query
    search_query = Q()
    
    for term in search_terms:
        term_query = (
            Q(title__icontains=term) |
            Q(author__icontains=term) |
            Q(isbn__icontains=term)
        )
        search_query &= term_query  # AND logic for multiple terms
    
    # Apply search
    queryset = Book.objects.filter(search_query)
    
    # Additional filters
    category = request.query_params.get('category')
    if category:
        # Assuming books have categories
        queryset = queryset.filter(category__icontains=category)
    
    # Price range
    price_range = request.query_params.get('price_range')
    if price_range:
        if price_range == 'under_20':
            queryset = queryset.filter(price__lt=20)
        elif price_range == '20_to_50':
            queryset = queryset.filter(price__gte=20, price__lte=50)
        elif price_range == 'over_50':
            queryset = queryset.filter(price__gt=50)
    
    # Recent publications (last year)
    recent_only = request.query_params.get('recent_only')
    if recent_only == 'true':
        one_year_ago = datetime.now().date() - timedelta(days=365)
        queryset = queryset.filter(published_date__gte=one_year_ago)
    
    # Order by relevance (simplified - by title match first)
    ordering = request.query_params.get('ordering', 'relevance')
    if ordering == 'relevance':
        # Custom relevance ordering
        queryset = queryset.extra(
            select={
                'title_match': "CASE WHEN title ILIKE %s THEN 1 ELSE 2 END"
            },
            select_params=[f'%{query}%']
        ).order_by('title_match', 'title')
    elif ordering in ['title', '-title', 'author', '-author', 'price', '-price']:
        queryset = queryset.order_by(ordering)
    
    # Limit results
    limit = int(request.query_params.get('limit', 50))
    if limit > 200:
        limit = 200
    
    queryset = queryset[:limit]
    
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'query': query,
        'search_terms': search_terms,
        'total_results': len(serializer.data),
        'results': serializer.data
    })

# Faceted search with aggregations
@api_view(['GET'])
def faceted_search(request):
    """
    Faceted search with aggregations for filters.
    """
    # Base queryset
    queryset = Book.objects.all()
    
    # Apply basic filters first
    search_query = request.query_params.get('q')
    if search_query:
        queryset = queryset.filter(
            Q(title__icontains=search_query) |
            Q(author__icontains=search_query)
        )
    
    # Get facet counts before applying facet filters
    facets = {
        'authors': queryset.values('author').annotate(
            count=Count('id')
        ).order_by('-count')[:20],
        
        'price_ranges': {
            'under_20': queryset.filter(price__lt=20).count(),
            '20_to_50': queryset.filter(price__gte=20, price__lte=50).count(),
            'over_50': queryset.filter(price__gt=50).count(),
        },
        
        'publication_years': queryset.extra(
            select={'year': 'EXTRACT(year FROM published_date)'}
        ).values('year').annotate(
            count=Count('id')
        ).order_by('-year')[:10],
        
        'page_ranges': {
            'short': queryset.filter(pages__lt=200).count(),
            'medium': queryset.filter(pages__gte=200, pages__lt=400).count(),
            'long': queryset.filter(pages__gte=400).count(),
        }
    }
    
    # Apply facet filters
    selected_author = request.query_params.get('author')
    if selected_author:
        queryset = queryset.filter(author=selected_author)
    
    selected_price_range = request.query_params.get('price_range')
    if selected_price_range:
        if selected_price_range == 'under_20':
            queryset = queryset.filter(price__lt=20)
        elif selected_price_range == '20_to_50':
            queryset = queryset.filter(price__gte=20, price__lte=50)
        elif selected_price_range == 'over_50':
            queryset = queryset.filter(price__gt=50)
    
    selected_year = request.query_params.get('year')
    if selected_year:
        try:
            year = int(selected_year)
            queryset = queryset.filter(published_date__year=year)
        except ValueError:
            pass
    
    # Get aggregation statistics
    stats = queryset.aggregate(
        total_books=Count('id'),
        avg_price=Avg('price'),
        min_price=Min('price'),
        max_price=Max('price'),
        avg_pages=Avg('pages')
    )
    
    # Order and serialize results
    ordering = request.query_params.get('ordering', 'title')
    if ordering in ['title', '-title', 'author', '-author', 'price', '-price']:
        queryset = queryset.order_by(ordering)
    
    serializer = BookSerializer(queryset[:100], many=True)  # Limit to 100 results
    
    return Response({
        'facets': facets,
        'statistics': stats,
        'applied_filters': {
            'search_query': search_query,
            'author': selected_author,
            'price_range': selected_price_range,
            'year': selected_year,
            'ordering': ordering
        },
        'results': serializer.data
    })

# Full-text search simulation
@api_view(['GET'])
def full_text_search(request):
    """
    Simulate full-text search with ranking.
    """
    query = request.query_params.get('q', '').strip()
    
    if not query:
        return Response({
            'error': 'Search query required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Different search strategies
    search_type = request.query_params.get('search_type', 'all_fields')
    
    if search_type == 'title_only':
        queryset = Book.objects.filter(title__icontains=query)
    elif search_type == 'author_only':
        queryset = Book.objects.filter(author__icontains=query)
    elif search_type == 'exact_title':
        queryset = Book.objects.filter(title__iexact=query)
    elif search_type == 'starts_with':
        queryset = Book.objects.filter(title__istartswith=query)
    else:  # all_fields
        queryset = Book.objects.filter(
            Q(title__icontains=query) |
            Q(author__icontains=query) |
            Q(isbn__icontains=query)
        )
    
    # Simulate ranking by multiple factors
    queryset = queryset.extra(
        select={
            'title_score': """
                CASE 
                    WHEN LOWER(title) = LOWER(%s) THEN 100
                    WHEN LOWER(title) LIKE LOWER(%s) THEN 80
                    WHEN LOWER(title) LIKE LOWER(%s) THEN 60
                    ELSE 40
                END
            """,
            'author_score': """
                CASE
                    WHEN LOWER(author) = LOWER(%s) THEN 50
                    WHEN LOWER(author) LIKE LOWER(%s) THEN 30
                    ELSE 10
                END
            """
        },
        select_params=[
            query, f'{query}%', f'%{query}%',  # for title_score
            query, f'%{query}%'  # for author_score
        ]
    ).extra(
        select={'total_score': 'title_score + author_score'}
    ).order_by('-total_score', 'title')
    
    # Highlight search terms in results (simplified)
    results = []
    for book in queryset[:50]:  # Limit results
        book_data = BookSerializer(book).data
        
        # Simple highlighting
        if query.lower() in book.title.lower():
            highlighted_title = book.title.replace(
                query, f'<mark>{query}</mark>'
            )
            book_data['highlighted_title'] = highlighted_title
        
        if query.lower() in book.author.lower():
            highlighted_author = book.author.replace(
                query, f'<mark>{query}</mark>'
            )
            book_data['highlighted_author'] = highlighted_author
        
        # Add relevance score if available
        if hasattr(book, 'total_score'):
            book_data['relevance_score'] = book.total_score
        
        results.append(book_data)
    
    return Response({
        'query': query,
        'search_type': search_type,
        'total_results': len(results),
        'results': results
    })

# Filter utility function
def apply_filters(queryset, filters_dict):
    """
    Utility function to apply filters to a queryset.
    """
    for field, value in filters_dict.items():
        if value is not None and value != '':
            if field.endswith('__gte') or field.endswith('__lte'):
                try:
                    value = float(value)
                except ValueError:
                    continue
            elif field.endswith('__icontains'):
                pass  # String field, no conversion needed
            
            filter_kwargs = {field: value}
            queryset = queryset.filter(**filter_kwargs)
    
    return queryset

@api_view(['GET'])
def books_with_utility_filters(request):
    """
    Example using the filter utility function.
    """
    queryset = Book.objects.all()
    
    # Define available filters
    filters = {
        'title__icontains': request.query_params.get('title'),
        'author__icontains': request.query_params.get('author'),
        'price__gte': request.query_params.get('min_price'),
        'price__lte': request.query_params.get('max_price'),
        'pages__gte': request.query_params.get('min_pages'),
        'pages__lte': request.query_params.get('max_pages'),
    }
    
    # Apply filters
    queryset = apply_filters(queryset, filters)
    
    # Apply ordering
    ordering = request.query_params.get('ordering', 'title')
    if ordering in ['title', '-title', 'author', '-author', 'price', '-price']:
        queryset = queryset.order_by(ordering)
    
    serializer = BookSerializer(queryset, many=True)
    
    return Response({
        'applied_filters': {k: v for k, v in filters.items() if v},
        'count': queryset.count(),
        'results': serializer.data
    })
```

### Explanation
- **Q objects**: Enable complex OR/AND queries
- **icontains**: Case-insensitive partial matching
- **gte/lte**: Greater than or equal / Less than or equal comparisons
- **Date filtering**: Handle date range queries with proper validation
- **Search terms**: Split search query and apply multiple filters
- **Faceted search**: Provide filter options with counts
- **Aggregations**: Count, Avg, Min, Max for statistics
- **Full-text search**: Simulate relevance scoring and ranking
- **Highlighting**: Mark search terms in results
- **Utility functions**: Reusable filtering logic
- **Input validation**: Always validate user input before filtering

---

## File Uploads

### Concept
Handle file uploads in DRF FBVs, including image uploads and file validation.

### Code Example
```python
from rest_framework.decorators import api_view, parser_classes
from rest_framework.parsers import MultiPartParser, FormParser, JSONParser
from rest_framework.response import Response
from rest_framework import status
from django.core.files.storage import default_storage
from django.core.files.base import ContentFile
import os
import uuid
from PIL import Image
import magic

# Enhanced Book model for file uploads (add to models.py)
"""
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    isbn = models.CharField(max_length=13, unique=True)
    published_date = models.DateField()
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    cover_image = models.ImageField(upload_to='book_covers/', blank=True, null=True)
    pdf_file = models.FileField(upload_to='book_pdfs/', blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
"""

# Basic file upload
@api_view(['POST'])
@parser_classes([MultiPartParser, FormParser])
def upload_book_cover(request, book_id):
    """
    Upload cover image for a book.
    """
    try:
        book = Book.objects.get(id=book_id)
    except Book.DoesNotExist:
        return Response({
            'error': 'Book not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    # Check if file is provided
    if 'cover_image' not in request.FILES:
        return Response({
            'error': 'No file provided. Use "cover_image" field name.'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    uploaded_file = request.FILES['cover_image']
    
    # Validate file type
    allowed_types = ['image/jpeg', 'image/png', 'image/gif', 'image/webp']
    file_type = magic.from_buffer(uploaded_file.read(1024), mime=True)
    uploaded_file.seek(0)  # Reset file pointer
    
    if file_type not in allowed_types:
        return Response({
            'error': f'Invalid file type: {file_type}. Allowed types: {allowed_types}'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Validate file size (max 5MB)
    max_size = 5 * 1024 * 1024  # 5MB
    if uploaded_file.size > max_size:
        return Response({
            'error': f'File too large. Maximum size: {max_size / (1024*1024)}MB'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Generate unique filename
    file_extension = os.path.splitext(uploaded_file.name)[1]
    unique_filename = f"{uuid.uuid4()}{file_extension}"
    
    # Save file
    book.cover_image.save(unique_filename, uploaded_file, save=True)
    
    return Response({
        'message': 'Cover image uploaded successfully',
        'book_id': book.id,
        'cover_url': book.cover_image.url if book.cover_image else None,
        'file_info': {
            'name': uploaded_file.name,
            'size': uploaded_file.size,
            'type': file_type
        }
    }, status=status.HTTP_201_CREATED)

# Multiple file uploads
@api_view(['POST'])
@parser_classes([MultiPartParser, FormParser])
def upload_multiple_files(request):
    """
    Upload multiple files at once.
    """
    # Check for files
    if not request.FILES:
        return Response({
            'error': 'No files provided'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    uploaded_files = []
    errors = []
    
    for field_name, uploaded_file in request.FILES.items():
        try:
            # Validate each file
            if uploaded_file.size > 10 * 1024 * 1024:  # 10MB limit
                errors.append(f'{uploaded_file.name}: File too large')
                continue
            
            # Determine file type
            file_type = magic.from_buffer(uploaded_file.read(1024), mime=True)
            uploaded_file.seek(0)
            
            # Generate unique path
            file_extension = os.path.splitext(uploaded_file.name)[1]
            unique_filename = f"uploads/{uuid.uuid4()}{file_extension}"
            
            # Save file
            saved_path = default_storage.save(unique_filename, uploaded_file)
            
            uploaded_files.append({
                'field_name': field_name,
                'original_name': uploaded_file.name,
                'saved_path': saved_path,
                'url': default_storage.url(saved_path),
                'size': uploaded_file.size,
                'type': file_type
            })
            
        except Exception as e:
            errors.append(f'{uploaded_file.name}: {str(e)}')
    
    return Response({
        'message': f'{len(uploaded_files)} files uploaded successfully',
        'uploaded_files': uploaded_files,
        'errors': errors
    }, status=status.HTTP_201_CREATED if uploaded_files else status.HTTP_400_BAD_REQUEST)

# Image processing and validation
@api_view(['POST'])
@parser_classes([MultiPartParser, FormParser])
def upload_and_process_image(request):
    """
    Upload image with processing (resize, format conversion).
    """
    if 'image' not in request.FILES:
        return Response({
            'error': 'No image file provided'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    uploaded_file = request.FILES['image']
    
    try:
        # Open and validate image
        image = Image.open(uploaded_file)
        
        # Validate image properties
        max_width, max_height = 2000, 2000
        if image.width > max_width or image.height > max_height:
            return Response({
                'error': f'Image too large. Maximum dimensions: {max_width}x{max_height}'
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Convert to RGB if necessary
        if image.mode not in ['RGB', 'RGBA']:
            image = image.convert('RGB')
        
        # Create thumbnail
        thumbnail_size = (300, 300)
        thumbnail = image.copy()
        thumbnail.thumbnail(thumbnail_size, Image.Resampling.LANCZOS)
        
        # Save original image
        original_filename = f"images/original/{uuid.uuid4()}.jpg"
        original_path = default_storage.save(original_filename, uploaded_file)
        
        # Save thumbnail
        from io import BytesIO
        thumbnail_buffer = BytesIO()
        thumbnail.save(thumbnail_buffer, format='JPEG', quality=85)
        thumbnail_content = ContentFile(thumbnail_buffer.getvalue())
        
        thumbnail_filename = f"images/thumbnails/{uuid.uuid4()}.jpg"
        thumbnail_path = default_storage.save(thumbnail_filename, thumbnail_content)
        
        return Response({
            'message': 'Image uploaded and processed successfully',
            'original': {
                'path': original_path,
                'url': default_storage.url(original_path),
                'width': image.width,
                'height': image.height
            },
            'thumbnail': {
                'path': thumbnail_path,
                'url': default_storage.url(thumbnail_path),
                'width': thumbnail.width,
                'height': thumbnail.height
            }
        }, status=status.HTTP_201_CREATED)
        
    except Exception as e:
        return Response({
            'error': f'Invalid image file: {str(e)}'
        }, status=status.HTTP_400_BAD_REQUEST)

# File upload with metadata
@api_view(['POST'])
@parser_classes([MultiPartParser, FormParser, JSONParser])
def upload_book_with_files(request):
    """
    Create book with file uploads and metadata.
    """
    # Extract book data
    book_data = {
        'title': request.data.get('title'),
        'author': request.data.get('author'),
        'isbn': request.data.get('isbn'),
        'published_date': request.data.get('published_date'),
        'pages': request.data.get('pages'),
        'price': request.data.get('price'),
    }
    
    # Validate required fields
    required_fields = ['title', 'author', 'isbn']
    missing_fields = [field for field in required_fields if not book_data.get(field)]
    
    if missing_fields:
        return Response({
            'error': 'Missing required fields',
            'missing_fields': missing_fields
        }, status=status.HTTP_400_BAD_REQUEST)
    
    try:
        # Create book instance
        book = Book.objects.create(**book_data)
        
        # Handle cover image upload
        if 'cover_image' in request.FILES:
            cover_file = request.FILES['cover_image']
            
            # Validate image
            try:
                image = Image.open(cover_file)
                if image.width > 1500 or image.height > 1500:
                    return Response({
                        'error': 'Cover image too large. Maximum: 1500x1500 pixels'
                    }, status=status.HTTP_400_BAD_REQUEST)
                
                book.cover_image.save(
                    f"cover_{book.id}_{uuid.uuid4()}.jpg",
                    cover_file,
                    save=True
                )
            except Exception as e:
                book.delete()  # Clean up created book
                return Response({
                    'error': f'Invalid cover image: {str(e)}'
                }, status=status.HTTP_400_BAD_REQUEST)
        
        # Handle PDF upload
        if 'pdf_file' in request.FILES:
            pdf_file = request.FILES['pdf_file']
            
            # Validate PDF
            file_type = magic.from_buffer(pdf_file.read(1024), mime=True)
            pdf_file.seek(0)
            
            if file_type != 'application/pdf':
                book.delete()  # Clean up
                return Response({
                    'error': 'Invalid PDF file'
                }, status=status.HTTP_400_BAD_REQUEST)
            
            # Check file size (max 50MB for PDFs)
            if pdf_file.size > 50 * 1024 * 1024:
                book.delete()  # Clean up
                return Response({
                    'error': 'PDF file too large. Maximum: 50MB'
                }, status=status.HTTP_400_BAD_REQUEST)
            
            book.pdf_file.save(
                f"pdf_{book.id}_{uuid.uuid4()}.pdf",
                pdf_file,
                save=True
            )
        
        # Serialize response
        from .serializers import BookSerializer
        serializer = BookSerializer(book)
        
        return Response({
            'message': 'Book created with files successfully',
            'book': serializer.data
        }, status=status.HTTP_201_CREATED)
        
    except Exception as e:
        # Clean up if book was created
        if 'book' in locals():
            book.delete()
        
        return Response({
            'error': f'Failed to create book: {str(e)}'
        }, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

# File download endpoint
@api_view(['GET'])
def download_book_file(request, book_id, file_type):
    """
    Download book files (cover image or PDF).
    """
    try:
        book = Book.objects.get(id=book_id)
    except Book.DoesNotExist:
        return Response({
            'error': 'Book not found'
        }, status=status.HTTP_404_NOT_FOUND)
    
    if file_type == 'cover':
        if not book.cover_image:
            return Response({
                'error': 'No cover image available for this book'
            }, status=status.HTTP_404_NOT_FOUND)
        
        from django.http import FileResponse
        return FileResponse(
            book.cover_image.open(),
            as_attachment=True,
            filename=f"cover_{book.title}.jpg"
        )
    
    elif file_type == 'pdf':
        if not book.pdf_file:
            return Response({
                'error': 'No PDF file available for this book'
            }, status=status.HTTP_404_NOT_FOUND)
        
        from django.http import FileResponse
        return FileResponse(
            book.pdf_file.open(),
            as_attachment=True,
            filename=f"{book.title}.pdf"
        )
    
    else:
        return Response({
            'error': 'Invalid file type. Use "cover" or "pdf"'
        }, status=status.HTTP_400_BAD_REQUEST)

# File validation utility
def validate_uploaded_file(uploaded_file, allowed_types=None, max_size=None):
    """
    Utility function to validate uploaded files.
    """
    errors = []
    
    # Check file type
    if allowed_types:
        file_type = magic.from_buffer(uploaded_file.read(1024), mime=True)
        uploaded_file.seek(0)
        
        if file_type not in allowed_types:
            errors.append(f'Invalid file type: {file_type}')
    
    # Check file size
    if max_size and uploaded_file.size > max_size:
        errors.append(f'File too large: {uploaded_file.size} bytes')
    
    # Check if file is empty
    if uploaded_file.size == 0:
        errors.append('File is empty')
    
    return errors

@api_view(['POST'])
@parser_classes([MultiPartParser, FormParser])
def upload_with_validation(request):
    """
    Example using file validation utility.
    """
    if 'file' not in request.FILES:
        return Response({
            'error': 'No file provided'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    uploaded_file = request.FILES['file']
    
    # Validate file
    allowed_types = ['image/jpeg', 'image/png', 'application/pdf']
    max_size = 10 * 1024 * 1024  # 10MB
    
    validation_errors = validate_uploaded_file(
        uploaded_file, 
        allowed_types=allowed_types, 
        max_size=max_size
    )
    
    if validation_errors:
        return Response({
            'error': 'File validation failed',
            'details': validation_errors
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # File is valid, process it
    unique_filename = f"validated/{uuid.uuid4()}_{uploaded_file.name}"
    saved_path = default_storage.save(unique_filename, uploaded_file)
    
    return Response({
        'message': 'File uploaded successfully',
        'file_info': {
            'original_name': uploaded_file.name,
            'saved_path': saved_path,
            'url': default_storage.url(saved_path),
            'size': uploaded_file.size
        }
    }, status=status.HTTP_201_CREATED)
```

### Explanation
- **@parser_classes**: Specifies how to parse incoming request data
- **MultiPartParser**: Handles file uploads and form data
- **FormParser**: Handles HTML form data
- **request.FILES**: Dictionary containing uploaded files
- **File validation**: Check file type, size, and format
- **magic library**: Detect actual file MIME type (more secure than extensions)
- **PIL/Pillow**: Process and manipulate images
- **UUID**: Generate unique filenames to prevent conflicts
- **default_storage**: Django's file storage system
- **ContentFile**: Create file objects from content in memory
- **FileResponse**: Return files for download
- **Error handling**: Clean up resources if upload fails
- **Utility functions**: Reusable file validation logic

---

## Custom Response Formats

### Concept
Create custom response formats and handle different content types in FBVs.

### Code Example
```python
from rest_framework.decorators import api_view, renderer_classes
from rest_framework.renderers import JSONRenderer, BrowsableAPIRenderer
from rest_framework.response import Response
from rest_framework import status
from django.http import HttpResponse, JsonResponse
from datetime import datetime
import csv
import json
import xml.etree.ElementTree as ET
from .models import Book
from .serializers import BookSerializer

# Custom JSON response format
@api_view(['GET'])
def books_custom_json_format(request):
    """
    Return books in a custom JSON format.
    """
    books = Book.objects.all()[:10]
    serializer = BookSerializer(books, many=True)
    
    # Custom response structure
    custom_response = {
        'status': 'success',
        'timestamp': datetime.now().isoformat(),
        'api_version': '1.0',
        'data': {
            'books': serializer.data,
            'meta': {
                'total_count': Book.objects.count(),
                'returned_count': len(serializer.data),
                'page_size': 10
            }
        },
        'links': {
            'self': request.build_absolute_uri(),
            'next': request.build_absolute_uri() + '?page=2',
            'documentation': 'https://api.example.com/docs'
        }
    }
    
    return Response(custom_response)

# CSV export
@api_view(['GET'])
def export_books_csv(request):
    """
    Export books as CSV file.
    """
    # Create HTTP response with CSV content type
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment; filename="books.csv"'
    
    # Create CSV writer
    writer = csv.writer(response)
    
    # Write header
    writer.writerow(['ID', 'Title', 'Author', 'ISBN', 'Published Date', 'Pages', 'Price'])
    
    # Write book data
    books = Book.objects.all()
    for book in books:
        writer.writerow([
            book.id,
            book.title,
            book.author,
            book.isbn,
            book.published_date.strftime('%Y-%m-%d'),
            book.pages,
            f'${book.price}'
        ])
    
    return response

# XML response
@api_view(['GET'])
def books_xml_format(request):
    """
    Return books in XML format.
    """
    books = Book.objects.all()[:10]
    
    # Create XML structure
    root = ET.Element('books')
    root.set('timestamp', datetime.now().isoformat())
    root.set('count', str(len(books)))
    
    for book in books:
        book_element = ET.SubElement(root, 'book')
        book_element.set('id', str(book.id))
        
        # Add book fields
        title_elem = ET.SubElement(book_element, 'title')
        title_elem.text = book.title
        
        author_elem = ET.SubElement(book_element, 'author')
        author_elem.text = book.author
        
        isbn_elem = ET.SubElement(book_element, 'isbn')
        isbn_elem.text = book.isbn
        
        price_elem = ET.SubElement(book_element, 'price')
        price_elem.text = str(book.price)
        price_elem.set('currency', 'USD')
        
        pages_elem = ET.SubElement(book_element, 'pages')
        pages_elem.text = str(book.pages)
        
        date_elem = ET.SubElement(book_element, 'published_date')
        date_elem.text = book.published_date.isoformat()
    
    # Convert to string
    xml_string = ET.tostring(root, encoding='unicode')
    
    return HttpResponse(xml_string, content_type='application/xml')

# Content negotiation based on Accept header
@api_view(['GET'])
def books_content_negotiation(request):
    """
    Return different formats based on Accept header.
    """
    books = Book.objects.all()[:5]
    accept_header = request.META.get('HTTP_ACCEPT', 'application/json')
    
    if 'application/xml' in accept_header:
        # Return XML format
        root = ET.Element('books')
        for book in books:
            book_elem = ET.SubElement(root, 'book')
            book_elem.set('id', str(book.id))
            book_elem.set('title', book.title)
            book_elem.set('author', book.author)
        
        xml_string = ET.tostring(root, encoding='unicode')
        return HttpResponse(xml_string, content_type='application/xml')
    
    elif 'text/csv' in accept_header:
        # Return CSV format
        response = HttpResponse(content_type='text/csv')
        writer = csv.writer(response)
        writer.writerow(['Title', 'Author', 'Price'])
        for book in books:
            writer.writerow([book.title, book.author, book.price])
        return response
    
    else:
        # Default to JSON
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)

# Custom error response format
def custom_error_response(error_type, message, details=None, status_code=400):
    """
    Utility function for consistent error responses.
    """
    error_response = {
        'status': 'error',
        'timestamp': datetime.now().isoformat(),
        'error': {
            'type': error_type,
            'message': message,
            'code': status_code
        }
    }
    
    if details:
        error_response['error']['details'] = details
    
    return Response(error_response, status=status_code)

@api_view(['POST'])
def create_book_custom_responses(request):
    """
    Create book with custom success/error response formats.
    """
    serializer = BookSerializer(data=request.data)
    
    if serializer.is_valid():
        book = serializer.save()
        
        # Custom success response
        success_response = {
            'status': 'success',
            'timestamp': datetime.now().isoformat(),
            'message': 'Book created successfully',
            'data': {
                'book': serializer.data,
                'created_at': book.created_at.isoformat(),
                'resource_url': f'/api/books/{book.id}/'
            },
            'meta': {
                'operation': 'create',
                'resource_type': 'book'
            }
        }
        
        return Response(success_response, status=status.HTTP_201_CREATED)
    
    else:
        # Custom error response
        return custom_error_response(
            'validation_error',
            'The provided data is invalid',
            serializer.errors,
            status.HTTP_400_BAD_REQUEST
        )

# JSON Lines format (for streaming large datasets)
@api_view(['GET'])
def books_jsonlines_format(request):
    """
    Return books in JSON Lines format (one JSON object per line).
    """
    response = HttpResponse(content_type='application/x-ndjson')
    response['Content-Disposition'] = 'attachment; filename="books.jsonl"'
    
    books = Book.objects.all()
    
    for book in books:
        book_data = {
            'id': book.id,
            'title': book.title,
            'author': book.author,
            'isbn': book.isbn,
            'price': float(book.price),
            'pages': book.pages,
            'published_date': book.published_date.isoformat()
        }
        response.write(json.dumps(book_data) + '\n')
    
    return response

# HAL (Hypertext Application Language) format
@api_view(['GET'])
def books_hal_format(request):
    """
    Return books in HAL format with hypermedia links.
    """
    books = Book.objects.all()[:5]
    
    hal_response = {
        '_links': {
            'self': {'href': request.build_absolute_uri()},
            'next': {'href': request.build_absolute_uri() + '?page=2'},
            'prev': {'href': request.build_absolute_uri() + '?page=1'}
        },
        'count': len(books),
        'total': Book.objects.count(),
        '_embedded': {
            'books': []
        }
    }
    
    for book in books:
        book_hal = {
            'id': book.id,
            'title': book.title,
            'author': book.author,
            'price': float(book.price),
            '_links': {
                'self': {'href': f'/api/books/{book.id}/'},
                'edit': {'href': f'/api/books/{book.id}/'},
                'delete': {'href': f'/api/books/{book.id}/'}
            }
        }
        hal_response['_embedded']['books'].append(book_hal)
    
    return Response(hal_response)

# Custom response wrapper
class CustomResponseWrapper:
    """
    Utility class to wrap responses in consistent format.
    """
    
    @staticmethod
    def success(data, message="Success", status_code=200, meta=None):
        response_data = {
            'status': 'success',
            'timestamp': datetime.now().isoformat(),
            'message': message,
            'data': data
        }
        
        if meta:
            response_data['meta'] = meta
        
        return Response(response_data, status=status_code)
    
    @staticmethod
    def error(error_type, message, details=None, status_code=400):
        response_data = {
            'status': 'error',
            'timestamp': datetime.now().isoformat(),
            'error': {
                'type': error_type,
                'message': message
            }
        }
        
        if details:
            response_data['error']['details'] = details
        
        return Response(response_data, status=status_code)

@api_view(['GET', 'POST'])
def books_with_wrapper(request):
    """
    Example using the custom response wrapper.
    """
    if request.method == 'GET':
        books = Book.objects.all()[:10]
        serializer = BookSerializer(books, many=True)
        
        return CustomResponseWrapper.success(
            data=serializer.data,
            message="Books retrieved successfully",
            meta={
                'count': len(serializer.data),
                'total': Book.objects.count()
            }
        )
    
    elif request.method == 'POST':
        serializer = BookSerializer(data=request.data)
        
        if serializer.is_valid():
            book = serializer.save()
            return CustomResponseWrapper.success(
                data=serializer.data,
                message="Book created successfully",
                status_code=201
            )
        else:
            return CustomResponseWrapper.error(
                error_type="validation_error",
                message="Invalid data provided",
                details=serializer.errors,
                status_code=400
            )

# Response format based on query parameter
@api_view(['GET'])
def books_format_parameter(request):
    """
    Return different formats based on ?format= parameter.
    """
    books = Book.objects.all()[:10]
    format_type = request.query_params.get('format', 'json').lower()
    
    if format_type == 'csv':
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="books.csv"'
        
        writer = csv.writer(response)
        writer.writerow(['Title', 'Author', 'ISBN', 'Price'])
        for book in books:
            writer.writerow([book.title, book.author, book.isbn, book.price])
        
        return response
    
    elif format_type == 'xml':
        root = ET.Element('books')
        for book in books:
            book_elem = ET.SubElement(root, 'book')
            book_elem.set('id', str(book.id))
            book_elem.set('title', book.title)
            book_elem.set('author', book.author)
        
        xml_string = ET.tostring(root, encoding='unicode')
        return HttpResponse(xml_string, content_type='application/xml')
    
    elif format_type == 'minimal':
        # Minimal response with only essential fields
        minimal_data = [
            {
                'id': book.id,
                'title': book.title,
                'author': book.author
            }
            for book in books
        ]
        return Response(minimal_data)
    
    else:
        # Default JSON format
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
```

### Explanation
- **Custom JSON structure**: Create standardized response formats with metadata
- **CSV export**: Generate downloadable CSV files using Python's csv module
- **XML responses**: Build XML using ElementTree for structured data
- **Content negotiation**: Handle different formats based on HTTP Accept header
- **Custom error formats**: Consistent error response structure
- **JSON Lines**: Stream large datasets efficiently
- **HAL format**: Hypermedia-driven responses with navigation links
- **Response wrappers**: Utility classes for consistent response formatting
- **Format parameters**: Allow clients to specify response format via query params
- **HttpResponse**: Direct HTTP responses for non-JSON content types

---

## Throttling

### Concept
Implement rate limiting to prevent abuse and ensure fair API usage.

### Code Example
```python
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle, BaseThrottle
from rest_framework.response import Response
from rest_framework import status
from django.core.cache import cache
from django.utils import timezone
from datetime import timedelta
import time

# settings.py configuration for throttling
"""
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
        'login': '5/min',
        'books': '60/hour',
        'uploads': '10/hour'
    }
}

CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
"""

# Custom throttle classes
class LoginRateThrottle(UserRateThrottle):
    scope = 'login'

class BookAPIThrottle(UserRateThrottle):
    scope = 'books'

class UploadThrottle(UserRateThrottle):
    scope = 'uploads'

class BurstRateThrottle(BaseThrottle):
    """
    Custom throttle for burst protection.
    Allows 10 requests per minute, then slower rate.
    """
    
    def allow_request(self, request, view):
        self.key = self.get_cache_key(request, view)
        self.history = self.cache.get(self.key, [])
        self.now = time.time()
        
        # Remove old entries
        while self.history and self.history[-1] <= self.now - 60:
            self.history.pop()
        
        # Check if under limit
        if len(self.history) < 10:
            return self.throttle_success()
        
        return self.throttle_failure()
    
    def get_cache_key(self, request, view):
        if request.user.is_authenticated:
            ident = request.user.pk
        else:
            ident = self.get_ident(request)
        
        return f'throttle_burst_{ident}'
    
    def throttle_success(self):
        self.history.insert(0, self.now)
        self.cache.set(self.key, self.history, 60)
        return True
    
    def throttle_failure(self):
        return False
    
    def wait(self):
        if self.history:
            remaining_duration = 60 - (self.now - self.history[-1])
            return remaining_duration
        return None

# Basic throttled endpoints
@api_view(['POST'])
@throttle_classes([LoginRateThrottle])
def login_throttled(request):
    """
    Login endpoint with rate limiting.
    """
    username = request.data.get('username')
    password = request.data.get('password')
    
    if not username or not password:
        return Response({
            'error': 'Username and password required'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Login logic here
    return Response({
        'message': 'Login successful',
        'throttle_info': 'Limited to 5 requests per minute'
    })

@api_view(['GET', 'POST'])
@throttle_classes([BookAPIThrottle])
def books_throttled(request):
    """
    Book API with throttling.
    """
    if request.method == 'GET':
        # Return books logic
        return Response({
            'message': 'Books retrieved',
            'throttle_limit': '60 requests per hour'
        })
    
    elif request.method == 'POST':
        # Create book logic
        return Response({
            'message': 'Book created',
            'throttle_limit': '60 requests per hour'
        }, status=status.HTTP_201_CREATED)

@api_view(['POST'])
@throttle_classes([UploadThrottle])
def upload_throttled(request):
    """
    File upload with throttling.
    """
    if 'file' not in request.FILES:
        return Response({
            'error': 'No file provided'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    return Response({
        'message': 'File uploaded successfully',
        'throttle_limit': '10 uploads per hour'
    }, status=status.HTTP_201_CREATED)

# Multiple throttle classes
@api_view(['GET'])
@throttle_classes([AnonRateThrottle, UserRateThrottle, BurstRateThrottle])
def multi_throttled_endpoint(request):
    """
    Endpoint with multiple throttling rules.
    """
    return Response({
        'message': 'Success',
        'user_authenticated': request.user.is_authenticated,
        'throttle_rules': [
            'Anonymous: 100/day',
            'Authenticated: 1000/day', 
            'Burst protection: 10/minute'
        ]
    })

# Custom throttle checking
def check_custom_throttle(request, identifier, limit, period_seconds):
    """
    Custom throttle checking function.
    """
    cache_key = f'throttle_{identifier}'
    current_time = time.time()
    
    # Get request history
    request_times = cache.get(cache_key, [])
    
    # Remove old requests outside the time window
    cutoff_time = current_time - period_seconds
    request_times = [req_time for req_time in request_times if req_time > cutoff_time]
    
    # Check if limit exceeded
    if len(request_times) >= limit:
        return False, request_times
    
    # Add current request
    request_times.append(current_time)
    cache.set(cache_key, request_times, period_seconds)
    
    return True, request_times

@api_view(['GET'])
def custom_throttled_endpoint(request):
    """
    Endpoint with custom throttle logic.
    """
    # Custom throttle: 5 requests per minute per IP
    client_ip = request.META.get('REMOTE_ADDR')
    allowed, request_history = check_custom_throttle(
        request, 
        f'ip_{client_ip}',
        limit=5,
        period_seconds=60
    )
    
    if not allowed:
        return Response({
            'error': 'Rate limit exceeded',
            'message': 'Maximum 5 requests per minute',
            'requests_made': len(request_history),
            'retry_after': 60
        }, status=status.HTTP_429_TOO_MANY_REQUESTS)
    
    return Response({
        'message': 'Request successful',
        'requests_made': len(request_history),
        'limit': 5,
        'window': '60 seconds'
    })

# Throttle information endpoint
@api_view(['GET'])
@throttle_classes([UserRateThrottle])
def throttle_status(request):
    """
    Get current throttle status for the user.
    """
    # This is a simplified example - actual implementation would need
    # to access throttle internals
    
    if request.user.is_authenticated:
        user_id = request.user.id
        cache_key = f'throttle_user_{user_id}'
    else:
        ip = request.META.get('REMOTE_ADDR')
        cache_key = f'throttle_anon_{ip}'
    
    # Get throttle data from cache
    throttle_data = cache.get(cache_key, [])
    current_time = time.time()
    
    # Count recent requests (last hour)
    recent_requests = [
        req_time for req_time in throttle_data 
        if req_time > current_time - 3600
    ]
    
    return Response({
        'user_type': 'authenticated' if request.user.is_authenticated else 'anonymous',
        'requests_last_hour': len(recent_requests),
        'hourly_limit': 1000 if request.user.is_authenticated else 100,
        'remaining_requests': (1000 if request.user.is_authenticated else 100) - len(recent_requests),
        'reset_time': int(current_time + 3600) if recent_requests else None
    })

# Conditional throttling
@api_view(['GET', 'POST', 'PUT', 'DELETE'])
def conditional_throttling(request):
    """
    Apply different throttling based on HTTP method.
    """
    method = request.method
    client_ip = request.META.get('REMOTE_ADDR')
    
    # Different limits for different methods
    throttle_config = {
        'GET': {'limit': 60, 'period': 60},      # 60 per minute
        'POST': {'limit': 10, 'period': 60},     # 10 per minute
        'PUT': {'limit': 20, 'period': 60},      # 20 per minute
        'DELETE': {'limit': 5, 'period': 60}     # 5 per minute
    }
    
    config = throttle_config.get(method, {'limit': 30, 'period': 60})
    
    allowed, request_history = check_custom_throttle(
        request,
        f'method_{method}_{client_ip}',
        config['limit'],
        config['period']
    )
    
    if not allowed:
        return Response({
            'error': 'Rate limit exceeded',
            'method': method,
            'limit': config['limit'],
            'period': config['period']
        }, status=status.HTTP_429_TOO_MANY_REQUESTS)
    
    return Response({
        'message': f'{method} request successful',
        'method_limits': throttle_config,
        'current_usage': len(request_history)
    })

# Throttle decorator
def throttle_decorator(limit, period_seconds, key_func=None):
    """
    Decorator for custom throttling.
    """
    def decorator(func):
        def wrapper(request, *args, **kwargs):
            if key_func:
                identifier = key_func(request)
            else:
                identifier = request.META.get('REMOTE_ADDR', 'unknown')
            
            allowed, _ = check_custom_throttle(
                request, identifier, limit, period_seconds
            )
            
            if not allowed:
                return Response({
                    'error': 'Rate limit exceeded',
                    'limit': limit,
                    'period': period_seconds
                }, status=status.HTTP_429_TOO_MANY_REQUESTS)
            
            return func(request, *args, **kwargs)
        return wrapper
    return decorator

@api_view(['GET'])
@throttle_decorator(limit=10, period_seconds=300)  # 10 requests per 5 minutes
def decorated_throttled_endpoint(request):
    """
    Endpoint using throttle decorator.
    """
    return Response({
        'message': 'Request successful',
        'throttle': '10 requests per 5 minutes'
    })

# Throttle bypass for staff users
@api_view(['GET'])
def staff_bypass_throttle(request):
    """
    Throttling that bypasses limits for staff users.
    """
    # Staff users bypass throttling
    if request.user.is_staff:
        return Response({
            'message': 'Staff user - no throttling applied',
            'user': request.user.username
        })
    
    # Regular throttling for non-staff
    client_ip = request.META.get('REMOTE_ADDR')
    allowed, request_history = check_custom_throttle(
        request,
        f'regular_{client_ip}',
        limit=20,
        period_seconds=60
    )
    
    if not allowed:
        return Response({
            'error': 'Rate limit exceeded',
            'message': 'Staff users have unlimited access'
        }, status=status.HTTP_429_TOO_MANY_REQUESTS)
    
    return Response({
        'message': 'Regular user request successful',
        'requests_made': len(request_history)
    })
```

### Explanation
- **Throttle classes**: Control request rates based on user type or custom logic
- **UserRateThrottle**: Rate limiting for authenticated users
- **AnonRateThrottle**: Rate limiting for anonymous users
- **Custom throttles**: Implement specific business logic for rate limiting
- **Cache-based throttling**: Use caching to track request history
- **Multiple throttles**: Apply different rate limits simultaneously
- **Conditional throttling**: Different limits based on request method or user type
- **Throttle decorators**: Reusable throttling logic
- **Staff bypass**: Allow privileged users to bypass limits
- **Throttle status**: Provide users with their current usage information

---

## Versioning

### Concept
Implement API versioning to manage changes and maintain backward compatibility.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.urls import path, include
from .models import Book
from .serializers import BookSerializer

# settings.py configuration for versioning
"""
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS': 'rest_framework.versioning.URLPathVersioning',
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2', 'v3'],
    'VERSION_PARAM': 'version'
}
"""

# URL versioning setup
# urls.py
urlpatterns = [
    path('api/v1/', include('myapp.urls_v1')),
    path('api/v2/', include('myapp.urls_v2')),
    path('api/v3/', include('myapp.urls_v3')),
]

# Version-specific serializers
class BookSerializerV1(BookSerializer):
    """Version 1 serializer - original fields"""
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price']

class BookSerializerV2(BookSerializer):
    """Version 2 serializer - added publication info"""
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price', 'published_date', 'pages']

class BookSerializerV3(BookSerializer):
    """Version 3 serializer - all fields plus computed fields"""
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'isbn', 'price', 'published_date', 'pages', 'created_at', 'updated_at']
    
    def to_representation(self, instance):
        data = super().to_representation(instance)
        # Add computed fields in v3
        data['price_category'] = 'expensive' if instance.price > 50 else 'affordable'
        data['is_recent'] = (timezone.now().date() - instance.published_date).days < 365
        return data

# Version-aware views
@api_view(['GET'])
def books_versioned(request):
    """
    Books endpoint that adapts based on API version.
    """
    version = getattr(request, 'version', 'v1')
    books = Book.objects.all()
    
    # Choose serializer based on version
    if version == 'v1':
        serializer_class = BookSerializerV1
        response_format = 'simple'
    elif version == 'v2':
        serializer_class = BookSerializerV2
        response_format = 'standard'
    elif version == 'v3':
        serializer_class = BookSerializerV3
        response_format = 'enhanced'
    else:
        # Default to latest version
        serializer_class = BookSerializerV3
        response_format = 'enhanced'
    
    serializer = serializer_class(books[:10], many=True)
    
    # Version-specific response format
    if version == 'v1':
        return Response(serializer.data)
    
    elif version == 'v2':
        return Response({
            'books': serializer.data,
            'count': len(serializer.data),
            'version': version
        })
    
    else:  # v3 and above
        return Response({
            'data': {
                'books': serializer.data,
                'pagination': {
                    'count': len(serializer.data),
                    'total': Book.objects.count()
                }
            },
            'meta': {
                'version': version,
                'format': response_format,
                'api_features': ['filtering', 'sorting', 'pagination']
            },
            'links': {
                'self': request.build_absolute_uri(),
                'documentation': f'/docs/{version}/'
            }
        })

@api_view(['POST'])
def create_book_versioned(request):
    """
    Create book endpoint with version-specific behavior.
    """
    version = getattr(request, 'version', 'v1')
    
    # Version-specific validation
    if version == 'v1':
        # v1: Only basic fields required
        required_fields = ['title', 'author', 'isbn', 'price']
        serializer_class = BookSerializerV1
    elif version == 'v2':
        # v2: Added publication info requirements
        required_fields = ['title', 'author', 'isbn', 'price', 'published_date']
        serializer_class = BookSerializerV2
    else:
        # v3+: All fields available, smart defaults
        required_fields = ['title', 'author', 'isbn']
        serializer_class = BookSerializerV3
    
    # Check required fields
    missing_fields = [field for field in required_fields if not request.data.get(field)]
    if missing_fields:
        return Response({
            'error': 'Missing required fields',
            'missing_fields': missing_fields,
            'version': version,
            'required_fields': required_fields
        }, status=status.HTTP_400_BAD_REQUEST)
    
    # Version-specific data processing
    data = request.data.copy()
    
    if version == 'v1':
        # v1: Set defaults for fields not supported
        data.setdefault('published_date', '2024-01-01')
        data.setdefault('pages', 200)
    elif version == 'v2':
        # v2: Set default pages if not provided
        data.setdefault('pages', 200)
    # v3+: Accept all fields as provided
    
    serializer = serializer_class(data=data)
    
    if serializer.is_valid():
        book = serializer.save()
        
        # Version-specific response
        if version == 'v1':
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response({
                'message': 'Book created successfully',
                'book': serializer.data,
                'version': version,
                'created_at': book.created_at.isoformat()
            }, status=status.HTTP_201_CREATED)
    
    return Response({
        'error': 'Validation failed',
        'details': serializer.errors,
        'version': version
    }, status=status.HTTP_400_BAD_REQUEST)

# Header-based versioning
@api_view(['GET'])
def books_header_versioning(request):
    """
    Version detection from custom header.
    Header: X-API-Version: v2
    """
    version = request.META.get('HTTP_X_API_VERSION', 'v1')
    
    # Validate version
    allowed_versions = ['v1', 'v2', 'v3']
    if version not in allowed_versions:
        return Response({
            'error': 'Unsupported API version',
            'provided_version': version,
            'supported_versions': allowed_versions
        }, status=status.HTTP_400_BAD_REQUEST)
    
    books = Book.objects.all()[:5]
    
    if version == 'v1':
        # Legacy format
        serializer = BookSerializerV1(books, many=True)
        return Response(serializer.data)
    
    elif version == 'v2':
        # Improved format
        serializer = BookSerializerV2(books, many=True)
        return Response({
            'status': 'success',
            'data': serializer.data,
            'version': version
        })
    
    else:  # v3
        # Latest format
        serializer = BookSerializerV3(books, many=True)
        return Response({
            'status': 'success',
            'data': serializer.data,
            'meta': {
                'version': version,
                'count': len(serializer.data),
                'features': ['enhanced_fields', 'computed_values']
            }
        })

# Query parameter versioning
@api_view(['GET'])
def books_query_versioning(request):
    """
    Version detection from query parameter.
    URL: /api/books/?version=v2
    """
    version = request.query_params.get('version', 'v1')
    
    books = Book.objects.all()[:5]
    
    # Version-specific logic
    if version == 'v1':
        serializer = BookSerializerV1(books, many=True)
        response_data = serializer.data
    elif version == 'v2':
        serializer = BookSerializerV2(books, many=True)
        response_data = {
            'books': serializer.data,
            'version_info': {
                'version': version,
                'deprecated': False,
                'sunset_date': None
            }
        }
    elif version == 'v3':
        serializer = BookSerializerV3(books, many=True)
        response_data = {
            'data': {
                'books': serializer.data
            },
            'meta': {
                'version': version,
                'current': True,
                'features': ['all_fields', 'computed_values', 'enhanced_metadata']
            }
        }
    else:
        return Response({
            'error': 'Invalid version',
            'supported_versions': ['v1', 'v2', 'v3']
        }, status=status.HTTP_400_BAD_REQUEST)
    
    return Response(response_data)

# Deprecation warnings
@api_view(['GET'])
def deprecated_endpoint(request):
    """
    Endpoint with deprecation warning.
    """
    version = getattr(request, 'version', 'v1')
    
    # Add deprecation headers
    response = Response({
        'message': 'This endpoint is deprecated',
        'data': 'some legacy data',
        'migration_info': {
            'deprecated_in': 'v2',
            'sunset_date': '2024-12-31',
            'replacement': '/api/v3/books/',
            'migration_guide': 'https://api.example.com/migration-guide'
        }
    })
    
    # Add deprecation headers
    response['Warning'] = '299 - "Deprecated API"'
    response['Sunset'] = 'Tue, 31 Dec 2024 23:59:59 GMT'
    response['Link'] = '</api/v3/books/>; rel="successor-version"'
    
    return response

# Version compatibility check
def check_version_compatibility(request, min_version='v1', max_version='v3'):
    """
    Utility function to check version compatibility.
    """
    version = getattr(request, 'version', 'v1')
    
    # Convert version strings to numbers for comparison
    version_map = {'v1': 1, 'v2': 2, 'v3': 3}
    
    current_version_num = version_map.get(version, 1)
    min_version_num = version_map.get(min_version, 1)
    max_version_num = version_map.get(max_version, 3)
    
    if current_version_num < min_version_num:
        return False, f'Minimum required version: {min_version}'
    
    if current_version_num > max_version_num:
        return False, f'Maximum supported version: {max_version}'
    
    return True, 'Version compatible'

@api_view(['POST'])
def version_gated_feature(request):
    """
    Feature that requires specific version range.
    """
    compatible, message = check_version_compatibility(
        request, 
        min_version='v2', 
        max_version='v3'
    )
    
    if not compatible:
        return Response({
            'error': 'Version not supported for this feature',
            'message': message,
            'current_version': getattr(request, 'version', 'v1'),
            'supported_range': 'v2 - v3'
        }, status=status.HTTP_400_BAD_REQUEST)
    
    return Response({
        'message': 'Feature available',
        'version': getattr(request, 'version', 'v1'),
        'feature': 'advanced_book_creation'
    })

# API version discovery
@api_view(['GET'])
def api_versions(request):
    """
    Endpoint to discover available API versions.
    """
    versions_info = {
        'v1': {
            'status': 'deprecated',
            'sunset_date': '2024-12-31',
            'features': ['basic_crud'],
            'documentation': '/docs/v1/'
        },
        'v2': {
            'status': 'supported',
            'sunset_date': None,
            'features': ['basic_crud', 'enhanced_fields'],
            'documentation': '/docs/v2/'
        },
        'v3': {
            'status': 'current',
            'sunset_date': None,
            'features': ['basic_crud', 'enhanced_fields', 'computed_values', 'advanced_metadata'],
            'documentation': '/docs/v3/'
        }
    }
    
    return Response({
        'current_version': 'v3',
        'default_version': 'v1',
        'supported_versions': list(versions_info.keys()),
        'versions': versions_info,
        'versioning_scheme': 'URL path versioning',
        'migration_guide': 'https://api.example.com/migration'
    })
```

### Explanation
- **URL Path Versioning**: Include version in URL (/api/v1/, /api/v2/)
- **Header Versioning**: Use custom headers (X-API-Version)
- **Query Parameter Versioning**: Version in query string (?version=v2)
- **Version-specific serializers**: Different data formats for each version
- **Backward compatibility**: Maintain older versions while adding new features
- **Deprecation warnings**: Inform clients about sunset timelines
- **Version gating**: Restrict features to specific version ranges
- **API discovery**: Help clients understand available versions
- **Migration support**: Provide guidance for version upgrades

---

## Content Negotiation

### Concept
Handle different content types and formats based on client preferences and Accept headers.

### Code Example
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status
from django.http import HttpResponse
import json
import csv
import xml.etree.ElementTree as ET
from datetime import datetime
from .models import Book
from .serializers import BookSerializer

@api_view(['GET'])
def books_content_negotiation(request):
    """
    Return different content types based on Accept header.
    """
    books = Book.objects.all()[:10]
    accept_header = request.META.get('HTTP_ACCEPT', 'application/json')
    
    # JSON format (default)
    if 'application/json' in accept_header or '*/*' in accept_header:
        serializer = BookSerializer(books, many=True)
        return Response({
            'format': 'json',
            'content_type': 'application/json',
            'books': serializer.data
        })
    
    # XML format
    elif 'application/xml' in accept_header or 'text/xml' in accept_header:
        root = ET.Element('response')
        root.set('format', 'xml')
        root.set('timestamp', datetime.now().isoformat())
        
        books_element = ET.SubElement(root, 'books')
        for book in books:
            book_element = ET.SubElement(books_element, 'book')
            book_element.set('id', str(book.id))
            
            title_elem = ET.SubElement(book_element, 'title')
            title_elem.text = book.title
            
            author_elem = ET.SubElement(book_element, 'author')
            author_elem.text = book.author
            
            price_elem = ET.SubElement(book_element, 'price')
            price_elem.text = str(book.price)
        
        xml_string = ET.tostring(root, encoding='unicode')
        return HttpResponse(xml_string, content_type='application/xml')
    
    # CSV format
    elif 'text/csv' in accept_header:
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment; filename="books.csv"'
        
        writer = csv.writer(response)
        writer.writerow(['ID', 'Title', 'Author', 'ISBN', 'Price'])
        
        for book in books:
            writer.writerow([book.id, book.title, book.author, book.isbn, book.price])
        
        return response
    
    # Plain text format
    elif 'text/plain' in accept_header:
        text_content = "BOOKS LIST\n"
        text_content += "=" * 50 + "\n\n"
        
        for book in books:
            text_content += f"ID: {book.id}\n"
            text_content += f"Title: {book.title}\n"
            text_content += f"Author: {book.author}\n"
            text_content += f"Price: ${book.price}\n"
            text_content += "-" * 30 + "\n"
        
        return HttpResponse(text_content, content_type='text/plain')
    
    # HTML format (for browsers)
    elif 'text/html' in accept_header:
        html_content = """
        <!DOCTYPE html>
        <html>
        <head>
            <title>Books API</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 40px; }
                .book { border: 1px solid #ddd; margin: 10px 0; padding: 15px; }
                .title { font-weight: bold; font-size: 18px; }
                .author { color: #666; }
                .price { color: #008000; font-weight: bold; }
            </style>
        </head>
        <body>
            <h1>Books API Response</h1>
            <p>Content Type: text/html</p>
        """
        
        for book in books:
            html_content += f"""
            <div class="book">
                <div class="title">{book.title}</div>
                <div class="author">by {book.author}</div>
                <div class="price">${book.price}</div>
            </div>
            """
        
        html_content += "</body></html>"
        return HttpResponse(html_content, content_type='text/html')
    
    # Unsupported format
    else:
        return Response({
            'error': 'Unsupported content type',
            'requested': accept_header,
            'supported': [
                'application/json',
                'application/xml',
                'text/csv',
                'text/plain',
                'text/html'
            ]
        }, status=status.HTTP_406_NOT_ACCEPTABLE)

# Language negotiation
@api_view(['GET'])
def books_language_negotiation(request):
    """
    Return content in different languages based on Accept-Language header.
    """
    accept_language = request.META.get('HTTP_ACCEPT_LANGUAGE', 'en')
    
    # Parse language preference
    if 'es' in accept_language:
        language = 'es'
        messages = {
            'title': 'Lista de Libros',
            'total': 'Total de libros',
            'format': 'formato'
        }
    elif 'fr' in accept_language:
        language = 'fr'
        messages = {
            'title': 'Liste des Livres',
            'total': 'Total des livres',
            'format': 'format'
        }
    else:
        language = 'en'
        messages = {
            'title': 'Books List',
            'total': 'Total books',
            'format': 'format'
        }
    
    books = Book.objects.all()[:5]
    serializer = BookSerializer(books, many=True)
    
    return Response({
        'language': language,
        'title': messages['title'],
        'data': serializer.data,
        'meta': {
            f"{messages['total']}": Book.objects.count(),
            f"{messages['format']}": 'JSON'
        }
    })

# Custom content type handling
@api_view(['POST'])
def create_book_content_types(request):
    """
    Handle different input content types.
    """
    content_type = request.content_type
    
    if content_type == 'application/json':
        # Standard JSON input
        data = request.data
    
    elif content_type == 'application/x-www-form-urlencoded':
        # Form data input
        data = {
            'title': request.POST.get('title'),
            'author': request.POST.get('author'),
            'isbn': request.POST.get('isbn'),
            'price': request.POST.get('price'),
            'pages': request.POST.get('pages'),
            'published_date': request.POST.get('published_date')
        }
        # Remove None values
        data = {k: v for k, v in data.items() if v is not None}
    
    elif content_type.startswith('multipart/form-data'):
        # Multipart form data
        data = {
            'title': request.data.get('title'),
            'author': request.data.get('author'),
            'isbn': request.data.get('isbn'),
            'price': request.data.get('price'),
            'pages': request.data.get('pages'),
            'published_date': request.data.get('published_date')
        }
    
    elif content_type == 'text/plain':
        # Custom plain text format parsing
        try:
            lines = request.body.decode('utf-8').strip().split('\n')
            data = {}
            for line in lines:
                if ':' in line:
                    key, value = line.split(':', 1)
                    data[key.strip().lower()] = value.strip()
        except Exception:
            return Response({
                'error': 'Invalid plain text format',
                'expected_format': 'key: value (one per line)'
            }, status=status.HTTP_400_BAD_REQUEST)
    
    else:
        return Response({
            'error': 'Unsupported content type',
            'provided': content_type,
            'supported': [
                'application/json',
                'application/x-www-form-urlencoded',
                'multipart/form-data',
                'text/plain'
            ]
        }, status=status.HTTP_415_UNSUPPORTED_MEDIA_TYPE)
    
    # Validate and create book
    serializer = BookSerializer(data=data)
    if serializer.is_valid():
        book = serializer.save()
        return Response({
            'message': 'Book created successfully',
            'input_content_type': content_type,
            'book': serializer.data
        }, status=status.HTTP_201_CREATED)
    
    return Response({
        'error': 'Validation failed',
        'input_content_type': content_type,
        'details': serializer.errors
    }, status=status.HTTP_400_BAD_REQUEST)

# Quality factor handling
@api_view(['GET'])
def books_quality_negotiation(request):
    """
    Handle Accept header with quality factors (q-values).
    Example: Accept: application/json;q=0.9, application/xml;q=0.8
    """
    accept_header = request.META.get('HTTP_ACCEPT', 'application/json')
    
    # Parse Accept header and quality factors
    accepted_types = []
    for item in accept_header.split(','):
        item = item.strip()
        if ';q=' in item:
            media_type, quality = item.split(';q=')
            quality = float(quality)
        else:
            media_type = item
            quality = 1.0
        accepted_types.append((media_type.strip(), quality))
    
    # Sort by quality factor (highest first)
    accepted_types.sort(key=lambda x: x[1], reverse=True)
    
    books = Book.objects.all()[:3]
    
    # Choose format based on highest quality preference
    for media_type, quality in accepted_types:
        if media_type in ['application/json', '*/*']:
            serializer = BookSerializer(books, many=True)
            return Response({
                'format': 'json',
                'chosen_type': media_type,
                'quality': quality,
                'books': serializer.data
            })
        
        elif media_type in ['application/xml', 'text/xml']:
            # Return XML format
            root = ET.Element('books')
            root.set('format', 'xml')
            root.set('chosen_type', media_type)
            root.set('quality', str(quality))
            
            for book in books:
                book_elem = ET.SubElement(root, 'book')
                book_elem.set('id', str(book.id))
                book_elem.set('title', book.title)
                book_elem.set('author', book.author)
            
            xml_string = ET.tostring(root, encoding='unicode')
            return HttpResponse(xml_string, content_type='application/xml')
        
        elif media_type == 'text/csv':
            response = HttpResponse(content_type='text/csv')
            writer = csv.writer(response)
            writer.writerow(['Title', 'Author'])
            for book in books:
                writer.writerow([book.title, book.author])
            return response
    
    # No acceptable format found
    return Response({
        'error': 'No acceptable format found',
        'requested_types': [f"{mt} (q={q})" for mt, q in accepted_types],
        'available_types': ['application/json', 'application/xml', 'text/csv']
    }, status=status.HTTP_406_NOT_ACCEPTABLE)

# Character encoding negotiation
@api_view(['GET'])
def books_encoding_negotiation(request):
    """
    Handle character encoding preferences.
    """
    accept_charset = request.META.get('HTTP_ACCEPT_CHARSET', 'utf-8')
    
    # Determine preferred encoding
    if 'utf-8' in accept_charset.lower():
        encoding = 'utf-8'
    elif 'iso-8859-1' in accept_charset.lower():
        encoding = 'iso-8859-1'
    else:
        encoding = 'utf-8'  # Default
    
    books = Book.objects.all()[:3]
    
    # Create response with special characters to test encoding
    response_data = {
        'encoding': encoding,
        'message': 'Café, naïve, résumé, piñata',  # Test characters
        'books': []
    }
    
    for book in books:
        response_data['books'].append({
            'title': book.title,
            'author': book.author,
            'price': f"${book.price}"
        })
    
    # Create response with specified encoding
    response_content = json.dumps(response_data, ensure_ascii=(encoding != 'utf-8'))
    response = HttpResponse(
        response_content,
        content_type=f'application/json; charset={encoding}'
    )
    
    return response

# Compression negotiation
@api_view(['GET'])
def books_compression_negotiation(request):
    """
    Handle compression preferences (Accept-Encoding).
    """
    accept_encoding = request.META.get('HTTP_ACCEPT_ENCODING', '')
    
    books = Book.objects.all()
    serializer = BookSerializer(books, many=True)
    
    response_data = {
        'compression_supported': 'gzip' in accept_encoding or 'deflate' in accept_encoding,
        'accept_encoding': accept_encoding,
        'books': serializer.data
    }
    
    response = Response(response_data)
    
    # Add compression hint header
    if 'gzip' in accept_encoding:
        response['Content-Encoding-Hint'] = 'gzip recommended'
    
    return response

# Conditional content negotiation
@api_view(['GET'])
def books_conditional_negotiation(request):
    """
    Conditional content based on multiple factors.
    """
    # Check user agent for mobile detection
    user_agent = request.META.get('HTTP_USER_AGENT', '').lower()
    is_mobile = any(mobile in user_agent for mobile in ['mobile', 'android', 'iphone'])
    
    # Check if client supports modern features
    accept_header = request.META.get('HTTP_ACCEPT', '')
    supports_json = 'application/json' in accept_header
    
    books = Book.objects.all()[:10 if not is_mobile else 5]  # Fewer books for mobile
    
    if is_mobile and supports_json:
        # Mobile-optimized JSON response
        mobile_data = []
        for book in books:
            mobile_data.append({
                'id': book.id,
                'title': book.title[:30] + '...' if len(book.title) > 30 else book.title,
                'author': book.author,
                'price': float(book.price)
            })
        
        return Response({
            'format': 'mobile_optimized',
            'device_type': 'mobile',
            'books': mobile_data
        })
    
    elif supports_json:
        # Full desktop JSON response
        serializer = BookSerializer(books, many=True)
        return Response({
            'format': 'desktop_full',
            'device_type': 'desktop',
            'books': serializer.data
        })
    
    else:
        # Fallback to simple format
        simple_data = [f"{book.title} by {book.author}" for book in books]
        return HttpResponse('\n'.join(simple_data), content_type='text/plain')

# Content negotiation utility
def negotiate_content_type(accept_header, available_types):
    """
    Utility function for content type negotiation.
    """
    if not accept_header:
        return available_types[0] if available_types else None
    
    # Parse Accept header
    accepted = []
    for item in accept_header.split(','):
        item = item.strip()
        if ';q=' in item:
            media_type, quality = item.split(';q=')
            quality = float(quality)
        else:
            media_type = item
            quality = 1.0
        accepted.append((media_type.strip(), quality))
    
    # Sort by quality
    accepted.sort(key=lambda x: x[1], reverse=True)
    
    # Find best match
    for media_type, quality in accepted:
        if media_type in available_types or media_type == '*/*':
            return available_types[0] if media_type == '*/*' else media_type
    
    return None

@api_view(['GET'])
def books_utility_negotiation(request):
    """
    Example using content negotiation utility.
    """
    accept_header = request.META.get('HTTP_ACCEPT', '')
    available_types = ['application/json', 'application/xml', 'text/csv']
    
    chosen_type = negotiate_content_type(accept_header, available_types)
    
    if not chosen_type:
        return Response({
            'error': 'No acceptable content type',
            'available': available_types
        }, status=status.HTTP_406_NOT_ACCEPTABLE)
    
    books = Book.objects.all()[:5]
    
    if chosen_type == 'application/json':
        serializer = BookSerializer(books, many=True)
        return Response({
            'content_type': chosen_type,
            'books': serializer.data
        })
    
    elif chosen_type == 'application/xml':
        root = ET.Element('response')
        root.set('content_type', chosen_type)
        
        for book in books:
            book_elem = ET.SubElement(root, 'book')
            book_elem.set('title', book.title)
            book_elem.set('author', book.author)
        
        xml_string = ET.tostring(root, encoding='unicode')
        return HttpResponse(xml_string, content_type='application/xml')
    
    elif chosen_type == 'text/csv':
        response = HttpResponse(content_type='text/csv')
        writer = csv.writer(response)
        writer.writerow(['Title', 'Author'])
        for book in books:
            writer.writerow([book.title, book.author])
        return response
    
    return Response({'error': 'Internal error'}, status=500)
```

### Explanation
- **Accept header parsing**: Parse client preferences with quality factors
- **Multiple content types**: Support JSON, XML, CSV, HTML, and plain text
- **Language negotiation**: Return content in different languages
- **Input content types**: Handle various input formats (JSON, form data, plain text)
- **Quality factors**: Respect q-values for content type preference
- **Character encoding**: Handle different character encodings
- **Compression hints**: Provide guidance for response compression
- **Conditional negotiation**: Adapt content based on device type and capabilities
- **Utility functions**: Reusable content negotiation logic
- **Error handling**: Proper 406 Not Acceptable responses for unsupported types

---

## Conclusion

This comprehensive cheatsheet covers all essential aspects of working with Django REST Framework using Function-Based Views. Each section provides practical, working examples that you can adapt to your specific needs. The examples progress from basic concepts to advanced implementations, ensuring you have the knowledge to build robust, scalable APIs.

Key takeaways:
- **FBVs offer flexibility** for custom logic and complex business requirements
- **Proper error handling** is crucial for maintainable APIs
- **Authentication and permissions** secure your endpoints appropriately
- **Pagination and filtering** improve performance with large datasets
- **Versioning and content negotiation** ensure long-term API sustainability
- **File uploads and custom responses** extend API functionality beyond basic CRUD

Remember to always validate input data, handle errors gracefully, and follow REST principles for consistent API design. The utility functions and patterns shown throughout this cheatsheet can be reused across different projects to maintain consistency and reduce development time.
        
