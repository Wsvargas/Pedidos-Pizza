Correcciones y Ejecución del Proyecto GoPizza (Versión Actual)

Este documento resume los ajustes realizados sobre el proyecto base de GoPizza para permitir su ejecución correcta con frontend y backend en modo local utilizando repositorios InMemory.

1. Problemas Iniciales del Proyecto Base

Al clonar y ejecutar el proyecto original, el frontend no cargaba en http://localhost:8082/ y los controladores no funcionaban. Esto se debía a:

El proyecto estaba configurado para usar KafkaPublisher, pero Kafka no está levantado ni configurado en el entorno local.

La presencia de beans relacionados con Kafka provocaba que el contexto de Spring no inicializara correctamente.

Sin evento publicado ni repositorio funcional, las órdenes y pizzas aparecían vacías en el frontend.

Para poder arrancar el sistema y validar flujo básico, fue necesario desactivar la publicación de eventos y utilizar repositorios InMemory.

2. Solución Aplicada
2.1 Reemplazo parcial del flujo de creación de órdenes

Se creó la clase:

CreateOrderUseCase


Ubicada en:

com.gopizza.ordering.order.application.create


Este caso de uso actualmente:

Recibe el ID y la lista de pizzas.

Crea la orden usando el método de dominio Order.create(...).

Guarda la orden usando InmemoryOrderRepository.

No publica eventos (debido a que KafkaPublisher no está operativo).

Esto permite que el frontend muestre correctamente las órdenes generadas desde el backend.

2.2 Activación del repositorio InMemory existente

El proyecto ya incluía:

InmemoryOrderRepository
InmemoryPizzaRepository


Ambos funcionan sin dependencias externas.
Se confirmó que son funcionales para pruebas locales y permiten validar el motor interno del dominio sin infraestructura adicional.

3. Corrección del Error del Frontend

El frontend no cargaba debido a:

La dependencia hacia Kafka impedía que Spring Boot inicializara el ApplicationContext.

Al no publicarse eventos ni procesarse subscriptores, el sistema no generaba datos y el frontend se mostraba vacío.

Tras desactivar la publicación de eventos y confirmar que se usan los repositorios InMemory, el frontend vuelve a funcionar y muestra órdenes y pizzas correctamente.

4. Eliminación Temporal de Kafka

KafkaPublisher no puede utilizarse en esta fase por dos razones:

No existe un clúster Kafka local definido en el docker-compose.yml.

El publisher fallaba al intentar resolver configuración que no está disponible.

Esto ocasionaba errores de inicialización de beans y detenía todo el sistema.

Por ese motivo, la solución fue:

No usar KafkaPublisher.

Evitar la publicación de eventos hasta que se integre RabbitMQ o Kafka con el entorno real de la prueba.

5. Ejecución Correcta del Proyecto
5.1 Requisitos

Java 21

Node y npm (solo para el frontend, ya incluido en Vaadin)

Docker Desktop (solo si luego se quiere integrar cola de mensajes)

5.2 Comandos de ejecución

Desde la raíz del proyecto:

./gradlew bootRun


Cuando el proceso termine de compilar, abrir:

http://localhost:8082/


Allí se visualizará el frontend Vaadin, que ya muestra:

Órdenes creadas

Pizzas de cada orden

Esto confirma que la funcionalidad de dominio está operativa con InMemory.

6. Estado Actual del Proyecto

El proyecto actualmente permite:

Crear órdenes desde el frontend.

Ver las pizzas asociadas a cada orden.

Validar el uso de repositorios InMemory.

Ejecutar el flujo básico sin infraestructura externa.

Aún no se implementan:

Publicación de eventos

Subscriptores

Flujo completo entre Ordering y Production

Pero el proyecto ya funciona y es estable en local.
