# Plan de Pruebas — LOYALTY

## 1. Identificación del Plan

| Campo | Detalle |
|---|---|
| **Nombre del proyecto** | LOYALTY — Calculadora de Descuentos Dinámicos |
| **Sistema bajo prueba** | Plataforma LOYALTY: Motor de descuentos (API REST S2S) + Dashboard de configuración |
| **Versión** | MVP v1.0 |
| **Fecha** | 25 de Marzo de 2026 |
| **Equipo** | Agustín Mites, Matias Regaló, Luis Ospino |

---

## 2. Contexto

### ¿Qué sistema se va a probar?

LOYALTY es una plataforma de descuentos dinámicos compuesta por dos módulos principales:

- **Motor de descuentos (API REST Stateless S2S):** recibe desde el backend del e-commerce un payload con el carrito de compras y las métricas del comprador, valida la autenticación mediante API Key y devuelve en tiempo real el descuento combinado o precio personalizado calculado.
- **Dashboard de configuración UI:** interfaz web para administradores del e-commerce que permite definir reglas de descuento (por temporada, fidelidad y tipo de producto), establecer topes máximos, gestionar API Keys (generar, rotar, revocar) y consultar logs de auditoría.

### ¿Qué problema de negocio resuelve?

Las tiendas de e-commerce B2C dependen actualmente de descuentos generales, estáticos y administrados de forma manual, lo que genera ineficiencias operativas y financieras. Este modelo aplica el mismo descuento a compradores recurrentes y a compradores de ofertas puntuales, lo que influye en los márgenes de ganancia, aumentando el costo de adquisición y fomentando que los clientes solo compren en épocas de rebaja.

LOYALTY resuelve este problema al introducir una capa de inteligencia entre el catálogo y el checkout, capaz de calcular e inyectar precios personalizados en tiempo real según la temporada, la fidelidad del cliente y el tipo de producto. Esto permite a los comercios proteger sus márgenes, aumentar la tasa de conversión y mejorar la retención de clientes frecuentes sin sacrificar rentabilidad.

---

## 3. Alcance de las Pruebas

El alcance de las pruebas para este micro-sprint incluye las historias de usuario **HU 3, HU 7, HU 8 y HU 9**. Para cada una se validarán los criterios de aceptación definidos en los escenarios descritos a continuación.

---

### HU 3 — Gestionar y validar las API Keys de cada ecommerce

**Como** Super Admin, **quiero** gestionar y validar las API Keys de cada ecommerce, **para** asegurar que solo sistemas autorizados puedan acceder a sus recursos en la plataforma.

**Criterios de Aceptación**

**Scenario: Crear una clave de acceso para un ecommerce válido**
- **Given**  que soy un Super Admin con acceso al sistema
- **And** existe un ecommerce registrado
- **When** creo una nueva clave de acceso para ese ecommerce
- **Then** el sistema genera la clave de acceso correctamente

**Scenario: Ver las claves de acceso de un ecommerce**
- **Given** que soy un Super Administrador con acceso al sistema
- **And** existen claves de acceso registradas para un ecommerce
- **When** consulto las claves de acceso de ese ecommerce
- **Then** el sistema muestra la lista de claves de acceso asociadas
- **And** oculta parte de la información de cada clave para proteger su seguridad

---

### HU 7 — Definir el tope máximo y la prioridad de descuentos

**Como** usuario de LOYALTY, **quiero** definir el tope máximo y la prioridad de descuentos, **para** proteger la rentabilidad del negocio.

**Criterios de Aceptación**

**Scenario: Configuración válida de tope y prioridad**
- **Given** existen tipos de descuento disponibles en el ecommerce
- **And** el tope máximo propuesto es un valor positivo 
- **And** la prioridad define un orden único entre descuentos
- **When** se registra la configuración de topes y prioridad
- **Then** la configuración queda vigente para el ecommerce
- **And** el motor utiliza ese orden para resolver descuentos concurrentes

**Scenario: Rechazo por tope máximo inválido**
- **Given** existen límites de negocio para proteger la rentabilidad
- **When** se intenta registrar un tope máximo menor o igual a cero
- **Then** la configuración es rechazada
- **And** se mantiene la última configuración válida

