# Bitácora de Requerimientos y Auditoría IA — Sesión 13
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Manejo de Ámbitos (Scope Global/Local)  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [No.24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Pila de tablas de símbolos (Stack Scopes)  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al programar a mano, el mayor reto fue asegurar que las declaraciones de variables se limitaran estrictamente al bloque actual sin buscar en los ámbitos padre, permitiendo de este modo el fenómeno de *shadowing* (ocultamiento). Asimismo, se debió diseñar el recorrido iterativo mediante el apuntador `padre` para resolver las búsquedas jerárquicas desde los bloques más internos hacia la raíz.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Ocultamiento de variables (Shadowing) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Contexto]: Desarrollo de un sistema de ámbitos anidados (pila de scopes) para el análisis semántico de un compilador en C99, cumpliendo restricciones estrictas de compilación (-Wall -Wextra -Werror).

[Rol]: Ingeniero de software experto en estructuras de datos jerárquicas y manejo de tablas de símbolos con ámbitos léxicos en lenguaje C.

[Especificación]: 
1. Implementar la función `int declarar(Ambito *ambito, const char *nombre);` que registre un nombre únicamente en el ámbito actual proporcionado. Debe validar argumentos nulos o cadenas vacías (retornando -1), verificar si el espacio está lleno respecto a `AMBITO_MAX` (retornando -1), comprobar si el nombre ya existe en el propio ámbito sin consultar a los padres (retornando 0 si se duplica) y retornar 1 si la inserción es exitosa.
2. Implementar la función `const char *buscar(const Ambito *ambito, const char *nombre);` que busque el identificador comenzando desde el ámbito actual y subiendo de forma iterativa por los punteros `padre` hacia la raíz (implementando el ocultamiento o shadowing). Debe retornar el campo `nombre` del ámbito en el que se localizó el símbolo, o `NULL` si no se encuentra en ningún nivel de la jerarquía.

[Output]: Únicamente el código modular en C compatible con el contrato "api_sesion13.h", sin función main().
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion13.h"
#include <string.h>

int declarar(Ambito *ambito, const char *nombre) {
    if (!ambito || !nombre || nombre[0] == '\0') return -1;

    for (int i = 0; i < ambito->total; i++) {
        if (strcmp(ambito->nombres[i], nombre) == 0) {
            return 0;
        }
    }

    if (ambito->total >= AMBITO_MAX) return -1;

    ambito->nombres[ambito->total++] = nombre;
    return 1;
}

const char *buscar(const Ambito *ambito, const char *nombre) {
    const Ambito *curr = ambito;
    while (curr) {
        if (nombre) {
            for (int i = 0; i < curr->total; i++) {
                if (strcmp(curr->nombres[i], nombre) == 0) {
                    return curr->nombre;
                }
            }
        }
        curr = curr->padre;
    }
    return NULL;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Búsqueda jerárquica de identificadores

### Salida del harness
```text
=== AUDITORIA SESION 13 - Pila de ambitos y busqueda jerarquica ===
Caso                                esperado:             obtenido:             resultado
[ 1] declarar(&g, "x")              esperado: 1           obtenido: 1           OK
[ 2] buscar en el mismo ambito      esperado: global      obtenido: global      OK
[ 3] buscar subiendo al padre       esperado: global      obtenido: global      OK
[ 4] shadowing en el hijo           esperado: bloque      obtenido: bloque      OK
[ 5] tres niveles hasta la raiz     esperado: raiz        obtenido: raiz        OK
[ 6] nombre no declarado            esperado: (nulo)      obtenido: (nulo)      OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> En su primera iteración, la IA omitió validar adecuadamente los límites de capacidad del arreglo local de nombres (AMBITO_MAX) y la revisión de cadenas vacías o punteros nulos. Esto se solucionó añadiendo protecciones explícitas al inicio de la función declarar para asegurar que el sistema devuelva -1 ante cualquier argumento inválido o desbordamiento de memoria estática.

---
**Firma digital:** [BlancaHoma14]
