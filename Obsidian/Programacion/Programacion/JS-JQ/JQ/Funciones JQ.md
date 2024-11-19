#### Seleccionar la id en jq

```
$('#id_del_objeto')
```

#### Seleccionar el value en jq de una id

```
$('#id_del_objeto').val()
```

#### Aisgnar un value a un objeto en jq

```
importe = x;
$('#id_del_objeto').val(importe)
```

#### Seleccionar por clase

```
$('.clase_del_objeto')
```

#### Seleccionar todos los elementos de un tipo

```
$('tag_name')
```
###### Ejemplo: $('div')
#### Seleccionar elementos anidados

```
$('div .clase_interna') 
```
###### Selecciona los elementos con la clase interna dentro de un div


#### Seleccionar elementos por atributo

```
$('[name="nombre_atributo"]')`
```
###### Selecciona todos los elementos con el atributo `name="nombre_atributo"`

---

#### Obtener el texto de un elemento

```
$('#id_del_objeto').text()`
```
###### Obtiene el texto contenido en el elemento con el id especificado.

---

#### Asignar texto a un elemento

```
$('#id_del_objeto').text('Nuevo texto')`
```
###### Cambia el texto del elemento con el id especificado a "Nuevo texto".

---

#### Obtener el HTML de un elemento

```
$('#id_del_objeto').html()`
```
###### Obtiene el contenido HTML del elemento con el id especificado.

---

#### Asignar HTML a un elemento

```
$('#id_del_objeto').html('<b>Texto en negrita</b>')`
```
###### Cambia el contenido HTML del elemento al valor indicado.

---

#### Obtener el valor de un atributo

```
$('#id_del_objeto').attr('atributo')`
```
###### Obtiene el valor del atributo especificado en el elemento.

---

#### Asignar un valor a un atributo

```
$('#id_del_objeto').attr('atributo', 'nuevo_valor')`
```
###### Cambia el atributo especificado al valor indicado.

---

#### Agregar una clase a un elemento

```
$('#id_del_objeto').addClass('nueva_clase')`
```
###### Agrega la clase "nueva_clase" al elemento con el id especificado.

---

#### Quitar una clase de un elemento

```
$('#id_del_objeto').removeClass('clase_existente')`
```

###### Elimina la clase "clase_existente" del elemento con el id especificado.

---

#### Alternar una clase

```
$('#id_del_objeto').toggleClass('clase_a_alternar')`
```

###### Si la clase "clase_a_alternar" está presente, la elimina; si no, la agrega.

---

#### Verificar si un elemento tiene una clase

```
$('#id_del_objeto').hasClass('clase_verificada')`
```

###### Retorna `true` si el elemento tiene la clase "clase_verificada"; de lo contrario, `false`.

---

#### Ocultar un elemento

```
$('#id_del_objeto').hide()`
```

###### Oculta el elemento con el id especificado.

---

#### Mostrar un elemento

```
$('#id_del_objeto').show()`
```

###### Muestra el elemento con el id especificado.

---

#### Alternar la visibilidad de un elemento

```
$('#id_del_objeto').toggle()`
```

###### Cambia entre ocultar y mostrar el elemento.

---

#### Detectar clic en un elemento

```
$('#id_del_objeto').on('click', function() {     console.log('Se hizo clic en el objeto'); })`
```

###### Ejecuta una acción al hacer clic en el elemento.

---

#### Detectar cambio en un input

```
$('#id_del_objeto').on('change', function() {     console.log('El valor cambió a: ' + $(this).val()); })`
```

###### Ejecuta una acción cuando cambia el valor del input.

---

#### Detectar hover sobre un elemento

```
$('#id_del_objeto').hover(     function() { console.log('Hover in'); },      function() { console.log('Hover out'); } )`
```

###### Detecta cuando el puntero entra y sale de un elemento.

---

#### Detectar envío de un formulario

```
$('#id_formulario').on('submit', function(event) {     event.preventDefault();     console.log('Formulario enviado'); })`
```

