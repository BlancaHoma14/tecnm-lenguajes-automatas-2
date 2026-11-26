# Bitácora de Requerimientos y Auditoría IA — Sesión 27
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Indexación de Arreglos en TAC  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de desplazamientos base + offset  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente las funciones emitir_acceso() y emitir_acceso_2d(), identifiqué que la principal dificultad era calcular correctamente el desplazamiento de memoria de cada elemento del arreglo.
En el arreglo unidimensional tuve que multiplicar el índice por el ancho de cada elemento para obtener el desplazamiento. Posteriormente, utilicé ese resultado para generar la instrucción de lectura del arreglo.
En el arreglo bidimensional tuve que realizar más operaciones, respetando el orden de la fórmula: i*filas + j
Después multipliqué el índice lineal por el ancho del elemento para obtener el desplazamiento definitivo.

> También identifiqué las siguientes restricciones:
> - Cada instrucción debe almacenarse en un espacio máximo de 64 caracteres.
> - Los temporales deben generarse consecutivamente y sin repetir nombres.
> - La capacidad del arreglo lineas debe ser suficiente para almacenar todas las instrucciones.
> - Los parámetros no deben ser NULL cuando sean necesarios.
> - El contador de temporales no debe superar el límite de un entero.
> La dificultad principal fue evitar que una instrucción incompleta o demasiado larga dejara el arreglo parcialmente modificado.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Aplanado de direcciones unidimensionales |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
**CONTRATO C-R-E-O**

**CONTEXTO:** Estoy trabajando en la sesión 27 de Lenguajes y Autómatas II
(SCD-1016). El tema es la indexación de arreglos en código
intermedio TAC. Utilizo lenguaje C y GCC.

**ROL:** Actúa como ingeniero de software especializado en compiladores,
generación de código intermedio y administración de memoria.

**ESPECIFICACIÓN:** Implementa las funciones respetando exactamente las firmas:

int emitir_acceso(char lineas[][64], int max,
                  const char *base, const char *indice,
                  int ancho, int *temporales);

int emitir_acceso_2d(char lineas[][64], int max,
                     const char *base, const char *i,
                     const char *j, int filas, int ancho,
                     int *temporales);

Requisitos para emitir_acceso:
1. Calcular el desplazamiento multiplicando el índice por el ancho.
2. Generar la instrucción de lectura base[offset].
3. Utilizar dos temporales consecutivos.
4. Incrementar el contador de temporales en dos.

Requisitos para emitir_acceso_2d:
1. Calcular i * filas.
2. Sumar j al resultado anterior.
3. Multiplicar el índice lineal por el ancho.
4. Generar la lectura base[offset].
5. Utilizar cuatro temporales consecutivos.
6. Incrementar el contador de temporales en cuatro.

Restricciones:
- Mantener exactamente las firmas originales.
- Si base es NULL, no generar instrucciones y devolver 0.
- Validar apuntadores, dimensiones, ancho y capacidad.
- Cada instrucción debe caber en 64 caracteres.
- Evitar desbordamiento del contador de temporales.
- Comprobar el resultado de snprintf().
- Evitar modificaciones parciales del arreglo lineas.
- No utilizar memoria dinámica.
- No incluir main().
- Compatible con C11 y GCC usando -Wall -Wextra -Werror.

**OUTPUT:** Entrega código modular, explicación del funcionamiento,
mejoras respecto a una versión manual y análisis de
complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion27_c.c

#include "api_sesion27.h"
#include <stdio.h>
#include <string.h>
#include <limits.h>

int emitir_acceso(char lineas[][64], int max,
                  const char *base, const char *indice,
                  int ancho, int *temporales) {
    char temp[2][64] = {{0}};
    int t;
    int r;

    if (base == NULL) return 0;

    if (lineas == NULL || indice == NULL ||
        temporales == NULL || max < 2 ||
        ancho <= 0 || *temporales < 0 ||
        *temporales > INT_MAX - 2) {
        return 0;
    }

    t = *temporales;

    r = snprintf(temp[0], sizeof(temp[0]),
                 "_t%d = %s * %d", t, indice, ancho);
    if (r < 0 || (size_t)r >= sizeof(temp[0])) {
        return 0;
    }

    r = snprintf(temp[1], sizeof(temp[1]),
                 "_t%d = %s[_t%d]", t + 1, base, t);
    if (r < 0 || (size_t)r >= sizeof(temp[1])) {
        return 0;
    }

    memcpy(lineas, temp, sizeof(temp));

    *temporales += 2;
    return 2;
}

