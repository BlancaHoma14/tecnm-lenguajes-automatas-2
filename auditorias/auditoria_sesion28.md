# Bitácora de Requerimientos y Auditoría IA — Sesión 28
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Optimización: Constant Folding  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Plegado y simplificación de constantes  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente la función plegar(), identifiqué que la principal dificultad era distinguir entre operandos que representan números enteros y operandos que son variables o expresiones que todavía no pueden calcularse.

> Para resolverlo utilicé la función strtoll(), que permite convertir una cadena de texto a un número entero y comprobar que la conversión haya terminado correctamente, sin aceptar caracteres sobrantes.

También identifiqué las siguientes restricciones:
- El destino dst tiene una capacidad limitada indicada por cap.
- Es necesario comprobar que los apuntadores recibidos no sean NULL.
- Se debe evitar que las operaciones aritméticas provoquen desbordamientos.
- Una división entre cero no debe calcularse ni provocar una excepción.
- Las identidades algebraicas deben aplicarse sin modificar el significado de la expresión.
- Si no es posible realizar una optimización, se debe conservar la instrucción TAC original.
- La función debe generar una cadena válida y no escribir fuera de los límites del destino.
La mayor dificultad fue controlar los diferentes casos de entrada y asegurar que una optimización no modificara incorrectamente el resultado del programa.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Reducción estática en compilación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTRATO C-R-E-O

CONTEXTO (C): Estoy desarrollando la Sesión 28 de Lenguajes y Autómatas II (SCD-1016). Necesito implementar una función de optimización local para código intermedio TAC mediante plegado de constantes e identidades algebraicas. El código debe ser compatible con C11 y GCC.

ROL (R): Actúa como ingeniero de software especializado en compiladores, optimización de código intermedio, programación en C y validación de entradas.

ESPECIFICACIÓN (E): Implementa exactamente esta función: void plegar(char *dst, int cap, const char *res, const char *op, const char *a, const char *b); Requisitos:

Si ambos operandos son enteros completos, calcular las operaciones +, -, * y /.
No aceptar números con texto sobrante, como 12abc.
Aplicar las identidades x + 0 = x, x * 1 = x y x * 0 = 0, incluyendo el orden inverso cuando corresponda.
No plegar una división entre cero.
Conservar la instrucción original cuando no sea posible optimizar.
Comprobar apuntadores y capacidad del destino.
Evitar desbordamientos aritméticos y de conversión.
Comprobar el resultado de snprintf().
No utilizar memoria dinámica ni bibliotecas externas.
No modificar la firma ni agregar main().
OUTPUT (O): Entrega el código completo y modular, compatible con:

gcc -std=c11 -Wall -Wextra -Werror

Explica las mejoras respecto a la versión manual, los casos límite considerados y la complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion28_c.c

#include "api_sesion28.h"
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <limits.h>
#include <string.h>

static int convertir_entero(const char *texto, long long *valor) {
    char *fin;
    long long numero;

    if (texto == NULL || *texto == '\0') {
        return 0;
    }

    errno = 0;
    numero = strtoll(texto, &fin, 10);

    if (errno == ERANGE || fin == texto || *fin != '\0') {
        return 0;
    }

    *valor = numero;
    return 1;
}

static int operar_seguro(char op, long long a,
                         long long b, long long *r) {
    switch (op) {
        case '+':
            if ((b > 0 && a > LLONG_MAX - b) ||
                (b < 0 && a < LLONG_MIN - b)) {
                return 0;
            }
            *r = a + b;
            return 1;

        case '-':
            if ((b < 0 && a > LLONG_MAX + b) ||
                (b > 0 && a < LLONG_MIN + b)) {
                return 0;
            }
            *r = a - b;
            return 1;

        case '*':
            if (a > 0) {
                if ((b > 0 && a > LLONG_MAX / b) ||
                    (b < 0 && b < LLONG_MIN / a)) {
                    return 0;
                }
            } else if (a < 0) {
                if ((b > 0 && a < LLONG_MIN / b) ||
                    (b < 0 && a < LLONG_MAX / b)) {
                    return 0;
                }
            }
            *r = a * b;
            return 1;

        case '/':
            if (b == 0 || (a == LLONG_MIN && b == -1)) {
                return 0;
            }
            *r = a / b;
            return 1;

        default:
            return 0;
    }
}

