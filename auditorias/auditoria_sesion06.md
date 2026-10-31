# Bitácora de Requerimientos y Auditoría IA — Sesión 06
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Sistema de Tipos Primitivos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [No.24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Chequeo de compatibilidad de tipos  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al implementar la lógica de compatibilidad de forma manual, el principal desafío fue estructurar correctamente la jerarquía de conversión implícita (`CHAR < INT < FLOAT`) sin caer en comportamientos indefinidos al comparar tipos de menor a mayor jerarquía, además de asegurar que el tipo de error (`TIPO_ERROR`) se maneje como un caso exclusivo que nunca deba permitir asignaciones automáticas. Asimismo, al retornar cadenas de caracteres estáticas en `nombre_tipo`, se debe asegurar que se cubran todos los casos del enumerador para evitar advertencias del compilador bajo las banderas estrictas `-Wall -Wextra -Werror`.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Matriz de compatibilidad estática |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

```text
[Contexto]: Estás programando en C99 bajo estándares estrictos de compilación (-Wall -Wextra -Werror) para un módulo de análisis semántico en un compilador.
[Rol]: Actúa como un Ingeniero de sistemas y compiladores experto en verificación de tipos.
[Especificación]: Implementa las funciones:
  1. `int compatible(Tipo destino, Tipo origen)`: evalúa la jerarquía `TIPO_CHAR < TIPO_INT < TIPO_FLOAT`. Devuelve 1 si el destino es de igual o mayor rango que el origen, y 0 en caso contrario. `TIPO_ERROR` nunca es compatible con nada.
  2. `const char *nombre_tipo(Tipo t)`: devuelve la representación en cadena de texto legible ("char", "int", "float", "error") correspondiente a cada valor del enum `Tipo`.
[Output]: Proporciona exclusivamente el código en C limpio, modular, sin incluir la función `main()`, que cumpla exactamente con el contrato definido en `api_sesion06.h`.
```

---

## 3. Código Devuelto por el Modelo
```c
/* ia_sesion06_c.c - Fase 2: El Cientifico */
#include "api_sesion06.h"

int compatible(Tipo destino, Tipo origen) {
    if (origen == TIPO_ERROR || destino == TIPO_ERROR) {
        return 0;
    }
    return ((int)destino >= (int)origen);
}

const char *nombre_tipo(Tipo t) {
    static const char *tabla_nombres[] = {
        "error",   // TIPO_ERROR (0)
        "char",    // TIPO_CHAR (1)
        "int",     // TIPO_INT (2)
        "float"    // TIPO_FLOAT (3)
    };

    if (t < TIPO_ERROR || t > TIPO_FLOAT) {
        return "desconocido";
    }
    
    return tabla_nombres[t];
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Rechazo de asignaciones incompatibles

### Salida del harness
```text
=== AUDITORIA SESION 06 - Tabla de compatibilidad de tipos ===
Caso                                 esperado:             obtenido:             resultado
[ 1] compatible(INT,INT)             esperado: 1           obtenido: 1           OK
[ 2] compatible(FLOAT,INT)           esperado: 1           obtenido: 1           OK
[ 3] compatible(INT,FLOAT)           esperado: 0           obtenido: 0           OK
[ 4] compatible(INT,ERROR)           esperado: 0           obtenido: 0           OK
[ 5] nombre_tipo(INT)                esperado: int         obtenido: int         OK
[ 6] nombre_tipo(ERROR)              esperado: error       obtenido: error       OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*

> La primera versión generada por la IA no validaba explícitamente si el parámetro origen o destino recibía valores fuera del rango esperado o si alguno correspondía a TIPO_ERROR antes de realizar la comparación numérica de jerarquía, lo que podría haber provocado falsos positivos si los enums se evaluaban de forma puramente aritmética. Esto se solucionó añadiendo una guarda inicial explícita (if (origen == TIPO_ERROR || destino == TIPO_ERROR) return 0;) y validando los límites del arreglo estático en nombre_tipo para garantizar robustez ante entradas inválidas.

---
**Firma digital:** [BlancaHoma14]
