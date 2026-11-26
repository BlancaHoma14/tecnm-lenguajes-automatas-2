# Bitácora de Requerimientos y Auditoría IA — Sesión 24
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Sentencia if-else  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Saltos condicionales y evaluación booleana  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente la función emitir_condicion(), identifiqué que la principal dificultad era representar correctamente las condiciones lógicas AND (&) y OR (|) mediante saltos condicionales.

Tuve que distinguir qué instrucción debía utilizar en cada caso: IFFALSE para AND y IFTRUE para OR. También debía asegurarme de que ambos saltos utilizaran la misma etiqueta de salida para mantener el comportamiento de cortocircuito.

Otra dificultad fue controlar el tamaño del arreglo lineas[][64], porque cada instrucción debe caber en un espacio de 64 caracteres, incluyendo el carácter nulo '\0'. Además, tuve que comprobar que el parámetro max fuera suficiente para almacenar una o dos instrucciones, dependiendo de si el segundo operando era NULL.

Para evitar errores de memoria utilicé snprintf(), validé los apuntadores y comprobé que el resultado de cada operación no excediera la capacidad disponible.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Cortocircuito lógico en expresiones |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTRATO C-R-E-O

Contexto (C): Estoy desarrollando una práctica de Lenguajes y Autómatas II (SCD-1016), correspondiente a la sesión 24: Control de Flujo, sentencias If e If-Else. Trabajo con lenguaje C y GCC. La implementación debe respetar el contrato de api_sesion24.h.

Rol (R): Actúa como un ingeniero de software especializado en compiladores, generación de código intermedio y control de flujo.

Especificación (E): Implementa y mejora la siguiente función, respetando exactamente su firma: int emitir_condicion(char lineas[][64], int max, const char *a, const char *b, char op, const char *etiqueta);
Requisitos:

1. Si op es &, generar instrucciones IFFALSE.
2. Si op es |, generar instrucciones IFTRUE.
3. Cada instrucción debe tener el formato: IFFALSE a GOTO L9 o IFTRUE a GOTO L9.
4. Si b no es NULL, generar dos instrucciones: una para a y otra para b.
5. Si b es NULL, generar solamente una instrucción para a.
6. Regresar el número de instrucciones generadas.
7. Validar apuntadores nulos, operador y capacidad disponible.
8. No escribir fuera de los límites de lineas ni de sus espacios de 64 caracteres.
9. Comprobar los resultados de snprintf() para detectar errores o truncamientos.
10. No modificar la firma, no incluir main() y no utilizar memoria dinámica.

Output (O): Entrega código modular compatible con C99/C11 y GCC, una explicación del funcionamiento, análisis de complejidad temporal y espacial y una descripción de los casos límite que deben probarse.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion24.h"
#include <stdio.h>
#include <string.h>

int emitir_condicion(char lineas[][64], int max,
                     const char *a, const char *b,
                     char op, const char *etiqueta) {
    char temporales[2][64] = {{0}};
    const char *instruccion;
    int cantidad;
    int resultado;

    /* Validar los argumentos principales */
    if (lineas == NULL || max <= 0 ||
        a == NULL || etiqueta == NULL) {
        return 0;
    }

    /* Seleccionar el salto según el operador lógico */
    switch (op) {
        case '&':
            instruccion = "IFFALSE";
            break;

        case '|':
            instruccion = "IFTRUE";
            break;

        default:
            return 0;
    }

    /* Determinar cuántas instrucciones se necesitan */
    cantidad = (b == NULL) ? 1 : 2;

    if (max < cantidad) {
        return 0;
    }

    /* Construir la primera instrucción temporal */
    resultado = snprintf(temporales[0], sizeof(temporales[0]),
                         "%s %s GOTO %s",
                         instruccion, a, etiqueta);

    if (resultado < 0 ||
        (size_t)resultado >= sizeof(temporales[0])) {
        return 0;
    }

    /* Construir la segunda instrucción si existe el operando b */
    if (b != NULL) {
        resultado = snprintf(temporales[1], sizeof(temporales[1]),
                             "%s %s GOTO %s",
                             instruccion, b, etiqueta);

        if (resultado < 0 ||
            (size_t)resultado >= sizeof(temporales[1])) {
            return 0;
        }
    }

    /* Copiar los resultados solamente si todo fue correcto */
    memcpy(lineas, temporales,
           (size_t)cantidad * sizeof(temporales[0]));

    return cantidad;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Condiciones compuestas con AND y OR

### Salida del harness
```text
=== AUDITORIA SESION 24 - Saltos condicionales con cortocircuito ===
Caso                               esperado:              obtenido:              resultado
[ 1] a && b (primer salto)
     esperado: IFFALSE a GOTO L9
     obtenido: IFFALSE a GOTO L9
     -> OK
[ 2] a && b (segundo salto)
     esperado: IFFALSE b GOTO L9
     obtenido: IFFALSE b GOTO L9
     -> OK
[ 3] a && b (dos saltos)                esperado: 2            obtenido: 2            OK
[ 4] a || b (primer salto)
     esperado: IFTRUE a GOTO L9
     obtenido: IFTRUE a GOTO L9
     -> OK
[ 5] a || b (segundo salto)
     esperado: IFTRUE b GOTO L9
     obtenido: IFTRUE b GOTO L9
     -> OK
[ 6] condicion simple (b NULL)
     esperado: IFFALSE a GOTO L0
     obtenido: IFFALSE a GOTO L0
     -> OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion24.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión del código identifiqué que era importante comprobar el comportamiento cuando el arreglo no tenía capacidad suficiente para almacenar las instrucciones. Por ejemplo, si existen dos operandos, pero max solamente permite una instrucción, el programa no debe intentar escribir la segunda posición.

Para solucionar este problema se agregó la validación:
if (max < cantidad) {
    return 0;
}
También se revisó el caso en el que un operando o la etiqueta fueran demasiado largos para caber en los 64 caracteres disponibles. Para ello se comprueba el valor que devuelve snprintf(). Si la cadena no cabe, la función regresa cero y no copia las instrucciones temporales al arreglo definitivo.

---
**Firma digital:** [BlancaHoma14]
