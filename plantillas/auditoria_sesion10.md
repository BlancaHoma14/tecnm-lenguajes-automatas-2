# Bitácora de Requerimientos y Auditoría IA — Sesión 10
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 1: Mini-Evaluador de Expresiones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración AST y Tipado Estático  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el código, identifiqué que la principal dificultad era organizar las funciones primaria(), termino() y expr() para que trabajaran de manera coordinada y respetaran la precedencia de los operadores matemáticos.

También tuve que considerar que las multiplicaciones y divisiones deben resolverse antes que las sumas y restas. Para conseguirlo utilicé llamadas entre funciones, de manera que expr() llamara a termino() y esta, a su vez, llamara a primaria().

Otra dificultad fue reconocer los paréntesis y comprobar que estuvieran correctamente cerrados. Además, tuve que controlar los errores para distinguir entre una división entre cero, identificada con el código 1, y una expresión con sintaxis inválida, identificada con el código 2.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato formal de Frontend Semántico |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO:
Estoy trabajando en la Sesión 10 de Lenguajes y Autómatas II.
Necesito desarrollar un mini-evaluador de expresiones aritméticas
en lenguaje C, utilizando un analizador recursivo descendente.

ROL:
Actúa como un ingeniero de software experto en lenguaje C,
compiladores, análisis sintáctico y evaluación de expresiones.

ESPECIFICACIÓN:
Implementa y completa las funciones primaria(), termino()
y expr() utilizando la estructura Parser proporcionada.

El programa debe:
1. Leer números enteros desde una cadena de texto.
2. Reconocer los operadores +, -, * y /.
3. Respetar la precedencia de operadores.
4. Resolver expresiones dentro de paréntesis.
5. Ignorar los espacios entre los elementos.
6. Reportar error 1 cuando exista división entre cero.
7. Reportar error 2 cuando exista sintaxis inválida.
8. Propagar los errores sin continuar realizando operaciones.
9. Detectar caracteres sobrantes al terminar de analizar.
10. Mantener la firma:
    long evaluar_expresion(const char *fuente, int *error);
11. No incluir main(), porque el harness ya lo proporciona.
12. No modificar el contrato ni el harness.
13. Compilar con:
    gcc -std=c11 -Wall -Wextra -Werror

OUTPUT:
Entrega el código completo para ia_sesion10_c.c.
Explica las mejoras respecto a una implementación manual,
los casos límite considerados y la complejidad temporal
y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion10_c.c
/* ia_sesion10_c.c - Fase 2: El Cientifico
 * Mini-evaluador de expresiones (Hito 1).
 */

#include "api_sesion10.h"
#include 
#include 
#include 

typedef struct {
    const char *s;
    int error;
} Parser;

/* Ignorar espacios */
static void saltar(Parser *p) {
    while (*p->s == ' ') {
        p->s++;
    }
}

long expr(Parser *p);

/* Leer numeros y resolver parentesis */
long primaria(Parser *p) {
    long valor = 0;

    saltar(p);

    /* Resolver expresiones entre parentesis */
    if (*p->s == '(') {
        p->s++;

        valor = expr(p);

        /* No continuar si ya existe un error */
        if (p->error != 0) {
            return 0;
        }

        saltar(p);

        if (*p->s != ')') {
            p->error = 2;
            return 0;
        }

        p->s++;
        return valor;
    }

    /* Leer un numero entero */
    if (isdigit((unsigned char)*p->s)) {
        while (isdigit((unsigned char)*p->s)) {
            valor = valor * 10 + (*p->s - '0');
            p->s++;
        }

        return valor;
    }

    /* Reportar error de sintaxis */
    p->error = 2;
    return 0;
}

/* Resolver multiplicaciones y divisiones */
long termino(Parser *p) {
    long valor = primaria(p);

    while (p->error == 0) {
        char operador;
        long siguiente;

        saltar(p);
        operador = *p->s;

        if (operador != '*' && operador != '/') {
            break;
        }

        p->s++;
        siguiente = primaria(p);

        /* Propagar errores */
        if (p->error != 0) {
            return 0;
        }

        if (operador == '*') {
            valor *= siguiente;
        } else {
            if (siguiente == 0) {
                p->error = 1;
                return 0;
            }

            valor /= siguiente;
        }
    }

    return valor;
}

/* Resolver sumas y restas */
long expr(Parser *p) {
    long valor = termino(p);

    while (p->error == 0) {
        char operador;
        long siguiente;

        saltar(p);
        operador = *p->s;

        if (operador != '+' && operador != '-') {
            break;
        }

        p->s++;
        siguiente = termino(p);

        /* No operar si el analizador detecto un error */
        if (p->error != 0) {
            return 0;
        }

        if (operador == '+') {
            valor += siguiente;
        } else {
            valor -= siguiente;
        }
    }

    return valor;
}

/* Funcion principal del evaluador */
long evaluar_expresion(const char *fuente, int *error) {
    static int sin_error;

    if (error == NULL) {
        error = &sin_error;
    }

    *error = 0;

    if (fuente == NULL) {
        *error = 2;
        return 0;
    }

    Parser p = { fuente, 0 };

    long valor = expr(&p);

    saltar(&p);

    /* Detectar caracteres sobrantes */
    if (p.error == 0 && *p.s != '\0') {
        p.error = 2;
    }

    *error = p.error;

    return p.error != 0 ? 0 : valor;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Precedencia, paréntesis, división entre cero y sintaxis inválida

### Salida del harness
```text
=== AUDITORIA SESION 10 - Mini-evaluador de expresiones (Hito 1) ===
Caso                            esperado:              obtenido:              resultado
[ 1] "2+3*4"                        esperado: 14           obtenido: 14           OK
[ 2] "(2+3)*4"                      esperado: 20           obtenido: 20           OK
[ 3] "10/0" -> error 1              esperado: 1            obtenido: 1            OK
[ 4] "2+" -> error 2                esperado: 2            obtenido: 2            OK
[ 5] "  7 "                         esperado: 7            obtenido: 7            OK
[ 6] "5-10"                         esperado: -5           obtenido: -5           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion10.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que no contempla todos los posibles casos límite. Por ejemplo, la función saltar() solamente reconoce el espacio en blanco normal ' ', por lo que no reconoce otros caracteres de separación, como tabulaciones (\t) o saltos de línea (\n).

También observé que el código no comprueba si las operaciones aritméticas con valores de tipo long producen un desbordamiento. Esto podría ocasionar resultados incorrectos cuando se introducen números demasiado grandes.

Para mejorar la solución, identifiqué que se puede utilizar isspace() de la biblioteca ctype.h dentro de saltar() para reconocer diferentes tipos de espacios en blanco. Además, sería necesario agregar comprobaciones de límites antes de realizar operaciones aritméticas para prevenir desbordamientos.

Estos casos no están incluidos en los seis ejercicios originales del harness, por lo que deben probarse adicionalmente.

---
**Firma digital:** [Blanca Janet Homa May]