**Scenario: Rechazo por prioridad ambigua**
- **Given** la prioridad de descuentos debe ser determinística
- **When** se registra una prioridad con empates o niveles duplicados
- **Then** la configuración es rechazada
- **And** no se altera la prioridad vigente

**Scenario: Aplicación del tope ante acumulación de descuentos**
- **Given** existe una configuración vigente de tope y prioridad
- **And** una transacción califica para múltiples descuentos
- **When** el descuento total calculado supera el tope máximo
- **Then** el descuento final aplicado se limita al tope máximo configurado

---

### HU 8 — Clasificar al cliente según el payload recibido

**Como** motor de descuentos, **quiero** clasificar al cliente según el payload recibido, **para** asignar su nivel de descuento.

**Criterios de Aceptación**

**Scenario: Clasificación exitosa con payload completo y válido**
- **Given** existe una matriz de clasificación vigente 
- **And** el payload contiene todos los atributos requeridos
- **When** el motor evalúa el payload del cliente
- **Then** el cliente queda asignado a un único nivel de fidelidad
- **And** el nivel asignado corresponde a las reglas de clasificación vigentes

**Scenario: Rechazo por payload incompleto**
- **Given** existen atributos obligatorios para clasificar al cliente
- **When** se recibe un payload sin uno o más atributos obligatorios
- **Then** la clasificación es rechazada
- **And** se informa que no es posible determinar el nivel de descuento

**Scenario: Rechazo por valores fuera de dominio**
- **Given** existen reglas de validación para los atributos del payload
- **When** se recibe un payload con valores inválidos para el dominio definido
- **Then** la clasificación es rechazada
- **And** no se asigna nivel de descuento al cliente

**Scenario: Consistencia de clasificación para mismo payload**
- **Given** existe una configuración de clasificación vigente
- **When** el motor evalúa dos veces el mismo payload
- **Then** el nivel de fidelidad asignado es el mismo en ambas evaluaciones

---

### HU 9 — Enviar el carrito y recibir el precio final con descuentos aplicados

**Como** ecommerce, **quiero** enviar el carrito y recibir el precio final con descuentos aplicados, **para** mostrarle al usuario final en el checkout.

**Criterios de Aceptación**

**Scenario: Cálculo exitoso de precio final**
- **Given** el ecommerce está autorizado para consumir el servicio
- **And** el carrito contiene ítems válidos
- **And** existen reglas de descuento vigentes aplicables
- **And** existe una configuración vigente de topes y prioridad
- **And** y el carrito califica para múltiples descuentos
- **When** el ecommerce solicita el cálculo del carrito
- **Then** el servicio responde el precio final del carrito
- **And** el total incluye los descuentos aplicados según reglas vigentes y prioridades configuradas
- **And** el orden de aplicación de descuentos respeta la prioridad configurada
- **And** el descuento total no supera el tope máximo vigente
- **And** el precio final refleja ambas restricciones de negocio


**Scenario: Carrito sin descuentos aplicables**
- **Given** el ecommerce está autorizado para consumir el servicio
- **And** el carrito es válido 
- **And** no existen descuentos aplicables para ese carrito
- **When** el ecommerce solicita el cálculo del carrito
- **Then** el precio final es igual al subtotal del carrito
- **And** la respuesta indica que no hubo descuentos aplicados

**Scenario: Rechazo por carrito inválido**
- **Given** el servicio requiere estructura mínima y datos válidos de carrito
- **When** el ecommerce envía un carrito con datos incompletos o inconsistentes
- **Then** la solicitud es rechazada
- **And** no se retorna precio final hasta contar con un carrito válido

---

### Fuera del Alcance

Las siguientes historias de usuario **no serán validadas** en este ciclo, quedando pendientes para futuros sprints:

| HU | Descripción |
|---|---|
| HU 1 | Inicio y cierre de sesión |
| HU 2 | Creación de usuarios vinculados a un ecommerce |
| HU 4 | Crear, editar y eliminar reglas de temporada |
| HU 5 | Crear, editar y eliminar reglas por tipo de producto |
| HU 6 | Definir rangos de clasificación de fidelidad |
| HU 10 | Consultar descuentos aplicados en los últimos 7 días |
| HU 11 | Ver registro de cambios de reglas de descuento (Super Admin) |

