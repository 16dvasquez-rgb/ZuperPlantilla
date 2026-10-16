# AGENTS.md — Guía y Reglas del Proyecto

Este documento establece la arquitectura, las convenciones de código y las reglas que **todos los agentes de IA y desarrolladores** deben seguir al trabajar o modificar este repositorio.

---

## 📌 1. Propósito del Proyecto y Modelo de Negocio

Este repositorio desarrolla una **plataforma web de minijuegos y juegos 2D/Flash nostálgicos de calidad**, orientada a un público joven de habla hispana. 

- **Propuesta de Valor:** Ofrecer entretenimiento accesible y nostálgico con una experiencia de usuario fluida.
- **Modelo de Acceso (Registro Opcional):** Los usuarios pueden jugar sin fricciones obligatorias. Sin embargo, registrarse desbloquea funciones avanzadas de retención:
  - Guardado de progreso en otros dispositivos (multiplataforma).
  - Rankings de puntaje y torneos.
  - Sistema de logros y premios por puntuación.
- **Monetización:** Sostenibilidad financiera basada en publicidad mediante **Google AdSense**.

Al tratarse de una plataforma interactiva, **se debe priorizar la claridad, el rendimiento, el orden y las buenas prácticas sencillas**, evitando sobre-diseñar o agregar abstracciones complejas e innecesarias (principio YAGNI).

---

## 🌐 2. Reglas de Idioma y Estilo (Crítico)

| Elemento | Idioma Obligatorio | Ejemplo |
|---|---|---|
| **Código Fuente** (Variables, funciones, módulos, rutas, BD) | **Inglés** | `verify_credentials`, `register_user`, `user_service.py`, `main_bp`, `email`, `save_progress` |
| **Comentarios en Código** | **Español** | `# Verifica la contraseña con bcrypt` |
| **Interfaz de Usuario (UI HTML / Mensajes Flash / Textos de la Plataforma)** | **Español** | `"Credenciales incorrectas"`, `<h2>Bienvenido a los Minijuegos</h2>`, `"Guardar progreso"` |

---

## 🏗️ 3. Arquitectura del Proyecto

El proyecto sigue una arquitectura **App Factory + Blueprint Único (`main_bp`)** estructurada en capas simples adaptadas a la gestión de usuarios, puntajes y anuncios:

```text
Desacoplado/
├── AGENTS.md                   # Instrucciones y reglas del proyecto
├── .env                        # Variables de entorno (DB_HOST, SECRET_KEY, etc.)
├── requirements.txt            # Dependencias instaladas
├── run.py                      # Punto de entrada de la aplicación
└── app/
    ├── __init__.py              # App Factory (crea Flask app y registra main_bp)
    ├── database.py              # Proveedor de conexión a MySQL (get_db_connection)
    ├── routes/
    │   └── main.py              # Rutas HTTP, sesión, juegos y decorador @login_required
    ├── services/
    │   └── user_service.py      # Lógica de datos, SQL, cifrado bcrypt y gestión de progreso/logros
    ├── static/                  # Estilos CSS estáticos y assets de minijuegos
    └── templates/               # Plantillas HTML planas (index, login, sign-up, dashboard, sala de juegos)
```

---

## 🛡️ 4. Reglas de Desarrollo y Buenas Practices

### 4.1. Separación de Responsabilidades (SRP)
- **Rutas (`app/routes/main.py`):** Su ÚNICA función es recibir solicitudes HTTP, procesar formularios, gestionar sesiones (`session`), invocar la capa de servicios y renderizar plantillas o redirigir. **Nunca** deben contener consultas SQL directas ni hashing de contraseñas.
- **Servicios (`app/services/user_service.py`):** Su ÚNICA función es comunicarse con la base de datos MySQL, aplicar la lógica de hashing (`bcrypt`) y gestionar el almacenamiento de progresos, rankings y logros. **Nunca** deben importar `flask`, `request` ni manejar sesiones HTTP.
- **Conexión a BD (`app/database.py`):** Proporciona la función `get_db_connection()`.

### 4.2. Manejo Seguro de Recursos y Datos
- **Cierre de Conexiones:** Toda función en la capa de servicios que abra un cursor o conexión a MySQL **debe hacer `close()` dentro de un bloque `finally`**.
- **Sanitización de Datos:** Eliminar el campo `'password'` de cualquier diccionario devuelto por la capa de servicios antes de enviarlo a la capa de rutas.
- **Manejo de Excepciones:** Capturar explícitamente excepciones de base de datos como `mysql.connector.IntegrityError` (para correos duplicados) y retornar valores booleanos (`True`/`False`) o limpios a la capa de rutas.

### 4.3. Plantillas HTML, Juegos y Anuncios
- Las vistas en `app/templates/` deben ser **minimalistas y limpias**, utilizando `url_for()` dinámico de Jinja2 para las acciones de formularios, enlaces a juegos y assets.
- Las plantillas que integren juegos (2D o contenedores Flash) deben mantener espacios estructurados y responsivos para la correcta visualización de los bloques publicitarios de **Google AdSense**.

---

## ⚡ 5. Comandos de Ejecución

1. **Activar entorno virtual:**
   ```powershell
   .\venv\Scripts\activate
   ```
2. **Ejecutar servidor:**
   ```powershell
   python run.py
   ```