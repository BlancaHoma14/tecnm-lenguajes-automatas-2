# Bitácora de Requerimientos y Auditoría IA — Sesión 22
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Generación de TAC para Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Traductor de nodos AST a TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el código, identifiqué que la principal dificultad era recorrer correctamente el árbol de sintaxis abstracta (AST) para generar las instrucciones en el orden adecuado.

> Tuve que comprender cómo funciona el recorrido en postorden, porque primero se deben procesar los hijos izquierdo y derecho y después el operador. Si no se respeta este orden, las operaciones pueden generarse incorrectamente y no se conserva la precedencia de las expresiones.

> Otra dificultad fue controlar los temporales, ya que cada operador necesita un identificador diferente, como _t0, _t1 y _t2. También tuve que considerar las restricciones de memoria del arreglo lineas[][64], porque no se deben escribir más líneas de las permitidas ni superar los 63 caracteres de contenido por línea.

> Para resolverlo manualmente, utilicé una función recursiva llamada traducir() que procesa los nodos y devuelve el nombre del temporal o el valor de la hoja.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Conservación de precedencia de operadores |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
**CONTEXTO (C):** Estoy trabajando en la materia Lenguajes y Autómatas II, Sesión 22: Generación de TAC para Expresiones Aritmético-Lógicas. Utilizo lenguaje C y GCC con las opciones `-std=c11 -Wall -Wextra -Werror`.

Tengo un archivo `api_sesion22.h` que define la estructura `Nodo` y la función `generar_tac()`.

**ROL (R):** Actúa como un ingeniero de software especializado en compiladores, árboles de sintaxis abstracta (AST), recorridos recursivos y generación de código intermedio.

**ESPECIFICACIÓN (E):** Implementa la función:

```c
int generar_tac(const Nodo *raiz, char lineas[][64], int max);
Debes cumplir lo siguiente:

1. Recorrer el AST en postorden: primero hijo izquierdo, después hijo derecho y finalmente el operador.
2. Reconocer los nodos hoja mediante `tipo == 'H'` y devolver su valor sin generar instrucciones.
3. Reconocer los nodos binarios mediante `tipo == 'B'`.
4. Generar exactamente una instrucción TAC por cada operador.
5. Utilizar obligatoriamente `nuevo_temporal()` para obtener los temporales `_t0`, `_t1`, etc.
6. Escribir las instrucciones con el formato `temporal = operando1 operador operando2`.
7. No escribir más de `max` líneas ni más de 63 caracteres por línea.
8. Validar punteros nulos, capacidad disponible y nodos inválidos.
9. Evitar accesos fuera de los límites del arreglo.
10. No modificar `api_sesion22.h` ni agregar una función `main()`.

**OUTPUT (O):** Entrega el código completo en C, modular, con funciones auxiliares, comentarios, validaciones y análisis de complejidad temporal y espacial. Explica las mejoras realizadas respecto a una implementación manual y asegúrate de conservar exactamente la firma del contrato.
```

---

