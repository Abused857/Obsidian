
#### **Cómo Funciona `MutationObserver`**

1. **Crear el Observador:**
    
    - Define un observador con una función de callback que manejará las mutaciones.
```
 const observer = new MutationObserver(function (mutationsList, observer) {
    mutationsList.forEach(mutation => {
        console.log(mutation); // Registra el tipo y detalles de la mutación.
    });
});

```

**Configurar Opciones:**

- Las opciones determinan qué tipos de cambios monitorear.

```
const config = {
    childList: true,  // Detectar cambios en nodos hijos.
    attributes: true, // Detectar cambios en atributos.
    characterData: true, // Detectar cambios en el contenido de texto.
    subtree: true,    // Observar recursivamente todos los descendientes.
};

```

**Aplicar el Observador a un Elemento:**

- Selecciona el elemento del DOM a observar.

```
observer.observe(document.querySelector('#elemento'), config);
```

**Desconectar el Observador:**

- Finaliza la observación cuando ya no sea necesaria.

```
observer.disconnect();
```

#### **Tipos de Mutaciones que Puede Detectar**

- **`childList`**: Cambios en nodos hijos (añadir/eliminar).
- **`attributes`**: Cambios en los atributos de un elemento.
- **`characterData`**: Cambios en el texto de un nodo.
- **`subtree`**: Cambios en descendientes (combinable con las anteriores).

### **Ejemplos Genéricos**

---

#### **1. Observar Añadidos y Eliminados en un Elemento**

```
const observer = new MutationObserver((mutationsList) => {
    mutationsList.forEach(mutation => {
        if (mutation.type === 'childList') {
            console.log('Hijos añadidos o eliminados:', mutation);
        }
    });
});

observer.observe(document.querySelector('#contenedor'), { childList: true });
```

**Uso esperado:**

- Añadir o eliminar nodos en `#contenedor` activará el callback.
- Ejemplo: Si se añade un nuevo `<div>` dentro de `#contenedor`, el observador lo detectará.

#### 2. Observar Cambios en Atributos

```
const observer = new MutationObserver((mutationsList) => {
    mutationsList.forEach(mutation => {
        if (mutation.type === 'attributes') {
            console.log(`Atributo cambiado: ${mutation.attributeName}`);
        }
    });
});

observer.observe(document.querySelector('#boton'), { attributes: true });
```

**Uso esperado:**

- Detectar cambios en atributos como `class`, `style` o `id`.
- Ejemplo: Si se cambia la clase de `#boton`, el observador lo registrará.


#### 3. Observar Cambios en el Texto de un Elemento

```
const observer = new MutationObserver((mutationsList) => {
    mutationsList.forEach(mutation => {
        if (mutation.type === 'characterData') {
            console.log('Texto modificado:', mutation.target.data);
        }
    });
});

observer.observe(document.querySelector('#texto'), { characterData: true, subtree: true });
```

**Uso esperado:**

- Detectar cambios en el contenido de texto de un elemento o sus descendientes.
- Ejemplo: Si se edita el contenido de `#texto` en tiempo real, el observador lo capturará.

#### 4. Observar Todo el Documento

```
const observer = new MutationObserver((mutationsList) => {
    console.log('Mutaciones en el documento:', mutationsList);
});

observer.observe(document.body, { childList: true, subtree: true });
```

**Uso esperado:**

- Detectar cualquier cambio en nodos hijos o descendientes en todo el documento.
- Ejemplo: Si se crea o elimina un nodo en cualquier parte del DOM, se notificará.


#### 5. Observar Cambios en Varias Secciones

```
const secciones = document.querySelectorAll('.seccion');

secciones.forEach(seccion => {
    observer.observe(seccion, { childList: true, subtree: true });
});
```

**Uso esperado:**

- Detectar cambios en múltiples elementos seleccionados por la clase `.seccion`.
- Ejemplo: Si se añade un nodo en cualquier sección, el observador lo registrará.

#### 6. Desactivar el Observador tras una Condición

```
const observer = new MutationObserver((mutationsList, observer) => {
    mutationsList.forEach(mutation => {
        if (mutation.type === 'childList' && mutation.addedNodes.length > 0) {
            console.log('Nodo añadido:', mutation.addedNodes[0]);
            observer.disconnect(); // Detener observación
        }
    });
});

observer.observe(document.querySelector('#contenedor'), { childList: true });
```

**Uso esperado:**

- Detectar el primer cambio y luego detener el observador.
- Ejemplo: Si se añade un nodo, se registra y se detiene la observación.

### **Resultados Esperados y Escenarios**

|**Acción en el DOM**|**Configuración Observada**|**Resultado**|
|---|---|---|
|Añadir un nodo hijo|`childList: true`|Callback detecta el nodo añadido.|
|Eliminar un nodo hijo|`childList: true`|Callback detecta el nodo eliminado.|
|Cambiar un atributo (`class`)|`attributes: true`|Callback detecta el cambio.|
|Modificar texto|`characterData: true`|Callback detecta texto modificado.|
|Cambiar en descendientes|`subtree: true`|Detecta cambios recursivamente.|

### **Consejos y Buenas Prácticas**

1. **Optimizar el Observador:**
    
    - No observes nodos demasiado amplios como `body` si no es necesario.
    - Usa condiciones en el callback para filtrar mutaciones relevantes.
2. **Desconectar Cuando No Sea Necesario:**
    
    - Libera recursos desconectando el observador con `observer.disconnect()`.
3. **Evitar Ciclos Inesperados:**
    
    - Si tu callback modifica el DOM, puede desencadenar el observador nuevamente. Usa banderas o desconecta temporalmente el observador.
4. **Combinar con Otras APIs:**
    
    - Puedes usar `IntersectionObserver` o eventos DOM como complementos según la necesidad.



[[funciones_js_jq]] [[Funciones_js]]