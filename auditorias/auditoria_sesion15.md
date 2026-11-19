# Bitácora de Requerimientos y Auditoría IA — Sesión 15
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Bloques Anidados y Ciclo de Vida  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Operaciones Push y Pop de ámbito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente las funciones push_ambito() y pop_ambito(), identifiqué que la principal dificultad era administrar correctamente la memoria dinámica y los apuntadores de la estructura Ambito.

> Para implementar push_ambito() tuve que utilizar malloc() para reservar memoria para un nuevo ámbito y enlazarlo con el ámbito anterior mediante el campo padre.

> En la función pop_ambito() encontré la dificultad de conservar la dirección del ámbito padre antes de liberar el ámbito actual con free(). También tuve que considerar que la raíz no debe liberarse, porque puede estar declarada directamente en el programa y no haber sido creada con malloc().

> Otra restricción importante fue comprobar que los argumentos recibidos no fueran NULL y verificar que la reserva de memoria se realizara correctamente para evitar errores durante la ejecución.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Liberación de memoria al cerrar bloque |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contexto:Estoy desarrollando la Sesión 15 de la materia Lenguajes y Autómatas II.
El objetivo es implementar bloques anidados y administrar el ciclo de
vida de los identificadores mediante una estructura enlazada llamada
Ambito, que contiene un nombre y un apuntador al ámbito padre.

Rol:Actúa como un ingeniero de software especializado en lenguaje C,
sistemas y compiladores, con conocimientos de memoria dinámica,
apuntadores y estructuras enlazadas.

Especificación:Implementa las siguientes funciones respetando exactamente las firmas
de api_sesion15.h:

Ambito *push_ambito(Ambito *actual, const char *nombre);
Ambito *pop_ambito(Ambito *actual);

La función push_ambito() debe:
1. Verificar que actual y nombre no sean NULL.
2. Reservar memoria para un nuevo ámbito utilizando malloc().
3. Comprobar que la reserva de memoria sea exitosa.
4. Guardar el nombre recibido.
5. Enlazar el nuevo ámbito con el ámbito actual como su padre.
6. Devolver el nuevo ámbito.

La función pop_ambito() debe:
1. Verificar que el ámbito actual no sea NULL.
2. Evitar liberar el ámbito raíz cuando su padre sea NULL.
3. Guardar el apuntador al padre antes de liberar memoria.
4. Liberar únicamente el ámbito actual utilizando free().
5. Devolver el apuntador al padre.

No cambies las estructuras ni las firmas originales.
No incluyas main(), no utilices memoria dinámica innecesaria
y evita advertencias con -Wall -Wextra -Werror.

Output:Entrega el código completo de ia_sesion15_c.c, compatible con C11,
con comentarios explicativos y un análisis de complejidad temporal
y espacial de ambas funciones.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion15_c.c - Fase 2: El Cientifico */

#include "api_sesion15.h"
#include <stdlib.h>

/* Crea un nuevo ambito enlazado con el actual */
Ambito *push_ambito(Ambito *actual, const char *nombre) {

    Ambito *nuevo;

    /* Validar los argumentos recibidos */
    if (actual == NULL || nombre == NULL) {
        return NULL;
    }

    /* Reservar memoria para el nuevo nodo */
    nuevo = (Ambito *)malloc(sizeof(Ambito));

    /* Comprobar si la reserva fue exitosa */
    if (nuevo == NULL) {
        return NULL;
    }

    /* Inicializar los datos del nuevo ambito */
    nuevo->nombre = nombre;
    nuevo->padre = actual;

    return nuevo;
}

/* Cierra el ambito actual y devuelve su padre */
Ambito *pop_ambito(Ambito *actual) {

    Ambito *padre;

    /* Si no existe un ambito, no hay nada que cerrar */
    if (actual == NULL) {
        return NULL;
    }

    /* Proteger el ambito raiz */
    if (actual->padre == NULL) {
        return NULL;
    }

    /* Conservar el padre antes de liberar memoria */
    padre = actual->padre;

    /* Liberar exclusivamente el nodo actual */
    free(actual);

    return padre;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Aislamiento de variables fuera de bloque

### Salida del harness
```text
=== AUDITORIA SESION 15 - Push y pop de ambitos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre del ambito nuevo            esperado: if           obtenido: if           OK
[ 2] profundidad con dos push           esperado: 3            obtenido: 3            OK
[ 3] pop devuelve el padre              esperado: 1            obtenido: 1            OK
[ 4] pop sobre la raiz                  esperado: 1            obtenido: 1            OK
[ 5] push con padre NULL                esperado: 1            obtenido: 1            OK
[ 6] profundidad tras pop               esperado: 2            obtenido: 2            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion15.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que el nombre del ámbito se guarda mediante un apuntador (nuevo->nombre = nombre), pero no se realiza una copia de la cadena.

> Esto puede ocasionar un problema si el nombre proviene de una variable local que deja de existir mientras el ámbito todavía conserva su apuntador.

> Detecté este detalle al revisar cómo se almacena el nombre dentro de la estructura Ambito.

> Para evitarlo, consideré necesario mantener la cadena original válida durante todo el tiempo de vida del ámbito. Utilicé nombres con almacenamiento persistente, como las cadenas literales "if" y "while" utilizadas en las pruebas.

> No modifiqué la estructura proporcionada por el profesor porque forma parte del contrato original.

> También comprobé que pop_ambito() guardara el apuntador al padre antes de liberar la memoria del ámbito actual, evitando perder la referencia al bloque anterior.

---
**Firma digital:** [BlancaHoma14]