## 3. Código Devuelto por el Modelo
```c

/* ia_sesion22_c.c - Fase 2: El Cientifico */

#include "api_sesion22.h"
#include <stdio.h>
#include <string.h>

#define MAX_TEMPORALES 64
#define TAM_TEMPORAL 16
#define TAM_LINEA 64

typedef struct {
    char nombre[MAX_TEMPORALES][TAM_TEMPORAL];
    int usados;
} Pool;

typedef struct {
    char (*lineas)[TAM_LINEA];
    int max;
    int cantidad;
    int contador;
    Pool pool;
    int error;
} Estado;

/* Genera un nuevo temporal */
static const char *nuevo_temporal(Pool *p, int *contador) {

    if (p == NULL || contador == NULL ||
        p->usados >= MAX_TEMPORALES) {
        return NULL;
    }

    int escrito = snprintf(
        p->nombre[p->usados],
        sizeof p->nombre[0],
        "_t%d",
        *contador
    );

    if (escrito < 0 ||
        escrito >= (int)sizeof p->nombre[0]) {
        return NULL;
    }

    (*contador)++;

    return p->nombre[p->usados++];
}

/* Traducción recursiva del AST */
static const char *traducir(const Nodo *n, Estado *e) {

    const char *izq;
    const char *der;
    const char *temp;
    int escrito;

    if (n == NULL || e == NULL || e->error) {
        if (e != NULL) {
            e->error = 1;
        }
        return NULL;
    }

    /* Procesamiento de hojas */
    if (n->tipo == 'H') {

        if (n->valor[0] == '\0') {
            e->error = 1;
            return NULL;
        }

        return n->valor;
    }

    /* Validación del tipo de nodo */
    if (n->tipo != 'B' || n->op == '\0') {
        e->error = 1;
        return NULL;
    }

    /* Recorrido en postorden: hijo izquierdo */
    izq = traducir(n->izq, e);

    if (izq == NULL || e->error) {
        return NULL;
    }

    /* Recorrido en postorden: hijo derecho */
    der = traducir(n->der, e);

    if (der == NULL || e->error) {
        return NULL;
    }

    /* Verifica la capacidad antes de emitir */
    if (e->cantidad >= e->max ||
        e->pool.usados >= MAX_TEMPORALES) {
        e->error = 1;
        return NULL;
    }

    /* Solicita un temporal */
    temp = nuevo_temporal(&e->pool, &e->contador);

    if (temp == NULL) {
        e->error = 1;
        return NULL;
    }

    /* Emite la instrucción TAC */
    escrito = snprintf(
        e->lineas[e->cantidad],
        TAM_LINEA,
        "%s = %s %c %s",
        temp, izq, n->op, der
    );

    if (escrito < 0 || escrito >= TAM_LINEA) {
        e->lineas[e->cantidad][0] = '\0';
        e->error = 1;
        return NULL;
    }

    e->cantidad++;

    return temp;
}

/* Interfaz pública */
int generar_tac(const Nodo *raiz, char lineas[][64], int max) {

    Estado estado;

    if (raiz == NULL || lineas == NULL || max <= 0) {
        return 0;
    }

    estado.lineas = lineas;
    estado.max = max;
    estado.cantidad = 0;
    estado.contador = 0;
    estado.pool.usados = 0;
    estado.error = 0;

    traducir(raiz, &estado);

    return estado.cantidad;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones anidadas con paréntesis

### Salida del harness
```text
=== AUDITORIA SESION 22 - Traductor de AST a TAC con precedencia ===
Caso                               esperado:              obtenido:              resultado
[ 1] a + b * c                          esperado: _t0 = b * c  obtenido: _t0 = b * c  OK
[ 2] a + b * c (2a linea)
     esperado: _t1 = a + _t0
     obtenido: _t1 = a + _t0
     -> OK
[ 3] (a + b) * c
     esperado: _t1 = _t0 * c
     obtenido: _t1 = _t0 * c
     -> OK
[ 4] x * (y - z) + w
     esperado: _t2 = _t1 + w
     obtenido: _t2 = _t1 + w
     -> OK
[ 5] hoja sin operaciones               esperado: 0            obtenido: 0            OK
[ 6] (a + b) * (c - d)
     esperado: _t2 = _t0 * _t1
     obtenido: _t2 = _t0 * _t1
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion22.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que, aunque contempla la validación de nodos nulos, tipos inválidos y capacidad disponible, existe una situación que debe considerarse: cuando el árbol contiene más operadores de los que permite el arreglo lineas, la traducción se detiene y puede devolver solamente las instrucciones que alcanzó a generar.

> Esto sucede porque la función devuelve un entero que representa la cantidad de instrucciones, pero no tiene un parámetro independiente para comunicar si la traducción terminó correctamente o si se interrumpió por falta de espacio.

> También consideré el caso de que una instrucción TAC fuera demasiado larga para almacenarse en los 64 caracteres disponibles. Para evitar aceptar una línea incompleta, la versión mejorada comprueba el resultado de snprintf() y activa el indicador de error cuando detecta truncamiento.

---
**Firma digital:** [BlancaHoma14]
