# Bitácora de Requerimientos y Auditoría IA — Sesión 25
**Materia:** Lenguajes y Autómatas II (SCD-1016)  
**Tema:** Control de Flujo: Bucles while y for  
**Docente Titular:** MCC. Iván Márquez Larios (`ijmarquezl`)  
**Estudiante:** [Blanca Janet Homa May]  
**Matrícula:** [24531351]  
**Track:** C (el harness evalúa C; la versión en Rust es opcional y no se califica con este harness)  

---

## 1. Fase 1: El Artesano (Dificultad Manual)
*Reto:* Estructura de ciclo en instrucciones TAC  
*Describe qué problema o restricción de memoria/sintaxis identificaste al codificar a mano:*
> Al desarrollar manualmente las funciones emitir_while() y emitir_for(), identifiqué que la principal dificultad era organizar correctamente las instrucciones del ciclo y controlar las etiquetas de inicio y salida.

En el ciclo while tuve que generar cuatro instrucciones en el orden correcto: etiqueta de inicio, salto condicional, salto de regreso y etiqueta de salida.

En el ciclo for fue necesario colocar la inicialización antes de comenzar el ciclo y la actualización del iterador después de evaluar la condición, pero antes del salto de regreso. También tuve que considerar que los argumentos inicio y post podían ser NULL, por lo que esas instrucciones debían omitirse cuando no existieran.

Otra dificultad fue controlar el tamaño del arreglo lineas[][64], porque cada instrucción debía ocupar como máximo 63 caracteres más el terminador nulo. Además, tuve que evitar que el contador n superara la capacidad max y asegurar que cada ciclo utilizara dos etiquetas diferentes.

Para resolver estas dificultades utilicé snprintf(), validaciones de apuntadores y comprobaciones de capacidad antes de escribir las instrucciones.

---

## 2. Fase 2: El Científico (Ingeniería de Requerimientos C-R-E-O)

### 2.1 Matriz de Especificación Formal
| Elemento C-R-E-O | Definición Técnica |
| :--- | :--- |
| **Contexto (C)** | C99 / GCC o Rust 2021 sin bibliotecas no estándar |
| **Rol (R)** | Ingeniero de software de sistemas y compiladores |
| **Especificación (E)** | Garantía de actualización de iterador |
| **Output (O)** | Solo código modular y análisis de complejidad |

### 2.2 Prompt Estructurado
```text
CONTRATO C-R-E-O

Contexto (C): Estoy trabajando en la materia Lenguajes y Autómatas II (SCD-1016), en la sesión 25, cuyo tema es Control de Flujo: bucles While y For. Utilizo lenguaje C con GCC y debo implementar las funciones declaradas en api_sesion25.h.

Rol (R): Actúa como un ingeniero de software especializado en compiladores, generación de código intermedio TAC y control de flujo.

Especificación (E): Implementa y mejora las siguientes funciones respetando exactamente sus firmas: void emitir_while(char lineas[][64], int max, int *n, const char *cond, int *etiquetas);

void emitir_for(char lineas[][64], int max, int *n, const char *inicio, const char *cond, const char *post, int *etiquetas);

Requisitos para emitir_while():
1. Generar cuatro instrucciones: etiqueta de inicio, salto condicional, GOTO de regreso y etiqueta de salida.
2. Utilizar IFFALSE para evaluar la condición.
3. El salto condicional debe apuntar a la etiqueta de salida.
4. El GOTO debe regresar a la etiqueta de inicio. Requisitos para emitir_for():
5. Generar la inicialización si inicio no es NULL.
6. Generar la etiqueta de inicio.
7. Generar el salto condicional con IFFALSE.
8. Generar la actualización si post no es NULL.
9. Generar el GOTO de regreso.
10. Generar la etiqueta de salida.

Restricciones generales:
- Cada ciclo debe consumir dos etiquetas consecutivas y únicas.
- Las etiquetas deben incrementarse en dos después de generar correctamente el ciclo.
- El parámetro n debe actualizarse con el número de líneas generadas.
- No emitir instrucciones para inicio o post cuando sean NULL.
- Validar apuntadores, capacidad del arreglo y límites de los contadores.
- Cada línea dispone de 64 caracteres.
- Utilizar snprintf() y comprobar errores o truncamientos.
- No utilizar memoria dinámica.
- No modificar las firmas ni incluir main().
- Mantener compatibilidad con C99/C11 y GCC.

Output (O): Entrega código modular en C para ambas funciones, explicación de su funcionamiento, análisis de complejidad temporal y espacial, y una explicación de los casos límite considerados.
```

---

