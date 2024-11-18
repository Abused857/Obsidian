
# Implementación de Merge Sort en PHP y busqueda binaria

Ordenacion, busqueda




```php
<?php

namespace App\Services;

class MergeSortService
{
    /**
     * Método para ordenar un array usando el algoritmo Merge Sort.
     *
     * @param array $array El array a ordenar.
     * @return array Array ordenado.
     */
    public function mergeSort(array $array)
    {
        $length = count($array);

        if ($length <= 1) {
            return $array;
        }

        $mid = (int) ($length / 2);
        $left = array_slice($array, 0, $mid);
        $right = array_slice($array, $mid);

        $left = $this->mergeSort($left);
        $right = $this->mergeSort($right);

        return $this->merge($left, $right);
    }

    /**
     * Método auxiliar para combinar dos arrays ordenados.
     *
     * @param array $left El primer array ordenado.
     * @param array $right El segundo array ordenado.
     * @return array Array combinado y ordenado.
     */
    private function merge(array $left, array $right)
    {
        $result = [];
        $leftIndex = 0;
        $rightIndex = 0;

        while ($leftIndex < count($left) && $rightIndex < count($right)) {
            if ($left[$leftIndex] < $right[$rightIndex]) {
                $result[] = $left[$leftIndex];
                $leftIndex++;
            } else {
                $result[] = $right[$rightIndex];
                $rightIndex++;
            }
        }

        while ($leftIndex < count($left)) {
            $result[] = $left[$leftIndex];
            $leftIndex++;
        }

        while ($rightIndex < count($right)) {
            $result[] = $right[$rightIndex];
            $rightIndex++;
        }

        return $result;
    }
}

```


**Palabras clave**: PHP, Merge Sort, algoritmo de ordenamiento, eficiencia, Obsidian.

# Implementación de Búsqueda Binaria en PHP

```php
<?php

namespace App\Services;

class BinarySearchService
{
    /**
     * Método para realizar una búsqueda binaria en un array ordenado.
     *
     * @param array $array El array ordenado donde buscar.
     * @param int $target El elemento que se busca.
     * @return int|bool Índice del elemento si se encuentra, o false si no está presente.
     */
    public function binarySearch(array $array, int $target)
    {
        $left = 0;
        $right = count($array) - 1;

        while ($left <= $right) {
            $mid = (int) (($left + $right) / 2);

            if ($array[$mid] == $target) {
                return $mid; // Elemento encontrado, devolver el índice
            } elseif ($array[$mid] < $target) {
                $left = $mid + 1; // Descartar la mitad izquierda
            } else {
                $right = $mid - 1; // Descartar la mitad derecha
            }
        }

        return false; // Elemento no encontrado
    }
}
```

**Palabras clave**: PHP, búsqueda binaria, algoritmo de búsqueda, eficiencia, markdown, Obsidian. 

[[PHP]] 
