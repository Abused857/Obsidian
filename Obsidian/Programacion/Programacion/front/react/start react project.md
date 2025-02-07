
en la ruta abrimos terminal:

npm create vite@latest

vanilla / javascript

cd a la app e instalamos plugin de vite npm install @vitejs/plugin-react -E

instalamos react en el proyecto:

npm install react react-dom -E

ahora la configuración de vite:

creamos en nuestro root un archivo:

vite.config.js

y dentro añadimos :

```
import { defineConfig } from "vite"

import react from '@vitejs/plugin-react'

  

export default defineConfig({

    plugins: [react()]

})
```


# Punto de entrada de la app

el archivo index.html lo que hace es cargar el main.js en esta linea 
 <script type="module" src="/src/main.js"></script>

vamos al main js, eliminamos todo y añadimos:

```
import { createRoot } from "react-dom/client"

  

const root = createRoot(document.getElementById('app'))

  

root.render (<h1>Hello, World!</h1>)
```


cambiamos el main.js a -----> main.jsx

utilizamos npm run dev para ver que esta todo ok.

# Instalamos el linter

npm install standard -D

ahora en package json despues de dependencies : 

```
 "eslintConfig": {
  "extends": "./node_modules/standard/eslintrc.json"
}

```

 para inicializarlo npx eslint --init

ahora pasamos a crear nuestro App.jsx dentro de src


```
export function App(){

    return(

        <h1>App de pokemon</h1>

    )

}
```

y lo renderizamos en nuestro main.js para empezar a trabajar ya 

```
import { createRoot } from "react-dom/client"

import { App } from "./App.jsx"

const root = createRoot(document.getElementById('app'))

  

root.render (<App></App>)
```

[[Front]]