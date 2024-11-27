
Court costs

```
<?php

/**

 * Archivo: CourtCostsTable.php

 * Descripción: Este archivo contiene la definición de la clase `CourtCostsTable`

 * que maneja la interacción con la tabla `court_costs`.

 * Autor: Germán <german.martinez@incentro.com>

 * Fecha: 26/11/2024

 * PHP Version: 7.4

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

namespace App\Model\Table;

  

use Cake\ORM\Table;

  

/**

 * Clase CourtCostsTable

 *

 * Esta clase maneja la interacción entre la aplicación y la tabla `court_costs`.

 * Define las relaciones y configuraciones necesarias para

 * el manejo de los costos judiciales.

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

class CourtCostsTable extends Table

{

       /**

     * Inicializa la tabla con las asociaciones y configuraciones necesarias.

     *

     * Esta función configura la tabla `court_costs`,

     *  establece las relaciones con otras tablas y

     * define el nombre de la tabla y la clave primaria.

     *

     * @param array $config Configuración de la tabla.

     *

     * @return void

     */

    public function initialize(array $config): void

    {

        parent::initialize($config);

  

        $this->setTable('court_costs');

        $this->setPrimaryKey('id');

  

        $this->belongsTo(

            'Records',

            [

            'foreignKey' => 'record_id',

            'joinType' => 'INNER',

            ]

        );

  

        $this->hasMany(

            'Income',

            [

            'foreignKey' => 'court_costs_id',

            ]

        );

  

        $this->hasOne(

            'LawyerCourtCosts',

            [

            'foreignKey' => 'court_costs_id',

            'cascadeCallbacks' => true,

            'dependent' => true,

            ]

        );

  

        $this->hasOne(

            'AttorneyCourtCosts',

            [

            'foreignKey' => 'court_costs_id',

            'cascadeCallbacks' => true,

            'dependent' => true,

            ]

        );

    }

}
```


LawyerCourtCosts

```
<?php

/**

 * Archivo: LawyerCourtCostsTable.php

 * Descripción: Este archivo contiene

 * la definición de la clase `LawyerCourtCostsTable`

 * que maneja la interacción con la tabla `lawyer_court_costs`.

 * Autor: Germán <german.martinez@incentro.com>

 * Fecha: 26/11/2024

 * PHP Version: 7.4

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

namespace App\Model\Table;

  

use Cake\ORM\Table;

  

/**

 * Clase LawyerCourtCostsTable

 *

 * Esta clase maneja la interacción entre la

 * aplicación y la tabla `lawyer_court_costs`.

 * Define las relaciones necesarias para gestionar los costos

 * judiciales asociados a los abogados.

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

  

class LawyerCourtCostsTable extends Table

{

     /**

     * Inicializa la tabla con las asociaciones y configuraciones necesarias.

     *

     * Esta función configura la tabla `lawyer_court_costs`

     * y establece las relaciones

     * necesarias con otras tablas.

     *

     * @param array $config Configuración de la tabla.

     *

     * @return void

     */

    public function initialize(array $config): void

    {

        parent::initialize($config);

  

        $this->setTable('lawyer_court_costs');

        $this->setPrimaryKey('court_costs_id');

  

        $this->belongsTo(

            'CourtCosts',

            [

            'foreignKey' => 'court_costs_id',

            ]

        );

    }

}
```


AttorneyCourtCosts

```
<?php

/**

 * Copyright ArriagaAsociados.

 *

 * @author   José Luis Itoiz

 */

namespace Records\Model\Table;

  

use Cake\ORM\RulesChecker;

use Cake\ORM\Table;

use Cake\Validation\Validator;

use \ArrayObject;

  

/**

 * Represents the "attorneys" DB table.

 */

class AttorneysTable extends Table

{

    /**

     * {@inheritDoc}

     */

    public function initialize(array $config)

    {

        $this->addBehavior('Watchdog.Events');

        $this->addBehavior('Timestamp');

  

        $this->belongsToMany('Cities', [

            'className' => 'Cities',

            'through' => 'Records.AttorneysCities'

        ]);

  

        $this->hasMany('AttorneysCities', [

            'className' => 'Records.AttorneysCities',

            'foreignKey' => 'attorney_id'

        ]);

    }

  

    /**

     * Default validation rules.

     *

     * @param \Cake\Validation\Validator $validator Validator object

     * @return \Cake\Validation\Validator

     */

    public function validationDefault(Validator $validator)

    {

        $validator->notEmpty('name', 'Attorney name is required.');

        $validator->notEmpty('lastname', 'Attorney name is required.');

  

        return $validator;

    }

  

    /**

     * {@inheritDoc}

     */

    public function buildRules(RulesChecker $rules)

    {

        $rules

            ->add($rules->isUnique(['name', 'lastname'], 'Procurador ya dado de alta.'))

            ->add(function ($entity, $options) {

                if (!$entity->get('signature')) {

                    return true;

                }

  

                $conditions = [

                    'signature' => $entity->get('signature')

                ];

  

                if (!$entity->isNew()) {

                    $conditions['id <>'] = $entity->get('id');

                }

  

                return (int)$this->find()

                    ->hydrate(false)

                    ->where($conditions)

                    ->count() === 0;

            }, 'isUnique', [

                'errorField' => 'signature',

                'message' => 'La firma ya existe.',

            ])

  

            ->add(function ($entity, $options) {

                if (!$entity->get('bar_association') || !$entity->get('license_number')) {

                    return true;

                }

  

                $conditions = [

                    'bar_association' => $entity->get('bar_association'),

                    'license_number' => $entity->get('license_number'),

                ];

  

                if (!$entity->isNew()) {

                    $conditions['id <>'] = $entity->get('id');

                }

  

                return (int)$this->find()

                    ->hydrate(false)

                    ->where($conditions)

                    ->count() === 0;

            }, 'isUnique', [

                'errorField' => 'signature',

                'message' => 'Esa licencia de ese colegio ya existe.',

            ]);

  

            return $rules;

    }

}
```

Income

```
<?php

/**

 * Archivo: IncomeTable.php

 * Descripción: Este archivo contiene la definición de la clase `IncomeTable`

 * que maneja la interacción con la tabla `income`.

 * Autor: Germán <german.martinez@incentro.com>

 * Fecha: 26/11/2024

 * PHP Version: 7.4

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

  

namespace App\Model\Table;

  

use Cake\ORM\Table;

use Cake\Validation\Validator;

  

/**

 * Clase IncomeTable

 *

 * Esta clase maneja la interacción entre la aplicación y la tabla `income`.

 * Define las relaciones y configuraciones necesarias para el manejo de los ingresos.

 *

 * @category Model

 * @package  App.Model.Table

 * @author   Germán <german.martinez@incentro.com>

 * @license  https://opensource.org/licenses/MIT MIT

 * @link     https://example.com

 */

class IncomeTable extends Table

{

     /**

     * Inicializa la tabla con las asociaciones y configuraciones necesarias.

     *

     * Esta función configura la tabla `income`,

     * establece las relaciones con otras tablas

     * y define el nombre de la tabla y la clave primaria.

     *

     * @param array $config Configuración de la tabla.

     *

     * @return void

     */

    public function initialize(array $config): void

    {

        parent::initialize($config);

  

        $this->setTable('income');

        $this->setPrimaryKey('id');

  

        $this->belongsTo(

            'Records',

            [

            'foreignKey' => 'record_id',

            'joinType' => 'INNER',

            ]

        );

    }

}
```

