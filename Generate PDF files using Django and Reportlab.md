In this article we will build an application that generate PDF files using Django and Reportlab

So first let's use the Pseudo code:

- create a virtual environment
- install dependencies
- create django project
- create an application
- create a view function that generate PDF files
- use our view to URLConf

### Create a virtual environment

use virtualenv to create a virtual environment
to use it just type  
`virtualenv django-reportalb-env`  
`cd django-reportlab-env`  
`source bin/activate`

### install dependencies

`python -m pip install --upgrade pip`  
`python -m pip install Django reportlab django-environ`

### create django project

`django-admin startproject generate_pdf_files`  
`cd generate_pdf_files/`

### test our django project

`python manage.py runserver`  
The server should be running on : http://localhost:8000/

### use .env file

`touch generate_pdf_files/.env`  
the .env file should have sensitive variables that can not been seen by others, so for now we'll move SECRET_KEY from settings.py to .env file

add this code at the top of settings.py

```python
import environ
```

and replace the secret key with this line

```python
SECRET_KEY = env("SECRET_KEY")
```

restart the server and it should be working just fine

### create an application

`python manage.py startapp pdf_files`  
`python manage.py migrate`  
`python manage.py runserver`

add pdf_files to INSTALLED_APPS

```python
INSTALLED_APPS = [
    ...
    'pdf_files',
]
```

### create a view function that generate PDF files

put this code inside pdf_files/views.py

```python
import io
from django.http import FileResponse
from reportlab.pdfgen import canvas
from reportlab.lib.units import inch
from reportlab.lib.pagesizes import letter


def employees_pdf(request):

    buffer = io.BytesIO()
    p = canvas.Canvas(buffer, pagesize=letter, bottomup=0)

    textobject = p.beginText()
    textobject.setTextOrigin(inch, inch)
    textobject.setFont("Helvetica", 13)

    employee_list = [
        {"name": "John", "age": 23},
        {"name": "Robert", "age": 25},
        {"name": "Jim", "age": 37},
        {"name": "Ron", "age": 37},
        {"name": "Rafael", "age": 37},
        {"name": "Jamal", "age": 37},
    ]

    for line in employee_list:
        l = 'name={0} - age={1}'.format(line["name"], line["age"])
        textobject.textLine(l)

    p.drawText(textobject)
    p.showPage()
    p.save()
    buffer.seek(0)

    return FileResponse(buffer, as_attachment=True, filename='employees_pdf.pdf')
```

### open generate_pdf_files/urls.py

replace the code with this

```python
from django.contrib import admin
from django.urls import path
from pdf_files import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('pdf/', views.employees_pdf()),
]
```
