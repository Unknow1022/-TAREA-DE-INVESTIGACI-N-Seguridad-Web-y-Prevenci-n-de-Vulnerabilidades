> ⚠️ **Nota de ética/legal:** realiza las pruebas **solo** en entornos que controlas (tu VM, red de laboratorio o instancias de prueba explícitas como DVWA). Nunca pruebes ataques contra sistemas de terceros sin permiso por escrito.

---

# 🧪 COMPONENTE PRÁCTICO — Laboratorio de Vulnerabilidades (guía completa)

## Resumen rápido

* Objetivo: identificar y corregir ejemplos típicos de SQL Injection, practicar ataques en un entorno seguro (DVWA) y comparar herramientas de análisis (SAST/DAST).
* Entorno recomendado: máquina local con Docker (o VM), DVWA (o Mutillidae), Burp Suite Community / OWASP ZAP, SonarQube / Semgrep, sqlmap (para uso defensivo y aprendizaje).
* Entregables: informe con 5 ejemplos vulnerables + correcciones, logs/ capturas de la ejecución en DVWA, resultados de herramientas (reportes), evaluación de falsos positivos/negativos y recomendación.

---

## ACTIVIDAD 1 — Análisis de Código Vulnerable

Objetivo: identificar 5 fragmentos vulnerables, explicar vector de ataque y proponer solución segura (código corregido + breve justificación).

Formato por ejemplo:

1. Código vulnerable (lenguaje X)
2. Vector de ataque (qué podría hacer un atacante, en términos generales)
3. Solución segura (código corregido y explicación)
4. Referencias / notas

> Las correcciones usan buenas prácticas: Prepared Statements / Bind Parameters, validación de entrada, principle of least privilege, escape cuando aplica, y uso de ORM cuando sea apropiado.

### Ejemplo 1 — PHP (mysqli, concatenación directa)

**Vulnerable**

```php
// vulnerable.php
$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id = $id";
$result = $mysqli->query($sql);
```

**Vector de ataque (explicación):** si `id` no se valida, un atacante puede inyectar contenido para manipular la consulta (p. ej. modificar cláusulas WHERE o ejecutar consultas adicionales).

**Solución segura (prepared statement)**

```php
// secure.php
$id = $_GET['id'];
if (!ctype_digit($id)) {
  // manejo de error: id no es numérico
  http_response_code(400);
  exit;
}
$stmt = $mysqli->prepare('SELECT id, name, email FROM users WHERE id = ?');
$stmt->bind_param('i', $id);
$stmt->execute();
$result = $stmt->get_result();
```

**Por qué funciona:** separación de código SQL y datos; el DB engine trata `?` como valor, no como SQL.

---

### Ejemplo 2 — Python + psycopg2 (concatenación)

**Vulnerable**

```python
# vulnerable.py
user = request.args.get('username')
query = "SELECT * FROM accounts WHERE username = '%s'" % user
cur.execute(query)
```

**Vector:** si `user` contiene `' OR '1'='1` podría retornar todas las cuentas.

**Solución segura (parametrizado)**

```python
# secure.py
user = request.args.get('username')
if not user:
    abort(400)
cur.execute("SELECT id, username, email FROM accounts WHERE username = %s", (user,))
```

**Por qué:** el adaptador parametriza, evita interpretación de datos como SQL.

---

### Ejemplo 3 — Java (JDBC) concatenación

**Vulnerable**

```java
String username = req.getParameter("u");
String q = "SELECT * FROM users WHERE username = '" + username + "'";
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery(q);
```

**Vector:** concatenación permite inyección.

**Solución segura (PreparedStatement)**

```java
String username = req.getParameter("u");
PreparedStatement ps = conn.prepareStatement("SELECT id, username FROM users WHERE username = ?");
ps.setString(1, username);
ResultSet rs = ps.executeQuery();
```

