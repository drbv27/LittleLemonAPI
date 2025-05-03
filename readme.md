# Little Lemon API 🍋

[![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-4.x-092E20?logo=django&logoColor=white)](https://www.djangoproject.com/)
[![Django REST Framework](https://img.shields.io/badge/Django%20REST%20Framework-3.x-A30000?logo=django&logoColor=white)](https://www.django-rest-framework.org/)
[![SQLite](https://img.shields.io/badge/SQLite-3-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org/index.html)

API RESTful desarrollada con Django y Django REST Framework para el restaurante ficticio "Little Lemon". Este proyecto forma parte del curso [Meta Backend Developer Professional Certificate](https://www.coursera.org/professional-certificates/meta-back-end-developer) en Coursera.

## Descripción General 📝

La gerencia de Little Lemon necesita un sistema para gestionar pedidos en línea y una API para su futura aplicación móvil. Esta API actúa como el _backend_, permitiendo a los usuarios realizar diversas acciones según su rol:

- **Clientes (`Customers`):** Pueden registrarse, iniciar sesión, explorar el menú (por categoría, precio, paginado), gestionar su carrito de compras y realizar/consultar sus pedidos.
- **Equipo de Entrega (`Delivery crew`):** Pueden consultar los pedidos asignados y marcarlos como entregados.
- **Gerentes (`Managers`):** Pueden gestionar ítems del menú, categorías, actualizar el plato del día, gestionar usuarios (asignarlos al equipo de entrega) y asignar/supervisar pedidos.
- **Administrador (`Admin`/`Superuser`):** Tiene control total a través del panel de administración de Django y permisos elevados en la API.

## Características Principales ✨

Esta API implementa las siguientes funcionalidades clave:

### Gestión de Usuarios y Roles:

1.  El `admin` (Administrador) puede asignar usuarios al grupo `Manager` (Gerente).
2.  Se puede acceder a las funcionalidades del grupo `Manager` con un `token` de `admin` (o un `token` de `Manager`). _(Aclaración: Un admin tiene permisos de manager)_
3.  Los `Managers` (Gerentes) pueden asignar usuarios al `Delivery crew` (Equipo de entrega).
4.  Los `Customers` (Clientes) pueden registrarse.
5.  Los `Customers` (Clientes) pueden iniciar sesión usando su `username` y `password` para obtener `tokens` de acceso (`access tokens`).

### Gestión del Menú:

6.  El `admin` (Administrador) puede añadir ítems al menú.
7.  El `admin` (Administrador) puede añadir categorías.
8.  Los `Managers` (Gerentes) pueden actualizar el ítem destacado del día (`item of the day`).
9.  Los `Customers` (Clientes) pueden explorar todas las categorías.
10. Los `Customers` (Clientes) pueden explorar todos los ítems del menú a la vez.
11. Los `Customers` (Clientes) pueden explorar los ítems del menú por categoría.
12. Los `Customers` (Clientes) pueden paginar los ítems del menú.
13. Los `Customers` (Clientes) pueden ordenar los ítems del menú por precio (`price`).

### Gestión del Carrito (`Cart`):

14. Los `Customers` (Clientes) pueden añadir ítems del menú al carrito (`cart`).
15. Los `Customers` (Clientes) pueden acceder a los ítems previamente añadidos en su carrito (`cart`).
16. Los `Customers` (Clientes) pueden vaciar su carrito (`cart`). _(Asumiendo funcionalidad DELETE)_

### Gestión de Pedidos (`Orders`):

17. Los `Customers` (Clientes) pueden realizar pedidos (`place orders`) basados en su carrito.
18. Los `Customers` (Clientes) pueden explorar sus propios pedidos (`orders`).
19. Los `Managers` (Gerentes) pueden asignar pedidos (`orders`) al `Delivery crew` (Equipo de entrega).
20. El `Delivery crew` (Equipo de entrega) puede acceder a los pedidos (`orders`) que les han sido asignados.
21. El `Delivery crew` (Equipo de entrega) puede marcar un pedido (`order`) como entregado (`delivered`).

## Estructura de la API (Endpoints) 🛠️

A continuación se detallan los principales endpoints y sus funcionalidades:

---

### Autenticación y Usuarios (`/api/auth/`)

- **`POST /api/auth/users/`**
  - **Descripción:** Registra un nuevo usuario (Cliente).
  - **Permisos:** Cualquiera.
  - **Datos Requeridos:** `username`, `password`, `email`.
- **`POST /api/auth/token/login/`**
  - **Descripción:** Obtiene un `token` de autenticación (`auth token`).
  - **Permisos:** Cualquiera.
  - **Datos Requeridos:** `username`, `password`.
- **`GET /api/auth/users/me/`**
  - **Descripción:** Obtiene la información del usuario autenticado actual (basado en el `token`).
  - **Permisos:** Usuario autenticado (`Manager`, `Delivery crew`, `Customer`).
- **`GET /api/auth/users/`**
  - **Descripción:** Obtiene información de usuarios.
  - **Permisos:**
    - Usuario Registrado (`Manager`, `Delivery crew`, `Customer`): Obtiene su propia información.
    - `Superuser`: Obtiene la información de _todos_ los usuarios.
    - Anónimo: `401 Unauthorized`.

---

### Menú (`/api/menu-items/`)

- **`GET /api/menu-items/`**
  - **Descripción:** Lista los ítems del menú con opciones de filtrado, paginación y ordenación.
  - **Permisos:** Todos (con límites de tasa distintos).
  - **Parámetros Query:**
    - Paginación: `?perpage={5-50}` , `?page={numero}`
    - Búsqueda: `?search={titulo_o_categoria}`
    - Ordenación: `?ordering=price` (ascendente), `?ordering=-price` (descendente), `?ordering=category`, etc.
  - **Límites de Tasa (Throttling):** Anónimo (ej: 2/min), Autenticado (ej: 100/min).
- **`POST /api/menu-items/`**
  - **Descripción:** Crea un nuevo ítem de menú.
  - **Permisos:** `Superuser` (o `Manager`, según tu implementación).
  - **Datos Requeridos:** `title`, `price`, `category_id` (probablemente), etc.
- **`GET /api/menu-items/{menuitemId}`**
  - **Descripción:** Obtiene los detalles de un ítem de menú específico.
  - **Permisos:** Todos.
- **`PATCH /api/menu-items/{menuitemId}`**
  - **Descripción:** Actualiza parcialmente un ítem (ej: marcar como destacado/`featured`).
  - **Permisos:** `Manager`, `Superuser`.
  - **Datos Opcionales:** `featured` (boolean), etc.
- **`DELETE /api/menu-items/{menuitemId}`**
  - **Descripción:** Elimina un ítem de menú.
  - **Permisos:** `Superuser` (o `Manager`).

---

### Categorías (`/api/categories/`)

- **`GET /api/categories/`**
  - **Descripción:** Lista todas las categorías de menú.
  - **Permisos:** Todos.
- **`POST /api/categories/`**
  - **Descripción:** Crea una nueva categoría.
  - **Permisos:** `Superuser` (o `Manager`).
  - **Datos Requeridos:** `slug`, `title`.

---

### Grupos y Roles (`/api/groups/`)

- **`GET /api/groups/managers/users`**
  - **Descripción:** Lista los usuarios en el grupo `Manager`.
  - **Permisos:** `Manager`, `Superuser`.
- **`POST /api/groups/managers/users`**
  - **Descripción:** Añade un usuario al grupo `Manager`.
  - **Permisos:** `Manager`, `Superuser`.
  - **Datos Requeridos:** `username` (o `user_id`).
- **`DELETE /api/groups/managers/users/{userId}`**
  - **Descripción:** Elimina un usuario del grupo `Manager`.
  - **Permisos:** `Manager`, `Superuser`.
- **(Endpoints similares para `/api/groups/delivery-crew/users`)**

---

### Carrito de Compras (`/api/cart/`)

- **`GET /api/cart/menu-items`**
  - **Descripción:** Muestra los ítems en el carrito del usuario actual.
  - **Permisos:** `Customer`.
- **`POST /api/cart/menu-items`**
  - **Descripción:** Añade un ítem (y cantidad) al carrito.
  - **Permisos:** `Customer`.
  - **Datos Requeridos:** `menuitem_id`, `quantity`.
- **`DELETE /api/cart/menu-items`**
  - **Descripción:** Vacía completamente el carrito del usuario.
  - **Permisos:** `Customer`.

---

### Pedidos (`/api/orders/`)

- **`GET /api/orders`**
  - **Descripción:** Lista pedidos según el rol del usuario.
  - **Permisos:**
    - `Customer`: Ve sus propios pedidos.
    - `Delivery crew`: Ve pedidos asignados a él/ella.
    - `Manager`, `Superuser`: Ven todos los pedidos.
- **`POST /api/orders`**
  - **Descripción:** Crea un nuevo pedido a partir del carrito del usuario y vacía el carrito.
  - **Permisos:** `Customer`.
- **`GET /api/orders/{orderId}`**
  - **Descripción:** Obtiene detalles de un pedido específico.
  - **Permisos:** `Customer` (si es su pedido), `Delivery crew` (si está asignado), `Manager`, `Superuser`.
- **`PATCH /api/orders/{orderId}`**
  - **Descripción:** Actualiza el estado del pedido (ej: marcar como entregado).
  - **Permisos:** `Delivery crew`, `Manager`, `Superuser`.
  - **Datos Opcionales:** `status` (boolean o string).
- **`PUT /api/orders/{orderId}`**
  - **Descripción:** Asigna un miembro del `Delivery crew` al pedido.
  - **Permisos:** `Manager`, `Superuser`.
  - **Datos Requeridos:** `delivery_crew_id`.
- **`DELETE /api/orders/{orderId}`**
  - **Descripción:** Elimina un pedido.
  - **Permisos:** `Manager`, `Superuser`.

---

## 🚀 Cómo Ejecutar el Proyecto Localmente

Sigue estos pasos para poner en marcha el proyecto en tu máquina local. Estas instrucciones asumen que tienes Python 3.x y Git instalados y estás usando una terminal tipo Bash (como Git Bash en Windows).

**1. Clonar el Repositorio**

Primero, clona este repositorio en tu máquina local:

```bash
# Clonar repositorio
git clone [https://github.com/tu-usuario/tu-repositorio.git](https://github.com/tu-usuario/tu-repositorio.git)
cd tu-repositorio
```

**2. Crear y Activar un entorno Virtual**

Es altamente recomendable usar un entorno virtual para aislar las dependencias del proyecto.

**_-Crear el entorno(dentro de la carpeta del proyecto):_**

```bash
python -m venv venv
```

**_-Activar el entorno (para Git Bash en Windows):_**

```bash
Source venv/Scripts/activate
```

(Deberías ver `(venv)` al inicio de tu prompt en la terminal si se activó correctamente)

**3. Instalar Dependencias**

Instala todas las librerías necesarias listadas en `requirements.txt`:

```bash
pip install -r requirements.txt
```

**4. Aplicar Migraciones de Base de Datos**

Django necesita configurar la base de datos. Ejecuta el comando de migración:

```bash
python manage.py migrate
```

**5. Crear un Superusuario (Opcional)**

Si deseas acceder al panel de administración de Django (`/admin/`), crea un superusuario:

```bash
python manage.py createsuperuser
```

Sigue las instrucciones para crear tu nombre de usuario y contraseña.

**6. Ejecutar el Servidor de Desarrollo**

¡Ahora puedes iniciar el servidor!

```bash
python manage.py runserver
```

**7. Abrir en el Navegador**

Una vez que el servidor esté corriendo, abre tu navegador web y ve a:

http://127.0.0.1:8000/

Deberías ver la página principal de Little Lemon.
Para detener el servidor, vuelve a la terminal y presiona `Ctrl + C`.

📚 Origen del Proyecto
Este proyecto se basa en el material y los ejercicios del curso:

[Meta Django Web Framework en Coursera](https://www.coursera.org/learn/django-web-framework-es)

## 📫 Contacto

[![GitHub](https://img.shields.io/badge/GitHub-drbv27-181717?logo=github)](https://github.com/drbv27)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-DiegoBonilla-0A66C2?logo=linkedin)](https://www.linkedin.com/in/diego-ricardo-bonilla-villa-7179254a/) [![Email](https://img.shields.io/badge/Email-DiegoBonilla-D14836?logo=gmail)](mailto:drbv27@gmail.com)
