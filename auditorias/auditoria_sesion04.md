# Bitácora de Requerimientos y Auditoría IA — Sesión 04
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Anatomía de un Árbol Sintáctico (AST)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janeth Homa May]  
**Matrícula:** [No. 24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de nodo binario (ASTNode)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*

> Durante la implementación manual, el principal desafío radicó en asegurar la correcta gestión de memoria dinámica en el *heap* utilizando `malloc`, garantizando la inicialización limpia de los punteros hacia los hijos izquierdo y derecho en los nodos numéricos. Asimismo, se requirió estructurar cuidadosamente la función recursiva de evaluación controlando rigurosamente el caso base (`NULL`) para prevenir accesos de lectura inválidos que desencadenaran un fallo de segmentación (*Segmentation Fault*), cumpliendo de forma estricta con el contrato de `api_sesion04.h` sin emplear campos mágicos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Gramática E -> E + T | T con tipos de nodo |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

```text
[Contexto] Estamos desarrollando un analizador sintáctico en C99 bajo compilación estricta GCC (-Wall -Wextra -Werror) para la materia de Lenguajes y Autómatas II.
[Rol] Actúa como un ingeniero de sistemas experto en diseño de compiladores y estructuras de datos en C.
[Especificación] Implementa las funciones declaradas en el contrato api_sesion04.h:
1. ASTNode *crear_numero(int v): Reserva memoria en el heap para un nodo numérico y asigna su valor.
2. ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der): Reserva memoria para un nodo de operación binaria enlazando sus hijos.
3. int evaluar(ASTNode *n): Recorre el AST de forma recursiva evaluando las operaciones (+, -, *) y devuelve -1 si el nodo es NULL sin causar fallos de segmentación.
[Salida] Devuelve exclusivamente el código en C puro, modular y sin incluir la función main().
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion04_c.c - Fase 2: El Cientifico */
#include "api_sesion04.h"
#include <stdlib.h>

ASTNode *crear_numero(int v) {
    ASTNode *nodo = (ASTNode *)calloc(1, sizeof(ASTNode));
    if (!nodo) return NULL;
    nodo->tipo = NODO_NUM;
    nodo->valor = v;
    return nodo;
}

ASTNode *crear_binario(char op, ASTNode *izq, ASTNode *der) {
    ASTNode *nodo = (ASTNode *)calloc(1, sizeof(ASTNode));
    if (!nodo) return NULL;
    nodo->tipo = NODO_BINOP;
    nodo->op = op;
    nodo->izq = izq;
    nodo->der = der;
    return nodo;
}

int evaluar(ASTNode *n) {
    if (!n) return -1;
    
    switch (n->tipo) {
        case NODO_NUM:
            return n->valor;
        case NODO_BINOP: {
            int izq = evaluar(n->izq);
            int der = evaluar(n->der);
            switch (n->op) {
                case '+': return izq + der;
                case '-': return izq - der;
                case '*': return izq * der;
                default: return -1;
            }
        }
        default:
            return -1;
    }
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
```text
=== AUDITORIA SESION 04 - Nodo binario de AST ===
Caso                              esperado:           obtenido:      resultado
[ 1] evaluar(nodo 42)             esperado: 42        obtenido: 42   OK
[ 2] evaluar(3+5*2)               esperado: 13        obtenido: 13   OK
[ 3] hijo izquierdo de (* 5 2)    esperado: 5         obtenido: 5    OK
[ 4] operador guardado            esperado: *         obtenido: *    OK
[ 5] evaluar(NULL)                esperado: -1        obtenido: -1   OK
[ 6] evaluar(10-4)                esperado: 6         obtenido: 6    OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La primera aproximación generada por la IA solía omitir validaciones exhaustivas de asignación dinámica de memoria tras invocar rutinas de reserva o descuidar la verificación defensiva del puntero NULL al inicio de la función recursiva evaluar. Si un nodo llegara sin enlazar o apuntara a una dirección nula intermedia, la evaluación intentaría consultar sus campos provocando un fallo de segmentación. Esto se solucionó incorporando una guarda explícita al comienzo (if (!n) return -1;) y validando el éxito de la asignación de memoria con calloc.

---
**Firma digital:** [BlancaHoma14]