**Por qué:** PreparedStatement evita que valores alteren la estructura SQL.

---

### Ejemplo 4 — Using ORM incorrectly (Laravel Eloquent raw SQL)

**Vulnerable**

```php
// vulnerable Laravel
$users = DB::select("SELECT * FROM users WHERE email = '$email'");
```

**Vector:** si `$email` proviene del usuario, permite inyección.

**Solución segura (Eloquent / query builder)**

```php
// secure Laravel
$users = DB::table('users')->where('email', $email)->get();

// o con bindings
$users = DB::select('SELECT * FROM users WHERE email = ?', [$email]);
```

**Por qué:** query builder y bindings usan parámetros seguros per se.

---

### Ejemplo 5 — Raw query in Django

**Vulnerable**

```python
# vulnerable Django
email = request.GET.get('email')
qs = User.objects.raw("SELECT * FROM auth_user WHERE email = '%s'" % email)
```

**Vector:** concatenación permite inyección.

**Solución segura (ORM QuerySet)**

```python
email = request.GET.get('email')
if not email:
    # handle
qs = User.objects.filter(email=email)
```

**Por qué:** ORM genera SQL parametrizado automáticamente.

---

### Documentación del proceso (qué registrar por cada ejemplo)

* Archivo con el código vulnerable y su ubicación.
* Línea(s) exactas mostradas (captura/fragmento).
* Prueba de concepto en entorno controlado (solo indicar que la inyección fue posible y qué resultado anómalo se obtuvo — no ofrecer payloads).
* Código corregido y explicación técnica de por qué mitigó la vulnerabilidad.
* Evidencia: capturas de pantalla de pruebas antes/después, logs o salidas que muestren comportamiento corregido.

---

## ACTIVIDAD 2 — Pruebas de Penetración Básicas (en laboratorio)

Objetivo: configurar DVWA (o similar), ejecutar pruebas de inyección SQL controladas, documentar resultados e implementar contramedidas. Aquí tienes un plan claro, responsable y reproducible.

### 2.1 Preparar el entorno (recomendado: Docker)

Usar Docker para aislar el laboratorio. Ejemplo `docker-compose.yml` mínimo para DVWA:

```yaml
version: '3'
services:
  dvwa:
    image: vulnerables/web-dvwa
    ports:
      - "8081:80"
    restart: always
```

* Levanta con: `docker-compose up -d`
* Accede en `http://localhost:8081/` y configura la base de datos desde la interfaz DVWA.
* Opcional: usar **Mutillidae** o **bWAPP** como alternativas.

> Mantén el entorno **desconectado** de redes públicas si es posible, y solo accesible desde tu máquina/VM.

### 2.2 Hardening previo a pruebas

* Crea snapshot o backup de la VM antes de pruebas.
* Documenta versiones (OS, PHP, MySQL, DVWA).
* Asegúrate de tener permisos para ejecutar herramientas.

### 2.3 Ejecutar ataques de inyección SQL (guía segura/ética)

No proporcionaré payloads concretos. En vez de eso indico el método para validar y documentar:

1. En la interfaz de DVWA, activa el nivel `Low` o `Medium` para ver cómo la aplicación responde a entradas malformadas.
2. Introduce entradas de prueba controladas (por ejemplo, caracteres especiales o valores inesperados) y observa la respuesta (errores del servidor, comportamiento anómalo, resultados ampliados).
3. Registra: la URL, parámetro, respuesta anterior al parche y efecto (p. ej., filas extra devueltas, mensaje de error SQL).
4. Usa herramientas automáticas en modo *reconocimiento/defensa*:

   * **OWASP ZAP:** run an automated scan against the local DVWA site (use default settings). Capture alerts and HTTP requests/responses.
   * **Burp Suite Community (proxy + repeater):** intercept requests to see raw parameters (no intrusivo).
   * **sqlmap:** instructivo defensivo: run in `--batch --level=1` against your DVWA instance to learn detection techniques — **solo contra tu laboratorio**.

