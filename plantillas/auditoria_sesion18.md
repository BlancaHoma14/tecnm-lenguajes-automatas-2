# Bitácora de Requerimientos y Auditoría IA — Sesión 18
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arreglos y Verificación de Límites  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Cálculo de tamaño de arreglo en memoria  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al comparar mi código manual con el que mejoré utilizando IA, observé que la segunda versión tiene una mejor organización, ya que agregué una función auxiliar llamada `obtener_tamano()` para identificar el espacio que ocupa cada tipo de dato. También incorporé la biblioteca `limits.h` para comprobar que la multiplicación no sobrepase el valor máximo permitido para una variable de tipo `long`. Además, mejoré las validaciones para detectar tipos nulos, cantidades negativas y evitar posibles errores al calcular el tamaño de los arreglos. De esta manera, el código quedó más ordenado, seguro y fácil de comprender.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Indexación entera no negativa |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[C – Contexto: Estoy trabajando en un proyecto de Lenguajes y Autómatas II utilizando el lenguaje C99. Necesito implementar dos funciones para calcular el tamaño de los arreglos y verificar sus límites.

R – Rol: Actúa como un ingeniero de software especializado en programación en C, estructuras de datos y desarrollo de compiladores.

E – Especificación: Desarrolla las funciones bytes_arreglo() e indice_en_rango() respetando exactamente las firmas declaradas en api_sesion18.h.
La función bytes_arreglo() debe:
*   Reconocer los tipos char, short, int, float, double y puntero.
*   Obtener el tamaño de cada tipo utilizando sizeof().
*   Multiplicar el tamaño por la cantidad de elementos.
*   Devolver -1 si el tipo es desconocido, es NULL o si la cantidad de elementos es negativa.
*   Devolver 0 cuando la cantidad de elementos sea cero.
*   Evitar desbordamientos al calcular el tamaño total.

La función indice_en_rango() debe:
*   Devolver 1 si el índice es válido.
*   Devolver 0 si el índice es negativo, si es igual o mayor que la cantidad de elementos o si la cantidad de elementos no es positiva.
*   Evitar comparaciones que puedan provocar errores con valores negativos.

O – Output: Entrega código modular en C99, sin función main(), compatible con -Wall -Wextra -Werror. Incluye las dos funciones solicitadas y explica las mejoras realizadas y su complejidad temporal y espacial.

]
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion18_c.c
/* ia_sesion18_c.c - Fase 2: El Cientifico
 *
 * Pega aqui el codigo que te devolvio la IA, con la MISMA firma
 * que declara api_sesion18.h. No puede traer main().
 */
#include "api_sesion18.h"
#include <string.h>
#include <limits.h>

static long obtener_tamano(const char *tipo)
{
    if (tipo == NULL)
    {
        return -1;
    }

    if (strcmp(tipo, "char") == 0)
        return (long)sizeof(char);

    if (strcmp(tipo, "short") == 0)
        return (long)sizeof(short);

    if (strcmp(tipo, "int") == 0)
        return (long)sizeof(int);

    if (strcmp(tipo, "float") == 0)
        return (long)sizeof(float);

    if (strcmp(tipo, "double") == 0)
        return (long)sizeof(double);

    if (strcmp(tipo, "puntero") == 0)
        return (long)sizeof(void *);

    return -1;
}

long bytes_arreglo(const char *tipo, int elementos)
{
    long tamano = obtener_tamano(tipo);

    if (tamano < 0 || elementos < 0)
    {
        return -1;
    }

    if (elementos == 0)
    {
        return 0;
    }

    if ((long)elementos > LONG_MAX / tamano)
    {
        return -1;
    }

    return tamano * (long)elementos;
}

int indice_en_rango(int indice, int elementos)
{
    if (elementos <= 0)
    {
        return 0;
    }

    return (indice >= 0 && indice < elementos) ? 1 : 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Límites estáticos y dimensiones inválidas

              === AUDITORIA SESION 18 - Tamano de arreglo y limites ===
              Caso                esperado:          obtenido:          resultado
        [ 1] int[10]              esperado: 40       obtenido: 40          OK
        [ 2] double[3]            esperado: 24       obtenido: 24          OK
        [ 3] int[-4]              esperado: -1       obtenido: -1          OK
        [ 4] indice 0 de 5        esperado: 1        obtenido: 1           OK
        [ 5] indice 5 de 5        esperado: 0        obtenido: 0           OK
        [ 6] indice negativo      esperado: 0        obtenido: 0           OK

        
### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas

```
RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion18.md

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código que generó la IA, identifiqué que era importante considerar el caso en que la multiplicación del tamaño de los elementos pudiera superar el límite máximo de una variable de tipo `long`. Si no se tomaba en cuenta esta situación, el resultado del cálculo podría ser incorrecto. Para solucionarlo, agregué una validación utilizando `LONG_MAX`, que permite comprobar si la cantidad de elementos puede multiplicarse sin provocar un desbordamiento. Con esta mejora conseguí que el programa fuera más seguro y que también pudiera manejar correctamente valores que superen los límites permitidos.

---
**Firma digital:** [BlancaHoma14]
