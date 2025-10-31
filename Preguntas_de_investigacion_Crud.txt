---

## 🔹 PREGUNTAS PRINCIPALES

### 1. ¿Qué patrón de estado ofrece mejor balance para un CRUD estándar?

**Respuesta:**
El patrón **Riverpod** ofrece el mejor equilibrio entre simplicidad, escalabilidad y testabilidad para un CRUD típico. Supera a Provider en modularidad y legibilidad, mientras que BLoC es más robusto pero excesivo para apps medianas. Riverpod permite un flujo reactivo claro, evita el `context` y facilita pruebas unitarias, ideal para manejar formularios, validaciones y peticiones HTTP sin boilerplate.

---

### 2. ¿Cómo estructurar `lib/` para mantener escalabilidad y testabilidad?

**Respuesta:**
La estructura recomendada es **feature-first modular**, es decir, cada módulo (por ejemplo, `users/`, `products/`) contiene sus propias carpetas `data/`, `domain/`, `presentation/`.
Ejemplo:

```
lib/
 └─ features/
     ├─ users/
     │   ├─ data/
     │   ├─ domain/
     │   └─ presentation/
 ├─ core/ (utilidades, temas, constantes)
 └─ app.dart (configuración general)
```

Esto mantiene independencia entre módulos, favorece la inyección de dependencias (DI) y permite pruebas unitarias aisladas.

---

### 3. ¿Qué cliente HTTP es más adecuado considerando robustez y DX (developer experience)?

**Respuesta:**
**Dio** es el cliente más completo para Flutter en 2025. Soporta interceptores, cancelación de requests, retries, timeouts, manejo de multipart y logs de red.
Aunque `http` es más liviano, requiere código adicional para middleware y manejo de errores. `Chopper` es útil en arquitecturas más grandes, pero su curva de aprendizaje es mayor. Para un CRUD estándar, **Dio** ofrece la mejor combinación de productividad y control.

---

### 4. ¿Qué controles mínimos de seguridad deben existir en `backend/`?

**Respuesta:**

* Validar todos los parámetros recibidos (nunca confiar en el cliente).
* Usar **prepared statements** o **ORM** para evitar inyección SQL.
* Autenticación basada en **JWT** con expiración corta y refresh tokens.
* Implementar **CORS controlado** (dominios permitidos específicos).
* Forzar HTTPS y sanitizar respuestas de error.
* Limitar solicitudes por IP (rate limiting) y manejar sesiones seguras con `SameSite` y `HttpOnly`.

---

## 🔹 PREGUNTAS ESPECÍFICAS

### 5. ¿Cómo manejar reintentos, timeouts y errores del lado de Flutter?

**Respuesta:**
Dio permite configurar interceptores globales para reintentos automáticos, control de timeouts y parseo de errores.
Ejemplo:

```dart
final dio = Dio(BaseOptions(connectTimeout: Duration(seconds: 10)));
dio.interceptors.add(RetryInterceptor());
```

A nivel de UX, mostrar `SnackBars` o `Dialogs` personalizados ayuda a informar errores y permitir reintentos manuales. Además, manejar errores de red con estados específicos (`loading`, `error`, `empty`) mejora la experiencia del usuario.

---

### 6. ¿Cómo versionar y documentar endpoints para evitar roturas?

**Respuesta:**
Se recomienda usar rutas versionadas (`/api/v1/users`, `/api/v2/products`) y documentar los endpoints con **OpenAPI (Swagger)**.
El backend debe mantener al menos una versión estable mientras la nueva se adopta. En PHP (por ejemplo con Laravel o Slim), separar controladores por versión y usar respuestas JSON estandarizadas (`{ "status": "ok", "data": {...} }`) garantiza compatibilidad.

---

### 7. ¿Qué métricas de rendimiento y UX son críticas en móviles?

**Respuesta:**

* **FPS estable (60+)** sin jank en scrolls y animaciones.
* **Tiempo de render inicial (TTI)** < 2s.
* **Tiempo de respuesta API** < 500 ms promedio.
* **Memoria en uso** < 150 MB en dispositivos medianos.
* En UX: feedback inmediato, accesibilidad (contraste, texto legible), y carga progresiva.
  Estas métricas pueden medirse con herramientas como **Flutter DevTools** y **Firebase Performance Monitoring**.

---

### 8. ¿Qué técnica de cacheo es más efectiva para listas paginadas?

**Respuesta:**
Usar **cache local con `Hive` o `sqflite`** combinada con sincronización selectiva.
La app guarda la última respuesta (página y fecha), y actualiza solo si el backend devuelve un cambio o se detecta conexión.
Además, usar `ListView.builder` con `AutomaticKeepAlive` evita renders innecesarios, mejorando rendimiento en listas largas.

---

## 🔹 ANÁLISIS CRÍTICO

### 9. ¿Cuándo una arquitectura más compleja empeora DX sin ganar calidad?

**Respuesta:**
Cuando el proyecto no requiere escalabilidad alta, usar patrones pesados como Clean Architecture + BLoC + Repository puede ralentizar el desarrollo sin beneficios reales.
Para CRUDs simples o medianos, un enfoque modular con Riverpod y repositorios livianos logra el mismo orden con menor fricción.

---

### 10. ¿Qué trade-offs existen entre rapidez de entrega vs. mantenibilidad?

**Respuesta:**
Una entrega rápida suele usar menos capas y menos pruebas, reduciendo mantenibilidad a largo plazo. En cambio, una arquitectura bien organizada requiere más tiempo inicial pero facilita escalar y corregir errores.
El equilibrio depende del contexto: para proyectos académicos o MVPs, priorizar productividad; para proyectos de empresa, priorizar mantenibilidad.

---

### 11. ¿Cómo balancear seguridad con usabilidad y rendimiento en móvil?

**Respuesta:**
Seguridad fuerte no debe comprometer la fluidez. Por ejemplo, tokens cifrados con `flutter_secure_storage` y validaciones livianas en cliente aseguran protección sin penalizar la UX.
El backend debe manejar autenticación y rate limiting, mientras que el cliente gestiona sesiones y expiraciones sin bloquear al usuario.
El principio clave: **seguridad invisible pero efectiva**.

---
