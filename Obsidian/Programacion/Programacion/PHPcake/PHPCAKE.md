
```
<style>
    .readonly-select {
        pointer-events: none;
        background-color: #f0f0f0;
    }

    .disabled-checkbox {
        pointer-events: none;

        opacity: 0.5;

    }

    .my-5 {
        margin-top: 3rem !important;
        margin-bottom: 3rem !important;
    }
</style>


<?php
$fechaPrimeraInstancia = isset($record->pro_fecha_sentencia_primera_instancia)
    ? $record->pro_fecha_sentencia_primera_instancia->format('d/m/Y')
    : '';

$fechaApeSentencia = isset($record->ape_sentencia)
    ? $record->ape_sentencia->format('d/m/Y')
    : '';


$judicializable = [];
$allowedSituations = [
    'ST FIRMES SIN APELACIÓN',
    'ST FIRMES CON APELACIÓN',
    'ST DESESTIMATORIAS FIRMES',
    'SENTENCIA DE CASACIÓN ESTIMATORIA',
    'SENTENCIA DE CASACIÓN DESESTIMATORIA',
    'ST DE APELACIÓN',
    'ST NO FIRME',
    'ST DESESTIMATORIAS APELADAS',
    'ST PROVISIONALES APELADAS',
    'RECURSO DE CASACIÓN',
    'CASACIÓN INADMITIDA'
];

$canAddCosts = !(in_array($record->status, $judicializable) || in_array($record->status, $allowedSituations));

$courtCosts = !empty($record->court_costs) ? $record->court_costs : [];
$incomeRecords = !empty($record->income) ? $record->income : [];


?>
<div id="cost-container">
    <?php
    if ($canAddCosts) {


        if (!empty($courtCosts)) {
            foreach ($courtCosts as $index => $cost) {


                $incomeExists = false;
                foreach ($incomeRecords as $income) {
                    if ($income->court_costs_id === $cost->id) {
                        $incomeExists = true;
                        break;
                    }
                }

                echo '<div id="court-cost-block-' . ($index + 1) . '" class="table-bordered well-lg my-5">';

    ?>
                <button type="button" class="btn btn-default fa fa-lg toggle-cost-block">&#9650;</button>
    <?php

                echo '<div class="cost-content" style="display: block;">';

                echo $this->Form->create($record, ['url' => ['controller' => 'Manage', 'action' => 'patchTC']]);
                echo '<div class="row">';
                echo '<div class="col-sm-2 col-sm-offset-10 text-right">';
                echo $this->Form->hidden('record_id', ['value' => $record->id]);
                echo $this->Form->hidden('pro_fecha_sentencia_primera_instancia', ['value' => $fechaPrimeraInstancia]);
                echo $this->Form->hidden('ape_sentencia', ['value' => $fechaApeSentencia]);
                echo $this->Form->hidden('court_cost_id', ['value' => $cost->id]);

                echo $this->Form->button('', [
                    'class' => 'btn btn-primary mr-2 fa fa-lg fa-floppy-o',
                    'type' => 'submit',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'patchTC']),
                ]);

                echo $this->Form->button('', [
                    'class' => 'btn btn-danger fa fa-lg fa-trash-o',
                    'type' => 'submit',
                    'name' => 'action',
                    'value' => 'delete',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'deleteTC', $cost->id]),
                    'onclick' => "return confirm('" . ($incomeExists ? "Se eliminarán todos los datos de tasación de costas, pero los documentos relacionados permanecerán en el expediente. ¿Desea continuar?" : "¿Está seguro de que desea eliminar esta tasación de costas?") . "');"
                ]);
                echo '</div>';
                echo '</div>';

                echo '<div class="row">';
                echo generateType($this, $index + 1, false, $cost);
                echo '</div>';
                echo presentation($this, $index + 1, $cost);
                echo transfer($this, $index + 1, $cost);
                echo decree($this, $index + 1, $cost);
                echo rulling($this, $index + 1, $cost);
                echo '</div>';
                echo $this->Form->end();
                echo '</div>';
            }
        }
    } else {

        echo '<div><p>No es posible la creación de una Tasación de Costas debido a la situación del expediente.</p></div>';
    }

    ?>
</div>

<div class="row">
    <div id="add-cost-button-container" class="col-sm-12 text-right">
        <button type="button" id="add-court-cost" class="btn btn-default fa fa-lg fa-plus" data-toggle="tooltip" data-original-title="Añadir tasación de costas"></button>
    </div>
</div>

<script>
    let costCounter = <?php echo count($courtCosts) > 0 ? count($courtCosts) : 0; ?>;


    document.getElementById('add-court-cost')?.addEventListener('click', function() {
        addCourtCostBlock();
    });
    document.getElementById('add-court-cost-first')?.addEventListener('click', function() {
        addCourtCostBlock();
    });





    function addCourtCostBlock() {
        costCounter++;

        let costBlock = document.createElement('div');
        costBlock.id = 'court-cost-block-' + costCounter;
        costBlock.classList.add('table-bordered', 'well-lg', 'my-5');
        costBlock.innerHTML = `
    
    <button type="button" class="btn btn-default toggle-cost-block fa fa-lg">&#9650;</button>
    <div class="cost-content" style="display: block;">
        <?php echo $this->Form->create($record, ['url' => ['controller' => 'Manage', 'action' => 'createTC']]); ?>
        <div class="row">
        <div class="col-sm-2 col-sm-offset-10 text-right">
        <?php echo $this->Form->hidden('record_id', ['value' => $record->id]); ?>
        <?php
        echo $this->Form->hidden('pro_fecha_sentencia_primera_instancia', ['value' => $fechaPrimeraInstancia]);
        echo $this->Form->hidden('ape_sentencia', ['value' => $fechaApeSentencia]); ?>
        <?php echo $this->Form->button('', ['class' => 'btn btn-primary mr-2 fa fa-lg fa-floppy-o', 'data-toggle' => 'tooltip', 'data-original-title' => __('Guardar')]); ?>
        <button type="button" class="btn btn-danger fa fa-lg fa-trash-o" data-toggle="tooltip" data-original-title="Eliminar tasación de costas"></button>
        </div>
        </div>
        <div class="row">
        <?php echo generateType($this, '${costCounter}', true); ?>
        </div>
        <?php echo presentation($this, '${costCounter}'); ?>
        <?php echo transfer($this, '${costCounter}'); ?>
        <?php echo decree($this, '${costCounter}'); ?>
        <?php echo rulling($this, '${costCounter}'); ?>
        <?php echo $this->Form->end(); ?>
    </div>
    `;

        document.getElementById('cost-container').appendChild(costBlock);

        const deleteButton = costBlock.querySelector('.btn-danger');

        deleteButton.addEventListener('click', function() {
            const id = costCounter;


            const confirmDelete = confirm("¿Estás seguro de que deseas restablecer los valores de este bloque de Tasación de Costas? Esta acción no se puede deshacer.");

            if (confirmDelete) {
                setDefaultValues(id);
                alert('Se han restablecido los valores del bloque de Tasación de Costas: ' + costBlock.id);
            } else {
                alert('Los valores del bloque de  Tasación de Costas no han sido restablecidos.');
            }
        });

        initializeDatepickers();
        capturePhase(costCounter);

    }


    function initializeDatepickers() {
        $('.datepicker').datepicker({
            format: 'dd/mm/yyyy',
            autoclose: true,
            todayHighlight: true,
            endDate: new Date()
        });
    }


    function setDefaultValues(id) {

        const fields = getFieldsById(id);

        fields.presentacion.fechaPresentacion.val('');

        fields.presentacion.impCostas.val('');
        fields.presentacion.impPerito.val('');
        fields.presentacion.impTestigo.val('');
        fields.presentacion.fechaCalculo.val('');

        fields.presentacion.impCostasPro.val('');
        fields.presentacion.impPeritoPro.val('');
        fields.presentacion.impTestigoPro.val('');
        fields.presentacion.fechaCalculoPro.val('');
        fields.presentacion.tasasPro.val('');

        fields.traslado.fechaTraslado.val('');

        fields.traslado.resultadoTasacion.val('');
        fields.traslado.importeLaj.val('');
        fields.traslado.lajDif.val('');
        fields.traslado.informeIca.val('');
        fields.traslado.fechaIca.val('');
        fields.traslado.impugnacionTraslado.val('');
        fields.traslado.tipoImpugnacion.val('');
        fields.traslado.fechaImpugnacion.val('');
        fields.traslado.impContrario.val('');
        fields.traslado.tipoImpugnacionContrario.val('');
        fields.traslado.fechaImpugnacionContrario.val('');
        fields.traslado.conformidadImpContrario.val('');


        fields.traslado.resultadoTasacionPro.val('');
        fields.traslado.importeLajPro.val('');
        fields.traslado.lajDifPro.val('');
        fields.traslado.impugnacionTrasladoPro.val('');
        fields.traslado.tipoImpugnacionPro.val('');
        fields.traslado.fechaImpugnacionPro.val('');
        fields.traslado.impContrarioPro.val('');
        fields.traslado.tipoImpugnacionContrarioPro.val('');
        fields.traslado.fechaImpugnacionContrarioPro.val('');
        fields.traslado.conformidadImpContrarioPro.val('');

        fields.decreto.fechaDecreto.val('');

        fields.decreto.resultadoDecreto.val('');
        fields.decreto.importeDecreto.val('');
        fields.decreto.decretoDif.val('');
        fields.decreto.recurso.val('');
        fields.decreto.fechaRecurso.val('');
        fields.decreto.impugnacionContrarioDecreto.val('');
        fields.decreto.fechaImpugnacionContrarioDecreto.val('');
        fields.decreto.recursoContrario.val('');
        fields.decreto.fechaRecursoContrarioDecreto.val('');
        fields.decreto.impugnacionRC.val('');
        fields.decreto.fechaImpugnacionRC.val('');
        fields.decreto.costasIncidenteDecreto.val('');
        fields.decreto.costasEcIncidenteDecreto.val('');
        fields.decreto.alegacionesImpContrario.val('');

        fields.decreto.resultadoDecretoPro.val('');
        fields.decreto.importeDecretoPro.val('');
        fields.decreto.decretoDifPro.val('');
        fields.decreto.recursoPro.val('');
        fields.decreto.fechaRecursoPro.val('');
        fields.decreto.impugnacionContrarioDecretoPro.val('');
        fields.decreto.fechaImpugnacionContrarioDecretoPro.val('');
        fields.decreto.recursoContrarioPro.val('');
        fields.decreto.fechaRecursoContrarioDecretoPro.val('');
        fields.decreto.impugnacionRCPro.val('');
        fields.decreto.fechaImpugnacionRCPro.val('');
        fields.decreto.costasIncidenteDecretoPro.val('');
        fields.decreto.costasEcIncidenteDecretoPro.val('');
        fields.decreto.alegacionesImpContrarioPro.val('');

        fields.resolucion.fechaResolucion.val('');

        fields.resolucion.resultadoResolucion.val('');
        fields.resolucion.importeResolucion.val('');
        fields.resolucion.costasIncidente.val('');
        fields.resolucion.costasEcIncidente.val('');

        fields.resolucion.resultadoResolucionPro.val('');
        fields.resolucion.importeResolucionPro.val('');
        fields.resolucion.costasIncidentePro.val('');
        fields.resolucion.costasEcIncidentePro.val('');


        $('input[type="checkbox"]').each(function() {
            $(this).prop('checked', false);
        });
    }




    function setDefaultBlock(id, block) {
        const fields = getFieldsById(id);


        switch (block) {
            case 'presentacion':
                fields.presentacion.fechaPresentacion.val('');

                fields.presentacion.impCostas.val('');
                fields.presentacion.impPerito.val('');
                fields.presentacion.impTestigo.val('');
                fields.presentacion.fechaCalculo.val('');

                fields.presentacion.impCostasPro.val('');
                fields.presentacion.impPeritoPro.val('');
                fields.presentacion.impTestigoPro.val('');
                fields.presentacion.fechaCalculoPro.val('');
                fields.presentacion.tasasPro.val('');


                cleanCheck();
                break;

            case 'traslado':
                fields.traslado.fechaTraslado.val('');

                fields.traslado.resultadoTasacion.val('');
                fields.traslado.importeLaj.val('');
                fields.traslado.lajDif.val('');
                fields.traslado.informeIca.val('');
                fields.traslado.fechaIca.val('');
                fields.traslado.impugnacionTraslado.val('');
                fields.traslado.tipoImpugnacion.val('');
                fields.traslado.fechaImpugnacion.val('');
                fields.traslado.impContrario.val('');
                fields.traslado.tipoImpugnacionContrario.val('');
                fields.traslado.fechaImpugnacionContrario.val('');
                fields.traslado.conformidadImpContrario.val('');


                fields.traslado.resultadoTasacionPro.val('');
                fields.traslado.importeLajPro.val('');
                fields.traslado.lajDifPro.val('');
                fields.traslado.impugnacionTrasladoPro.val('');
                fields.traslado.tipoImpugnacionPro.val('');
                fields.traslado.fechaImpugnacionPro.val('');
                fields.traslado.impContrarioPro.val('');
                fields.traslado.tipoImpugnacionContrarioPro.val('');
                fields.traslado.fechaImpugnacionContrarioPro.val('');
                fields.traslado.conformidadImpContrarioPro.val('');
                cleanCheck();
                break;

            case 'decreto':
                fields.decreto.fechaDecreto.val('');

                fields.decreto.resultadoDecreto.val('');
                fields.decreto.importeDecreto.val('');
                fields.decreto.decretoDif.val('');
                fields.decreto.recurso.val('');
                fields.decreto.fechaRecurso.val('');
                fields.decreto.impugnacionContrarioDecreto.val('');
                fields.decreto.fechaImpugnacionContrarioDecreto.val('');
                fields.decreto.recursoContrario.val('');
                fields.decreto.fechaRecursoContrarioDecreto.val('');
                fields.decreto.impugnacionRC.val('');
                fields.decreto.fechaImpugnacionRC.val('');
                fields.decreto.costasIncidenteDecreto.val('');
                fields.decreto.costasEcIncidenteDecreto.val('');
                fields.decreto.alegacionesImpContrario.val('');

                fields.decreto.resultadoDecretoPro.val('');
                fields.decreto.importeDecretoPro.val('');
                fields.decreto.decretoDifPro.val('');
                fields.decreto.recursoPro.val('');
                fields.decreto.fechaRecursoPro.val('');
                fields.decreto.impugnacionContrarioDecretoPro.val('');
                fields.decreto.fechaImpugnacionContrarioDecretoPro.val('');
                fields.decreto.recursoContrarioPro.val('');
                fields.decreto.fechaRecursoContrarioDecretoPro.val('');
                fields.decreto.impugnacionRCPro.val('');
                fields.decreto.fechaImpugnacionRCPro.val('');
                fields.decreto.costasIncidenteDecretoPro.val('');
                fields.decreto.costasEcIncidenteDecretoPro.val('');
                fields.decreto.alegacionesImpContrarioPro.val('');
                cleanCheck();
                break;

            case 'resolucion':
                fields.resolucion.fechaResolucion.val('');

                fields.resolucion.resultadoResolucion.val('');
                fields.resolucion.importeResolucion.val('');
                fields.resolucion.costasIncidente.val('');
                fields.resolucion.costasEcIncidente.val('');

                fields.resolucion.resultadoResolucionPro.val('');
                fields.resolucion.importeResolucionPro.val('');
                fields.resolucion.costasIncidentePro.val('');
                fields.resolucion.costasEcIncidentePro.val('');
                cleanCheck();
                break;

            default:
                console.warn('Bloque no reconocido: ' + block);
                break;
        }


        function cleanCheck() {
            $('input[type="checkbox"]').each(function() {
                $(this).prop('checked', false);
            });
        }
    }





    function getFieldsById(id) {
        return {

            presentacion: {
                impCostas: $(`#imp_costas_${id}`),
                impPerito: $(`#imp_perito_${id}`),
                impTestigo: $(`#imp_testigo_${id}`),
                fechaCalculo: $(`#fecha_calculo_${id}`),
                fechaPresentacion: $(`#fecha_presentacion_${id}`),

                impCostasPro: $(`#procurador_imp_costas_${id}`),
                impPeritoPro: $(`#procurador_imp_perito_${id}`),
                impTestigoPro: $(`#procurador_imp_testigo_${id}`),
                fechaCalculoPro: $(`#procurador_fecha_calculo_${id}`),
                tasasPro: $(`#procurador_imp_tasa_${id}`),
            },

            traslado: {
                fechaTraslado: $(`#fecha_traslado_${id}`),
                resultadoTasacion: $(`#resultado_tasacion_${id}`),
                importeLaj: $(`#importe_laj_${id}`),
                lajDif: $(`#laj_dif_${id}`),
                informeIca: $(`#informe_ica_${id}`),
                fechaIca: $(`#fecha_ica_${id}`),
                impugnacionTraslado: $(`#impugnacion_traslado_${id}`),
                tipoImpugnacion: $(`#tipo_impugnacion_${id}`),
                fechaImpugnacion: $(`#fecha_impugnacion_${id}`),
                impContrario: $(`#imp_contrario_${id}`),
                tipoImpugnacionContrario: $(`#tipo_impugnacion_contrario_traslado_${id}`),
                fechaImpugnacionContrario: $(`#fecha_impugnacion_contrario_traslado_${id}`),
                conformidadImpContrario: $(`#conformidad_imp_contrario_${id}`),



                resultadoTasacionPro: $(`#procurador_resultado_tasacion_${id}`),
                importeLajPro: $(`#procurador_importe_laj_${id}`),
                lajDifPro: $(`#procurador_laj_dif_${id}`),
                impugnacionTrasladoPro: $(`#procurador_impugnacion_traslado_${id}`),
                tipoImpugnacionPro: $(`#procurador_tipo_impugnacion_${id}`),
                fechaImpugnacionPro: $(`#procurador_fecha_impugnacion_${id}`),
                impContrarioPro: $(`#procurador_imp_contrario_${id}`),
                tipoImpugnacionContrarioPro: $(`#procurador_tipo_impugnacion_contrario_traslado_${id}`),
                fechaImpugnacionContrarioPro: $(`#procurador_fecha_impugnacion_contrario_traslado_${id}`),
                conformidadImpContrarioPro: $(`#procurador_conformidad_imp_contrario_${id}`),

            },

            decreto: {
                fechaDecreto: $(`#fecha_decreto_${id}`),

                resultadoDecreto: $(`#resultado_decreto_${id}`),
                importeDecreto: $(`#importe_decreto_${id}`),
                decretoDif: $(`#decree_dif_${id}`),
                recurso: $(`#recurso_${id}`),
                fechaRecurso: $(`#fecha_recurso_${id}`),
                impugnacionContrarioDecreto: $(`#impugnacion_contrario_decreto_${id}`),
                fechaImpugnacionContrarioDecreto: $(`#fecha_impugnacion_contrario_decreto_${id}`),
                recursoContrario: $(`#recurso_contrario_${id}`),
                fechaRecursoContrarioDecreto: $(`#fecha_recurso_contrario_decreto_${id}`),
                impugnacionRC: $(`#impugnacion_rc_${id}`),
                fechaImpugnacionRC: $(`#fecha_impugnacion_rc_${id}`),
                costasIncidenteDecreto: $(`#costas_incidente_decreto_${id}`),
                costasEcIncidenteDecreto: $(`#costas_ec_incidente_decreto_${id}`),
                alegacionesImpContrario: $(`#alegaciones_imp_contrario_${id}`),

                resultadoDecretoPro: $(`#procurador_resultado_decreto_${id}`),
                importeDecretoPro: $(`#procurador_importe_decreto_${id}`),
                decretoDifPro: $(`#procurador_decree_dif_${id}`),
                recursoPro: $(`#procurador_recurso_${id}`),
                fechaRecursoPro: $(`#procurador_fecha_recurso_${id}`),
                impugnacionContrarioDecretoPro: $(`#procurador_impugnacion_contrario_decreto_${id}`),
                fechaImpugnacionContrarioDecretoPro: $(`#procurador_fecha_impugnacion_contrario_decreto_${id}`),
                recursoContrarioPro: $(`#procurador_recurso_contrario_${id}`),
                fechaRecursoContrarioDecretoPro: $(`#procurador_fecha_recurso_contrario_decreto_${id}`),
                impugnacionRCPro: $(`#procurador_impugnacion_rc_${id}`),
                fechaImpugnacionRCPro: $(`#procurador_fecha_impugnacion_rc_${id}`),
                costasIncidenteDecretoPro: $(`#procurador_costas_incidente_decreto_${id}`),
                costasEcIncidenteDecretoPro: $(`#procurador_costas_ec_incidente_decreto_${id}`),
                alegacionesImpContrarioPro: $(`#procurador_alegaciones_imp_contrario_${id}`),
            },

            resolucion: {
                fechaResolucion: $(`#fecha_resolucion_${id}`),

                resultadoResolucion: $(`#resultado_Resolucion_${id}`),
                importeResolucion: $(`#importe_resolucion_${id}`),
                resolucionDif: $(`#ruling_dif_${id}`),
                costasIncidente: $(`#costas_incidente_${id}`),
                costasEcIncidente: $(`#costas_ec_incidente_${id}`),

                resultadoResolucionPro: $(`#procurador_resultado_Resolucion_${id}`),
                importeResolucionPro: $(`#procurador_importe_resolucion_${id}`),
                resolucionDifPro: $(`#procurador_ruling_dif_${id}`),
                costasIncidentePro: $(`#procurador_costas_incidente_${id}`),
                costasEcIncidentePro: $(`#procurador_costas_ec_incidente_${id}`)
            }
        };
    }


    function capturePhase(id) {
        const fields = getFieldsById(id);
        const faseSelector = $(`[id^="fase-select-${id}"]`);
        const tipoSelector = $(`#tipo-select-${id}`);
        const isPreselected = faseSelector.prop('defaultSelected');

        if (isPreselected) {
            return;
        }

        const defaultType = tipoSelector.find('option:selected').text().toLowerCase();
        const defaultFase = faseSelector.find('option:selected').text().toLowerCase();


        if (defaultType === 'a favor' && defaultFase === 'presentación') {

            $('#bloque-presentacion-' + id).show();


            updateFieldsBasedOnSelectedFase('presentación', id);
        }
        if (defaultType === 'en contra') {

            $('#bloque-presentacion-' + id).hide();

            faseSelector.find('option[value="presentacion"]').remove();
            updateFieldsBasedOnSelectedFase('traslado', id);
        }
        updateFieldsBasedOnSelectedFase(defaultFase, id);


        faseSelector.off('change');

        const initialSelectedFase = faseSelector.find('option:selected').text().toLowerCase();
        faseSelector.data('previousFase', initialSelectedFase);
        tipoSelector.on('click', function() {
            currentValue = $(this).val();
        });
        tipoSelector.on('change', function() {

            const selectedType = $(this).find('option:selected').text().toLowerCase();


            if (selectedType === 'a favor' || selectedType === 'en contra') {
                const confirmMessage = "Se eliminarán en caso de haberlos, los datos rellenados. ¿Deseas continuar?";
                const userConfirmed = confirm(confirmMessage);

                if (userConfirmed) {
                    clearAllFields(id, 'presentacion');
                    clearAllFields(id, 'traslado');
                    clearAllFields(id, 'decreto');
                    clearAllFields(id, 'resolucion');

                    if (selectedType === 'en contra') {
                        $('#bloque-presentacion-' + id).hide();
                        faseSelector.find('option[value="presentacion"]').remove();
                        faseSelector.val('traslado').data('previousFase', 'traslado');
                    } else {
                        $('#bloque-presentacion-' + id).show();

                        if (faseSelector.find('option[value="presentacion"]').length === 0) {
                            const newOption = '<option value="presentacion">PRESENTACIÓN</option>';
                            faseSelector.prepend(newOption);
                        }

                        faseSelector.val('presentacion').data('previousFase', 'presentacion');
                    }

                    updateFieldsBasedOnSelectedFase(faseSelector.find('option:selected').text().toLowerCase(), id);
                } else {
                    $(this).val(currentValue);

                }
            } else {

            }


            tipoSelector.data('previousType', selectedType);
        });

        const phaseSelector = $(`#fase-select-${id}`);
        const phasesOrder = ['presentación', 'traslado', 'decreto', 'resolución'];
        let currentPhase = phaseSelector.find('option:selected').text().toLowerCase();

        phaseSelector.on('change', function() {
            const selectedPhase = $(this).find('option:selected').text().toLowerCase();
            const currentPhaseIndex = phasesOrder.indexOf(currentPhase);
            const selectedPhaseIndex = phasesOrder.indexOf(selectedPhase);


            if (selectedPhaseIndex < currentPhaseIndex) {
                const confirmChange = confirm("Vas a retroceder de fase. Los datos de las fases posteriores se borrarán. ¿Deseas continuar?");
                if (!confirmChange) {
                    phaseSelector.val(currentPhase);
                    return;
                }
            }

            if (selectedPhaseIndex > currentPhaseIndex + 1) {
                alert("No puedes avanzar más de una fase a la vez.");


                if (currentPhase === 'presentación') {
                    phaseSelector.val('presentacion');
                } else {
                    phaseSelector.val(currentPhase);
                }

                return;
            }



            updateFieldsBasedOnSelectedFase(selectedPhase, id);


            currentPhase = selectedPhase;
        });
    }


    function clearAllFields(id, groupToClear) {
        const fields = getFieldsById(id);


        function clearFields(fieldGroup) {
            $.each(fieldGroup, function(key, field) {
                if (field.is(':checkbox')) {
                    field.prop('checked', false);
                } else if (field.is('select')) {
                    field.prop('selectedIndex', 0);
                } else {
                    field.val('');
                }
            });
        }


        if (groupToClear === 'presentacion') {
            clearFields(fields.presentacion);
        } else if (groupToClear === 'traslado') {
            clearFields(fields.traslado);
        } else if (groupToClear === 'decreto') {
            clearFields(fields.decreto);
        } else if (groupToClear === 'resolucion') {
            clearFields(fields.resolucion);
        }
    }










    function setDisabledCheckboxesTC(fields) {
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

    function updateFieldsBasedOnSelectedFase(selectedPhase, id) {

        const fields = getFieldsById(id);
        $.each(fields, function(phase, phaseFields) {
            $.each(phaseFields, function(key, field) {

                $(field).removeAttr('readonly').removeClass('readonly-select');
                $(field).off('click.prevent');
            });
        });
        if (selectedPhase === 'presentación') {

            clearAllFields(id, 'traslado');
            clearAllFields(id, 'decreto');
            clearAllFields(id, 'resolucion');
            $.each(fields.presentacion, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');
            });

            setDisabledCheckboxesTC(fields.traslado);
            setDisabledCheckboxesTC(fields.decreto);
            setDisabledCheckboxesTC(fields.resolucion);

        } else if (selectedPhase === 'traslado') {

            clearAllFields(id, 'decreto');
            clearAllFields(id, 'resolucion');
            $.each(fields.traslado, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');

            });

            setDisabledCheckboxesTC(fields.presentacion);
            setDisabledCheckboxesTC(fields.decreto);
            setDisabledCheckboxesTC(fields.resolucion);

            $('[id^="laj_dif_"]').attr('readonly', true);
            $('[id^="procurador_laj_dif_"]').attr('readonly', true);


        } else if (selectedPhase === 'decreto') {

            clearAllFields(id, 'resolucion');
            $.each(fields.decreto, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');
            });

            setDisabledCheckboxesTC(fields.presentacion);
            setDisabledCheckboxesTC(fields.traslado);
            setDisabledCheckboxesTC(fields.resolucion);

            $('[id^="decree_dif_"]').attr('readonly', true);
            $('[id^="procurador_decree_dif_"]').attr('readonly', true);

        } else if (selectedPhase === 'resolución') {
            $.each(fields.traslado, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');
            });

            setDisabledCheckboxesTC(fields.presentacion);
            setDisabledCheckboxesTC(fields.traslado);
            setDisabledCheckboxesTC(fields.decreto);

            $('[id^="ruling_dif_"]').attr('readonly', true);
            $('[id^="procurador_ruling_dif_"]').attr('readonly', true);

        }

    }




    document.addEventListener('click', function(event) {
        if (event.target.classList.contains('toggle-cost-block')) {
            const costBlockContent = event.target.nextElementSibling;
            if (costBlockContent.style.display === 'none' || costBlockContent.style.display === '') {
                costBlockContent.style.display = 'block';
                event.target.innerHTML = '&#9650;';
            } else {
                costBlockContent.style.display = 'none';
                event.target.innerHTML = '&#9660;';
            }
        }
    });
