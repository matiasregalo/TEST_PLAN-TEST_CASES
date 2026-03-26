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
| **Funcional — API REST S2S** | HU 8, HU 9 | Verificar que los endpoints del Motor de Descuentos respondan correctamente en escenarios positivos y negativos, cumpliendo los criterios de aceptación definidos. | Karate |
| **Funcional — Dashboard UI** | HU 3, HU 7 | Verificar que las funcionalidades del Dashboard de configuración se implementen correctamente y cumplan los criterios de aceptación establecidos. | SerenityBDD + Cucumber |
| **Rendimiento — Motor de Descuentos** | HU 9 | Verificar que el Motor de Descuentos soporte la carga esperada de peticiones simultáneas dentro de los umbrales de tiempo de respuesta aceptables para el negocio. | k6 |