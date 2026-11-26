# Bitácora de Requerimientos y Auditoría IA — Sesión 29
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Optimización: Dead Code Elimination  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Detección de código inalcanzable  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente la función eliminar_muerto(), identifiqué que la principal dificultad era reconocer cuándo una instrucción ya no podía ejecutarse y determinar en qué momento debía volver a conservar las instrucciones.

Para resolverlo utilicé una variable de control llamada muerto, que me permitió identificar si el programa había encontrado una instrucción GOTO o RETURN.

También identifiqué las siguientes restricciones:
- El arreglo lineas tiene un espacio fijo de 64 caracteres por instrucción.
- La función debe modificar el mismo arreglo, sin utilizar memoria dinámica.
- Las instrucciones GOTO y RETURN deben conservarse.
- Las instrucciones posteriores a un salto o retorno deben eliminarse hasta encontrar una etiqueta.
- Las etiquetas terminadas en dos puntos deben conservarse porque representan puntos alcanzables.
- El orden original de las instrucciones que sobreviven no debe cambiar.
- La función debe devolver la nueva cantidad de instrucciones.
- Una lista vacía debe devolver 0 sin intentar leer posiciones inexistentes.
La principal dificultad fue compactar las instrucciones dentro del mismo arreglo sin sobrescribir información que todavía necesitaba revisar. También tuve que identificar correctamente las etiquetas y distinguir las instrucciones de salto de otros nombres de variables.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Análisis de flujo tras retornos y gotos |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
**CONTRATO C-R-E-O**

**CONTEXTO:** Estoy desarrollando la Sesión 29 de Lenguajes y Autómatas II (SCD-1016). El tema es la eliminación de código muerto (Dead Code Elimination) en código intermedio TAC. Trabajo con C11 y GCC.

**ROL:** Actúa como ingeniero de software especializado en compiladores, optimización de código y programación en C.

**ESPECIFICACIÓN:** Implementa exactamente la siguiente función:
int eliminar_muerto(char lineas[][64], int n);
Requisitos:
1. Detectar instrucciones GOTO y RETURN.
2. Conservar la instrucción GOTO o RETURN.
3. Eliminar todas las instrucciones posteriores hasta encontrar una etiqueta.
4. Una etiqueta se reconoce porque termina en dos puntos.
5. Conservar las etiquetas porque siempre son alcanzables.
6. Después de una etiqueta, volver a conservar las instrucciones.
7. Compactar el arreglo original sin utilizar memoria dinámica.
8. Mantener el orden original de las instrucciones que sobreviven.
9. Devolver la nueva cantidad de instrucciones.
10. Si la lista está vacía, devolver 0.
11. Validar el apuntador del arreglo y el valor de n.
12. Reconocer correctamente las palabras GOTO y RETURN, sin confundirlas con identificadores que solamente comienzan con esas letras.
13. Considerar espacios y tabulaciones al identificar instrucciones.

**OUTPUT:** Entrega código modular en C11, sin función main(), respetando exactamente la firma original. Explica las mejoras respecto al código manual y analiza la complejidad temporal y espacial.

Debe compilar con:

gcc -std=c11 -Wall -Wextra -Werror
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion29_c.c

#include "api_sesion29.h"
#include <string.h>
#include <ctype.h>

static int es_etiqueta_ia(const char *linea) {
    size_t longitud;

    if (linea == NULL) {
        return 0;
    }

    longitud = strlen(linea);

    while (longitud > 0 &&
           isspace((unsigned char)linea[longitud - 1])) {
        longitud--;
    }

    return longitud > 0 && linea[longitud - 1] == ':';
}

static int palabra_completa(const char *linea,
                            const char *palabra) {
    size_t longitud;

    while (*linea != '\0' &&
           isspace((unsigned char)*linea)) {
        linea++;
    }

    longitud = strlen(palabra);

    if (strncmp(linea, palabra, longitud) != 0) {
        return 0;
    }

    return linea[longitud] == '\0' ||
           isspace((unsigned char)linea[longitud]);
}

static int es_terminador_ia(const char *linea) {
    return palabra_completa(linea, "GOTO") ||
           palabra_completa(linea, "RETURN");
}

int eliminar_muerto(char lineas[][64], int n) {
    int i;
    int escribir = 0;
    int codigo_muerto = 0;

    if (lineas == NULL || n <= 0) {
        return 0;
    }

    for (i = 0; i < n; i++) {

        if (codigo_muerto) {
            if (es_etiqueta_ia(lineas[i])) {
                codigo_muerto = 0;
            } else {
                continue;
            }
        }

        if (escribir != i) {
            memcpy(lineas[escribir], lineas[i], 64);
        }

        escribir++;

        if (es_terminador_ia(lineas[i])) {
            codigo_muerto = 1;
        }
    }

    return escribir;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Eliminación de bloques inalcanzables

### Salida del harness
```text
=== AUDITORIA SESION 29 - Eliminacion de codigo inalcanzable ===
Caso                               esperado:              obtenido:              resultado
[ 1] tras GOTO: nuevo tamano            esperado: 4            obtenido: 4            OK
[ 2] tras GOTO: etiqueta viva           esperado: L1:          obtenido: L1:          OK
[ 3] tras RETURN: nuevo tamano          esperado: 4            obtenido: 4            OK
[ 4] tras RETURN: etiqueta viva         esperado: L5:          obtenido: L5:          OK
[ 5] sin codigo muerto                  esperado: 3            obtenido: 3            OK
[ 6] lista vacia                        esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion29.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión del código identifiqué que el harness original solamente comprueba los casos principales y no contempla directamente situaciones como instrucciones con espacios iniciales, etiquetas con espacios al final o identificadores que comienzan con las mismas letras que las instrucciones de salto.

Por ejemplo, una cadena como RETURN_VALUE = 5 no debe interpretarse como una instrucción RETURN, porque es una asignación y no un retorno. De igual manera, GOTO_EXTRA = 3 no debe provocar que se eliminen las instrucciones siguientes.

Para atender estos casos, se mejoró la detección de instrucciones utilizando una función que comprueba que GOTO y RETURN sean palabras completas. También se incorporó el uso de isspace() para reconocer espacios y tabulaciones.

---
**Firma digital:** [BlancaHoma14]