</script>
<?php
if (!empty($courtCosts)) :
?>
    <script>
        function sumarTrasladoDif(blockId) {

            var importeLaj = parseFloat($('#importe_laj_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#imp_costas_' + blockId).val()) || 0;
            var importePerito = parseFloat($('#imp_perito_' + blockId).val()) || 0;
            var importeTestigo = parseFloat($('#imp_testigo_' + blockId).val()) || 0;


            var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


            $('#laj_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }

        function sumarTrasladoDifProcurador(blockId) {

            var importeLaj = parseFloat($('#procurador_importe_laj_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#procurador_imp_costas_' + blockId).val()) || 0;
            var importeTasa = parseFloat($('#procurador_imp_tasa_' + blockId).val()) || 0;



            var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


            $('#procurador_laj_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }

        function sumarDecretoDif(blockId) {

            var importeLaj = parseFloat($('#importe_decreto_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#imp_costas_' + blockId).val()) || 0;
            var importePerito = parseFloat($('#imp_perito_' + blockId).val()) || 0;
            var importeTestigo = parseFloat($('#imp_testigo_' + blockId).val()) || 0;


            var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


            $('#decree_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }

        function sumarDecretoDifProcurador(blockId) {

            var importeLaj = parseFloat($('#procurador_importe_decreto_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#procurador_imp_costas_' + blockId).val()) || 0;
            var importeTasa = parseFloat($('#procurador_imp_tasa_' + blockId).val()) || 0;



            var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


            $('#procurador_decree_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }

        function sumarResolucionDif(blockId) {

            var importeLaj = parseFloat($('#importe_resolucion_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#imp_costas_' + blockId).val()) || 0;
            var importePerito = parseFloat($('#imp_perito_' + blockId).val()) || 0;
            var importeTestigo = parseFloat($('#imp_testigo_' + blockId).val()) || 0;


            var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


            $('#ruling_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }

        function sumarResolucionDifProcurador(blockId) {

            var importeLaj = parseFloat($('#procurador_importe_resolucion_' + blockId).val()) || 0;
            var importeCostas = parseFloat($('#procurador_imp_costas_' + blockId).val()) || 0;
            var importeTasa = parseFloat($('#procurador_imp_tasa_' + blockId).val()) || 0;



            var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


            $('#procurador_ruling_dif_' + blockId).val(diferenciaTotal.toFixed(2));
        }
        $(document).ready(function() {

            initializeDatepickers();
            let costCounter = 1;


            let blockIds = [];
            let blockId;


            <?php foreach ($courtCosts as $index => $cost): ?>
                blockId = <?= $index + 1; ?>;
                blockIds.push(blockId);
            <?php endforeach; ?>




            blockIds.forEach(function(blockId) {
                capturePhase(blockId);
                sumarTrasladoDif(blockId);
                sumarTrasladoDifProcurador(blockId);
                sumarDecretoDif(blockId);
                sumarDecretoDifProcurador(blockId);
                sumarResolucionDif(blockId);
                sumarResolucionDifProcurador(blockId);
            });

            //checkboxes fields associate reset

            $(document).on('change', '[id^="impugnacion_traslado_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#tipo_impugnacion_' + num).val("");
                    $('#fecha_impugnacion_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_impugnacion_traslado_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[3];


                if (!$(this).prop('checked')) {

                    $('#procurador_tipo_impugnacion_' + num).val("");
                    $('#procurador_fecha_impugnacion_' + num).val("");
                }
            });

            $(document).on('change', '[id^="informe_ica_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#fecha_ica_' + num).val("");
                }
            });

            $(document).on('change', '[id^="imp_contrario_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#tipo_impugnacion_contrario_traslado_' + num).val("");
                    $('#fecha_impugnacion_contrario_traslado_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_imp_contrario_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[3];


                if (!$(this).prop('checked')) {

                    $('#procurador_tipo_impugnacion_contrario_traslado_' + num).val("");
                    $('#procurador_fecha_impugnacion_contrario_traslado_' + num).val("");
                }
            });

            $(document).on('change', '[id^="recurso_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[1];


                if (!$(this).prop('checked')) {

                    $('#fecha_recurso_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_recurso_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#procurador_fecha_recurso_' + num).val("");
                }
            });

            $(document).on('change', '[id^="impugnacion_contrario_decreto_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[3];


                if (!$(this).prop('checked')) {

                    $('#fecha_impugnacion_contrario_decreto_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_impugnacion_contrario_decreto_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[4];


                if (!$(this).prop('checked')) {

                    $('#procurador_fecha_impugnacion_contrario_decreto_' + num).val("");
                }
            });

            $(document).on('change', '[id^="recurso_contrario_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#fecha_recurso_contrario_decreto_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_recurso_contrario_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[3];


                if (!$(this).prop('checked')) {

                    $('#procurador_fecha_recurso_contrario_decreto_' + num).val("");
                }
            });

            $(document).on('change', '[id^="impugnacion_rc_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[2];


                if (!$(this).prop('checked')) {

                    $('#fecha_impugnacion_rc_' + num).val("");
                }
            });

            $(document).on('change', '[id^="procurador_impugnacion_rc_"]', function() {

                var id = $(this).attr('id');
                var num = id.split('_')[3];


                if (!$(this).prop('checked')) {

                    $('#procurador_fecha_impugnacion_rc_' + num).val("");
                }
            });

            $(document).on('change input blur', '[id^="imp_costas_"], [id^="importe_laj_"], [id^="imp_perito_"], [id^="imp_testigo_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#importe_laj_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#imp_costas_' + num).val()) || 0;
                    var importePerito = parseFloat($('#imp_perito_' + num).val()) || 0;
                    var importeTestigo = parseFloat($('#imp_testigo_' + num).val()) || 0;


                    var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


                    $('#laj_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });

            $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_laj_"], [id^="imp_tasa_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#procurador_importe_laj_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                    var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                    var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


                    $('#procurador_laj_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });

            $(document).on('change input blur', '[id^="imp_costas_"], [id^="importe_decreto_"], [id^="imp_perito_"], [id^="imp_testigo_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#importe_decreto_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#imp_costas_' + num).val()) || 0;
                    var importePerito = parseFloat($('#imp_perito_' + num).val()) || 0;
                    var importeTestigo = parseFloat($('#imp_testigo_' + num).val()) || 0;


                    var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


                    $('#decree_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });


            $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_decreto_"], [id^="imp_tasa_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#procurador_importe_decreto_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                    var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                    var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


                    $('#procurador_decree_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });

            $(document).on('change input blur', '[id^="imp_costas_"], [id^="importe_resolucion_"], [id^="imp_perito_"], [id^="imp_testigo_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#importe_resolucion_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#imp_costas_' + num).val()) || 0;
                    var importePerito = parseFloat($('#imp_perito_' + num).val()) || 0;
                    var importeTestigo = parseFloat($('#imp_testigo_' + num).val()) || 0;


                    var diferenciaTotal = importeLaj - (importeCostas + importePerito + importeTestigo);


                    $('#ruling_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });

            $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_resolucion_"], [id^="imp_tasa_"]', function() {
                var id = $(this).attr('id');


                var num = id.match(/\d+$/);

                if (num) {
                    num = num[0];


                    var importeLaj = parseFloat($('#procurador_importe_resolucion_' + num).val()) || 0;

                    var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                    var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                    var diferenciaTotal = importeLaj - (importeCostas + importeTasa);


                    $('#procurador_ruling_dif_' + num).val(diferenciaTotal.toFixed(2));
                }
            });







        });
    </script>
<?php
endif;
?>





<?php
function generateType($context, $blockId, $isFirstDynamic, $cost = null)
{

    $defaultType = $isFirstDynamic ? 'a favor' : ($cost['stage'] ?? '');
    $defaultCostType = $isFirstDynamic ? '' : ($cost['costs_type'] ?? '');
    $defaultPhase = $isFirstDynamic ? 'presentacion' : ($cost['procedure_type'] ?? '');

    ob_start();
?>


<?php
    echo '<div class="col-sm-4">';
    echo $context->Form->input('tipo', [
        'label' => 'Tipo',
        'type' => 'select',
        'options' => jsonc('Records.type'),
        'empty' => '-- Seleccione una opción --',
        'value' => $defaultType,
        'id' => 'tipo-select-' . $blockId,
        'required' => true
    ]);
    echo '</div>';

    echo '<div class="col-sm-4">';
    echo $context->Form->input('tipo_procedimiento', [
        'label' => 'Tipo Procedimiento',
        'type' => 'select',
        'options' => jsonc('Records.costs_type'),
        'empty' => '-- Seleccione una opción --',
        'value' => $defaultCostType,
        'id' => 'tipo_procedimiento_' . $blockId,
        'required' => true
    ]);
    echo '</div>';

    echo '<div class="col-sm-4">';
    echo $context->Form->input('fase', [
        'label' => 'Fase',
        'type' => 'select',
        'options' => jsonc('Records.procedure_type'),
        'id' => 'fase-select-' . $blockId,
        'value' => $defaultPhase,
        'required' => true
    ]);
    echo '</div>';

    return ob_get_clean();
}


function presentation($context, $blockId, $cost = null, $arriaga = null, $procurador = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-presentacion-<?php echo $blockId; ?>" class="row">
        <div class="col-sm-2"><br>
            <h4>Presentación</h4>
        </div>
        <?php
        $presentationDate = isset($cost['presentation_date']) ? $cost['presentation_date']->format('d/m/Y') : '';
        //arriaga variables
        $impCosts = isset($arriaga['fee_amount']) ? $arriaga['fee_amount'] : null;
        $proficientAmount = isset($arriaga['proficient_amount']) ? $arriaga['proficient_amount'] : '';
        $witnessAmount = isset($arriaga['witness_amount']) ? $arriaga['witness_amount'] : '';
        $calculationDate = isset($arriaga['calculation_date']) ? $arriaga['calculation_date']->format('d/m/Y') : '';

        //procurador variables

        $impCostsPro = isset($procurador['fee_amount']) ? $procurador['fee_amount'] : null;
        $proficientAmountPro = isset($procurador['proficient_amount']) ? $procurador['proficient_amount'] : '';
        $witnessAmountPro = isset($procurador['witness_amount']) ? $procurador['witness_amount'] : '';
        $calculationDatePro = isset($procurador['calculation_date']) ? $procurador['calculation_date']->format('d/m/Y') : '';
        $TaxAmountPro = isset($procurador['tax_amount']) ? $procurador['tax_amount'] : '';



        ?>
        <div class="row">
            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input('fecha_presentacion', [
                'type' => 'text',
                'placeholder' => '__/__/____',
                'class' => 'datepicker',
                'label' => 'F. Presentación(CS)',
                'autocomplete' => 'off',
                'id' => 'fecha_presentacion_' . $blockId,
                'value' => $presentationDate,
                'required' => true
            ]);
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Presentacion arriaga asociados -->
                <li>
                    <a data-toggle="tab" href="#arriagaPresentation_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Presentacion Procurador-->
                <li>
                    <a data-toggle="tab" href="#procuradorPresentation_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
                </li>
            </ul>
        </div>

        <div class="tab-content">
            <div class="row">
                <!-- Contenido de la pestaña Arriaga Asociados Presentacion-->
                <div id="arriagaPresentation_<?php echo $blockId; ?>" class="tab-pane fade">
                    <div class="col-sm-2">

                    </div>
                    <?php
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('imp_costas', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de costas letrado',
                        'label' => 'Imp. Costas',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_costas_' . $blockId,
                        'value' => $impCosts,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('imp_perito', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de perito',
                        'label' => 'Imp. Perito',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_perito_' . $blockId,
                        'value' => $proficientAmount
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('imp_testigo', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'class' => 'form-control',
                        'placeholder' => 'Ingrese el importe de testigo',
                        'label' => 'Imp. Testigo',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_testigo_' . $blockId,
                        'value' => $witnessAmount
                    ]);
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('fecha_calculo', [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker form-control',
                        'label' => 'Fecha de Cálculo',
                        'autocomplete' => 'off',
                        'id' => 'fecha_calculo_' . $blockId,
                        'value' => $calculationDate,

                    ]);
                    echo '</div>';
                    ?>

                </div>

                <!-- Contenido de la pestaña Procurador Presentacion-->
                <div id="procuradorPresentation_<?php echo $blockId; ?>" class="tab-pane fade">
                    <div class="col-sm-2">
                    </div>
                    <?php
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_imp_costas', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de costas letrado',
                        'label' => 'Imp. Costas',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_costas_' . $blockId,
                        'value' => $impCostsPro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_imp_perito', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de perito',
                        'label' => 'Imp. Perito',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_perito_' . $blockId,
                        'value' => $proficientAmountPro
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_imp_testigo', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'class' => 'form-control',
                        'placeholder' => 'Ingrese el importe de testigo',
                        'label' => 'Imp. Testigo',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_testigo_' . $blockId,
                        'value' => $witnessAmountPro
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_fecha_calculo', [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker form-control',
                        'label' => 'Fecha de Cálculo',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_calculo_' . $blockId,
                        'value' => $calculationDatePro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_imp_tasa', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de tasa',
                        'label' => 'Imp. Tasa',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_tasa_' . $blockId,
                        'value' => $TaxAmountPro
                    ]);
                    echo '</div>';
                    ?>
                </div>
            </div>
        </div>
    </div>
<?php
    return ob_get_clean();
}




function transfer($context, $blockId, $cost = null, $arriaga = null, $procurador = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-traslado-<?php echo $blockId; ?>" class="row">
        <div class="col-sm-2"><br>
            <h4>Traslado</h4>
        </div>
        <?php

        $transferDate = isset($cost['transfer_date']) ? $cost['transfer_date']->format('d/m/Y') : '';

        //arraiga variables
        //first row
        $taxationResult  = isset($arriaga['taxation_result']) ? $arriaga['taxation_result'] : '';
        $lajAmount  = isset($arriaga['laj_amount']) ? $arriaga['laj_amount'] : '';
        $lajRequestedDiff  = isset($arriaga['laj_requested_difference_amount']) ? $arriaga['laj_requested_difference_amount'] : '';
        $icaReport  = (isset($arriaga['ica_report']) && $arriaga['ica_report'] == 1);
        $icaDate = isset($arriaga['ica_report_date']) ? $arriaga['ica_report_date']->format('d/m/Y') : '';
        //second row
        $ownChallenge = (isset($arriaga['own_challenge']) && $arriaga['own_challenge'] == 1) ? 1 : 0;
        $ownChallengeType = isset($arriaga['own_challenge_type']) ? $arriaga['own_challenge_type'] : '';
        $ownChallengeDate = isset($arriaga['own_challenge_date']) ? $arriaga['own_challenge_date']->format('d/m/Y') : '';
        //third row
        $opponnentChallenge  = (isset($arriaga['opponent_challenge']) && $arriaga['opponent_challenge'] == 1);
        $opponentChallengeType = isset($arriaga['opponent_challenge_type']) ? $arriaga['opponent_challenge_type'] : '';
        $opponentChallengeDate = isset($arriaga['opponent_challenge_date']) ? $arriaga['opponent_challenge_date']->format('d/m/Y') : '';
        //fourth
        $opponentChallengeAcceptance  = isset($arriaga['opponent_challenge_acceptance']) ? $arriaga['opponent_challenge_acceptance'] : '';


        //procurador variables
        //first row
        $taxationResultPro  = isset($procurador['taxation_result']) ? $procurador['taxation_result'] : '';
        $lajAmountPro  = isset($procurador['laj_amount']) ? $procurador['laj_amount'] : '';
        $lajRequestedDiffPro  = isset($procurador['laj_requested_difference_amount']) ? $procurador['laj_requested_difference_amount'] : '';
        //second row
        $ownChallengePro = (isset($procurador['own_challenge']) && $procurador['own_challenge'] == 1) ? 1 : 0;
        $ownChallengeTypePro = isset($procurador['own_challenge_type']) ? $procurador['own_challenge_type'] : '';
        $ownChallengeDatePro = isset($procurador['own_challenge_date']) ? $procurador['own_challenge_date']->format('d/m/Y') : '';
        //third row
        $opponnentChallengePro  = (isset($procurador['opponent_challenge']) && $procurador['opponent_challenge'] == 1);
        $opponentChallengeTypePro = isset($procurador['opponent_challenge_type']) ? $procurador['opponent_challenge_type'] : '';
        $opponentChallengeDatePro = isset($procurador['opponent_challenge_date']) ? $procurador['opponent_challenge_date']->format('d/m/Y') : '';
        //fourth
        $opponentChallengeAcceptancePro  = isset($procurador['opponent_challenge_acceptance']) ? $procurador['opponent_challenge_acceptance'] : '';







        echo '<div class="col-sm-2">';
        echo $context->Form->input('fecha_traslado', [
            'type' => 'text',
            'class' => 'form-control datepicker',
            'class' => 'datepicker',
            'placeholder' => '__/__/____',
            'label' => 'F. Traslado',
            'id' => 'fecha_traslado_' .  $blockId,
            'value' => $transferDate,
            'autocomplete' => 'off',
            'required' => true
        ]);
        echo '</div>';


        ?>
        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Transfer arriaga asociados -->
                <li>
                    <a data-toggle="tab" href="#arriagaTransfer_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Transfer Procurador-->
                <li>
                    <a data-toggle="tab" href="#procuradorTransfer_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
                </li>
            </ul>
        </div>




        <div class="tab-content">

            <!-- Contenido de la pestaña Arriaga Asociados Transfer-->
            <div id="arriagaTransfer_<?php echo $blockId; ?>" class="tab-pane fade">
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('resultado_tasacion', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_tasacion_' .  $blockId,
                        'value' => $taxationResult,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('importe_laj', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe LAJ',
                        'label' => 'Importe LAJ',
                        'id' => 'importe_laj_' .  $blockId,
                        'value' => $lajAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('laj_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'laj_dif_' .  $blockId,
                        'value' => $lajRequestedDiff,
                        'readonly',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('informe_ica', [
                        'type' => 'checkbox',
                        'label' => 'Informe ICA',
                        'id' => 'informe_ica_' .  $blockId,
                        'checked' => $icaReport,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('fecha_ica', [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'Fecha ICA',
                        'autocomplete' => 'off',
                        'value' => $icaDate,
                        'id' => 'fecha_ica_' .  $blockId
                    ]);
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('impugnacion_traslado', [
                        'type' => 'checkbox',
                        'label' => 'Impugnación',
                        'id' => 'impugnacion_traslado_' .  $blockId,
                        'checked' => $ownChallenge,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('tipo_impugnacion', [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo de Impugnación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tipo_impugnacion_' .  $blockId,
                        'value' => $ownChallengeType,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('fecha_impugnacion', [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'F. Impug. (CS)',
                        'autocomplete' => 'off',
                        'value' => $ownChallengeDate,
                        'id' => 'fecha_impugnacion_' .  $blockId
                    ]);
                    echo '</div>';
                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('imp_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Imp. Contrario',
                        'id' => 'imp_contrario_' .  $blockId,
                        'checked' => $opponnentChallenge,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('tipo_impugnacion_contrario', [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo Imp. Contrario',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tipo_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeType,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('fecha_impugnacion_contrario', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Imp. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'fecha_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeDate,
                    ]);
                    echo '</div>';



                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-10">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('conformidad_imp_contrario', [
                        'type' => 'select',
                        'options' => jsonc('Records.yesNo'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'conformidad_imp_contrario_' .  $blockId,
                        'value' => $opponentChallengeAcceptance,

                    ]);
                    echo '</div>';
                    ?>

                </div>
            </div>

            <!-- Contenido de la pestaña Procurador Transfer-->
            <div id="procuradorTransfer_<?php echo $blockId; ?>" class="tab-pane fade">
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_resultado_tasacion', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_tasacion_' .  $blockId,
                        'value' => $taxationResultPro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_importe_laj', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe LAJ',
                        'label' => 'Importe LAJ',
                        'id' => 'procurador_importe_laj_' .  $blockId,
                        'value' => $lajAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_laj_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_laj_dif_' .  $blockId,
                        'value' => $lajRequestedDiffPro,
                        'readonly',

                    ]);
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_impugnacion_traslado', [
                        'type' => 'checkbox',
                        'label' => 'Impugnación',
                        'id' => 'procurador_impugnacion_traslado_' .  $blockId,
                        'checked' => $ownChallengePro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_tipo_impugnacion', [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo de Impugnación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_tipo_impugnacion_' .  $blockId,
                        'value' => $ownChallengeTypePro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_fecha_impugnacion', [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'F. Impug. (CS)',
                        'autocomplete' => 'off',
                        'value' => $ownChallengeDatePro,
                        'id' => 'procurador_fecha_impugnacion_' .  $blockId
                    ]);
                    echo '</div>';
                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_imp_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Imp. Contrario',
                        'id' => 'procurador_imp_contrario_' .  $blockId,
                        'checked' => $opponnentChallengePro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_tipo_impugnacion_contrario', [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo Imp. Contrario',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_tipo_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeTypePro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_fecha_impugnacion_contrario', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Imp. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeDatePro,
                    ]);
                    echo '</div>';



                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-10">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_conformidad_imp_contrario', [
                        'type' => 'select',
                        'options' => jsonc('Records.yesNo'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_conformidad_imp_contrario_' .  $blockId,
                        'value' => $opponentChallengeAcceptancePro,

                    ]);
                    echo '</div>';
                    ?>

                </div>

            </div>


        </div>





    </div>




<?php
































    return ob_get_clean();
}
function decree($context, $blockId, $cost = null, $arriaga = null, $procurador = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-decreto-<?php echo $blockId; ?>" class="row">
        <div class="col-sm-2"><br>
            <h4>Decreto</h4>
        </div>
        <?php

        $decreeDate = isset($cost['decree_date']) ? $cost['decree_date']->format('d/m/Y') : '';

        //arriaga variables
        //first row
        $decreeResult  = isset($arriaga['decree_result']) ? $arriaga['decree_result'] : '';
        $decreeAmount  = isset($arriaga['decree_amount']) ? $arriaga['decree_amount'] : '';
        $difReqDecree  = isset($arriaga['decree_requested_difference_amount']) ? $arriaga['decree_requested_difference_amount'] : '';
        //second block
        $decreeOwnAppeal  = (isset($arriaga['decree_own_appeal']) && $arriaga['decree_own_appeal'] == 1);
        $decreeOwnAppealDate = isset($arriaga['decree_own_appeal_date']) ? $arriaga['decree_own_appeal_date']->format('d/m/Y') : '';
        $decreeOpponentChallengeToOwnAppeal  = (isset($arriaga['decree_opponent_challenge_to_own_appeal']) && $arriaga['decree_opponent_challenge_to_own_appeal'] == 1);
        $decreeOpponentChallengeToOwnAppealDate = isset($arriaga['decree_opponent_challenge_to_own_appeal_date']) ? $arriaga['decree_opponent_challenge_to_own_appeal_date']->format('d/m/Y') : '';
        //third block
        $decreeOpponentAppeal  = (isset($arriaga['decree_opponent_appeal']) && $arriaga['decree_opponent_appeal'] == 1);
        $decreeOpponentAppealDate = isset($arriaga['decree_opponent_appeal_date']) ? $arriaga['decree_opponent_appeal_date']->format('d/m/Y') : '';
        $decreeOwnChallengeToOpponentAppeal   = (isset($arriaga['decree_own_challenge_to_opponent_appeal']) && $arriaga['decree_own_challenge_to_opponent_appeal'] == 1);
        $decreeOwnChallengeToOpponentAppealDate = isset($arriaga['decree_own_challenge_to_opponent_appeal_date']) ? $arriaga['decree_own_challenge_to_opponent_appeal_date']->format('d/m/Y') : '';
        //fourth block
        $decreeFavorableCosts  = (isset($arriaga['decree_favorable_costs']) && $arriaga['decree_favorable_costs'] == 1);
        $DecreeAgainstCosts   = (isset($arriaga['decree_against_costs']) && $arriaga['decree_against_costs'] == 1);
        $ownAllegationsToOpponentChallenge  = (isset($arriaga['own_allegations_to_opponent_challenge']) && $arriaga['own_allegations_to_opponent_challenge'] == 1);


        //procurador variables
        //first row
        $decreeResultPro  = isset($procurador['decree_result']) ? $procurador['decree_result'] : '';
        $decreeAmountPro  = isset($procurador['decree_amount']) ? $procurador['decree_amount'] : '';
        $difReqDecreePro  = isset($procurador['decree_requested_difference_amount']) ? $procurador['decree_requested_difference_amount'] : '';
        //second block
        $decreeOwnAppealPro  = (isset($procurador['decree_own_appeal']) && $procurador['decree_own_appeal'] == 1);
        $decreeOwnAppealDatePro = isset($procurador['decree_own_appeal_date']) ? $procurador['decree_own_appeal_date']->format('d/m/Y') : '';
        $decreeOpponentChallengeToOwnAppealPro  = (isset($procurador['decree_opponent_challenge_to_own_appeal']) && $procurador['decree_opponent_challenge_to_own_appeal'] == 1);
        $decreeOpponentChallengeToOwnAppealDatePro = isset($procurador['decree_opponent_challenge_to_own_appeal_date']) ? $procurador['decree_opponent_challenge_to_own_appeal_date']->format('d/m/Y') : '';
        //third block
        $decreeOpponentAppealPro  = (isset($procurador['decree_opponent_appeal']) && $procurador['decree_opponent_appeal'] == 1);
        $decreeOpponentAppealDatePro = isset($procurador['decree_opponent_appeal_date']) ? $procurador['decree_opponent_appeal_date']->format('d/m/Y') : '';
        $decreeOwnChallengeToOpponentAppealPro   = (isset($procurador['decree_own_challenge_to_opponent_appeal']) && $procurador['decree_own_challenge_to_opponent_appeal'] == 1);
        $decreeOwnChallengeToOpponentAppealDatePro = isset($procurador['decree_own_challenge_to_opponent_appeal_date']) ? $procurador['decree_own_challenge_to_opponent_appeal_date']->format('d/m/Y') : '';
        //fourth block
        $decreeFavorableCostsPro  = (isset($procurador['decree_favorable_costs']) && $procurador['decree_favorable_costs'] == 1);
        $DecreeAgainstCostsPro   = (isset($procurador['decree_against_costs']) && $procurador['decree_against_costs'] == 1);
        $ownAllegationsToOpponentChallengePro  = (isset($procurador['own_allegations_to_opponent_challenge']) && $procurador['own_allegations_to_opponent_challenge'] == 1);
        ?>
        <div class="row">
            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input('fecha_decreto', [
                'type' => 'text',
                'class' => 'datepicker',
                'placeholder' => '__/__/____',
                'label' => 'Fecha Decreto',
                'id' => 'fecha_decreto_' .  $blockId,
                'value' => $decreeDate,
                'autocomplete' => 'off',
                'required' => true
            ]);
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Decree arriaga asociados -->
                <li>
                    <a data-toggle="tab" href="#arriagaDecree_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Decree Procurador-->
                <li>
                    <a data-toggle="tab" href="#procuradorDecree_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
                </li>
            </ul>
        </div>

        <div class="tab-content">

            <!-- Contenido de la pestaña Arriaga Asociados Decree-->
            <div id="arriagaDecree_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('resultado_decreto', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Decreto',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_decreto_' .  $blockId,
                        'value' => $decreeResult,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('importe_decreto', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese importe de decreto',
                        'label' => 'Importe Decreto',
                        'id' => 'importe_decreto_' .  $blockId,
                        'value' => $decreeAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('decree_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'decree_dif_' .  $blockId,
                        'value' => $difReqDecree,
                        'readonly',

                    ]);
                    echo '</div>';


                    ?>

                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('recurso', [
                        'type' => 'checkbox',
                        'label' => 'Recurso',
                        'id' => 'recurso_' .  $blockId,
                        'checked' => $decreeOwnAppeal,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('fecha_recurso', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso (CS)',
                        'autocomplete' => 'off',
                        'id' => 'fecha_recurso_' .  $blockId,
                        'value' => $decreeOwnAppealDate
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('impugnacion_contrario_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Impugnación Cont.',
                        'id' => 'impugnacion_contrario_decreto_' .  $blockId,
                        'checked' => $decreeOpponentChallengeToOwnAppeal,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('fecha_impugnacion_contrario_decreto', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'fecha_impugnacion_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentChallengeToOwnAppealDate,
                    ]);
                    echo '</div>';


                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('recurso_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Recurso Contrario',
                        'id' => 'recurso_contrario_' .  $blockId,
                        'checked' => $decreeOpponentAppeal,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('fecha_recurso_contrario_decreto', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso Contrario',
                        'id' => 'fecha_recurso_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentAppealDate,
                        'autocomplete' => 'off',
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('impugnacion_rc', [
                        'type' => 'checkbox',
                        'label' => 'impugnación rc',
                        'id' => 'impugnacion_rc_' .  $blockId,
                        'checked' => $decreeOwnChallengeToOpponentAppeal
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('fecha_impugnacion_rc', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug RC (CS)',
                        'id' => 'fecha_impugnacion_rc_' .  $blockId,
                        'value' => $decreeOwnChallengeToOpponentAppealDate,
                        'autocomplete' => 'off',
                    ]);
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('costas_incidente_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'costas_incidente_decreto_' .  $blockId,
                        'checked' => $decreeFavorableCosts,
                    ]);
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('costas_ec_incidente_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'costas_ec_incidente_decreto_' .  $blockId,
                        'checked' => $DecreeAgainstCosts,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-4"><br>';
                    echo $context->Form->input('alegaciones_imp_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Alegaciones Imp. Contrario',
                        'id' => 'alegaciones_imp_contrario_' .  $blockId,
                        'checked' => $ownAllegationsToOpponentChallenge,
                    ]);
                    echo '</div>';

                    ?>
                </div>
            </div>

            <!-- Contenido de la pestaña Procurador Transfer-->
            <div id="procuradorDecree_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_resultado_decreto', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Decreto',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_decreto_' .  $blockId,
                        'value' => $decreeResultPro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_importe_decreto', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese importe de decreto',
                        'label' => 'Importe Decreto',
                        'id' => 'procurador_importe_decreto_' .  $blockId,
                        'value' => $decreeAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_decree_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_decree_dif_' .  $blockId,
                        'value' => $difReqDecreePro,
                        'readonly',

                    ]);
                    echo '</div>';


                    ?>

                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_recurso', [
                        'type' => 'checkbox',
                        'label' => 'Recurso',
                        'id' => 'procurador_recurso_' .  $blockId,
                        'checked' => $decreeOwnAppealPro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_fecha_recurso', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso (CS)',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_recurso_' .  $blockId,
                        'value' => $decreeOwnAppealDatePro
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_impugnacion_contrario_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Impugnación Cont.',
                        'id' => 'procurador_impugnacion_contrario_decreto_' .  $blockId,
                        'checked' => $decreeOpponentChallengeToOwnAppealPro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_fecha_impugnacion_contrario_decreto', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_impugnacion_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentChallengeToOwnAppealDatePro,
                    ]);
                    echo '</div>';


                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_recurso_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Recurso Contrario',
                        'id' => 'procurador_recurso_contrario_' .  $blockId,
                        'checked' => $decreeOpponentAppealPro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_fecha_recurso_contrario_decreto', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso Contrario',
                        'id' => 'procurador_fecha_recurso_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentAppealDatePro,
                        'autocomplete' => 'off',
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_impugnacion_rc', [
                        'type' => 'checkbox',
                        'label' => 'impugnación rc',
                        'id' => 'procurador_impugnacion_rc_' .  $blockId,
                        'checked' => $decreeOwnChallengeToOpponentAppealPro
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_fecha_impugnacion_rc', [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug RC (CS)',
                        'id' => 'procurador_fecha_impugnacion_rc_' .  $blockId,
                        'value' => $decreeOwnChallengeToOpponentAppealDatePro,
                        'autocomplete' => 'off',
                    ]);
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_costas_incidente_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'procurador_costas_incidente_decreto_' .  $blockId,
                        'checked' => $decreeFavorableCostsPro,
                    ]);
                    echo '</div>';
                    echo '<div class="col-sm-2"><br>';
                    echo $context->Form->input('procurador_costas_ec_incidente_decreto', [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'procurador_costas_ec_incidente_decreto_' .  $blockId,
                        'checked' => $DecreeAgainstCostsPro,
                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-4"><br>';
                    echo $context->Form->input('procurador_alegaciones_imp_contrario', [
                        'type' => 'checkbox',
                        'label' => 'Alegaciones Imp. Contrario',
                        'id' => 'procurador_alegaciones_imp_contrario_' .  $blockId,
                        'checked' => $ownAllegationsToOpponentChallengePro,
                    ]);
                    echo '</div>';

                    ?>
                </div>

            </div>





        </div>
    </div>
<?php

    return ob_get_clean();
}

function rulling($context, $blockId, $cost = null, $arriaga = null, $procurador = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-resolucion-<?php echo $blockId; ?>" class="row">
        <div class="col-sm-2"><br>
            <h4>Resolución</h4>
        </div>
        <?php

        $rulingDate = isset($cost['ruling_date']) ? $cost['ruling_date']->format('d/m/Y') : '';

        //arriaga variables
        $rulingResult  = isset($arriaga['ruling_result']) ? $arriaga['ruling_result'] : '';
        $rulingAmount  = isset($arriaga['ruling_amount']) ? $arriaga['ruling_amount'] : '';
        $difReqResult  = isset($arriaga['ruling_requested_difference_amount']) ? $arriaga['ruling_requested_difference_amount'] : '';
        $rulingFavorableCosts  = (isset($arriaga['ruling_favorable_costs']) && $arriaga['ruling_favorable_costs'] == 1);
        $rulingAgainstCosts  = (isset($arriaga['ruling_against_costs']) && $arriaga['ruling_against_costs'] == 1);



        //procurador variables
        $rulingResultPro  = isset($procurador['ruling_result']) ? $procurador['ruling_result'] : '';
        $rulingAmountPro  = isset($procurador['ruling_amount']) ? $procurador['ruling_amount'] : '';
        $difReqResultPro  = isset($procurador['ruling_requested_difference_amount']) ? $arriaga['ruling_requested_difference_amount'] : '';
        $rulingFavorableCostsPro  = (isset($procurador['ruling_favorable_costs']) && $procurador['ruling_favorable_costs'] == 1);
        $rulingAgainstCostsPro = (isset($procurador['ruling_against_costs']) && $procurador['ruling_against_costs'] == 1);



        ?>
        <div class="row">
            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input('fecha_resolucion', [
                'type' => 'text',
                'class' => 'datepicker',
                'placeholder' => '__/__/____',
                'id' => 'fecha_resolucion_' .  $blockId,
                'value' => $rulingDate,
                'label' => 'Fecha Resolución',
                'autocomplete' => 'off',
                'required' => true
            ]);
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Ruling arriaga asociados -->
                <li>
                    <a data-toggle="tab" href="#arriagaRuling_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Ruling Procurador-->
                <li>
                    <a data-toggle="tab" href="#procuradorRuling_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
                </li>
            </ul>
        </div>



        <div class="tab-content">

            <!-- Contenido de la pestaña Arriaga Asociados Decree-->
            <div id="arriagaRuling_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('resultado_resolucion', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Resolución',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_Resolucion_' .  $blockId,
                        'value' => $rulingResult,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('importe_resolucion', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de resolución',
                        'label' => 'Imp. Resolución',
                        'id' => 'importe_resolucion_' .  $blockId,
                        'value' => $rulingAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('ruling_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'ruling_dif_' .  $blockId,
                        'value' => $difReqResult,
                        'readonly',

                    ]);
                    echo '</div>';



                    ?>

                </div>

                <div class="row">
                    <?php

                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('costas_incidente', [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'costas_incidente_' .  $blockId,
                        'checked' => $rulingFavorableCosts,
                    ]);
                    echo '</div>';
                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('costas_ec_incidente', [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'costas_ec_incidente_' .  $blockId,
                        'checked' => $rulingAgainstCosts,
                    ]);
                    echo '</div>';
                    ?>
                </div>
            </div>

            <!-- Contenido de la pestaña Procurador Transfer-->
            <div id="procuradorRuling_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_resultado_resolucion', [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Resolución',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_Resolucion_' .  $blockId,
                        'value' => $rulingResultPro,

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_importe_resolucion', [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de resolución',
                        'label' => 'Imp. Resolución',
                        'id' => 'procurador_importe_resolucion_' .  $blockId,
                        'value' => $rulingAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                    ]);
                    echo '</div>';

                    echo '<div class="col-sm-2">';
                    echo $context->Form->input('procurador_ruling_dif', [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_ruling_dif_' .  $blockId,
                        'value' => $difReqResultPro,
                        'readonly',

                    ]);
                    echo '</div>';



                    ?>

                </div>

                <div class="row">
                    <?php

                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_costas_incidente', [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'procurador_costas_incidente_' .  $blockId,
                        'checked' => $rulingFavorableCostsPro,
                    ]);
                    echo '</div>';
                    echo '<div class="col-sm-3">';
                    echo $context->Form->input('procurador_costas_ec_incidente', [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'procurador_costas_ec_incidente_' .  $blockId,
                        'checked' => $rulingAgainstCostsPro,
                    ]);
                    echo '</div>';
                    ?>
                </div>



            </div>





        </div>


    </div>


