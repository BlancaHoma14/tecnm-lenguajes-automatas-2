# Bitácora de Requerimientos y Auditoría IA — Sesión 21
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Fundamentos de Código de 3 Direcciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Representación en Cuádruplos (op,a,b,r)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el código, identifiqué que una de las principales dificultades era generar nombres temporales únicos y consecutivos, como _t0, _t1, _t2, sin repetirlos.

También tuve que considerar las restricciones de memoria al escribir los resultados en un arreglo de caracteres. Si el tamaño del búfer no es suficiente, existe el riesgo de que la cadena quede incompleta o se produzcan problemas al manipularla.

Otra dificultad fue respetar el formato de los cuádruplos, ya que cada uno debe contener cuatro elementos: operador, primer operando, segundo operando y resultado.

Para resolverlo manualmente, utilicé snprintf() para escribir las cadenas dentro del espacio disponible y un contador entero para generar los temporales.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Generación de temporales únicos (_t0..) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Estoy trabajando en la materia Lenguajes y Autómatas II,
Sesión 21: Fundamentos de Código de Tres Direcciones (TAC).
Utilizo lenguaje C y el compilador GCC. Tengo un archivo
api_sesion21.h que contiene las declaraciones de las funciones.

ROL: Actúa como un ingeniero de software especializado en
compiladores, generación de código intermedio y lenguaje C.

ESPECIFICACIÓN: Implementa las funciones nuevo_temporal() y emitir_cuadruplo()
respetando exactamente las firmas declaradas en api_sesion21.h.

1. nuevo_temporal() debe generar identificadores consecutivos
   con el formato _t0, _t1, _t2, etc.
2. Debe utilizar el contador recibido y aumentarlo después
   de generar cada temporal.
3. emitir_cuadruplo() debe producir una cadena con el formato:
   (op, a1, a2, res)
4. Si alguno de los campos del cuádruplo es NULL,
   debe sustituirse por el carácter _.
5. Ambas funciones deben validar los punteros y la capacidad
   del búfer antes de escribir.
6. Utiliza snprintf() para evitar escrituras fuera del búfer.
7. No modifiques el archivo api_sesion21.h.
8. No agregues una función main().
9. El código debe compilar con:
   gcc -std=c11 -Wall -Wextra -Werror

OUTPUT: Entrega el código completo y modular en C, con comentarios
explicativos y análisis de complejidad temporal y espacial.
Explica las mejoras realizadas y considera los casos límite.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion21_c.c - Fase 2: El Cientifico
 * Fundamentos de Codigo de Tres Direcciones (TAC).
 */

#include "api_sesion21.h"
#include <stdio.h>
#include <limits.h>

/* Sustituye los campos NULL por "_" */
static const char *campo_seguro(const char *texto) {
    return texto != NULL ? texto : "_";
}

/* Genera un identificador temporal */
void nuevo_temporal(char *dst, int cap, int *contador) {

    if (dst == NULL || contador == NULL || cap <= 0) {
        return;
    }

    snprintf(dst, (size_t)cap, "_t%d", *contador);

    /* Evita desbordamiento del contador */
    if (*contador < INT_MAX) {
        (*contador)++;
    }
}

/* Genera la representacion de un cuadruplo */
void emitir_cuadruplo(char *dst, int cap,
                      const char *op, const char *a1,
                      const char *a2, const char *res) {

    if (dst == NULL || cap <= 0) {
        return;
    }

    snprintf(dst, (size_t)cap, "(%s, %s, %s, %s)",
             campo_seguro(op),
             campo_seguro(a1),
             campo_seguro(a2),
             campo_seguro(res));
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Expresiones aritméticas compuestas

### Salida del harness
```text
=== AUDITORIA SESION 21 - Cuadruplos y temporales unicos ===
Caso                               esperado:              obtenido:              resultado
[ 1] nuevo_temporal(_t0)                esperado: _t0          obtenido: _t0          OK
[ 2] nuevo_temporal(_t7)                esperado: _t7          obtenido: _t7          OK
[ 3] cuadruplo de a + b
     esperado: (+, a, b, _t0)
     obtenido: (+, a, b, _t0)
     -> OK
[ 4] cuadruplo de _t0 * c
     esperado: (*, _t0, c, _t1)
     obtenido: (*, _t0, c, _t1)
     -> OK
[ 5] asignacion simple                  esperado: (=, a, _, x) obtenido: (=, a, _, x) OK
[ 6] goto sin operandos
     esperado: (goto, L1, _, _)
     obtenido: (goto, L1, _, _)
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion21.md

```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La IA no consideró inicialmente el posible desbordamiento del contador entero al generar los identificadores temporales. Si el contador llegara a su valor máximo (INT_MAX) y se incrementara nuevamente, podría producirse un desbordamiento de entero con signo.

Para solucionar este problema, agregué la biblioteca limits.h y una condición que comprueba que el contador sea menor que INT_MAX antes de incrementarlo.

También consideré el caso en el que alguno de los campos del cuádruplo fuera NULL. Para evitar generar cadenas incorrectas o acceder a punteros nulos, utilicé la función auxiliar campo_seguro(), que sustituye esos valores por _.

Finalmente, identifiqué que snprintf() evita escribir más allá de la capacidad indicada, pero puede truncar el resultado si el búfer es demasiado pequeño. Por eso, es importante proporcionar una capacidad suficiente para almacenar el temporal o el cuádruplo completo.
---
**Firma digital:** [BlancaHoma14]
