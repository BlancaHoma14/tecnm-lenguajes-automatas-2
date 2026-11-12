# Bitácora de Requerimientos y Auditoría IA — Sesión 12
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Atributos de Símbolos y Memoria  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [No.24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de registro de símbolos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al codificar de forma manual, el mayor reto consistió en aplicar correctamente las reglas matemáticas de alineación natural de memoria (padding) utilizando aritmética de enteros para redondear el cursor al siguiente múltiplo exacto del ancho del tipo de dato, evitando así accesos a direcciones de memoria desalineadas en la pila.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cálculo de desplazamientos y alineación |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Contexto]: Desarrollo de un módulo de análisis semántico y asignación de memoria para un compilador en C99, manejando alineación de estructuras en pila bajo reglas estrictas de compilación (-Wall -Wextra -Werror).

[Rol]: Ingeniero de software experto en sistemas de compiladores, asignación de registros y arquitectura de memoria en lenguaje C.

[Especificación]: 
1. Implementar la función `int ancho_de_tipo(const char *tipo);` que regrese el tamaño en bytes según el tipo de dato ("char": 1, "short": 2, "int": 4, "float": 4, "double": 8, "puntero": 8), o -1 si el tipo es desconocido o NULL.
2. Implementar la función `int asignar_offset(Simbolo *simbolo, int cursor);` que valide que el símbolo no sea NULL y que el cursor no sea negativo (retornando -1 en tal caso). Debe alinear el cursor actual al múltiplo del ancho del tipo del símbolo, guardar dicho offset en el campo correspondiente del registro, y retornar el siguiente cursor disponible (offset + ancho).

[Output]: Únicamente el código modular en C compatible con el contrato "api_sesion12.h", sin función main().
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion12.h"
#include <string.h>

int ancho_de_tipo(const char *tipo) {
    if (!tipo) return -1;
    if (strcmp(tipo, "char") == 0) return 1;
    if (strcmp(tipo, "short") == 0) return 2;
    if (strcmp(tipo, "int") == 0) return 4;
    if (strcmp(tipo, "float") == 0) return 4;
    if (strcmp(tipo, "double") == 0) return 8;
    if (strcmp(tipo, "puntero") == 0) return 8;
    return -1;
}

int asignar_offset(Simbolo *simbolo, int cursor) {
    if (!simbolo || cursor < 0) return -1;

    int ancho = ancho_de_tipo(simbolo->tipo);
    if (ancho == -1) return -1;

    int offset = (cursor + ancho - 1) / ancho * ancho;
    simbolo->offset = offset;

    return offset + ancho;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Alineación de tipos y tamaños en bytes

### Salida del harness
```text
=== AUDITORIA SESION 12 - Registro de simbolo y offsets alineados ===
Caso                                esperado:             obtenido:             resultado
[ 1] ancho_de_tipo("int")           esperado: 4           obtenido: 4           OK
[ 2] ancho_de_tipo("double")        esperado: 8           obtenido: 8           OK
[ 3] int en cursor 0                esperado: 4           obtenido: 4           OK
[ 4] char e int: offset de b        esperado: 4           obtenido: 4           OK
[ 5] double desde cursor 4          esperado: 8           obtenido: 8           OK
[ 6] registro NULL                  esperado: -1          obtenido: -1          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> En versiones preliminares, el modelo no consideraba de forma automática el redondeo al alza necesario para alinear correctamente los punteros o datos cuando el cursor intermedio no coincidía con el múltiplo requerido (por ejemplo, acomodar un tipo de 4 u 8 bytes a partir de un cursor impar). Esto se solucionó aplicando estrictamente la fórmula estándar de alineación de enteros (cursor + ancho - 1) / ancho * ancho.

---
**Firma digital:** [BlancaHoma14]