<?php

    return ob_get_clean();
}



?>
```



grabar en varias tablas php cake
```
if ($this->Income->save($Income)) {

                $incomeId = $Income->id;

  
  

                if (!empty($clientsData)) {

                    foreach ($clientsData as $clientData) {

                        $assignment = $this->CustomerAmountAssignment->newEntity(

                            [

                            'income_id' => $incomeId,

                            'client_id' => $clientData['client_id'],

                            'customer_amount_assignment' => $clientData['importe_cliente'],

                            'delivery_transfer_bank_account' => $clientData['pais_iban_cuenta'],

                            ]

                        );

  

                        if (!$this->CustomerAmountAssignment->save($assignment)) {

                            $this->Flash->error(__('No se pudo guardar la relación Income-Client para el cliente con ID: ' . $clientData['client_id']));

                        }

                    }

                }

  
  

                if (!empty($arriagaData)) {

                    foreach ($arriagaData as $arriaga) {

  

                        $assignmentAA = $this->AaAmountAssignment->newEntity(

                            [

                            'income_id' => $incomeId,

                            'client_id' => $arriaga['client_id'],

                            'total_amount' => $formData['total_imp_aa'] ? $formData['total_imp_aa'] : 0,

                            ]

                        );

  

                        if ($this->AaAmountAssignment->save($assignmentAA)) {

                            $assignmentId = $assignmentAA->id;

  
  

                            $assignmentconceptAA = $this->AaAmountAssignmentConcept->newEntity(

                                [

                                'aa_amount_assignment_id' => $assignmentId,

                                'assignable_concept_id' => $arriaga['concepto'],

                                'source' => $arriaga['origen'],

                                'procedure_name' => $arriaga['procedimiento'],

                                'amount' => $arriaga['amount_aa'],

                                'stipulations' => $arriaga['estipulaciones'],

                                ]

                            );

  

                            if (!$this->AaAmountAssignmentConcept->save($assignmentconceptAA)) {

                                $this->Flash->error(__('No se pudo guardar AaAmountAssignmentConcept para el cliente con ID: ' . $arriaga['client_id']));

                            }

                        } else {

                            $this->Flash->error(__('No se pudo guardar AaAmountAssignment para el cliente con ID: ' . $arriaga['client_id']));

                        }

                    }

                }

  
  

                $this->Flash->success(__('Los costos del tribunal han sido guardados.'));

                $destination = 'procedure-man';

                $this->redirect(['plugin' => 'Records', 'controller' => 'Manage', 'action' => 'edit', $record->get('id'), '#' => $destination]);

            } else {

                $this->Flash->error(__('No se pudo guardar los costos del tribunal. Por favor, intente de nuevo.'));

            }
```



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