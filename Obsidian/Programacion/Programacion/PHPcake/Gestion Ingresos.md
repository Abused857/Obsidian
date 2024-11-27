
Gestion ingresos ctp
```
<?php

/**
 * PHP version 7
 *
 * @category Template_Class
 * @package  Template_Class
 * @author   Author <author@domain.com>
 * @license  https://opensource.org/licenses/MIT MIT License
 * @link     http://localhost/
 */
?>
<style>
    .readonly-select {
        pointer-events: none;

        background-color: #d3d3d3;

        color: #a9a9a9;

        border-color: #a9a9a9;
    }

    .disabled-checkbox {
        pointer-events: none;
        opacity: 0.5;
    }

    .custom-hr {
        border-bottom: 1px solid grey;
        background-color: #d9dbdc;
        height: 4px;
    }

    fieldset {
        border: 1px solid #ddd;
        border-radius: 4px;
    }

    legend {
        border: none;
        padding: 1rem;
        width: auto;
        font-size: 16px;
    }

    .round-button {
        border-radius: 20px;
    }
</style>
<br>
<button type="button" class="btn btn-secondary toggle-income-block">&#9650;</button>
<div id='' class='ingress-content'>

    <?php
    $allowedSituations = [
        'REDACCIÓN',
        'REDACCIÓN PARALIZADA',
        'REDACCIÓN TERMINADA',
        'COMPLETÁNDOSE DOCUMENTACIÓN',
        'COMPLETAR POR JURÍDICO',
        'REVISIÓN DEL COMPLETADO',
        'DOCUMENTACIÓN COMPLETADA',
        'DUPLICADO',
        'CITA',
        'ENTREVISTA',
        'EXPEDIENTE INICIADO',
        'NO CLIENTE',
        'NO VIABLE',
        'JUDICIAL',
        'PARALIZADO',
        'PARALIZADO ESPERA VIABILIDAD',
        'PRESENTADO A PROCURADOR',
        'PRESENTADO EN JUZGADO',
        'PROCURADOR CALIDAD PARADO',
        'SELLADOS EN JUZGADO',
        'REVOCADO',
        'VIABLE'

    ];


    $canAddIncome = !in_array($record->status, $allowedSituations);


    if ($canAddIncome) :

        echo $this->Form->create($record, ['url' => ['controller' => 'Manage', 'action' => 'ingressIncome']]);

        echo $this->Form->hidden('record_id', ['value' => $record->id]);


    ?>
        <div class="row">
            <div class="col-md-10">
                <h4>Fase</h4>
            </div>
            <div class="col-md-2 text-right">
                <button
                    type="submit"
                    data-loading-text="Guardando..."
                    class="btn btn-primary register-record">
                    <i class="fa fa-floppy-o"></i>
                </button>
                <?php
                /*echo $this->Form->button(__('Guardar'), [
                    'class' => 'btn btn-primary register-record',
                    'type' => 'submit',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'ingressIncome']),
                ]);
                
                echo $this->Form->button(__('Borrar'), [
                    'class' => 'btn btn-danger',
                    'type' => 'submit',
                    'name' => 'action',
                    'value' => 'delete',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'deleteIncome', $income->id]),
                    'onclick' => "return confirm('" . ($incomeExists ? "Se eliminarán todos los datos de ingresos, pero los documentos relacionados permanecerán en el expediente. ¿Desea continuar?" : "¿Está seguro de que desea eliminar este ingreso?") . "');"
                ]); */
                ?>
            </div>
        </div>

        <div class="row">
            <div class="col-md-4">
                <?php echo
                //$defaultStage = $isFirst ? 'grabacion' : ($income['stage'] ?? null);
                //$nonJudicialIncome = (isset($income['non_judicial_income']) && $income['non_judicial_income'] == 1) ? 1 : 0;
                $this->Form->input(
                    'fase',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_phase'),
                        'label' => false,
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'fase_',
                        'value' => '',
                        'required' => true,
                    ]
                );
                ?>
            </div>
            <div class="col-md-4">
                <?php echo
                $this->Form->input(
                    'ingreso_no_judicial',
                    [
                        'type' => 'checkbox',
                        'label' => 'Ingreso no judicial',
                        'id' => 'ingreso_no_judicial_',
                        //'checked' => $nonJudicialIncome
                    ]
                );
                ?>
            </div>
        </div>

        <hr class="custom-hr" />

        <div class="row">
            <div class="col-md-2">
                <h4>Grabación</h4>
            </div>

            <div class="col-md-3">
                <?php echo
                /*
                $incomeSource = isset($income['income_source']) ? $income['income_source'] : null;
                $incomeSourceCourtBank = isset($income['income_source_court_bank']) ? $income['income_source_court_bank'] : null; #TODO: de donde viene el numero de cuenta del juzgado en el select
                $incomeAmount  = isset($income['income_amount']) ? $income['income_amount'] : '';
                $incomeEvidenceDate = isset($income['income_evidence_date']) ? $income['income_evidence_date']->format('d/m/Y') : '';
                */
                $this->Form->input(
                    'origen',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_source'),
                        'label' => 'Origen',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'origen_',
                        //'value' => $incomeSource,
                    ]
                ); ?>
            </div>
            <div class="col-md-3">
                <?php echo
                $this->Form->input(
                    'n_cuenta_juzgado',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_source'), // TODO: Definir de donde viene la cuenta si hay varias o que
                        'label' => 'Nº Cuenta Juzgado',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'n_cuenta_juzgado_',
                        //'value' => $incomeSourceCourtBank,
                    ]
                );
                ?>
            </div>
            <div class="col-md-2">
                <?php echo $this->Form->input(
                    'imp_ingreso',
                    [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de ingreso',
                        'label' => 'Imp. Ingreso',
                        'id' => 'imp_ingreso_',
                        //'value' => $incomeAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).'
                    ]
                ); ?>
            </div>
            <div class="col-md-2">
                <?php echo $this->Form->input(
                    'fecha_constancia',
                    [
                        'type' => 'text',
                        'class' => 'form-control datepicker',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'Fecha Constancia',
                        'id' => 'fecha_constancia_',
                        //'value' => $incomeEvidenceDate,
                        'autocomplete' => 'off',
                    ]
                ); ?>
            </div>
        </div>

        <hr class="custom-hr" />
        <br>
        <div class="row">
            <div class="col-md-2">
                <h4>Consolidación</h4>
            </div>

            <div class="col-md-2">
                <?php echo
                /*
                $commitmentInAttorneyBankAccount  = isset($cost['commitment_in_attorney_bank_account']) ? $cost['commitment_in_attorney_bank_account'] : '';
                $commitmentRequestType  = isset($income['commitment_request_type']) ? $income['commitment_request_type'] : '';
                $commitmentRequestDate = isset($income['commitment_request_date']) ? $income['commitment_request_date']->format('d/m/Y') : '';
                $paymentOrderLocation  = isset($income['payment_order_location']) ? $income['payment_order_location'] : '';
                $paymentOrderIssueDate = isset($income['payment_order_issue_date']) ? $income['payment_order_issue_date']->format('d/m/Y') : '';
                $paymentOrderExpiryDate = isset($income['payment_order_expiry_date']) ? $income['payment_order_expiry_date']->format('d/m/Y') : '';
                $paymentOrderTaxPaid  = isset($income['payment_order_tax_paid']) ? $income['payment_order_tax_paid'] : '';
                $commitmentDate = isset($income['commitment_date']) ? $income['commitment_date']->format('d/m/Y') : '';
                $commitmentDestination  = isset($income['commitment_destination']) ? $income['commitment_destination'] : '';
                */
                $this->Form->input(
                    'en_cuenta_procurador',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_yes_or_no'),
                        'label' => 'En Cuenta Procurador',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'en_cuenta_procurador_',
                        //'value' => $commitmentInAttorneyBankAccount,
                    ]
                ); ?>
            </div>
            <div class="col-md-4">
                <?php echo
                $this->Form->input(
                    'tipo_solicitud',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_commitment_type'), // TODO: que valores tiene este campo
                        'label' => 'Tipo Solicitud',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tipo_solicitud_',
                        //'value' => $commitmentRequestType,
                    ]
                );
                ?>
            </div>
            <div class="col-md-2">
                <?php echo $this->Form->input(
                    'fecha_solicitud',
                    [
                        'type' => 'text',
                        'class' => 'form-control datepicker',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Solicitud',
                        'id' => 'fecha_solicitud_',
                        // 'value' => $commitmentRequestDate,
                        'autocomplete' => 'off',
                    ]
                ); ?>
            </div>

        </div>

        <!-- Grabación MP ROW-->
        <br>
        <div class="row">
            <div class="col-md-2">
            </div>

            <div class="col-md-2">
                <?php echo
                $this->Form->input(
                    'ubicacion_mp',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_commitment_order_location'), // TODO: que valores tiene este campo
                        'label' => 'Ubicación MP',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'ubicacion_mp_',
                        //'value' => $paymentOrderLocation,
                    ]
                ); ?>
            </div>
            <div class="col-md-2">
                <?php echo
                $this->Form->input(
                    'fecha_expedicion_mp',
                    [
                        'type' => 'text',
                        'class' => 'form-control datepicker',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Expedición MP',
                        'id' => 'fecha_expedicion_mp_',
                        //'value' => $paymentOrderIssueDate,
                        'autocomplete' => 'off',
                    ]
                );
                ?>
            </div>
            <div class="col-md-2">
                <?php echo $this->Form->input(
                    'fecha_caducidad_mp',
                    [
                        'type' => 'text',
                        'class' => 'form-control datepicker',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Caducidad MP',
                        'id' => 'fecha_caducidad_mp_',
                        // 'value' => $paymentOrderExpiryDate,
                        'autocomplete' => 'off',
                    ]
                ); ?>
            </div>

            <div class="col-md-2">
                <?php echo $this->Form->input(
                    'tasa_pagada_mp',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_yes_or_no'),
                        'label' => 'Tasa pagada MP',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tasa_pagada_mp_',
                        //'value' => $paymentOrderTaxPaid,
                    ]
                ); ?>
            </div>

        </div>

        <!-- Grabación Consolidación ROW-->
        <br>
        <div class="row">
            <div class="col-md-2">
            </div>

            <div class="col-md-2">
                <?php echo
                $this->Form->input(
                    'fecha_consolidacion',
                    [
                        'type' => 'text',
                        'class' => 'form-control datepicker',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Consolidación',
                        'id' => 'fecha_consolidacion_',
                        // 'value' => $commitmentDate,
                        'autocomplete' => 'off',
                    ]
                );
                ?>
            </div>

            <div class="col-md-4">
                <?php echo
                $this->Form->input(
                    'destino_consolidacion',
                    [
                        'type' => 'select',
                        'options' => jsonc('Records.income_commitment_destination'),
                        'label' => 'Destino Consolidación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'destino_consolidacion_',
                        // 'value' => $commitmentDestination,
                        'required' => true,
                    ]
                ); ?>
            </div>
        </div>
        <hr class="custom-hr" />
        <br>
        <div class="row">
            <div class="col-md-2">
                <h4>Asignación</h4>
            </div>

            <div id="appendClient">
            </div>

            <div class="col-md-2 text-right">
                <button type="button" class="btn btn-primary" id="firstClient" style="display: none;">Agregar Cliente</button>
            </div>

            <div class="row" style='display:none;' id="clientBlock">
                <div class="col-md-10">
                    <fieldset class="col-sm-11">
                        <legend>Importe Clientes (Principal)</legend>
                        <div id="clientRows">
                        </div>
                        <br>

                        <br>
                        <div class="row">
                            <div class="col-md-10"></div>
                            <div class="col-md-2 text-right">
                                <button type="button" class="btn btn-primary" onclick="duplicateClients()">+</button>
                            </div>
                        </div>
                        <div id="ccRow_clone" style="display:none;">

                            <!-- Asiganción ROW-->

                            <div class="row" id="clientTitular_0">
                                <div class="col-md-6">
                                    <?php echo $this->Form->input(
                                        'titular_cliente[clone]',
                                        [
                                            'type' => 'select',
                                            'options' => jsonc('Records.income_yes_or_no'),
                                            'label' => 'Titular',
                                            'empty' => '-- Seleccione una opción --',
                                            'id' => 'titular_cliente_clone',
                                        ]
                                    ); ?>
                                </div>
                                <div class="col-md-6">
                                    <?php echo $this->Form->input(
                                        'modo_de_entrega[clone]',
                                        [
                                            'type' => 'select',
                                            'options' => jsonc('Records.income_request_type'),
                                            'label' => 'Modo de Entrega',
                                            'empty' => '-- Seleccione una opción --',
                                            'id' => 'modo_de_entrega_clone',
                                        ]
                                    ); ?>
                                </div>
                            </div>
                            <!-- Asiganción IBAN ROW-->
                            <br>
                            <div class="row" id="clientContainer_clone">
                                <div class="col-md-3">
                                    <?php echo $this->Form->input(
                                        'importe_cliente[clone]',
                                        [
                                            'type' => 'number',
                                            'step' => '0.01',
                                            'min' => 0,
                                            'placeholder' => 'Ingrese el importe ',
                                            'label' => 'Importe',
                                            'id' => 'importe_cliente_clone',
                                            'title' => 'Ingrese un número decimal válido (hasta 2 decimales).'
                                        ]
                                    ); ?>
                                </div>
                                <div class="col-md-1"></div>
                                <div class="col-md-2">
                                    <?php echo $this->Form->input(
                                        'pais[0]',
                                        [
                                            'type' => 'select',
                                            'options' => jsonc('Records.income_country'),
                                            'label' => 'País',
                                            'id' => 'pais_clone',
                                            'required' => true,
                                            'empty' => '-- Seleccione una opción --',

                                        ]
                                    ); ?>
                                </div>
                                <div class="col-md-2">
                                    <?php echo $this->Form->input(
                                        'codigo_iban[clone]',
                                        [
                                            'type' => 'number',
                                            'label' => 'Cod. IBAN',
                                            'id' => 'codigo_iban_clone',
                                            'min' => '0',
                                            'step' => '1',
                                            'maxlength' => 34,
                                            'title' => 'Ingrese un número de Iban válido (sin decimales ni negativos)',
                                            'required' => true,
                                        ]
                                    ); ?>
                                </div>
                                <div class="col-md-4">
                                    <?php echo $this->Form->input(
                                        'numero_cuenta[clone]',
                                        [
                                            'type' => 'text',
                                            'label' => 'Número de Cuenta',
                                            'id' => 'numero_cuenta_clone',
                                            'title' => 'Ingrese un número de cuenta válido (puede incluir letras y números)',
                                            'required' => true,
                                        ]
                                    ); ?>
                                </div>
                            </div>

                        </div>

                        <br>
                        <!-- Row para el total del importe -->
                        <div class="row">
                            <div class="col-md-9"></div>
                            <div class="col-md-3">
                                <?php echo $this->Form->input(
                                    'total_importe_cliente',
                                    [
                                        'type' => 'text',
                                        'label' => 'Total Importe',
                                        'id' => 'total_importe_cliente_',
                                        'readonly' => true,
                                        'value' => '0.00',
                                    ]
                                ); ?>
                            </div>
                        </div>
                        <br>
                    </fieldset>
                </div>
            </div>
        </div>
        <div id="appendAA">
        </div>

        <div class="col-md-2 text-right">
            <button type="button" class="btn btn-primary" id="firstAA" style="display:none;">Agregar Importe Asociados</button>
        </div>

        <!-- AA ROW-->
        <div class="row" style='display:none;' id="aaBlock">
            <div class="col-md-2"></div>
            <div class="col-md-10">
                <fieldset class="col-sm-11">
                    <legend>Importe Arriaga Asociados</legend>
                    <div id="aaRow">
                    </div>
                    <br>
                    <div class="row">
                        <div class="col-md-10"></div>
                        <div class="col-md-2 text-right">
                            <button type="button" class="btn btn-primary" onclick="duplicateAA()">+</button>
                        </div>
                    </div>
                    <br>


                    <br>
                    <div class="row">
                        <div class="col-md-9"></div>
                        <div class="col-md-3">
                            <?php echo $this->Form->input(
                                'total_imp_aa',
                                [
                                    'label' => 'Total imp. AA',
                                    'type' => 'number',
                                    'id' => 'total_imp_aa_',
                                    'readonly' => true,
                                ]
                            ); ?>
                        </div>
                    </div>
                    <br>
                </fieldset>
            </div>
        </div>

        <div id="arriagaRows_clone" style="display:none">
            <div class="row">

                <div class="col-md-6">
                    <?php
                    /*
                                    $customerAmountAssignmentClient  = isset($income['commitment_in_attorney_bank_account']) ? $income['commitment_in_attorney_bank_account'] : ''; #TODO corregir de donde viene este dato
                                    $source = isset($income['source']) ? $income['source'] : null;
                                    $description = isset($income['description']) ? $income['description'] : null;
                                    $procedure = isset($income['procedure']) ? $income['procedure'] : null;
                                    $amount  = isset($cost['amount']) ? $cost['amount'] : '';
                                    $stipulationsTemplate = isset($income['stipulations_template']) ? $income['stipulations_template'] : null;

                                    */

                    echo $this->Form->input(
                        'titular_aa[clone]',
                        [
                            'type' => 'select',
                            'options' => jsonc('Records.income_yes_or_no'),
                            'label' => 'Titular',
                            'empty' => '-- Seleccione una opción --',
                            'id' => 'titular_aa_0',
                        ]
                    ); ?>
                </div>
                <div class="col-md-6">
                    <?php echo $this->Form->input(
                        'origen_aa[clone]',
                        [
                            'type' => 'select',
                            'options' => jsonc('Records.income_source_aa'),
                            'label' => 'Origen',
                            'empty' => '-- Seleccione una opción --',
                            'id' => 'origen_aa_clone',
                        ]
                    ); ?>
                </div>
            </div>
            <br>
            <div class="row">
                <div class="col-md-4">
                    <?php echo $this->Form->input(
                        'concepto[clone]',
                        [
                            'type' => 'select',
                            'options' => jsonc('Records.income_source_aa'),
                            'label' => 'Concepto',
                            'empty' => '-- Seleccione una opción --',
                            'id' => 'concepto_clone',
                        ]
                    ); ?>
                </div>
                <div class="col-md-4">
                    <?php echo $this->Form->input(
                        'procedimiento[clone]',
                        [
                            'type' => 'select',
                            'options' => jsonc('Records.income_source_aa'),
                            'label' => 'Procedimiento',
                            'empty' => '-- Seleccione una opción --',
                            'id' => 'procedimiento_clone',
                        ]
                    ); ?>
                </div>
                <div class="col-md-3">
                    <?php echo $this->Form->input(
                        'amount_aa[clone]',
                        [
                            'type' => 'number',
                            'step' => '0.01',
                            'min' => 0,
                            'placeholder' => 'Ingrese el importe',
                            'label' => 'Importe',
                            'id' => 'importe_aa_clone',
                            'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        ]
                    ); ?>
                </div>
                <div class="col-md-1 align-bottom">
                    <br>
                    <button type="button" class="btn btn-primary round-button" onclick="generateLine(0)" id="addLineButton_0">+</button>
                </div>
            </div>
            <br>
            <div class="row">
                <div class="col-md-11">
                    <span style="font-weight: 700;">Estipulaciones</span><br>
                    <?php echo $this->Form->textarea(
                        'estipulaciones[clone]',
                        [
                            'label' => 'Estipulaciones',
                            'id' => 'estipulaciones_clone',
                            'rows' => 2,
                            'cols' => 50,
                            'placeholder' => 'Escribe aquí tu comentario...',
                        ]
                    ); ?>
                </div>
            </div>
        </div>

        <!-- Tabla para mostrar los datos ingresados -->
        <table class="table" id="aaDataTable_clone" style="display:none;">
            <thead>
                <tr>
                    <th>Concepto</th>
                    <th>Origen</th>
                    <th>Importe</th>
                    <th>Estipulaciones</th>
                </tr>
            </thead>
            <tbody>

            </tbody>
            <tfoot>
                <tr>
                    <td colspan="2" class="text-right"><strong>Total:</strong></td>
                    <td id="totalAmount_clone" readonly>0.00</td>
                    <td></td>
                </tr>
            </tfoot>
        </table>

        <br>
        <!-- Other imports ROW-->
        <div class="row">
            <div class="col-md-2"></div>
            <div class="col-md-10">
                <fieldset class="col-sm-11">
                    <legend>Otros Importes</legend>
                    <div id="otherRows">
                        <div class="row">
                            <div class="col-md-3">
                                <?php echo
                                /*
                                $totalAttorneyAmount  = isset($income['total_attorney_amount']) ? $income['total_attorney_amount'] : '';
                                $totalCourtRefundAmount  = isset($income['total_court_refund_amount']) ? $income['total_court_refund_amount'] : '';
                                */
                                $this->Form->input(
                                    'procurador',
                                    [
                                        'type' => 'number',
                                        'step' => '0.01',
                                        'min' => 0,
                                        'placeholder' => 'Ingrese el importe del procurador',
                                        'label' => 'Procurador',
                                        'id' => 'procurador_',
                                        // 'value' => $totalAttorneyAmount,
                                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                                    ]
                                ); ?>
                            </div>
                            <div class="col-md-3">
                                <?php echo $this->Form->input(
                                    'dev_juzgado_asignacion',
                                    [
                                        'type' => 'number',
                                        'step' => '0.01',
                                        'min' => 0,
                                        'placeholder' => 'Ingrese el importe del juzgado',
                                        'label' => 'Dev. Juzgado',
                                        'id' => 'dev_juzgado_asignacion_',
                                        //  'value' => $totalCourtRefundAmount,
                                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                                    ]
                                ); ?>
                            </div>
                            <div class="col-md-3"></div>
                            <div class="col-md-3">
                                <?php echo $this->Form->input(
                                    'total_otros_importes',
                                    [
                                        'label' => 'Total Otros Importes',
                                        'type' => 'number',
                                        // 'value' => '',
                                        'id' => 'total_otros_importes_',
                                        'readonly' => true,
                                    ]
                                ); ?>
                            </div>
                        </div>
                    </div>
                    <br>
                </fieldset>
            </div>
        </div>

        <hr class="custom-hr" />
        <br>

        <div class="row">
            <div class="col-md-2">
                <h4>Confirmación</h4>
            </div>
            <div class="col-md-10">
                <div class="row">
                    <div class="col-md-2">
                        <?php echo
                        /*
                        $ownChallengeDate = isset($income['own_challenge_date']) ? $income['own_challenge_date']->format('d/m/Y') : ''; 
                        */
                        $this->Form->input(
                            'albaranes',
                            [
                                'label' => 'Albaranes',
                                'type' => 'number',
                                //'value' => '',
                                'id' => 'albaranes_',
                                'readonly' => true,
                            ]
                        ); ?>
                    </div>
                    <div class="col-md-2">
                        <?php echo
                        $this->Form->input(
                            'pagos_a_clientes',
                            [
                                'label' => 'Pagos a Clientes',
                                'type' => 'number',
                                // 'value' => '',
                                'id' => 'pagos_a_clientes_',
                                'readonly' => true,
                            ]
                        );
                        ?>
                    </div>
                    <div class="col-md-2">
                        <?php echo
                        $this->Form->input(
                            'pagos_a_aa',
                            [
                                'label' => 'Pagos a AA',
                                'type' => 'number',
                                //'value' => '',
                                'id' => 'pagos_a_aa_',
                                'readonly' => true,
                            ]
                        );
                        ?>
                    </div>

                    <div class="col-md-2">
                        <?php echo
                        $this->Form->input(
                            'asig_procurador',
                            [
                                'label' => 'Asig. Procurador',
                                'type' => 'number',
                                // 'value' => '',
                                'id' => 'asig_procurador_',
                                'readonly' => true,
                            ]
                        );
                        ?>
                    </div>

                    <div class="col-md-2">
                        <?php echo
                        $this->Form->input(
                            'dev_juzgado',
                            [
                                'label' => 'Dev. Juzgado',
                                'type' => 'number',
                                //'value' => '',
                                'id' => 'dev_juzgado_',
                                'readonly' => true,
                            ]
                        );
                        ?>
                    </div>

                    <div class="col-md-2">
                        <?php echo
                        $this->Form->input(
                            'fecha_confirmacion',
                            [
                                'type' => 'text',
                                'class' => 'form-control datepicker',
                                'class' => 'datepicker',
                                'placeholder' => '__/__/____',
                                'label' => 'F. Confirmación',
                                'id' => 'fecha_confirmacion_',
                                // 'value' => $ownChallengeDate,
                                'autocomplete' => 'off',
                            ]
                        );
                        ?>
                    </div>
                </div>
            </div>
        </div>

        <?php echo $this->Form->end(); ?>
    <?php else: ?>
        <div class="alert alert-warning">
            La situación del expediente no permite realizar ninguna acción.
        </div>
    <?php endif; ?>

</div>



<script type="text/javascript">
    let ccCount = 0;
    let aaCount = 0;
    let addedProcedures = [];

    $("#firstClient").click(function() {

        if ($("#clientBlock").is(":hidden")) {
            $("#clientBlock").show();
            $(this).hide();
        }
        duplicateClients();
    });

    function duplicateClients() {
        ccCount++;
        var $newCC = $("#ccRow_clone").clone();
        $newCC.find("input, select, textarea").val("");
        $newCC.find("select").prop("selectedIndex", 0);

        $newCC.attr("id", "ccRow_" + ccCount);
        $newCC.show();


        $newCC.find("select, input, textarea").each(function() {
            const oldId = $(this).attr("id");
            const newId = oldId.replace(/_clone$/, `_${ccCount}`);
            $(this).attr("id", newId);

            const oldName = $(this).attr("name");
            const newName = oldName.replace(/\[clone\]/, `[${ccCount}]`);
            $(this).attr("name", newName);
        });

        $newCC.find("label").each(function() {
            const oldFor = $(this).attr("for");
            if (oldFor) {
                const newFor = oldFor.replace(/_clone$/, `_${ccCount}`);
                $(this).attr("for", newFor);
            }
        });


        $("#clientRows").append(`
        <div class="row">
            <div class="col-md-2 text-right">
                <button type="button" class="btn btn-secondary toggle-client-block" data-target="ccRow_${ccCount}">▲</button>
            </div>
        </div>
    `);


        $newCC.prepend(`
        <div class="row">
            <div class="col-md-2 text-right">
                <button type="button" class="btn btn-danger" onclick="removeCC(${ccCount})">Eliminar</button>
            </div>
        </div>
    `);


        $("#clientRows").append("<hr>").append($newCC);


    }

    $(document).on("input change", "input[id^='importe_cliente_']", function() {
        calculateTotalImportClients();
    });

    function calculateTotalImportClients() {
        let total = 0;

        $("[id^='importe_cliente_']").each(function() {
            total += parseFloat($(this).val()) || 0;
        });


        $("#total_importe_cliente_").val(total.toFixed(2));
    }


    function validateIBAN(pais, codigoIBAN, numeroCuenta) {

        pais = pais.toUpperCase();
        const iban = `${pais}${codigoIBAN}${numeroCuenta}`;

        console.log("IBAN completo:", iban);


        const longitudIBAN = {
            ES: 24,
            FR: 27,
            IT: 27,
            DE: 22,
            PT: 25,
            GB: 22,
        };


        if (!longitudIBAN[pais] || iban.length !== longitudIBAN[pais]) {
            return false;
        }


        const ibanReordenado = iban.slice(4) + iban.slice(0, 4);

        const ibanNumerico = ibanReordenado.replace(/[A-Z]/g, function(char) {
            return char.charCodeAt(0) - 55;
        });
        const mod97 = BigInt(ibanNumerico) % 97n;
        return mod97 === 1n;
    }

    $(document).on("change", "select[id^='pais_'], input[id^='codigo_iban_'], input[id^='numero_cuenta_']", function() {
        const idSuffix = $(this).attr("id").split("_").pop();
        const pais = $(`#pais_${idSuffix}`).val();
        const codigoIBAN = $(`#codigo_iban_${idSuffix}`).val();
        const numeroCuenta = $(`#numero_cuenta_${idSuffix}`).val();

        if (pais && codigoIBAN && numeroCuenta) {
            if (validateIBAN(pais, codigoIBAN, numeroCuenta)) {
                alert("IBAN válido");
            } else {
                alert("IBAN inválido, vuelva a ingresarlo");
                $(`#codigo_iban_${idSuffix}`).val('');
                $(`#numero_cuenta_${idSuffix}`).val('');
                $(`#pais_${idSuffix}`).val('');
            }
        }
    });




    function removeCC(counter) {
        $("#importe_cliente_" + counter).val(0);
        calculateTotalImportClients();
        var row = $("#ccRow_" + counter);
        row.prev("hr").remove();


        row.remove();

        $(".toggle-client-block[data-target='ccRow_" + counter + "']").remove();

        if ($("#clientRows").children("div[id^='ccRow_']").length === 0) {
            $("#ccRow_").hide();
            $("#firstClient").show();
            $("#clientBlock").hide();
        }
        ccCount--;

    }

    function removeC() {
        const clientRows = $("#clientRows").children("div[id^='ccRow_']");
        $("[id^='importe_cliente_']").val(0);
        calculateTotalImportClients();
        if (clientRows.length > 0) {
            clientRows.each(function() {
                $(this).prev("hr").remove();

                const rowId = $(this).attr('id');
                $(".toggle-client-block[data-target='" + rowId + "']").remove();

                $(this).remove();
            });
        }

        calculateTotalImportClients();
        $("#firstClient").show();

        $("#clientBlock").hide();
        ccCount = 0;
    }








    function generateLine(groupIndex) {
        const concepto = $(`#concepto_${groupIndex}`).val();
        const origen = $(`#origen_aa_${groupIndex}`).val();
        console.log(origen);
        const importe = parseFloat($(`#importe_aa_${groupIndex}`).val()) || 0;
        const estipulaciones = $(`#estipulaciones_${groupIndex}`).val();
        const procedimiento = $(`#procedimiento_${groupIndex}`).val();

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


        if (isDuplicate) {

            return;
        }


        addedProcedures.push({
            origen: origen,
            procedimiento: procedimiento
        });

        const newRow = `
        <tr data-group="${groupIndex}">
            <td>${concepto}</td>
            <td>${origen}</td>
            <td class="importe-value">${importe.toFixed(2)}</td>
            <td>${estipulaciones}</td>
            <td><button type="button" class="btn btn-danger" onclick="removeLine(this, ${groupIndex})">Eliminar</button></td>
        </tr>
    `;

        $(`#aaDataTable_${groupIndex} tbody`).append(newRow);


        updateTotalAmount(groupIndex);
        calculateTotalImportAA();
    }





    function updateTotalAmount(groupIndex) {
        let total = 0;



        $(`#aaDataTable_${groupIndex} .importe-value`).each(function() {

            const value = parseFloat($(this).text()) || 0;


            total += value;
        });



        $(`#totalAmount_${groupIndex}`).text(total.toFixed(2));

    }



    $("#firstAA").click(function() {
        $("#aaBlock").show();
        $(this).hide();
        duplicateAA();
    });




    function duplicateAA() {
        var $newAA = $("#arriagaRows_clone").clone();
        $newAA.find("input, select, textarea").val("");
        $newAA.find("select").prop("selectedIndex", 0);

        $newAA.attr("id", "arriagaRows_" + aaCount);

        $newAA.css("display", "");

        $newAA.find("select, input, textarea").each(function() {
            const oldId = $(this).attr("id");
            const newId = oldId.replace(/_clone$/, `_${aaCount}`);
            $(this).attr("id", newId);

            const baseName = $(this).attr("name").replace(/\[clone\]/, `[${aaCount}]`);
            $(this).attr("name", baseName);
        });

        $newAA.find("label").each(function() {
            const oldFor = $(this).attr("for");
            if (oldFor) {
                const newFor = oldFor.replace(/_clone$/, `_${aaCount}`);
                $(this).attr("for", newFor);
            }
        });

        $("#aaRow").append(`
        <div class="row">
            <div class="col-md-2 text-right">
                <button type="button" class="btn btn-secondary toggle-arriaga-block" data-target="arriagaRows_${aaCount}">▲</button>
            </div>
        </div>
    `);

        $newAA.prepend(`
        <div class="row">
            <div class="col-md-2 text-right">
                <button type="button" class="btn btn-danger" onclick="removeAA(${aaCount})">Eliminar</button>
            </div>
        </div>
    `);




        $newAA.find("button.round-button").attr("onclick", `generateLine(${aaCount})`);
        $newAA.find("button.round-button").attr("id", `addLineButton_${aaCount}`);


        $("#aaRow").append("<hr>").append($newAA);


        var $newTable = $("#aaDataTable_clone").clone();
        $newTable.find("tbody").empty();


        $newTable.attr("id", "aaDataTable_" + aaCount);
        $newTable.find("#totalAmount_clone").attr("id", "totalAmount_" + aaCount);
        $newTable.css("display", "");


        $("#arriagaRows_" + aaCount).append($newTable);
        handleNonJudicialIncomeChange();

        aaCount++;
    }

    $(document).on('click', '.toggle-arriaga-block', function() {

        var targetId = $(this).data('target');
        var $incomeBlockContent = $('#' + targetId);


        if ($incomeBlockContent.is(':visible')) {

            $incomeBlockContent.hide();
            $(this).html('&#9660;');
        } else {

            $incomeBlockContent.show();
            $(this).html('&#9650;');
        }
    });
    $(document).on('click', '.toggle-client-block', function() {

        var targetId = $(this).data('target');
        var $incomeBlockContent = $('#' + targetId);


        if ($incomeBlockContent.is(':visible')) {

            $incomeBlockContent.hide();
            $(this).html('&#9660;');
        } else {

            $incomeBlockContent.show();
            $(this).html('&#9650;');
        }
    });


    function observeRowChanges() {
        const targetNode = document;;

        const config = {
            childList: true,
            subtree: true
        };

        const observer = new MutationObserver(function(mutationsList) {
            for (const mutation of mutationsList) {
                if (mutation.type === 'childList') {
                    checkClientCount();
                    checkArriagaCount();
                    calculateTotalImportAA();
                    checkAlbaranes();
                }
            }
        });

        observer.observe(targetNode, config);

        return observer;
    }
    const observer = observeRowChanges();


    function removeA() {
        const observer = observeRowChanges();

        const aaRows = $("#aaRow").children("div[id^='arriagaRows_']");
        calculateTotalImportAA();
        if (aaRows.length > 0) {
            aaRows.each(function() {
                $(this).prev("hr").remove();

                const rowId = $(this).attr('id');
                $(".toggle-arriaga-block[data-target='" + rowId + "']").remove();

                $(this).remove();
            });
        }
        calculateTotalImportAA();
        clearAddedProcedures();
        $("#firstAA").show();


        $("#aaBlock").hide();
        aaCount = 0;

    }

    function removeAA(counter) {
        const observer = observeRowChanges();
        var row = $("#arriagaRows_" + counter);
        row.prev("hr").remove();


        row.remove();


        $("#aaDataTable_" + counter).remove();



        $(".toggle-arriaga-block[data-target='arriagaRows_" + counter + "']").remove();



        if ($("#aaRow").children("div[id^='arriagaRows_']").length === 0) {
            $("#aaBlock").css("display", "none");
            $("#firstAA").css("display", "inline-block");
        }
        calculateTotalImportAA();
        clearAddedProcedures();
        aaCount--;

    }



    function clearAddedProcedures() {
        addedProcedures = [];

    }



    function showTable(counter) {

        var $newTable = $("#aaDataTable_clone").clone();
        $newTable.find("tbody").empty();


        $newTable.attr("id", "aaDataTable_" + counter);
        $newTable.find("#totalAmount_clone").attr("id", "totalAmount_" + counter);


        $newTable.css("display", "table");


        $("#arriagaRows_" + counter).append($newTable);
    }




    function removeLine(button, groupIndex) {

        const row = $(button).closest("tr");


        const origen = row.find("td").eq(1).text();
        const procedimiento = row.find("td").eq(0).text();


        row.remove();


        addedProcedures = addedProcedures.filter(item => !(item.origen === origen && item.procedimiento === procedimiento));


        updateTotalAmount(groupIndex);
        calculateTotalImportAA();
    }





    function calculateTotalImportAA() {
        let totalImport = 0;


        $("td[id^='totalAmount_']").each(function() {
            totalImport += parseFloat($(this).text()) || 0;
        });


        $("#total_imp_aa_").val(totalImport.toFixed(2));
    }

    function initializeDatepickers() {
        $('.datepicker').datepicker({
            format: 'dd/mm/yyyy',
            autoclose: true,
            todayHighlight: true,
        });
    }

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
                    titularAA: $(`totular_aa_`),
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



    function setDefaultValuesIncome() {

        const fields = getFieldsByIdIncome();
        fields.fase.fase.val(''),
            fields.fase.ingresoNoJudicial.val(0),


            //block record

            fields.grabacion.origen.val(''),
            fields.grabacion.nCuentaJuzgado.val(''),
            fields.grabacion.impIngreso.val(''),
            fields.grabacion.fechaConstancia.val(''),

            //block consolidation

            fields.consolidacion.enCuentaProcurador.val(''),
            fields.consolidacion.tipoSolicitud.val(''),
            fields.consolidacion.fechaSolicitud.val(''),
            fields.consolidacion.ubicacionMp.val(''),
            fields.consolidacion.fechaExpedicionMp.val(''),
            fields.consolidacion.fechaCaducidadMp.val(''),
            fields.consolidacion.tasaPagadaMp.val(''),
            fields.consolidacion.fechaConsolidacion.val(''),
            fields.consolidacion.destinoConsolidacion.val(''),

            //block  commitment clients

            fields.asignacion.clientes.titularCliente.val(''),
            fields.asignacion.clientes.modoDeEntrega.val(''),
            fields.asignacion.clientes.importeCliente.val(''),
            fields.asignacion.clientes.pais.val(''),
            fields.asignacion.clientes.codigoIban.val(''),
            fields.asignacion.clientes.numeroCuenta.val(''),

            //block commitment aa

            fields.asignacion.aa.titularAA.val(''),
            fields.asignacion.aa.origenAA.val(''),
            fields.asignacion.aa.concepto.val(''),
            fields.asignacion.aa.procedimiento.val(''),
            fields.asignacion.aa.amountAA.val(''),
            fields.asignacion.aa.estipulaciones.val(''),

            //block commitment other imports

            fields.asignacion.otrosImportes.procurador.val(''),
            fields.asignacion.otrosImportes.devJuzgadoAsignacion.val(''),

            //block agreement

            fields.confirmacion.fechaConfirmacion.val('');






        $('input[type="checkbox"]').each(function() {
            $(this).prop('checked', false);
        });
    }

    function setDisabledCheckboxes(fields) {
        $.each(fields, function(key, field) {

            var $field = $(field);

            if ($field.is(':checkbox')) {
                $field.on('click.prevent', function(event) {
                    event.preventDefault();
                });
            } else {
                $field.attr('readonly', true).addClass('readonly-select');
            }
        });
    }

    function handleOriginChange() {
        const fields = getFieldsByIdIncome();
        const originSelector = $(`#origen_`);
        const selectedOrigin = originSelector.find('option:selected').text().toLowerCase();

        if (selectedOrigin === 'cuenta juzgado') {
            $(`#n_cuenta_juzgado_`).show();
            $(`label[for="n_cuenta_juzgado_"]`).show();
        } else {
            $(`#n_cuenta_juzgado_`).hide();
            $(`label[for="n_cuenta_juzgado_"]`).hide();
            fields.grabacion.nCuentaJuzgado.val('');

        }
    }

    function handleclientDelivery(elementId) {
        const fields = getFieldsByIdIncome();
        const requestSelector = $(`#${elementId}`);
        const selectedRequest = requestSelector.find('option:selected').text().toLowerCase();
        const idSuffix = elementId.split("_").pop();

        if (selectedRequest === 'transferencia bancaria') {

            $(`#pais_${idSuffix}`).show();
            $(`label[for="pais_${idSuffix}"]`).show();

            $(`#codigo_iban_${idSuffix}`).show();
            $(`label[for="codigo_iban_${idSuffix}"]`).show();

            $(`#numero_cuenta_${idSuffix}`).show();
            $(`label[for="numero_cuenta_${idSuffix}"]`).show();
        } else {

            $(`#pais_${idSuffix}`).hide();
            $(`label[for="pais_${idSuffix}"]`).hide();
            if (fields.asignacion.clientes[`pais`]) {
                $(`#pais_${idSuffix}`).val('');
            }

            $(`#codigo_iban_${idSuffix}`).hide();
            $(`label[for="codigo_iban_${idSuffix}"]`).hide();
            if (fields.asignacion.clientes[`codigoIban`]) {
                $(`#codigo_iban_${idSuffix}`).val('');
            }

            $(`#numero_cuenta_${idSuffix}`).hide();
            $(`label[for="numero_cuenta_${idSuffix}"]`).hide();
            if (fields.asignacion.clientes[`numeroCuenta`]) {
                $(`#numero_cuenta_${idSuffix}`).val('');
            }
        }
    }

    function setupDynamicDeliveryMethodListener() {
        $(document).on('change', '[id^="modo_de_entrega_"]', function() {
            handleclientDelivery(this.id);
        });
    }

    function handleRequestChange() {
        const fields = getFieldsByIdIncome();
        const requestSelector = $(`#tipo_solicitud_`);
        const selectedRequest = requestSelector.find('option:selected').text().toLowerCase();



        if (selectedRequest === 'mandamiento de pago') {
            $(`#ubicacion_mp_`).show();
            $(`label[for="ubicacion_mp_"]`).show();

            $(`#fecha_expedicion_mp_`).show();
            $(`label[for="fecha_expedicion_mp_"]`).show();

            $(`#fecha_caducidad_mp_`).show();
            $(`label[for="fecha_caducidad_mp_"]`).show();

            $(`#tasa_pagada_mp_`).show();
            $(`label[for="tasa_pagada_mp_"]`).show();
        } else {
            $(`#ubicacion_mp_`).hide();
            $(`label[for="ubicacion_mp_"]`).hide();
            fields.consolidacion.ubicacionMp.val('');


            $(`#fecha_expedicion_mp_`).hide();
            $(`label[for="fecha_expedicion_mp_"]`).hide();
            fields.consolidacion.fechaExpedicionMp.val('');

            $(`#fecha_caducidad_mp_`).hide();
            $(`label[for="fecha_caducidad_mp_"]`).hide();
            fields.consolidacion.fechaCaducidadMp.val('');

            $(`#tasa_pagada_mp_`).hide();
            $(`label[for="tasa_pagada_mp_"]`).hide();
            fields.consolidacion.tasaPagadaMp.val('');

        }
    }

    function handleAttorneyChange() {
        const attorneyValue = $(`#en_cuenta_procurador_`).val();
        const requestTypeField = $(`#tipo_solicitud_`);
        const requestDateField = $(`#fecha_solicitud_`);
        const locationDestinyField = $(`#destino_consolidacion_`);

        if (attorneyValue === 'no') {

            requestTypeField.prop('readonly', false).removeClass('readonly-select').prop('required', true);
            requestDateField.prop('readonly', false).prop('required', true);
            locationDestinyField.prop('readonly', false).removeClass('readonly-select').prop('required', true).val('');
            handleRequestChange();
        } else if (attorneyValue === 'si') {

            requestTypeField.prop('readonly', true).addClass('readonly-select').val('');
            requestDateField.prop('readonly', true).val('');
            locationDestinyField.prop('readonly', true).addClass('readonly-select').val('cuenta procurador');
            handleRequestChange();
        }
    }

    function handleNonJudicialIncomeChange() {
        const nonJudicialCheckbox = $(`#ingreso_no_judicial_`);
        const originField = $(`#origen_`);
        const procedimientoFields = $('[id^="procedimiento_"]');
        const procedimientoLabels = $(`label[for^="procedimiento_"]`);

        const origenFields = $('[id^="origen_aa_"]');

        origenFields.each(function() {
            const select = $(this);

            if (nonJudicialCheckbox.is(':checked')) {
                select.find('option[value="ingreso no judicial"]').show();

            } else {
                select.find('option[value="ingreso no judicial"]').hide();

            }
        });

        if (nonJudicialCheckbox.is(':checked')) {
            originField.val('cuenta cliente').prop('readonly', true).addClass('readonly-select');
            origenFields.val('ingreso no judicial').prop('readonly', true).addClass('readonly-select');
            procedimientoFields.hide();
            procedimientoLabels.hide();
        } else {
            originField.prop('readonly', false).removeClass('readonly-select');
            origenFields.prop('readonly', false).removeClass('readonly-select');
            origenFields.val('');
            originField.val('');
            procedimientoFields.show();
            procedimientoLabels.show();
        }
    }


    let isConfirmingPhaseChange = false;

    function capturePhaseIncome() {
        const originSelector = $(`#origen_`);
        const requestSelector = $(`#tipo_solicitud_`);
        const phaseSelector = $(`#fase_`);
        const attorneySelector = $(`#en_cuenta_procurador_`);
        const nonJudicialCheckbox = $(`#ingreso_no_judicial_`);
        const deliverySelector = $(`#modo_de_entrega_`);

        handleOriginChange();
        handleRequestChange();
        handleAttorneyChange();
        handleNonJudicialIncomeChange();

        const defaultPhase = phaseSelector.find('option:selected').text().toLowerCase();
        updateFieldsIncome(defaultPhase);

        attorneySelector.on('change', function() {
            handleAttorneyChange();
        });

        originSelector.on('change', function() {

            handleOriginChange();
        });

        requestSelector.on('change', function() {

            handleRequestChange();
        });

        nonJudicialCheckbox.on('change', function() {
            handleNonJudicialIncomeChange();
        });

        const phasesOrder = ['-- seleccione una opción --', 'grabación', 'consolidación', 'asignación', 'confirmación'];
        let currentPhase = phaseSelector.find('option:selected').text().toLowerCase();


        phaseSelector.off('change').on('change', function() {
            const selectedPhase = $(this).find('option:selected').text().toLowerCase();


            const currentPhaseIndex = phasesOrder.indexOf(currentPhase);
            const selectedPhaseIndex = phasesOrder.indexOf(selectedPhase);

            if (selectedPhaseIndex > currentPhaseIndex + 1) {
                alert("No puedes avanzar más de una fase a la vez.");

                currentPhase === '-- seleccione una opción --' ?
                    $('#fase_').val('') :
                    phaseSelector.val(normalizeText(currentPhase));

                return;
            }

            if (selectedPhaseIndex < currentPhaseIndex) {
                if (!isConfirmingPhaseChange) {
                    const confirmChange = confirm("Vas a retroceder de fase. Los datos de las fases posteriores se borrarán. ¿Deseas continuar?");
                    if (confirmChange) {
                        isConfirmingPhaseChange = true;
                        updateFieldsIncome(selectedPhase);
                        currentPhase = selectedPhase;
                    } else {

                        phaseSelector.val(normalizeText(currentPhase));

                    }
                }
            } else {
                isConfirmingPhaseChange = false;
                updateFieldsIncome(selectedPhase);
                currentPhase = selectedPhase;
            }
        });
    }

    function normalizeText(text) {
        return text.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '');
    }

    $('#fase_').on('change', capturePhaseIncome);


    function updateFieldsIncome(selectedPhase) {
        const fields = getFieldsByIdIncome();
        const buttonClientAdd = $('#firstClient');
        const buttonArriagaAdd = $('#firstAA');
        const buttonAAAdd = document.getElementById(`add_aa_`);

        $.each(fields, function(phase, phaseFields) {
            $.each(phaseFields, function(key, field) {

                $(field).removeAttr('readonly').removeClass('readonly-select');
                $(field).off('click.prevent');
            });
        });
        if (selectedPhase === '-- seleccione una opción --') {

            $('#ingreso_no_judicial_').prop('checked', false);


            removeC();
            removeA();
            buttonClientAdd.hide();
            buttonArriagaAdd.hide();
            setDisabledCheckboxes(fields.grabacion);
            setDisabledCheckboxes(fields.consolidacion);
            setDisabledCheckboxes(fields.asignacion.clientes);
            setDisabledCheckboxes(fields.asignacion.aa);
            setDisabledCheckboxes(fields.asignacion.otrosImportes);
            setDisabledCheckboxes(fields.confirmacion);


            clearAllFieldsIncome('grabacion');
            clearAllFieldsIncome('consolidacion');
            clearAllFieldsIncome('asignacion');
            clearAllFieldsIncome('confirmacion');

            //$('#n_cuenta_juzgado_').closest('div.input.select').hide();
        } else if (selectedPhase === 'grabación') {
            removeC();
            removeA();
            buttonClientAdd.hide();
            buttonArriagaAdd.hide();
            $.each(fields.grabacion, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');
            });
            setDisabledCheckboxes(fields.fase.ingresoNoJudicial);
            setDisabledCheckboxes(fields.consolidacion);
            setDisabledCheckboxes(fields.asignacion.aa);
            setDisabledCheckboxes(fields.asignacion.otrosImportes);
            setDisabledCheckboxes(fields.confirmacion);

            clearAllFieldsIncome('grabacion');
            clearAllFieldsIncome('consolidacion');
            clearAllFieldsIncome('asignacion');
            clearAllFieldsIncome('confirmacion');
        } else if (selectedPhase === 'consolidación') {
            removeC();
            removeA();

            buttonClientAdd.hide();
            buttonArriagaAdd.hide();
            setDisabledCheckboxes(fields.fase.ingresoNoJudicial);
            setDisabledCheckboxes(fields.grabacion);
            $.each(fields.consolidacion, function(key, field) {
                field.prop('readonly', false).removeClass('readonly-select');
            });



            setDisabledCheckboxes(fields.asignacion.aa);
            setDisabledCheckboxes(fields.asignacion.otrosImportes);
            setDisabledCheckboxes(fields.confirmacion);

            clearAllFieldsIncome('consolidacion');
            clearAllFieldsIncome('asignacion');
            clearAllFieldsIncome('confirmacion');

        } else if (selectedPhase === 'asignación') {
            buttonClientAdd.show();
            buttonArriagaAdd.show();
            setDisabledCheckboxes(fields.fase.ingresoNoJudicial);
            setDisabledCheckboxes(fields.grabacion);
            setDisabledCheckboxes(fields.consolidacion);
            $.each(fields.asignacion, function(sectionKey, sectionFields) {
                $.each(sectionFields, function(key, field) {
                    $(field).prop('readonly', false).removeClass('readonly-select');
                });
            });
            setDisabledCheckboxes(fields.confirmacion);

            clearAllFieldsIncome('asignacion');
            clearAllFieldsIncome('confirmacion');

        } else if (selectedPhase === 'confirmación') {
            buttonClientAdd.hide();
            buttonArriagaAdd.hide();
            setDisabledCheckboxes(fields.grabacion);
            setDisabledCheckboxes(fields.consolidacion);
            setDisabledCheckboxes(fields.asignacion.clientes);
            setDisabledCheckboxes(fields.asignacion.aa);
            setDisabledCheckboxes(fields.asignacion.otrosImportes);
            $.each(fields.confirmacion, function(key, field) {
                field.prop('readonly', false).removeClass('readonly-select');
            });

            clearAllFieldsIncome('confirmacion');

        }
    }
    /*
        function validateBeforeChangePhase(selectedPhase) {
            const fields = getFieldsByIdIncome();
            console.log(selectedPhase);

            let errorMessages = [];

            if (selectedPhase === 'grabación') {
                if (fields.grabacion.origen.val() === '') {
                    errorMessages.push('Por favor, seleccione un valor para el campo "Origen".');
                } else if (fields.grabacion.origen.val() === 'cuenta juzgado' && fields.grabacion.nCuentaJuzgado.val() === '') {
                    errorMessages.push('Por favor, seleccione un valor para el campo Mº Cuenta del Juzgado.');
                }

                const incomeValue = parseFloat(fields.grabacion.impIngreso.val());
                if (isNaN(incomeValue) || incomeValue <= 0) {
                    errorMessages.push('El campo "Importe Ingreso" es obligatorio y debe ser un valor positivo.');
                }

                if (fields.grabacion.fechaConstancia.val() === '') {
                    errorMessages.push('El campo "Fecha Constancia" es obligatorio.');
                }

            } else if (selectedPhase === 'consolidación') {

                if (fields.consolidacion.enCuentaProcurador.val() === 'si') {
                    if (fields.consolidacion.fechaConsolidacion.val() === '') {
                        errorMessages.push('Por favor, seleccione una opción en el campo Fecha de Consolidación.');
                    }
                } else if (fields.consolidacion.enCuentaProcurador.val() === 'no') {
                    if (fields.consolidacion.tipoSolicitud.val() === 'mandamiento de pago') {
                        if (fields.consolidacion.fechaSolicitud.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Fecha de Solicitud.');
                        }
                        if (fields.consolidacion.ubicacionMp.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Ubicación MP');
                        }
                        if (fields.consolidacion.fechaExpedicionMp.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Fecha Expedición MP.');
                        }
                        if (fields.consolidacion.fechaCaducidadMp.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Fecha Caducidad MP.');
                        }
                        if (fields.consolidacion.tasaPagadaMp.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Tasa Pagada MP.');
                        }
                        if (fields.consolidacion.fechaConsolidacion.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Fecha de Consolidación.');
                        }
                        if (fields.consolidacion.destinoConsolidacion.val() === '') {
                            errorMessages.push('Por favor, seleccione una opción en el campo Destino de Consolidación.');
                        }
                    } else if (fields.consolidacion.tipoSolicitud.val() === '') {
                        errorMessages.push('Por favor, seleccione una opción en el campo Tipo de Solicitud.');
                    }
                } else {
                    errorMessages.push('Por favor, seleccione una opción en el campo En Cuenta de Procurador.');
                }
            } else if (selectedPhase === 'asignación') {

            }

            if (errorMessages.length > 0) {
                alert(errorMessages.join('\n'));
                return false;
            }

            return true;
        }

        */
    $(document).on('click', '.toggle-income-block', function() {

        var $incomeBlockContent = $(this).next();


        if ($incomeBlockContent.is(':visible')) {

            $incomeBlockContent.hide();
            $(this).html('&#9660;');
        } else {

            $incomeBlockContent.show();
            $(this).html('&#9650;');
        }
    });
    $('#firstClient').on('click', function() {
        $(this).hide();
    });


    function checkProcuradorValue() {

        const procuradorValue = $('#procurador_').val();


        if (procuradorValue !== "" && !isNaN(procuradorValue)) {
            $(`#asig_procurador_`).show();
            $(`label[for="asig_procurador_"]`).show();
            importe = $('#procurador_').val();
            $(`#asig_procurador_`).val(importe);
        } else {
            $(`#asig_procurador_`).hide();
            $(`label[for="asig_procurador_"]`).hide();
        }

    }
    $('#procurador_').on('input blur change', function() {
        checkProcuradorValue();
    });

    function checkDevJuzgadoValue() {

        const devValue = $('#dev_juzgado_asignacion_').val();


        if (devValue !== "" && !isNaN(devValue)) {
            $(`#dev_juzgado_`).show();
            $(`label[for="dev_juzgado_"]`).show();
            importe = $('#dev_juzgado_asignacion_').val();
            $(`#dev_juzgado_`).val(importe);
        } else {
            $(`#dev_juzgado_`).hide();
            $(`label[for="dev_juzgado_"]`).hide();
        }

    }
    $('#dev_juzgado_asignacion_').on('input blur change', function() {
        checkDevJuzgadoValue();

    });

    function updateTotalOtrosImportes() {

        const procuradorValue = parseFloat($('#procurador_').val()) || 0;
        const devJuzgadoValue = parseFloat($('#dev_juzgado_asignacion_').val()) || 0;


        const totalOtrosImportes = procuradorValue + devJuzgadoValue;


        $('#total_otros_importes_').val(totalOtrosImportes.toFixed(2));
    }


    $('#procurador_, #dev_juzgado_asignacion_').on('input change', function() {
        updateTotalOtrosImportes();
    });

    $('#procurador_, #dev_juzgado_asignacion_').on('input change', function() {
        updateTotalOtrosImportes();
    });

    function checkClientCount() {

        const clientes = ccCount;

        $('#pagos_a_clientes_').val(clientes);
    }

    function checkArriagaCount() {

        const arriaga = aaCount;

        $('#pagos_a_aa_').val(arriaga);
    }

    function checkAlbaranes() {

        const clientes = ccCount;
        const arriaga = aaCount;

        const total = ccCount + aaCount;

        $('#albaranes_').val(arriaga);
    }


    $(document).ready(function() {
        initializeDatepickers();
        calculateTotalImportAA();
        setupDynamicDeliveryMethodListener();
        checkProcuradorValue();
        checkDevJuzgadoValue();
        //capturePhaseIncome();
    });
</script>
```