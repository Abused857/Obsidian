# Documentación: Asociación de `LawyerCourtCosts` y `AttorneyCourtCosts` a `CourtCosts`

```php
if (!empty($record->court_costs)) { 
    # Obtener los IDs de los CourtCosts asociados al record.
    $courtCostsIds = array_map(function($courtCost) {
        return $courtCost->id;
    }, $record->court_costs);

    ## LawyerCourtCosts
    # Buscar todas las LawyerCourtCosts asociadas a los IDs obtenidos.
    $lawyerCourtCosts = $this->LawyerCourtCosts->find('all', [
        'conditions' => ['LawyerCourtCosts.court_costs_id IN' => $courtCostsIds]
    ])->toArray();

    ## AttorneyCourtCosts
    # Buscar todas las AttorneyCourtCosts asociadas a los IDs obtenidos.
    $attorneyCourtCosts = $this->AttorneyCourtCosts->find('all', [
        'conditions' => ['AttorneyCourtCosts.court_costs_id IN' => $courtCostsIds]
    ])->toArray();

    # Iterar sobre cada CourtCost y asociar los datos relevantes de Lawyer y Attorney Court Costs.
    foreach ($record->court_costs as $courtCost) {
        ## Asociación de LawyerCourtCosts
        # Filtrar LawyerCourtCosts que coincidan con el ID del CourtCost actual.
        $relatedLawyerCourtCosts = array_filter($lawyerCourtCosts, function($lawyerCourtCost) use ($courtCost) {
            return $lawyerCourtCost->court_costs_id == $courtCost->id;
        });

        ## Asociación de AttorneyCourtCosts
        # Filtrar AttorneyCourtCosts que coincidan con el ID del CourtCost actual.
        $relatedAttorneyCourtCosts = array_filter($attorneyCourtCosts, function($attorneyCourtCost) use ($courtCost) {
            return $attorneyCourtCost->court_costs_id == $courtCost->id;
        });

        ## Asignar LawyerCourtCosts al CourtCost actual
        if (!empty($relatedLawyerCourtCosts)) {
            $courtCost->lawyer_court_costs = $relatedLawyerCourtCosts;
        } else {
            $courtCost->lawyer_court_costs = null; # Si no hay registros, asignar null.
        }

        ## Asignar AttorneyCourtCosts al CourtCost actual
        if (!empty($relatedAttorneyCourtCosts)) {
            $courtCost->attorney_court_costs = $relatedAttorneyCourtCosts;
        } else {
            $courtCost->attorney_court_costs = null; # Si no hay registros, asignar null.
        }
    }
}

```

[[Programación]]

 