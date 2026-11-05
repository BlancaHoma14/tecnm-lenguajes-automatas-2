# Bitácora de Requerimientos y Auditoría IA — Sesión 08
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Inferencia y Promoción de Tipos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Reglas de casteo implícito  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente el código, mi principal dificultad fue comprender cómo comparar los rangos de los tipos de datos CHAR, INT y FLOAT para determinar cuál tiene mayor jerarquía.

También tuve que identificar cuándo una conversión puede provocar pérdida de información, especialmente al convertir un tipo de mayor rango a uno más pequeño.

Otra dificultad fue manejar correctamente el valor especial TIPO_ERROR y utilizar las estructuras condicionales y el switch para devolver el nombre correspondiente a cada tipo.

Para resolverlo utilicé comparaciones entre los valores del enumerado y condiciones que permiten identificar si una conversión pierde información.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Promoción entera y ensanchamiento |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
**Contrato C-R-E-O **

C – Contexto: Estoy desarrollando un programa en lenguaje C para la asignatura Lenguajes y Autómatas II. Necesito implementar reglas de promoción de tipos y detección de pérdida de información utilizando un enumerado llamado Tipo, que contiene los valores TIPO_ERROR, TIPO_CHAR, TIPO_INT y TIPO_FLOAT.

R – Rol: Actúa como un ingeniero de software especializado en lenguaje C, sistemas y desarrollo de compiladores.

E – Especificación: Necesito implementar las siguientes funciones:

Tipo promover(Tipo a, Tipo b);

int hay_perdida(Tipo destino, Tipo origen);

const char *nombre_tipo(Tipo t);

Deben cumplir las siguientes condiciones:

1. La jerarquía de tipos es CHAR < INT < FLOAT.

2. La función promover debe devolver el tipo de mayor rango.

3. Si alguno de los tipos recibidos es TIPO_ERROR, debe devolver TIPO_ERROR.

4. La función hay_perdida debe devolver 1 cuando el tipo destino sea más angosto que el tipo origen.

5. Debe devolver 0 cuando el destino tenga el mismo rango o mayor rango.

6. La función nombre_tipo debe devolver "error", "char", "int" o "float", según corresponda.

7. Si recibe un tipo desconocido, debe manejarlo de forma segura.

8. Respetar exactamente las firmas declaradas en api_sesion08.h.

9. No incluir la función main().

10. El código debe compilar sin advertencias utilizando GCC y las opciones -Wall -Wextra -Werror.

O – Output: Proporciona el código completo y modular para ia_sesion08_c.c.

Explica las mejoras realizadas respecto a mi implementación manual y analiza la complejidad temporal y espacial.

No modifiques el contrato api_sesion08.h ni el harness proporcionado por el docente.
```

---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion08_c.c
/*
 * ia_sesion08_c.c
 * Fase 2: El Cientifico (con IA)
 * Reglas de promocion y perdida mejoradas.
 */

#include "api_sesion08.h"

/* Comprobar si un tipo pertenece a la jerarquia */
static int tipo_valido(Tipo t) {

    return t >= TIPO_CHAR && t <= TIPO_FLOAT;
}

/* Determinar el tipo de mayor rango */
Tipo promover(Tipo a, Tipo b) {

    /* Verificar que ambos tipos sean validos */
    if (!tipo_valido(a) || !tipo_valido(b)) {
        return TIPO_ERROR;
    }

    /* Devolver el tipo de mayor rango */
    if (a > b) {
        return a;
    }

    return b;
}

/* Detectar si una conversion pierde informacion */
int hay_perdida(Tipo destino, Tipo origen) {

    /* Validar los tipos recibidos */
    if (!tipo_valido(destino) || !tipo_valido(origen)) {
        return 0;
    }

    /* Comprobar si el destino es mas angosto */
    return destino < origen;
}

/* Obtener el nombre correspondiente al tipo */
const char *nombre_tipo(Tipo t) {

    switch (t) {

        case TIPO_ERROR:
            return "error";

        case TIPO_CHAR:
            return "char";

        case TIPO_INT:
            return "int";

        case TIPO_FLOAT:
            return "float";

        default:
            return "desconocido";
    }
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Mezcla de tipos enteros y flotantes

### Salida del harness
```text
=== AUDITORIA SESION 08 - Reglas de promocion y perdida ===
Caso                               esperado:              obtenido:              resultado
[ 1] promover(CHAR,INT)                 esperado: int          obtenido: int          OK
[ 2] promover(INT,FLOAT)                esperado: float        obtenido: float        OK
[ 3] promover(FLOAT,FLOAT)              esperado: float        obtenido: float        OK
[ 4] promover(ERROR,INT)                esperado: error        obtenido: error        OK
[ 5] hay_perdida(CHAR,INT)              esperado: 1            obtenido: 1            OK
[ 6] hay_perdida(INT,CHAR)              esperado: 0            obtenido: 0            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion08.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la revisión de la propuesta de inteligencia artificial identifiqué que era necesario controlar los valores de enumeración desconocidos, ya que podrían recibirse valores que no pertenecen a la jerarquía CHAR, INT y FLOAT.

Para solucionar este problema agregué una función auxiliar llamada tipo_valido(), que comprueba que los tipos recibidos estén dentro del rango permitido.

También incorporé validaciones en las funciones promover() y hay_perdida() para evitar realizar comparaciones con tipos inválidos.

Este caso adicional lo detecté mediante la revisión del código, ya que los seis casos del harness proporcionado por el docente no comprueban valores de enumeración desconocidos.

Con estas modificaciones conseguí una implementación más segura, organizada y con un mejor control de los tipos recibidos.

---
**Firma digital:** [Blanca Janet Homa May]
