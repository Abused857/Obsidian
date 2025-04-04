
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


# FWB Process Check: Comparing SPH Arrays

### Context:
The process checks if the **FWB** already exists in the database with the same **CNE**, **SPH**, and **Flight Number**. If it does, the new **FWB** is skipped. If the **SPH** values are different, the process continues.

### Code Snippet:

```php
if ($fwbIncoming) {
    $sphNew = $fwbNew->SPH;
    $sphIncoming = $fwbIncoming->SPH;

    sort($sphNew);       
    sort($sphIncoming);  
    $cloudLogger->info("sphNew: " . json_encode($sphNew));
    $cloudLogger->info("sphIncoming: " . json_encode($sphIncoming));

    if ($sphNew == $sphIncoming) {
        $cloudLogger->info("ProcessFWB: FWB already exists in DB with the same CNE, SPH and Flight number. Stored FWB ID IN DB: " . $fwbIncoming->fwb_id . "\n" .
        "New FWB details: " . json_encode($fwbNew) . "\n" .
        "Stored in DB FWB details: " . json_encode($fwbIncoming));
        return;
    } else {
        $cloudLogger->info("ProcessFWB: SPH is different for new FWB and stored FWB. Continuing processing.");
    }
}

```


### Explanation:

1. **Sorting the Arrays**:
    
    - The function `sort()` is used to sort the elements of the arrays `$sphNew` and `$sphIncoming`.
        
    - This is necessary because **array comparison** in PHP checks both **the values** and **the order** of the elements. Sorting the arrays ensures that the comparison only looks at the **values**, regardless of the order in which they were provided.
        
2. **Example Arrays**:
    
    - **Example 1**: `["BUP", "DDX", "SPX"]`
        
    - **Example 2**: `["ECC", "EAP", "NSC", "CAO", "RBI"]`
        
    
    After sorting, both arrays will be in a predictable order, so the comparison becomes reliable:
```php
sort(["BUP", "DDX", "SPX"]); // Sorted -> ["BUP", "DDX", "SPX"]
sort(["ECC", "EAP", "NSC", "CAO", "RBI"]); // Sorted -> ["CAO", "EAP", "ECC", "NSC", "RBI"]
```
**Why Sorting Works**:

- **Sorting** ensures that the order of elements doesn't matter for the comparison.
    
- If the arrays have the same elements but in different orders, sorting both arrays makes sure they are compared correctly.
    

For example:

```php
// Before sorting
$sphNew = ["BUP", "DDX", "SPX"];
$sphIncoming = ["SPX", "DDX", "BUP"];

// After sorting
sort($sphNew); // ["BUP", "DDX", "SPX"]
sort($sphIncoming); // ["BUP", "DDX", "SPX"]
```
1. his way, the condition `$sphNew == $sphIncoming` will return `true` because both arrays are identical in terms of values, even though the original order was different.
    
2. **Outcome**:
    
    - If both the **SPH** arrays are identical after sorting, the process logs that the **FWB** already exists in the database with the same **CNE**, **SPH**, and **Flight Number**.
        
    - If they differ, the process continues with further handling.
        

### Key Takeaway:

- The use of `sort()` ensures that the comparison of the **SPH** values is independent of the order of elements in the arrays. This is crucial for correct comparison when the order of the elements doesn't matter, only the values do.
[[PHP]] 
