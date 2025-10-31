## 📊 **Tabla Comparativa de Frameworks**

| **Framework**                     | **Técnica Principal** | **Ventajas**                                                                                                                                               | **Limitaciones**                                                                                                       | **Casos Vulnerables**                                                   |
| --------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Laravel Eloquent (PHP)**        | ORM + Query Builder   | - Sintaxis fluida y segura con consultas parametrizadas.<br>- Escapado automático de variables.<br>- Integración con validadores y middleware.             | - Posible riesgo si se usan consultas `DB::raw()` sin validación.<br>- Menor rendimiento en consultas muy complejas.   | Uso de consultas nativas sin parámetros; malas prácticas con `raw SQL`. |
| **Django ORM (Python)**           | ORM + Raw Queries     | - Seguridad por defecto en consultas ORM.<br>- Validación de campos en modelos.<br>- Integración nativa con middleware CSRF y sanitización.                | - Vulnerable si se emplean `RawSQL()` o concatenaciones manuales.<br>- Limitaciones con consultas SQL muy específicas. | Mal uso de `extra()` o `RawSQL()` en vistas o modelos.                  |
| **Spring JPA (Java)**             | ORM + Hibernate       | - Implementa JPA estándar con Hibernate.<br>- Soporta parámetros nombrados y tipados.<br>- Integración con validaciones por anotaciones (Bean Validation). | - Requiere configuración avanzada.<br>- Vulnerable si se usa `createQuery()` sin parámetros.                           | Consultas HQL o SQL dinámicas concatenadas manualmente.                 |
| **Ruby on Rails (Ruby)**          | ActiveRecord          | - Consultas parametrizadas por defecto.<br>- Protección automática contra inyecciones en `where`, `find_by`.                                               | - Vulnerable al usar `find_by_sql` o `update_all` sin parámetros.<br>- Riesgos en metaprogramación mal utilizada.      | Uso de interpolación de strings dentro de consultas SQL.                |
| **ASP.NET Entity Framework (C#)** | Entity Framework ORM  | - Mapeo fuerte entre objetos y tablas.<br>- Consultas LINQ seguras por diseño.<br>- Protección automática ante concatenación directa.                      | - Vulnerable si se ejecutan comandos con `SqlQuery()` o `ExecuteSqlCommand()`.                                         | Inserción de SQL dinámico sin parámetros o concatenaciones.             |

---

## 🧠 **Matriz de Técnicas de Prevención**

| **Técnica**                        | **Efectividad**                                        | **Complejidad**                                     | **Rendimiento**                                           | **Casos de Uso**                                                           |
| ---------------------------------- | ------------------------------------------------------ | --------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Prepared Statements**            | 🔹 Muy alta (previene casi todas las inyecciones SQL)  | Baja – sencilla de implementar                      | Alta (rendimiento nativo optimizado)                      | CRUD clásicos en PHP, Java, Node.js; aplicaciones con acceso directo a BD. |
| **ORM (Object Relational Mapper)** | Alta (protege automáticamente si se usa correctamente) | Media – requiere comprensión del framework          | Media-Alta (puede impactar en grandes volúmenes de datos) | Aplicaciones empresariales y proyectos de mediana a gran escala.           |
| **Validación de Input**            | Media-Alta (depende de la lógica aplicada)             | Media (necesita reglas específicas)                 | Alta (mínimo impacto)                                     | Formularios, APIs REST, autenticación de usuarios.                         |
| **Web Application Firewall (WAF)** | Alta (bloquea patrones conocidos de ataque)            | Media-Alta (requiere configuración y mantenimiento) | Media (puede generar latencia mínima)                     | Aplicaciones públicas o de alto tráfico; protección de endpoints externos. |
| **Análisis Estático (SAST)**       | Media (detecta código vulnerable antes de compilar)    | Alta (requiere herramientas especializadas)         | Alta (no afecta ejecución)                                | Integración en pipelines CI/CD; auditorías de seguridad preventiva.        |

---

📌 **Conclusión Rápida del Análisis Comparativo:**

* Los frameworks modernos (Eloquent, Django, Spring JPA, etc.) **implementan defensas automáticas**, pero siguen siendo vulnerables si se usan consultas nativas sin parámetros.
* La combinación ideal en entornos profesionales es:
  🔸 *ORM + Validación de Entrada + WAF + Auditoría continua (SAST/DAST)*
* Las **consultas preparadas** siguen siendo la técnica **más eficiente y segura** en escenarios donde se interactúa directamente con SQL.

---