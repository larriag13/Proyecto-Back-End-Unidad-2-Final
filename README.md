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
