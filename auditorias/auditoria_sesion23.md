# Bitácora de Requerimientos y Auditoría IA — Sesión 23
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Etiquetas y Saltos Incondicionales  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Generador de etiquetas e instrucciones goto  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar manualmente el generador de etiquetas identifiqué que debía controlar correctamente el contador para generar etiquetas únicas y consecutivas, comenzando desde L0. También tuve que considerar el tamaño del arreglo de caracteres para evitar escribir fuera de los límites de memoria.

> Otra dificultad fue construir correctamente las instrucciones de flujo de control. Cuando se trata de un salto, la salida debe tener el formato GOTO Lx, mientras que cuando se declara una etiqueta debe terminar con dos puntos, por ejemplo L2:.

> Además, identifiqué que era necesario validar los apuntadores nulos y el tamaño del búfer, ya que trabajar con cadenas en C requiere tener cuidado con la memoria disponible y el carácter nulo '\0' que indica el final de una cadena.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Trazabilidad del flujo de control |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTRATO C-R-E-O

Contexto (C): Estoy trabajando en la Sesión 23 de Lenguajes y Autómatas II. Necesito implementar dos funciones en C11 utilizando GCC y bibliotecas estándar.

Rol (R): Asume el papel de un ingeniero de software experto en generación de código intermedio y control de flujo.

Especificación (E): Implementa las funciones nueva_etiqueta() y emitir_flujo() respetando exactamente las siguientes firmas:

void nueva_etiqueta(char *dst, int cap, int *contador);
void emitir_flujo(char *dst, int cap, const char *etiqueta, int es_salto);
La función nueva_etiqueta() debe generar etiquetas consecutivas L0, L1, L2, etc., utilizando el contador recibido e incrementándolo después de generar correctamente la etiqueta.

La función emitir_flujo() debe generar GOTO Lx cuando es_salto sea diferente de cero y Lx: cuando sea cero.

Si la etiqueta es NULL, debe producir una cadena vacía. También debe validar los punteros y la capacidad del búfer para evitar accesos inválidos y desbordamientos.

No modifiques las firmas, no incluyas main() y utiliza snprintf() para controlar la escritura en memoria.

Output (O): Entrega el código completo y modular para ia_sesion23_c.c, una explicación de su funcionamiento, las mejoras realizadas respecto a una implementación manual y el análisis de complejidad temporal y espacial.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion23.h"
#include <stdio.h>
#include <limits.h>

void nueva_etiqueta(char *dst, int cap, int *contador) {
    int resultado;

    if (dst == NULL || cap <= 0 || contador == NULL)
        return;

    dst[0] = '\0';

    if (*contador < 0 || *contador == INT_MAX)
        return;

    resultado = snprintf(dst, (size_t)cap, "L%d", *contador);

    if (resultado < 0 || resultado >= cap) {
        dst[0] = '\0';
        return;
    }

    (*contador)++;
}

void emitir_flujo(char *dst, int cap,
                  const char *etiqueta, int es_salto) {
    int resultado;

    if (dst == NULL || cap <= 0)
        return;

    dst[0] = '\0';

    if (etiqueta == NULL)
        return;

    if (es_salto != 0) {
        resultado = snprintf(dst, (size_t)cap,
                             "GOTO %s", etiqueta);
    } else {
        resultado = snprintf(dst, (size_t)cap,
                             "%s:", etiqueta);
    }

    if (resultado < 0 || resultado >= cap)
        dst[0] = '\0';
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Saltos adelante y atrás estructurados

### Salida del harness
```text
=== AUDITORIA SESION 23 - Etiquetas y saltos incondicionales ===
Caso                               esperado:              obtenido:              resultado
[ 1] nueva_etiqueta(L0)                 esperado: L0           obtenido: L0           OK
[ 2] nueva_etiqueta(L5)                 esperado: L5           obtenido: L5           OK
[ 3] emitir_flujo(GOTO L2)              esperado: GOTO L2      obtenido: GOTO L2      OK
[ 4] emitir_flujo(marca L2)             esperado: L2:          obtenido: L2:          OK
[ 5] emitir_flujo(NULL)                 esperado:              obtenido:              OK
[ 6] nueva_etiqueta(L10)                esperado: L10          obtenido: L10          OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion23.md
```

* [x] Caso base y límites superados sin *Crash*.
* [x] No existen accesos fuera de límites ni *Memory Leaks*.
* [x] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión del código identifiqué que un caso importante era el posible desbordamiento del contador cuando alcanzara el valor máximo permitido para un entero (INT_MAX). Si el contador se incrementara después de llegar a ese límite, podría producirse un comportamiento indefinido.

> Para solucionar este problema incorporé la biblioteca limits.h y agregué una validación que impide incrementar el contador cuando alcanza INT_MAX.

> También reforcé la validación de la capacidad del búfer y revisé el valor devuelto por snprintf(). Si la cadena no cabe completamente en el espacio disponible, el código limpia el destino para evitar conservar una salida incompleta.

---
**Firma digital:** [BlancaHoma14]
