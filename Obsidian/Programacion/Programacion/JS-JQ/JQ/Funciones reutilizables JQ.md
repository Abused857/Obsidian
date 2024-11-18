
#### **Función `getFieldsByIdIncome`**

###### Devuelve un objeto estructurado que agrupa los campos del formulario por categorías según su fase o asignación.


```

function getFieldsByIdIncome() {
    return {
        fase: {
            fase: $(`#fase_`),
            ingresoNoJudicial: $(`#ingreso_no_judicial_`),
        },
        grabacion: {
            origen: $(`#origen_`),
            nCuentaJuzgado: $(`#n_cuenta_juzgado_`),
            impIngreso: $(`#imp_ingreso_`),
            fechaConstancia: $(`#fecha_constancia_`),
        },
        consolidacion: {
            enCuentaProcurador: $(`#en_cuenta_procurador_`),
            tipoSolicitud: $(`#tipo_solicitud_`),
            fechaSolicitud: $(`#fecha_solicitud_`),
            ubicacionMp: $(`#ubicacion_mp_`),
            fechaExpedicionMp: $(`#fecha_expedicion_mp_`),
            fechaCaducidadMp: $(`#fecha_caducidad_mp_`),
            tasaPagadaMp: $(`#tasa_pagada_mp_`),
            fechaConsolidacion: $(`#fecha_consolidacion_`),
            destinoConsolidacion: $(`#destino_consolidacion_`),
        },
        asignacion: {
            clientes: {
                titularCliente: $(`#titular_cliente_`),
                modoDeEntrega: $(`#modo_de_entrega_`),
                importeCliente: $(`#importe_cliente_`),
                pais: $(`#pais_`),
                codigoIban: $(`#codigo_iban_`),
                numeroCuenta: $(`#numero_cuenta_`),
            },
            aa: {
                titularAA: $(`#totular_aa_`),
                origenAA: $(`#origen_aa_`),
                concepto: $(`#concepto_`),
                procedimiento: $(`#prodecimiento_`),
                amountAA: $(`#amount_aa_`),
                estipulaciones: $(`#estipulaciones_`),
            },
            otrosImportes: {
                procurador: $(`#procurador_`),
                devJuzgadoAsignacion: $(`#dev_juzgado_asignacion_`),
            }
        },
        confirmacion: {
            fechaConfirmacion: $(`#fecha_confirmacion_`)
        }
    };
}

```

#### **Función `setDefaultValuesIncome`**

###### Restablece todos los campos del formulario relacionados con ingresos a sus valores predeterminados.
```
function setDefaultValuesIncome() {
    const fields = getFieldsByIdIncome();

    // Fase
    fields.fase.fase.val('');
    fields.fase.ingresoNoJudicial.val(0);

    // Grabación
    fields.grabacion.origen.val('');
    fields.grabacion.nCuentaJuzgado.val('');
    fields.grabacion.impIngreso.val('');
    fields.grabacion.fechaConstancia.val('');

    // Consolidación
    fields.consolidacion.enCuentaProcurador.val('');
    fields.consolidacion.tipoSolicitud.val('');
    fields.consolidacion.fechaSolicitud.val('');
    fields.consolidacion.ubicacionMp.val('');
    fields.consolidacion.fechaExpedicionMp.val('');
    fields.consolidacion.fechaCaducidadMp.val('');
    fields.consolidacion.tasaPagadaMp.val('');
    fields.consolidacion.fechaConsolidacion.val('');
    fields.consolidacion.destinoConsolidacion.val('');

    // Asignación - Clientes
    fields.asignacion.clientes.titularCliente.val('');
    fields.asignacion.clientes.modoDeEntrega.val('');
    fields.asignacion.clientes.importeCliente.val('');
    fields.asignacion.clientes.pais.val('');
    fields.asignacion.clientes.codigoIban.val('');
    fields.asignacion.clientes.numeroCuenta.val('');

    // Asignación - AA
    fields.asignacion.aa.titularAA.val('');
    fields.asignacion.aa.origenAA.val('');
    fields.asignacion.aa.concepto.val('');
    fields.asignacion.aa.procedimiento.val('');
    fields.asignacion.aa.amountAA.val('');
    fields.asignacion.aa.estipulaciones.val('');

    // Asignación - Otros Importes
    fields.asignacion.otrosImportes.procurador.val('');
    fields.asignacion.otrosImportes.devJuzgadoAsignacion.val('');

    // Confirmación
    fields.confirmacion.fechaConfirmacion.val('');

    // Resetear checkboxes
    $('input[type="checkbox"]').each(function() {
        $(this).prop('checked', false);
    });
}

