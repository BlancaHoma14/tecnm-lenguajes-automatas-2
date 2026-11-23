# Bitácora de Requerimientos y Auditoría IA — Sesión 20
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Hito 2: Analizador Semántico y Scopes  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Integración de Tabla de Símbolos y AST  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el código, mi principal dificultad fue comprender cómo recorrer los diferentes ámbitos mediante el puntero padre para encontrar una variable y determinar su profundidad.

También tuve dificultad al implementar la función contar_visibles(), porque debía evitar que un mismo nombre se contara varias veces cuando aparecía declarado en diferentes ámbitos. Tuve que considerar el concepto de shadowing, donde la declaración del ámbito más interno tiene prioridad sobre las declaraciones externas.

Otra restricción que identifiqué fue el tamaño máximo del arreglo simbolos, definido mediante AMBITO_MAX = 8. Por eso, fue necesario controlar los límites del arreglo y verificar que los punteros y nombres no fueran NULL antes de utilizarlos.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Contrato de Validación Semántica Integral |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Estoy trabajando en Lenguajes y Autómatas II, Sesión 20,
utilizando lenguaje C11 y el compilador GCC. Tengo una
estructura llamada Ambito que contiene un nombre, un arreglo
de símbolos con capacidad máxima AMBITO_MAX, un contador
de símbolos y un puntero al ámbito padre.

ROL: Actúa como un ingeniero de software especializado en
compiladores, análisis semántico, tablas de símbolos
y administración de ámbitos.

ESPECIFICACIÓN: Implementa las funciones profundidad_de() y contar_visibles()
respetando exactamente las firmas establecidas en
api_sesion20.h.

1. profundidad_de() debe buscar un nombre desde el ámbito
   actual hacia sus ámbitos padres.
2. Debe devolver 0 si el nombre está en el ámbito actual.
3. Debe devolver el nivel correspondiente si está en un
   ámbito superior.
4. Debe devolver -1 si el nombre no existe.
5. Debe devolver -2 si alguno de sus argumentos es NULL.
6. contar_visibles() debe contar cada nombre una sola vez.
7. Debe respetar el shadowing, dando prioridad al ámbito
   más interno.
8. Debe evitar accesos fuera de los límites del arreglo
   simbolos.
9. Debe ignorar los nombres NULL.
10. No debe modificar las estructuras originales ni utilizar
    memoria dinámica.
11. No debe incluir una función main().
12. Debe compilar con -std=c11 -Wall -Wextra -Werror.

OUTPUT: Entrega el código completo y modular en C, respetando las
firmas originales, con comentarios explicativos y un análisis
de complejidad temporal y espacial. No modifiques
api_sesion20.h ni el contrato del harness.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion20.h"
#include <string.h>

/* Comprueba si un nombre existe en un ambito */
static int buscar_simbolo(const Ambito *ambito,
                          const char *nombre) {
    if (ambito == NULL || nombre == NULL) {
        return 0;
    }

    int total = ambito->total;

    if (total < 0) {
        total = 0;
    }

    if (total > AMBITO_MAX) {
        total = AMBITO_MAX;
    }

    for (int i = 0; i < total; i++) {
        const char *simbolo = ambito->simbolos[i];

        if (simbolo != NULL &&
            strcmp(simbolo, nombre) == 0) {
            return 1;
        }
    }

    return 0;
}

/* Busca el nombre y devuelve su profundidad */
int profundidad_de(const Ambito *actual,
                   const char *nombre) {
    if (actual == NULL || nombre == NULL) {
        return -2;
    }

    int nivel = 0;
    const Ambito *ambito = actual;

    while (ambito != NULL) {
        if (buscar_simbolo(ambito, nombre)) {
            return nivel;
        }

        ambito = ambito->padre;
        nivel++;
    }

    return -1;
}

/* Cuenta los nombres diferentes que son visibles */
int contar_visibles(const Ambito *actual) {
    if (actual == NULL) {
        return -2;
    }

    int total_visibles = 0;
    const Ambito *ambito = actual;

    while (ambito != NULL) {
        int total = ambito->total;

        if (total < 0) {
            total = 0;
        }

        if (total > AMBITO_MAX) {
            total = AMBITO_MAX;
        }

        for (int i = 0; i < total; i++) {
            const char *nombre = ambito->simbolos[i];

            if (nombre == NULL) {
                continue;
            }

            int oculto = 0;

            /* Comprueba si un ambito mas interno
               ya declara el mismo nombre */
            const Ambito *interno = actual;

            while (interno != ambito) {
                if (buscar_simbolo(interno, nombre)) {
                    oculto = 1;
                    break;
                }

                interno = interno->padre;
            }

            /* Comprueba duplicados del mismo ambito */
            for (int j = 0; j < i && !oculto; j++) {
                const char *anterior = ambito->simbolos[j];

                if (anterior != NULL &&
                    strcmp(anterior, nombre) == 0) {
                    oculto = 1;
                }
            }

            if (!oculto) {
                total_visibles++;
            }
        }

        ambito = ambito->padre;
    }

    return total_visibles;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Casos complejos de shadowing y ámbito

### Salida del harness
```text
=== AUDITORIA SESION 20 - Resolucion de nombres y visibilidad ===
Caso                               esperado:              obtenido:              resultado
[ 1] nombre en el ambito actual         esperado: 0            obtenido: 0            OK
[ 2] nombre dos niveles arriba          esperado: 2            obtenido: 2            OK
[ 3] shadowing gana el interno          esperado: 0            obtenido: 0            OK
[ 4] nombre no visible                  esperado: -1           obtenido: -1           OK
[ 5] visibles sin repetir               esperado: 3            obtenido: 3            OK
[ 6] ambito NULL                        esperado: -2           obtenido: -2           OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion20.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que no contempla la detección de ciclos en la cadena de ámbitos. Si un ámbito tuviera como padre a otro que finalmente vuelve a apuntar al primero, los recorridos podrían continuar indefinidamente.

Para evitar este problema durante la actividad, mantuve la estructura de ámbitos como una cadena jerárquica sin ciclos, verificando que cada ámbito tenga correctamente asignado su padre. También conservé las comprobaciones de límites del arreglo simbolos, ignoré los nombres NULL y verifiqué los nombres repetidos dentro del mismo ámbito.

De esta manera, la implementación cumple los casos establecidos por el harness bajo la condición de que la cadena de ámbitos esté correctamente construida.

---
**Firma digital:** [BlancaHoma14]
