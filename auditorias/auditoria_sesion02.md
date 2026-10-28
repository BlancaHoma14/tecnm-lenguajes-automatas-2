## Bitácora de Requerimientos y Auditoría IA — Sesión 02

**Materia:** Lenguajes y Autómatas II (SCD-1016)

**Tema:** Gestión en GitHub Web y Nodos Dinámicos

**Docente Titular:** MCC. Iván Márquez Larios (ijmarquezl)

**Estudiante:** [Blanca Janet Homa May]

**Matrícula:** [24531351]

**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)

### 1. Fase 1: El Artesano (Dificultad Manual)

_Reto:_ Lista enlazada de tokens en Heap.

_Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:_

Tuve dificultades al implementar la reserva dinámica de memoria para cada nodo y al enlazar correctamente los nodos de la lista. También fue necesario asegurar que el último nodo apuntara a NULL y que la liberación de memoria se realizara sin perder la referencia al siguiente nodo.

### 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

#### 2.1 Matriz de Especificación Formal

| Elemento C-R-E-O | Definición Técnica |
|------------------|-------------------|
| **Contexto (C)** | C99 / GCC sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Implementar una lista enlazada de tokens utilizando memoria dinámica con malloc y free |
| **Output (O)** | Código modular en C compatible con el harness |

#### 2.2 Prompt Estructurado

**Contexto:** Estoy desarrollando una lista enlazada de tokens en C y debo respetar exactamente las firmas definidas en api_sesion02.h.

**Rol:** Actúa como un programador experto en C y estructuras dinámicas de datos.

**Especificación:** Implementa las funciones crear_nodo, agregar_token, contar_tokens y liberar_lista usando memoria dinámica. El último nodo debe apuntar a NULL. contar_tokens(NULL) debe devolver 0 y liberar_lista(NULL) no debe provocar errores.

**Output:** Devuelve únicamente código C compilable sin incluir main().

### 3. Código Devuelto por el Modelo

La IA generó una implementación funcional de las cuatro funciones requeridas utilizando malloc(), free(), strncpy() y recorrido de listas enlazadas.

```c
#include "api_sesion02.h"
#include <stdlib.h>
#include <string.h>

NodoToken *crear_nodo(const char *lexema) {
    NodoToken *nuevo = malloc(sizeof(NodoToken));

    if (nuevo == NULL) {
        return NULL;
    }

    strncpy(nuevo->lexema, lexema, sizeof(nuevo->lexema) - 1);
    nuevo->lexema[sizeof(nuevo->lexema) - 1] = '\0';
    nuevo->siguiente = NULL;

    return nuevo;
}

void agregar_token(NodoToken **cabeza, const char *lexema) {
    NodoToken *nuevo = crear_nodo(lexema);

    if (nuevo == NULL) {
        return;
    }

    if (*cabeza == NULL) {
        *cabeza = nuevo;
        return;
    }

    NodoToken *actual = *cabeza;

    while (actual->siguiente != NULL) {
        actual = actual->siguiente;
    }

    actual->siguiente = nuevo;
}

int contar_tokens(NodoToken *cabeza) {
    int contador = 0;

    while (cabeza != NULL) {
        contador++;
        cabeza = cabeza->siguiente;
    }

    return contador;
}

void liberar_lista(NodoToken *cabeza) {
    NodoToken *temp;

    while (cabeza != NULL) {
        temp = cabeza->siguiente;
        free(cabeza);
        cabeza = temp;
    }
}
```

### 4. Fase 3: El Auditor (Resultados del Test Harness)

_Pruebas ejecutadas:_ Prevención de Use-After-Free y cadenas nulas.

#### Salida del harness

```text
=== AUDITORIA SESION 02 - Lista enlazada de tokens en heap ===

[ 1] crear_nodo("if")->lexema        esperado: if         obtenido: if         OK
[ 2] contar_tokens(3 tokens)         esperado: 3          obtenido: 3          OK
[ 3] orden: segundo token            esperado: dos        obtenido: dos        OK
[ 4] ultimo->siguiente == NULL       esperado: 1          obtenido: 1          OK
[ 5] contar_tokens(NULL)             esperado: 0          obtenido: 0          OK
[ 6] liberar_lista(NULL)             esperado: sin crash  obtenido: sin crash  OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion02.md
```

### Reporte Crítico

**¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?**

Durante la revisión observé que un posible caso borde es la falla de la función malloc(). Si la reserva de memoria no tiene éxito, malloc() devuelve NULL y utilizar ese puntero podría provocar un fallo de ejecución. Para evitar este problema, se verificó que el puntero retornado por malloc() fuera distinto de NULL antes de acceder a sus campos.

Además, se comprobó mediante el harness que las funciones contar_tokens(NULL) y liberar_lista(NULL) funcionaran correctamente cuando la lista está vacía, evitando errores por referencias nulas. Después de estas verificaciones, la implementación superó satisfactoriamente las 6 de 6 pruebas del auditor.

**Resultado final:** 6/6 pruebas superadas.

**Firma digital:** BlancaHoma14