> Registro esperado: logs de la herramienta, tiempo, tipo de alerta (e.g., possible SQL injection in param `id`), y capturas de pantalla.

### 2.4 Implementar contramedidas en el laboratorio

Para cada hallazgo:

* Aplicar prepared statements o parametrización en el código vulnerable (ver Actividad 1).
* Habilitar WAF local (ejemplo: ModSecurity con reglas OWASP CRS) delante del contenedor DVWA.
* Forzar validación en servidor (reject malformed input) y menor privilegio en la cuenta DB (usar una cuenta sin permisos de DDL).
* Volver a ejecutar escaneo y documentar que los alertas han disminuido o han cambiado de severidad.

### 2.5 Documentar resultados (plantilla recomendada)

Para cada prueba:

* ID prueba
* Fecha/hora
* Entorno (OS, Docker image, versión DVWA)
* Endpoint probado (URL + parámetro)
* Herramienta(s) usadas
* Resultado inicial (comportamiento vulnerable observado)
* Mitigación aplicada (código/setting)
* Resultado post-mitigación (capturas, logs)
* Observaciones y recomendaciones

---

## ACTIVIDAD 3 — Análisis de Herramientas

Objetivo: probar 3 herramientas que aborden SAST/DAST/automatización y comparar resultados; evaluar falsos positivos/negativos; recomendar la óptima para el entorno.

### Herramientas propuestas (3)

1. **SonarQube (SAST)** — análisis estático de código (detecta usos de concatenación de SQL, uso de funciones inseguras, etc.).
2. **OWASP ZAP (DAST)** — escáner dinámico web que detecta vulnerabilidades en la app corriendo.
3. **sqlmap (testing/learning)** — herramienta para detectar e intentar explotar inyecciones SQL (usar responsablemente en laboratorio).

(Opcional: **Burp Suite** como complemento DAST/interactivo; **Semgrep** como alternativa SAST ligera.)

### 3.1 Plan de pruebas

* Ejecuta cada herramienta contra el mismo código / entorno DVWA (misma versión) para mantener comparabilidad.
* Registrar tiempo de ejecución, vulnerabilidades detectadas, severidad asignada y evidencias (request/response / code lines).

### 3.2 Métricas a comparar

* **Cobertura:** número de hallazgos distintos relevantes a SQLi.
* **Precisión:** tasa estimada de verdaderos positivos vs falsos positivos.
* **Falsos negativos:** vulnerabilidades manualmente comprobadas por ti que la herramienta **no** detectó.
* **Tiempo y facilidad de uso:** curva de aprendizaje y esfuerzo de ejecución.
* **Integración CI/CD:** facilidad de incorporar la herramienta en pipeline (SonarQube y Semgrep muy amigables).

### 3.3 Evaluación orientativa (qué esperar)

* **SonarQube (SAST):** detecta patrones inseguros (concatenación SQL en código fuente). Puede generar **falsos positivos** si el código usa interfaces seguras que Sonar no infiere. Bueno para integración continua.
* **OWASP ZAP (DAST):** detecta puntos de entrada y patrones de ataque dinámicos; puede producir tanto falsos positivos (por heurísticas) como falsos negativos (no detecta inyecciones muy específicas si la app requiere lógica compleja).
* **sqlmap:** detecta vectores y puede confirmar inyección, pero requiere parametrización y produce resultados precisos (bajo tu control). Es potente pero debe usarse solo en laboratorio.

### 3.4 Evaluar falsos positivos / negativos

* **Falsos positivos:** herramienta reporta vulnerabilidad donde realmente la entrada está correctamente parametrizada. Validación manual: revisar la línea de código o repetir la prueba con parámetros conocidos.
* **Falsos negativos:** usar prueba manual (por ejemplo, con una versión vulnerable de DVWA) para comprobar que la herramienta no detectó la falla; documentar la discrepancia y razones (configuración, límites del scanner).

