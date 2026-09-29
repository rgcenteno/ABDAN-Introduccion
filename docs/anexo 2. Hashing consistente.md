# Hashing consistente en sharding

## 1. Problema del hash clásico

En un sistema de sharding simple se usa:

```
shard = hash(clave) % N
```

Ejemplo con 3 shards:

```
user1 → shard 1
user2 → shard 2
user3 → shard 0
```

### Problema

Si añadimos un nodo (de 3 a 4 shards):

```
hash(clave) % 4
```

👉 Todos los datos cambian de shard

Consecuencias:
- Migración masiva
- Coste elevado
- Riesgo en producción

---

## 2. Solución: hashing consistente

### Idea clave

En lugar de usar módulo, se utiliza un **anillo de hash (ring)**.


### Ejemplo visual

```
[ Nodo A ] ---- [ Nodo B ] ---- [ Nodo C ]
      ↑                              ↓
      └───────────────(círculo)──────┘
```
### Funcionamiento consistent hashing

- **Espacio de hash circular:** Imagina que tienes un espacio de hash circular, representado por un círculo donde cada posición corresponde a un valor de hash. Los servidores se distribuyen por este espacio de manera que cada uno tiene asignado un rango de valores de hash.
- **Asignación de datos a servidores:** Cuando se recibe una clave (por ejemplo, una ID de usuario), esta clave se convierte en un valor de hash. El valor de hash de la clave se mapea a una posición en el círculo de hash. Luego, el dato se asigna al primer servidor que se encuentra en el sentido horario desde ese valor de hash. Si el valor de hash cae directamente sobre un servidor, ese servidor será responsable de la clave.
- **Adición y eliminación de servidores:** Cuando se agrega un servidor, solo las claves que caen dentro del rango del servidor recién agregado deben ser redistribuidas. Lo mismo ocurre cuando un servidor es eliminado: solo las claves asignadas a ese servidor deben ser reasignadas a otros servidores cercanos en el círculo.
Virtualización de servidores: Para mejorar aún más la distribución de claves y evitar el desbalance de carga, a menudo se emplea la técnica de virtualización de servidores. Esto significa que cada servidor físico puede representar múltiples servidores virtuales en el círculo de hash, lo que ayuda a equilibrar la carga entre servidores, especialmente en escenarios donde algunos servidores son mucho más potentes que otros.

```
hash(user123) → posición en el anillo
```

Se guarda en el nodo más cercano hacia la derecha.

---

## 3. ¿Qué pasa al añadir o eliminar un nodo?

Cuando se agrega un servidor, **solo las claves que caen dentro del rango del servidor recién agregado** deben ser redistribuidas. Lo mismo ocurre cuando un servidor es eliminado: solo las claves asignadas a ese servidor deben ser reasignadas a otros servidores cercanos en el círculo.

Resultado:

- No se mueve todo el sistema
- Solo una pequeña parte de los datos

---

## Resultado clave

> Solo se mueve aproximadamente 1/N de los datos

---

## 4. Ventajas

- Escalabilidad real  
- Rebalanceo mínimo  
- Distribución uniforme  
- Permite añadir nodos sin impacto global  

---

## 5. Virtual Nodes (vnodes)

Para mejorar aún más la distribución de claves y evitar el desbalance de carga, a menudo se emplea la técnica de **virtualización de servidores**. Esto significa que cada servidor físico puede representar múltiples servidores virtuales en el círculo de hash, lo que ayuda a equilibrar la carga entre servidores, especialmente en escenarios donde algunos servidores son mucho más potentes que otros.

- Cada nodo físico aparece varias veces en el anillo

Ejemplo:

```
Nodo A → posiciones 10, 50, 90
Nodo B → posiciones 20, 60, 100
```

👉 Evita desequilibrios de carga

---

## 6. Uso en sistemas reales

- Cassandra  
- DynamoDB  
- Riak  
- Sistemas distribuidos de caché  

---

## 7. Comparativa

| Hash clásico | Hashing consistente |
|-------------|--------------------|
| hash % N | Anillo de hash |
| Rebalanceo total | Rebalanceo parcial |
| Poco flexible | Muy flexible |
| Malo para escalado | Ideal para escalado |

---

## Resumen

El hashing consistente es una técnica de particionado que distribuye datos en un anillo de hash, de forma que al añadir o eliminar nodos solo se redistribuye una pequeña parte de los datos, facilitando la escalabilidad en sistemas distribuidos.