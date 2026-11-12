# Bitácora de Requerimientos y Auditoría IA — Sesión 11
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Arquitectura de la Tabla de Símbolos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [No.24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Tabla Hash con encadenamiento  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al codificar a mano, el principal desafío fue estructurar correctamente el encadenamiento mediante listas enlazadas para evitar perder nodos ante una colisión de índices en la tabla hash. Asimismo, fue necesario recordar el manejo seguro de cadenas con `strncpy` para asegurar que el campo `nombre` no exceda los 16 bytes permitidos por la estructura `Simbolo`, garantizando siempre el carácter nulo al final de la cadena (`\0`).

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Función de dispersión y colisiones O(1) |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[Contexto]: Desarrollando una tabla hash con encadenamiento en C99 para la gestión de símbolos de un compilador, ejecutando compilación estricta con GCC (-Wall -Wextra -Werror).

[Rol]: Ingeniero de software experto en estructuras de datos, sistemas de compiladores y optimización de memoria en lenguaje C.

[Especificación]: 
1. Implementar la función de dispersión `unsigned int hash_djb2(const char *clave);` utilizando el algoritmo clásico hash djb2 (valor inicial de 5381, multiplicación por 33 sumando cada byte).
2. Implementar la función `int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset);` que calcule el índice hash, verifique duplicados para evitar reescribir nodos (retornando 0 si ya existe), maneje colisiones mediante listas enlazadas (insertando al inicio de la cubeta) y valide argumentos de entrada nulos o cubetas en cero (retornando -1).

[Output]: Únicamente el código modular en C listo para compilar con la cabecera "api_sesion11.h", sin incluir la función main().
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion11.h"
#include <stdlib.h>
#include <string.h>

unsigned int hash_djb2(const char *clave) {
    unsigned int hash = 5381;
    int c;
    if (!clave) return hash;
    while ((c = *clave++)) {
        hash = ((hash << 5) + hash) + c;
    }
    return hash;
}

int insertar_simbolo(Simbolo *tabla[], unsigned int cubetas, const char *nombre, int tipo, int offset) {
    if (!tabla || !nombre || cubetas == 0) return -1;

    unsigned int idx = hash_djb2(nombre) % cubetas;
    Simbolo *curr = tabla[idx];

    // Comprobar duplicados
    while (curr) {
        if (strcmp(curr->nombre, nombre) == 0) {
            return 0;
        }
        curr = curr->sig;
    }

    // Reservar memoria e insertar
    Simbolo *nuevo = (Simbolo *)malloc(sizeof(Simbolo));
    if (!nuevo) return -1;

    strncpy(nuevo->nombre, nombre, sizeof(nuevo->nombre) - 1);
    nuevo->nombre[sizeof(nuevo->nombre) - 1] = '\0';
    nuevo->tipo = tipo;
    nuevo->offset = offset;
    nuevo->sig = tabla[idx];
    tabla[idx] = nuevo;

    return 1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Manejo de claves duplicadas y colisiones

### Salida del harness
```text
=== AUDITORIA SESION 11 - Tabla hash con encadenamiento ===
Caso                                esperado:             obtenido:             resultado
[ 1] hash_djb2("")                  esperado: 5381        obtenido: 5381        OK
[ 2] hash_djb2("abc")               esperado: 193485963   obtenido: 193485963   OK
[ 3] insertar x dos veces           esperado: 0           obtenido: 0           OK
[ 4] dos claves, una cubeta         esperado: 2           obtenido: 2           OK
[ 5] offset de beta en cadena       esperado: 4           obtenido: 4           OK
[ 6] insertar en tabla NULL         esperado: -1          obtenido: -1          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE.
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La versión inicial generada por el modelo a veces suele omitir la validación de seguridad de la cadena al copiarla (pudiendo causar desbordamientos si el nombre es mayor de 16 caracteres) o no maneja de forma estricta los punteros nulos en el parámetro cubetas o tabla. Para solucionarlo, se reforzó el prompt especificando el uso de acotación de longitud con strncpy asegurando el terminador nulo \0, así como la validación inicial de argumentos nulos exigida por el contrato (tabla == NULL || nombre == NULL || cubetas == 0).

---
**Firma digital:** [BlancaHoma14]