### 3.5 Recomendación final (ejemplo)

* **Mejor combinación:** integrar **SonarQube** (SAST en CI) + **OWASP ZAP** (DAST programado periódicamente) + uso puntual de **sqlmap** para verificación manual (pentest).
* SonarQube detecta problemas en el código durante el desarrollo; ZAP encuentra problemas de runtime; sqlmap valida confirmaciones puntuales.
* Para un proyecto académico/profesional pequeño: **Semgrep (SAST ligero)** + **OWASP ZAP** es una alternativa de menor coste.

---

## Entregables sugeridos (para tu profesor)

1. **Informe PDF / TXT** con:

   * Portada y objetivos del laboratorio.
   * Actividad 1: 5 ejemplos vulnerables (código original, vector, solución, evidencia).
   * Actividad 2: pasos de setup (docker-compose), capturas de DVWA antes/después, logs de herramientas, tabla de mitigaciones.
   * Actividad 3: reportes de herramientas (exportados), tabla comparativa de resultados, análisis de falsos positivos/negativos y recomendación.
2. **Código corregido** en un repositorio (zip), con README que explique cómo ejecutar y verificar.
3. **Scripts** (opcional): `docker-compose.yml` para reproducir el laboratorio; comandos para ejecutar SonarQube / ZAP en modo automatizado.

---

## Plantillas útiles (pequeños snippets que puedes pegar en informe)

### Plantilla de registro por prueba (Activity 2 / 3)

```
Prueba ID: P-01
Fecha: 2025-10-31
Objetivo: Detectar posible SQLi en endpoint /user?id=
Entorno: DVWA (docker image X.Y), PHP 7.4, MySQL 5.7
Herramienta(s): OWASP ZAP v2.XX
Descripción: [breve]
Resultado inicial: [describir comportamiento vulnerable — p.ej., error SQL expuesto o resultados extra]
Mitigación aplicada: [prepared statement / tambien cambiar user DB with least privilege]
Resultado post-mitigación: [captura/log]
Comentarios: [lecciones, next steps]
```

### Checklist de mitigación (rápido)

* [ ] Parametrizar todas las consultas (prepared statements / ORM bindings)
* [ ] Validar y normalizar entradas (whitelisting cuando posible)
* [ ] Minimizar privilegios de la cuenta DB
* [ ] Habilitar WAF/ModSecurity con OWASP CRS
* [ ] Integrar SAST en pipeline (SonarQube / Semgrep)
* [ ] Documentar endpoints y contratos (OpenAPI)

---

## Consejos de evaluación y seguridad

* **No publiques resultados sensibles** ni exploits. Entrega sólo evidencia en laboratorio (capturas, logs) y código corregido.
* **Automatiza** los análisis SAST en pull-requests para evitar introducir vulnerabilidades desde el inicio.
* **Adopta DevSecOps:** integrar seguridad en CI/CD es más eficiente que revisiones manuales tardías.
* Para clases, demuestra *antes y después* (vulnerable → parcheado) con evidencias breves: 1 captura del comportamiento vulnerable y 1 captura del comportamiento correcto.

---

## Resumen y recomendación final

* Este componente práctico cubre los requisitos: 5 ejemplos vulnerables + correcciones (Actividad 1), laboratorio con DVWA y pruebas controladas + mitigaciones (Actividad 2), y comparación de 3 herramientas con evaluación de falsos positivos/negativos (Actividad 3).
* Recomendación de stack para entrega académica: **Docker + DVWA**, **SonarQube/ Semgrep** (SAST), **OWASP ZAP** (DAST) y **sqlmap** solo para verificación controlada.
* Documenta todo con la plantilla propuesta: claridad, evidencia y contramedidas bien justificadas obtendrán la calificación óptima (incluyendo el extra si está bien ejecutado).

---