###### Previene el envío del formulario y ejecuta una acción.

---

#### Agregar contenido al final de un elemento

```
`$('#id_del_objeto').append('<p>Texto agregado al final</p>')`
```

###### Agrega el contenido indicado al final del elemento.

---

#### Agregar contenido al inicio de un elemento

```
$('#id_del_objeto').prepend('<p>Texto agregado al inicio</p>')`
```
###### Agrega el contenido indicado al inicio del elemento.

---

#### Eliminar un elemento del DOM

```
$('#id_del_objeto').remove()`
```
###### Elimina el elemento con el id especificado del DOM.

---

#### Vaciar el contenido de un elemento

```
`$('#id_del_objeto').empty()`
```
###### Elimina todo el contenido dentro del elemento especificado.

---

#### Obtener un valor CSS

```
$('#id_del_objeto').css('propiedad_css')`
```
###### Obtiene el valor de la propiedad CSS indicada.

---

#### Asignar un valor CSS

```
$('#id_del_objeto').css('propiedad_css', 'valor')`
```
###### Cambia el valor de la propiedad CSS indicada.

---

#### Asignar múltiples valores CSS

```
$('#id_del_objeto').css({     'color': 'red',     'font-size': '16px' })`
```
###### Asigna varias propiedades CSS al elemento.

#### Seleccionar todas las IDs que empiecen por un valor específico

```
$('[id^="inicio_id"]')`
```
###### Selecciona todos los elementos cuyo `id` comience con "inicio_id".

---

#### Seleccionar todas las IDs que terminen por un valor específico

```
$('[id$="fin_id"]')`
```
###### Selecciona todos los elementos cuyo `id` termine con "fin_id".

---

#### Seleccionar todas las IDs que contengan un valor específico

```
$('[id*="parte_id"]')`
```
###### Selecciona todos los elementos cuyo `id` contenga "parte_id".

---

#### Ocultar un elemento con animación

```
$('#id_del_objeto').fadeOut(400)`
```
###### Oculta el elemento con un efecto de desvanecimiento. El valor `400` representa la duración en milisegundos.

---

#### Mostrar un elemento con animación

```
$('#id_del_objeto').fadeIn(400)`
```
###### Muestra el elemento con un efecto de desvanecimiento. El valor `400` representa la duración en milisegundos.

---

#### Alternar visibilidad con animación

```
$('#id_del_objeto').fadeToggle(400)`
```
###### Alterna entre ocultar y mostrar con un efecto de desvanecimiento.

---

#### Ocultar un elemento con un slide hacia arriba

```
$('#id_del_objeto').slideUp(400)`
```
###### Oculta el elemento con un efecto de deslizamiento hacia arriba.

---

#### Mostrar un elemento con un slide hacia abajo

```
$('#id_del_objeto').slideDown(400)`
```
###### Muestra el elemento con un efecto de deslizamiento hacia abajo.

---

#### Alternar visibilidad con un slide

```
$('#id_del_objeto').slideToggle(400)`
```
###### Alterna entre ocultar y mostrar con un efecto de deslizamiento.

---

#### Deshabilitar un elemento

```
$('#id_del_objeto').prop('disabled', true)`
```
###### Deshabilita el elemento seleccionado.

---

#### Habilitar un elemento

```
$('#id_del_objeto').prop('disabled', false)`
```
###### Habilita el elemento seleccionado.

---

#### Seleccionar el padre de un elemento

```
$('#id_del_objeto').parent()`
```
###### Selecciona el padre directo del elemento.

---

#### Seleccionar todos los hijos de un elemento

```
$('#id_del_objeto').children()`
```
###### Selecciona todos los hijos directos del elemento.

---

#### Seleccionar un hijo específico (por índice)

```
`$('#id_del_objeto').children().eq(0)`
```
###### Selecciona el primer hijo del elemento (índice empieza en 0).

---

#### Seleccionar todos los elementos hermanos

