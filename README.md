# Drone Delivery

Glovo + Drones
El ejercicio está pensado para que haya business logic con más
interés que un CRUD y se aprecie major la utilidad de una cola de mensajes en un scenario de tiempo real.

# Componentes

Un puñado de APIs de HTTP + una cola de mensajes.
Se asume que los drones llevan instalado un cliente + servidor para comunicarse con los servicios que corresponden.

| Componente                  | Responsabilidad Principal                                                                 | Interacciones Clave |
|----------------------------|-------------------------------------------------------------------------------------------|---------------------|
| Órdenes                    | Gestión de pedidos (crear, leer, actualizar, eliminar)                                   | Publica `NEW_ORDER` en la cola |
| Clientes                   | Gestión de clientes                                                                       | Relacionado con Órdenes |
| Inventario                 | Gestión de drones disponibles y sus capacidades                                           | Consultado por Control |
| Control                    | Asigna drones a pedidos, coordina rutas, gestiona estados operativos                     | Consume `NEW_ORDER`, llama a Inventario y Rutas |
| Rutas                      | Calcula rutas considerando zonas restringidas y estaciones; monitorea anomalías          | Recibe solicitudes de Control, consume `STOP`, emite `ANOMALY` |
| Telemetry                  | Recibe telemetría en tiempo real, actualiza estado drones, emite eventos operativos      | Emite `START`, `STOP`, `ORDER_COMPLETED`; consume `ANOMALY` |
| Cola de Mensajes           | Comunicación asíncrona entre servicios                                                    | Transporta eventos (`NEW_ORDER`, `STOP`, `ANOMALY`, etc.) |

## Ordenes

CRUD

## Clientes

CRUD

## Inventario

CRUD

## Control

Controla el estado de los drones

Lee mensajes de `NEW_ORDER` y se comunica con Inventario para asignar el más indicado. Escoge un dron y le asigna una ruta calculada por Rutas

Recibe requests de Telemetry para marcar drones como averiados o bajos de batería



## Rutas

Calcula rutas, teniendo en cuenta una config de zonas restringidas y estaciones
Expone una API para reporter el progreso / tiempo previsto de un dron en camino
Lee mensajes de `STOP` y verifica la posición para levantar un evento de `ANOMALY`



## Telemetry

Un dron en ruta streamea datos como posicion y bateria en tiempo real. Telemetry recibe estos datos y emite eventos de `START`, `STOP`, `ORDER_COMPLETED`

Lee eventos de `ANOMALY`, confirma el estado del dron y levanta una incidencia a un servicio de soporte imaginario

Expone una API para consultar el estado de los drones en tiempo real



## Cola de Mensajes

No hay mucho que explicar.



# Posibles Extras

* Si nos parece poco, podemos pedir que programen el cliente/servidor que lleva el dron, o la simulacion para un E2E
* Tambien podemos especificar que los drones usan TCP para sus comunicaciones (imagino que si no decimos nada, usarán HTTP)
* Podemos especificar tambien que incluyan una KV tipo Redis. A Telemetry le vendria bien para almacenar el historico de sus streams
* Se puede añadir algun servicio que lance tareas largas para darle más uso a la Cola... Algo tipo un servicio de analisis de imagenes que capturan los drones