Adicionalmente, quedan explícitamente fuera del alcance de este ciclo:

- Pruebas de la interfaz del Dashboard de configuración (UI end-to-end)
- Pruebas de administración de usuarios y roles (HU 1 y HU 2)
- Cualquier otra validación o funcionalidad no incluida explícitamente en la
estrategia

---

## 4. Estrategia de Pruebas

Las pruebas iniciarán con la ejecución de una Prueba de Arranque para verificar que el Motor de Descuentos (API REST S2S) responde correctamente y que el Dashboard de configuración está accesible. Si el entorno está disponible, se procederá con la ejecución de los casos de prueba de cada HU priorizados por riesgo de negocio.

Se ejecutarán los escenarios definidos para cada historia de usuario de acuerdo a sus criterios de aceptación. Si se detectan desviaciones respecto al resultado esperado, se reportará el defecto en un issue para que el equipo de desarrollo lo solucione y pueda ser re-validado en el siguiente sprint.

| Tipo de Prueba | HU Cubiertas | Objetivo | Herramienta |
|---|---|---|---|
| **Smoke — API REST S2S** | HU 8, HU 9 | Verificar conectividad básica y estructura de respuesta de los endpoints antes de ejecutar la suite automatizada. | Postman |
| **Funcional — API REST S2S** | HU 8, HU 9 | Verificar que los endpoints del Motor de Descuentos respondan correctamente en escenarios positivos y negativos, cumpliendo los criterios de aceptación definidos. | Karate |
| **Funcional — Dashboard UI** | HU 3, HU 7 | Verificar que las funcionalidades del Dashboard de configuración se implementen correctamente y cumplan los criterios de aceptación establecidos. | SerenityBDD + Cucumber |
| **Rendimiento — Motor de Descuentos** | HU 9 | Verificar que el Motor de Descuentos soporte la carga esperada de peticiones simultáneas dentro de los umbrales de tiempo de respuesta aceptables para el negocio. | k6 |
| **Regresión automatizada — CI/CD** | HU 3, HU 7, HU 8, HU 9 | Ejecutar automáticamente las suites de Karate y SerenityBDD+Cucumber ante cada PR o merge a la rama QA, garantizando que no se introduzcan regresiones. | GitHub Actions |

---

## 5. Criterios de Entrada y Salida

### 5.1 Criterios de Entrada

| # | Criterio |
|---|---|
| 1 | Código fuente disponible en el repositorio con PR aprobado |
| 2 | Ambiente de pruebas desplegado y estable |
| 3 | Base de datos de pruebas configurada y accesible |
| 4 | Endpoints de la API documentados y disponibles |
| 5 | Pruebas unitarias ejecutadas por DEV con cobertura > 80% |
| 6 | Historias de usuario y criterios de aceptación revisados y aprobados |
| 7 | Datos de prueba preparados |
| 8 | Herramientas de prueba configuradas (Karate, SerenityBDD, Cucumber, k6, Postman, Maven, Docker, GitHub Actions, GitHub Issues, IntelliJ IDEA) |

### 5.2 Criterios de Salida

| # | Criterio |
|---|---|
| 1 | Todos los criterios de aceptación validados (100% ejecutados) |
| 2 | Tasa de aprobación de casos de prueba ≥ 95% |
| 3 | Sin defectos críticos o bloqueantes abiertos |
| 4 | Defectos de severidad media resueltos o con plan de mitigación aprobado |
| 5 | Reporte de SerenityBDD generado y revisado (aplica a HU 3 y HU 7) |
| 6 | Reporte de Karate generado y revisado (aplica a HU 8 y HU 9) |
| 7 | Pruebas de rendimiento k6 ejecutadas y dentro de umbrales aceptables (aplica a HU 9) |

## 6. Entorno de Pruebas

Entorno dedicado de QA, aislado de producción, con datos controlados y sin impacto en clientes reales.


