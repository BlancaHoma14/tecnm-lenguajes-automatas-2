# Bitácora de Requerimientos y Auditoría IA — Sesión 26
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** TAC para Llamadas a Funciones  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Instrucciones param, call y return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente las funciones emitir_llamada() y emitir_return(), identifiqué que la principal dificultad consistía en generar las instrucciones TAC respetando el orden de los argumentos y la cantidad real de parámetros enviados a cada función.

En la función emitir_llamada() tuve que comprobar si los argumentos a1 y a2 eran diferentes de NULL, porque solamente los argumentos existentes debían generar una instrucción PARAM. También fue necesario llevar un contador para indicar correctamente el número de parámetros en la instrucción CALL.

Otra dificultad fue controlar el espacio disponible en el arreglo lineas[][64], ya que cada instrucción debía caber en un máximo de 63 caracteres más el terminador nulo. Si el arreglo no tenía suficiente capacidad, la generación podía quedar incompleta.

En la función emitir_return() tuve que considerar dos situaciones: generar RETURN acompañado de un valor o generar únicamente RETURN cuando el argumento recibido fuera NULL.

También identifiqué que las llamadas anidadas, como f(g(x)), requieren conservar los temporales generados por las llamadas anteriores. Por eso es importante actualizar correctamente el contador n y no reiniciar las instrucciones que ya existen.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Convención de paso de argumentos TAC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTRATO C-R-E-O

CONTEXTO: Estoy desarrollando la sesión 26 de Lenguajes y Autómatas II.
El objetivo es generar código intermedio TAC para llamadas a
funciones, parámetros y retornos en lenguaje C.

ROL: Actúa como ingeniero de software especializado en compiladores
y generación de código intermedio.

ESPECIFICACIÓN: Implementa las funciones respetando exactamente estas firmas:

int emitir_llamada(char lineas[][64], int max, int *n,
                   const char *funcion, const char *a1,
                   const char *a2, const char *resultado);

void emitir_return(char *dst, int cap, const char *valor);

Requisitos:
1. Emitir PARAM para cada argumento distinto de NULL.
2. Mantener el orden a1 y después a2.
3. Generar CALL con el número real de parámetros.
4. Si resultado es NULL, generar CALL sin asignación.
5. Si valor es NULL, generar RETURN sin valor.
6. Permitir llamadas anidadas sin sobrescribir temporales anteriores.
7. Actualizar n solamente cuando la llamada se genere correctamente.
8. Validar apuntadores, capacidad y truncamiento.
9. Evitar escrituras parciales en el arreglo lineas.
10. No utilizar memoria dinámica ni incluir main().
11. Mantener compatibilidad con C11 y GCC usando
    -Wall -Wextra -Werror.

OUTPUT: Entrega el código completo, modular y compatible con api_sesion26.h.
Explica las mejoras respecto a una implementación manual,
incluye casos límite y analiza complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion26.h"
#include <stdio.h>
#include <string.h>

int emitir_llamada(char lineas[][64], int max, int *n,
                   const char *funcion, const char *a1,
                   const char *a2, const char *resultado) {
    char temp[3][64] = {{0}};
    int cantidad = 0;
    int parametros = 0;
    int r;

    if (lineas == NULL || n == NULL || funcion == NULL ||
        max < 0 || *n < 0 || *n > max) {
        return -1;
    }

    /* Preparar el primer parámetro, si existe. */
    if (a1 != NULL) {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "PARAM %s", a1);
        if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) {
            return -1;
        }
        cantidad++;
        parametros++;
    }

    /* Preparar el segundo parámetro, si existe. */
    if (a2 != NULL) {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "PARAM %s", a2);
        if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) {
            return -1;
        }
        cantidad++;
        parametros++;
    }

    /* Preparar la instrucción CALL. */
    if (resultado != NULL) {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "%s = CALL %s, %d",
                     resultado, funcion, parametros);
    } else {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "CALL %s, %d", funcion, parametros);
    }

    if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) {
        return -1;
    }
    cantidad++;

    /* Comprobar espacio antes de modificar el arreglo final. */
    if (max - *n < cantidad) {
        return -1;
    }

    /* Copiar únicamente cuando todas las instrucciones son válidas. */
    memcpy(&lineas[*n], temp,
           (size_t)cantidad * sizeof(temp[0]));

    *n += cantidad;

    return parametros;
}

void emitir_return(char *dst, int cap, const char *valor) {
    int r;

    if (dst == NULL || cap <= 0) {
        return;
    }

    if (valor != NULL) {
        r = snprintf(dst, (size_t)cap, "RETURN %s", valor);
    } else {
        r = snprintf(dst, (size_t)cap, "RETURN");
    }

    /* Evitar dejar una instrucción incompleta por truncamiento. */
    if (r < 0 || r >= cap) {
        dst[0] = '\0';
    }
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Llamadas anidadas f(g(x))

### Salida del harness
```text
=== AUDITORIA SESION 26 - Llamadas, parametros y retorno ===
Caso                               esperado:              obtenido:              resultado
[ 1] PARAM x                            esperado: PARAM x      obtenido: PARAM x      OK
[ 2] PARAM y                            esperado: PARAM y      obtenido: PARAM y      OK
[ 3] CALL con dos argumentos
     esperado: _t0 = CALL f, 2
     obtenido: _t0 = CALL f, 2
     -> OK
[ 4] llamada anidada f(g(x))
     esperado: _t1 = CALL f, 1
     obtenido: _t1 = CALL f, 1
     -> OK
[ 5] CALL sin argumentos
     esperado: _t0 = CALL p, 0
     obtenido: _t0 = CALL p, 0
     -> OK
[ 6] RETURN con valor                   esperado: RETURN _t0   obtenido: RETURN _t0   OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion26.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la auditoría identifiqué que el harness principal solamente comprueba seis casos y no verifica todas las situaciones posibles de las funciones.

Uno de los casos que requiere atención es cuando el arreglo lineas no tiene espacio suficiente para almacenar todos los parámetros y la instrucción CALL. También es importante comprobar qué sucede cuando una cadena es demasiado larga para caber en los 64 caracteres disponibles.

La versión manual podía escribir algunos parámetros antes de descubrir que no existía espacio para la instrucción CALL. Para solucionar esta situación, la versión mejorada utiliza un arreglo temporal y realiza todas las validaciones antes de modificar el arreglo definitivo.

También se mejoró el manejo del retorno cuando el destino es demasiado pequeño, limpiando la cadena para evitar conservar una instrucción incompleta.

---
**Firma digital:** [BlancaHoma14]
