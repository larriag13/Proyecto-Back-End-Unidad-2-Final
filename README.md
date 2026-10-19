# Guía de Desarrollo Backend Paso a Paso: Mundo Figuras
### Construcción de una Aplicación Web Completa con Django y Bootstrap 5

Esta guía técnica explica paso a paso cómo construir desde cero la plataforma de comercio electrónico y gestión de inventario **Mundo Figuras**. En cada etapa se detallan los comandos, el código fuente necesario y los fundamentos técnicos que justifican cada decisión de arquitectura y desarrollo en Django.

---

## 📑 Tabla de Contenidos

1. [Arquitectura y Fundamentos de Django](#1-arquitectura-y-fundamentos-de-django)
2. [Paso 1: Entorno de Desarrollo y Dependencias](#paso-1-entorno-de-desarrollo-y-dependencias)
3. [Paso 2: Inicialización del Proyecto Django](#paso-2-inicialización-del-proyecto-django)
4. [Paso 3: Arquitectura Modular y Creación de Aplicaciones (`apps/`)](#paso-3-arquitectura-modular-y-creación-de-aplicaciones-apps)
5. [Paso 4: Configuración Central (`config/settings.py`)](#paso-4-configuración-central-configsettingspy)
6. [Paso 5: Capa de Datos — Django ORM y Modelos Relacionales](#paso-5-capa-de-datos--django-orm-y-modelos-relacionales)
7. [Paso 6: Panel de Administración de Django (`admin.py`)](#paso-6-panel-de-administración-de-django-adminpy)
8. [Paso 7: Formularios y Operaciones CRUD en el Catálogo](#paso-7-formularios-y-operaciones-crud-en-el-catálogo)
9. [Paso 8: Manejo de Estado y Sesiones HTTP (Carrito de Compras)](#paso-8-manejo-de-estado-y-sesiones-http-carrito-de-compras)
10. [Paso 9: Autenticación de Clientes y Control de Acceso](#paso-9-autenticación-de-clientes-y-control-de-acceso)
11. [Paso 10: Capa de Presentación con DTL y Bootstrap 5](#paso-10-capa-de-presentación-con-dtl-y-bootstrap-5)
12. [Paso 11: Enrutamiento Principal con Namespaces (`config/urls.py`)](#paso-11-enrutamiento-principal-con-namespaces-configurlspy)
13. [Paso 12: Carga Inicial de Datos y Verificación del Sistema](#paso-12-carga-inicial-de-datos-y-verificación-del-sistema)
14. [Glosario de Conceptos Técnicos](#glosario-de-conceptos-técnicos)

---

## 1. Arquitectura y Fundamentos de Django

Django estructura el desarrollo web bajo el patrón **MTV (Model - Template - View)**:

```text
[Navegador Web] 
       │ 1. Petición HTTP (Request)
       ▼
[Enrutador: urls.py] 
       │ 2. Asocia la URL a una función o clase
       ▼
[Vista: views.py] ◄────────► [Modelo / ORM: models.py] ◄──► [Base de Datos]
       │                          (Consultas y lógica de datos)
       │ 3. Inyecta diccionario de contexto
       ▼
[Plantilla: templates (.html)] 
       │ 4. Renderiza HTML con DTL
       ▼
[Navegador Web] ◄── 5. Respuesta HTTP (Response)
```

* **Modelo (`models.py`):** Define las tablas de la base de datos como clases de Python. El ORM de Django traduce estas clases a sentencias SQL.
* **Vista (`views.py`):** Contiene la lógica del negocio. Recibe el objeto `request`, interactúa con los modelos y retorna un objeto `response` (renderizando una plantilla HTML o enviando una redirección).
* **Plantilla (`templates`):** Archivos HTML procesados por el motor de plantillas de Django (*Django Template Language - DTL*), permitiendo inyectar variables, condicionales y bucles.
* **Enrutador (`urls.py`):** Asocia rutas HTTP (endpoints) con las vistas correspondientes.

---

## Paso 1: Entorno de Desarrollo y Dependencias

### Fundamento Técnico: Aislamiento del Entorno
En Python, instalar librerías de forma global puede provocar conflictos de versiones entre distintos proyectos. Un entorno virtual crea un directorio aislado con un intérprete de Python y una carpeta de dependencias propia (`site-packages`), garantizando portabilidad y estabilidad.

### Comandos de Ejecución

1. Crea la carpeta principal y accede a ella:
   ```bash
   mkdir tienda_figuras
   cd tienda_figuras
   ```

2. Crea el entorno virtual `.venv`:
   * **Windows:**
     ```powershell
     python -m venv .venv
     .venv\Scripts\activate
     ```
   * **macOS / Linux:**
     ```bash
     python3 -m venv .venv
     source .venv/bin/activate
     ```

3. Instala las dependencias del proyecto:
   ```bash
   pip install "django>=5.1" pillow python-dotenv
   ```
   * **`django`:** El framework web backend.
   * **`pillow`:** Biblioteca de procesamiento de imágenes requerida obligatoriamente por Django para utilizar campos `ImageField`.
   * **`python-dotenv`:** Permite desacoplar variables de configuración sensibles (`SECRET_KEY`, credenciales de BD) en un archivo `.env`.

4. Genera el archivo `requirements.txt`:
   ```bash
   pip freeze > requirements.txt
   ```

---

## Paso 2: Inicialización del Proyecto Django

Crea el esqueleto inicial del proyecto ejecutando:
```bash
django-admin startproject config .
```

> **Detalle técnico:** El punto final (`.`) indica a Django que inicialice la configuración dentro del directorio actual, evitando una doble carpeta redundante (`tienda_figuras/tienda_figuras`).

### Estructura generada:
* **`manage.py`:** Interfaz de línea de comandos para administrar el proyecto (migraciones, servidor local, superusuarios).
* **`config/settings.py`:** Archivo central de configuración técnica.
* **`config/urls.py`:** Tabla maestra de rutas de la aplicación.
* **`config/wsgi.py` / `asgi.py`:** Interfaces estándar para despliegue en servidores web de producción.

---

## Paso 3: Arquitectura Modular y Creación de Aplicaciones (`apps/`)

### Fundamento Técnico: ¿Por qué usar un directorio contenedor `apps/`?
En proyectos profesionales, colocar todas las aplicaciones en la raíz genera desorden visual y mezcla archivos de infraestructura (`manage.py`, `.env`, `requirements.txt`) con la lógica del negocio. Usar la carpeta `apps/` aporta:
1. **Directorio Raíz Limpio:** Facilita la navegación del repositorio.
2. **Espacio de Nombres Seguro (Namespacing):** Evita colisiones de nombres entre apps locales y paquetes de terceros o módulos estándar de Python (por ejemplo, tener una app llamada `auth` o `requests`).
3. **Modularidad:** Cada módulo encapsula un dominio concreto del sistema.

### Estructura de Aplicaciones del Proyecto
1. **`apps/core`:** Páginas generales (portada institucional, página de información de la tienda).
2. **`apps/usuarios`:** Registro, login, logout y perfiles de clientes.
3. **`apps/catalogo`:** Modelos de figuras y franquicias, panel de inventario y operaciones CRUD.
4. **`apps/pedidos`:** Gestión de compras y almacenamiento de figuras seleccionadas en la sesión del cliente.

### Comandos de Creación

* **En Windows (PowerShell):**
  ```powershell
  mkdir apps, static/js, static/img, media/figuras
  mkdir templates/core, templates/usuarios, templates/catalogo, templates/pedidos
  New-Item apps/__init__.py
  python manage.py startapp core apps/core
  python manage.py startapp usuarios apps/usuarios
  python manage.py startapp catalogo apps/catalogo
  python manage.py startapp pedidos apps/pedidos
  ```

* **En macOS / Linux:**
  ```bash
  mkdir -p apps static/js static/img media/figuras templates/core templates/usuarios templates/catalogo templates/pedidos
  touch apps/__init__.py
  python3 manage.py startapp core apps/core
  python3 manage.py startapp usuarios apps/usuarios
  python3 manage.py startapp catalogo apps/catalogo
  python3 manage.py startapp pedidos apps/pedidos
  ```

### Configuración del nombre de cada aplicación en `apps.py`
Dado que las aplicaciones están dentro del subdirectorio `apps/`, se debe actualizar el atributo `name` en cada archivo de configuración:

* **`apps/core/apps.py`**:
  ```python
  from django.apps import AppConfig

  class CoreConfig(AppConfig):
      default_auto_field = 'django.db.models.BigAutoField'
      name = 'apps.core'
      verbose_name = 'Núcleo del Sitio'
  ```

* **`apps/usuarios/apps.py`**:
  ```python
  from django.apps import AppConfig

  class UsuariosConfig(AppConfig):
      default_auto_field = 'django.db.models.BigAutoField'
      name = 'apps.usuarios'
      verbose_name = 'Usuarios y Clientes'
  ```

* **`apps/catalogo/apps.py`**:
  ```python
  from django.apps import AppConfig

  class CatalogoConfig(AppConfig):
      default_auto_field = 'django.db.models.BigAutoField'
      name = 'apps.catalogo'
      verbose_name = 'Catálogo de Figuras'
  ```

* **`apps/pedidos/apps.py`**:
  ```python
  from django.apps import AppConfig

  class PedidosConfig(AppConfig):
      default_auto_field = 'django.db.models.BigAutoField'
      name = 'apps.pedidos'
      verbose_name = 'Pedidos y Carrito de Compras'
  ```

---

## Paso 4: Configuración Central (`config/settings.py`)

Abre `config/settings.py` y aplica las siguientes configuraciones:

```python
from pathlib import Path
from django.contrib.messages import constants as messages

BASE_DIR = Path(__file__).resolve().parent.parent

# 1. Registro de Aplicaciones
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Módulos del Proyecto
    'apps.core',
    'apps.usuarios',
    'apps.catalogo',
    'apps.pedidos',
]

# 2. Configuración de Plantillas y Context Processors
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'], # Directorio global de plantillas
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.media',
                # Inyección global de la cantidad de artículos en el carrito:
                'apps.pedidos.context_processors.carrito_context',
            ],
        },
    },
]

# 3. Idioma y Zona Horaria
LANGUAGE_CODE = 'es-cl'
TIME_ZONE = 'America/Santiago'
USE_I18N = True
USE_TZ = True

# 4. Archivos Estáticos (Diseño) y Archivos Multimedia (Subidas)
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
STATIC_ROOT = BASE_DIR / 'staticfiles'

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# 5. URLs de Control de Autenticación
LOGIN_URL = 'usuarios:login'
LOGIN_REDIRECT_URL = 'catalogo:lista'
LOGOUT_REDIRECT_URL = 'core:inicio'

# 6. Mapeo de Mensajes Flash a clases de Bootstrap 5
MESSAGE_TAGS = {
    messages.ERROR: 'danger',
}
```

### Context Processor para el Carrito
Crea el archivo `apps/pedidos/context_processors.py`:
```python
def carrito_context(request):
    """Permite acceder al número de figuras en el carrito desde cualquier plantilla."""
    carrito = request.session.get('carrito', {})
    total_items = sum(item.get('cantidad', 0) for item in carrito.values())
    return {
        'carrito_total_items': total_items
    }
```

---

## Paso 5: Capa de Datos — Django ORM y Modelos Relacionales

### Fundamento Técnico: Django ORM y Claves Foráneas
El ORM traduce clases de Python en tablas de base de datos relacionales. En este modelo:
* **`Categoria`:** Franquicia o universo temático.
* **`Figura`:** Entidad principal que almacena los atributos del coleccionable. La relación `ForeignKey` vincula muchas figuras a una categoría.
* **`on_delete=models.PROTECT`:** Regla de integridad referencial que impide borrar una categoría si existen figuras vinculadas a ella, evitando registros huérfanos.

Edita `apps/catalogo/models.py`:

```python
from django.db import models
from django.utils.text import slugify
from django.core.validators import MinValueValidator


class Categoria(models.Model):
    nombre = models.CharField(max_length=100, unique=True, verbose_name="Categoría o Franquicia")
    slug = models.SlugField(max_length=120, unique=True, blank=True, help_text="URL amigable")
    descripcion = models.TextField(blank=True, verbose_name="Descripción")
    activo = models.BooleanField(default=True, verbose_name="¿Activo?")
    creado_en = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name_plural = "Categorías"
        ordering = ['nombre']

    def __str__(self):
        return self.nombre

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.nombre)
        super().save(*args, **kwargs)


class Figura(models.Model):
    nombre = models.CharField(max_length=200, verbose_name="Nombre de la Figura")
    slug = models.SlugField(max_length=220, unique=True, blank=True)
    categoria = models.ForeignKey(Categoria, on_delete=models.PROTECT, related_name="figuras", verbose_name="Categoría")
    fabricante = models.CharField(max_length=100, verbose_name="Fabricante / Marca")
    escala = models.CharField(max_length=50, blank=True, verbose_name="Escala o Altura")
    descripcion = models.TextField(verbose_name="Descripción y Materiales")
    precio = models.DecimalField(max_digits=10, decimal_places=0, validators=[MinValueValidator(0)], verbose_name="Precio en CLP")
    stock = models.PositiveIntegerField(default=0, verbose_name="Stock disponible")
    imagen = models.ImageField(upload_to="figuras/", blank=True, null=True, verbose_name="Fotografía")
    destacado = models.BooleanField(default=False, verbose_name="¿Destacado?")
    activo = models.BooleanField(default=True, verbose_name="¿Disponible?")
    fecha_creacion = models.DateTimeField(auto_now_add=True)
    fecha_actualizacion = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name_plural = "Figuras Coleccionables"
        ordering = ['-fecha_creacion']

    def __str__(self):
        return f"{self.nombre} ({self.fabricante})"

    @property
    def en_stock(self):
        return self.stock > 0

    @property
    def precio_formateado(self):
        return f"${self.precio:,.0f}".replace(",", ".")

    def save(self, *args, **kwargs):
        if not self.slug:
            base_slug = slugify(self.nombre)
            slug = base_slug
            contador = 1
            while Figura.objects.filter(slug=slug).exclude(pk=self.pk).exists():
                slug = f"{base_slug}-{contador}"
                contador += 1
            self.slug = slug
        super().save(*args, **kwargs)
```

### Ejecutar Migraciones
Crea los archivos de migración y aplícalos a la base de datos:
```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Paso 6: Panel de Administración de Django (`admin.py`)

Personaliza el panel de gestión en `apps/catalogo/admin.py`:

```python
from django.contrib import admin
from .models import Categoria, Figura

admin.site.site_header = "Mundo Figuras - Panel de Administración"
admin.site.site_title = "Mundo Figuras Admin"
admin.site.index_title = "Gestión del Catálogo e Inventario"


@admin.register(Categoria)
class CategoriaAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'slug', 'activo', 'creado_en')
    search_fields = ('nombre', 'descripcion')
    prepopulated_fields = {'slug': ('nombre',)}
    list_editable = ('activo',)


@admin.register(Figura)
class FiguraAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'categoria', 'fabricante', 'precio_formateado', 'stock', 'activo', 'destacado')
    list_filter = ('categoria', 'fabricante', 'activo', 'destacado')
    search_fields = ('nombre', 'fabricante', 'descripcion')
    list_editable = ('stock', 'activo', 'destacado')
    prepopulated_fields = {'slug': ('nombre',)}
    list_per_page = 15
```

Crea el usuario administrador:
```bash
python manage.py createsuperuser
```

---

## Paso 7: Formularios y Operaciones CRUD en el Catálogo

### Fundamento Técnico: Uso de `ModelForm` y Seguridad
`forms.ModelForm` inspecciona el modelo `Figura`, deduce los tipos de campos HTML y añade validaciones automáticas contra inyección de datos no válidos.

Crea `apps/catalogo/forms.py`:
```python
from django import forms
from .models import Figura


class FiguraForm(forms.ModelForm):
    class Meta:
        model = Figura
        fields = [
            'nombre', 'categoria', 'fabricante', 'escala', 
            'descripcion', 'precio', 'stock', 'imagen', 'destacado', 'activo'
        ]
        widgets = {
            'nombre': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Ej. Figura Son Goku 18cm'}),
            'categoria': forms.Select(attrs={'class': 'form-select'}),
            'fabricante': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Ej. Bandai Spirits'}),
            'escala': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Ej. Escala 1/7 (24 cm)'}),
            'descripcion': forms.Textarea(attrs={'class': 'form-control', 'rows': 4}),
            'precio': forms.NumberInput(attrs={'class': 'form-control', 'min': 0}),
            'stock': forms.NumberInput(attrs={'class': 'form-control', 'min': 0}),
            'imagen': forms.ClearableFileInput(attrs={'class': 'form-control'}),
            'destacado': forms.CheckboxInput(attrs={'class': 'form-check-input'}),
            'activo': forms.CheckboxInput(attrs={'class': 'form-check-input'}),
        }

    def clean_precio(self):
        precio = self.cleaned_data.get('precio')
        if precio is not None and precio <= 0:
            raise forms.ValidationError("El precio debe ser superior a 0 CLP.")
        return precio
```

### Vistas en `apps/catalogo/views.py`:
Implementa las operaciones de consulta, creación, edición y borrado:

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required, user_passes_test
from django.contrib import messages
from django.core.paginator import Paginator
from django.db.models import Q
from .models import Figura, Categoria
from .forms import FiguraForm


def es_administrador(user):
    return user.is_authenticated and (user.is_staff or user.is_superuser)


def figura_lista(request):
    query = request.GET.get('q', '').strip()
    categoria_slug = request.GET.get('categoria', '')
    orden = request.GET.get('orden', '-fecha_creacion')

    figuras = Figura.objects.filter(activo=True).select_related('categoria')

    if categoria_slug:
        figuras = figuras.filter(categoria__slug=categoria_slug)

    if query:
        figuras = figuras.filter(
            Q(nombre__icontains=query) |
            Q(fabricante__icontains=query) |
            Q(categoria__nombre__icontains=query)
        )

    ordenes_validos = {
        'precio_asc': 'precio',
        'precio_desc': '-precio',
        'nombre_asc': 'nombre',
        'recientes': '-fecha_creacion',
    }
    figuras = figuras.order_by(ordenes_validos.get(orden, '-fecha_creacion'))

    paginador = Paginator(figuras, 6)
    pagina = paginador.get_page(request.GET.get('page'))
    categorias = Categoria.objects.filter(activo=True)

    return render(request, 'catalogo/figura_lista.html', {
        'figuras': pagina,
        'categorias': categorias,
        'categoria_seleccionada': categoria_slug,
        'query_actual': query,
        'orden_actual': orden,
        'total_resultados': figuras.count(),
    })


def figura_detalle(request, slug):
    figura = get_object_or_404(Figura.objects.select_related('categoria'), slug=slug, activo=True)
    relacionadas = Figura.objects.filter(categoria=figura.categoria, activo=True).exclude(pk=figura.pk)[:3]
    return render(request, 'catalogo/figura_detalle.html', {'figura': figura, 'relacionadas': relacionadas})


@login_required
@user_passes_test(es_administrador, login_url='catalogo:lista')
def gestion_figuras(request):
    query = request.GET.get('q', '').strip()
    figuras = Figura.objects.all().select_related('categoria')
    if query:
        figuras = figuras.filter(Q(nombre__icontains=query) | Q(fabricante__icontains=query))
    paginador = Paginator(figuras.order_by('-fecha_creacion'), 10)
    pagina = paginador.get_page(request.GET.get('page'))
    return render(request, 'catalogo/gestion_figuras.html', {'figuras': pagina, 'query': query})


@login_required
@user_passes_test(es_administrador, login_url='catalogo:lista')
def figura_crear(request):
    if request.method == 'POST':
        form = FiguraForm(request.POST, request.FILES)
        if form.is_valid():
            figura = form.save()
            messages.success(request, f"¡Figura '{figura.nombre}' creada exitosamente!")
            return redirect('catalogo:gestion')
    else:
        form = FiguraForm()
    return render(request, 'catalogo/figura_form.html', {'form': form, 'titulo_accion': 'Ingresar Nueva Figura', 'boton_texto': 'Guardar'})


@login_required
@user_passes_test(es_administrador, login_url='catalogo:lista')
def figura_editar(request, id):
    figura = get_object_or_404(Figura, id=id)
    if request.method == 'POST':
        form = FiguraForm(request.POST, request.FILES, instance=figura)
        if form.is_valid():
            form.save()
            messages.success(request, f"Figura '{figura.nombre}' actualizada.")
            return redirect('catalogo:gestion')
    else:
        form = FiguraForm(instance=figura)
    return render(request, 'catalogo/figura_form.html', {'form': form, 'figura': figura, 'titulo_accion': 'Editar Figura', 'boton_texto': 'Actualizar'})


@login_required
@user_passes_test(es_administrador, login_url='catalogo:lista')
def figura_eliminar(request, id):
    figura = get_object_or_404(Figura, id=id)
    if request.method == 'POST':
        nombre = figura.nombre
        figura.delete()
        messages.success(request, f"Figura '{nombre}' eliminada.")
        return redirect('catalogo:gestion')
    return render(request, 'catalogo/figura_confirmar_eliminar.html', {'figura': figura})
```

### URLs en `apps/catalogo/urls.py`:
```python
from django.urls import path
from . import views

app_name = 'catalogo'

urlpatterns = [
    path('', views.figura_lista, name='lista'),
    path('figura/<slug:slug>/', views.figura_detalle, name='detalle'),
    path('gestion/', views.gestion_figuras, name='gestion'),
    path('gestion/nueva/', views.figura_crear, name='crear'),
    path('gestion/editar/<int:id>/', views.figura_editar, name='editar'),
    path('gestion/eliminar/<int:id>/', views.figura_eliminar, name='eliminar'),
]
```

---

## Paso 8: Manejo de Estado y Sesiones HTTP (Carrito de Compras)

### Fundamento Técnico: Sesiones en Arquitecturas Web
HTTP es un protocolo sin estado (*stateless*). Para que el servidor reconozca los artículos seleccionados por un cliente sin obligarlo a autenticarse previamente ni escribir en la base de datos, Django provee `request.session`, respaldado por una cookie encriptada en el navegador.

En `apps/pedidos/views.py`:

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from django.contrib.auth.decorators import login_required
from apps.catalogo.models import Figura


def _obtener_carrito(request):
    return request.session.setdefault('carrito', {})


def carrito_ver(request):
    carrito = _obtener_carrito(request)
    subtotal = sum(item['subtotal'] for item in carrito.values())
    iva = int(subtotal * 0.19)
    total = subtotal + iva
    return render(request, 'pedidos/carrito.html', {
        'items_carrito': carrito.values(),
        'subtotal': subtotal,
        'iva': iva,
        'total': total,
        'esta_vacio': len(carrito) == 0,
    })


def carrito_agregar(request, id):
    figura = get_object_or_404(Figura, id=id, activo=True)
    carrito = _obtener_carrito(request)
    str_id = str(figura.id)

    cantidad_actual = carrito.get(str_id, {}).get('cantidad', 0)
    nueva_cantidad = cantidad_actual + 1

    if nueva_cantidad > figura.stock:
        messages.warning(request, f"Límite de stock alcanzado ({figura.stock} unidades) para '{figura.nombre}'.")
        return redirect('pedidos:ver')

    imagen_url = figura.imagen.url if figura.imagen else ''
    carrito[str_id] = {
        'id': figura.id,
        'nombre': figura.nombre,
        'slug': figura.slug,
        'fabricante': figura.fabricante,
        'precio': int(figura.precio),
        'cantidad': nueva_cantidad,
        'subtotal': int(figura.precio) * nueva_cantidad,
        'imagen_url': imagen_url,
    }

    # Notifica a Django que la sesión fue modificada
    request.session.modified = True
    messages.success(request, f"'{figura.nombre}' añadida al carrito.")
    return redirect(request.META.get('HTTP_REFERER') or 'pedidos:ver')


def carrito_restar(request, id):
    carrito = _obtener_carrito(request)
    str_id = str(id)
    if str_id in carrito:
        carrito[str_id]['cantidad'] -= 1
        if carrito[str_id]['cantidad'] <= 0:
            del carrito[str_id]
        else:
            carrito[str_id]['subtotal'] = carrito[str_id]['precio'] * carrito[str_id]['cantidad']
        request.session.modified = True
    return redirect('pedidos:ver')


def carrito_eliminar(request, id):
    carrito = _obtener_carrito(request)
    str_id = str(id)
    if str_id in carrito:
        del carrito[str_id]
        request.session.modified = True
        messages.warning(request, "Figura eliminada del carrito.")
    return redirect('pedidos:ver')


def carrito_limpiar(request):
    if 'carrito' in request.session:
        del request.session['carrito']
        request.session.modified = True
    return redirect('pedidos:ver')


@login_required
def checkout_simulado(request):
    carrito = _obtener_carrito(request)
    if not carrito:
        return redirect('catalogo:lista')

    subtotal = sum(item['subtotal'] for item in carrito.values())
    iva = int(subtotal * 0.19)
    total = subtotal + iva
    items_comprados = list(carrito.values())

    del request.session['carrito']
    request.session.modified = True

    return render(request, 'pedidos/checkout_exito.html', {
        'items': items_comprados,
        'subtotal': subtotal,
        'iva': iva,
        'total': total,
        'orden_id': f"FIG-{request.user.id}-2026",
    })
```

Configura `apps/pedidos/urls.py`:
```python
from django.urls import path
from . import views

app_name = 'pedidos'

urlpatterns = [
    path('carrito/', views.carrito_ver, name='ver'),
    path('carrito/agregar/<int:id>/', views.carrito_agregar, name='agregar'),
    path('carrito/restar/<int:id>/', views.carrito_restar, name='restar'),
    path('carrito/eliminar/<int:id>/', views.carrito_eliminar, name='eliminar'),
    path('carrito/limpiar/', views.carrito_limpiar, name='limpiar'),
    path('checkout/', views.checkout_simulado, name='checkout'),
]
```

---

## Paso 9: Autenticación de Clientes y Control de Acceso

### Fundamento Técnico: Seguridad y Hashing de Contraseñas
Django utiliza el algoritmo criptográfico PBKDF2 con sal (SHA-256) para proteger contraseñas. Nunca almacena contraseñas en texto claro.

Crea `apps/usuarios/forms.py`:
```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm


class RegistroForm(UserCreationForm):
    first_name = forms.CharField(max_length=50, required=True, label="Nombres", widget=forms.TextInput(attrs={'class': 'form-control'}))
    last_name = forms.CharField(max_length=50, required=True, label="Apellidos", widget=forms.TextInput(attrs={'class': 'form-control'}))
    email = forms.EmailField(required=True, label="Correo Electrónico", widget=forms.EmailInput(attrs={'class': 'form-control'}))

    class Meta(UserCreationForm.Meta):
        model = User
        fields = ('username', 'first_name', 'last_name', 'email')

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['username'].widget.attrs.update({'class': 'form-control'})

    def clean_email(self):
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("Este correo ya está registrado.")
        return email


class LoginForm(AuthenticationForm):
    username = forms.CharField(widget=forms.TextInput(attrs={'class': 'form-control'}))
    password = forms.CharField(widget=forms.PasswordInput(attrs={'class': 'form-control'}))
```

En `apps/usuarios/views.py`:
```python
from django.shortcuts import render, redirect
from django.contrib.auth import login, logout
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .forms import RegistroForm, LoginForm


def registro_usuario(request):
    if request.user.is_authenticated:
        return redirect('catalogo:lista')
    if request.method == 'POST':
        form = RegistroForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            messages.success(request, f"¡Bienvenido {user.first_name}! Cuenta creada.")
            return redirect('catalogo:lista')
    else:
        form = RegistroForm()
    return render(request, 'usuarios/registro.html', {'form': form})


def login_usuario(request):
    if request.user.is_authenticated:
        return redirect('catalogo:lista')
    if request.method == 'POST':
        form = LoginForm(request, data=request.POST)
        if form.is_valid():
            login(request, form.get_user())
            messages.success(request, "Sesión iniciada correctamente.")
            return redirect(request.POST.get('next') or 'catalogo:lista')
    else:
        form = LoginForm()
    return render(request, 'usuarios/login.html', {'form': form, 'next': request.GET.get('next', 'catalogo:lista')})


def logout_usuario(request):
    logout(request)
    messages.info(request, "Has cerrado sesión.")
    return redirect('core:inicio')


@login_required
def perfil_usuario(request):
    return render(request, 'usuarios/perfil.html', {'usuario': request.user})
```

Configura `apps/usuarios/urls.py`:
```python
from django.urls import path
from . import views

app_name = 'usuarios'

urlpatterns = [
    path('registro/', views.registro_usuario, name='registro'),
    path('login/', views.login_usuario, name='login'),
    path('logout/', views.logout_usuario, name='logout'),
    path('perfil/', views.perfil_usuario, name='perfil'),
]
```

---

## Paso 10: Capa de Presentación con DTL y Bootstrap 5

### Plantilla Maestra (`templates/base.html`)
Centraliza el diseño visual, la barra de navegación, el contenedor de mensajes y el pie de página:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Mundo Figuras{% endblock %}</title>
    <!-- Bootstrap 5.3 CSS & Icons -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">
</head>
<body class="bg-light d-flex flex-column min-vh-100">

    <!-- Barra de Navegación -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark sticky-top shadow-sm py-3">
        <div class="container">
            <a class="navbar-brand fw-bold text-warning fs-4" href="{% url 'core:inicio' %}">
                <i class="bi bi-stars me-2"></i>Mundo Figuras
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#menu">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="menu">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item"><a class="nav-link" href="{% url 'core:inicio' %}">Inicio</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'catalogo:lista' %}">Catálogo</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'core:nosotros' %}">Quiénes Somos</a></li>
                    {% if user.is_staff %}
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle text-info" href="#" data-bs-toggle="dropdown">Administración</a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="{% url 'catalogo:gestion' %}">Inventario</a></li>
                            <li><a class="dropdown-item" href="{% url 'catalogo:crear' %}">Nueva Figura</a></li>
                        </ul>
                    </li>
                    {% endif %}
                </ul>
                <div class="d-flex align-items-center gap-3">
                    <a href="{% url 'pedidos:ver' %}" class="btn btn-outline-warning position-relative">
                        <i class="bi bi-cart3"></i>
                        {% if carrito_total_items > 0 %}
                            <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger">
                                {{ carrito_total_items }}
                            </span>
                        {% endif %}
                    </a>
                    {% if user.is_authenticated %}
                        <a href="{% url 'usuarios:perfil' %}" class="btn btn-secondary btn-sm">{{ user.username }}</a>
                        <a href="{% url 'usuarios:logout' %}" class="btn btn-outline-danger btn-sm">Salir</a>
                    {% else %}
                        <a href="{% url 'usuarios:login' %}" class="btn btn-outline-light btn-sm">Ingresar</a>
                        <a href="{% url 'usuarios:registro' %}" class="btn btn-warning btn-sm fw-semibold">Registrarse</a>
                    {% endif %}
                </div>
            </div>
        </div>
    </nav>

    <!-- Contenedor de Mensajes Flash de Django -->
    <div class="container mt-3">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        {% endif %}
    </div>

    <!-- Contenido Principal -->
    <main class="container my-4 flex-grow-1">
        {% block content %}{% endblock %}
    </main>

    <!-- Pie de Página -->
    <footer class="bg-dark text-white py-4 mt-auto">
        <div class="container text-center">
            <small>&copy; 2026 Mundo Figuras - Envíos a todo Chile.</small>
        </div>
    </footer>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### Vistas de la Aplicación General (`apps/core/views.py`)

La aplicación `core` gestiona las páginas institucionales e informativas del sitio: la portada principal (`inicio`), que consulta las figuras destacadas y categorías del catálogo para presentarlas al usuario, y la página informativa de la tienda (`nosotros`).

Crea o edita `apps/core/views.py`:

```python
from django.shortcuts import render
from apps.catalogo.models import Figura, Categoria


def inicio(request):
    """
    Portada de Mundo Figuras con figuras destacadas, categorías principales
    y promociones vigentes.
    """
    destacadas = Figura.objects.filter(
        activo=True,
        destacado=True
    ).select_related('categoria')[:4]

    categorias = Categoria.objects.filter(activo=True)[:6]

    return render(request, 'core/index.html', {
        'destacadas': destacadas,
        'categorias': categorias,
        'total_figuras': Figura.objects.filter(activo=True).count(),
        'total_categorias': Categoria.objects.filter(activo=True).count(),
    })


def nosotros(request):
    """
    Información institucional sobre la tienda Mundo Figuras,
    garantía de originalidad, envíos a todo Chile y atención al coleccionista.
    """
    return render(request, 'core/nosotros.html')
```

### Enrutamiento de la Aplicación General (`apps/core/urls.py`)

Crea `apps/core/urls.py` para mapear las rutas de la portada e información institucional bajo el namespace `'core'`:

```python
from django.urls import path
from . import views

app_name = 'core'

urlpatterns = [
    path('', views.inicio, name='inicio'),
    path('nosotros/', views.nosotros, name='nosotros'),
]
```

### Plantillas de la Aplicación Core

#### Portada de la Tienda (`templates/core/index.html`)

Hereda de `base.html` y renderiza el banner de bienvenida, beneficios, cuadrícula de franquicias y figuras destacadas:

```html
{% extends 'base.html' %}

{% block title %}Inicio | Mundo Figuras - Coleccionables & Figuras de Acción{% endblock %}

{% block content %}
<!-- Banner Principal de la Tienda -->
<div class="p-5 mb-5 rounded-4 bg-dark text-white shadow">
    <div class="row align-items-center">
        <div class="col-lg-8">
            <span class="badge bg-warning text-dark px-3 py-2 fw-bold mb-3 fs-6">
                <i class="bi bi-fire me-1"></i>¡Nuevas Figuras Disponibles!
            </span>
            <h1 class="display-4 fw-bold mb-3">Tu Pasión por el Coleccionismo Comienza Aquí</h1>
            <p class="lead text-light opacity-75 mb-4">
                Encuentra las figuras de acción, estatuas y coleccionables más codiciados de anime, videojuegos, cómics y cine. Piezas 100% auténticas de fabricantes oficiales.
            </p>
            <div class="d-flex flex-wrap gap-2">
                <a href="{% url 'catalogo:lista' %}" class="btn btn-warning btn-lg fw-bold px-4">
                    <i class="bi bi-cart3 me-2"></i>Ver Catálogo Completo
                </a>
                <a href="{% url 'core:nosotros' %}" class="btn btn-outline-light btn-lg px-4">
                    <i class="bi bi-info-circle me-2"></i>Sobre Nosotros
                </a>
            </div>
        </div>
        <div class="col-lg-4 text-center d-none d-lg-block">
            <i class="bi bi-trophy text-warning opacity-75" style="font-size: 8rem;"></i>
        </div>
    </div>
</div>

<!-- Beneficios de Comprar en Mundo Figuras -->
<div class="row g-3 mb-5 text-center">
    <div class="col-md-4">
        <div class="p-4 bg-white rounded-3 shadow-sm border h-100">
            <i class="bi bi-shield-check text-success display-5 mb-2"></i>
            <h5 class="fw-bold mb-1">100% Originales</h5>
            <p class="text-muted small mb-0">Trabajamos únicamente con distribuidores autorizados y sellos oficiales.</p>
        </div>
    </div>
    <div class="col-md-4">
        <div class="p-4 bg-white rounded-3 shadow-sm border h-100">
            <i class="bi bi-box-seam text-primary display-5 mb-2"></i>
            <h5 class="fw-bold mb-1">Embalaje Protector</h5>
            <p class="text-muted small mb-0">Cuidamos cada caja con plástico burbuja reforzado para coleccionistas exigentes.</p>
        </div>
    </div>
    <div class="col-md-4">
        <div class="p-4 bg-white rounded-3 shadow-sm border h-100">
            <i class="bi bi-truck text-warning display-5 mb-2"></i>
            <h5 class="fw-bold mb-1">Envíos Rápidos</h5>
            <p class="text-muted small mb-0">Despachos asegurados a todo el territorio nacional con seguimiento en línea.</p>
        </div>
    </div>
</div>

<!-- Categorías / Franquicias -->
{% if categorias %}
<div class="mb-5">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2 class="h4 fw-bold mb-0"><i class="bi bi-collection-fill text-warning me-2"></i>Franquicias y Líneas de Colección</h2>
        <a href="{% url 'catalogo:lista' %}" class="text-decoration-none fw-semibold">Ver todas &rarr;</a>
    </div>
    <div class="row g-3">
        {% for cat in categorias %}
        <div class="col-6 col-md-4 col-lg-3">
            <a href="{% url 'catalogo:lista' %}?categoria={{ cat.slug }}" class="text-decoration-none">
                <div class="card border text-center p-3 h-100 shadow-sm bg-white">
                    <i class="bi bi-bookmark-star-fill text-warning fs-3 mb-2"></i>
                    <h6 class="fw-bold text-dark mb-1">{{ cat.nombre }}</h6>
                    <span class="badge bg-light text-secondary border">{{ cat.figuras.count }} figuras</span>
                </div>
            </a>
        </div>
        {% endfor %}
    </div>
</div>
{% endif %}

<!-- Figuras Destacadas -->
<div class="mb-5">
    <div class="d-flex justify-content-between align-items-center mb-3">
        <h2 class="h4 fw-bold mb-0"><i class="bi bi-star-fill text-warning me-2"></i>Piezas Destacadas</h2>
        <a href="{% url 'catalogo:lista' %}" class="btn btn-outline-dark btn-sm">Ver todo el catálogo</a>
    </div>

    <div class="row g-4">
        {% for figura in destacadas %}
        <div class="col-md-6 col-lg-3">
            <div class="card h-100 shadow-sm border">
                <div class="bg-white text-center p-3 border-bottom position-relative" style="height: 230px; display: flex; align-items: center; justify-content: center; overflow: hidden;">
                    {% if figura.imagen %}
                        <img src="{{ figura.imagen.url }}" alt="{{ figura.nombre }}" class="img-fluid" style="max-height: 200px; object-fit: contain;">
                    {% else %}
                        <i class="bi bi-image text-muted display-4"></i>
                    {% endif %}
                    {% if figura.destacado %}
                        <span class="position-absolute top-0 start-0 m-2 badge bg-warning text-dark fw-bold">
                            <i class="bi bi-star-fill me-1"></i>Destacado
                        </span>
                    {% endif %}
                </div>

                <div class="card-body d-flex flex-column p-3">
                    <div class="d-flex justify-content-between align-items-center small text-muted mb-1">
                        <span>{{ figura.categoria.nombre }}</span>
                        <span class="fw-semibold text-secondary">{{ figura.fabricante }}</span>
                    </div>

                    <h5 class="card-title fs-6 fw-bold mb-2 text-truncate" title="{{ figura.nombre }}">
                        {{ figura.nombre }}
                    </h5>

                    <p class="card-text text-muted small flex-grow-1" style="display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;">
                        {{ figura.descripcion }}
                    </p>

                    <div class="d-flex justify-content-between align-items-center pt-2 mt-2 border-top">
                        <span class="fs-5 fw-bold text-dark">{{ figura.precio_formateado }}</span>
                        {% if figura.en_stock %}
                            <span class="badge bg-success-subtle text-success border border-success-subtle">
                                Stock: {{ figura.stock }}
                            </span>
                        {% else %}
                            <span class="badge bg-danger-subtle text-danger border border-danger-subtle">
                                Agotado
                            </span>
                        {% endif %}
                    </div>

                    <div class="d-grid gap-2 mt-3">
                        <a href="{% url 'catalogo:detalle' figura.slug %}" class="btn btn-outline-dark btn-sm">
                            <i class="bi bi-eye me-1"></i>Ver Ficha
                        </a>
                        {% if figura.en_stock %}
                            <a href="{% url 'pedidos:agregar' figura.id %}" class="btn btn-warning btn-sm fw-semibold">
                                <i class="bi bi-cart-plus me-1"></i>Agregar al Carrito
                            </a>
                        {% endif %}
                    </div>
                </div>
            </div>
        </div>
        {% empty %}
        <div class="col-12 text-center py-5">
            <div class="card p-5 bg-white shadow-sm border">
                <i class="bi bi-box-seam display-3 text-muted mb-3"></i>
                <h5 class="text-secondary">Pronto tendremos nuevas figuras disponibles.</h5>
                <p class="text-muted small">Vuelve a consultar nuestro catálogo o comunícate con nosotros.</p>
            </div>
        </div>
        {% endfor %}
    </div>
</div>
{% endblock %}
```

#### Página Institucional Quiénes Somos (`templates/core/nosotros.html`)

```html
{% extends 'base.html' %}

{% block title %}Quiénes Somos | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-lg-10">
        <div class="card border-0 shadow-sm rounded-4 overflow-hidden mb-5">
            <div class="bg-dark text-white p-5 text-center">
                <i class="bi bi-stars text-warning display-3 mb-2"></i>
                <h1 class="fw-bold mb-2">Sobre Mundo Figuras</h1>
                <p class="lead text-light opacity-75 mb-0">Pasión, dedicación y compromiso con los coleccionistas en Chile.</p>
            </div>

            <div class="card-body p-4 p-md-5">
                <div class="row gy-4 mb-5">
                    <div class="col-md-6">
                        <h4 class="fw-bold text-dark mb-3"><i class="bi bi-heart-fill text-danger me-2"></i>Nuestra Historia</h4>
                        <p class="text-secondary lh-base">
                            Mundo Figuras nació como un proyecto de fanáticos y coleccionistas que buscaban acercar al público chileno figuras de acción, estatuas de resina y piezas exclusivas con total garantía de autenticidad.
                        </p>
                        <p class="text-secondary lh-base">
                            Conocemos el valor sentimental y la dedicación que implica completar una colección, por lo que cada figura que enviamos es tratada con el máximo cuidado y respeto.
                        </p>
                    </div>

                    <div class="col-md-6">
                        <h4 class="fw-bold text-dark mb-3"><i class="bi bi-patch-check-fill text-warning me-2"></i>Nuestros Compromisos</h4>
                        <ul class="list-group list-group-flush">
                            <li class="list-group-item px-0">
                                <i class="bi bi-check-circle-fill text-success me-2"></i>
                                <strong>Sellos Oficiales:</strong> Solo importamos productos originales con licencia directa.
                            </li>
                            <li class="list-group-item px-0">
                                <i class="bi bi-check-circle-fill text-success me-2"></i>
                                <strong>Embalaje a Prueba de Golpes:</strong> Cajas reforzadas con protección interior para evitar cualquier daño en las esquinas.
                            </li>
                            <li class="list-group-item px-0">
                                <i class="bi bi-check-circle-fill text-success me-2"></i>
                                <strong>Soporte Personalizado:</strong> Te asesoramos ante dudas sobre preventas, escalas y marcas fabricantes.
                            </li>
                        </ul>
                    </div>
                </div>

                <hr class="my-4">

                <!-- Canales de Atención -->
                <div class="row g-4 text-center">
                    <div class="col-md-4">
                        <div class="p-3 bg-light rounded-3 border">
                            <i class="bi bi-shop text-primary fs-2 mb-2"></i>
                            <h6 class="fw-bold">Tienda y Retiro</h6>
                            <p class="small text-muted mb-0">Santiago Centro, Región Metropolitana</p>
                        </div>
                    </div>
                    <div class="col-md-4">
                        <div class="p-3 bg-light rounded-3 border">
                            <i class="bi bi-envelope-at text-info fs-2 mb-2"></i>
                            <h6 class="fw-bold">Correo Electrónico</h6>
                            <p class="small text-muted mb-0">contacto@mundofiguras.cl</p>
                        </div>
                    </div>
                    <div class="col-md-4">
                        <div class="p-3 bg-light rounded-3 border">
                            <i class="bi bi-whatsapp text-success fs-2 mb-2"></i>
                            <h6 class="fw-bold">Atención WhatsApp</h6>
                            <p class="small text-muted mb-0">+56 9 8765 4321</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Plantillas del Catálogo e Inventario (`templates/catalogo/`)

#### Listado y Búsqueda de Figuras (`templates/catalogo/figura_lista.html`)

Esta plantilla implementa el catálogo público con filtros por franquicia en barra lateral, buscador por texto, selector de ordenamiento, cuadrícula de productos con badges de stock y paginación Bootstrap:

```html
{% extends 'base.html' %}

{% block title %}Catálogo de Figuras Coleccionables | Mundo Figuras{% endblock %}

{% block content %}
<div class="row g-4">
    <!-- Filtros Laterales (Franquicias / Categorías) -->
    <div class="col-lg-3">
        <div class="card border shadow-sm rounded-3 mb-4 bg-white">
            <div class="card-header bg-dark text-white py-3 fw-bold">
                <i class="bi bi-funnel-fill text-warning me-2"></i>Franquicias
            </div>
            <div class="list-group list-group-flush">
                <a href="{% url 'catalogo:lista' %}{% if query_actual %}?q={{ query_actual }}{% endif %}" 
                   class="list-group-item list-group-item-action d-flex justify-content-between align-items-center {% if not categoria_seleccionada %}active bg-warning text-dark border-warning fw-bold{% endif %}">
                    <span>Todas las Figuras</span>
                    <span class="badge rounded-pill bg-light text-dark border">{{ total_resultados }}</span>
                </a>
                {% for cat in categorias %}
                <a href="{% url 'catalogo:lista' %}?categoria={{ cat.slug }}{% if query_actual %}&q={{ query_actual }}{% endif %}" 
                   class="list-group-item list-group-item-action d-flex justify-content-between align-items-center {% if categoria_seleccionada == cat.slug %}active bg-warning text-dark border-warning fw-bold{% endif %}">
                    <span>{{ cat.nombre }}</span>
                    <span class="badge rounded-pill bg-light text-dark border">
                        {{ cat.figuras.count }}
                    </span>
                </a>
                {% endfor %}
            </div>
        </div>

        {% if user.is_staff %}
        <div class="card bg-dark text-white p-3 rounded-3 shadow-sm mb-4">
            <h6 class="fw-bold text-warning mb-2"><i class="bi bi-shield-lock me-1"></i>Personal de Tienda</h6>
            <p class="small text-secondary mb-3">Puedes agregar nuevas piezas o editar el stock desde aquí.</p>
            <a href="{% url 'catalogo:crear' %}" class="btn btn-warning btn-sm w-100 fw-bold">
                <i class="bi bi-plus-circle me-1"></i>Ingresar Nueva Figura
            </a>
        </div>
        {% endif %}
    </div>

    <!-- Contenedor Principal: Búsqueda, Orden y Grilla -->
    <div class="col-lg-9">
        <!-- Barra de Búsqueda y Filtros de Orden -->
        <div class="card border shadow-sm p-3 mb-4 rounded-3 bg-white">
            <form method="get" class="row g-2 align-items-center">
                {% if categoria_seleccionada %}
                    <input type="hidden" name="categoria" value="{{ categoria_seleccionada }}">
                {% endif %}
                <div class="col-md-7">
                    <div class="input-group">
                        <span class="input-group-text bg-white border-end-0"><i class="bi bi-search text-muted"></i></span>
                        <input type="text" name="q" class="form-control border-start-0" placeholder="Buscar por personaje, anime, marca..." value="{{ query_actual }}">
                    </div>
                </div>
                <div class="col-md-3">
                    <select name="orden" class="form-select" onchange="this.form.submit()">
                        <option value="recientes" {% if orden_actual == 'recientes' %}selected{% endif %}>Más recientes</option>
                        <option value="precio_asc" {% if orden_actual == 'precio_asc' %}selected{% endif %}>Precio: Menor a Mayor</option>
                        <option value="precio_desc" {% if orden_actual == 'precio_desc' %}selected{% endif %}>Precio: Mayor a Menor</option>
                        <option value="nombre_asc" {% if orden_actual == 'nombre_asc' %}selected{% endif %}>Nombre: A-Z</option>
                    </select>
                </div>
                <div class="col-md-2 d-grid">
                    <button type="submit" class="btn btn-dark">Buscar</button>
                </div>
            </form>
        </div>

        <!-- Grilla de Figuras -->
        <div class="row g-4 mb-4">
            {% for figura in figuras %}
            <div class="col-md-6 col-xl-4">
                <div class="card h-100 shadow-sm border bg-white">
                    <div class="bg-white text-center p-3 border-bottom position-relative" style="height: 230px; display: flex; align-items: center; justify-content: center; overflow: hidden;">
                        {% if figura.imagen %}
                            <img src="{{ figura.imagen.url }}" alt="{{ figura.nombre }}" class="img-fluid" style="max-height: 200px; object-fit: contain;">
                        {% else %}
                            <i class="bi bi-image text-muted display-4"></i>
                        {% endif %}
                        {% if figura.destacado %}
                            <span class="position-absolute top-0 start-0 m-2 badge bg-warning text-dark fw-bold">
                                <i class="bi bi-star-fill me-1"></i>Destacado
                            </span>
                        {% endif %}
                    </div>
                    <div class="card-body p-3 d-flex flex-column">
                        <div class="d-flex justify-content-between small text-muted mb-1">
                            <span>{{ figura.categoria.nombre }}</span>
                            <span class="fw-semibold text-secondary">{{ figura.fabricante }}</span>
                        </div>
                        <h5 class="card-title fs-6 fw-bold mb-2 text-truncate" title="{{ figura.nombre }}">
                            {{ figura.nombre }}
                        </h5>
                        <p class="card-text text-muted small flex-grow-1" style="display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;">
                            {{ figura.descripcion }}
                        </p>
                        <div class="d-flex justify-content-between align-items-center mt-3 pt-2 border-top">
                            <span class="fs-5 fw-bold text-dark">{{ figura.precio_formateado }}</span>
                            {% if figura.en_stock %}
                                <span class="badge bg-success-subtle text-success border border-success-subtle">
                                    Stock: {{ figura.stock }}
                                </span>
                            {% else %}
                                <span class="badge bg-danger-subtle text-danger border border-danger-subtle">
                                    Agotado
                                </span>
                            {% endif %}
                        </div>
                        <div class="d-grid gap-2 mt-3">
                            <a href="{% url 'catalogo:detalle' figura.slug %}" class="btn btn-outline-dark btn-sm">
                                <i class="bi bi-eye me-1"></i>Ver Ficha
                            </a>
                            {% if figura.en_stock %}
                                <a href="{% url 'pedidos:agregar' figura.id %}" class="btn btn-warning btn-sm fw-semibold">
                                    <i class="bi bi-cart-plus me-1"></i>Agregar al Carrito
                                </a>
                            {% endif %}
                        </div>
                    </div>
                </div>
            </div>
            {% empty %}
            <div class="col-12 text-center py-5">
                <div class="p-4 bg-white rounded-3 border shadow-sm">
                    <i class="bi bi-search display-4 text-muted mb-2"></i>
                    <h5 class="text-secondary">No se encontraron figuras con los filtros seleccionados.</h5>
                    <p class="text-muted small">Prueba buscando por otro término o revisa todas las categorías.</p>
                    <a href="{% url 'catalogo:lista' %}" class="btn btn-outline-dark btn-sm mt-2">
                        Ver todas las figuras
                    </a>
                </div>
            </div>
            {% endfor %}
        </div>

        <!-- Paginación con Bootstrap -->
        {% if figuras.has_other_pages %}
        <nav aria-label="Navegación de páginas">
            <ul class="pagination justify-content-center">
                {% if figuras.has_previous %}
                    <li class="page-item">
                        <a class="page-link text-dark" href="?page={{ figuras.previous_page_number }}{% if categoria_seleccionada %}&categoria={{ categoria_seleccionada }}{% endif %}{% if query_actual %}&q={{ query_actual }}{% endif %}{% if orden_actual %}&orden={{ orden_actual }}{% endif %}">
                            Anterior
                        </a>
                    </li>
                {% else %}
                    <li class="page-item disabled"><span class="page-link">Anterior</span></li>
                {% endif %}

                {% for i in figuras.paginator.page_range %}
                    {% if figuras.number == i %}
                        <li class="page-item active"><span class="page-link bg-dark border-dark">{{ i }}</span></li>
                    {% else %}
                        <li class="page-item">
                            <a class="page-link text-dark" href="?page={{ i }}{% if categoria_seleccionada %}&categoria={{ categoria_seleccionada }}{% endif %}{% if query_actual %}&q={{ query_actual }}{% endif %}{% if orden_actual %}&orden={{ orden_actual }}{% endif %}">
                                {{ i }}
                            </a>
                        </li>
                    {% endif %}
                {% endfor %}

                {% if figuras.has_next %}
                    <li class="page-item">
                        <a class="page-link text-dark" href="?page={{ figuras.next_page_number }}{% if categoria_seleccionada %}&categoria={{ categoria_seleccionada }}{% endif %}{% if query_actual %}&q={{ query_actual }}{% endif %}{% if orden_actual %}&orden={{ orden_actual }}{% endif %}">
                            Siguiente
                        </a>
                    </li>
                {% else %}
                    <li class="page-item disabled"><span class="page-link">Siguiente</span></li>
                {% endif %}
            </ul>
        </nav>
        {% endif %}
    </div>
</div>
{% endblock %}
```

#### Ficha Técnica de la Figura (`templates/catalogo/figura_detalle.html`)

Muestra el detalle del producto, breadcrumbs, fotografía ampliada, especificaciones (fabricante, escala, precio en pesos chilenos, stock), botón de compra y carrusel de figuras relacionadas:

```html
{% extends 'base.html' %}

{% block title %}{{ figura.nombre }} | Mundo Figuras{% endblock %}

{% block content %}
<nav aria-label="breadcrumb" class="mb-4">
    <ol class="breadcrumb">
        <li class="breadcrumb-item"><a href="{% url 'core:inicio' %}" class="text-decoration-none text-dark">Inicio</a></li>
        <li class="breadcrumb-item"><a href="{% url 'catalogo:lista' %}" class="text-decoration-none text-dark">Catálogo</a></li>
        <li class="breadcrumb-item"><a href="{% url 'catalogo:lista' %}?categoria={{ figura.categoria.slug }}" class="text-decoration-none text-dark">{{ figura.categoria.nombre }}</a></li>
        <li class="breadcrumb-item active" aria-current="page">{{ figura.nombre }}</li>
    </ol>
</nav>

<div class="card border-0 shadow-sm rounded-4 overflow-hidden mb-5 bg-white">
    <div class="row g-0">
        <!-- Fotografía de la Figura -->
        <div class="col-md-6 bg-white d-flex align-items-center justify-content-center p-4 p-md-5 border-end">
            {% if figura.imagen %}
                <img src="{{ figura.imagen.url }}" alt="{{ figura.nombre }}" class="img-fluid rounded-3" style="max-height: 420px; object-fit: contain;">
            {% else %}
                <div class="text-center py-5 text-muted">
                    <i class="bi bi-image display-1"></i>
                    <p class="mt-2 mb-0 small">Fotografía no disponible</p>
                </div>
            {% endif %}
        </div>

        <!-- Información y Compra -->
        <div class="col-md-6 p-4 p-md-5 d-flex flex-column justify-content-between">
            <div>
                <div class="d-flex justify-content-between align-items-start mb-2">
                    <span class="badge bg-dark px-3 py-2 fw-semibold">
                        {{ figura.categoria.nombre }}
                    </span>
                    {% if user.is_staff %}
                        <div class="btn-group btn-group-sm">
                            <a href="{% url 'catalogo:editar' figura.id %}" class="btn btn-outline-warning text-dark">
                                <i class="bi bi-pencil me-1"></i>Editar
                            </a>
                            <a href="{% url 'catalogo:eliminar' figura.id %}" class="btn btn-outline-danger">
                                <i class="bi bi-trash me-1"></i>Eliminar
                            </a>
                        </div>
                    {% endif %}
                </div>

                <h1 class="h2 fw-bold text-dark mb-2">{{ figura.nombre }}</h1>

                <div class="mb-3 text-muted">
                    <span class="me-3"><i class="bi bi-building me-1"></i>Fabricante: <strong>{{ figura.fabricante }}</strong></span>
                    {% if figura.escala %}
                        <span><i class="bi bi-rulers me-1"></i>Escala: <strong>{{ figura.escala }}</strong></span>
                    {% endif %}
                </div>

                <div class="mb-4">
                    <span class="display-5 fw-bold text-dark">{{ figura.precio_formateado }}</span>
                    <span class="text-muted small ms-2">CLP (IVA incluido)</span>
                </div>

                <div class="mb-4">
                    {% if figura.en_stock %}
                        <span class="badge bg-success px-3 py-2 fs-6">
                            <i class="bi bi-check-circle me-1"></i>En Stock: {{ figura.stock }} piezas disponibles
                        </span>
                    {% else %}
                        <span class="badge bg-danger px-3 py-2 fs-6">
                            <i class="bi bi-x-circle me-1"></i>Agotado Temporalmente
                        </span>
                    {% endif %}
                </div>

                <div class="mb-4">
                    <h5 class="fw-bold fs-6 text-uppercase text-secondary">Descripción y Detalles</h5>
                    <p class="text-muted lh-base" style="white-space: pre-line;">{{ figura.descripcion }}</p>
                </div>
            </div>

            <!-- Botón de Agregar al Carrito -->
            <div class="border-top pt-4">
                {% if figura.en_stock %}
                    <a href="{% url 'pedidos:agregar' figura.id %}" class="btn btn-warning btn-lg w-100 py-3 fw-bold text-dark shadow-sm">
                        <i class="bi bi-cart-plus-fill me-2"></i>Agregar al Carrito de Compras
                    </a>
                {% else %}
                    <button class="btn btn-secondary btn-lg w-100 py-3" disabled>
                        <i class="bi bi-slash-circle me-2"></i>Pieza Agotada
                    </button>
                {% endif %}
            </div>
        </div>
    </div>
</div>

<!-- Figuras Relacionadas -->
{% if relacionadas %}
<div class="mb-5">
    <h3 class="h4 fw-bold mb-3"><i class="bi bi-grid-3x3-gap-fill text-warning me-2"></i>Más figuras de {{ figura.categoria.nombre }}</h3>
    <div class="row g-4">
        {% for rel in relacionadas %}
        <div class="col-md-4">
            <div class="card h-100 shadow-sm border bg-white">
                <div class="bg-white text-center p-3 border-bottom" style="height: 180px; display: flex; align-items: center; justify-content: center;">
                    {% if rel.imagen %}
                        <img src="{{ rel.imagen.url }}" alt="{{ rel.nombre }}" style="max-height: 150px; object-fit: contain;">
                    {% else %}
                        <i class="bi bi-image fs-1 text-muted"></i>
                    {% endif %}
                </div>
                <div class="card-body p-3">
                    <h6 class="fw-bold mb-1 text-truncate">{{ rel.nombre }}</h6>
                    <span class="fw-bold text-dark">{{ rel.precio_formateado }}</span>
                    <div class="mt-2">
                        <a href="{% url 'catalogo:detalle' rel.slug %}" class="btn btn-outline-dark btn-sm w-100">Ver Ficha</a>
                    </div>
                </div>
            </div>
        </div>
        {% endfor %}
    </div>
</div>
{% endif %}
{% endblock %}
```

#### Panel de Gestión de Inventario (`templates/catalogo/gestion_figuras.html`)

Panel protegido para administradores y personal de tienda (`@staff_member_required`) que presenta la tabla de stock, miniaturas y acciones de edición rápida:

```html
{% extends 'base.html' %}

{% block title %}Gestión de Inventario | Mundo Figuras{% endblock %}

{% block content %}
<div class="d-flex justify-content-between align-items-center mb-4 flex-wrap gap-2">
    <div>
        <h1 class="h3 fw-bold mb-1"><i class="bi bi-boxes text-warning me-2"></i>Gestión de Inventario</h1>
        <p class="text-muted small mb-0">Panel operativo para administrar el catálogo de figuras, stock y precios.</p>
    </div>
    <div class="d-flex gap-2">
        <a href="{% url 'catalogo:crear' %}" class="btn btn-warning fw-semibold text-dark">
            <i class="bi bi-plus-circle me-1"></i>Ingresar Nueva Figura
        </a>
        <a href="/admin/catalogo/figura/" target="_blank" class="btn btn-outline-dark">
            <i class="bi bi-shield-lock me-1"></i>Panel de Control
        </a>
    </div>
</div>

<!-- Filtro de Búsqueda Rápida -->
<div class="card border shadow-sm p-3 mb-4 rounded-3 bg-white">
    <form method="get" class="row g-2">
        <div class="col-md-10">
            <input type="text" name="q" class="form-control" placeholder="Buscar por nombre, fabricante o franquicia..." value="{{ query }}">
        </div>
        <div class="col-md-2 d-grid">
            <button type="submit" class="btn btn-dark">Filtrar</button>
        </div>
    </form>
</div>

<!-- Tabla de Figuras -->
<div class="card border-0 shadow-sm rounded-4 overflow-hidden bg-white">
    <div class="table-responsive">
        <table class="table table-hover align-middle mb-0">
            <thead class="table-dark">
                <tr>
                    <th scope="col" style="width: 70px;">ID</th>
                    <th scope="col" style="width: 80px;">Foto</th>
                    <th scope="col">Nombre de la Figura</th>
                    <th scope="col">Categoría</th>
                    <th scope="col">Fabricante</th>
                    <th scope="col">Precio</th>
                    <th scope="col">Stock</th>
                    <th scope="col">Estado</th>
                    <th scope="col" class="text-end" style="width: 150px;">Acciones</th>
                </tr>
            </thead>
            <tbody>
                {% for fig in figuras %}
                <tr>
                    <td class="fw-bold text-muted">#{{ fig.id }}</td>
                    <td>
                        {% if fig.imagen %}
                            <img src="{{ fig.imagen.url }}" alt="{{ fig.nombre }}" class="rounded border" style="width: 48px; height: 48px; object-fit: cover;">
                        {% else %}
                            <div class="bg-light rounded border text-center d-flex align-items-center justify-content-center" style="width: 48px; height: 48px;">
                                <i class="bi bi-image text-muted"></i>
                            </div>
                        {% endif %}
                    </td>
                    <td>
                        <a href="{% url 'catalogo:detalle' fig.slug %}" class="fw-bold text-dark text-decoration-none">
                            {{ fig.nombre }}
                        </a>
                        {% if fig.destacado %}
                            <span class="badge bg-warning text-dark ms-1" style="font-size: 0.65rem;">Destacada</span>
                        {% endif %}
                    </td>
                    <td><span class="badge bg-light text-dark border">{{ fig.categoria.nombre }}</span></td>
                    <td class="text-secondary small">{{ fig.fabricante }}</td>
                    <td class="fw-bold text-dark">{{ fig.precio_formateado }}</td>
                    <td>
                        {% if fig.en_stock %}
                            <span class="badge bg-success-subtle text-success border border-success-subtle">{{ fig.stock }} un.</span>
                        {% else %}
                            <span class="badge bg-danger-subtle text-danger border border-danger-subtle">Sin stock</span>
                        {% endif %}
                    </td>
                    <td>
                        {% if fig.activo %}
                            <span class="badge bg-success">Activo</span>
                        {% else %}
                            <span class="badge bg-secondary">Inactivo</span>
                        {% endif %}
                    </td>
                    <td class="text-end">
                        <div class="btn-group btn-group-sm">
                            <a href="{% url 'catalogo:editar' fig.id %}" class="btn btn-outline-primary" title="Editar Figura">
                                <i class="bi bi-pencil"></i>
                            </a>
                            <a href="{% url 'catalogo:eliminar' fig.id %}" class="btn btn-outline-danger" title="Eliminar Figura">
                                <i class="bi bi-trash"></i>
                            </a>
                        </div>
                    </td>
                </tr>
                {% empty %}
                <tr>
                    <td colspan="9" class="text-center py-5 text-muted">
                        <i class="bi bi-inbox display-4 d-block mb-2"></i>
                        No hay figuras registradas en el inventario.
                    </td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    </div>

    <!-- Paginación de la Tabla -->
    {% if figuras.has_other_pages %}
    <div class="card-footer bg-white py-3">
        <ul class="pagination pagination-sm justify-content-center mb-0">
            {% if figuras.has_previous %}
                <li class="page-item"><a class="page-link text-dark" href="?page={{ figuras.previous_page_number }}{% if query %}&q={{ query }}{% endif %}">Anterior</a></li>
            {% endif %}
            {% for i in figuras.paginator.page_range %}
                <li class="page-item {% if figuras.number == i %}active{% endif %}">
                    <a class="page-link {% if figuras.number == i %}bg-dark border-dark{% else %}text-dark{% endif %}" href="?page={{ i }}{% if query %}&q={{ query }}{% endif %}">{{ i }}</a>
                </li>
            {% endfor %}
            {% if figuras.has_next %}
                <li class="page-item"><a class="page-link text-dark" href="?page={{ figuras.next_page_number }}{% if query %}&q={{ query }}{% endif %}">Siguiente</a></li>
            {% endif %}
        </ul>
    </div>
    {% endif %}
</div>
{% endblock %}
```

#### Formulario de Creación y Edición (`templates/catalogo/figura_form.html`)

Reutilizable para crear o actualizar figuras con subida de imágenes binarias (`enctype="multipart/form-data"`):

```html
{% extends 'base.html' %}

{% block title %}{{ titulo_accion }} | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-lg-8">
        <nav aria-label="breadcrumb" class="mb-3">
            <ol class="breadcrumb small">
                <li class="breadcrumb-item"><a href="{% url 'core:inicio' %}" class="text-decoration-none text-dark">Inicio</a></li>
                <li class="breadcrumb-item"><a href="{% url 'catalogo:gestion' %}" class="text-decoration-none text-dark">Inventario</a></li>
                <li class="breadcrumb-item active">{{ titulo_accion }}</li>
            </ol>
        </nav>

        <div class="card shadow-sm border-0 rounded-4 p-4 p-md-5 bg-white">
            <div class="d-flex justify-content-between align-items-center mb-4 border-bottom pb-3">
                <div>
                    <h2 class="h4 fw-bold mb-1">{{ titulo_accion }}</h2>
                    <p class="text-muted small mb-0">Completa la ficha técnica para publicar o actualizar el producto.</p>
                </div>
                <a href="{% url 'catalogo:gestion' %}" class="btn btn-outline-secondary btn-sm">
                    <i class="bi bi-arrow-left me-1"></i>Volver al Listado
                </a>
            </div>

            <!-- Formulario con soporte para subida de fotos de figuras -->
            <form method="post" enctype="multipart/form-data" novalidate>
                {% csrf_token %}

                {% if form.non_field_errors %}
                    <div class="alert alert-danger py-2 small">
                        {{ form.non_field_errors }}
                    </div>
                {% endif %}

                <!-- Nombre y Categoría -->
                <div class="row g-3 mb-3">
                    <div class="col-md-7">
                        <label for="{{ form.nombre.id_for_label }}" class="form-label fw-semibold">{{ form.nombre.label }}</label>
                        {{ form.nombre }}
                        {% if form.nombre.errors %}
                            <div class="text-danger small mt-1">{{ form.nombre.errors }}</div>
                        {% endif %}
                    </div>
                    <div class="col-md-5">
                        <label for="{{ form.categoria.id_for_label }}" class="form-label fw-semibold">{{ form.categoria.label }}</label>
                        {{ form.categoria }}
                        {% if form.categoria.errors %}
                            <div class="text-danger small mt-1">{{ form.categoria.errors }}</div>
                        {% endif %}
                    </div>
                </div>

                <!-- Fabricante y Escala -->
                <div class="row g-3 mb-3">
                    <div class="col-md-6">
                        <label for="{{ form.fabricante.id_for_label }}" class="form-label fw-semibold">{{ form.fabricante.label }}</label>
                        {{ form.fabricante }}
                        {% if form.fabricante.errors %}
                            <div class="text-danger small mt-1">{{ form.fabricante.errors }}</div>
                        {% endif %}
                    </div>
                    <div class="col-md-6">
                        <label for="{{ form.escala.id_for_label }}" class="form-label fw-semibold">{{ form.escala.label }}</label>
                        {{ form.escala }}
                        {% if form.escala.errors %}
                            <div class="text-danger small mt-1">{{ form.escala.errors }}</div>
                        {% endif %}
                    </div>
                </div>

                <!-- Descripción -->
                <div class="mb-3">
                    <label for="{{ form.descripcion.id_for_label }}" class="form-label fw-semibold">{{ form.descripcion.label }}</label>
                    {{ form.descripcion }}
                    {% if form.descripcion.errors %}
                        <div class="text-danger small mt-1">{{ form.descripcion.errors }}</div>
                    {% endif %}
                </div>

                <!-- Precio y Stock -->
                <div class="row g-3 mb-3">
                    <div class="col-md-6">
                        <label for="{{ form.precio.id_for_label }}" class="form-label fw-semibold">{{ form.precio.label }}</label>
                        {{ form.precio }}
                        {% if form.precio.errors %}
                            <div class="text-danger small mt-1">{{ form.precio.errors }}</div>
                        {% endif %}
                    </div>
                    <div class="col-md-6">
                        <label for="{{ form.stock.id_for_label }}" class="form-label fw-semibold">{{ form.stock.label }}</label>
                        {{ form.stock }}
                        {% if form.stock.errors %}
                            <div class="text-danger small mt-1">{{ form.stock.errors }}</div>
                        {% endif %}
                    </div>
                </div>

                <!-- Fotografía -->
                <div class="mb-3">
                    <label for="{{ form.imagen.id_for_label }}" class="form-label fw-semibold">{{ form.imagen.label }}</label>
                    {{ form.imagen }}
                    {% if form.imagen.errors %}
                        <div class="text-danger small mt-1">{{ form.imagen.errors }}</div>
                    {% endif %}
                    {% if figura and figura.imagen %}
                        <div class="mt-2 p-2 bg-light rounded d-flex align-items-center gap-3 border">
                            <img src="{{ figura.imagen.url }}" alt="Vista previa" style="width: 50px; height: 50px; object-fit: cover;" class="rounded border">
                            <span class="small text-muted">Fotografía cargada actualmente.</span>
                        </div>
                    {% endif %}
                </div>

                <!-- Opciones: Destacado y Activo -->
                <div class="row g-3 mb-4 p-3 bg-light rounded-3 border">
                    <div class="col-sm-6">
                        <div class="form-check form-switch">
                            {{ form.activo }}
                            <label class="form-check-label fw-semibold" for="{{ form.activo.id_for_label }}">
                                {{ form.activo.label }}
                            </label>
                        </div>
                        <small class="text-muted d-block ps-4">Visible inmediatamente para la compra de clientes.</small>
                    </div>
                    <div class="col-sm-6">
                        <div class="form-check form-switch">
                            {{ form.destacado }}
                            <label class="form-check-label fw-semibold" for="{{ form.destacado.id_for_label }}">
                                {{ form.destacado.label }}
                            </label>
                        </div>
                        <small class="text-muted d-block ps-4">Aparece en la sección destacada de la portada.</small>
                    </div>
                </div>

                <!-- Botones de Acción -->
                <div class="d-flex justify-content-end gap-2">
                    <a href="{% url 'catalogo:gestion' %}" class="btn btn-outline-secondary">Cancelar</a>
                    <button type="submit" class="btn btn-warning fw-bold text-dark px-4">
                        <i class="bi bi-save me-1"></i>{{ boton_texto }}
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>
{% endblock %}
```

#### Confirmación de Eliminación (`templates/catalogo/figura_confirmar_eliminar.html`)

Pantalla de confirmación con método POST y token CSRF para prevenir eliminaciones accidentales:

```html
{% extends 'base.html' %}

{% block title %}Confirmar Eliminación | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card border-danger shadow-sm rounded-4 overflow-hidden bg-white">
            <div class="card-header bg-danger text-white py-3">
                <h5 class="card-title mb-0"><i class="bi bi-exclamation-triangle-fill me-2"></i>Confirmar Eliminación de Pieza</h5>
            </div>
            <div class="card-body p-4 p-md-5 text-center">
                <i class="bi bi-trash3-fill text-danger display-3 mb-3"></i>
                <h4 class="fw-bold mb-2">¿Estás seguro de que deseas eliminar esta figura?</h4>
                <p class="text-muted mb-4">Esta acción no se puede deshacer y retirará el registro permanentemente de la base de datos:</p>

                <div class="p-3 bg-light rounded-3 border mb-4 text-start">
                    <ul class="list-unstyled mb-0">
                        <li><strong>ID:</strong> #{{ figura.id }}</li>
                        <li><strong>Nombre:</strong> {{ figura.nombre }}</li>
                        <li><strong>Categoría:</strong> {{ figura.categoria.nombre }}</li>
                        <li><strong>Fabricante:</strong> {{ figura.fabricante }}</li>
                        <li><strong>Precio:</strong> {{ figura.precio_formateado }}</li>
                        <li><strong>Stock actual:</strong> {{ figura.stock }} piezas</li>
                    </ul>
                </div>

                <form method="post">
                    {% csrf_token %}
                    <div class="d-flex justify-content-center gap-2">
                        <a href="{% url 'catalogo:gestion' %}" class="btn btn-outline-secondary px-4">
                            Cancelar
                        </a>
                        <button type="submit" class="btn btn-danger px-4 fw-semibold">
                            <i class="bi bi-trash-fill me-1"></i>Sí, Eliminar Definitivamente
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Plantillas del Carrito y Pedidos (`templates/pedidos/`)

#### Carrito de Compras (`templates/pedidos/carrito.html`)

Visualiza los productos acumulados en la sesión HTTP, control interactivo de cantidades (+ / -), subtotales calculados dinámicamente, cálculo de IVA y botón de checkout:

```html
{% extends 'base.html' %}

{% block title %}Mi Carrito de Compras | Mundo Figuras{% endblock %}

{% block content %}
<div class="mb-4">
    <h1 class="h3 fw-bold mb-1"><i class="bi bi-cart3 text-warning me-2"></i>Carrito de Compras</h1>
    <p class="text-muted small">Colección de figuras seleccionadas para tu compra.</p>
</div>

{% if esta_vacio %}
<div class="card border-0 shadow-sm rounded-4 text-center py-5 bg-white">
    <div class="card-body py-5">
        <i class="bi bi-cart-x text-muted display-1 mb-3"></i>
        <h3 class="fw-bold text-secondary">Tu carrito de compras está vacío</h3>
        <p class="text-muted mb-4">Aún no has agregado figuras a tu pedido.</p>
        <a href="{% url 'catalogo:lista' %}" class="btn btn-warning px-4 py-2 fw-bold text-dark">
            <i class="bi bi-grid-3x3-gap me-2"></i>Explorar Figuras Coleccionables
        </a>
    </div>
</div>
{% else %}
<div class="row g-4">
    <!-- Tabla de Ítems en Carrito -->
    <div class="col-lg-8">
        <div class="card border-0 shadow-sm rounded-4 overflow-hidden mb-3 bg-white">
            <div class="table-responsive">
                <table class="table table-hover align-middle mb-0">
                    <thead class="table-dark">
                        <tr>
                            <th scope="col" style="width: 80px;">Foto</th>
                            <th scope="col">Figura</th>
                            <th scope="col">Precio Unit.</th>
                            <th scope="col" class="text-center" style="width: 150px;">Cantidad</th>
                            <th scope="col" class="text-end">Subtotal</th>
                            <th scope="col" class="text-end" style="width: 60px;"></th>
                        </tr>
                    </thead>
                    <tbody>
                        {% for item in items_carrito %}
                        <tr>
                            <td>
                                {% if item.imagen_url %}
                                    <img src="{{ item.imagen_url }}" alt="{{ item.nombre }}" class="rounded border" style="width: 50px; height: 50px; object-fit: cover;">
                                {% else %}
                                    <div class="bg-light rounded border text-center d-flex align-items-center justify-content-center" style="width: 50px; height: 50px;">
                                        <i class="bi bi-image text-muted"></i>
                                    </div>
                                {% endif %}
                            </td>
                            <td>
                                <a href="{% url 'catalogo:detalle' item.slug %}" class="fw-bold text-dark text-decoration-none">
                                    {{ item.nombre }}
                                </a>
                                <small class="text-muted d-block">{{ item.fabricante }}</small>
                            </td>
                            <td class="text-muted">${{ item.precio|floatformat:0 }}</td>
                            <td class="text-center">
                                <div class="btn-group btn-group-sm border rounded">
                                    <a href="{% url 'pedidos:restar' item.id %}" class="btn btn-light px-2" title="Restar">-</a>
                                    <span class="btn btn-light disabled text-dark fw-bold px-3">{{ item.cantidad }}</span>
                                    <a href="{% url 'pedidos:agregar' item.id %}" class="btn btn-light px-2" title="Sumar">+</a>
                                </div>
                            </td>
                            <td class="text-end fw-bold text-dark">${{ item.subtotal|floatformat:0 }}</td>
                            <td class="text-end">
                                <a href="{% url 'pedidos:eliminar' item.id %}" class="text-danger" title="Eliminar ítem">
                                    <i class="bi bi-trash fs-5"></i>
                                </a>
                            </td>
                        </tr>
                        {% endfor %}
                    </tbody>
                </table>
            </div>

            <div class="card-footer bg-white p-3 d-flex justify-content-between align-items-center">
                <a href="{% url 'catalogo:lista' %}" class="btn btn-outline-secondary btn-sm">
                    <i class="bi bi-arrow-left me-1"></i>Seguir Viendo Figuras
                </a>
                <a href="{% url 'pedidos:limpiar' %}" class="btn btn-outline-danger btn-sm" onclick="return confirm('¿Deseas vaciar el carrito?');">
                    <i class="bi bi-trash me-1"></i>Vaciar Carrito
                </a>
            </div>
        </div>
    </div>

    <!-- Resumen de Compra -->
    <div class="col-lg-4">
        <div class="card border-0 shadow-sm rounded-4 p-4 bg-white">
            <h5 class="fw-bold mb-3"><i class="bi bi-receipt me-2 text-warning"></i>Resumen de Compra</h5>

            <div class="d-flex justify-content-between text-muted mb-2">
                <span>Subtotal neto:</span>
                <span>${{ subtotal|floatformat:0 }} CLP</span>
            </div>
            <div class="d-flex justify-content-between text-muted mb-3">
                <span>IVA (19%):</span>
                <span>${{ iva|floatformat:0 }} CLP</span>
            </div>

            <hr>

            <div class="d-flex justify-content-between align-items-center mb-4">
                <span class="fs-5 fw-bold text-dark">Total a Pagar:</span>
                <span class="fs-4 fw-bold text-dark">${{ total|floatformat:0 }} CLP</span>
            </div>

            {% if user.is_authenticated %}
                <a href="{% url 'pedidos:checkout' %}" class="btn btn-warning w-100 py-3 fw-bold text-dark shadow-sm">
                    <i class="bi bi-credit-card me-2"></i>Confirmar y Procesar Compra
                </a>
            {% else %}
                <div class="alert alert-warning small mb-3">
                    <i class="bi bi-info-circle me-1"></i>Inicia sesión con tu cuenta para confirmar la compra.
                </div>
                <a href="{% url 'usuarios:login' %}?next={% url 'pedidos:checkout' %}" class="btn btn-warning w-100 py-3 fw-bold text-dark">
                    <i class="bi bi-box-arrow-in-right me-2"></i>Iniciar Sesión para Comprar
                </a>
            {% endif %}
        </div>
    </div>
</div>
{% endif %}
{% endblock %}
```

#### Confirmación de Compra Exitosa (`templates/pedidos/checkout_exito.html`)

Comprobante digital del pedido con código de orden, desglose de ítems comprados y total pagado:

```html
{% extends 'base.html' %}

{% block title %}Compra Confirmada | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-lg-8">
        <div class="card border-0 shadow-sm rounded-4 text-center p-4 p-md-5 bg-white">
            <div class="mb-4">
                <i class="bi bi-check-circle-fill text-success display-1"></i>
                <h1 class="h2 fw-bold mt-3">¡Orden Confirmada con Éxito!</h1>
                <p class="text-muted lead">Muchas gracias por tu compra, {{ user.first_name|default:user.username }}. Tus piezas coleccionables están siendo preparadas.</p>
                <span class="badge bg-dark fs-6 px-3 py-2 text-warning">Código de Pedido: {{ orden_id }}</span>
            </div>

            <div class="text-start border rounded-3 p-3 p-md-4 mb-4 bg-light">
                <h5 class="fw-bold mb-3"><i class="bi bi-box-seam me-2 text-warning"></i>Figuras adquiridas:</h5>
                <ul class="list-group list-group-flush mb-3">
                    {% for item in items %}
                    <li class="list-group-item bg-transparent d-flex justify-content-between align-items-center px-0">
                        <div>
                            <strong>{{ item.nombre }}</strong>
                            <span class="text-muted small d-block">{{ item.fabricante }} - Cantidad: {{ item.cantidad }} x ${{ item.precio|floatformat:0 }}</span>
                        </div>
                        <span class="fw-bold">${{ item.subtotal|floatformat:0 }} CLP</span>
                    </li>
                    {% endfor %}
                </ul>
                <div class="d-flex justify-content-between pt-2 border-top">
                    <span>Subtotal neto:</span>
                    <span>${{ subtotal|floatformat:0 }} CLP</span>
                </div>
                <div class="d-flex justify-content-between text-muted">
                    <span>IVA (19%):</span>
                    <span>${{ iva|floatformat:0 }} CLP</span>
                </div>
                <div class="d-flex justify-content-between fs-5 fw-bold text-dark pt-2 border-top mt-2">
                    <span>Total Pagado:</span>
                    <span class="text-dark">${{ total|floatformat:0 }} CLP</span>
                </div>
            </div>

            <div class="d-flex justify-content-center gap-3">
                <a href="{% url 'catalogo:lista' %}" class="btn btn-warning fw-bold text-dark px-4 py-2">
                    <i class="bi bi-grid-3x3-gap me-1"></i>Seguir Explorando
                </a>
                <a href="{% url 'usuarios:perfil' %}" class="btn btn-outline-secondary px-4 py-2">
                    <i class="bi bi-person me-1"></i>Ver Mi Cuenta
                </a>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Plantillas de Usuarios y Sesión (`templates/usuarios/`)

#### Inicio de Sesión (`templates/usuarios/login.html`)

Formulario de login con parámetro `next` para redirigir al checkout o a la vista previa tras identificarse:

```html
{% extends 'base.html' %}

{% block title %}Iniciar Sesión | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-6 col-lg-5">
        <div class="card shadow-sm border-0 rounded-4 p-4 p-md-5 bg-white">
            <div class="text-center mb-4">
                <i class="bi bi-person-lock text-warning display-4"></i>
                <h2 class="h4 fw-bold mt-2">Acceso a Tu Cuenta</h2>
                <p class="text-muted small">Ingresa tus datos para continuar con tus pedidos</p>
            </div>

            <form method="post" novalidate>
                {% csrf_token %}
                <input type="hidden" name="next" value="{{ next }}">

                {% if form.non_field_errors %}
                    <div class="alert alert-danger py-2 small">
                        {{ form.non_field_errors }}
                    </div>
                {% endif %}

                <div class="mb-3">
                    <label for="{{ form.username.id_for_label }}" class="form-label fw-semibold">Nombre de Usuario</label>
                    {{ form.username }}
                    {% if form.username.errors %}
                        <div class="text-danger small mt-1">{{ form.username.errors }}</div>
                    {% endif %}
                </div>

                <div class="mb-4">
                    <label for="{{ form.password.id_for_label }}" class="form-label fw-semibold">Contraseña</label>
                    {{ form.password }}
                    {% if form.password.errors %}
                        <div class="text-danger small mt-1">{{ form.password.errors }}</div>
                    {% endif %}
                </div>

                <div class="d-grid gap-2">
                    <button type="submit" class="btn btn-warning py-2 fw-bold text-dark">
                        <i class="bi bi-box-arrow-in-right me-1"></i>Iniciar Sesión
                    </button>
                </div>
            </form>

            <hr class="my-4">

            <div class="text-center">
                <p class="text-muted small mb-0">¿Nuevo coleccionista? 
                    <a href="{% url 'usuarios:registro' %}" class="fw-semibold text-warning text-decoration-none">Regístrate gratis</a>
                </p>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

#### Registro de Nuevos Clientes (`templates/usuarios/registro.html`)

Formulario de registro basado en `UserCreationForm` con validación de contraseña segura y correo electrónico único:

```html
{% extends 'base.html' %}

{% block title %}Crear Cuenta de Cliente | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-7 col-lg-6">
        <div class="card shadow-sm border-0 rounded-4 p-4 p-md-5 bg-white">
            <div class="text-center mb-4">
                <i class="bi bi-person-plus-fill text-warning display-4"></i>
                <h2 class="h4 fw-bold mt-2">Crear Cuenta de Cliente</h2>
                <p class="text-muted small">Únete a nuestra comunidad de coleccionistas y guarda tus compras</p>
            </div>

            <form method="post" novalidate>
                {% csrf_token %}

                {% if form.non_field_errors %}
                    <div class="alert alert-danger py-2 small">
                        {{ form.non_field_errors }}
                    </div>
                {% endif %}

                <div class="row g-3 mb-3">
                    <div class="col-sm-6">
                        <label for="{{ form.first_name.id_for_label }}" class="form-label fw-semibold">Nombres</label>
                        {{ form.first_name }}
                        {% if form.first_name.errors %}
                            <div class="text-danger small mt-1">{{ form.first_name.errors }}</div>
                        {% endif %}
                    </div>
                    <div class="col-sm-6">
                        <label for="{{ form.last_name.id_for_label }}" class="form-label fw-semibold">Apellidos</label>
                        {{ form.last_name }}
                        {% if form.last_name.errors %}
                            <div class="text-danger small mt-1">{{ form.last_name.errors }}</div>
                        {% endif %}
                    </div>
                </div>

                <div class="mb-3">
                    <label for="{{ form.username.id_for_label }}" class="form-label fw-semibold">Nombre de Usuario</label>
                    {{ form.username }}
                    <small class="text-muted d-block mt-1">{{ form.username.help_text }}</small>
                    {% if form.username.errors %}
                        <div class="text-danger small mt-1">{{ form.username.errors }}</div>
                    {% endif %}
                </div>

                <div class="mb-3">
                    <label for="{{ form.email.id_for_label }}" class="form-label fw-semibold">Correo Electrónico</label>
                    {{ form.email }}
                    {% if form.email.errors %}
                        <div class="text-danger small mt-1">{{ form.email.errors }}</div>
                    {% endif %}
                </div>

                <div class="mb-3">
                    <label for="{{ form.password1.id_for_label }}" class="form-label fw-semibold">Contraseña</label>
                    {{ form.password1 }}
                    {% if form.password1.errors %}
                        <div class="text-danger small mt-1">{{ form.password1.errors }}</div>
                    {% endif %}
                </div>

                <div class="mb-4">
                    <label for="{{ form.password2.id_for_label }}" class="form-label fw-semibold">Confirmar Contraseña</label>
                    {{ form.password2 }}
                    {% if form.password2.errors %}
                        <div class="text-danger small mt-1">{{ form.password2.errors }}</div>
                    {% endif %}
                </div>

                <div class="d-grid gap-2">
                    <button type="submit" class="btn btn-warning py-2 fw-bold text-dark">
                        <i class="bi bi-check-circle me-1"></i>Registrar Cuenta
                    </button>
                </div>
            </form>

            <hr class="my-4">

            <div class="text-center">
                <p class="text-muted small mb-0">¿Ya tienes una cuenta registrada? 
                    <a href="{% url 'usuarios:login' %}" class="fw-semibold text-warning text-decoration-none">Inicia sesión</a>
                </p>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

#### Perfil del Usuario (`templates/usuarios/perfil.html`)

Muestra la información de la cuenta del cliente logueado (`@login_required`), insignia de rol y accesos a inventario si es staff:

```html
{% extends 'base.html' %}

{% block title %}Mi Cuenta | Mundo Figuras{% endblock %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-lg-8">
        <div class="card border-0 shadow-sm rounded-4 overflow-hidden bg-white">
            <div class="bg-dark text-white p-4 p-md-5">
                <div class="d-flex align-items-center gap-3">
                    <i class="bi bi-person-circle fs-1 text-warning"></i>
                    <div>
                        <h2 class="h3 fw-bold mb-0">{{ usuario.get_full_name|default:usuario.username }}</h2>
                        <span class="badge bg-warning text-dark fw-bold mt-1">@{{ usuario.username }}</span>
                        {% if usuario.is_staff %}
                            <span class="badge bg-info text-dark fw-bold mt-1 ms-1">Personal de Tienda</span>
                        {% endif %}
                    </div>
                </div>
            </div>

            <div class="card-body p-4 p-md-5">
                <h5 class="fw-bold mb-3"><i class="bi bi-person-lines-fill me-2 text-warning"></i>Datos del Cliente</h5>
                <div class="table-responsive">
                    <table class="table table-bordered">
                        <tbody>
                            <tr>
                                <th class="bg-light text-muted w-25">Nombre Completo:</th>
                                <td>{{ usuario.get_full_name|default:"No especificado" }}</td>
                            </tr>
                            <tr>
                                <th class="bg-light text-muted">Correo Electrónico:</th>
                                <td>{{ usuario.email|default:"No registrado" }}</td>
                            </tr>
                            <tr>
                                <th class="bg-light text-muted">Fecha de Creación:</th>
                                <td>{{ usuario.date_joined|date:"d \d\e F \d\e Y" }}</td>
                            </tr>
                            <tr>
                                <th class="bg-light text-muted">Último Acceso:</th>
                                <td>{{ usuario.last_login|date:"d \d\e F \d\e Y, H:i"|default:"Primer acceso" }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>

                <div class="mt-4 d-flex gap-2">
                    <a href="{% url 'catalogo:lista' %}" class="btn btn-warning fw-semibold">
                        <i class="bi bi-grid me-1"></i>Explorar Catálogo
                    </a>
                    {% if usuario.is_staff %}
                        <a href="{% url 'catalogo:gestion' %}" class="btn btn-dark">
                            <i class="bi bi-boxes me-1"></i>Gestionar Inventario
                        </a>
                    {% endif %}
                </div>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

---

## Paso 11: Enrutamiento Principal con Namespaces (`config/urls.py`)

Abre `config/urls.py` y conecta todas las aplicaciones mediante `include`:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('apps.core.urls', namespace='core')),
    path('usuarios/', include('apps.usuarios.urls', namespace='usuarios')),
    path('catalogo/', include('apps.catalogo.urls', namespace='catalogo')),
    path('pedidos/', include('apps.pedidos.urls', namespace='pedidos')),
]

# Servir archivos multimedia durante el desarrollo local
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

---

## Paso 12: Carga Inicial de Datos y Verificación del Sistema

### Creación del Comando Personalizado `poblar_datos`
Para inicializar la tienda con datos realistas, crea `apps/catalogo/management/commands/poblar_datos.py`:

```python
from django.core.management.base import BaseCommand
from django.contrib.auth.models import User
from apps.catalogo.models import Categoria, Figura


class Command(BaseCommand):
    help = "Pobla la base de datos con categorías y figuras de prueba."

    def handle(self, *args, **kwargs):
        if not User.objects.filter(username="admin").exists():
            User.objects.create_superuser("admin", "admin@mundofiguras.cl", "admin")
            self.stdout.write(self.style.SUCCESS("✓ Superusuario 'admin' creado."))

        categorias = {
            "Anime y Manga": "Series animadas japonesas.",
            "Cómics y Superhéroes": "Universo Marvel y DC Comics.",
            "Videojuegos": "Franquicias de consolas y PC.",
            "Cine y Televisión": "Películas de culto y sagas."
        }
        cat_map = {}
        for nombre, desc in categorias.items():
            cat, _ = Categoria.objects.get_or_create(nombre=nombre, defaults={'descripcion': desc})
            cat_map[nombre] = cat

        figuras = [
            ("Figura Son Goku Super Saiyan", cat_map["Anime y Manga"], "Bandai Spirits", 34990, 10, True),
            ("Figura Monkey D. Luffy Gear 5", cat_map["Anime y Manga"], "Banpresto", 39990, 8, True),
            ("Estatua Batman The Dark Knight", cat_map["Cómics y Superhéroes"], "Kotobukiya", 189990, 3, True),
            ("Figura Link Tears of the Kingdom", cat_map["Videojuegos"], "Good Smile", 69990, 6, True),
        ]
        for nom, cat, fab, pre, stk, dest in figuras:
            Figura.objects.get_or_create(nombre=nom, defaults={
                'categoria': cat, 'fabricante': fab, 'precio': pre, 'stock': stk, 'destacado': dest, 'activo': True
            })

        self.stdout.write(self.style.SUCCESS("✓ Catálogo poblado exitosamente."))
```

### Ejecución
```bash
python manage.py poblar_datos
python manage.py runserver
```

Abre en tu navegador:
* **Catálogo y Portada:** [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
* **Gestión Interna de Inventario:** [http://127.0.0.1:8000/catalogo/gestion/](http://127.0.0.1:8000/catalogo/gestion/)
* **Django Admin:** [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/) (Usuario: `admin` / Clave: `admin`)

---

## 📚 Glosario de Conceptos Técnicos

| Concepto | Definición Técnica |
| :--- | :--- |
| **ORM (Object-Relational Mapping)** | Capa de abstracción que mapea tablas relacionales a clases de Python, gestionando consultas y transacciones sin escribir código SQL directo. |
| **Migración** | Archivo de registro secuencial que permite versionar y sincronizar las alteraciones de la estructura de datos en el motor SQL físico. |
| **Slug** | Fragmento de texto simplificado (alfanumérico y guiones) que reemplaza identificadores numéricos en las URLs para mejorar legibilidad y posicionamiento en buscadores. |
| **CSRF (Cross-Site Request Forgery)** | Mecanismo de protección que exige un token criptográfico en peticiones POST para garantizar que provienen del propio sitio y no de un tercero malicioso. |
| **Sesión HTTP** | Almacenamiento temporal del lado del servidor referenciado mediante cookies en el cliente, permitiendo persistir estado entre distintas peticiones web. |
| **Context Processor** | Función de Django que inyecta variables globales en el contexto de todas las plantillas renderizadas por el motor DTL. |
| **Integridad Referencial (`PROTECT`)** | Política de clave foránea que bloquea la eliminación de un registro padre mientras existan registros dependientes vinculados. |
| **Namespaces de URL** | Mecanismo de prefijos que permite invocar rutas de forma única (`app_name:route_name`), evitando colisiones de nombres entre distintas aplicaciones. |