## 3. Código Devuelto por el Modelo
```c
#include "api_sesion25.h"
#include 
#include 
#include 

void emitir_while(char lineas[][64], int max,
                  int *n, const char *cond, int *etiquetas) {
    char temp[4][64] = {{0}};
    int inicio;
    int fin;
    int r;

    /* Validar los argumentos */
    if (lineas == NULL || n == NULL || cond == NULL ||
        etiquetas == NULL || max < 0 ||
        *n < 0 || *n > max ||
        *etiquetas < 0 || *etiquetas > INT_MAX - 2) {
        return;
    }

    /* El while necesita cuatro líneas */
    if (max - *n < 4) {
        return;
    }

    inicio = *etiquetas;
    fin = inicio + 1;

    r = snprintf(temp[0], sizeof(temp[0]), "L%d:", inicio);
    if (r < 0 || (size_t)r >= sizeof(temp[0])) return;

    r = snprintf(temp[1], sizeof(temp[1]),
                 "IFFALSE %s GOTO L%d", cond, fin);
    if (r < 0 || (size_t)r >= sizeof(temp[1])) return;

    r = snprintf(temp[2], sizeof(temp[2]), "GOTO L%d", inicio);
    if (r < 0 || (size_t)r >= sizeof(temp[2])) return;

    r = snprintf(temp[3], sizeof(temp[3]), "L%d:", fin);
    if (r < 0 || (size_t)r >= sizeof(temp[3])) return;

    /* Copiar solo después de generar todo correctamente */
    memcpy(&lineas[*n], temp, sizeof(temp));

    *n += 4;
    *etiquetas += 2;
}

void emitir_for(char lineas[][64], int max, int *n,
                const char *inicio, const char *cond,
                const char *post, int *etiquetas) {
    char temp[6][64] = {{0}};
    int etiqueta_inicio;
    int etiqueta_fin;
    int cantidad = 0;
    int r;

    /* Validar los argumentos */
    if (lineas == NULL || n == NULL || cond == NULL ||
        etiquetas == NULL || max < 0 ||
        *n < 0 || *n > max ||
        *etiquetas < 0 || *etiquetas > INT_MAX - 2) {
        return;
    }

    etiqueta_inicio = *etiquetas;
    etiqueta_fin = etiqueta_inicio + 1;

    /* Inicialización opcional */
    if (inicio != NULL) {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "%s", inicio);
        if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
        cantidad++;
    }

    /* Etiqueta de inicio */
    r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                 "L%d:", etiqueta_inicio);
    if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
    cantidad++;

    /* Condición de salida */
    r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                 "IFFALSE %s GOTO L%d", cond, etiqueta_fin);
    if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
    cantidad++;

    /* Actualización opcional */
    if (post != NULL) {
        r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                     "%s", post);
        if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
        cantidad++;
    }

    /* Salto de regreso */
    r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                 "GOTO L%d", etiqueta_inicio);
    if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
    cantidad++;

    /* Etiqueta de salida */
    r = snprintf(temp[cantidad], sizeof(temp[cantidad]),
                 "L%d:", etiqueta_fin);
    if (r < 0 || (size_t)r >= sizeof(temp[cantidad])) return;
    cantidad++;

    /* Verificar capacidad antes de modificar el arreglo final */
    if (max - *n < cantidad) {
        return;
    }

    /* Copiar las instrucciones generadas */
    memcpy(&lineas[*n], temp,
           (size_t)cantidad * sizeof(temp[0]));

    *n += cantidad;
    *etiquetas += 2;
}

```

---

## 4. Fase 3: El Auditor (Resultados del Test Harness)
*Pruebas ejecutadas:* Bucles anidados y condiciones de parada

### Salida del harness
```text
=== AUDITORIA SESION 25 - Esqueleto TAC de while y for ===
Caso                               esperado:              obtenido:              resultado
[ 1] while: marca de inicio             esperado: L0:          obtenido: L0:          OK
[ 2] while: condicion
     esperado: IFFALSE a < b GOTO L1
     obtenido: IFFALSE a < b GOTO L1
     -> OK
[ 3] dos ciclos sin colision            esperado: L2:          obtenido: L2:          OK
[ 4] for: inicializacion                esperado: i = 0        obtenido: i = 0        OK
[ 5] for: actualizacion                 esperado: i = i + 1    obtenido: i = i + 1    OK
[ 6] for sin post (NULL)                esperado: 5            obtenido: 5            OK

RESULTADO: 6/6 pruebas superadas
TODO EN VERDE. Documenta tu auditoria en auditorias/auditoria_sesion25.md
```

* [✓] Caso base y límites superados sin *Crash*.
* [✓] No existen accesos fuera de límites ni *Memory Leaks*.
* [✓] La complejidad teórica coincide con la implementada.

### Reporte Crítico
*¿Qué caso borde o problema omitió la IA y cómo lo solucionaste?*
> Durante la auditoría identifiqué que era importante comprobar qué sucede cuando el arreglo no tiene espacio suficiente para almacenar todas las instrucciones de un ciclo.

Por ejemplo, si el ciclo while necesita cuatro líneas y el arreglo solamente tiene espacio para tres, no debe escribir parcialmente el ciclo ni modificar los contadores.

También consideré el caso en el que la inicialización o la actualización del for sean demasiado largas para caber en los 64 caracteres disponibles.

Para prevenir estos problemas se utilizaron arreglos temporales y se verificó el resultado de cada llamada a snprintf(). Además, se comprobó la capacidad disponible antes de copiar las instrucciones al arreglo definitivo.

---
**Firma digital:** [BlancaHoma14]
