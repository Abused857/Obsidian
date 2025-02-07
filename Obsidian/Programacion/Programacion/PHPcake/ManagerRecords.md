
```php


<?php

/**
 * Copyright ArriagaAsociados.
 *
 * @author Christopher Castro <chris@quickapps.es>
 */

namespace Records\Controller;

use App\ExternalSystems\Api\Minerva;
use App\Filesystem\FilesystemFactory;
use App\Filesystem\LocalFile;
use App\Filesystem\Util;
use Cake\Core\Plugin;
use Cake\Filesystem\Folder;
use Cake\I18n\Date;
use Cake\Log\Log;
use Cake\Network\Exception\BadRequestException;
use Cake\Network\Exception\ForbiddenException;
use Cake\Network\Exception\InternalErrorException;
use Cake\Network\Exception\NotFoundException;
use Cake\Utility\Hash;
use Cake\Utility\Security;
use Cake\View\CellTrait;
use Documents\Lib\Mpdf;
use Queues\JobRegistry;
use Records\Controller\AppController;
use Records\Lib\NotificadosOperators\NotificadosOperators;

/**
 * Controller.
 */
class ManageController extends AppController
{

    use CellTrait;

    /**
     * {@inheritDoc}
     */
    public $helpers = [
        'Users.UserLogs',
        'Records.Notes',
        'Records.Records',
    ];

    /**
     * Records paginate
     *
     * @var array
     */
    public $paginate = [
        'limit' => 3,
    ];

    /**
     * Búsqueda rápida de expedientes por dni o número de expediente.
     *
     * @return \Cake\Network\Response|void
     */
    public function index()
    {
        $this->loadModel('Records.Records');
        $users = usersList()->toArray();
        $tags = $this->Records->Tags->find('all')
            ->cache('record_index', 'record_tags')
            ->select(['id', 'name', 'color', 'ordering'])
            ->order(['ordering' => 'ASC'])
            ->formatResults(
                function ($results) {
                    return $results->map(
                        function ($tag) {
                            return [
                                'id' => $tag->get('id'),
                                'name' => $tag->get('name'),
                                'color' => $tag->get('color'),
                                'ordering' => $tag->get('ordering'),
                            ];
                        }
                    );
                }
            );

        if (!empty($this->request->query['filter'])) {
            if (user()->isAllowed('#clients:censorship')) {
                $this->Records->behaviors()->get('SoftDelete')->config('filterFlag', null, false);
            }

            $this->Records->Clients->eav(false);
            $records = $this->Records
                ->find()
                ->contain(
                    [
                        'Clients' => function ($q) {
                            return $q->select(['Clients.id', 'Clients.name', 'Clients.lastname', 'Clients.confirmed', 'Clients.email', 'Clients.cif', 'Clients.birth_date', 'Clients.deceased']);
                        },
                        'Company' => function ($q) {
                            return $q->select(['id', 'name']);
                        },
                        'MyBookmark',
                        'Reminds' => function ($q) {
                            return $q
                                ->where(['Reminds.user_id' => uid()])
                                ->andWhere(['Reminds.status IN' => ['1', '2']])
                                ->limit(1);
                        },
                        'Tags',
                        'Labels',
                        'MyHashtags',
                        'InformacionDimension'
                    ]
                )
                ->select(
                    [
                        'Company.id',
                        'Company.name',
                        'MyBookmark.id',
                        'Records.id',
                        'Records.company_id',
                        'Records.nig',
                        'Records.nig2',
                        'Records.producto',
                        'Records.importe',
                        'Records.banco',
                        'Records.type',
                        'Records.status',
                        'Records.created',
                        'Records.finished',
                        'Records.suceso',
                        'Records.pro-autos',
                        'Records.pro_jpi',
                        'Records.ciudadjpi',
                        'Records.track',
                        'Records.pro_fecha_vistas',
                        'Records.pro_fecha_juicio',
                        'Records.pro_fecha_audiencia',
                        'Records.abogado_vistas',
                        'Records.abogado_juicio',
                        'Records.abogado_audiencia',
                        'Records.procurador-contrario',
                        'Records.abogado-contrario',
                        'Records.pro_decreto_admision_tramite_fecha',
                        'Records.pro_fecha_sentencia_primera_instancia',
                        'Records.poder_notarial',
                        'Records.reviewed',
                        'Records.department_risk',
                        'InformacionDimension.id'
                    ]
                );

            $records = $this->Records->search($this->request->query['filter'], $records)->group('Records.id');
            $customOrder = mb_strpos(mb_strtolower($this->request->query['filter']), 'orden:') !== false;

            if (!isset($this->request->query['get-lucky'])) {
                if (!isset($customOrder) || !$customOrder) {
                    $records->order(['Records.id' => 'DESC']);
                }
            }

            if (isset($this->request->query['page'])) {
                $records->counter(
                    function ($query) {
                        return $query
                            ->select(['Records.id'], true)
                            ->removeJoin('Company')
                            ->removeJoin('MyBookmark')
                            ->count();
                    }
                );
                $this->paginate['limit'] = 5;
                $records = $this->paginate($records);
            } else {
                $records = $records->limit(6);
            }

            if (isset($this->request->query['get-lucky'])) {
                $luckyOne = $records
                    ->select(['Records.id'], true)
                    ->limit(1)
                    ->first();

                if ($luckyOne) {
                    return $this->redirect(
                        [
                            'plugin' => 'Records',
                            'controller' => 'Manage',
                            'action' => 'edit',
                            $luckyOne->get('id')
                        ]
                    );
                }
            }

            $records = $records->toArray();
            $this->set(compact('records', 'tags', 'users'));
        } else {
            $this->set(compact('tags', 'users'));
        }
    }

    /**
     * Controller action.
     *
     * @return \Cake\Network\Response|void
     */
    public function fastSearch()
    {
        $this->loadModel('Records.Records');

        if (!empty($this->request->query['filter'])) {
            if (user()->isAllowed('#clients:censorship')) {
                $this->Records->behaviors()->get('SoftDelete')->config('filterFlag', null, false);
            }
            $users = usersList()->toArray();
            $tags = $this->Records->Tags->find('all')
                ->cache('record_index', 'record_tags')
                ->select(['id', 'name', 'color', 'ordering'])
                ->order(['ordering' => 'ASC'])
                ->formatResults(
                    function ($results) {
                        return $results->map(
                            function ($tag) {
                                return [
                                    'id' => $tag->get('id'),
                                    'name' => $tag->get('name'),
                                    'color' => $tag->get('color'),
                                    'ordering' => $tag->get('ordering'),
                                ];
                            }
                        );
                    }
                );
            $this->Records->Clients->eav(false);
            $filter = $this->request->query['filter'];
            if (is_numeric(substr($filter, -1))) {
                $records = $this->Records->find('all', ['eav' => false])
                    ->select(
                        [
                            'Company.id',
                            'Company.name',
                            'MyBookmark.id',
                            'Records.id',
                            'Records.company_id',
                            'Records.nig',
                            'Records.producto',
                            'Records.importe',
                            'Records.banco',
                            'Records.type',
                            'Records.status',
                            'Records.created',
                            'Records.finished',
                            'Records.pro_jpi',
                            'Records.ciudadjpi',
                            'Records.pro_fecha_vistas',
                            'Records.pro_fecha_juicio',
                            'Records.pro_fecha_audiencia',
                            'Records.abogado_vistas',
                            'Records.abogado_juicio',
                            'Records.abogado_audiencia',
                            'Records.pro_decreto_admision_tramite_fecha',
                            'Records.pro_fecha_sentencia_primera_instancia',
                            'Records.poder_notarial',
                            'Records.reviewed',
                            'Records.department_risk',
                            'InformacionDimension.id',
                            'MandamientosDimension.id',
                            'ProcedimientoDimension.id',
                            'RevolvingDimension.id',
                            'PaternidadDimension.id',
                            'InterestSettlementDimension.id'
                        ]
                    )
                    ->contain(
                        [
                            'Clients' => function ($q) {
                                return $q->select(['Clients.id', 'Clients.name', 'Clients.lastname', 'Clients.confirmed', 'Clients.email', 'Clients.cif', 'Clients.birth_date', 'Clients.deceased']);
                            },
                            'Company' => function ($q) {
                                return $q->select(['id', 'name']);
                            },
                            'MyBookmark',
                            'Reminds' => function ($q) {
                                return $q
                                    ->where(['Reminds.user_id' => uid()])
                                    ->andWhere(['Reminds.status IN' => ['1', '2']])
                                    ->limit(1);
                            },
                            'Tags',
                            'Labels',
                            'MyHashtags',
                            'InformacionDimension',
                            'MandamientosDimension',
                            'ProcedimientoDimension',
                            'RevolvingDimension',
                            'PaternidadDimension',
                            'InterestSettlementDimension'
                        ]
                    )
                    ->where(['Records.crm_id' => $filter]);
            } else {
                $records = $this->Records->find('all', ['eav' => false])
                    ->select(
                        [
                            'Company.id',
                            'Company.name',
                            'MyBookmark.id',
                            'Records.id',
                            'Records.company_id',
                            'Records.nig',
                            'Records.producto',
                            'Records.importe',
                            'Records.banco',
                            'Records.type',
                            'Records.status',
                            'Records.created',
                            'Records.finished',
                            'Records.pro_jpi',
                            'Records.ciudadjpi',
                            'Records.pro_fecha_vistas',
                            'Records.pro_fecha_juicio',
                            'Records.pro_fecha_audiencia',
                            'Records.abogado_vistas',
                            'Records.abogado_juicio',
                            'Records.abogado_audiencia',
                            'Records.pro_decreto_admision_tramite_fecha',
                            'Records.pro_fecha_sentencia_primera_instancia',
                            'Records.poder_notarial',
                            'Records.reviewed',
                            'Records.department_risk',
                            'InformacionDimension.id',
                            'MandamientosDimension.id',
                            'ProcedimientoDimension.id',
                            'RevolvingDimension.id',
                            'PaternidadDimension.id',
                            'InterestSettlementDimension.id'
                        ]
                    )
                    ->join(
                        [
                            'RecordClient' => [
                                'table' => 'clients_records',
                                'type' => 'INNER',
                                'conditions' => ['RecordClient.record_id = Records.id']
                            ],
                            'Clients' => [
                                'table' => 'clients',
                                'type' => 'INNER',
                                'conditions' => ['RecordClient.client_id = Clients.id']
                            ]
                        ]
                    )
                    ->contain(
                        [
                            'Clients' => function ($q) {
                                return $q->select(['Clients.id', 'Clients.name', 'Clients.lastname', 'Clients.confirmed', 'Clients.email', 'Clients.cif', 'Clients.birth_date', 'Clients.deceased']);
                            },
                            'Company' => function ($q) {
                                return $q->select(['id', 'name']);
                            },
                            'MyBookmark',
                            'Reminds' => function ($q) {
                                return $q
                                    ->where(['Reminds.user_id' => uid()])
                                    ->andWhere(['Reminds.status IN' => ['1', '2']])
                                    ->limit(1);
                            },
                            'Tags',
                            'Labels',
                            'MyHashtags',
                            'InformacionDimension',
                            'MandamientosDimension',
                            'ProcedimientoDimension',
                            'RevolvingDimension',
                            'PaternidadDimension',
                            'InterestSettlementDimension'
                        ]
                    )
                    ->where(['Clients.dni' => $filter]);
            }

            if (isset($this->request->query['page'])) {
                $records->counter(
                    function ($query) {
                        return $query
                            ->select(['Records.id'], true)
                            ->count();
                    }
                );
                $this->paginate['limit'] = 5;
                $records = $this->paginate($records);
            } else {
                $records = $records->limit(6);
            }

            $records = $records->toArray();
            $this->set(compact('records', 'tags', 'users'));
        }
    }

    /**
     * Shows children records of the given root record.
     *
     * @param  int $id Record ID
     * @return \Cake\Network\Response|void
     */
    public function children($id)
    {
        $this->loadModel('Records.Records');
        $record = $this->Records->get(
            $id,
            [
                'eav' => false,
                'conditions' => ['Records.type' => 'COLECTIVA']
            ]
        );
        $records = $this->Records
            ->find()
            ->where(['parent_id' => $id]);
        if (!empty($this->request->query['filter'])) {
            $this->Records->search($this->request->query['filter'], $records);
        }
        $records = $this->paginate($records);
        $this->set(compact('records', 'record'));
    }

    /**
     * Controller action.
     *
     * @param  int $id Record ID
     * @return \Cake\Network\Response|void
     */
    public function view($id)
    {
        return $this->redirect(
            [
                'plugin' => 'Records',
                'controller' => 'Manage',
                'action' => 'edit',
                $id,
            ]
        );
    }

    /**
     * Controller action.
     *
     * @return \Cake\Network\Response|void
     */
    public function add()
    {
        $this->loadModel('Records.Records');
        $record = $this->Records->newEntity();
        if ($this->request->is('post')) {
            $record = $this->Records->patchEntity($record, $this->request->data());
            if ($this->Records->save($record)) {
                $this->Flash->success('Expediente creado correctamente.');
                $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id'), '#' => 'clients']);
            } else {
                $this->Flash->error(
                    'El expediente no pudo ser registrado, compruebe los datos.',
                    [
                        'params' => ['errors' => $record->errors()]
                    ]
                );
            }
        }
        $this->set(compact('client', 'record'));
    }

    /**
     * Controller action.
     *
     * @param  int $id Record ID
     * @return \Cake\Network\Response|void
     */
    public function edit($id)
    {
        $this->loadModel('Records.Records');
        $tags = $this->Records->Tags->find('all')
            ->cache('record_edit', 'record_tags')
            ->select(['id', 'name', 'color', 'ordering'])
            ->order(['ordering' => 'ASC'])
            ->formatResults(
                function ($results) {
                    return $results->combine(
                        'id',
                        function ($tag) {
                            return sprintf('<span class="label label-success" style="background-color:#%s" data-ordering="%s">%s</span>', $tag->get('color'), $tag->get('ordering'), $tag->get('name'));
                        }
                    );
                }
            );

        if (user()->isAllowed('#clients:censorship')) {
            $this->Records->behaviors()->get('SoftDelete')->config('filterFlag', null, false);
        }
        if ($this->request->is('get')) {
            $record = $this->Records->get(
                $id,
                [
                    'eav' => true,
                    'fields' => $this->_calculatePersistableFields(
                        [
                            'MyBookmark.id'
                        ]
                    ),
                    'contain' => [
                        'Company',
                        'Clients',
                        'MyBookmark',
                        'MyHashtags',
                        'Tags' => function ($q) {
                            return $q->order(['ordering' => 'DESC']);
                        },
                        'Labels',
                        'MyReminds' => function ($q) {
                            return $q->where(['status IN' => [1, 2]])->limit(1);
                        },
                        'Appointments' => function ($q) {
                            return $q->order(['created' => 'DESC'])->limit(1);
                        },
                        'InformacionDimension',
                        'MandamientosDimension',
                        'ProcedimientoDimension',
                        'RevolvingDimension',
                        'PlusvaliaDimension',
                        'PaternidadDimension',
                        'InterestSettlementDimension',
                        'OperacionesDemandaPopular',
                        'CartelVehicles'
                    ]
                ]
            );
        } else {
            $record = $this->Records->get(
                $id,
                [
                    'eav' => true,
                    'fields' => $this->_calculatePersistableFields(
                        [
                            'MyBookmark.id',
                            'InformacionDimension.id',
                            'MandamientosDimension.id',
                            'ProcedimientoDimension.id',
                            'RevolvingDimension.id',
                            'PaternidadDimension.id',
                            'InterestSettlementDimension.id',
                            'PlusvaliaDimension.id',
                            'PlusvaliaDimension.fecha_pago_IIVTNU',
                            'PlusvaliaDimension.fecha_notificacion_liquidacion',
                            'PlusvaliaDimension.fecha_transmision',
                            'PlusvaliaDimension.fecha_adquisicion',
                            'PlusvaliaDimension.fecha_vto_redactar_indebidos',
                            'PlusvaliaDimension.fecha_vto_redactar_rea',
                            'PlusvaliaDimension.fecha_vto_redactar_reposicion',
                            'PlusvaliaDimension.fecha_vto_redactar_alzada',
                            'PlusvaliaDimension.fecha_vto_redactar_contencioso',
                            'PlusvaliaDimension.fecha_vto_redactar_impulso_resolucion',
                            'PlusvaliaDimension.fecha_presentacion_escrito_indebidos',
                            'PlusvaliaDimension.fecha_presentacion_escrito_reposicion',
                            'PlusvaliaDimension.fecha_presentacion_escrito_alegaciones',
                            'PlusvaliaDimension.fecha_presentacion_escrito_rea',
                            'PlusvaliaDimension.fecha_presentacion_escrito_alzada',
                            'PlusvaliaDimension.fecha_pte_contestacion_resolucion_indebidos',
                            'PlusvaliaDimension.fecha_pte_contestacion_rea',
                            'PlusvaliaDimension.fecha_pte_contestacion_reposicion',
                            'PlusvaliaDimension.fecha_pte_contestacion_alzada',
                            'PlusvaliaDimension.fecha_escrito_indebidos',
                            'PlusvaliaDimension.fecha_escrito_rea',
                            'PlusvaliaDimension.fecha_escrito_reposicion',
                            'PlusvaliaDimension.fecha_escrito_alzada',
                            'PlusvaliaDimension.fecha_presentacion_documento_terceros',
                            'PlusvaliaDimension.fecha_prevista_pago',
                            'PlusvaliaDimension.fecha_pago_ayuntamiento',
                            'PlusvaliaDimension.fecha_presentacion_requerimiento_administrativo',
                            'InterestSettlementDimension.presentation_date',
                            'InterestSettlementDimension.decree_date',
                            'InterestSettlementDimension.decree_own_appeal_date',
                            'InterestSettlementDimension.decree_oponent_appeal_date',
                            'InterestSettlementDimension.ruling_date',
                            'InterestSettlementDimension.notary_invoice_date',
                            'InterestSettlementDimension.management_invoice_date',
                            'InterestSettlementDimension.appraisal_invoice_date',
                            'InterestSettlementDimension.registration_invoice_date',
                        ]
                    ),
                    'contain' => [
                        'Company',
                        'Clients',
                        'MyBookmark',
                        'MyHashtags',
                        'Tags' => function ($q) {
                            return $q->order(['ordering' => 'DESC']);
                        },
                        'Labels',
                        'MyReminds' => function ($q) {
                            return $q->where(['status IN' => [1, 2]])->limit(1);
                        },
                        'Appointments' => function ($q) {
                            return $q->order(['created' => 'DESC'])->limit(1);
                        },
                        'InformacionDimension',
                        'MandamientosDimension',
                        'ProcedimientoDimension',
                        'RevolvingDimension',
                        'PlusvaliaDimension',
                        'PaternidadDimension',
                        'InterestSettlementDimension',
                        'OperacionesDemandaPopular',
                        'CartelVehicles'
                    ]
                ]
            );
        }

        if ($record->get('producto') === 'ACUERDO GASTOS HIPOTECA' && !(user()->isAllowed('#gestion-acuerdos'))) {
            $this->Flash->error('No tiene permiso para gestionar este expediente.');

            return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'index']);
        }

        if ($record->isEditable() && $this->request->is('put')) {
            $data = $this->request->data();
            if (isset($data['operaciones_demanda_popular'])) {
                foreach ($data['operaciones_demanda_popular'] as $key => $value) {
                    if (empty($data['operaciones_demanda_popular'][$key]['num_titulos']) && empty($data['operaciones_demanda_popular'][$key]['precio_titulo'])) {
                        unset($data['operaciones_demanda_popular'][$key]);
                    }
                }
            }
            if ($record->get('is_revolving_product')) {
                if (!empty($data['revolving_dimension']) && !empty($data['revolving_dimension']['lengua'])) {
                    if ($data['revolving_dimension']['lengua'] == 'CATALÁN') {
                        $this->Records->addLabel($record->get('id'), 'revolvingcontratocatalan');
                    } elseif ($data['revolving_dimension']['lengua'] == 'VASCUENCE') {
                        $this->Records->addLabel($record->get('id'), 'revolvingcontratovascuence');
                    }
                }
            }

            $record = $this->Records->patchEntity($record, $data);

            if (!empty($this->request->data()['new_contract_date'])) {
                $record->set('contract_date', date('Y-m-d'));
                if (!empty($record->get('appointments')[0])) {
                    $record->set('appointment_office', $record->get('appointments')[0]->get('room_id'));
                }
            }

            $record->unsetProperty('company'); // prevent company data indexing along with record
            if ($this->Records->save($record)) {
                $this->Flash->success('Expediente actualizado correctamente.');
            } else {
                $this->Flash->error(
                    'El expediente no pudo ser actualizado, compruebe los datos.',
                    [
                        'params' => ['errors' => $record->errors()]
                    ]
                );
            }
            $this->request->data('activeTab') ? $params = explode('&', $this->request->data('activeTab')) : $params = [];
            !empty($params[0]) ? $destination = str_replace('#', '', $params[0]) : $destination = '';
            if (!empty($params[1])) {
                $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id'), 'tab' => $params[1], '#' => $destination]);
            } else {
                $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id'), '#' => $destination]);
            }
        } else {
            $recentlySeen = $this->Records->Events
                ->find('all')
                ->where(
                    [
                        'user_id' => uid(),
                        'action' => 'view',
                        'table_alias' => 'records',
                        'entity_id' => $record->get('id'),
                        'created >=' => time() - 10 * MINUTE,
                    ]
                )
                ->count();

            if (!$recentlySeen) {
                $this->Records->registerLog($record, 'view');
            }
        }

        $record->setupDirectory();
        $this->set(compact('record', 'tags'));
    }

    /**
     * Calculate web counter clients.
     *
     * @return \Cake\Network\Response|void
     */
    public function calculateweb()
    {
        $this->loadModel('Records.Records');
        $clients = $this->Records->calculateWebCounterClients();
        $this->set(compact('clients'));
    }

    /**
     * Unlinks a client from the given record
     *
     * @param  int $recordId The record ID
     * @param  int $assocId  ID of the association between Record & Client
     * @return void Redirects
     */
    public function removeClient($recordId, $assocId)
    {
        $this->loadModel('Clients.ClientsRecords');
        $assoc = $this->ClientsRecords->find()
            ->where(
                [
                    'id' => $assocId,
                    'record_id' => $recordId,
                ]
            )
            ->limit(1)
            ->first();
        if ($assoc) {
            if ($this->ClientsRecords->delete($assoc)) {
                $this->Flash->success('El cliente fue removido del expediente exitosamente.');
            } else {
                $this->Flash->error(
                    'El cliente pudo ser removido del expediente.',
                    [
                        'params' => ['errors' => [$assoc->get('errors')]]
                    ]
                );
            }
        } else {
            $this->Flash->error('El cliente especificado no está asociado al expediente, quizás ya fue removido anteriormente.');
        }
        $this->redirect($this->referer());
    }

    /**
     * Removes a meeting from specified record.
     *
     * @param  int $recordId Record ID
     * @param  int $assocId  Association ID (Appointments <-> Meetings)
     * @return \Cake\Network\Response|void
     */
    public function removeMeeting($recordId, $assocId)
    {
        $this->loadModel('Records.Meetings');
        $assoc = $this->Meetings->find()
            ->where(
                [
                    'id' => $assocId,
                    'record_id' => $recordId,
                ]
            )
            ->limit(1)
            ->first();
        if (user()->isAdmin() || ($assoc->get('user_id') == user()->get('id'))) {
            if ($assoc) {
                if ($this->Meetings->delete($assoc)) {
                    $this->Flash->success('La cita ha sido "desatendida" de forma exitosa.');
                } else {
                    $this->Flash->error(
                        'La cita no ha podido ser "desatendida", inténtelo más tarde.',
                        [
                            'params' => ['errors' => $assoc->errors()]
                        ]
                    );
                }
            } else {
                $this->Flash->error('La cita atendida especificada no está asociada al expediente, quizás ya fue eliminada anteriormente.');
            }
        } else {
            $this->Flash->error('Esta cita fue atentida por otra persona, no está autorizado para eliminar esta información.');
        }
        $this->redirect("/records/Manage/edit/{$recordId}#meetings");
    }

    /**
     * Add a record to the bookmarks list
     *
     * @param  int $id Record ID
     * @return \Cake\Network\Response|void
     */
    public function follow($id)
    {
        $this->loadModel('Records.Records');
        if ($this->Records->follow($id)) {
            $this->Flash->success('Ha añadido el expediente #' . $id . ' a sus favoritos.');
        } else {
            $this->Flash->error('No se ha podido añadir el expediente #' . $id . ' a sus favoritos.');
        }
        $this->redirect($this->referer());
    }

    /**
     * Remove a record from the bookmarks list
     *
     * @param  int $id Record ID
     * @return \Cake\Network\Response|void
     */
    public function unfollow($id)
    {
        $this->loadModel('Records.Records');
        if ($this->Records->unfollow($id)) {
            $this->Flash->success('Ha eliminado el expediente #' . $id . ' de sus favoritos.');
        } else {
            $this->Flash->error('No se ha podido eliminar el expediente #' . $id . ' de sus favoritos. Puede haber sido eliminado anteriormente.');
        }
        $this->redirect($this->referer());
    }

    /**
     * Send an email.
     *
     * @return \Cake\Network\Response|void
     * @throws \Cake\Network\Exception\BadRequestException When invalid request data is sent
     * @throws \Cake\Network\Exception\InternalErrorException When saving process
     *  fails to register the provided email (due to internal errors or mode
     *  validations)
     * @see    https://github.com/TMServices/arriaga/issues/2167
     */
    public function sendEmail()
    {
        $data = $this->request->data();
        $this->loadModel('Records.Records');
        $response = [
            'message' => 'No se ha podido enviar el email, inténtelo más tarde.',
            'errors' => [],
            'success' => false,
        ];

        if (empty($data['record_id'])
            || (empty($data['recipients']) && empty($data['additional_recipients']))
        ) {
            throw new BadRequestException('Recipients, record id and inbox can\'t be null');
        }

        $record = $this->Records->get($this->request->data('record_id'), ['fields' => 'id']);
        try {
            $additionalRecipients = (array)explode(',', (string)$this->request->data('additional_recipients'));
            $additionalRecipients = array_map('trim', $additionalRecipients);

            if (!empty($data['recipients'])) {
                $emailsToSend = [];
                if (!is_array($data['recipients'])) {
                    $recipients[] = $data['recipients'];
                } else {
                    $recipients = $data['recipients'];
                }

                foreach ($recipients as $recipient) {
                    list($clientId, $email) = explode('::', $recipient);
                    $client = $this->Records->Clients->get($clientId, ['fields' => ['email', 'email2']]);
                    if (empty($client->get('email')) && empty($client->get('email2'))) {
                        throw new BadRequestException('Empty client id or email');
                    } else {
                        $emailsToSend[] = !empty($client->get('email')) ? $client->get('email') : $client->get('email2');
                    }
                }
                $recipientsArray = array_merge($emailsToSend, $additionalRecipients);
            } else {
                $recipientsArray = $additionalRecipients;
            }

            $recipientsArray = array_map('trim', $recipientsArray);
            $recipientsArray = array_filter($recipientsArray);
            $envelope = $this->Records
                ->composeOutgoingEnvelope('default')
                ->setSubject($this->request->data('subject'))
                ->setFrom([$record->get('inbox_email') => 'Arriaga Asociados'])
                ->setContent($this->request->data('body'))
                ->replyTo($record->get('inbox_email'))
                ->emailFormat('html')
                ->template('default', 'Records.layout-marketing-banner-default');

            foreach ($recipientsArray as $address) {
                $envelope->addTo($address);
            }

            if ($this->request->data('cc')) {
                foreach ((array)$this->request->data('cc') as $address) {
                    $envelope->addCc($address);
                }
            }

            if ($this->request->data('email_attachments')) {
                foreach ($this->request->data('email_attachments') as $attachment) {
                    $file = new LocalFile(['path' => $attachment['tmp_name']]);
                    $envelope->addAttachment($file, $attachment['name']);
                }
            }

            if ($this->request->data('path')) {
                $file = $record->fs()->getHandler($this->request->data('path'));
                $envelope->addAttachment($file);
            }

            $envelope->viewVars(
                [
                    'record' => $record,
                    'envelope' => $envelope
                ]
            );

            $errors = new \ArrayObject([]);
            if ($this->Records->sendEnvelope($envelope, $this->request->data('record_id'), $errors)) {
                $response = [
                    'message' => 'El email se ha enviado correctamente.',
                    'errors' => (array)$errors,
                    'success' => true,
                ];
            } else {
                $response = [
                    'message' => 'No ha sido posible enviar el email',
                    'success' => false,
                    'errors' => (array)$errors,
                ];
            }
        } catch (\Exception $ex) {
            $response = [
                'message' => 'El mensaje no ha podido ser entregado por un error interno.',
                'errors' => ['Detalles: ' . $ex->getMessage()],
                'success' => false,
            ];
        }

        if (!$response['success']) {
            Log::write('debug', '[EMAIL] - ' . $response['message'] . ' - Errors: ' . implode("\n", $response['errors']), 'email');
        }

        $this->set(compact('response'));
    }

    /**
     * Send an email to Wizink.
     *
     * @return \Cake\Network\Response|void
     * @throws \Cake\Network\Exception\BadRequestException When invalid request data is sent
     * @throws \Cake\Network\Exception\InternalErrorException When saving process
     *  fails to register the provided email (due to internal errors or mode
     *  validations)
     */
    public function emailToWizink()
    {
        $data = $this->request->data();
        $this->loadModel('Records.Records');
        $response = [
            'message' => 'No se ha podido enviar el email, inténtelo más tarde.',
            'errors' => [],
            'success' => false,
        ];

        if (empty($data['record_id']) || empty($data['client_id'])) {
            throw new BadRequestException('Recipients, record id or client id can\'t be null');
        }

        $record = $this->Records
            ->find('all', ['eav' => false])
            ->contain(
                [
                    'Clients' => function ($q) use ($data) {
                        return $q->where(['Clients.id' => $data['client_id']]);
                    },
                    'RevolvingDimension'
                ]
            )
            ->where(['Records.id' => $data['record_id']])
            ->first();

        try {
            if ($record->get('revolving_dimension') == null || $record->get('revolving_dimension')->get('numeracion_tarjeta') == null) {
                throw new BadRequestException('El expediente debe tener "NUMERACIÓN TARJETA" en la pestaña "Redacción".');
            }

            $body = jsonc('Records.email-wizink.texto');
            $numTarjeta = substr($record->get('revolving_dimension')->get('numeracion_tarjeta'), -4);
            $replace = [
                '<<nombre_cliente>>' => $record->get('clients')[0]->get('lastname') . ', ' . $record->get('clients')[0]->get('name'),
                '<<dni_cliente>>' => $record->get('clients')[0]->get('dni'),
                '<<record_id>>' => $record->get('crm_id'),
                '<<ntarjeta>>' => $numTarjeta,
                '<<colegiada_name>>' => settings('Collegiate.name'),
                '<<colegiada_number>>' => settings('Collegiate.number'),
            ];
            foreach ($replace as $key => $value) {
                $body = str_replace($key, $value, $body);
            }
            $envelope = $this->Records
                ->composeOutgoingEnvelope('default')
                ->setSubject('Solicitud documentación')
                ->setFrom([$record->get('inbox_email') => 'Arriaga Asociados'])
                ->setContent($body)
                ->replyTo($record->get('inbox_email'))
                ->emailFormat('html')
                ->addTo('reclamaciones@wizink.es')
                ->template('default', 'Records.default');

            //adjunto archivo de mandato
            $files = $record->files(
                [
                    'filter' => '*mandato*',
                    'cache' => true,
                    'insensitive' => true,
                ]
            );

            $path = false;
            foreach ($files as $file) {
                if (stripos($file['basename'], 'firmado') !== false) {
                    $path = $file['path'];
                    break;
                }
            }

            if ($path) {
                $file = $record->fs()->getHandler($path);
                $envelope->addAttachment($file);
            } else {
                throw new BadRequestException('El cliente elegido no tiene el documento mandato firmado.');
            }

            //adjunto dni responsable Arriaga
            $path = normalizePath(Plugin::path('Records') . settings('Collegiate.file-dni'));
            $dniImage = new LocalFile(['path' => $path]);
            $envelope->addAttachment($dniImage);

            //adjunto dni cliente
            $client = $record->get('clients')[0];
            if ($files = $client->files(['filter' => 'files/DNI*.*'])->toArray()) {
                $path = 'files/' . $files[0]->get('filename');
                $file = $client->fs()->getHandler($files[0]->get('path'));
                $envelope->addAttachment($file);
            } else {
                throw new BadRequestException('El cliente elegido no tiene DNI.');
            }

            $envelope->viewVars(
                [
                    'record' => $record,
                    'envelope' => $envelope
                ]
            );

            $errors = new \ArrayObject([]);
            if ($this->Records->sendEnvelope($envelope, $this->request->data('record_id'), $errors)) {
                $record = $this->Records->patchEntity($record, ['revolving_dimension' => ['send_email' => 1]]);
                $this->Records->save($record);
                $response = [
                    'message' => 'El email se ha enviado correctamente.',
                    'errors' => (array)$errors,
                    'success' => true,
                ];
            } else {
                $response = [
                    'message' => 'No ha sido posible enviar el email',
                    'success' => false,
                    'errors' => (array)$errors,
                ];
            }
        } catch (\Exception $ex) {
            $response = [
                'message' => 'El mensaje no ha podido ser entregado por un error interno o falta de documentación.',
                'errors' => ['Detalles: ' . $ex->getMessage()],
                'success' => false,
            ];
        }

        if (!$response['success']) {
            Log::write('debug', '[EMAIL] - ' . $response['message'] . ' - Errors: ' . implode("\n", $response['errors']), 'email');
        }
        $this->set(compact('response'));
    }

    /**
     * Users are redirected here after payment process completes.
     *
     * Here we show them a `successfully` like message about the order.
     *
     * @param  int $recordId record id
     * @return void|\Cake\Network\Response
     */
    public function paymentCompleted($recordId)
    {
        $this->Flash->success('Se ha pagado correctamente la orden.');
        $this->redirect("/records/Manage/edit/{$recordId}#record-orders");
    }

    /**
     * Users are redirected here when payment process fails complete.
     *
     * Here we show them a `failure` like message about the order.
     *
     * @param  int $recordId record id
     * @return void|\Cake\Network\Response
     */
    public function paymentError($recordId)
    {
        $this->Flash->error('Se ha producido un error al pagar la orden.');
        $this->redirect("/records/Manage/edit/{$recordId}#record-orders");
    }

    /**
     * Crea una copia de un fichero del expediente en un directorio llamado `para-lexnet`.
     *
     * @param  int    $recordId Rrecord ID
     * @param  string $handler  Tipo de demand handler
     * @return \Cake\Network\Response|null
     * @throws \Cake\Network\Exception\BadRequestException Cuando no se ha podido subir el
     *  archivo o cuando el handler correspondiente no existe
     */
    public function uploadToLexnet($recordId, $handler)
    {
        $this->loadModel('Records.Records');
        $record = $this->Records->get(
            $recordId,
            [
                'eav' => false,
                'contain' => ['Clients']
            ]
        );

        $handler = '\Records\Lib\DemandFile\\' . $handler;
        if (!class_exists($handler)) {
            throw new BadRequestException('Demand handler not found.');
        }
        $demandHandler = new $handler($record);

        if ($this->request->is('post') && $this->request->data('file')) {
            if ($this->request->data('file')['type'] === 'application/pdf' && $this->request->data('file')['size'] < 12582912) {
                $uploadRequest = $this->request;
            }
        } else {
            $files = $demandHandler->search();
            $filePath = $this->request->query('path');
            if (substr($filePath, 0, 6) == 'hash::') {
                $filePath = str_replace('hash::', '', $filePath);
                $filePath = base64UrlDecode($filePath);
                $filePath = Security::decrypt($filePath, Security::salt());
            }

            $file = $files->firstMatch(['path' => $filePath]);

            if ($file
                && $file->get('mimetype') === 'application/pdf'
                && $file->get('size') < 12582912 // 12 MB
            ) {
                $stream = $file->readStream();
                $tmpFp = tmpfile();
                stream_copy_to_stream($stream, $tmpFp);
                rewind($tmpFp);
                fclose($stream);

                $metadata = stream_get_meta_data($tmpFp);
                $this->request->data['file'] = [
                    'tmp_name' => $metadata['uri'],
                    'fp' => $tmpFp,
                ];
            } else {
                throw new BadRequestException('File could not be uploaded.');
            }
        }

        $demandHandler->upload($this->request);

        return $this->redirect("/records/Manage/edit/{$recordId}#drafting");
    }

    /**
     * Unfolds (splits) the given record into another.
     *
     * @param  int $recordId Record to be split
     * @return void|\Cake\Network\Response
     */
    public function unfoldRecord($recordId)
    {
        if (!settings('App.instance-tags.development')) { //Funcionalidad CRM
            throw new ForbiddenException('Esta funcionalidad NO está opertativa, inténtelo a través del gestor de clientes.');
        } else {
            $this->loadModel('Records.Records');
            $result = [];
            $transactionResult = $this->Records
                ->connection()
                ->transactional(
                    function () use ($recordId, &$result) {
                        $original = $this->Records
                            ->find('all', ['eav' => false])
                            ->where(['id' => $recordId])
                            ->contain(['Clients'])
                            ->epilog('FOR UPDATE')
                            ->firstOrFail();

                        $typeUnfold = explode('#', $original->get('unfold'));
                        $assignation = array_pop($typeUnfold);
                        if ($assignation != 'W' && $assignation != 'X') {
                            if ($original->get('producto') == 'GASTOS HIPOTECA') {
                                $unfold = $original->get('id') . '#D';
                                $producto = 'RECLAMACIÓN POR CANTIDAD';
                                $symbol = '#X';
                            } else {
                                $unfold = $original->get('id') . '#D';
                                $producto = 'CLAUSULAS SUELO';
                                $original->set('producto', 'GASTOS HIPOTECA');
                                $symbol = '#O';
                            }

                            $companyId = $original->get('company_id') ? $original->get('company_id') : 2;
                            $clone = $this->Records->newEntity(
                                [
                                    'company_id' => $companyId,
                                    'producto' => $producto,
                                    'pf_importe' => 0,
                                    'pf_estado' => 'SI',
                                    'pf_fecha' => date('Y-m-d'),
                                    'pf_justificante_pago' => 'PROCEDENTE DE DESDOBLADO',
                                    'persona_comercial' => $original->get('persona_comercial'),
                                    'appointment_office' => $original->get('appointment_office'),
                                    'ciudadjpi' => $original->get('ciudadjpi'),
                                    'email_responsible' => $original->get('email_responsible'),
                                    'status' => 'EXPEDIENTE INICIADO',
                                    'poder_notarial' => $original->get('poder_notarial'),
                                    'unfold' => $unfold,
                                ],
                                [
                                    'associated' => ['Clients'],
                                    'validate' => false
                                ]
                            );

                            if (!$this->Records->save($original)) {
                                $errors = Hash::flatten($original->errors());
                                $errors = implode('<br />', array_values($errors));
                                $result = [
                                    'msg' => [
                                        'type' => 'error',
                                        'text' => 'El expediente original no pudo ser modificado, compruebe los datos: ' . $errors
                                    ],
                                    'redirect' => $recordId
                                ];

                                return false;
                            }

                            if ($this->Records->save($clone, ['associated' => false])) {
                                $clone->addTag('EXP. DESDOBLADO');

                                foreach ($original->get('clients') as $client) {
                                    $this->Records->linkWithClient($clone, $client, $client->get('_joinData')->get('client_role'));
                                }

                                $files = $original->files(
                                    [
                                        'filter' => 'files/*',
                                        'cache' => true,
                                    ]
                                )
                                    ->filter(
                                        function ($file) {
                                            return !str_starts_with($file->get('path'), 'files/facturas/');
                                        }
                                    );

                                $manager = FilesystemFactory::manager(
                                    [
                                        'original' => $original->fs(),
                                        'clone' => $clone->fs(),
                                    ]
                                );

                                foreach ($files as $file) {
                                    try {
                                        $manager->copy('original://' . $file->get('path'), 'clone://' . $file->get('path'));
                                    } catch (\Exception $ex) {
                                        // failed to copy
                                    }
                                }

                                $clone->touchContainer();
                                if ($original->get('unfold')) {
                                    $fatherId = explode('#', $original->get('unfold'));
                                    $this->Records->updateAll(['unfold' => $fatherId[0] . '#' . $clone->get('id') . '#W'], ['id' => $original->get('id')]);
                                } else {
                                    $this->Records->updateAll(['unfold' => $clone->get('id') . $symbol], ['id' => $original->get('id')]);
                                }
                                $result = [
                                    'msg' => [
                                        'type' => 'success',
                                        'text' => 'Expediente desdoblado correctamente.'
                                    ],
                                    'redirect' => $clone->get('id')
                                ];

                                $clone->set('status', 'COMPLETANDOSE DOCUMENTACIÓN');
                                $this->Records->save($clone);

                                return true;
                            } else {
                                $errors = Hash::flatten($clone->errors());
                                $errors = implode('<br />', array_values($errors));
                                $result = [
                                    'msg' => [
                                        'type' => 'error',
                                        'text' => 'El expediente no pudo ser desdoblado, compruebe los datos: ' . $errors
                                    ],
                                    'redirect' => $recordId
                                ];

                                return false;
                            }
                        } else {
                            $result = [
                                'msg' => [
                                    'type' => 'error',
                                    'text' => '¡¡ERROR!! Este expediente ya ha sido desdoblado en una reclamación de cantidad.'
                                ],
                                'redirect' => $recordId
                            ];

                            return false;
                        }
                    }
                );

            if ($result['msg']['type'] == 'success') {
                $this->Flash->success($result['msg']['text']);
            } else {
                $this->Flash->error($result['msg']['text']);
            }

            $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $result['redirect']]);
        }
    }

    /**
     * Marks the given Records as finished.
     *
     * @param  int $recordId Record to mark as finished
     * @return void|\Cake\Network\Response
     * @throws \Cake\Network\Exception\ForbiddenException When user has no permissions
     */
    public function finishRecord($recordId)
    {
        $this->loadModel('Records.Records');
        $record = $this->Records->get(
            $recordId,
            [
                'eav' => false,
                'fields' => ['id', 'status', 'finished'],
                'conditions' => ['finished IS' => null]
            ]
        );

        if (!user()->isAdmin()) {
            throw new ForbiddenException('You are not allowed to mark as finished any Record.');
        }

        $success = $this->Records->updateAll(['finished' => time()], ['id' => $recordId]);

        if ($success) {
            $this->Flash->success('El expediente ha sido marcado como finalizado.');
        } else {
            $this->Flash->error('El expediente no ha podido ser marcado como finalizado. ¿Ya lo está?');
        }

        return $this->redirect($this->referer());
    }

    /**
     * Retorna un listado de atributos de Records susceptible de ser utilizado como parte
     * de un `SELECT`.
     *
     * El conjunto retornado varía en función de la petición actual, de forma que se
     * garantice la trazabilidad de los atributos modificados en un proceso de
     * persistencia.
     *
     * NOTA: Este método tiene en consideración tanto atributos físicos como virtuales
     * (EAV).
     *
     * @param  array $additional Atributos adicionales
     * @return array Subconjunto de atributos de Records.
     */
    protected function _calculatePersistableFields(array $additional = [])
    {
        $defaultFields = [
            'id',
            'group_id',
            'order_id',
            'pin',
            'company_id',
            'nig',
            'parent_id',
            'aditional_notes',
            'notes_history',
            'status',
            'type',
            'files_count',
            'importe',
            'producto',
            'banco',
            'pf_estado',
            'pf_importe',
            'pf_medio_cobro',
            'pf_fecha',
            'pf_justificante_pago',
            'pf_banco',
            'est_costas_arr_cantidad',
            'est_costas_proc',
            'costas_primera_ins',
            'pro_fecha_demanda',
            'pro_decreto_admision_tramite_fecha',
            'pro_fecha_copia_sellada',
            'pro_fecha_sentencia_primera_instancia',
            'pro_fecha_audiencia',
            'pro_fecha_juicio',
            'pro_fecha_vistas',
            'created',
            'modified',
            'deleted',
            'finished',
            'reviewed',
            'intereses',
            'referencia_ext',
            'fact_arr',
            'cobrado_arr',
            'ciudadjpi',
            'contract_date',
            'appointment_office',
            'email_responsible',
            'unfold',
            'pro_jpi',
            'pro-autos',
            'poder_notarial',
            'gh_protocolo_escritura',
            'gh_protocolo_escritura_segunda',
            'gh_protocolo_escritura_tercera',
            'csb_protocolo_escritura',
            'csb_protocolo_escritura_novacion',
            'csb_protocolo_escritura_segunda_novacion',
            'csb_protocolo_escritura_promotor',
            'department_risk',
            'RevolvingDimension.tae_contratacion',
            'RevolvingDimension.tae_contrato',
            'RevolvingDimension.tae_maximo',
            'RevolvingDimension.tae_contrato_efectivo',
            'RevolvingDimension.tae_incrementado_efectivo',
            'RevolvingDimension.tipo_its',
            'RevolvingDimension.nuevo_tipo_its',
            'RevolvingDimension.limite',
            'RevolvingDimension.nuevo_limite'
        ];

        $defaultFields = array_unique(array_merge($defaultFields, $additional));

        if ($this->request->data()) {
            $this->loadModel('Records.Records');
            $schemaReal = $this->Records->schema();
            $schemaVirtual = $this->Records->listColumns();
            $totalSchema = array_merge($schemaReal->columns(), array_keys($schemaVirtual));

            $modifiedFields = array_intersect($totalSchema, array_keys($this->request->data()));
            $defaultFields = array_unique(array_merge($defaultFields, $modifiedFields));
        }

        return $defaultFields;
    }

    /**
     * Controller action.
     *
     * @return void|\Cake\Network\Response
     */
    public function groupers()
    {
        $this->loadModel('Records.Records');

        if ($this->request->is('post')) {
            $data = $this->request->data();
            $dateIni = !empty($data['dateIni']) ? date_create_from_format($data['formatDate'], $data['dateIni']) : '';
            $dateFin = !empty($data['dateFin']) ? date_create_from_format($data['formatDate'], $data['dateFin']) : '';
            $dateIni = !empty($dateIni) ? date_format($dateIni, 'Y-m-d') : '2008-01-01';
            $dateFin = (!empty($dateFin) ? date_format($dateFin, 'Y-m-d') : date('Y-m-d')) . ' 23:59:59';

            $filter = $data['filter'];
            $filter = str_replace('*', '%', trim($filter));

            $whereArray = [
                'Records.created >=' => $dateIni,
                'Records.created <=' => $dateFin,
                'group_id IS NOT' => null
            ];
            $groupers = $this->Records
                ->find('all')
                ->contain(['Clients'])
                ->where($whereArray)
                ->order(['Records.id' => 'DESC']);

            if (!empty($filter)) {
                foreach (explode(' ', $filter) as $word) {
                    $groupers->andWhere(
                        [
                            'OR' => [
                                'Records.id' => $word,
                                'Records.status LIKE' => "%{$word}%",
                                'Records.crm_id' => $word,
                                'Records.group_id LIKE' => "%{$word}%"
                            ]
                        ]
                    );
                }
            }

            if (isset($data['busqueda'])) {
                $this->paginate['limit'] = 20;
                $groupers = $this->paginate($groupers);
                $this->set(compact('groupers', 'data'));
            }
        } else {
            $groupers = $this->Records
                ->find('all', ['eav' => false])
                ->where(['group_id IS NOT' => null])
                ->contain(
                    [
                        'Clients' => function ($q) {
                            return $q
                                ->select(['Clients.lastname']);
                        }
                    ]
                );

            $this->paginate['limit'] = 20;
            $groupers = $this->paginate($groupers);
            $this->set(compact('groupers'));
            $this->set('_serialize', ['groupers']);
        }
    }

    /**
     * Intenta crear un expediente agrupador (actualmente para cártel de coches) y redirigir al navegador a este.
     *
     * @return void|\Cake\Network\Response
     */
    public function createCartelCochesGroupalRecord()
    {
        $this->loadModel('Records.Records');
        $this->loadModel('Clients.Clients');
        $this->loadModel('Clients.ClientsRecords');

        if (!$this->request->is('post')) {
            $vehiclesBrandsClients = $this->Clients
                ->find('all', ['eav' => false])
                ->select(
                    [
                        'id',
                        'lastname',
                    ]
                )
                ->where(
                    [
                        'name' => 'MARCA AUTOMOVILÍSTICA',
                        'censored IS' => null,
                        'deleted IS' => null,
                    ]
                )
                ->formatResults(
                    function ($results) {
                        $results = $results->map(
                            function ($result) {
                                return [
                                    'id' => $result['id'],
                                    'lastname' => $result['lastname'],
                                ];
                            }
                        );

                        return $results->combine('id', 'lastname');
                    }
                )
                ->toArray();

            asort($vehiclesBrandsClients);

            $this->set('vehiclesBrandsClients', $vehiclesBrandsClients);
            $this->set('products', ['CÁRTEL DE COCHES' => 'CÁRTEL DE COCHES', 'CÁRTEL de COCHES REQUERIMIENTO' => 'CÁRTEL de COCHES REQUERIMIENTO']);
            $this->set('usersList', usersList([], null)->toArray());
        } else {
            $data = $this->request->data();

            $properties = [
                'producto' => $data['product'],
                'status' => 'REDACCIÓN',
                'persona_redactor' => $data['persona-redactor'],
                'email_responsible' => $data['persona-redactor'],
            ];

            $vehicleBrandName = $this->Clients->get(intval($data['vehicle-brand-id']), ['eav' => false])->get('lastname');

            $specialGroupsMapping = [
                'FIAT CHRYSLER AUTOMOBILES SPAIN S.A.' => 'FIAT',
                'MERCEDES-BENZ' => 'MERCEDES',
                'PEUGEOT ESPAÑA S.A.' => 'PEUGEOT',
            ];

            if (array_key_exists($vehicleBrandName, $specialGroupsMapping)) {
                $vehicleBrandName = $specialGroupsMapping[$vehicleBrandName];
            }

            $groupIdPrefix = $vehicleBrandName;
            $groupIdPrefix = str_replace('.', '', $groupIdPrefix);
            $groupIdPrefix = str_replace(' ', '_', $groupIdPrefix);
            $groupIdPrefix = strtoupper(replaceSpecialChars($groupIdPrefix));
            $groupIdPrefix = "DC-{$groupIdPrefix}";

            $lastUsedGroupId = $this->Records
                ->find('all', ['eav' => false])
                ->select(
                    [
                        'group_id',
                    ]
                )
                ->where(
                    [
                        'group_id LIKE' => "{$groupIdPrefix}%",
                    ]
                )
                ->order(
                    [
                        'group_id' => 'DESC',
                    ]
                )
                ->extract('group_id')
                ->first();

            if (empty($lastUsedGroupId)) {
                $groupIdSuffix = 0;
            } else {
                preg_match('/\d{4}$/', $lastUsedGroupId, $groupIdSuffix);
                $groupIdSuffix = intval(end($groupIdSuffix));
            }

            $groupIdSuffix++;
            $groupIdSuffix = str_pad((string)$groupIdSuffix, 4, '0', STR_PAD_LEFT);
            $groupId = $groupIdPrefix . $groupIdSuffix;

            $properties['group_id'] = $groupId;

            $record = $this->Records->newEntity($properties);

            $conn = $this->Records->connection();
            $conn->begin();

            if (!$this->Records->save($record)) {
                $conn->rollback();

                $this->Flash->error('El expediente agrupador no ha podido crearse.', ['params' => ['errors' => $record->errors()]]);

                return $this->redirect($this->referer());
            }

            $record = $this->Records->get($record->get('id'), ['eav' => false]);
            $record->set('crm_id', intval('50' . $record->get('id')));

            if (!$this->Records->save($record)) {
                $conn->rollback();

                $this->Flash->error('El expediente agrupador no ha podido crearse.', ['params' => ['errors' => $record->errors()]]);

                return $this->redirect($this->referer());
            }

            $clientAssoc = $this->ClientsRecords->newEntity(
                [
                    'record_id' => $record->get('id'),
                    'client_id' => intval($data['vehicle-brand-id']),
                    'client_role' => 'Demandado Contrario',
                ]
            );

            try {
                if (!$this->ClientsRecords->save($clientAssoc)) {
                    $conn->rollback();

                    $this->Flash->error('El expediente agrupador no ha podido crearse.', ['params' => ['errors' => $clientAssoc->errors()]]);

                    return $this->redirect($this->referer());
                }
            } catch (\Exception $ex) {
                $conn->rollback();

                $this->Flash->error('El expediente agrupador no ha podido crearse; no ha sido posible asociar la marca demandada');

                return $this->redirect($this->referer());
            }

            $conn->commit();

            $this->Flash->success('Expediente agrupador creado correctamente.');
            $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id')]);
        }
    }

    /**
     * Crea un job para añadir expedientes a un expediente agrupador para cártel de coches.
     *
     * @return void|\Cake\Network\Response
     */
    public function addRecordsToGroupCartelCoches()
    {
        if (!$this->request->is('post')) {
            return $this->redirect($this->referer());
        }

        $data = $this->request->data();

        $args = [
            'groupalRecordId' => $data['groupal-record'],
            'recordsToAddCrmIds' => $data['records-to-add'],
        ];

        $options = [
            'replayMode' => 'queue',
        ];

        $job = JobRegistry::get('Records.CartelCochesGrouping/AddRecordsToGroup/AddRecordsToGroup');
        $queuedJob = $job->enqueue($args, $options);

        if (!$queuedJob) {
            $this->Flash->error('No ha podido crearse un trabajo para añadir expedientes a un expediente agrupador');

            return $this->redirect($this->referer());
        }

        return $this->redirect(
            [
                'plugin' => 'Queues',
                'controller' => 'jobs',
                'action' => 'details',
                $queuedJob->get('id'),
                'watch' => 1,
            ]
        );
    }

    /**
     * Crea un job para eliminar expedientes de un expediente agrupador para cártel de coches.
     *
     * @return void|\Cake\Network\Response
     */
    public function removeRecordsFromGroupCartelCoches()
    {
        if (!$this->request->is('post')) {
            return $this->redirect($this->referer());
        }

        $data = $this->request->data();

        $args = [
            'groupalRecordId' => $data['groupal-record'],
            'recordsToRemoveIds' => array_filter($data['records-to-remove']),
        ];

        $options = [
            'replayMode' => 'queue',
        ];

        $job = JobRegistry::get('Records.CartelCochesGrouping/RemoveRecordsFromGroup/RemoveRecordsFromGroup');
        $queuedJob = $job->enqueue($args, $options);

        if (!$queuedJob) {
            $this->Flash->error('No ha podido crearse un trabajo para eliminar expedientes del expediente agrupador');

            return $this->redirect($this->referer());
        }

        return $this->redirect(
            [
                'plugin' => 'Queues',
                'controller' => 'jobs',
                'action' => 'details',
                $queuedJob->get('id'),
                'watch' => 1,
            ]
        );
    }

    /**
     * Request process type to minerva.
     *
     * @param  bool   $guarantee crm guarantee
     * @param  int    $crmId     crm record number
     * @param  string $type      process type to run in minerva
     * @return void|\Cake\Network\Response
     */
    public function minervaProcess($guarantee, $crmId, $type)
    {
        throw new ForbiddenException('Esta funcionalidad NO está operativa todavía.');

        /*$callMinerva = Minerva::minervaProcessCall($guarantee, $crmId, $type);

        if ($callMinerva) {
            $this->loadModel('Records.Records');
            $transactionResult = $this->Records
                ->connection()
                ->transactional(function () use ($crmId, &$result) {
                    $record = $this->Records
                        ->find('all', ['eav' => false])
                        ->select(['id', 'crm_id', 'status'])
                        ->where(['crm_id' => $crmId])
                        ->firstOrFail();
                    $processStatus = [
                        'precautionaryMeasure' => 'MEDIDAS CAUTELARES',
                        'preliminaryHearing' => 'DILIGENCIAS PRELIMINARES',
                        'actConciliation' => 'ACTO DE CONCILIACIÓN'
                    ];
                    $record->set('status', jsonc('Records.statuses.' . $processStatus[$type]));
                    $recordId = $record->get('id');
                    if ($this->Records->save($record)) {
                        $this->Flash->success('El expediente ha pasado a ' . $processStatus[$type] . ' correctamente.');
                        $this->redirect("/records/Manage/edit/{$recordId}#drafting");
                    } else {
                        $this->Flash->error('NO se ha podido pasar el expediente a ' . $processStatus[$type] . '.');
                        $this->redirect("/records/Manage/edit/{$recordId}#drafting");
                    }
                });
        }*/
    }

    /**
     * Consulta el estado de una carta certificada en notificados.com
     *
     * @param  int $recordId id record number
     * @return void|\Cake\Network\Response
     */
    public function consultarEnvioNotificados($recordId)
    {
        if ($this->request->query('id_burofax')) {
            $notificadoId = $this->request->query('id_burofax');
            $update = NotificadosOperators::updateNotificado($notificadoId);
            if ($update) {
                $this->Flash->success('Actualización de estado realizada con éxito');

                return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
            } else {
                $this->Flash->error('Error!! No se ha podido actualizar el estado');

                return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
            }
        }
        $this->Flash->error('Error!! Se necesita un identificativo de carta certificada con notificados.com');

        return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
    }

    /**
     * Descarga el documento que certifica la entrega desde notificados.com
     *
     * @param  int $recordId id record number
     * @return void|\Cake\Network\Response
     */
    public function descargaCertificadoNotificados($recordId)
    {
        if ($this->request->query('id_envio')) {
            $notificadoId = $this->request->query('id_envio');
            $content = NotificadosOperators::downloadNotificado($notificadoId);
            if ($content) {
                $this->response->body(base64_decode($content, true));
                $this->response->type('pdf');
                $this->response->download($notificadoId . '.pdf');

                return $this->response;
            } else {
                $this->Flash->error('Error!! No se ha podido descargar el acuse de recibo.');

                return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
            }
        }

        return false;
    }

    /**
     * envía un archivo a notificados.com
     *
     * @param  int $recordId id record number
     * @return void|\Cake\Network\Response
     */
    public function envioNotificados($recordId)
    {
        if ($this->request->query('file')) {
            $file = Util::normalizeRelativePath($this->request->query('file'));
            $content = NotificadosOperators::sendNotificado($recordId, $file);
            if ($content) {
                $this->Flash->success('Envío a Notificados.com realizado con éxito');

                return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
            } else {
                $this->Flash->error('Error!! No se ha podido realizar el envío a notificados.com.');

                return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $recordId, '#' => 'record-burofax']);
            }
        }

        return false;
    }




    public function editTC($id)
    {
        $this->loadModel('Records.Records');
        $tags = $this->Records->Tags->find('all')
            ->cache('record_edit', 'record_tags')
            ->select(['id', 'name', 'color', 'ordering'])
            ->order(['ordering' => 'ASC'])
            ->formatResults(
                function ($results) {
                    return $results->combine(
                        'id',
                        function ($tag) {
                            return sprintf('<span class="label label-success" style="background-color:#%s" data-ordering="%s">%s</span>', $tag->get('color'), $tag->get('ordering'), $tag->get('name'));
                        }
                    );
                }
            );

        if (user()->isAllowed('#clients:censorship')) {
            $this->Records->behaviors()->get('SoftDelete')->config('filterFlag', null, false);
        }
        if ($this->request->is('get')) {
            $record = $this->Records->get(
                $id,
                [
                    'eav' => true,
                    'fields' => $this->_calculatePersistableFields(
                        [
                            'MyBookmark.id'
                        ]
                    ),
                    'contain' => [
                        'Company',
                        'Clients',
                        'MyBookmark',
                        'MyHashtags',
                        'Tags' => function ($q) {
                            return $q->order(['ordering' => 'DESC']);
                        },
                        'Labels',
                        'MyReminds' => function ($q) {
                            return $q->where(['status IN' => [1, 2]])->limit(1);
                        },
                        'Appointments' => function ($q) {
                            return $q->order(['created' => 'DESC'])->limit(1);
                        },
                        'InformacionDimension',
                        'MandamientosDimension',
                        'ProcedimientoDimension',
                        'RevolvingDimension',
                        'PlusvaliaDimension',
                        'PaternidadDimension',
                        'OperacionesDemandaPopular',
                        'CartelVehicles',
                    ]
                ]
            );
        } else {
            $record = $this->Records->get(
                $id,
                [
                    'eav' => true,
                    'fields' => $this->_calculatePersistableFields(
                        [
                            'MyBookmark.id',
                            'InformacionDimension.id',
                            'MandamientosDimension.id',
                            'ProcedimientoDimension.id',
                            'RevolvingDimension.id',
                            'PaternidadDimension.id',
                            'PlusvaliaDimension.id',
                            'PlusvaliaDimension.fecha_pago_IIVTNU',
                            'PlusvaliaDimension.fecha_notificacion_liquidacion',
                            'PlusvaliaDimension.fecha_transmision',
                            'PlusvaliaDimension.fecha_adquisicion',
                            'PlusvaliaDimension.fecha_vto_redactar_indebidos',
                            'PlusvaliaDimension.fecha_vto_redactar_rea',
                            'PlusvaliaDimension.fecha_vto_redactar_reposicion',
                            'PlusvaliaDimension.fecha_vto_redactar_alzada',
                            'PlusvaliaDimension.fecha_vto_redactar_contencioso',
                            'PlusvaliaDimension.fecha_vto_redactar_impulso_resolucion',
                            'PlusvaliaDimension.fecha_presentacion_escrito_indebidos',
                            'PlusvaliaDimension.fecha_presentacion_escrito_reposicion',
                            'PlusvaliaDimension.fecha_presentacion_escrito_alegaciones',
                            'PlusvaliaDimension.fecha_presentacion_escrito_rea',
                            'PlusvaliaDimension.fecha_presentacion_escrito_alzada',
                            'PlusvaliaDimension.fecha_pte_contestacion_resolucion_indebidos',
                            'PlusvaliaDimension.fecha_pte_contestacion_rea',
                            'PlusvaliaDimension.fecha_pte_contestacion_reposicion',
                            'PlusvaliaDimension.fecha_pte_contestacion_alzada',
                            'PlusvaliaDimension.fecha_escrito_indebidos',
                            'PlusvaliaDimension.fecha_escrito_rea',
                            'PlusvaliaDimension.fecha_escrito_reposicion',
                            'PlusvaliaDimension.fecha_escrito_alzada',
                            'PlusvaliaDimension.fecha_presentacion_documento_terceros',
                            'PlusvaliaDimension.fecha_prevista_pago',
                            'PlusvaliaDimension.fecha_pago_ayuntamiento',
                            'PlusvaliaDimension.fecha_presentacion_requerimiento_administrativo'
                        ]
                    ),
                    'contain' => [
                        'Company',
                        'Clients',
                        'MyBookmark',
                        'MyHashtags',
                        'Tags' => function ($q) {
                            return $q->order(['ordering' => 'DESC']);
                        },
                        'Labels',
                        'MyReminds' => function ($q) {
                            return $q->where(['status IN' => [1, 2]])->limit(1);
                        },
                        'Appointments' => function ($q) {
                            return $q->order(['created' => 'DESC'])->limit(1);
                        },
                        'InformacionDimension',
                        'MandamientosDimension',
                        'ProcedimientoDimension',
                        'RevolvingDimension',
                        'PlusvaliaDimension',
                        'PaternidadDimension',
                        'OperacionesDemandaPopular',
                        'CartelVehicles',
                    ]
                ]
            );
        }

        if ($record->get('producto') === 'ACUERDO GASTOS HIPOTECA' && !(user()->isAllowed('#gestion-acuerdos'))) {
            $this->Flash->error('No tiene permiso para gestionar este expediente.');

            return $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'index']);
        }

        if ($record->isEditable() && $this->request->is('put')) {
            $data = $this->request->data();
            if (isset($data['operaciones_demanda_popular'])) {
                foreach ($data['operaciones_demanda_popular'] as $key => $value) {
                    if (empty($data['operaciones_demanda_popular'][$key]['num_titulos']) && empty($data['operaciones_demanda_popular'][$key]['precio_titulo'])) {
                        unset($data['operaciones_demanda_popular'][$key]);
                    }
                }
            }
            if ($record->get('is_revolving_product')) {
                if (!empty($data['revolving_dimension']) && !empty($data['revolving_dimension']['lengua'])) {
                    if ($data['revolving_dimension']['lengua'] == 'CATALÁN') {
                        $this->Records->addLabel($record->get('id'), 'revolvingcontratocatalan');
                    } elseif ($data['revolving_dimension']['lengua'] == 'VASCUENCE') {
                        $this->Records->addLabel($record->get('id'), 'revolvingcontratovascuence');
                    }
                }
            }

            $record = $this->Records->patchEntity($record, $data);

            if (!empty($this->request->data()['new_contract_date'])) {
                $record->set('contract_date', date('Y-m-d'));
                if (!empty($record->get('appointments')[0])) {
                    $record->set('appointment_office', $record->get('appointments')[0]->get('room_id'));
                }
            }

            $record->unsetProperty('company'); // prevent company data indexing along with record
            if ($this->Records->save($record)) {
                $this->Flash->success('Expediente actualizado correctamente.');
            } else {
                $this->Flash->error(
                    'El expediente no pudo ser actualizado, compruebe los datos.',
                    [
                        'params' => ['errors' => $record->errors()]
                    ]
                );
            }

            $this->request->data('activeTab') ? $destination = str_replace('#', '', $this->request->data('activeTab')) : $destination = '';
            $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id'), '#' => $destination]);
        } else {
            $recentlySeen = $this->Records->Events
                ->find('all')
                ->where(
                    [
                        'user_id' => uid(),
                        'action' => 'view',
                        'table_alias' => 'records',
                        'entity_id' => $record->get('id'),
                        'created >=' => time() - 10 * MINUTE,
                    ]
                )
                ->count();

            if (!$recentlySeen) {
                $this->Records->registerLog($record, 'view');
            }
        }

        $record->setupDirectory();
        $this->set(compact('record', 'tags'));
    }

        /**
     * Crea una TC (Tasación de Costas) y valida los datos del formulario.
     *
     * Esta función maneja la creación de una TC, realizando varias validaciones
     * basadas en los datos del formulario enviados. Si se encuentran errores
     * en los datos, se añaden al arreglo de errores, de lo contrario, se
     * procede a guardar la información y redirigir al usuario a la página
     * correspondiente.
     *
     * @return \Cake\Network\Response Redirige al usuario dependiendo del
     *         resultado de la validación.
     */
    public function createTC()
    {

        $errors = [];
        $destination = 'procedure-man';
        $this->loadModel('CourtCosts');
        $this->loadModel('Records');
        $this->loadModel('LawyerCourtCosts');
        $this->loadModel('AttorneyCourtCosts');
        $formData = $this->request->data();

        if (!isset($formData['impugnacion_traslado'])) {
            $formData['impugnacion_traslado'] = 0;
        }
        $recordId = isset($formData['record_id']) ? $formData['record_id'] : null;
        $record = $this->Records->findById($recordId)->first();


        if (!$record) {
            $this->Flash->error(__('No se encontró el registro correspondiente.'));
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $record->get('id'),
                    '#'          => $destination
                ]
            );
        }

        $tipoProcedimiento = isset(
            $formData["tipo_procedimiento"]
        ) ?
            $formData["tipo_procedimiento"] : null;

        if ($tipoProcedimiento !== 'incidente') {
            $existingCost = $this->CourtCosts->find()
                ->where(
                    [
                        'record_id' => $recordId,
                        'stage' => $formData["tipo"],
                        'costs_type' => $tipoProcedimiento,
                    ]
                )
                ->first();


            if ($existingCost) {
                $this->Flash->error(
                    __(
                        'No se puede guardar la tasación '.
                        'de costas porque ya existe una tasación de '.
                        'costas del mismo tipo, con el mismo tipo de '.
                        'procedimiento y fase en el expediente.'
                    )
                );
                $destination = 'procedure-man';
                return $this->redirect(
                    [
                        'plugin'     => 'Records',
                        'controller' => 'Manage',
                        'action'     => 'edit',
                        $record->get('id'),
                        '#'          => $destination
                    ]
                );
            }
        }








        if ($formData['tipo'] == 'a favor') {
            if ($formData['fase'] == 'presentacion'
                || $formData['fase'] == 'traslado'
                || $formData['fase'] == 'decreto'
                || $formData['fase'] == 'resolucion'
            ) {
                if (empty($formData['fecha_presentacion'])) {
                    $errors[] = __('La fecha de presentación es obligatoria.');
                } else {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'fecha_presentacion',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de presentación '.
                            'no puede ser mayor a hoy.'
                        );
                    }
                }
                //arriaga tab optional validations
                if (!empty($formData['imp_costas'])
                    || !empty($formData['imp_perito'])
                    || !empty($formData['imp_testigo'])
                ) {
                    $optionalFields = [
                        'imp_costas' =>
                            __(
                                'El Importe Costas debe '.
                                'ser un importe positivo en '.
                                'caso de haberse informado.'
                            ),
                        'imp_perito' =>
                            __(
                                'El Importe Perito debe ser '.
                                'un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'imp_testigo' =>
                            __(
                                'El Importe Testigo debe ser un '.
                                'importe positivo en caso de '.
                                'haberse informado.'
                            ),
                    ];
                    $this->_validateOptionalFields(
                        $formData,
                        $optionalFields,
                        $errors
                    );
                }

                if (!empty($formData['fecha_calculo'])) {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'fecha_calculo',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de cálculo '.
                            'no puede ser mayor a hoy.'
                        );
                    }
                }


                //procurador tab optional validations
                if (!empty($formData['procurador_imp_costas'])
                    || !empty($formData['procurador_imp_perito'])
                    || !empty($formData['procurador_imp_testigo'])
                    || !empty($formData['procurador_imp_tasa'])
                ) {
                    $optionalFields = [
                        'procurador_imp_costas' =>
                            __(
                                'El Importe Procurador Costas '.
                                'debe ser un importe positivo en '.
                                'caso de haberse informado.'
                            ),
                        'procurador_imp_tasa' =>
                            __(
                                'El Importe Procurador Tasa debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'procurador_imp_perito' =>
                            __(
                                'El Importe Procurador Perito debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'procurador_imp_testigo' =>
                            __(
                                'El Importe Procurador Testigo debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                    ];
                    $this->_validateOptionalFields(
                        $formData,
                        $optionalFields,
                        $errors
                    );
                }

                if (!empty($formData['procurador_fecha_calculo'])) {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'procurador_fecha_calculo',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de cálculo en la '.
                            'pestaña procurador no puede '.
                            'ser mayor a hoy.'
                        );
                    }
                }
            }
        }
        if ($formData['fase'] == 'traslado'
            || $formData['fase'] == 'decreto'
            || $formData['fase'] == 'resolucion'
        ) {
            if (empty($formData['fecha_traslado'])) {
                $errors[] = __('La fecha de traslado es obligatoria.');
            } else {
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_traslado',
                    $errors
                )
                ) {
                }
            }

            //arriaga tab optional validations
            if (!empty($formData['importe_laj'])
                || !empty($formData['procurador_importe_laj'])
            ) {
                $optionalFields = [
                    'importe_laj' =>
                        __(
                            'El Importe LAJ debe '.
                            'ser un importe positivo.'
                        ),
                    'procurador_importe_laj' =>
                        __(
                            'El Importe LAJ en procurador '.
                            'debe ser un importe positivo '.
                            'en caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }

            if ($formData['impugnacion_traslado'] == 1) {
                if (empty($formData['tipo_impugnacion'])) {
                    $errors[] = __(
                        'El Tipo Impugnación es obligatorio en la '.
                        'pestaña Arriaga y debe ser uno de los siguientes: '.
                        'POR EXCESIVAS, POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion']
                ) ?
                    $formData['fecha_impugnacion'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es obligatoria y '.
                        'debe ser válida en la pestaña Arriaga.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha impugnación o traslado es '.
                            'inválida en la pestaña Arriaga.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación debe ser '.
                                'posterior a la fecha de '.
                                'traslado en la pestaña Arriaga.'
                            );
                        } else {
                            $formattedDateImpugnacion = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }


            if ($formData['procurador_impugnacion_traslado'] == 1) {
                if (empty($formData['procurador_tipo_impugnacion'])) {
                    $errors[] = __(
                        'El Tipo Impugnación en Procurador es '.
                        'obligatorio en la pestaña procurador y debe '.
                        'ser uno de los siguientes: POR EXCESIVAS, '.
                        'POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['procurador_fecha_impugnacion']
                ) ?
                    $formData['procurador_fecha_impugnacion'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es obligatoria '.
                        'y debe ser válida en la pestaña procurador.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha impugnación o traslado es inválida '.
                            'en la pestaña procurador.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                '
                            La fecha de impugnación debe ser posterior '.
                                'a la fecha de traslado en la pestaña procurador.'
                            );
                        } else {
                            $formattedDateImpugnacion = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }


            if ($formData['imp_contrario'] == 1) {
                if (empty($formData['tipo_impugnacion_contrario'])) {
                    $errors[] = __(
                        'El Tipo Impugnación contrario es obligatorio '.
                        'en la pestaña Arriaga y debe ser uno de los '.
                        'siguientes: POR EXCESIVAS, POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion_contrario']
                ) ?
                    $formData['fecha_impugnacion_contrario'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        '
                    La fecha de impugnación es obligatoria '.
                        'y debe ser válida en la pestaña Arriaga.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            '
                        Fecha de impugnación o traslado '.
                            'es incorrecta en la pestaña Arriaga.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación debe '.
                                'ser posterior a la fecha de '.
                                'traslado en la pestaña Arriaga.'
                            );
                        } else {
                            $formattedDateImpugnacionContrario = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }





            if ($formData['procurador_imp_contrario'] == 1) {
                if (empty($formData['procurador_tipo_impugnacion_contrario'])) {
                    $errors[] = __(
                        'El Tipo Impugnación contrario '.
                        'es obligatorio en la pestaña '.
                        'procurador y debe ser uno de '.
                        'los siguientes: POR EXCESIVAS, '.
                        'POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion_contrario']
                ) ?
                    $formData['fecha_impugnacion_contrario'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es '.
                        'obligatoria y debe ser válida '.
                        'en la pestaña procurador.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha de impugnación o '.
                            'traslado es incorrecta '.
                            'en la pestaña procurador.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación '.
                                    'debe ser posterior a la '.
                                    'fecha de traslado en la pestaña procurador.'
                            );
                        } else {
                            $formattedDateImpugnacionContrario = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }
            if ($formData['tipo'] == 'a favor') {
                if (empty($formData['fecha_traslado'])) {
                    $errors[] = __('La fecha de traslado es obligatoria.');
                } else {
                    if (!$this->validateDateNotGreater(
                        $formData,
                        'fecha_traslado',
                        'fecha_presentacion',
                        $errors
                    )
                    ) {
                    }
                }
            }

            if ($formData['tipo'] == 'en contra') {
            }
        }
        if ($formData['fase'] == 'decreto' || $formData['fase'] == 'resolucion') {
            if (empty($formData['fecha_decreto'])) {
                $errors[] = __('La fecha de decreto es obligatoria.');
            } else {
                if (!$this->validateDateNotGreater(
                    $formData,
                    'fecha_decreto',
                    'fecha_traslado',
                    $errors
                )
                ) {
                }
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_decreto',
                    $errors
                )
                ) {
                    $errors[] = __('La fecha de decreto no puede ser mayor a hoy.');
                }
            }



            if (!empty($formData['importe_decreto'])
                || !empty($formData['procurador_importe_decreto'])
            ) {
                $optionalFields = [
                    'importe_decreto' =>
                        __(
                            'El Importe Decreto en la '.
                            'pestaña Arriaga debe ser '.
                            'un importe positivo en '.
                            'caso de haberse informado.'
                        ),
                    'procurador_importe_decreto' =>
                        __(
                            'El Importe Decreto en procurador '.
                            'debe ser un importe positivo '.
                            'en caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }

            $this->_validateDateResource(
                $formData,
                'recurso',
                'fecha_recurso',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_recurso',
                'procurador_fecha_recurso',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'recurso_contrario',
                'fecha_recurso_contrario_decreto',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_recurso_contrario',
                'procurador_fecha_recurso_contrario_decreto',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'impugnacion_contrario_decreto',
                'fecha_impugnacion_contrario_decreto',
                'fecha_recurso',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_impugnacion_contrario_decreto',
                'procurador_fecha_impugnacion_contrario_decreto',
                'procurador_fecha_recurso',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'impugnacion_rc',
                'fecha_impugnacion_rc',
                'fecha_impugnacion_contrario_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_impugnacion_rc',
                'procurador_fecha_impugnacion_rc',
                'procurador_fecha_impugnacion_contrario_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña procurador.')
            );
            if ($formData['tipo'] == 'a favor') {
            }
            if ($formData['tipo'] == 'en contra') {
            }
        }

        if ($formData['fase'] == 'resolucion') {
            if (empty($formData['fecha_resolucion'])) {
                $errors[] = __('La fecha de resolución es obligatoria.');
            } else {
                if (!$this->validateDateNotGreater(
                    $formData,
                    'fecha_resolucion',
                    'fecha_decreto',
                    $errors
                )
                ) {
                }
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_resolucion',
                    $errors
                )
                ) {
                    $errors[] = __(
                        'La fecha de resolución '.
                        'no puede ser mayor a hoy.'
                    );
                }
            }

            if (!empty($formData['importe_resolucion'])
                || !empty($formData['procurador_importe_resolucion'])
            ) {
                $optionalFields = [
                    'importe_resolucion' =>
                        __(
                            'El Importe Resolucion en la '.
                            'pestaña Arriaga debe ser un importe '.
                            'positivo en caso de haberse informado.'
                        ),
                    'procurador_importe_resolucion' =>
                        __(
                            'El Importe Resolucion en procurador '.
                            'debe ser un importe positivo en '.
                            'caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }



            if ($formData['tipo'] == 'a favor') {
            }
            if ($formData['tipo'] == 'en contra') {
            }
        }

        $fechaPrimeraInstancia = isset(
            $record
                ->pro_fecha_sentencia_primera_instancia
        )
            ? $record->pro_fecha_sentencia_primera_instancia->format('d/m/Y')
            : null;

        $fechaPresentacion = isset($formData['fecha_presentacion'])
            ? $formData['fecha_presentacion']
            : null;

        if (isset($formData['tipo_procedimiento'])
            && $formData['tipo_procedimiento'] === 'primera instancia'
            && $formData['tipo'] !== 'en contra'
            && $fechaPrimeraInstancia != null
        ) {
            if ($fechaPrimeraInstancia && $fechaPresentacion) {
                $datePrimeraInstancia = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPrimeraInstancia
                );
                $datePresentacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPresentacion
                );


                if ($datePrimeraInstancia && $datePresentacion) {
                    if ($datePresentacion < $datePrimeraInstancia) {
                        $errors[] = __(
                            'La fecha de presentación no puede '.
                            'ser menor que la fecha de '.
                            'sentencia de primera instancia.'
                        );
                    }
                } else {
                    $errors[] = __('Una o ambas fechas son inválidas.');
                }
            } else {
                if (empty($fechaPrimeraInstancia)) {
                    $errors[] = __(
                        'La fecha de sentencia de ' .
                        'primera instancia no está definida.'
                    );
                }
                if (empty($fechaPresentacion)) {
                    $errors[] = __('La fecha de presentación no está definida.');
                }
            }
        }

        $fechaApelacion = isset($record->ape_apelacion)
            ? $record->ape_apelacion->format('d/m/Y')
            : null;

        $fechaPresentacion = isset($formData['fecha_presentacion'])
            ? $formData['fecha_presentacion']
            : null;

        if (isset($formData['tipo_procedimiento'])
            && $formData['tipo_procedimiento'] === 'apelacion'
            && $formData['tipo'] !== 'en contra'
            && $fechaApelacion != null
        ) {
            if ($fechaApelacion && $fechaPresentacion) {
                $dateApelacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaApelacion
                );
                $datePresentacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPresentacion
                );


                if ($dateApelacion && $datePresentacion) {
                    if ($datePresentacion < $dateApelacion) {
                        $errors[] = __(
                            'La fecha de presentación no puede' .
                            ' ser menor que la fecha de apelación.'
                        );
                    }
                } else {
                    $errors[] = __('Una o ambas fechas son inválidas.');
                }
            } else {
                if (empty($fechaApelacion)) {
                    $errors[] = __('La fecha de apelación no está definida.');
                }
                if (empty($fechaPresentacion)) {
                    $errors[] = __('La fecha de presentación no está definida.');
                }
            }
        }




        if (!empty($errors)) {
            foreach ($errors as $error) {
                $this->Flash->error($error);
            }
            $destination = 'procedure-man';
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $record->get('id'),
                    '#'          => $destination
                ]
            );


            // $this->redirect($this->referer());
        } else {
            $connCourtCosts = $this->CourtCosts->connection();
            $connCourtCosts->begin();


            try {
                $courtCosts = $this->CourtCosts->newEntity(
                    [
                        'record_id' => $recordId,
                        'stage' =>
                            isset($formData["tipo"]) ?
                            $formData["tipo"] : null,
                        'costs_type' =>
                            isset($formData["tipo_procedimiento"]) ?
                            $formData["tipo_procedimiento"] : null,
                        'procedure_type' =>
                            isset($formData["fase"]) ?
                            $formData["fase"] : null,
                        'presentation_date' =>
                            !empty($formData['fecha_presentacion']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_presentacion'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'transfer_date' =>
                            !empty($formData['fecha_traslado']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_traslado'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_date' =>
                            !empty($formData['fecha_decreto']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'ruling_date' =>
                            !empty($formData['fecha_resolucion']) ?
                             date(
                                 'Y-m-d',
                                 strtotime(
                                     \DateTime::createFromFormat(
                                         'd/m/Y',
                                         $formData[
                                            'fecha_resolucion'
                                         ]
                                     )
                                     ->format('Y-m-d')
                                 )
                             ) : null,
                    ]
                );
                if (!$this->CourtCosts->save($courtCosts)) {
                    throw new \Exception('Error al guardar CourtCosts');
                }


                $courtID = $courtCosts->id;
                $this->_tagsCourtCosts($courtCosts);


                $lcc = $this->LawyerCourtCosts->newEntity(
                    [
                        'court_costs_id' => $courtID,

                        //presentation data
                        'fee_amount' =>
                            !empty($formData["imp_costas"]) ?
                            $formData["imp_costas"] : 0,
                        'proficient_amount' =>
                            !empty($formData["imp_perito"]) ?
                            $formData["imp_perito"] : 0,
                        'witness_amount' =>
                            !empty($formData["imp_testigo"]) ?
                            $formData["imp_testigo"] : 0,
                        'calculation_date' =>
                            !empty($formData['fecha_calculo'])
                                ? \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData[
                                        'fecha_calculo'
                                    ]
                                )
                                : null,

                        //transfer data
                        'taxation_result' =>
                            isset(
                                $formData["resultado_tasacion"]
                            ) ?
                                $formData["resultado_tasacion"] : null,
                        'laj_amount' =>
                            !empty($formData["importe_laj"]) ?
                            $formData["importe_laj"] : 0,
                        'laj_requested_difference_amount' =>
                            !empty($formData["laj_requested_difference_amount"]) ?
                            $formData["laj_requested_difference_amount"] : 0,
                            'ica_report' => $formData['informe_ica'] == 1 ? 1 : 0,
                        'ica_report_date' =>
                            $formData['informe_ica'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_ica'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'own_challenge' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            1 : 0,
                        'own_challenge_type' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            $formData["tipo_impugnacion"] : null,
                        'own_challenge_date' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_impugnacion'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'opponent_challenge' =>
                            $formData['imp_contrario'] == 1 ?
                            1 : 0,
                        'opponent_challenge_type' =>
                            $formData['imp_contrario'] == 1 ?
                            $formData["tipo_impugnacion_contrario"] : null,
                        'opponent_challenge_date' =>
                            $formData['imp_contrario'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_impugnacion_contrario'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'opponent_challenge_acceptance' =>
                            $formData['conformidad_imp_contrario'] == 'yes' ?
                            1 : 0,


                        //decree data
                        'decree_result' =>
                            isset(
                                $formData["resultado_decreto"]
                            ) ?
                                $formData["resultado_decreto"] : null,
                        'decree_amount' =>
                            !empty($formData["importe_decreto"]) ?
                            $formData['importe_decreto'] : 0,
                        'decree_requested_difference_amount' =>
                            !empty($formData["decree_dif"]) ?
                            $formData['decree_dif'] : 0,
                        'decree_own_appeal' =>
                            $formData['recurso'] == 1 ? 1 : 0,
                        'decree_own_appeal_date' =>
                            $formData['recurso'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_recurso'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_opponent_challenge_to_own_appeal' =>
                            $formData['impugnacion_contrario_decreto'] == 1 ?
                            1 : 0,
                        'decree_opponent_challenge_to_own_appeal_date' =>
                            $formData['impugnacion_contrario_decreto'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_impugnacion_contrario_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_opponent_appeal' =>
                            $formData['recurso_contrario'] == 1 ? 1 : 0,
                        'decree_opponent_appeal_date' =>
                            $formData['recurso_contrario'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_recurso_contrario_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_own_challenge_to_opponent_appeal' =>
                            $formData['impugnacion_rc'] == 1 ?
                            1 : 0,
                        'decree_own_challenge_to_opponent_appeal_date' =>
                            $formData['impugnacion_rc'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_impugnacion_rc'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_favorable_costs' =>
                            !empty($formData['costas_incidente_decreto']) ?
                            1 : 0,
                        'decree_against_costs' =>
                            !empty($formData['costas_ec_incidente_decreto']) ?
                            1 : 0,
                        'own_allegations_to_opponent_challenge' =>
                            isset(
                                $formData["alegaciones_imp_contrario"]
                            ) ?
                                $formData["alegaciones_imp_contrario"] : false,

                        //ruling data
                        'ruling_result' =>
                            isset(
                                $formData["resultado_resolucion"]
                            ) ?
                                $formData["resultado_resolucion"] : null,
                        'ruling_amount' =>
                            !empty($formData["importe_resolucion"]) ?
                            $formData["importe_resolucion"] : 0,
                        'ruling_requested_difference_amount' =>
                            !empty($formData["ruling_dif"]) ?
                            $formData["ruling_dif"] : 0,
                        'ruling_favorable_costs' =>
                            !empty($formData['costas_incidente']) ?
                            1 : 0,
                        'ruling_against_costs' =>
                            !empty($formData['costas_ec_incidente']) ?
                            1 : 0,

                    ]
                );
                if (!$this->LawyerCourtCosts->save($lcc)) {
                    throw new \Exception('Error al guardar LawyerCourtCosts');
                }
                $pro = $this->AttorneyCourtCosts->newEntity(
                    [
                        'court_costs_id' => $courtID,

                        //presentation data
                        'fee_amount' => !empty(
                            $formData["procurador_imp_costas"]
                        ) ?
                            $formData["procurador_imp_costas"] : 0,
                        'proficient_amount' => !empty(
                            $formData["procurador_imp_perito"]
                        ) ?
                            $formData["procurador_imp_perito"] : 0,
                        'witness_amount' =>
                        !empty($formData["procurador_imp_testigo"]) ?
                        $formData["procurador_imp_testigo"] : 0,
                        'tax_amount' => !empty(
                            $formData["procurador_imp_tasa"]
                        ) ?
                            $formData["procurador_imp_tasa"] : 0,
                        'calculation_date' => !empty(
                            $formData['procurador_fecha_calculo']
                        ) ?
                             \DateTime::createFromFormat(
                                 'd/m/Y',
                                 $formData[
                                    'procurador_fecha_calculo'
                                 ]
                             ) : null,

                        //transfer data
                        'taxation_result' => isset(
                            $formData["procurador_resultado_tasacion"]
                        ) ?
                            $formData["procurador_resultado_tasacion"] : null,
                        'laj_amount' =>
                        !empty(
                            $formData[
                                "procurador_importe_laj"
                            ]
                        ) ?
                                 $formData[
                                    "procurador_importe_laj"
                                    ] : 0,
                        'laj_requested_difference_amount' =>
                        !empty(
                            $formData[
                                "procurador_laj_requested_difference_amount"
                            ]
                        ) ?
                                $formData[
                                    "procurador_laj_requested_difference_amount"
                                    ] : 0,
                        'own_challenge' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         1 : 0,
                        'own_challenge_type' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         $formData["procurador_tipo_impugnacion"] : null,
                        'own_challenge_date' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion'
                                     ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'opponent_challenge' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         1 : 0,
                        'opponent_challenge_type' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         $formData[
                            "procurador_tipo_impugnacion_contrario"
                            ] : null,
                        'opponent_challenge_date' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion_contrario'
                                        ]
                                 )->format('Y-m-d')
                             )
                         ) : null,
                        'opponent_challenge_acceptance' =>
                        $formData[
                            'procurador_conformidad_imp_contrario'
                            ] == 'yes' ?
                         1 : 0,


                        //decree data
                        'decree_result' => isset(
                            $formData["procurador_resultado_decreto"]
                        ) ?
                             $formData["procurador_resultado_decreto"] : null,
                        'decree_amount' =>
                        !empty($formData["procurador_importe_decreto"]) ?
                         $formData['procurador_importe_decreto'] : 0,
                        'decree_requested_difference_amount' =>
                        !empty($formData["procurador_decree_dif"]) ?
                         $formData['procurador_decree_dif'] : 0,
                        'decree_own_appeal' =>
                        $formData['procurador_recurso'] == 1 ?
                         1 : 0,
                        'decree_own_appeal_date' =>
                        $formData['procurador_recurso'] == 1 ?
                        date(
                            'Y-m-d',
                            strtotime(
                                \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData[
                                        'procurador_fecha_recurso'
                                    ]
                                )
                                ->format('Y-m-d')
                            )
                        ) : null,
                        'decree_opponent_challenge_to_own_appeal' =>
                        $formData['procurador_impugnacion_contrario_decreto'] == 1 ?
                         1 : 0,
                        'decree_opponent_challenge_to_own_appeal_date' =>
                        $formData['procurador_impugnacion_contrario_decreto'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion' .
                                        '_contrario_decreto'
                                     ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'decree_opponent_appeal' =>
                        $formData['procurador_recurso_contrario'] == 1 ?
                         1 : 0,
                        'decree_opponent_appeal_date' =>
                        $formData['procurador_recurso_contrario'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_recurso_contrario_decreto'
                                        ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'decree_own_challenge_to_opponent_appeal' =>
                        $formData['procurador_impugnacion_rc'] == 1 ?
                         1 : 0,
                        'decree_own_challenge_to_opponent_appeal_date' =>
                        $formData['procurador_impugnacion_rc'] == 1 ?
                        date(
                            'Y-m-d',
                            strtotime(
                                \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData['procurador_fecha_impugnacion_rc']
                                )
                                ->format('Y-m-d')
                            )
                        ) : null,
                        'decree_favorable_costs' =>
                        !empty($formData['procurador_costas_incidente_decreto']) ?
                            1 : 0,
                        'decree_against_costs' =>
                        !empty($formData['procurador_costas_ec_incidente_decreto']) ?
                            1 : 0,
                        'own_allegations_to_opponent_challenge' => isset(
                            $formData["procurador_alegaciones_imp_contrario"]
                        ) ?
                            $formData["procurador_alegaciones_imp_contrario"]
                            : false,

                        //ruling data
                        'ruling_result' =>
                        isset(
                            $formData["procurador_procurador_resultado_resolucion"]
                        ) ?
                            $formData["procurador_resultado_resolucion"] : null,
                        'ruling_amount' =>
                        !empty($formData["procurador_importe_resolucion"]) ?
                            $formData["procurador_importe_resolucion"] : 0,
                        'ruling_requested_difference_amount' =>
                        !empty($formData["procurador_ruling_dif"]) ?
                            $formData["procurador_ruling_dif"] : 0,
                        'ruling_favorable_costs' =>
                        !empty($formData['procurador_costas_incidente']) ? 1 : 0,
                        'ruling_against_costs' =>
                        !empty($formData['procurador_costas_ec_incidente']) ? 1 : 0,

                    ]
                );
                if (!$this->AttorneyCourtCosts->save($pro)) {
                    throw new \Exception('Error al guardar AttorneyCourtCosts');
                }
                $connCourtCosts->commit();
                $this->Flash->success(
                    __(
                        'La tasación de costas se '.
                        'ha guardado correctamente.'
                    )
                );
            } catch (\Exception $e) {
                $connCourtCosts->rollback();
                $this->Flash->error(
                    __(
                        'No se ha podido guardar la '.
                        'tasación de costas. Error: ' .
                        $e->getMessage()
                    )
                );
                return $this->redirect(
                    [
                        'plugin'     => 'Records',
                        'controller' => 'Manage',
                        'action'     => 'edit',
                        $record->get('id'),
                        '#'          => $destination
                    ]
                );
            }
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $record->get('id'),
                    '#'          => $destination
                ]
            );
        }
        return $this->redirect(
            [
                'plugin'     => 'Records',
                'controller' => 'Manage',
                'action'     => 'edit',
                $record->get('id'),
                '#'          => $destination
            ]
        );
    }

    /**
     * Valida que una fecha no sea anterior a una fecha de referencia.
     *
     * Esta función compara una fecha a validar con una fecha de referencia.
     * Si la fecha a validar es anterior o igual a la fecha de referencia,
     * se añade un mensaje de error al arreglo de errores.
     *
     * @param string $dateToValidate La fecha a validar.
     * @param string $referenceDate  La fecha de referencia para la validación.
     * @param string $errorMessage   El mensaje de error que se añadirá si la
     *                               validación falla.
     * @param array  $errors         El arreglo donde se agregarán los errores
     *                               si la validación falla.
     *
     * @return void
     */
    private function _validateDateNotEarlier(
        $dateToValidate,
        $referenceDate,
        $errorMessage,
        &$errors
    ) {
    

        if (!empty($dateToValidate) && !empty($referenceDate)) {
            if ($dateToValidate <= $referenceDate) {
                $errors[] = $errorMessage;
            }
        }
    }
    /**
     * Valida la fecha de recurso en función de un checkbox y una fecha de
     * referencia.
     *
     * Esta función verifica si el checkbox está marcado y, si es así,
     * valida que la fecha de recurso esté informada y sea posterior a la
     * fecha de referencia, si corresponde. Si el checkbox no está marcado,
     * se asegura de que la fecha de recurso no esté informada.
     *
     * @param array  $formData             Los datos del formulario.
     * @param string $checkboxName         El nombre del checkbox para validar.
     * @param string $fechaField           El nombre del campo de la
     *                                     fecha de recurso.
     * @param string $fechaReferenciaField El nombre del campo
     *                                     de la fecha de referencia.
     * @param array  $errors               El arreglo donde se agregarán los errores
     *                                     si la validación falla.
     * @param string $errorMessage         El mensaje de error si la fecha de recurso
     *                                     no está informada.
     *
     * @return void
     */
    private function _validateDateResource(
        $formData,
        $checkboxName,
        $fechaField,
        $fechaReferenciaField,
        &$errors,
        $errorMessage
    ) {
        
    

        $fechaRecurso = isset(
            $formData[$fechaField]
        ) ?
            $formData[$fechaField] : null;
        $fechaReferencia = isset(
            $formData[$fechaReferenciaField]
        ) ?
            $formData[$fechaReferenciaField] : null;

        if (!empty($formData[$checkboxName])) {
            if (empty($fechaRecurso)) {
                $errors[] = $errorMessage;
            } else {
                if ($this->_validateDate(
                    $fechaRecurso,
                    __(
                        'La fecha de recurso debe ser válida.'
                    ),
                    $errors
                )
                ) {
                    if (!empty($fechaReferencia)
                        && $fechaRecurso < $fechaReferencia
                    ) {
                        $errors[] = __(
                            'La fecha' .
                            $fechaRecurso .
                            ' debe ser posterior a ' .
                            $fechaReferencia
                        );
                    }
                }
            }
        } else {
            if (!empty($fechaRecurso)) {
                $errors[] = __(
                    'La fecha de recurso no debe '.
                    'estar informada si el recurso no está marcado.'
                );
            }
        }
    }
        /**
     * Valida que la fecha no esté vacía y tenga un formato correcto.
     *
     * Esta función comprueba que el campo de la fecha no esté vacío
     * y que su formato sea válido (DD/MM/YYYY). Si la fecha no es
     * válida o está vacía, se agrega un mensaje de error al arreglo
     * de errores.
     *
     * @param string $date   La fecha a validar.
     * @param array  $errors El arreglo donde se agregarán los errores
     *                       si la validación falla.
     *
     * @return bool Retorna true si la fecha es válida, false en caso contrario.
     */
    private function _validateDate($date, $errors)
    {
        if (empty($date)) {
            $errors[] = __('La fecha es obligatoria.');
            return false;
        }

        $dateParts = explode('/', $date);
        if (count($dateParts) === 3) {
            return true;
        }

        $errors[] = __('El formato de la fecha es inválido.');
        return false;
    }
        /**
     * Valida los campos opcionales en el formulario.
     *
     * Esta función revisa si existen campos opcionales en el formulario
     * y si tienen valores no vacíos. Si es así, se validan mediante
     * la función `_validateSelectFields`. Si se encuentran errores,
     * se agrega un mensaje de error.
     *
     * @param array $formData       Los datos del formulario a validar.
     * @param array $optionalFields Los campos opcionales con sus
     *                               respectivos mensajes de error.
     * @param array $errors         El arreglo donde se agregarán los errores
     *                              si la validación falla.
     *
     * @return void
     */
    private function _validateOptionalFields($formData, $optionalFields, &$errors)
    {

        $optionalNumberValidate = [];


        foreach ($optionalFields as $field => $errorMessage) {
            if (array_key_exists($field, $formData) && !empty($formData[$field])) {
                $optionalNumberValidate[$field] = $errorMessage;
            }
        }


        if (!empty($optionalNumberValidate)) {
            if (!$this->_validateSelectFields(
                $formData,
                $optionalNumberValidate,
                $errors
            )
            ) {
                $this->Flash->error(
                    __(
                        'Se encontraron errores '.
                        'en los campos de selección.'
                    )
                );
            }
        }
    }


    /**
     * Valida que la fecha no sea futura.
     *
     * Esta función verifica si el campo de fecha proporcionado en
     * `$dateKey` es una fecha válida y si esa fecha es anterior
     * o igual a la fecha actual. Si la fecha es mayor que la fecha
     * actual, se retorna `false`, indicando que la fecha no es válida.
     *
     * @param array  $formData Los datos del formulario a validar.
     * @param string $dateKey  La clave del campo de fecha.
     * @param array  $errors   El arreglo donde se agregarán los errores
     *                         si la validación falla.
     *
     * @return bool Retorna `true` si la fecha es válida y no es futura,
     *              o `false` si la fecha es mayor a la fecha actual.
     */
    private function _validateNotFutureDate($formData, $dateKey, &$errors)
    {

        if (!empty($formData[$dateKey])) {
            $today = new \DateTime();


            $dateParts = explode('/', $formData[$dateKey]);
            if (count($dateParts) === 3) {
                $dateFormatted = sprintf(
                    '%04d-%02d-%02d',
                    $dateParts[2],
                    $dateParts[1],
                    $dateParts[0]
                );
                $dateToValidate = new \DateTime($dateFormatted);


                if ($dateToValidate >= $today) {
                    return false;
                }
            } else {
                $errors[] = __('Formato de fecha inválido para ' . $dateKey . '.');
                return false;
            }
        }

        return true;
    }
    /**
     * Valida que una fecha no sea mayor que otra en los datos del formulario.
     *
     * Esta función verifica si ambos campos de fecha tienen valores válidos
     * y si la fecha especificada en `$dateKey1` es mayor o igual que la
     * fecha en `$dateKey2`. Si alguna de las condiciones no se cumple,
     * se agrega un mensaje de error al arreglo `$errors`.
     *
     * @param array  $formData Los datos del formulario a validar.
     * @param string $dateKey1 La clave del primer campo de fecha.
     * @param string $dateKey2 La clave del segundo campo de fecha.
     * @param array  $errors   El arreglo donde se agregarán los errores
     *                         si la validación falla.
     *
     * @return bool Retorna `true` si las fechas son válidas, o `false` si
     *              alguna de las validaciones falla.
     */
    function validateDateNotGreater($formData, $dateKey1, $dateKey2, &$errors)
    {

        if (!empty($formData[$dateKey1]) && !empty($formData[$dateKey2])) {
            $dateParts1 = explode('/', $formData[$dateKey1]);
            $dateParts2 = explode('/', $formData[$dateKey2]);

            if (count($dateParts1) === 3 && count($dateParts2) === 3) {
                $dateFormatted1 = sprintf(
                    '%04d-%02d-%02d',
                    $dateParts1[2],
                    $dateParts1[1],
                    $dateParts1[0]
                );
                $dateFormatted2 = sprintf(
                    '%04d-%02d-%02d',
                    $dateParts2[2],
                    $dateParts2[1],
                    $dateParts2[0]
                );

                $dateToValidate1 = new \DateTime($dateFormatted1);
                $dateToValidate2 = new \DateTime($dateFormatted2);


                if ($dateToValidate1 < $dateToValidate2) {
                    $errors[] = __(
                        'La fecha ' .
                        $dateKey1 .
                        ' debe ser mayor o igual que la fecha ' .
                        $dateKey2 . '.'
                    );
                    return false;
                }
            } else {
                $errors[] = __(
                    'Formato de fecha inválido para ' .
                    $dateKey1 . ' o ' .
                    $dateKey2 . '.'
                );
                return false;
            }
        }

        return true;
    }
    /**
     * Valida que los campos de selección no estén
     * vacíos en los datos del formulario.
     *
     * Esta función recorre los campos especificados en `$fields`
     * y verifica que los valores
     * correspondientes en `$formData` no estén vacíos.
     * Si algún campo está vacío, se agrega
     * un mensaje de error al arreglo
     * de errores y se retorna `false`.
     *
     * @param array $formData Los datos del formulario a validar.
     * @param array $fields   Un arreglo asociativo donde las claves son
     *                        los nombres de los campos y los valores
     *                        son los mensajes de error
     *                        correspondientes.
     * @param array $errors   El arreglo donde se agregarán los mensajes
     *                        de error si se encuentran problemas.
     *
     * @return bool Retorna `true` si todos los campos
     *                       son válidos, o `false` si alguno no es válido.
     */
    private function _validateSelectFields($formData, $fields, &$errors)
    {
        foreach ($fields as $field => $errorMessage) {
            if (empty($formData[$field])) {
                $errors[] = __($errorMessage);
                return false;
            }
        }
        return true;
    }
    /**
     * Valida que los campos numéricos en los
     * datos del formulario sean mayores o iguales a 0.
     *
     * Esta función recorre los campos especificados
     * y verifica que sus valores no estén vacíos
     * y sean mayores o iguales a 0. Si alguno de
     * los valores no cumple con estas condiciones,
     * se agrega un mensaje de error al arreglo
     * de errores y se retorna `false`.
     *
     * @param array $formData Los datos del formulario a validar.
     * @param array $fields   Un arreglo asociativo donde las claves
     *                        son los nombres de los campos y los
     *                        valores son los mensajes de error
     *                        correspondientes.
     * @param array $errors   El arreglo donde se agregarán los mensajes
     *                        de error si se encuentran problemas.
     *
     * @return bool Retorna `true` si todos los campos
     *                       son válidos, o `false` si alguno no es válido.
     */
    private function _validateNumericFields($formData, $fields, &$errors)
    {
        foreach ($fields as $field => $errorMessage) {
            $value = isset($formData[$field]) ? $formData[$field] : null;

            if (empty($value) || $value < 0) {
                $errors[] = $errorMessage;
                return false;
            }
        }
        return true;
    }
        /**
     * Elimina una tasación de costas y sus
     * registros asociados en diferentes tablas.
     *
     * Esta función elimina una tasación de costas
     * (`CourtCosts`) y los registros relacionados
     * en las tablas `Income`, `LawyerCourtCosts` y
     * `AttorneyCourtCosts`. Si algún registro no
     * se puede eliminar, se muestra un mensaje de error.
     * Si la eliminación es exitosa, se
     * elimina la tasación de costas y se actualizan las
     * etiquetas del registro correspondiente.
     *
     * @param int $courtCostId El ID de la tasación de costas a eliminar.
     *
     * @return \Cake\Network\Response Redirige a la página de
     * edición del registro con un mensaje de éxito o error.
     */
    public function deleteTC($courtCostId)
    {
        $destination = 'procedure-man';
        $this->loadModel('Income');
        $this->loadModel('CourtCosts');
        $this->loadModel('Records.Records');
        $this->loadModel('LawyerCourtCosts');
        $this->loadModel('AttorneyCourtCosts');

        $courtCost = $this->CourtCosts->get($courtCostId);


        $recordId = $courtCost->record_id;


        $incomeRecords = $this->Income->find()
            ->where(['record_id' => $recordId])
            ->andWhere(['court_costs_id' => $courtCostId])
            ->all();




        foreach ($incomeRecords as $income) {
            if (!$this->Income->delete($income)) {
                $this->Flash->error(
                    __(
                        'No se pudo eliminar el ingreso '.
                        'asociado. Por favor, inténtelo de nuevo.'
                    )
                );

                return $this->redirect(
                    [
                        'plugin'     => 'Records',
                        'controller' => 'Manage',
                        'action'     => 'edit',
                        $recordId,
                        '#'          => $destination
                    ]
                );
            }
        }


        $lawyerCourtCost = $this->LawyerCourtCosts->find()
            ->where(['court_costs_id' => $courtCostId])
            ->first();


        if ($lawyerCourtCost && !$this->LawyerCourtCosts->delete($lawyerCourtCost)) {
            $this->Flash->error(
                __(
                    'No se pudo eliminar el registro'.
                    ' relacionado en LawyerCourtCosts.'
                )
            );

            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }


        $attorneyCourtCosts = $this->AttorneyCourtCosts->find()
            ->where(['court_costs_id' => $courtCostId])
            ->all();

        foreach ($attorneyCourtCosts as $attorneyCourtCost) {
            if (!$this->AttorneyCourtCosts->delete($attorneyCourtCost)) {
                $this->Flash->error(
                    __(
                        'No se pudo eliminar el registro '.
                        'relacionado en attorney_court_costs. '.
                        'Por favor, inténtelo de nuevo.'
                    )
                );
                return $this->redirect(
                    [
                        'plugin'     => 'Records',
                        'controller' => 'Manage',
                        'action'     => 'edit',
                        $recordId,
                        '#'          => $destination
                    ]
                );
            }
        }


        if ($this->CourtCosts->delete($courtCost)) {
            $record = $this->Records->get($recordId);
            if ($courtCost->get('costs_type') == 'primera instancia') {
                if ($record->hasTag('COSTAS ORDINARIO TASADAS')) {
                    $record->removeTag('COSTAS ORDINARIO TASADAS');
                }
                if ($record->hasTag('COSTAS TASADAS PRESENTADAS')) {
                    $record->removeTag('COSTAS TASADAS PRESENTADAS');
                }
            }
            if ($courtCost->get('costs_type') == 'apelacion') {
                if ($record->hasTag('COSTAS APELACIÓN TASADAS')) {
                    $record->removeTag('COSTAS APELACIÓN TASADAS');
                }
            }
            $this->Flash->success(
                __(
                    'La tasación de costas '.
                    'ha sido eliminada con éxito.'
                )
            );
        } else {
            $this->Flash->error(
                __(
                    'No se pudo eliminar la tasación '.
                    'de costas. Por favor, inténtelo de nuevo.'
                )
            );
        }



        return $this->redirect(
            [
                'plugin'     => 'Records',
                'controller' => 'Manage',
                'action'     => 'edit',
                $record->get('id'),
                '#'          => $destination
            ]
        );
    }
    /**
     * Actualiza la tasación de costas para el proceso
     * y valida los datos del formulario.
     * Realiza las validaciones necesarias
     * para las fechas y los importes, y luego
     * realiza el `patch` en las tablas correspondientes.
     * Si algo sale mal, se realiza
     * un rollback de la transacción.
     *
     * @return \Cake\Network\Response Redirige al usuario
     * a la página de edición si algo falla.
     */
    public function patchTC()
    {
        $errors = [];
        $destination = 'procedure-man';
        $this->loadModel('CourtCosts');
        $this->loadModel('Records.Records');
        $this->loadModel('LawyerCourtCosts');
        $this->loadModel('AttorneyCourtCosts');


        $formData = $this->request->data();



        $courtCostId = $formData['court_cost_id'];
        $recordId = $formData['record_id'];


        $courtCost = $this->CourtCosts->findById($courtCostId)->first();
        if (!$courtCost) {
            $this->Flash->error(__('No se encontró la tasación de costas'));
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }


        $record = $this->Records->findById($recordId)->first();
        if (!$record) {
            $this->Flash->error(
                __(
                    'No se encontró la '.
                    'tasación de costas.'
                )
            );
            $destination = 'procedure-man';
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }

        $lawsuitCost = $this->LawyerCourtCosts
            ->findByCourtCostsId($courtCostId)
            ->first();

        if (!$lawsuitCost) {
            $this->Flash->error(
                __(
                    'No se encontró el registro '.
                    'de LawyerCourtCosts asociado.'
                )
            );

            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }

        $attorneyCost = $this->AttorneyCourtCosts
            ->findByCourtCostsId($courtCostId)
            ->first();

        if (!$attorneyCost) {
            $this->Flash->error(
                __(
                    'No se encontró el registro '.
                    'de AttorneyCourtCosts asociado.'
                )
            );
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }


        $tipoProcedimiento = isset(
            $formData["tipo_procedimiento"]
        ) ?
            $formData["tipo_procedimiento"] : null;

        // Check if the tipoProcedimiento is not 'incidente'
        if ($tipoProcedimiento !== 'incidente') {
            if ($courtCost->stage !== $formData["tipo"]
                || $courtCost->costs_type !== $tipoProcedimiento
            ) {
                $existingCost = $this->CourtCosts->find()
                    ->where(
                        [
                            'record_id' => $recordId,
                            'stage' => $formData["tipo"],
                            'costs_type' => $tipoProcedimiento,
                        ]
                    )
                    ->first();

                if ($existingCost) {
                    $this->Flash->error(
                        __(
                            'Ya existe una tasación de '.
                            'costas del mismo tipo, tipo de '.
                            'procedimiento y fase para este registro.'
                        )
                    );
                    return $this->redirect(
                        [
                            'plugin'     => 'Records',
                            'controller' => 'Manage',
                            'action'     => 'edit',
                            $recordId,
                            '#'          => $destination
                        ]
                    );
                }
            }
        }
        if ($formData['tipo'] == 'a favor') {
            if ($formData['fase'] == 'presentacion'
                || $formData['fase'] == 'traslado'
                || $formData['fase'] == 'decreto'
                || $formData['fase'] == 'resolucion'
            ) {
                if (empty($formData['fecha_presentacion'])) {
                    $errors[] = __('La fecha de presentación es obligatoria.');
                } else {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'fecha_presentacion',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de presentación '.
                            'no puede ser mayor a hoy.'
                        );
                    }
                }
                //arriaga tab optional validations
                if (!empty($formData['imp_costas'])
                    || !empty($formData['imp_perito'])
                    || !empty($formData['imp_testigo'])
                ) {
                    $optionalFields = [
                        'imp_costas' =>
                            __(
                                'El Importe Costas debe '.
                                'ser un importe positivo en '.
                                'caso de haberse informado.'
                            ),
                        'imp_perito' =>
                            __(
                                'El Importe Perito debe ser '.
                                'un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'imp_testigo' =>
                            __(
                                'El Importe Testigo debe ser un '.
                                'importe positivo en caso de '.
                                'haberse informado.'
                            ),
                    ];
                    $this->_validateOptionalFields(
                        $formData,
                        $optionalFields,
                        $errors
                    );
                }

                if (!empty($formData['fecha_calculo'])) {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'fecha_calculo',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de cálculo '.
                            'no puede ser mayor a hoy.'
                        );
                    }
                }


                //procurador tab optional validations
                if (!empty($formData['procurador_imp_costas'])
                    || !empty($formData['procurador_imp_perito'])
                    || !empty($formData['procurador_imp_testigo'])
                    || !empty($formData['procurador_imp_tasa'])
                ) {
                    $optionalFields = [
                        'procurador_imp_costas' =>
                            __(
                                'El Importe Procurador Costas '.
                                'debe ser un importe positivo en '.
                                'caso de haberse informado.'
                            ),
                        'procurador_imp_tasa' =>
                            __(
                                'El Importe Procurador Tasa debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'procurador_imp_perito' =>
                            __(
                                'El Importe Procurador Perito debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                        'procurador_imp_testigo' =>
                            __(
                                'El Importe Procurador Testigo debe '.
                                'ser un importe positivo en caso '.
                                'de haberse informado.'
                            ),
                    ];
                    $this->_validateOptionalFields(
                        $formData,
                        $optionalFields,
                        $errors
                    );
                }

                if (!empty($formData['procurador_fecha_calculo'])) {
                    if (!$this->_validateNotFutureDate(
                        $formData,
                        'procurador_fecha_calculo',
                        $errors
                    )
                    ) {
                        $errors[] = __(
                            'La fecha de cálculo en la '.
                            'pestaña procurador no puede '.
                            'ser mayor a hoy.'
                        );
                    }
                }
            }
        }
        if ($formData['fase'] == 'traslado'
            || $formData['fase'] == 'decreto'
            || $formData['fase'] == 'resolucion'
        ) {
            if (empty($formData['fecha_traslado'])) {
                $errors[] = __('La fecha de traslado es obligatoria.');
            } else {
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_traslado',
                    $errors
                )
                ) {
                }
            }

            //arriaga tab optional validations
            if (!empty($formData['importe_laj'])
                || !empty($formData['procurador_importe_laj'])
            ) {
                $optionalFields = [
                    'importe_laj' =>
                        __(
                            'El Importe LAJ debe '.
                            'ser un importe positivo.'
                        ),
                    'procurador_importe_laj' =>
                        __(
                            'El Importe LAJ en procurador '.
                            'debe ser un importe positivo '.
                            'en caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }

            if ($formData['impugnacion_traslado'] == 1) {
                if (empty($formData['tipo_impugnacion'])) {
                    $errors[] = __(
                        'El Tipo Impugnación es obligatorio en la '.
                        'pestaña Arriaga y debe ser uno de los siguientes: '.
                        'POR EXCESIVAS, POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion']
                ) ?
                    $formData['fecha_impugnacion'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es obligatoria y '.
                        'debe ser válida en la pestaña Arriaga.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha impugnación o traslado es '.
                            'inválida en la pestaña Arriaga.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación debe ser '.
                                'posterior a la fecha de '.
                                'traslado en la pestaña Arriaga.'
                            );
                        } else {
                            $formattedDateImpugnacion = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }


            if ($formData['procurador_impugnacion_traslado'] == 1) {
                if (empty($formData['procurador_tipo_impugnacion'])) {
                    $errors[] = __(
                        'El Tipo Impugnación en Procurador es '.
                        'obligatorio en la pestaña procurador y debe '.
                        'ser uno de los siguientes: POR EXCESIVAS, '.
                        'POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['procurador_fecha_impugnacion']
                ) ?
                    $formData['procurador_fecha_impugnacion'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es obligatoria '.
                        'y debe ser válida en la pestaña procurador.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha impugnación o traslado es inválida '.
                            'en la pestaña procurador.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                '
                            La fecha de impugnación debe ser posterior '.
                                'a la fecha de traslado en la pestaña procurador.'
                            );
                        } else {
                            $formattedDateImpugnacion = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }


            if ($formData['imp_contrario'] == 1) {
                if (empty($formData['tipo_impugnacion_contrario'])) {
                    $errors[] = __(
                        'El Tipo Impugnación contrario es obligatorio '.
                        'en la pestaña Arriaga y debe ser uno de los '.
                        'siguientes: POR EXCESIVAS, POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion_contrario']
                ) ?
                    $formData['fecha_impugnacion_contrario'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        '
                    La fecha de impugnación es obligatoria '.
                        'y debe ser válida en la pestaña Arriaga.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            '
                        Fecha de impugnación o traslado '.
                            'es incorrecta en la pestaña Arriaga.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación debe '.
                                'ser posterior a la fecha de '.
                                'traslado en la pestaña Arriaga.'
                            );
                        } else {
                            $formattedDateImpugnacionContrario = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }





            if ($formData['procurador_imp_contrario'] == 1) {
                if (empty($formData['procurador_tipo_impugnacion_contrario'])) {
                    $errors[] = __(
                        'El Tipo Impugnación contrario '.
                        'es obligatorio en la pestaña '.
                        'procurador y debe ser uno de '.
                        'los siguientes: POR EXCESIVAS, '.
                        'POR INDEBIDAS o POR AMBAS.'
                    );
                }


                $challengeDate = isset(
                    $formData['fecha_impugnacion_contrario']
                ) ?
                    $formData['fecha_impugnacion_contrario'] : null;
                $fechaTraslado = isset(
                    $formData['fecha_traslado']
                ) ?
                    $formData['fecha_traslado'] : null;


                if (empty($challengeDate)) {
                    $errors[] = __(
                        'La fecha de impugnación es '.
                        'obligatoria y debe ser válida '.
                        'en la pestaña procurador.'
                    );
                } else {
                    $fechaTrasladoObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $fechaTraslado
                    );
                    $challengeDateObj = \DateTime::createFromFormat(
                        'd/m/Y',
                        $challengeDate
                    );


                    if (!$fechaTrasladoObj || !$challengeDateObj) {
                        $errors[] = __(
                            'Fecha de impugnación o '.
                            'traslado es incorrecta '.
                            'en la pestaña procurador.'
                        );
                    } else {
                        if ($challengeDateObj < $fechaTrasladoObj) {
                            $errors[] = __(
                                'La fecha de impugnación '.
                                    'debe ser posterior a la '.
                                    'fecha de traslado en la pestaña procurador.'
                            );
                        } else {
                            $formattedDateImpugnacionContrario = $challengeDateObj
                            ->format('Y-m-d');
                        }
                    }
                }
            }
            if ($formData['tipo'] == 'a favor') {
                if (empty($formData['fecha_traslado'])) {
                    $errors[] = __('La fecha de traslado es obligatoria.');
                } else {
                    if (!$this->validateDateNotGreater(
                        $formData,
                        'fecha_traslado',
                        'fecha_presentacion',
                        $errors
                    )
                    ) {
                    }
                }
            }

            if ($formData['tipo'] == 'en contra') {
            }
        }
        if ($formData['fase'] == 'decreto' || $formData['fase'] == 'resolucion') {
            if (empty($formData['fecha_decreto'])) {
                $errors[] = __('La fecha de decreto es obligatoria.');
            } else {
                if (!$this->validateDateNotGreater(
                    $formData,
                    'fecha_decreto',
                    'fecha_traslado',
                    $errors
                )
                ) {
                }
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_decreto',
                    $errors
                )
                ) {
                    $errors[] = __('La fecha de decreto no puede ser mayor a hoy.');
                }
            }



            if (!empty($formData['importe_decreto'])
                || !empty($formData['procurador_importe_decreto'])
            ) {
                $optionalFields = [
                    'importe_decreto' =>
                        __(
                            'El Importe Decreto en la '.
                            'pestaña Arriaga debe ser '.
                            'un importe positivo en '.
                            'caso de haberse informado.'
                        ),
                    'procurador_importe_decreto' =>
                        __(
                            'El Importe Decreto en procurador '.
                            'debe ser un importe positivo '.
                            'en caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }

            $this->_validateDateResource(
                $formData,
                'recurso',
                'fecha_recurso',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_recurso',
                'procurador_fecha_recurso',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'recurso_contrario',
                'fecha_recurso_contrario_decreto',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_recurso_contrario',
                'procurador_fecha_recurso_contrario_decreto',
                'fecha_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'impugnacion_contrario_decreto',
                'fecha_impugnacion_contrario_decreto',
                'fecha_recurso',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_impugnacion_contrario_decreto',
                'procurador_fecha_impugnacion_contrario_decreto',
                'procurador_fecha_recurso',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña procurador.')
            );
            $this->_validateDateResource(
                $formData,
                'impugnacion_rc',
                'fecha_impugnacion_rc',
                'fecha_impugnacion_contrario_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña Arriaga.')
            );
            $this->_validateDateResource(
                $formData,
                'procurador_impugnacion_rc',
                'procurador_fecha_impugnacion_rc',
                'procurador_fecha_impugnacion_contrario_decreto',
                $errors,
                __('La Fecha Recurso  es obligatoria en la pestaña procurador.')
            );
            if ($formData['tipo'] == 'a favor') {
            }
            if ($formData['tipo'] == 'en contra') {
            }
        }

        if ($formData['fase'] == 'resolucion') {
            if (empty($formData['fecha_resolucion'])) {
                $errors[] = __('La fecha de resolución es obligatoria.');
            } else {
                if (!$this->validateDateNotGreater(
                    $formData,
                    'fecha_resolucion',
                    'fecha_decreto',
                    $errors
                )
                ) {
                }
                if (!$this->_validateNotFutureDate(
                    $formData,
                    'fecha_resolucion',
                    $errors
                )
                ) {
                    $errors[] = __(
                        'La fecha de resolución '.
                        'no puede ser mayor a hoy.'
                    );
                }
            }

            if (!empty($formData['importe_resolucion'])
                || !empty($formData['procurador_importe_resolucion'])
            ) {
                $optionalFields = [
                    'importe_resolucion' =>
                        __(
                            'El Importe Resolucion en la '.
                            'pestaña Arriaga debe ser un importe '.
                            'positivo en caso de haberse informado.'
                        ),
                    'procurador_importe_resolucion' =>
                        __(
                            'El Importe Resolucion en procurador '.
                            'debe ser un importe positivo en '.
                            'caso de haberse informado.'
                        ),
                ];
                $this->_validateOptionalFields($formData, $optionalFields, $errors);
            }



            if ($formData['tipo'] == 'a favor') {
            }
            if ($formData['tipo'] == 'en contra') {
            }
        }

        $fechaPrimeraInstancia = isset(
            $record
                ->pro_fecha_sentencia_primera_instancia
        )
            ? $record->pro_fecha_sentencia_primera_instancia->format('d/m/Y')
            : null;

        $fechaPresentacion = isset($formData['fecha_presentacion'])
            ? $formData['fecha_presentacion']
            : null;

        if (isset($formData['tipo_procedimiento'])
            && $formData['tipo_procedimiento'] === 'primera instancia'
            && $formData['tipo'] !== 'en contra'
            && $fechaPrimeraInstancia != null
        ) {
            if ($fechaPrimeraInstancia && $fechaPresentacion) {
                $datePrimeraInstancia = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPrimeraInstancia
                );
                $datePresentacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPresentacion
                );


                if ($datePrimeraInstancia && $datePresentacion) {
                    if ($datePresentacion < $datePrimeraInstancia) {
                        $errors[] = __(
                            'La fecha de presentación no puede '.
                            'ser menor que la fecha de '.
                            'sentencia de primera instancia.'
                        );
                    }
                } else {
                    $errors[] = __('Una o ambas fechas son inválidas.');
                }
            } else {
                if (empty($fechaPrimeraInstancia)) {
                    $errors[] = __(
                        'La fecha de sentencia de ' .
                        'primera instancia no está definida.'
                    );
                }
                if (empty($fechaPresentacion)) {
                    $errors[] = __('La fecha de presentación no está definida.');
                }
            }
        }

        $fechaApelacion = isset($record->ape_apelacion)
            ? $record->ape_apelacion->format('d/m/Y')
            : null;

        $fechaPresentacion = isset($formData['fecha_presentacion'])
            ? $formData['fecha_presentacion']
            : null;

        if (isset($formData['tipo_procedimiento'])
            && $formData['tipo_procedimiento'] === 'apelacion'
            && $formData['tipo'] !== 'en contra'
            && $fechaApelacion != null
        ) {
            if ($fechaApelacion && $fechaPresentacion) {
                $dateApelacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaApelacion
                );
                $datePresentacion = \DateTime::createFromFormat(
                    'd/m/Y',
                    $fechaPresentacion
                );


                if ($dateApelacion && $datePresentacion) {
                    if ($datePresentacion < $dateApelacion) {
                        $errors[] = __(
                            'La fecha de presentación no puede' .
                            ' ser menor que la fecha de apelación.'
                        );
                    }
                } else {
                    $errors[] = __('Una o ambas fechas son inválidas.');
                }
            } else {
                if (empty($fechaApelacion)) {
                    $errors[] = __('La fecha de apelación no está definida.');
                }
                if (empty($fechaPresentacion)) {
                    $errors[] = __('La fecha de presentación no está definida.');
                }
            }
        }




        if (!empty($errors)) {
            foreach ($errors as $error) {
                $this->Flash->error($error);
            }
            $destination = 'procedure-man';
            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        } else {
            $connCourtCosts = $this->CourtCosts->connection();
            $connCourtCosts->begin();


            try {
                $courtCost = $this->CourtCosts->patchEntity(
                    $courtCost,
                    [
                        'record_id' => $recordId,
                        'stage' =>
                            isset($formData["tipo"]) ?
                            $formData["tipo"] : null,
                        'costs_type' =>
                            isset($formData["tipo_procedimiento"]) ?
                            $formData["tipo_procedimiento"] : null,
                        'procedure_type' =>
                            isset($formData["fase"]) ?
                            $formData["fase"] : null,
                        'presentation_date' =>
                            !empty($formData['fecha_presentacion']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_presentacion'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'transfer_date' =>
                            !empty($formData['fecha_traslado']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_traslado'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_date' =>
                            !empty($formData['fecha_decreto']) ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'ruling_date' =>
                            !empty($formData['fecha_resolucion']) ?
                             date(
                                 'Y-m-d',
                                 strtotime(
                                     \DateTime::createFromFormat(
                                         'd/m/Y',
                                         $formData[
                                            'fecha_resolucion'
                                         ]
                                     )
                                     ->format('Y-m-d')
                                 )
                             ) : null,

                    ]
                );
                if (!$this->CourtCosts->save($courtCost)) {
                    throw new \Exception('Error al guardar CourtCosts');
                }

                $lcc = $this->LawyerCourtCosts->patchEntity(
                    $lawsuitCost,
                    [

                        //presentation data
                        'fee_amount' =>
                            !empty($formData["imp_costas"]) ?
                            $formData["imp_costas"] : 0,
                        'proficient_amount' =>
                            !empty($formData["imp_perito"]) ?
                            $formData["imp_perito"] : 0,
                        'witness_amount' =>
                            !empty($formData["imp_testigo"]) ?
                            $formData["imp_testigo"] : 0,
                        'calculation_date' =>
                            !empty($formData['fecha_calculo'])
                                ? \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData[
                                        'fecha_calculo'
                                    ]
                                )
                                : null,

                        //transfer data
                        'taxation_result' =>
                            isset(
                                $formData["resultado_tasacion"]
                            ) ?
                                $formData["resultado_tasacion"] : null,
                        'laj_amount' =>
                            !empty($formData["importe_laj"]) ?
                            $formData["importe_laj"] : 0,
                        'laj_requested_difference_amount' =>
                            !empty($formData["laj_requested_difference_amount"]) ?
                            $formData["laj_requested_difference_amount"] : 0,
                            'ica_report' => $formData['informe_ica'] == 1 ? 1 : 0,
                        'ica_report_date' =>
                            $formData['informe_ica'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_ica'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'own_challenge' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            1 : 0,
                        'own_challenge_type' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            $formData["tipo_impugnacion"] : null,
                        'own_challenge_date' =>
                            $formData['impugnacion_traslado'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_impugnacion'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'opponent_challenge' =>
                            $formData['imp_contrario'] == 1 ?
                            1 : 0,
                        'opponent_challenge_type' =>
                            $formData['imp_contrario'] == 1 ?
                            $formData["tipo_impugnacion_contrario"] : null,
                        'opponent_challenge_date' =>
                            $formData['imp_contrario'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_impugnacion_contrario'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'opponent_challenge_acceptance' =>
                            $formData['conformidad_imp_contrario'] == 'yes' ?
                            1 : 0,


                        //decree data
                        'decree_result' =>
                            isset(
                                $formData["resultado_decreto"]
                            ) ?
                                $formData["resultado_decreto"] : null,
                        'decree_amount' =>
                            !empty($formData["importe_decreto"]) ?
                            $formData['importe_decreto'] : 0,
                        'decree_requested_difference_amount' =>
                            !empty($formData["decree_dif"]) ?
                            $formData['decree_dif'] : 0,
                        'decree_own_appeal' =>
                            $formData['recurso'] == 1 ? 1 : 0,
                        'decree_own_appeal_date' =>
                            $formData['recurso'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_recurso'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_opponent_challenge_to_own_appeal' =>
                            $formData['impugnacion_contrario_decreto'] == 1 ?
                            1 : 0,
                        'decree_opponent_challenge_to_own_appeal_date' =>
                            $formData['impugnacion_contrario_decreto'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                        'fecha_impugnacion_contrario_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_opponent_appeal' =>
                            $formData['recurso_contrario'] == 1 ? 1 : 0,
                        'decree_opponent_appeal_date' =>
                            $formData['recurso_contrario'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_recurso_contrario_decreto'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_own_challenge_to_opponent_appeal' =>
                            $formData['impugnacion_rc'] == 1 ?
                            1 : 0,
                        'decree_own_challenge_to_opponent_appeal_date' =>
                            $formData['impugnacion_rc'] == 1 ?
                            date(
                                'Y-m-d',
                                strtotime(
                                    \DateTime::createFromFormat(
                                        'd/m/Y',
                                        $formData[
                                            'fecha_impugnacion_rc'
                                        ]
                                    )
                                    ->format('Y-m-d')
                                )
                            ) : null,
                        'decree_favorable_costs' =>
                            !empty($formData['costas_incidente_decreto']) ?
                            1 : 0,
                        'decree_against_costs' =>
                            !empty($formData['costas_ec_incidente_decreto']) ?
                            1 : 0,
                        'own_allegations_to_opponent_challenge' =>
                            isset(
                                $formData["alegaciones_imp_contrario"]
                            ) ?
                                $formData["alegaciones_imp_contrario"] : false,

                        //ruling data
                        'ruling_result' =>
                            isset(
                                $formData["resultado_resolucion"]
                            ) ?
                                $formData["resultado_resolucion"] : null,
                        'ruling_amount' =>
                            !empty($formData["importe_resolucion"]) ?
                            $formData["importe_resolucion"] : 0,
                        'ruling_requested_difference_amount' =>
                            !empty($formData["ruling_dif"]) ?
                            $formData["ruling_dif"] : 0,
                        'ruling_favorable_costs' =>
                            !empty($formData['costas_incidente']) ?
                            1 : 0,
                        'ruling_against_costs' =>
                            !empty($formData['costas_ec_incidente']) ?
                            1 : 0,

                    ]
                );

                if (!$this->LawyerCourtCosts->save($lcc)) {
                    throw new \Exception('Error al guardar LawyerCourtCosts');
                }
                $pro = $this->AttorneyCourtCosts->patchEntity(
                    $attorneyCost,
                    [


                        //presentation data
                        'fee_amount' => !empty(
                            $formData["procurador_imp_costas"]
                        ) ?
                            $formData["procurador_imp_costas"] : 0,
                        'proficient_amount' => !empty(
                            $formData["procurador_imp_perito"]
                        ) ?
                            $formData["procurador_imp_perito"] : 0,
                        'witness_amount' =>
                        !empty($formData["procurador_imp_testigo"]) ?
                        $formData["procurador_imp_testigo"] : 0,
                        'tax_amount' => !empty(
                            $formData["procurador_imp_tasa"]
                        ) ?
                            $formData["procurador_imp_tasa"] : 0,
                        'calculation_date' => !empty(
                            $formData['procurador_fecha_calculo']
                        ) ?
                             \DateTime::createFromFormat(
                                 'd/m/Y',
                                 $formData[
                                    'procurador_fecha_calculo'
                                 ]
                             ) : null,

                        //transfer data
                        'taxation_result' => isset(
                            $formData["procurador_resultado_tasacion"]
                        ) ?
                            $formData["procurador_resultado_tasacion"] : null,
                        'laj_amount' =>
                        !empty(
                            $formData[
                                "procurador_importe_laj"
                            ]
                        ) ?
                                 $formData[
                                    "procurador_importe_laj"
                                    ] : 0,
                        'laj_requested_difference_amount' =>
                        !empty(
                            $formData[
                                "procurador_laj_requested_difference_amount"
                            ]
                        ) ?
                                $formData[
                                    "procurador_laj_requested_difference_amount"
                                    ] : 0,
                        'own_challenge' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         1 : 0,
                        'own_challenge_type' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         $formData["procurador_tipo_impugnacion"] : null,
                        'own_challenge_date' =>
                        $formData['procurador_impugnacion_traslado'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion'
                                     ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'opponent_challenge' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         1 : 0,
                        'opponent_challenge_type' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         $formData[
                            "procurador_tipo_impugnacion_contrario"
                            ] : null,
                        'opponent_challenge_date' =>
                        $formData['procurador_imp_contrario'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion_contrario'
                                        ]
                                 )->format('Y-m-d')
                             )
                         ) : null,
                        'opponent_challenge_acceptance' =>
                        $formData[
                            'procurador_conformidad_imp_contrario'
                            ] == 'yes' ?
                         1 : 0,


                        //decree data
                        'decree_result' => isset(
                            $formData["procurador_resultado_decreto"]
                        ) ?
                             $formData["procurador_resultado_decreto"] : null,
                        'decree_amount' =>
                        !empty($formData["procurador_importe_decreto"]) ?
                         $formData['procurador_importe_decreto'] : 0,
                        'decree_requested_difference_amount' =>
                        !empty($formData["procurador_decree_dif"]) ?
                         $formData['procurador_decree_dif'] : 0,
                        'decree_own_appeal' =>
                        $formData['procurador_recurso'] == 1 ?
                         1 : 0,
                        'decree_own_appeal_date' =>
                        $formData['procurador_recurso'] == 1 ?
                        date(
                            'Y-m-d',
                            strtotime(
                                \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData[
                                        'procurador_fecha_recurso'
                                    ]
                                )
                                ->format('Y-m-d')
                            )
                        ) : null,
                        'decree_opponent_challenge_to_own_appeal' =>
                        $formData['procurador_impugnacion_contrario_decreto'] == 1 ?
                         1 : 0,
                        'decree_opponent_challenge_to_own_appeal_date' =>
                        $formData['procurador_impugnacion_contrario_decreto'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_impugnacion' .
                                        '_contrario_decreto'
                                     ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'decree_opponent_appeal' =>
                        $formData['procurador_recurso_contrario'] == 1 ?
                         1 : 0,
                        'decree_opponent_appeal_date' =>
                        $formData['procurador_recurso_contrario'] == 1 ?
                         date(
                             'Y-m-d',
                             strtotime(
                                 \DateTime::createFromFormat(
                                     'd/m/Y',
                                     $formData[
                                        'procurador_fecha_recurso_contrario_decreto'
                                        ]
                                 )
                                 ->format('Y-m-d')
                             )
                         ) : null,
                        'decree_own_challenge_to_opponent_appeal' =>
                        $formData['procurador_impugnacion_rc'] == 1 ?
                         1 : 0,
                        'decree_own_challenge_to_opponent_appeal_date' =>
                        $formData['procurador_impugnacion_rc'] == 1 ?
                        date(
                            'Y-m-d',
                            strtotime(
                                \DateTime::createFromFormat(
                                    'd/m/Y',
                                    $formData['procurador_fecha_impugnacion_rc']
                                )
                                ->format('Y-m-d')
                            )
                        ) : null,
                        'decree_favorable_costs' =>
                        !empty($formData['procurador_costas_incidente_decreto']) ?
                            1 : 0,
                        'decree_against_costs' =>
                        !empty($formData['procurador_costas_ec_incidente_decreto']) ?
                            1 : 0,
                        'own_allegations_to_opponent_challenge' => isset(
                            $formData["procurador_alegaciones_imp_contrario"]
                        ) ?
                            $formData["procurador_alegaciones_imp_contrario"]
                            : false,

                        //ruling data
                        'ruling_result' =>
                        isset(
                            $formData["procurador_procurador_resultado_resolucion"]
                        ) ?
                            $formData["procurador_resultado_resolucion"] : null,
                        'ruling_amount' =>
                        !empty($formData["procurador_importe_resolucion"]) ?
                            $formData["procurador_importe_resolucion"] : 0,
                        'ruling_requested_difference_amount' =>
                        !empty($formData["procurador_ruling_dif"]) ?
                            $formData["procurador_ruling_dif"] : 0,
                        'ruling_favorable_costs' =>
                        !empty($formData['procurador_costas_incidente']) ? 1 : 0,
                        'ruling_against_costs' =>
                        !empty($formData['procurador_costas_ec_incidente']) ? 1 : 0,

                    ]
                );
                if (!$this->AttorneyCourtCosts->save($pro)) {
                    throw new \Exception('Error al guardar AttorneyCourtCosts');
                }
                $connCourtCosts->commit();
                $this->Flash->success(
                    __('La tasación de costas se ha actualizado correctamente.')
                );
            } catch (\Exception $e) {
                $connCourtCosts->rollback();
                $this->Flash->error(
                    __(
                        'No se ha podido actualizar la tasación de costas. Error: '
                            .
                            $e->getMessage()
                    )
                );
                return $this->redirect(
                    [
                        'plugin'     => 'Records',
                        'controller' => 'Manage',
                        'action'     => 'edit',
                        $recordId,
                        '#'          => $destination
                    ]
                );
            }

            return $this->redirect(
                [
                    'plugin'     => 'Records',
                    'controller' => 'Manage',
                    'action'     => 'edit',
                    $recordId,
                    '#'          => $destination
                ]
            );
        }
        // return $this->redirect($this->request->referer());
        $destination = 'procedure-man';
        return $this->redirect(
            [
                'plugin'     => 'Records',
                'controller' => 'Manage',
                'action'     => 'edit',
                $recordId,
                '#'          => $destination
            ]
        );
    }

    /**
     * Añade, si son necesarias, las etiquetas del proceso
     * de tasación de costas para ejecuciones.
     *
     * @param object $courtCost El objeto que representa el costo del tribunal.
     *
     * @return void|\Cake\Network\Response
     */
    private function _tagsCourtCosts($courtCost)
    {
        $this->loadModel('Records.Records');
        $record = $this->Records->get(
            $courtCost->get('record_id'),
            [
                'eav' => false
            ]
        );
        if ($courtCost->get('costs_type') == 'primera instancia'
            && $courtCost->get('stage') == 'a favor'
            && in_array(
                $courtCost->get('procedure_type'),
                ['traslado', 'decreto', 'resolucion']
            )
        ) {
            if (!$record->hasTag('COSTAS ORDINARIO-->NO PROCEDE')) {
                $record->addTag('COSTAS ORDINARIO TASADAS');
                $record->addTag('COSTAS TASADAS PRESENTADAS');
                if ($courtCost->get('proficient_amount') > 0) {
                    $record->addTag('PERITAJE');
                } else {
                    if ($record->hasTag('PERITAJE')) {
                        $record->removeTag('PERITAJE');
                    }
                }
                if ($courtCost->get('witness_amount') > 0) {
                    $record->addTag('TESTIFICAL');
                } else {
                    if ($record->hasTag('TESTIFICAL')) {
                        $record->removeTag('TESTIFICAL');
                    }
                }
            }
        }
        if ($courtCost->get('costs_type') == 'apelacion'
            && $courtCost->get('stage') == 'a favor'
            && in_array(
                $courtCost->get('procedure_type'),
                ['traslado', 'decreto', 'resolucion']
            )
        ) {
            if (!$record->hasTag('COSTAS APELACIÓN-->NO PROCEDE')) {
                $record->addTag('COSTAS APELACIÓN TASADAS');
            }
        }

        return;
    }
}




```

[[Programación]]