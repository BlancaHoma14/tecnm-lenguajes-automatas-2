# Bitácora de Requerimientos y Auditoría IA — Sesión 07
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Evaluación de Expresiones Aritméticas  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Evaluador recursivo de operaciones  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> En esta primera fase desarrollé manualmente la función evaluar() en lenguaje C.
Comencé identificando los tipos de nodos del árbol sintáctico abstracto.

> Después implementé las condiciones necesarias para reconocer los nodos numéricos y los nodos de operaciones binarias.

> Utilicé recursividad para evaluar primero el hijo izquierdo y posteriormente el hijo derecho. También incorporé comprobaciones para detectar la división y el módulo entre cero, así como los nodos inválidos.

> Dificultad encontrada:
La principal dificultad fue controlar correctamente los errores durante las llamadas recursivas, ya que cuando un hijo presenta un error es necesario detener la evaluación del nodo padre.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Manejo estricto de división entre cero |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contrato C-R-E-O 

C – Contexto: Estoy desarrollando un evaluador recursivo de expresiones aritméticas en lenguaje C para la asignatura Lenguajes y Autómatas II. Tengo un árbol sintáctico abstracto (AST) con nodos numéricos y nodos de operaciones binarias.

R – Rol: Actúa como un programador experto en lenguaje C y compiladores. Ayúdame a mejorar mi implementación manual, respetando las condiciones establecidas por el docente.

E – Especificación: Necesito implementar la función:

int evaluar(ASTNode *n, int *error);

Debe cumplir las siguientes condiciones:

Si el nodo es NULL, debe asignar error = 2.

Si es un nodo numérico, debe devolver su valor.

Debe evaluar recursivamente los hijos izquierdo y derecho.

Debe realizar las operaciones suma, resta, multiplicación, división y módulo.

Antes de dividir o calcular el módulo, debe comprobar que el divisor sea diferente de cero.

Si existe división o módulo entre cero, debe asignar error = 1.

Si el nodo u operador no es válido, debe asignar error = 2.

Si un hijo presenta un error, debe detener la evaluación y propagarlo al padre.

Debe respetar exactamente la estructura y firma de api_sesion07.h.

No debe incluir main().

El código debe compilar con -std=c11 -Wall -Wextra -Werror.

O – Output: Proporciona el código completo y modular para ia_sesion07_c.c, utilizando una función auxiliar recursiva cuando sea conveniente.

Explica las mejoras realizadas respecto a mi implementación manual y analiza la complejidad temporal y espacial del algoritmo.

Respeta el contrato api_sesion07.h y no modifiques el harness proporcionado por el docente.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion07_c.c
/*
 * ia_sesion07_c.c
 * Fase 2: El Cientifico (con IA)
 * Evaluador recursivo mejorado.
 */

#include "api_sesion07.h"
#include <stddef.h>

/* Funcion auxiliar para evaluar el arbol */
static int evaluar_recursivo(ASTNode *n, int *error) {

    int izquierda;
    int derecha;

    /* Verificar si el nodo existe */
    if (n == NULL) {
        *error = 2;
        return 0;
    }

    /* Evaluar nodos numericos */
    if (n->tipo == NODO_NUM) {
        return n->valor;
    }

    /* Verificar el tipo de nodo */
    if (n->tipo != NODO_BINOP) {
        *error = 2;
        return 0;
    }

    /* Validar el operador */
    switch (n->op) {
        case '+':
        case '-':
        case '*':
        case '/':
        case '%':
            break;

        default:
            *error = 2;
            return 0;
    }

    /* Evaluar el hijo izquierdo */
    izquierda = evaluar_recursivo(n->izq, error);

    /* Propagar errores del hijo izquierdo */
    if (*error != 0) {
        return 0;
    }

    /* Evaluar el hijo derecho */
    derecha = evaluar_recursivo(n->der, error);

    /* Propagar errores del hijo derecho */
    if (*error != 0) {
        return 0;
    }

    /* Ejecutar la operacion correspondiente */
    switch (n->op) {

        case '+':
            return izquierda + derecha;

        case '-':
            return izquierda - derecha;

        case '*':
            return izquierda * derecha;

        case '/':
            if (derecha == 0) {
                *error = 1;
                return 0;
            }
            return izquierda / derecha;

        case '%':
            if (derecha == 0) {
                *error = 1;
                return 0;
            }
            return izquierda % derecha;

        default:
            *error = 2;
            return 0;
    }
}

/* Funcion principal solicitada por el contrato */
int evaluar(ASTNode *n, int *error) {

    /* Comprobar que el puntero de error exista */
    if (error == NULL) {
        return 0;
    }

    /* Inicializar el estado del error */
    *error = 0;

    /* Evaluar el arbol recursivamente */
    return evaluar_recursivo(n, error);
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones complejas y división por 0

### Salida del harness
```text
=== AUDITORIA SESION 07 - Evaluador recursivo con deteccion de errores ===
Caso                               esperado:              obtenido:              resultado
[ 1] (2+3)*4                            esperado: 20           obtenido: 20           OK
[ 2] 7/0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 3] 7%0 -> error 1                     esperado: 1            obtenido: 1            OK
[ 4] arbol NULL -> error 2              esperado: 2            obtenido: 2            OK
[ 5] 10-2-3                             esperado: 5            obtenido: 5            OK
[ 6] (8/2)+1                            esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión identifiqué que una propuesta inicial de IA podía omitir la validación de los hijos de un nodo de operación cuando estos fueran NULL.

> Para corregirlo, incorporé una comprobación dentro de la función recursiva, de manera que los nodos inexistentes generen el error 2 y no provoquen una caída del programa.

---
**Firma digital:** [BlancaHoma14]
