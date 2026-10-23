# Bitácora de Requerimientos y Auditoría IA — Sesión 03
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Google Colab, Mágicos y Flujo de Entrega  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Manipulación de archivos y llamadas OS  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al implementar las funciones en C, identifiqué que debía respetar exactamente las firmas declaradas en api_sesion03.h para que el harness pudiera compilar. Una dificultad fue manejar correctamente los archivos mediante fopen(), comprobar si se abrían y cerrarlos con fclose() incluso cuando ocurría un error.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Salida estándar y códigos de retorno ABI |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado

**Contexto:** Estoy realizando la Sesión 03 de Lenguajes y Autómatas II. Necesito implementar en C tres funciones para manipular archivos y ejecutar comandos externos. El proyecto utiliza `api_sesion03.h` y un harness de pruebas que compila con GCC y las opciones `-std=c11 -Wall -Wextra -Werror`.

**Rol:** Actúa como un ingeniero de software especializado en C, sistemas operativos y desarrollo de compiladores.

**Especificación:**
1. Implementa `int escribir_texto(const char *ruta, const char *texto)`. Debe escribir el texto en un archivo, devolver `0` si tiene éxito y `-1` si ocurre un error.
2. Implementa `int contar_lineas(const char *ruta)`. Debe contar las líneas, incluyendo la última cuando no termina con `\n`. Si el archivo no puede abrirse o se produce un error de lectura, debe devolver `-1`.
3. Implementa `int ejecutar(const char *comando)`. Debe utilizar `system()`, comprobar si el proceso terminó normalmente mediante `WIFEXITED()` y extraer el código de salida con `WEXITSTATUS()`. Debe devolver `-1` cuando no pueda obtenerse un resultado válido.
4. Respeta exactamente las firmas de `api_sesion03.h`.
5. No incluyas la función `main()`, porque el harness ya la proporciona.
6. Cierra los archivos en todos los caminos de ejecución.
7. Asegura que el código compile sin advertencias con las opciones indicadas.
8. Considera archivos vacíos, rutas inválidas, archivos inexistentes y archivos sin salto de línea final.

**Output:** Devuelve el código C completo para `ia_sesion03_c.c`, explica brevemente la función de cada implementación y señala los casos borde considerados. No inventes resultados de pruebas que no hayas ejecutado.
```

---

## 3. Código Devuelto por el Modelo

#include "api_sesion03.h"
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>

int escribir_texto(const char *ruta, const char *texto) {
    if (ruta == NULL || texto == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "w");
    if (archivo == NULL) {
        return -1;
    }

    int resultado = 0;

    if (fputs(texto, archivo) == EOF) {
        resultado = -1;
    }

    if (fclose(archivo) == EOF) {
        resultado = -1;
    }

    return resultado;
}

int contar_lineas(const char *ruta) {
    if (ruta == NULL) {
        return -1;
    }

    FILE *archivo = fopen(ruta, "r");
    if (archivo == NULL) {
        return -1;
    }

    int caracter;
    int lineas = 0;
    int ultimo = '\n';
    int hubo_caracter = 0;

    while ((caracter = fgetc(archivo)) != EOF) {
        hubo_caracter = 1;
        ultimo = caracter;

        if (caracter == '\n') {
            lineas++;
        }
    }

    if (ferror(archivo)) {
        fclose(archivo);
        return -1;
    }

    if (fclose(archivo) == EOF) {
        return -1;
    }

    if (hubo_caracter && ultimo != '\n') {
        lineas++;
    }

    return lineas;
}

int ejecutar(const char *comando) {
    if (comando == NULL) {
        return -1;
    }

    int estado = system(comando);

    if (estado == -1) {
        return -1;
    }

    if (WIFEXITED(estado)) {
        return WEXITSTATUS(estado);
    }

    return -1;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Compilación y ejecución del harness con GCC (-std=c11 -Wall -Wextra -Werror) para verificar escritura de archivos, conteo de líneas y códigos de salida.

### Salida del harness
```text
=== AUDITORIA SESION 03 - Archivos y codigos de salida ===
Caso                               esperado:              obtenido:              resultado
[ 1] escribir_texto(...)==0        esperado: 0             obtenido: 0             OK
[ 2] contar_lineas(3 lineas)       esperado: 3             obtenido: 3             OK
[ 3] contar_lineas(sin salto final) esperado: 2             obtenido: 2             OK
[ 4] contar_lineas(inexistente)    esperado: -1            obtenido: -1            OK
[ 5] ejecutar("true")              esperado: 0             obtenido: 0             OK
[ 6] ejecutar("false")             esperado: 1             obtenido: 1             OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion03.md
```

* [✔ ] Caso base y límites superados sin *Crash*.
* [✔ ] No existen accesos fuera de límites ni *Memory Leaks*.
* [✔ ] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión de la implementación de IA, GCC reportó el error unterminated comment, que indica que existía un comentario de bloque sin su cierre correspondiente. El problema impedía compilar el archivo antes de ejecutar las pruebas. Lo solucioné reemplazando el contenido incompleto por una implementación completa y verificando que la celda %%writefile ia_sesion03_c.c se ejecutara antes de la compilación.

También revisé el caso borde de un archivo cuya última línea no termina en \n. Para resolverlo, la función contar_lineas() registra si leyó caracteres y comprueba el último carácter del archivo, agregando una línea cuando corresponde.

---
**Firma digital:** [BlancaHoma14]
