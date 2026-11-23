# Bitácora de Requerimientos y Auditoría IA — Sesión 19
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Estructuras (struct) y Miembros  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Offsets de miembros en estructuras  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la implementación manual, la principal dificultad fue comprender cómo se calculan los offsets considerando la alineación y el padding de cada tipo de dato. También fue necesario identificar cuándo agregar bytes de relleno para que cada campo comenzara en una dirección de memoria válida según su tamaño. Además, tuve que asegurarme de calcular correctamente el padding final para obtener el tamaño total de la estructura.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Alineación de memoria por palabras |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[C-Contexto: Estoy desarrollando una práctica de Lenguajes y Autómatas II en C. Debo implementar las funciones offset_campo() y tamano_struct() para calcular offsets de miembros y tamaño total de una estructura aplicando alineación y padding.

R-Rol: Actúa como un desarrollador experto en lenguaje C y arquitectura de computadoras, respetando estrictamente el contrato definido en api_sesion19.h.

E-Especificación: Implementar las funciones offset_campo() y tamano_struct() respetando las firmas definidas en api_sesion19.h. La solución debe soportar los tipos de datos char (1 byte), short (2 bytes), int (4 bytes) y double (8 bytes). Cada campo debe alinearse según su propio tamaño antes de asignar su posición en memoria. El tamaño total de la estructura debe incluir el padding necesario y redondearse al múltiplo de la alineación más grande utilizada. Si se encuentra un tipo desconocido, la función debe devolver -1. Si el miembro solicitado no existe, offset_campo() debe devolver -1. Si la estructura no tiene campos (total = 0), tamano_struct() debe devolver 0. La implementación no debe incluir una función main() y debe ser compatible con la compilación usando gcc -std=c11 -Wall -Wextra -Werror.

O-Output: Generar el código completo del archivo ia_sesion19_c.c, listo para compilar, sin modificar las firmas del contrato y sin incluir una función main().]
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion19_c.c
/* ia_sesion19_c.c - Fase 2: El Cientifico
 *
 * Pega aqui el codigo que te devolvio la IA, con la MISMA firma
 * que declara api_sesion19.h. No puede traer main().
 */
#include "api_sesion19.h"
#include <string.h>

static int obtener_alineacion(const char *tipo)
{
    if (tipo == NULL)
        return -1;

    if (strcmp(tipo, "char") == 0)   return 1;
    if (strcmp(tipo, "short") == 0)  return 2;
    if (strcmp(tipo, "int") == 0)    return 4;
    if (strcmp(tipo, "double") == 0) return 8;

    return -1;
}

static int alinear(int valor, int alineacion)
{
    int resto = valor % alineacion;

    if (resto == 0)
        return valor;

    return valor + (alineacion - resto);
}

int offset_campo(const Campo *campos, int total, const char *nombre)
{
    int offset = 0;

    if (campos == NULL || nombre == NULL || total < 0)
        return -1;

    for (int i = 0; i < total; i++)
    {
        int alineacion = obtener_alineacion(campos[i].tipo);

        if (alineacion == -1)
            return -1;

        offset = alinear(offset, alineacion);

        if (strcmp(campos[i].nombre, nombre) == 0)
            return offset;

        offset += alineacion;
    }

    return -1;
}

int tamano_struct(const Campo *campos, int total)
{
    int offset = 0;
    int max_alineacion = 1;

    if (campos == NULL)
        return -1;

    if (total == 0)
        return 0;

    for (int i = 0; i < total; i++)
    {
        int alineacion = obtener_alineacion(campos[i].tipo);

        if (alineacion == -1)
            return -1;

        offset = alinear(offset, alineacion);

        if (alineacion > max_alineacion)
            max_alineacion = alineacion;

        offset += alineacion;
    }

    return alinear(offset, max_alineacion);
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Acceso a miembros inexistentes

                  === AUDITORIA SESION 19 - Offsets de miembros y tamano de struct ===
            Caso                      esperado:              obtenido:         resultado
      [ 1] struct char e int: b       esperado: 4            obtenido: 4            OK
      [ 2] struct de tres campos: c   esperado: 8            obtenido: 8            OK
      [ 3] tamano total con padding   esperado: 12           obtenido: 12           OK
      [ 4] double seguido de char     esperado: 16           obtenido: 16           OK
      [ 5] miembro inexistente        esperado: -1           obtenido: -1           OK
      [ 6] struct sin campos          esperado: 0            obtenido: 0            OK


### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion19.md  
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> La IA resolvió correctamente los casos del harness, pero inicialmente podía omitir la validación de tipos desconocidos. Para verificar este caso agregué pruebas con tipos inexistentes, observando que el programa debía devolver -1 en lugar de continuar calculando offsets. Se corrigió agregando una validación explícita en la función que obtiene la alineación de cada tipo.

---
**Firma digital:** [BlancaHoma14]
