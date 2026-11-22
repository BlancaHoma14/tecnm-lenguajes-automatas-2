# Bitácora de Requerimientos y Auditoría IA — Sesión 17
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Comprobación Semántica de Retornos  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Validación de tipo de sentencia return  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al realizar la implementación manual, identifiqué que era necesario comparar correctamente los tipos de datos que devuelve una función con el tipo que tiene declarado. También tuve que considerar las conversiones permitidas, por ejemplo, cuando una función de tipo `double` recibe un retorno de tipo `int`, ya que esta conversión sí es válida. Otra dificultad fue comprobar que todas las rutas de ejecución tuvieran un retorno y validar que los datos de entrada no fueran nulos o incoherentes. Para solucionar estos problemas utilicé condiciones `if`, comparaciones con `strcmp()` y validaciones antes de realizar el diagnóstico.
---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Rutas de ejecución con retorno garantizado |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
[C-Contexto: Necesito implementar una función en C para un compilador simplificado que valide retornos y rutas de ejecución.

R-Rol: Actúa como un desarrollador experto en C y análisis semántico de compiladores.

E-Especificación:La función diagnosticar_retorno() debe comprobar que el tipo de valor devuelto sea compatible con el tipo declarado de la función y verificar que todas las rutas de ejecución finalicen con un return. También debe detectar errores en los datos de entrada, incompatibilidades de tipos y rutas sin retorno, devolviendo el código correspondiente según cada caso.

O-Output: Entregar únicamente código C compilable.]
```
---

## 3. Código Devuelto por el Modelo
```c
%%writefile ia_sesion17_c.c
/* ia_sesion17_c.c - Fase 2: El Cientifico
 *
 * Pega aqui el codigo que te devolvio la IA, con la MISMA firma
 * que declara api_sesion17.h. No puede traer main().
 */
#include "api_sesion17.h"
#include "api_sesion17.h"
#include 

static int compatible(const char *destino, const char *origen)
{
    if (strcmp(destino, "void") == 0)
    {
        return strcmp(origen, "void") == 0;
    }

    if (strcmp(destino, "int") == 0)
    {
        return strcmp(origen, "int") == 0;
    }

    if (strcmp(destino, "double") == 0)
    {
        return strcmp(origen, "double") == 0 ||
               strcmp(origen, "int") == 0;
    }

    return 0;
}

int diagnosticar_retorno(const char *tipo_funcion,
                         const char *tipo_expresion,
                         int rutas_totales,
                         int rutas_con_return)
{
    if (tipo_funcion == NULL ||
        tipo_expresion == NULL)
    {
        return -1;
    }

    if (rutas_totales <= 0 ||
        rutas_con_return > rutas_totales)
    {
        return -1;
    }

    if (!compatible(tipo_funcion, tipo_expresion))
    {
        return 1;
    }

    if (rutas_con_return < rutas_totales)
    {
        return 2;
    }

    return 0;
}
```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Funciones sin retorno o retornos mixtos

### Salida del harness
```text
Pega aquí la salida de la celda del harness. Debe terminar con:
RESULTADO: n/n pruebas superadas
```            === AUDITORIA SESION 17 - Validacion de retornos y rutas ===
Caso                            esperado:          obtenido:       resultado
[ 1] int con int y dos rutas    esperado: 0        obtenido: 0         OK
[ 2] void devuelve valor        esperado: 1        obtenido: 1         OK
[ 3] ruta sin retorno           esperado: 2        obtenido: 2         OK
[ 4] double acepta int          esperado: 0        obtenido: 0         OK
[ 5] int no acepta double       esperado: 1        obtenido: 1         OK
[ 6] cero rutas de ejecucion    esperado: -1       obtenido: -1        OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion17.md
 
* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Al revisar el código generado por la IA, identifiqué que faltaba validar el caso en el que el número de rutas con return fuera negativo. Aunque el programa comprobaba que no existieran más retornos que rutas totales, no verificaba que la cantidad de retornos fuera menor que cero. Esto podía permitir que se procesaran datos incoherentes. Para solucionarlo, agregué la condición `rutas_con_return < 0` en la validación inicial, haciendo que la función devolviera -1 cuando se presentara ese caso. Con esta corrección mejoré la validación de los datos de entrada y reforcé la confiabilidad del programa.
---
**Firma digital:** [BlancaHoma14]
