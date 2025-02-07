

```
/**

     * AJAX Controller action.

     *

     * @param int $recordId Record ID

     * @param int $info Info to load

     * @return \Cake\Network\Response|void JSON RESPONSE

     */

    public function getRecordInfo($recordId, $info)

    {

        $this->loadModel('Records.Records');

        $this->loadModel('Users.Users');

        $this->loadModel('LawyerCourtCosts');

        $this->loadModel('AttorneyCourtCosts');

        $info = str_replace('-', '_', $info);

        $finderName = Inflector::camelize($info);

        $finderName = lcfirst($finderName);

  

        $record = $this->Records->find($finderName, [

            'for' => $recordId,

            'contain' => ['CourtCosts', 'Income']

        ]);

        $usersList = usersList()->toArray();

        if (!empty($record->court_costs)) {

            $courtCostsIds = array_map(function($courtCost) {

                return $courtCost->id;

            }, $record->court_costs);

            $lawyerCourtCosts = $this->LawyerCourtCosts->find('all', [

                'conditions' => ['LawyerCourtCosts.court_costs_id IN' => $courtCostsIds]

            ])->toArray();

            $attorneyCourtCosts = $this->AttorneyCourtCosts->find('all', [

                'conditions' => ['AttorneyCourtCosts.court_costs_id IN' => $courtCostsIds]

            ])->toArray();

            foreach ($record->court_costs as $courtCost) {

                $relatedLawyerCourtCosts = array_filter($lawyerCourtCosts, function($lawyerCourtCost) use ($courtCost) {

                    return $lawyerCourtCost->court_costs_id == $courtCost->id;

                });

                $relatedAttorneyCourtCosts = array_filter($attorneyCourtCosts, function($attorneyCourtCost) use ($courtCost) {

                    return $attorneyCourtCost->court_costs_id == $courtCost->id;

                });

                if (!empty($relatedLawyerCourtCosts)) {

                    $courtCost->lawyer_court_costs = $relatedLawyerCourtCosts;

                } else {

                    $courtCost->lawyer_court_costs = null;

                }

                if (!empty($relatedAttorneyCourtCosts)) {

                    $courtCost->attorney_court_costs = $relatedAttorneyCourtCosts;

                } else {

                    $courtCost->attorney_court_costs = null;

                }

            }

        }

  

        $this->set(compact('record', 'usersList'));

        $this->render("/Element/record_edit/{$info}");

    }
```

[[Programación]]