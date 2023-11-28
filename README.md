# 4-Create-Operations

1. [Intro](#schema1)
2. [Working with Ordered Inserts](#schema2)
3. [Understanding the `writeConcern`](#schema3)


<hr>

<a name="schema1"></a>

## 1. Intro
![create](./img/create1.png)

![create](./img/create2.png)


En MongoDB, insertOne, insertMany, y insert son métodos que se utilizan para insertar documentos en una colección. 
Aquí hay algunas diferencias clave entre ellos:

**insertOne:**

insertOne se utiliza para insertar un solo documento en una colección.
Recibe un solo documento como argumento, que debe ser un objeto de JavaScript o un documento BSON.
Devuelve un objeto que contiene información sobre la operación de inserción, como el identificador del documento 
insertado.

**insertMany:**

insertMany se utiliza para insertar varios documentos en una colección.
Recibe un array de documentos como argumento, cada uno de los cuales debe ser un objeto de JavaScript o un documento 
BSON.
Devuelve un objeto que contiene información sobre la operación de inserción, incluyendo los identificadores de los 
documentos insertados.

**insert (deprecated):**

El método insert se ha marcado como **obsoleto (deprecated)** y se desaconseja su uso en versiones más recientes de MongoDB.
Se utilizaba para insertar uno o varios documentos, dependiendo de si se le pasaba un solo documento o un array de 
documentos.
A partir de MongoDB 4.2, se recomienda utilizar insertOne o insertMany en lugar de insert.



<hr>

<a name="schema2"></a>

## 2. Working with Ordered Inserts

- Insertamos unos hobbies creando el id único `_id` nosotros.
```
sports> db.hobbies.insertMany([{_id:'sports', name:'Sports'},{_id:'cooking',name:'Cooking'},{_id:'cars', name:'Cars'}])
{
  acknowledged: true,
  insertedIds: { '0': 'sports', '1': 'cooking', '2': 'cars' }
}
sports> db.hobbies.find()
[
  { _id: 'sports', name: 'Sports' },
  { _id: 'cooking', name: 'Cooking' },
  { _id: 'cars', name: 'Cars' }
]
sports> 

```
- Insertamos más hobbies pero repitiendo uno, `Cooking`, dando un error porque se ha duplicado el `_id` y a su vez se 
ha insertado solo un elemento, el anterior al error, en este caso el hoobie `Yoga`. Se insertarán datos hasta que
encuentre un error que parará de insertar elementos.
```
sports> db.hobbies.insertMany([{_id:'yoga', name:'Yoga'},{_id:'cooking',name:'Cooking'},{_id:'hiking', name:'Hiking'}])
Uncaught:
MongoBulkWriteError: E11000 duplicate key error collection: sports.hobbies index: _id_ dup key: { _id: "cooking" }
Result: BulkWriteResult {
  insertedCount: 1,
  matchedCount: 0,
  modifiedCount: 0,
  deletedCount: 0,
  upsertedCount: 0,
  upsertedIds: {},
  insertedIds: { '0': 'yoga' }
}
Write Errors: [
  WriteError {
    err: {
      index: 1,
      code: 11000,
      errmsg: 'E11000 duplicate key error collection: sports.hobbies index: _id_ dup key: { _id: "cooking" }',
      errInfo: undefined,
      op: { _id: 'cooking', name: 'Cooking' }
    }
  }
]
sports> db.hobbies.find()
[
  { _id: 'sports', name: 'Sports' },
  { _id: 'cooking', name: 'Cooking' },
  { _id: 'cars', name: 'Cars' },
  { _id: 'yoga', name: 'Yoga' }
]
sports> 

```
- Cambiamos la sentencia y le añadimos `{ordered:false}` y ahora hace inserta todos lo valores pero el error no.
 Al utilizar `{ordered: false}`, algunos documentos se insertarán y otros no, dependiendo de si hay conflictos 
de clave única.


```
sports> db.hobbies.insertMany([{_id:'yoga', name:'Yoga'},{_id:'cooking',name:'Cooking'},{_id:'hiking', name:'Hiking'}],{ordered: false})
Uncaught:
MongoBulkWriteError: E11000 duplicate key error collection: sports.hobbies index: _id_ dup key: { _id: "yoga" }
Result: BulkWriteResult {
  insertedCount: 1,
  matchedCount: 0,
  modifiedCount: 0,
  deletedCount: 0,
  upsertedCount: 0,
  upsertedIds: {},
  insertedIds: { '2': 'hiking' }
}
Write Errors: [
  WriteError {
    err: {
      index: 0,
      code: 11000,
      errmsg: 'E11000 duplicate key error collection: sports.hobbies index: _id_ dup key: { _id: "yoga" }',
      errInfo: undefined,
      op: { _id: 'yoga', name: 'Yoga' }
    }
  },
  WriteError {
    err: {
      index: 1,
      code: 11000,
      errmsg: 'E11000 duplicate key error collection: sports.hobbies index: _id_ dup key: { _id: "cooking" }',
      errInfo: undefined,
      op: { _id: 'cooking', name: 'Cooking' }
    }
  }
]
sports> db.hobbies.find()
[
  { _id: 'sports', name: 'Sports' },
  { _id: 'cooking', name: 'Cooking' },
  { _id: 'cars', name: 'Cars' },
  { _id: 'yoga', name: 'Yoga' },
  { _id: 'hiking', name: 'Hiking' }
]
sports> 

```




<hr>

<a name="schema3"></a>

## 3. Understanding the `writeConcern`

![create](./img/create3.png)

En MongoDB, el Write Concern (preocupación por la escritura) se refiere al nivel de confirmación que el 
sistema de base de datos debe recibir antes de considerar una operación de escritura como exitosa. 
Específicamente, el Write Concern determina el número de nodos del clúster de MongoDB que deben confirmar 
la operación de escritura antes de que se considere completa.

Algunos conceptos clave asociados con el Write Concern en MongoDB incluyen:

**Acknowledged (Asegurado):** Este es el nivel predeterminado de Write Concern en MongoDB. 
En este nivel, la operación de escritura es confirmada solo cuando el nodo primario (primary) recibe 
la operación y la replica a al menos un número especificado de nodos secundarios (secondaries). 
Este número se puede configurar usando el parámetro `{w:1}` (write concern).


**Unacknowledged (No asegurado):** En este nivel, la operación de escritura se envía al servidor sin esperar 
confirmación. No hay garantía de que la operación se haya realizado correctamente. 
Este nivel se establece mediante { w: 0 }.


**Majority (Mayoría):** Este nivel garantiza que la operación se replica en la mayoría de los nodos del clúster 
(nodos primarios y secundarios) antes de considerarla exitosa. Este nivel se establece mediante { w: "majority" }.


Dentro de la opción writeConcern en MongoDB, puedes especificar varias configuraciones para ajustar el comportamiento 
de las operaciones de escritura. Algunas de las configuraciones adicionales que puedes incluir son:

**wtimeout:** Este parámetro especifica un límite de tiempo (en milisegundos) para la confirmación de la operación 
de escritura. Si la operación no se confirma dentro de este límite, se genera un error.


**fsync:** Este parámetro, cuando se establece en true, asegura que los datos se han sincronizado en disco antes de que 
se considere completa la operación de escritura. Este es un nivel más alto de garantía de durabilidad 
que escribir en el diario. 