static int escribir(char *dst, int cap, const char *formato,
                    const char *res, const char *valor) {
    int n = snprintf(dst, (size_t)cap, formato, res, valor);

    if (n < 0 || n >= cap) {
        dst[0] = '\0';
        return 0;
    }

    return 1;
}

void plegar(char *dst, int cap, const char *res,
            const char *op, const char *a, const char *b) {
    long long x = 0, y = 0, z = 0;
    int es_a, es_b;
    int n;
    char resultado[32];

    if (dst == NULL || cap <= 0) {
        return;
    }

    dst[0] = '\0';

    if (res == NULL || op == NULL || a == NULL || b == NULL) {
        return;
    }

    es_a = convertir_entero(a, &x);
    es_b = convertir_entero(b, &y);

    if (op[0] != '\0' && op[1] == '\0' &&
        es_a && es_b && operar_seguro(op[0], x, y, &z)) {
        n = snprintf(resultado, sizeof(resultado), "%lld", z);

        if (n < 0 || (size_t)n >= sizeof(resultado)) {
            return;
        }

        (void)escribir(dst, cap, "%s = %s", res, resultado);
        return;
    }

    if (op[0] == '+' && op[1] == '\0') {
        if (es_b && y == 0) {
            (void)escribir(dst, cap, "%s = %s", res, a);
            return;
        }
        if (es_a && x == 0) {
            (void)escribir(dst, cap, "%s = %s", res, b);
            return;
        }
    }

    if (op[0] == '*' && op[1] == '\0') {
        if (es_b && y == 1) {
            (void)escribir(dst, cap, "%s = %s", res, a);
            return;
        }
        if (es_a && x == 1) {
            (void)escribir(dst, cap, "%s = %s", res, b);
            return;
        }
        if ((es_b && y == 0) || (es_a && x == 0)) {
            (void)snprintf(dst, (size_t)cap, "%s = 0", res);
            if (strlen(dst) >= (size_t)cap - 1) {
                dst[0] = '\0';
            }
            return;
        }
    }

    n = snprintf(dst, (size_t)cap, "%s = %s %s %s",
                 res, a, op, b);

    if (n < 0 || n >= cap) {
        dst[0] = '\0';
    }
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Simplificación de expresiones constantes

### Salida del harness
```text
=== AUDITORIA SESION 28 - Plegado de constantes e identidades ===
Caso                               esperado:              obtenido:              resultado
[ 1] 3 + 5                              esperado: _t0 = 8      obtenido: _t0 = 8      OK
[ 2] 2 * 3                              esperado: _t0 = 6      obtenido: _t0 = 6      OK
[ 3] 7 - 10                             esperado: _t0 = -3     obtenido: _t0 = -3     OK
[ 4] x + 0                              esperado: _t0 = x      obtenido: _t0 = x      OK
[ 5] x + y (sin plegar)                 esperado: _t0 = x + y  obtenido: _t0 = x + y  OK
[ 6] 10 / 0 (no plegar)                 esperado: _t0 = 10 / 0 obtenido: _t0 = 10 / 0 OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion28.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión del código identifiqué que el harness únicamente comprueba seis situaciones principales, por lo que no garantiza que todos los casos límite estén cubiertos.

Un caso importante es la multiplicación de dos enteros cuyo resultado supera el rango permitido por long long. También se debe considerar la división de LLONG_MIN entre -1, porque esa operación puede provocar un desbordamiento.

Otro caso importante es cuando el destino tiene una capacidad insuficiente para almacenar la instrucción completa. Si no se comprueba el resultado de snprintf(), podría quedar una cadena incompleta.

Para solucionar estos problemas se incorporaron validaciones de rango, comprobaciones del resultado de las operaciones aritméticas y verificaciones de capacidad antes de aceptar la cadena generada.

---
**Firma digital:** [BlancaHoma14]