```
#### **Función `clearAllFieldsIncome`**

###### Limpia todos los campos de una fase o subsección específica. Si no se especifica, limpia todo el formulario.

```
function clearAllFieldsIncome(selectedPhase) {
    const fields = getFieldsByIdIncome();

    function clearFields(fieldGroup) {
        if (fieldGroup) {
            $.each(fieldGroup, function(key, field) {
                const $field = $(field);

                if ($field.is(':checkbox')) {
                    $field.prop('checked', false);
                } else if ($field.is('select')) {
                    $field.prop('selectedIndex', 0);
                } else {
                    $field.val('');
                }
            });
        }
    }

    if (selectedPhase in fields) {
        clearFields(fields[selectedPhase]);
    }

    if (selectedPhase === 'asignacion') {
        $.each(fields.asignacion, function(subgroupKey, subgroupFields) {
            clearFields(subgroupFields);
        });
    }

    if (selectedPhase === 'consolidacion') {
        $.each(fields.asignacion, function(subgroupKey, subgroupFields) {
            clearFields(subgroupFields);
        });
    }
}

```

#### **Función `normalizeText`**

###### Normaliza un texto, convirtiéndolo a minúsculas y eliminando acentos o caracteres especiales.

```
function normalizeText(text) {
    return text.toLowerCase()
               .normalize('NFD')
               .replace(/[\u0300-\u036f]/g, '');
}

```

#### **Inicialización con `$(document).ready`**

###### Garantiza que el código se ejecuta solo cuando el DOM está completamente cargado.

```
$(document).ready(function() {
    console.log('DOM cargado y listo para usar.');
});

```

### **Título: Función `checkForDuplicateProcedure`**

#### **Descripción General**

La función `checkForDuplicateProcedure` es un método de validación que se utiliza para verificar si ya existe una fila en un listado de procedimientos (`addedProcedures`) con la misma combinación de `origen` y `procedimiento`, o si el `origen` es "liquidacion de intereses" y ya existe una fila con ese `origen` en el listado.

---

#### **Funcionamiento Detallado**

- **`addedProcedures`**: Es un arreglo donde se almacenan los procedimientos previamente agregados. Cada elemento del arreglo es un objeto que contiene propiedades como `origen`, `procedimiento`, y otros datos relevantes.
    
- **Parámetros de la función**:
    
    - `origen`: Una cadena que representa el origen del procedimiento a verificar.
    - `procedimiento`: Una cadena que representa el nombre del procedimiento a verificar.
- **Lógica de Validación**:
    
    - **Duplicado de `origen` y `procedimiento`**: La función itera sobre cada `item` en `addedProcedures` usando `forEach`. Para cada `item`, se verifica si el `origen` y el `procedimiento` coinciden exactamente con los de `item`. Si es así, se muestra una alerta y `isDuplicate` se establece en `true`.
    - **Duplicado de `origen` específico "liquidacion de intereses"**: Además, la función también verifica si el `origen` es "liquidacion de intereses" y si ya existe alguna fila con ese mismo `origen` en el listado. Si esto se cumple, también se muestra una alerta y `isDuplicate` se establece en `true`.

---

#### **Ejemplos de Uso**

1. **Validación de Procedimientos por `origen` y `procedimiento`**:

```
let addedProcedures = [
    { origen: 'liquidacion de intereses', procedimiento: 'ABC' },
    { origen: 'venta', procedimiento: 'XYZ' }
];