| Componente | Versión | Acceso / Uso |
|---|---|---|
| Motor de Descuentos (API REST S2S) | Docker 26.x | `http://localhost:8080` · HU 8, HU 9 · auth header `x-api-key`; sin autenticación de usuario |
| Dashboard de configuración (UI) | Docker 26.x | `http://localhost:3000` · HU 3, HU 7 · cuenta Super Admin activa; sin MFA |
| Base de datos | Docker 26.x | Solo red interna `loyalty-qa-network` · inicializada con datos de prueba |
| Windows 11 / Ubuntu (runner CI) | — | Local: ejecución manual y automatizada · CI: runner `ubuntu-latest` en GitHub Actions |
| JDK | 17 LTS (OpenJDK) | Runtime de Maven, Karate y SerenityBDD |
| Maven | 3.9.x | Ejecución de suites Karate y SerenityBDD+Cucumber |
| k6 | 0.50.x | Pruebas de rendimiento (HU 9) |
| Postman | Estable actual | Smoke tests manuales (HU 8, HU 9) |
| Google Chrome | Estable actual | Pruebas UI del Dashboard (HU 3, HU 7) |
| GitHub Actions | — | Trigger: PR o merge a rama `qa` · publica reportes Karate y SerenityBDD como artefactos |

### 6.2 Datos de prueba

- Ecommerce de prueba con ID fijo
- Dos API Keys: una activa y una revocada
- Reglas de temporada, fidelidad y tipo de producto que activan todos los escenarios
- Tope máximo positivo y prioridades únicas configuradas
- Cuenta Super Admin con credenciales conocidas por el equipo QA

> **Prerequisito de arranque:** Se verificará `GET http://localhost:8080/health → HTTP 200` en el Motor de Descuentos y que `http://localhost:3000` es accesible desde el navegador. Si alguno falla, la ejecución queda bloqueada.

---

## 7. Herramientas

| Herramienta | Versión | Categoría | Propósito específico |
|---|---|---|---|
| **Karate** | 1.4.x | Automatización API | Framework de pruebas de API REST. Automatiza los escenarios funcionales del Motor de Descuentos (HU 8, HU 9): envío de payloads, validación de respuestas HTTP, verificación de estructura JSON y flujos negativos. |
| **SerenityBDD** | 4.x | Automatización UI / Reporte | Framework de reporte y orquestación BDD. Integrado con Cucumber, genera reportes narrativos de los escenarios del Dashboard de configuración (HU 3, HU 7), documentando la evidencia de ejecución paso a paso. |
| **Cucumber** | 7.x | Especificación BDD | Motor de especificación ejecutable en Gherkin. Define los escenarios de aceptación del Dashboard (HU 3, HU 7) en lenguaje natural, conectando criterios de aceptación con la automatización de SerenityBDD. |
| **k6** | 0.50.x | Rendimiento | Herramienta de pruebas de rendimiento. Ejecuta escenarios de carga sobre el Motor de Descuentos (HU 9) para verificar el comportamiento bajo volumen de peticiones simultáneas y validar que los tiempos de respuesta se mantienen dentro de los umbrales aceptables. |
| **Postman** | Actual | Exploración manual / Smoke | Cliente HTTP para pruebas exploratorias y smoke tests manuales sobre los endpoints del Motor de Descuentos. Permite inspeccionar respuestas, manipular headers y validar contratos de la API antes de incorporar escenarios a la suite automatizada. |
| **Maven** | 3.9.x | Build y ejecución | Herramienta de construcción y gestión de dependencias. Orquesta la compilación y ejecución de las suites de Karate y SerenityBDD+Cucumber desde línea de comandos o pipeline CI/CD. |
| **Docker** | 26.x | Entorno / Infraestructura QA | Conteneriza los servicios del entorno de QA (Motor de Descuentos, base de datos) garantizando reproducibilidad y aislamiento entre ejecuciones. Permite levantar y destruir el entorno de forma consistente. |
| **GitHub Actions** | — | CI/CD | Pipeline de integración continua que dispara automáticamente la ejecución de las suites de pruebas ante cada PR o merge a la rama de QA, publicando los reportes generados como artefactos del workflow. |
| **GitHub Issues** | — | Gestión de defectos | Registro y seguimiento de defectos detectados durante la ejecución. Cada issue captura: pasos para reproducir, resultado esperado vs. obtenido, severidad y evidencia adjunta (logs, capturas). |
| **IntelliJ IDEA** | 2024.x | IDE de desarrollo | Entorno de desarrollo integrado utilizado para escribir, depurar y mantener los scripts de prueba de Karate y SerenityBDD+Cucumber. |

---