int emitir_acceso_2d(char lineas[][64], int max,
                     const char *base, const char *i,
                     const char *j, int filas, int ancho,
                     int *temporales) {
    char temp[4][64] = {{0}};
    int t;
    int r;

    if (base == NULL) return 0;

    if (lineas == NULL || i == NULL || j == NULL ||
        temporales == NULL || max < 4 ||
        filas <= 0 || ancho <= 0 ||
        *temporales < 0 || *temporales > INT_MAX - 4) {
        return 0;
    }

    t = *temporales;

    /* Paso 1: calcular i * filas. */
    r = snprintf(temp[0], sizeof(temp[0]),
                 "_t%d = %s * %d", t, i, filas);
    if (r < 0 || (size_t)r >= sizeof(temp[0])) {
        return 0;
    }

    /* Paso 2: sumar j al resultado anterior. */
    r = snprintf(temp[1], sizeof(temp[1]),
                 "_t%d = _t%d + %s", t + 1, t, j);
    if (r < 0 || (size_t)r >= sizeof(temp[1])) {
        return 0;
    }

    /* Paso 3: multiplicar el índice lineal por el ancho. */
    r = snprintf(temp[2], sizeof(temp[2]),
                 "_t%d = _t%d * %d", t + 2, t + 1, ancho);
    if (r < 0 || (size_t)r >= sizeof(temp[2])) {
        return 0;
    }

    /* Paso 4: leer el elemento del arreglo. */
    r = snprintf(temp[3], sizeof(temp[3]),
                 "_t%d = %s[_t%d]", t + 3, base, t + 2);
    if (r < 0 || (size_t)r >= sizeof(temp[3])) {
        return 0;
    }

    memcpy(lineas, temp, sizeof(temp));

    *temporales += 4;
    return 4;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Arreglos multidimensionales en TAC

### Salida del harness
```text
=== AUDITORIA SESION 27 - Desplazamientos para arreglos 1D y 2D ===
Caso                               esperado:              obtenido:              resultado
[ 1] 1D: i * 4                          esperado: _t0 = i * 4  obtenido: _t0 = i * 4  OK
[ 2] 1D: A[_t0]                         esperado: _t1 = A[_t0] obtenido: _t1 = A[_t0] OK
[ 3] 2D: i * 3                          esperado: _t0 = i * 3  obtenido: _t0 = i * 3  OK
[ 4] 2D: offset por ancho
     esperado: _t2 = _t1 * 4
     obtenido: _t2 = _t1 * 4
     -> OK
[ 5] 2D: M[_t2]                         esperado: _t3 = M[_t2] obtenido: _t3 = M[_t2] OK
[ 6] base NULL (sin acceso)             esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion27.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la auditoría identifiqué que el harness proporcionado comprueba los seis casos principales, pero no evalúa directamente situaciones como recibir un índice demasiado largo, tener una capacidad insuficiente en el arreglo lineas o utilizar un contador de temporales cercano al límite de un entero.

> Para atender estas situaciones, incorporé validaciones adicionales en el código mejorado:
> - Comprobación de la capacidad mínima: dos instrucciones para 1D y cuatro para 2D.
> - Verificación del tamaño de las cadenas generadas con snprintf().
> - Validación del contador antes de generar nuevos temporales.
> - Comprobación de que las dimensiones y el ancho sean mayores que cero.
> - Preparación de todas las instrucciones en arreglos temporales antes de copiarlas al resultado definitivo.
> También observé que el harness no comprueba directamente el valor final del contador de temporales. Por eso, considero importante verificarlo en pruebas adicionales para asegurar que cada función incremente correctamente el contador.
> Estas validaciones permiten mejorar la seguridad de la generación de instrucciones y evitar modificaciones parciales cuando ocurre un error.

---
**Firma digital:** [BlancaHoma14]