```
`$('#id_del_objeto').siblings()`
```
###### Selecciona todos los elementos hermanos del elemento.

---

#### Seleccionar el siguiente hermano

```
`$('#id_del_objeto').next()`
```
###### Selecciona el siguiente hermano directo del elemento.

---

#### Seleccionar el hermano anterior

```
$('#id_del_objeto').prev()`
```
###### Selecciona el hermano directo anterior del elemento.

---

#### Ejecutar código después de que el DOM esté listo

```
$(document).ready(function() {     console.log('El DOM está listo'); })`
```
###### Asegura que el código se ejecute solo después de que el DOM esté completamente cargado.

---

#### Ejecutar una función al cargar una página completamente (incluyendo imágenes)

```
$(window).on('load', function() {     console.log('La página está completamente cargada'); })`
```
###### Detecta cuando todo el contenido de la página (incluidas imágenes) ha cargado.

---

#### Ejecutar una función al cambiar el tamaño de la ventana

```
$(window).on('resize', function() {     console.log('Se cambió el tamaño de la ventana'); })`
```
###### Ejecuta una función cuando la ventana cambia de tamaño.

---

#### Scroll hacia un elemento específico

```
$('html, body').animate({     scrollTop: $('#id_del_objeto').offset().top }, 500)`
```
###### Realiza un scroll suave hacia el elemento especificado. El valor `500` representa la duración en milisegundos.

---

#### Comprobar si un elemento está visible

```
if ($('#id_del_objeto').is(':visible')) {     console.log('El elemento está visible'); }`
```
###### Verifica si el elemento está visible en la página.

---

#### Comprobar si un elemento está oculto

```
if ($('#id_del_objeto').is(':hidden')) {     console.log('El elemento está oculto'); }`
```
###### Verifica si el elemento está oculto.

---

#### Clonar un elemento

```
var clon = $('#id_del_objeto').clone() $('#otra_id').append(clon)`
```
###### Clona el elemento seleccionado y lo agrega a otro elemento.

---

#### Comprobar si un checkbox está seleccionado

```
`if ($('#id_checkbox').is(':checked')) {     console.log('El checkbox está seleccionado'); }`
```
###### Verifica si un checkbox está seleccionado (`true` o `false`).

---

#### Enfocar un input

```
`$('#id_del_input').focus()`
```
###### Coloca el cursor en el input especificado.

---

#### Eliminar el foco de un input

```
`$('#id_del_input').blur()`
```
###### Elimina el foco del input especificado.

---

#### Asignar un retraso antes de ejecutar una función

```
`setTimeout(function() {     console.log('Función ejecutada después de 2 segundos'); }, 2000)`
```
###### Ejecuta una función después de 2000 milisegundos (2 segundos).


#### Seleccionar una opción en un select por su id y this en una funcion de select, comprobarlo respecto a un array

```
const nameSelector = $('idOfElementi);

nameSelector.find(ioption:selected).text().(aqui se podría usar un toLOwertCase() por ejemplo).

const phaseOrder = ['index1', 'index2', 'index3'];

nameSelector.on('change', function(){
const newSelected = $(this).find('option:selected').text();

const currentPhaseIndex = phaseOrder.indexOf(nameSelector);
const selectedPhaseIndex = phaseOrder.idexOf(newSelected);

if (selectedPhaseIndex < currentPhaseIndex) {
} else {
}


})
```
###### Selecciona la id de un elemento con opciones de select, se coje el valor que del select y se hacen las comprobaciones necesarias con el.


#### Alert, mensaje advertencia

```
const confirm = confirm("mensaje");

if (confirmChange) {
} else {
}
```
###### Mensaje de alerta  / advertencia usado para informar y que diga si o no.

#### Seleccionar value de un select

```
let currentPhase = phaseSelector.val() (se podria añadir por ejemplo .toLowerCase());
```
###### Seleccionar el value de un combobox / select.




[[funciones_js_jq]]