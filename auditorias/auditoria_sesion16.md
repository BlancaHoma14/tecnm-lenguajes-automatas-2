# Bitácora de Requerimientos y Auditoría IA — Sesión 16
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Firmas de Función y Parámetros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Registro de prototipos y aridad  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el código, identifiqué que uno de los principales problemas era recorrer correctamente el catálogo de funciones para encontrar el nombre solicitado y comprobar que la cantidad de argumentos coincidiera con su aridad registrada.

También tuve que considerar diferentes situaciones: cuando una función no está registrada, cuando se llama con una cantidad incorrecta de argumentos y cuando se trata de una función sin parámetros. Otra dificultad fue utilizar correctamente strcmp() para comparar los nombres de las funciones, ya que en C no se deben comparar cadenas utilizando directamente el operador ==.

Además, fue necesario validar los punteros para evitar errores cuando el catálogo o el nombre de la función fueran NULL.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Coincidencia estricta de parámetros |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTEXTO: Estoy trabajando en la Sesión 16 de la materia Lenguajes y Autómatas II.
El objetivo es implementar un registro de prototipos y verificar la
cantidad de argumentos de las llamadas a funciones utilizando C99
y GCC.

ROL: Actúa como un ingeniero de software especializado en programación en C
y desarrollo de compiladores.

ESPECIFICACIÓN: Implementa la siguiente estructura:

typedef struct Funcion {
    const char *nombre;
    int aridad;
} Funcion;

Implementa la función:

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs);

La función debe cumplir con estas condiciones:

1. Buscar el nombre de la función mediante coincidencia exacta
   utilizando strcmp().
2. Recorrer el catálogo completo hasta encontrar la función.
3. Si el nombre existe y la cantidad de argumentos coincide
   con su aridad, devolver 1.
4. Si el nombre existe, pero la cantidad de argumentos es diferente,
   devolver 0.
5. Si la función no está registrada, devolver -1.
6. Si catalogo o nombre son NULL, devolver -1.
7. Si nargs o total son negativos, devolver -1.
8. Si una entrada del catálogo tiene nombre NULL, omitirla
   sin provocar un error.
9. Permitir correctamente funciones sin parámetros (aridad 0).
10. No modificar la estructura ni utilizar una función main().
11. El código debe compilar con:
    gcc -std=c11 -Wall -Wextra -Werror

OUTPUT: Entrega únicamente la implementación modular en C y explica
brevemente su complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion16.h"
#include <string.h>

int verificar_llamada(const Funcion *catalogo, int total,
                      const char *nombre, int nargs) {
    int i;

    /* Validar los datos de entrada */
    if (catalogo == NULL || nombre == NULL ||
        total < 0 || nargs < 0) {
        return -1;
    }

    /* Buscar la función dentro del catálogo */
    for (i = 0; i < total; i++) {

        /* Evitar comparar un nombre NULL */
        if (catalogo[i].nombre == NULL) {
            continue;
        }

        /* Comparar el nombre de manera exacta */
        if (strcmp(catalogo[i].nombre, nombre) == 0) {

            /* Verificar que coincida la cantidad de argumentos */
            if (catalogo[i].aridad == nargs) {
                return 1;
            }

            /* La función existe, pero tiene diferente aridad */
            return 0;
        }
    }

    /* La función no está registrada */
    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas con número incorrecto de args

### Salida del harness
```text
=== AUDITORIA SESION 16 - Registro de prototipos y aridad ===
Caso                               esperado:              obtenido:              resultado
[ 1] suma con 3 argumentos              esperado: 1            obtenido: 1            OK
[ 2] suma con 2 argumentos              esperado: 0            obtenido: 0            OK
[ 3] suma con 4 argumentos              esperado: 0            obtenido: 0            OK
[ 4] funcion no registrada              esperado: -1           obtenido: -1           OK
[ 5] main sin argumentos                esperado: 1            obtenido: 1            OK
[ 6] nombre NULL                        esperado: -1           obtenido: -1           OK

RESULTADO: 6/6 pruebas superadas
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar la primera propuesta de código generada por la IA, identifiqué que no contemplaba el caso en el que alguno de los registros del catálogo tuviera el campo nombre con valor NULL. Esto podía provocar un comportamiento incorrecto al utilizar strcmp(), ya que esta función necesita recibir cadenas válidas.

Para solucionar el problema, agregué la siguiente condición dentro del ciclo:
if (catalogo[i].nombre == NULL) {
    continue;
}

Con esta validación, el programa omite los registros que no tienen un nombre válido y continúa buscando en el catálogo sin provocar un error por esa comparación.

También agregué la validación total < 0 para rechazar cantidades negativas de registros. De esta manera, el código queda mejor protegido ante entradas inválidas y cumple con las condiciones solicitadas.

---
**Firma digital:** [Blanca Janet Homa May]
