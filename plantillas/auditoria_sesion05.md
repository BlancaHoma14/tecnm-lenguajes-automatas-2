# Bitácora de Requerimientos y Auditoría IA — Sesión 05
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Recorridos del AST y Notación Polaca  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Impresión en Postorden y generación de notación polaca inversa RPN  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al implementar manualmente la función postorden(), identifiqué que debía respetar exactamente la firma declarada en api_sesion05.h y controlar el espacio disponible en el buffer de salida. También comprendí que el recorrido debía realizarse en el orden izquierdo, derecho y raíz para generar correctamente la notación polaca inversa.

Una dificultad fue construir la cadena RPN sin agregar espacios innecesarios ni dejar un espacio al final. Para resolverlo, utilicé la recursividad y la función snprintf() para convertir los valores numéricos a texto y concatenar los resultados de los subárboles.

Mi implementación manual utilizó buffers temporales de tamaño fijo de 256 caracteres. Aunque superó los cinco casos del harness, identifiqué que esta estrategia puede provocar truncamiento cuando se procesan expresiones grandes. Por ello, reconocí la necesidad de mejorar el control de memoria y la construcción de la cadena.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Invariantes de recorrido recursivo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
``
[C — Contexto: Estoy trabajando en la asignatura Lenguajes y Autómatas II. Necesito implementar una función en C que recorra un árbol sintáctico abstracto (AST) en postorden y genere la notación polaca inversa (RPN) en un buffer.

R — Rol: Actúa como un ingeniero de software especializado en lenguaje C, estructuras de datos recursivas y compiladores.

E — Especificación: Implementa la función postorden(ASTNode *n, char *buf, int cap) respetando exactamente la declaración de api_sesion05.h. El recorrido debe seguir el orden izquierdo, derecho y raíz. Los números y operadores deben separarse con un solo espacio, sin espacio final. Si el árbol es nulo, la salida debe ser una cadena vacía. No se debe imprimir con printf() desde la función ni agregar una función main().

Controla la capacidad del buffer para no escribir fuera de sus límites, conserva el terminador '\0' cuando el buffer y su capacidad lo permitan y evita advertencias con -std=c11 -Wall -Wextra -Werror. Utiliza una implementación modular que facilite la revisión y las pruebas.

O — Output: Entrega el código completo de ia_sesion05_c.c, listo para compilar con el harness de la Sesión 05. Explica las mejoras respecto a una implementación manual que utiliza buffers temporales de tamaño fijo y describe la complejidad temporal y espacial.Pega aquí el prompt formal que redactaste para la IA]
``



## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion05_c.c
/* ia_sesion05_c.c - Fase 2: El Cientifico */
#include "api_sesion05.h"
#include <stdio.h>
#include <string.h>

/* Estado del recorrido y control de capacidad. */
typedef struct {
    char *buf;
    int cap;
    int pos;
    int error;
} EstadoRPN;

/* Agrega un token completo, sin exceder la capacidad. */
static void agregar_token(EstadoRPN *e, const char *token) {
    size_t longitud;
    size_t separador;
    size_t disponible;

    if (e == NULL || token == NULL || e->error) {
        return;
    }

    longitud = strlen(token);
    separador = (e->pos > 0) ? 1U : 0U;
    disponible = (size_t)(e->cap - e->pos - 1);

    if (longitud + separador > disponible) {
        e->error = 1;
        return;
    }

    if (separador != 0U) {
        e->buf[e->pos++] = ' ';
    }

    memcpy(e->buf + e->pos, token, longitud);
    e->pos += (int)longitud;
    e->buf[e->pos] = '\0';
}

/* Recorre el AST en postorden: izquierdo, derecho, raíz. */
static void recorrer(ASTNode *n, EstadoRPN *e) {
    char token[32];

    if (n == NULL || e == NULL || e->error) {
        return;
    }

    if (n->tipo == NODO_NUM) {
        (void)snprintf(token, sizeof(token), "%d", n->valor);
        agregar_token(e, token);
        return;
    }

    if (n->tipo == NODO_BINOP) {
        recorrer(n->izq, e);
        recorrer(n->der, e);

        if (e->error) {
            return;
        }

        token[0] = n->op;
        token[1] = '\0';
        agregar_token(e, token);
    }
}

void postorden(ASTNode *n, char *buf, int cap) {
    EstadoRPN estado;

    if (buf == NULL || cap <= 0) {
        return;
    }

    buf[0] = '\0';

    estado.buf = buf;
    estado.cap = cap;
    estado.pos = 0;
    estado.error = 0;

    recorrer(n, &estado);
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:*compilación con GCC y validación funcional mediante los cinco casos del harness proporcionado.
### Salida del harness
``
=== AUDITORIA SESION 05 - Recorrido postorden y RPN ===                                            
Caso                               esperado:              obtenido:              resultado
[ 1] hoja 42                            esperado: 42           obtenido: 42           OK
[ 2] 3+(5*2)                            esperado: 3 5 2 * +    obtenido: 3 5 2 * +    OK
[ 3] arbol NULL                         esperado:              obtenido:              OK
[ 4] (1+2)*(3+4)
     esperado: 1 2 + 3 4 + *
     obtenido: 1 2 + 3 4 + *
     -> OK
[ 5] 8-3                                esperado: 8 3 -        obtenido: 8 3 -        OK

RESULTADO: 5/5 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion05.mdPega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas
``

* [ ✓] Caso base y límites superados sin *Crash*.
* [ ✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [ ✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> El caso borde identificado fue el uso de un buffer demasiado pequeño para almacenar toda la expresión RPN. La implementación manual puede truncar resultados por utilizar buffers fijos de 256 caracteres. La versión mejorada verifica el espacio disponible antes de agregar cada token, evitando escribir fuera de los límites y manteniendo la cadena terminada en '\0'. Sin embargo, el harness original no prueba buffers pequeños ni árboles muy profundos, por lo que sería necesario agregar pruebas para comprobar estos casos.

---
**Firma digital:** [Blanca Janet Homa May]