function checkForDuplicateProcedure(origen, procedimiento) {
    let isDuplicate = false;

    addedProcedures.forEach(item => {
        if (item.origen === origen && item.procedimiento === procedimiento) {
            alert("Ya existe una fila con la misma combinación de Origen y Procedimiento.");
            isDuplicate = true;
        }
        if (origen === "liquidacion de intereses" && item.origen === "liquidacion de intereses") {
            alert("Ya existe una fila con el mismo Origen (Liquidación de intereses) para el cliente.");
            isDuplicate = true;
        }
    });

    return isDuplicate;
}

// Ejemplo de uso:
let origen = 'liquidacion de intereses';
let procedimiento = 'ABC';

if (!checkForDuplicateProcedure(origen, procedimiento)) {
    // Agregar nuevo procedimiento ya que no es duplicado
    addedProcedures.push({ origen: origen, procedimiento: procedimiento });
}

```

**Validar antes de Agregar un Nuevo Procedimiento**:

```
let addedProcedures = [];

function checkForDuplicateProcedure(origen, procedimiento) {
    let isDuplicate = false;

    addedProcedures.forEach(item => {
        if (item.origen === origen && item.procedimiento === procedimiento) {
            alert("Ya existe una fila con la misma combinación de Origen y Procedimiento.");
            isDuplicate = true;
        }
        if (origen === "liquidacion de intereses" && item.origen === "liquidacion de intereses") {
            alert("Ya existe una fila con el mismo Origen (Liquidación de intereses) para el cliente.");
            isDuplicate = true;
        }
    });

    return isDuplicate;
}

// Ejemplo de uso:
let origenNuevo = 'liquidacion de intereses';
let procedimientoNuevo = 'XYZ';

if (!checkForDuplicateProcedure(origenNuevo, procedimientoNuevo)) {
    // Agregar nuevo procedimiento ya que no es duplicado
    addedProcedures.push({ origen: origenNuevo, procedimiento: procedimientoNuevo });
}

```

**Captura de Error y Manejo de Datos Ingresados**:


```
let addedProcedures = [
    { origen: 'liquidacion de intereses', procedimiento: 'ABC' },
    { origen: 'venta', procedimiento: 'XYZ' }
];

function checkForDuplicateProcedure(origen, procedimiento) {
    let isDuplicate = false;

    addedProcedures.forEach(item => {
        if (item.origen === origen && item.procedimiento === procedimiento) {
            alert("Ya existe una fila con la misma combinación de Origen y Procedimiento.");
            isDuplicate = true;
        }
        if (origen === "liquidacion de intereses" && item.origen === "liquidacion de intereses") {
            alert("Ya existe una fila con el mismo Origen (Liquidación de intereses) para el cliente.");
            isDuplicate = true;
        }
    });

    return isDuplicate;
}

// Ejemplo de uso en un escenario donde el usuario introduce datos:
let origenIngresado = prompt("Introduce el origen:");
let procedimientoIngresado = prompt("Introduce el procedimiento:");

if (!checkForDuplicateProcedure(origenIngresado, procedimientoIngresado)) {
    // Procedimiento válido, se puede agregar a la lista
    addedProcedures.push({ origen: origenIngresado, procedimiento: procedimientoIngresado });
} else {
    // Manejar el error de duplicado
    console.log("No se puede agregar, hay un duplicado.");
}
```


### **Notas Adicionales:**

- **Alertas**: Las alertas están ahí para indicar al usuario cuando se encuentra una fila duplicada. En un entorno de producción, es recomendable manejar estos errores de una manera menos intrusiva, como mostrando un mensaje en el interfaz del usuario o registrando el error para análisis posterior.
- **Retorno de la función**: `checkForDuplicateProcedure` devuelve `true` si se encuentra un duplicado y `false` si no hay duplicados. Esto permite decidir si se procede a agregar el nuevo procedimiento o no en el flujo de la aplicación.

Este tipo de función es útil en aplicaciones que manejan múltiples entradas de datos donde se necesita asegurarse de que no existan duplicados antes de proceder con la acción de agregar.


[[funciones_js_jq]]