GoPizza – Implementación Inicial con Repositorios InMemory

Este proyecto representa la primera fase funcional del sistema GoPizza.
Actualmente, el backend y el frontend están operativos, las órdenes se crean correctamente y las pizzas aparecen en la interfaz.
El sistema funciona utilizando repositorios InMemory y el generador automático de órdenes incluido en la plantilla.

1. Estado del Proyecto

Backend ejecutándose con Spring Boot (Java 21).

Frontend Vaadin disponible en http://localhost:8082/.

Órdenes generadas automáticamente mediante FakeOrderGenerator.

Caso de uso para crear órdenes implementado.

Controlador conectado correctamente al caso de uso.

Repositorios InMemory activos para órdenes y pizzas.

API REST funcionando según lo definido.

2. Arquitectura Actual
2.1 CreateOrderUseCase

Ubicación:

src/ordering/main/com/gopizza/ordering/order/application/create/


Responsabilidades:

Crear una orden usando Order.create(id, pizzas).

Guardar la orden en InmemoryOrderRepository.

Finalizar el proceso sin publicar eventos (fase inicial).

2.2 OrderPostController
POST /api/orders


Acciones:

Recibe CreateOrderRequest.

Ejecuta el caso de uso CreateOrderUseCase.

Devuelve HTTP 204 No Content.

2.3 Repositorios InMemory

Orden:

InmemoryOrderRepository


Pizza:

InmemoryPizzaRepository


Implementan:

save()

search()

searchAll()

Datos almacenados en memoria durante la ejecución.

2.4 FakeOrderGenerator

Componente encargado de generar órdenes automáticamente para permitir visualizar datos en el frontend sin necesidad de un cliente externo.

3. Ejecución del Sistema
3.1 Levantar backend:
./gradlew bootRun

3.2 Frontend:

Abrir en navegador:

http://localhost:8082/

3.3 Endpoints disponibles:

Listar órdenes:

GET /api/orders


Crear orden manual:

POST /api/orders
Content-Type: application/json

{
  "id": "order-001",
  "pizzas": [
    {
      "id": "pz-1",
      "type": "MARGHERITA",
      "size": "SMALL",
      "completed": false,
      "price": 10.0
    }
  ]
}

4. Flujo Operativo Actual

FakeOrderGenerator crea órdenes cada cierto tiempo.

El generador envía las órdenes al OrderPostController.

El controlador ejecuta el caso de uso y registra la orden.

Las órdenes se almacenan en memoria.

El frontend consulta continuamente las órdenes.

Las órdenes y pizzas aparecen en la interfaz.

5. Próximas Fases (pendientes)

Migración a MongoDB usando MongoTemplate.

Implementación de eventos:

OrderCreatedEvent

PizzaCreatedEvent

Implementación de casos de uso reactivos:

Crear pizzas al recibir OrderCreatedEvent.

Completar órdenes al recibir PizzaCreatedEvent.

Añadir suscriptores Kafka/RabbitMQ.

Reemplazar almacenamiento InMemory.