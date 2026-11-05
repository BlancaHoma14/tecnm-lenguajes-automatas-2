# Bitácora de Requerimientos y Auditoría IA — Sesión 09
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Reporte Formal de Errores Semánticos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Diagnóstico con línea y columna  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente la función reportar(), identifiqué que la principal dificultad era construir una cadena con el nombre del archivo, el número de línea, la columna y el mensaje de error, respetando exactamente el formato solicitado.

También tuve que considerar la capacidad limitada del buffer, ya que no debía escribir más caracteres de los que podía almacenar. Para solucionar este problema utilicé la función snprintf(), porque permite limitar la cantidad de caracteres escritos y evitar desbordamientos de memoria.

Otra dificultad fue contemplar los casos en los que los parámetros archivo o mensaje pudieran ser NULL. Por ello agregué validaciones para sustituirlos por cadenas vacías y evitar problemas durante la construcción del diagnóstico.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Estructura estándar de errores GCC |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
Contrato C-R-E-O

CONTEXTO: Estoy trabajando en la Sesión 09 de Lenguajes y Autómatas II. Necesito implementar una función en C que genere diagnósticos de errores semánticos con nombre de archivo, línea, columna y mensaje.

ROL: Actúa como un programador experto en lenguaje C y desarrollo de compiladores. Escribe código claro, seguro y modular.

ESPECIFICACIÓN: Implementa la función:

void reportar(const char *archivo, int linea, int columna, const char *mensaje, char *salida, int cap);

Debe cumplir lo siguiente:

1. Construir el texto con el formato: archivo:linea:columna: error: mensaje
2. Respetar exactamente el formato, sin espacios adicionales ni punto final.
3. Utilizar snprintf para evitar desbordamientos.
4. No escribir más de cap-1 caracteres.
5. Garantizar que la cadena termine en '\0' cuando el buffer sea válido y cap sea mayor que cero.
6. Comprobar que salida no sea NULL y que cap sea válido.
7.  Manejar archivo y mensaje cuando sean NULL.
8. Conservar exactamente la firma solicitada.
9. No incluir main(), porque el harness ya lo proporciona.
10. El código debe compilar con: gcc -std=c11 -Wall -Wextra -Werror
OUTPUT: Entrega el código completo para ia_sesion09_c.c. Explica las mejoras respecto a una implementación manual, los casos límite considerados y la complejidad temporal y espacial. No modifiques el contrato ni el harness.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion09_c.c

#include "api_sesion09.h"
#include 
#include 

void reportar(const char *archivo, int linea, int columna,
              const char *mensaje, char *salida, int cap) {

    if (salida == NULL || cap <= 0) {
        return;
    }

    salida[0] = '\0';

    if (archivo == NULL) {
        archivo = "";
    }

    if (mensaje == NULL) {
        mensaje = "";
    }

    (void)snprintf(salida, (size_t)cap,
                   "%s:%d:%d: error: %s",
                   archivo, linea, columna, mensaje);

    salida[cap - 1] = '\0';
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Múltiples errores sin abortar parser

### Salida del harness
```text
=== AUDITORIA SESION 09 - Diagnostico con linea y columna ===
Caso                               esperado:              obtenido:              resultado
[ 1] main.c:14:5
     esperado: main.c:14:5: error: tipo incompatible
     obtenido: main.c:14:5: error: tipo incompatible
     -> OK
[ 2] a.c:1:1
     esperado: a.c:1:1: error: x
     obtenido: a.c:1:1: error: x
     -> OK
[ 3] linea 0
     esperado: x.c:0:0: error: y
     obtenido: x.c:0:0: error: y
     -> OK
[ 4] cap=10 trunca                      esperado: main.c:14    obtenido: main.c:14    OK

RESULTADO: 4/4 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion09.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar la propuesta de la IA, identifiqué que el harness solamente contempla cuatro pruebas y no verifica todos los posibles casos límite de la función. Por ejemplo, no comprueba qué sucede cuando la capacidad del buffer es uno, cuando es cero o cuando alguno de los parámetros de texto es NULL.

Para mejorar la solución, agregué validaciones que comprueban que el buffer exista y que su capacidad sea mayor que cero. También inicialicé la cadena de salida antes de utilizar snprintf() y forcé el carácter terminador '\0' en la última posición disponible.

Con estas modificaciones, la función puede manejar de manera más segura las entradas límite. Sin embargo, considero importante señalar que estos casos adicionales no forman parte de las cuatro pruebas originales del harness, por lo que sería conveniente probarlos por separado.

---
**Firma digital:** [Blanca Janet Homa May]
