# Anexo 1 – Sistemas de búsqueda de datos fragmentados

En algunas arquitecturas fragmentadas noSQL necesitamos un componente que nos permitan resolver el problema de dónde encontrar el datos. Tres de las soluciones más populares son **Sistemas determinísticos**, las **Tablas de búsqueda** y los **Proxys intermedios**.

## Sistema determinísticos (Cálculo directo a partir de la clave)

El cliente calcula directamente el shard a consultar aplicando una función sobre la clave de particionado. Aplicable a sistemas por rangos, por listas o por hash.

| Ventajas                          | Desventajas                             |
| :-------------------------------- | :-----------------------------          |
| Muy rápido (Sin consultas extra)  | Poco flexible                           |
| Sin metadatos externos            | Cambiar el número de shards es complejo |

## Tabla de búsqueda

Es una técnica de mapeo explícito. Es ideal cuando el sharding es irregular. Existe una tabla o servicio central que mantiene un mapa:

> clave / rango / hash → shard

Flujo:

1. El cliente pregunta a la tabla de búqueda.
2. La tabla devuelve el shard.
3. El cliente accede al shard

| Ventajas                                  | Desventajas                             |
| :--------------------------------         | :-----------------------------          |
| Muy fléxible                              | Consulta extra                          |
| Permite mover shards sin avisar a cliente | Sistema con mayor complejidad           |

## Proxy Intermedio

Actúa como un cerebro central. La aplicación se conecta al proxy como si fuera una base de datos única. El proxy analiza la consulta, decide a qué shards enviarla y combina los resultados. El cliente no sabe nada del sharding.

Flujo:

1. Cliente &rarr; Router
2. Router &rarr; Shard(s) correcto(s)
3. Respuesta &rarr; Cliente

| Ventajas                                  | Desventajas                             |
| :--------------------------------         | :-----------------------------          |
| Transparente para el cliente              | Un componente más a mantener            |
| Simplifica el desarrollo de clientes      | Posible cuello de botella               |

## Tabla comparativa

| Característica |Sistemas determinísticos                  | Tabla de Búsqueda (Lookup Table)                       |Proxy Intermedio (Query Router)                                        |
| -------------- | :-----------------------                 | :----------------------------------------------------- | :------------------------------------------------                     |
| **Ubicación**  | Cliente grueso                           | Base de datos interna o caché (ej. Redis).             | Capa de red/software (ej. Mongos en MongoDB).                         |
|**Lógica**      | El cliente calcula el shard directamente | El cliente consulta la tabla para saber a qué nodo ir. | El cliente no sabe nada; el proxy intercepta y redirige.              |
|**Flexibilidad**| Baja. Implica cambiar los clientes       | Muy alta: puedes mover un solo registro a otro shard.  | Alta: gestiona la topología completa del clúster.                     |
|**Latencia**    | **Ninguna** cálculo realizado en cliente | **Doble** salto: 1° buscar ubicación, 2° buscar dato.  | **Salto único** percibido por el cliente (el proxy hace el trabajo).  |
| **Ejemplos**   | Cassandra, DynamoDB                      | MongoDB (config servers)                               | MongoDB (mongos), API con gateway                                     |
