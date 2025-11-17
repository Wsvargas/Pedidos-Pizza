# GoPizza – Prueba Técnica Back-End (Java + Spring Boot)

GoPizza es una pizzería en Quito que implementó una máquina automatizada capaz de preparar pizzas de forma automática a partir de órdenes digitales.  
El objetivo de esta prueba es modelar el flujo entre el departamento que recibe los pedidos y la máquina que los procesa, aplicando Java 21, Spring Boot y una arquitectura basada en eventos internos.

---

## Descripción del Flujo

El sistema está dividido en dos módulos:

### 1. Departamento de Órdenes (Ordering)

- Recibe y registra una orden.
- Envía la orden completa al módulo de producción.
- Escucha notificaciones de pizzas completadas.
- Actualiza progresivamente el estado de la orden hasta finalizarla.

### 2. Departamento de Producción (Production)

- Recibe la orden enviada desde el departamento de órdenes.
- Procesa cada pizza individualmente.
- Cuando termina una pizza, envía un evento notificando su finalización.

---

## Estructura del Proyecto

Todo el código fuente se encuentra dentro del directorio:

```
src/ contiene la lógica de dominio, aplicación e infraestructura.
apps/ contiene únicamente las capas de entrada (controladores y endpoints).
target/** ignorar
src/main/** ignorar
```

---

## Requisitos de Implementación

Antes de iniciar el desarrollo, es obligatorio ejecutar:

```bash
docker compose up -d
```

Este comando levanta todos los servicios necesarios definidos en el `docker-compose.yml`.  
La configuración del proyecto ya está preparada para apuntar a estos servicios mediante `application.yml`.

---

## 1. Casos de Uso

Establezcan un nombre semántico a cada caso de uso.

## 1. Casos de Uso

Cada caso de uso debe tener un nombre semántico.

### Caso de uso que crea una orden

- Crea una orden usando `Order.create(...)`.
- Guarda la orden en su repositorio.
- Publica `OrderCreatedEvent` usando `KafkaPublisher` o `RabbitPublisher`.

**IMPORTANTE**

Se debe usar el caso de uso que crea la orden dentro del `OrderPostController`.

Este controlador **no debe modificarse** bajo ninguna circunstancia:  
debe mantener exactamente el mismo _path_ (`/api/orders`) y la misma firma del método:

```java
public ResponseEntity<Void> create(@RequestBody CreateOrderRequest request)
```

La razón es que un script automático envía órdenes directamente a este endpoint.  
Por lo tanto, **lo único que debe hacerse** es invocar desde este método el caso de uso que crea la orden, usando los valores recibidos en:

```
request.id()
request.pizzas()
```

### Caso de uso que crea una pizza

- Crea pizzas usando `Pizza.create(...)`.
- **Simula el tiempo de preparación real de la pizza en segundos**.
- Después de esperar esos segundos, debe **establecer ese mismo valor como `creationTimeMinutes`** al crear la pizza.
- Guarda la pizza creada en el repositorio.
- Publica `PizzaCreatedEvent` al terminar de crear la pizza.

### Caso de uso que completa progresivamente una orden (de pizza en pizza)

- Recibe `orderId` y `pizzaId`.
- Busca la orden correspondiente en el repositorio.
- Completa **solo esa pizza específica** usando `order.completePizza(...)`.  
  (Este método del dominio actualiza el estado interno de la orden y va marcando cada pizza como completada. Si todas las pizzas están completadas, la orden pasa a estado `COMPLETED`.)
- Finalmente, guarda la orden actualizada en el repositorio.

---

## 2. Subscribers obligatorios

### CompleteOrderPizzaOnPizzaCreated

Debe:

- Recibir el evento enrutado con `pizza.created` y con el record `PizzaCreatedEvent`.
- Deserializar.
- Llamar y ejecutar `El caso de Uso que completa progresivamente una orden (de pizza en pizza)`.

### CreatePizzaOnOrderCreated

Debe:

- Recibir el evento enrutado con `order.created` y con el record `OrderCreatedEvent`.
- Deserializar.
- Iterar cada pizza de la orden.
- Crear cada pizza mediante `El caso de Uso que crea una pizza`.

---

## 3. Publicación de eventos

Todos los eventos deben publicarse usando uno de los publishers provistos:

- `KafkaPublisher`
- `RabbitPublisher`

Los eventos que deben emitirse y deserializarse son:

- `OrderCreatedEvent`
- `PizzaCreatedEvent`

---

## 4. Métodos de dominio obligatorios

Deben utilizar los métodos ya implementados en el dominio:

## 5. Casos de uso de búsqueda (Searchers) — No deben modificarse

El proyecto incluye casos de uso de lectura ya implementados para consultar órdenes y pizzas.  
**Estos casos de uso NO deben modificarse**, porque son utilizadas por el frontend básico incluido en el proyecto.

---

## 6. Resultado Final de la Prueba

Cuando hayan terminado de implementar el flujo completo, deben abrir:

```
http://localhost:8082/
```

Y deberán ver la lista de órdenes y pizzas,
crearse y completarse automáticamente.

---

## Se evaluará:

- Claridad de la arquitectura
- Correcto uso de eventos internos
- Flujo completo funcionando
- Limpieza del código
- Buenas prácticas
