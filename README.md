# Práctica XML y DTD

## Objetivo

Diseñar documentos XML a partir de información no estructurada, construir
documentos bien formados, definir DTD internos y externos, aplicar
cardinalidades y restricciones de atributos, validar los documentos y
gestionar los artefactos mediante Git.

## Ejercicio 1: Pedido

### Modelo propuesto

Elemento raíz: pedido.

Estructura:

    pedido
    ├── destinatario
    ├── articulo
    ├── direccion
    │   ├── calle
    │   ├── numero
    │   ├── piso
    │   └── letra
    └── fecha_entrega

### Decisiones de diseño

Se descompone la dirección en calle, número, piso y letra para poder
tratar cada componente de forma independiente y facilitar su
procesamiento.

La fecha se almacena en formato ISO 8601 (AAAA-MM-DD) para que sea
fácilmente procesable.

El destinatario se modela como elemento y no como atributo, porque
contiene información propia que puede crecer en el futuro.

## Ejercicio 2: Nota

### DTD externo

Archivos: nota.xml y nota.dtd.

El DTD externo declara:

    <!ELEMENT nota (para, de, titulo, contenido)>
    <!ELEMENT para (#PCDATA)>
    <!ELEMENT de (#PCDATA)>
    <!ELEMENT titulo (#PCDATA)>
    <!ELEMENT contenido (#PCDATA)>

El orden de los elementos es significativo.

### DTD interno

Archivo: nota-interno.xml.

El DTD se declara dentro del propio documento XML mediante:

    <!DOCTYPE nota [
        ...
    ]>

### Pruebas realizadas

| Modificación | Bien formado | Válido | Por qué |
|--------------|--------------|--------|---------|
| Cambiar para por destinatario | Sí | No | El elemento destinatario no está declarado en el DTD |
| Cambiar el orden de para y de | Sí | No | El DTD exige el orden (para, de, titulo, contenido) |
| Agregar telefono | Sí | No | El elemento telefono no está declarado en el DTD |

### Comparación DTD interno y externo

| Característica | DTD interno | DTD externo |
|----------------|-------------|-------------|
| Ubicación | Dentro del propio XML | En un archivo independiente |
| Reutilizable entre XML | No | Sí |
| Archivo adicional | No | Sí |
| Conveniente para un único documento | Sí | No |
| Conveniente para muchos documentos | No | Sí |

## Ejercicio 3: Matrícula

### Modelo

Elemento raíz: matricula.

Estructura:

    matricula
    ├── personal
    │   ├── dni
    │   ├── nombre
    │   ├── titulacion
    │   ├── curso_academico
    │   └── domicilios
    │       └── domicilio+
    └── pago
        └── tipo_matricula

Clasificación de los elementos:

| Elemento | Tipo | Repetible | Atributos | Restricciones |
|----------|------|-----------|-----------|---------------|
| matricula | Compuesto (raíz) | No | No | - |
| personal | Compuesto | No | No | - |
| dni | Simple | No | No | - |
| nombre | Simple | No | No | - |
| titulacion | Simple | No | No | - |
| curso_academico | Simple | No | No | - |
| domicilios | Compuesto | No | No | - |
| domicilio | Compuesto | Sí (+) | tipo | tipo obligatorio y solo familiar o habitual |
| pago | Compuesto | No | No | - |
| tipo_matricula | Simple | No | No | - |

### Cardinalidad

El requisito establece que debe existir al menos un domicilio. El
operador que corresponde a "uno o más" es +.

Se incorpora al DTD de la siguiente forma:

    <!ELEMENT domicilios (domicilio+)>

Prueba eliminando temporalmente todos los domicilios:

    ¿XML bien formado? Sí
    ¿XML válido? No
    ¿Por qué? El DTD exige (domicilio+), es decir, al menos un
    domicilio. Al no haber ninguno se viola la cardinalidad mínima.

### Restricción del atributo tipo

El atributo tipo se declara como obligatorio y con una enumeración que
solo admite familiar o habitual:

    <!ELEMENT domicilio (nombre)>
    <!ATTLIST domicilio
        tipo (familiar | habitual) #REQUIRED>

Pruebas realizadas:

| Caso | Predicción | Resultado | Explicación |
|------|------------|-----------|-------------|
| tipo="familiar" | Válido | Válido | Está en la enumeración permitida |
| tipo="habitual" | Válido | Válido | Está en la enumeración permitida |
| tipo="temporal" | No válido | No válido | No pertenece a la enumeración permitida |
| sin tipo | No válido | No válido | El atributo es #REQUIRED, es obligatorio |

### DTD externo

Archivos: matricula.xml y matricula.dtd.

El XML referencia el DTD mediante:

    <!DOCTYPE matricula SYSTEM "matricula.dtd">

### DTD interno

Archivo: matricula-interno.xml.

El DTD se declara dentro del propio documento XML mediante:

    <!DOCTYPE matricula [
        ...
    ]>

### Pruebas realizadas

Las pruebas del atributo tipo y de la cardinalidad domicilio+ se
realizaron tanto en la versión con DTD externo como en la versión con
DTD interno, obteniendo los mismos resultados en ambos casos.

### Uso de ramas en Git

Para desarrollar la variante con DTD interno se creó la rama
dtd-interno-matricula, se realizó el commit correspondiente y después
se fusionó con la rama principal mediante git merge.

## Conclusiones

1. Un XML bien formado cumple las reglas sintácticas del lenguaje:
   un único elemento raíz, etiquetas cerradas, anidamiento correcto y
   atributos entre comillas. Un XML válido además cumple las reglas
   definidas en un DTD o esquema.

2. Un DTD define la estructura permitida de un documento XML:
   qué elementos existen, en qué orden aparecen, cuántas veces pueden
   repetirse y qué atributos admiten.

3. Un DTD interno se declara dentro del propio documento XML y solo
   se aplica a ese documento. Un DTD externo se guarda en un archivo
   independiente y puede reutilizarse en varios documentos XML.

4. La cardinalidad en DTD se expresa con los símbolos ?, *, y +:
   ? indica cero o uno, * indica cero o más, y + indica uno o más.

5. Un atributo se restringe a determinados valores mediante una
   enumeración en el ATTLIST, por ejemplo:
   tipo (familiar | habitual) #REQUIRED.

6. Git permitió mantener un historial de los cambios, probar
   modificaciones sin perder la versión válida y revertir estados
   anteriores cuando fue necesario.

7. git diff permitió ver las diferencias entre la versión actual y la
   última confirmación. git restore permitió recuperar el documento
   válido tras las pruebas negativas.

8. El uso de una rama permitió desarrollar la variante con DTD interno
   sin afectar a la rama principal, y fusionarla después cuando estuvo
   lista.