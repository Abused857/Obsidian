
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

    ul.nav.nav-tabs {
        border: none !important;
    }

    a:focus,
    a:hover,
    a {
        color: black;
    }

    li.active a {
        color: black !important;
    }

    .ma {
        min-height: 120px;
        background-color: #FFE5DC !important;
        color: black !important;
    }

    .mp {
        min-height: 120px;
        background-color: #DCF6FF !important;
        color: black !important;
    }


    .py {
        padding: 3rem;
    }

    .pa {
        padding: 2rem;
    }

    .mx-24 {
        margin: 0px -24px;
    }

    .my-5 {
        margin: 3rem 0rem !important;
    }

    .mb {
        margin-bottom: 3rem !important;
    }

    .bg-blue {
        background-color: #DCF6FF !important;
        color: black !important;
    }

    .bg-orange {
        background-color: #FFE5DC !important;
        color: black !important;
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

                echo $this->Form->button(
                    '',
                    [
                    'class' => 'btn btn-primary mr-2 fa fa-lg fa-floppy-o',
                    'type' => 'submit',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'patchTC']),
                    ]
                );

                echo $this->Form->button(
                    '',
                    [
                    'class' => 'btn btn-danger fa fa-lg fa-trash-o',
                    'type' => 'submit',
                    'name' => 'action',
                    'value' => 'delete',
                    'formaction' => $this->Url->build(['controller' => 'Manage', 'action' => 'deleteTC', $cost->id]),
                    'onclick' => "return confirm('" . ($incomeExists ? "Se eliminarán todos los datos de tasación de costas, pero los documentos relacionados permanecerán en el expediente. ¿Desea continuar?" : "¿Está seguro de que desea eliminar esta tasación de costas?") . "');"
                    ]
                );
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

                const tipoValue = $('#tipo-select-' + id).val();


                if (tipoValue === 'en contra') {
                    $('#fase-select-' + id).val('traslado');
                } else {
                    $('#fase-select-' + id).val('presentacion');
                }

                alert('Se han restablecido los valores del bloque de Tasación de Costas: ' + costBlock.id);
            } else {
                alert('Los valores del bloque de  Tasación de Costas no han sido restablecidos.');
            }
        });


        initializeDatepickers();
        capturePhase(costCounter);
        checkAndDif();

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
        const phaseSelector = $(`#fase-select-${id}`);
        let currentPhase = phaseSelector.find('option:selected').text().toLowerCase();


        if (isPreselected) {
            return;
        }

        const defaultType = tipoSelector.find('option:selected').text().toLowerCase();
        const defaultFase = faseSelector.find('option:selected').text().toLowerCase();


        if (defaultType === 'a favor' && defaultFase === 'presentación') {

            $('#bloque-presentacion-' + id).show();

            currentPhase = 'presentación';
            updateFieldsBasedOnSelectedFase('presentación', id);

        }
        if (defaultType === 'en contra') {

            $('#bloque-presentacion-' + id).hide();

            faseSelector.find('option[value="presentacion"]').remove();
            currentPhase = 'traslado';

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
                        currentPhase = 'traslado';
                    } else {
                        $('#bloque-presentacion-' + id).show();

                        if (faseSelector.find('option[value="presentacion"]').length === 0) {
                            const newOption = '<option value="presentacion">PRESENTACIÓN</option>';
                            faseSelector.prepend(newOption);
                        }

                        faseSelector.val('presentacion').data('previousFase', 'presentacion');
                        currentPhase = 'presentación';
                    }

                    updateFieldsBasedOnSelectedFase(faseSelector.find('option:selected').text().toLowerCase(), id);
                } else {
                    $(this).val(currentValue);

                }
            } else {

            }


            tipoSelector.data('previousType', selectedType);
        });


        const phasesOrder = ['presentación', 'traslado', 'decreto', 'resolución'];


        phaseSelector.on('change', function() {

            const selectedPhase = $(this).find('option:selected').text().toLowerCase();
            const typeValue = $('#tipo-select-' + id).val();
            if (currentPhase == 'presentación' && typeValue == 'en contra') {
                currentPhase = 'traslado';
                console.log(currentPhase);
            }
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

            sumarTrasladoDif(id);
            sumarTrasladoDifProcurador(id);
            sumarDecretoDif(id);
            sumarDecretoDifProcurador(id);
            sumarResolucionDif(id);
            sumarResolucionDifProcurador(id);

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

            sumarTrasladoDif(id);
            sumarTrasladoDifProcurador(id);
            sumarDecretoDif(id);
            sumarDecretoDifProcurador(id);
            sumarResolucionDif(id);
            sumarResolucionDifProcurador(id);


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

            sumarTrasladoDif(id);
            sumarTrasladoDifProcurador(id);
            sumarDecretoDif(id);
            sumarDecretoDifProcurador(id);
            sumarResolucionDif(id);
            sumarResolucionDifProcurador(id);

        } else if (selectedPhase === 'resolución') {
            $.each(fields.traslado, function(key, field) {
                $(field).prop('readonly', false).removeClass('readonly-select');
            });

            setDisabledCheckboxesTC(fields.presentacion);
            setDisabledCheckboxesTC(fields.traslado);
            setDisabledCheckboxesTC(fields.decreto);

            $('[id^="ruling_dif_"]').attr('readonly', true);
            $('[id^="procurador_ruling_dif_"]').attr('readonly', true);

            sumarTrasladoDif(id);
            sumarTrasladoDifProcurador(id);
            sumarDecretoDif(id);
            sumarDecretoDifProcurador(id);
            sumarResolucionDif(id);
            sumarResolucionDifProcurador(id);

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

    function checkAndDif() {
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

                $('#laj_dif_' + num).val('');
                $('#laj_dif_' + num).val(diferenciaTotal.toFixed(2));
            }
        });

        $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_laj_"], [id^="procurador_imp_tasa_"]', function() {
            var id = $(this).attr('id');


            var num = id.match(/\d+$/);

            if (num) {
                num = num[0];


                var importeLaj = parseFloat($('#procurador_importe_laj_' + num).val()) || 0;

                var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                var diferenciaTotal = importeLaj - (importeCostas + importeTasa);

                $('#procurador_laj_dif_' + num).val('');
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

                $('#decree_dif_' + num).val('');
                $('#decree_dif_' + num).val(diferenciaTotal.toFixed(2));
            }
        });


        $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_decreto_"], [id^="procurador_imp_tasa_"]', function() {
            var id = $(this).attr('id');


            var num = id.match(/\d+$/);

            if (num) {
                num = num[0];


                var importeLaj = parseFloat($('#procurador_importe_decreto_' + num).val()) || 0;

                var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                var diferenciaTotal = importeLaj - (importeCostas + importeTasa);

                $('#procurador_decree_dif_' + num).val('');
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

                $('#ruling_dif_' + num).val('');
                $('#ruling_dif_' + num).val(diferenciaTotal.toFixed(2));
            }
        });

        $(document).on('change input blur', '[id^="procurador_imp_costas_"], [id^="procurador_importe_resolucion_"], [id^="procurador_imp_tasa_"]', function() {
            var id = $(this).attr('id');


            var num = id.match(/\d+$/);

            if (num) {
                num = num[0];


                var importeLaj = parseFloat($('#procurador_importe_resolucion_' + num).val()) || 0;

                var importeCostas = parseFloat($('#procurador_imp_costas_' + num).val()) || 0;
                var importeTasa = parseFloat($('#procurador_imp_tasa_' + num).val()) || 0;



                var diferenciaTotal = importeLaj - (importeCostas + importeTasa);

                $('#procurador_ruling_dif_' + num).val('');
                $('#procurador_ruling_dif_' + num).val(diferenciaTotal.toFixed(2));
            }
        });
    }
</script>
<?php
if (!empty($courtCosts)) :
?>
    <script>
        $(document).ready(function() {

            initializeDatepickers();
            let costCounter = 1;


            let blockIds = [];
            let blockId;


            <?php foreach ($courtCosts as $index => $cost) : ?>
                blockId = <?php echo $index + 1; ?>;
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

            checkAndDif();
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
    echo $context->Form->input(
        'tipo',
        [
        'label' => 'Tipo',
        'type' => 'select',
        'options' => jsonc('Records.type'),
        'empty' => '-- Seleccione una opción --',
        'value' => $defaultType,
        'id' => 'tipo-select-' . $blockId,
        'required' => true
        ]
    );
    echo '</div>';

    echo '<div class="col-sm-4">';
    echo $context->Form->input(
        'tipo_procedimiento',
        [
        'label' => 'Tipo Procedimiento',
        'type' => 'select',
        'options' => jsonc('Records.costs_type'),
        'empty' => '-- Seleccione una opción --',
        'value' => $defaultCostType,
        'id' => 'tipo_procedimiento_' . $blockId,
        'required' => true
        ]
    );
    echo '</div>';

    echo '<div class="col-sm-4">';
    echo $context->Form->input(
        'fase',
        [
        'label' => 'Fase',
        'type' => 'select',
        'options' => jsonc('Records.procedure_type'),
        'id' => 'fase-select-' . $blockId,
        'value' => $defaultPhase,
        'required' => true
        ]
    );
    echo '</div>';

    return ob_get_clean();
}


function presentation($context, $blockId, $cost = null)
{
    ob_start();


?>


    <hr>
    <div id="bloque-presentacion-<?php echo $blockId; ?>">

        <?php
        $presentationDate = isset($cost['presentation_date']) ? $cost['presentation_date']->format('d/m/Y') : '';
        //arriaga variables
        $impCosts = isset($cost->lawyer_court_costs[$blockId - 1]->fee_amount) ? $cost->lawyer_court_costs[$blockId - 1]->fee_amount : null;
        $proficientAmount = isset($cost->lawyer_court_costs[$blockId - 1]->proficient_amount) ? $cost->lawyer_court_costs[$blockId - 1]->proficient_amount : '';
        $witnessAmount = isset($cost->lawyer_court_costs[$blockId - 1]->witness_amount) ? $cost->lawyer_court_costs[$blockId - 1]->witness_amount : '';
        $calculationDate = isset($cost->lawyer_court_costs[$blockId - 1]->calculation_date) ? $cost->lawyer_court_costs[$blockId - 1]->calculation_date->format('d/m/Y') : '';

        //procurador variables

        $impCostsPro = isset($cost->attorney_court_costs[$blockId - 1]->fee_amount) ? $cost->attorney_court_costs[$blockId - 1]->fee_amount : null;
        $proficientAmountPro = isset($cost->attorney_court_costs[$blockId - 1]->proficient_amount) ? $cost->attorney_court_costs[$blockId - 1]->proficient_amount : '';
        $witnessAmountPro = isset($cost->attorney_court_costs[$blockId - 1]->witness_amount) ? $cost->attorney_court_costs[$blockId - 1]->witness_amount : '';
        $calculationDatePro = isset($cost->attorney_court_costs[$blockId - 1]->calculation_date) ? $cost->attorney_court_costs[$blockId - 1]->calculation_date->format('d/m/Y') : '';
        $TaxAmountPro = isset($cost->attorney_court_costs[$blockId - 1]->tax_amount) ? $cost->attorney_court_costs[$blockId - 1]->tax_amount : '';



        ?>
        <div class="row mb">
            <div class="col-sm-2"><br>
                <h4>Presentación</h4>
            </div>

            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input(
                'fecha_presentacion',
                [
                'type' => 'text',
                'placeholder' => '__/__/____',
                'class' => 'datepicker',
                'label' => 'F. Presentación(CS)',
                'autocomplete' => 'off',
                'id' => 'fecha_presentacion_' . $blockId,
                'value' => $presentationDate,
                'required' => true
                ]
            );
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Presentacion arriaga asociados -->
                <li>
                    <a class="bg-orange" data-toggle="tab" href="#arriagaPresentation_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Presentacion Procurador-->
                <li>
                    <a class="bg-blue" data-toggle="tab" href="#procuradorPresentation_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
                </li>
            </ul>
        </div>

        <div class="tab-content">

            <!-- Contenido de la pestaña Arriaga Asociados Presentacion-->
            <div id="arriagaPresentation_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <div class="col-sm-2">

                    </div>
                    <?php
                    echo '<div class="col-sm-2 ma py">';
                    echo $context->Form->input(
                        'imp_costas',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de costas letrado',
                        'label' => 'Imp. Costas',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_costas_' . $blockId,
                        'value' => $impCosts,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2  ma py">';
                    echo $context->Form->input(
                        'imp_perito',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de perito',
                        'label' => 'Imp. Perito',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_perito_' . $blockId,
                        'value' => $proficientAmount
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2  ma py">';
                    echo $context->Form->input(
                        'imp_testigo',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'class' => 'form-control',
                        'placeholder' => 'Ingrese el importe de testigo',
                        'label' => 'Imp. Testigo',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'imp_testigo_' . $blockId,
                        'value' => $witnessAmount
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-2  ma py">';
                    echo '</div>';
                    echo '<div class="col-sm-2  ma py">';
                    echo $context->Form->input(
                        'fecha_calculo',
                        [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker form-control',
                        'label' => 'Fecha de Cálculo',
                        'autocomplete' => 'off',
                        'id' => 'fecha_calculo_' . $blockId,
                        'value' => $calculationDate,

                        ]
                    );
                    echo '</div>';
                    ?>
                </div>


            </div>

            <!-- Contenido de la pestaña Procurador Presentacion-->
            <div id="procuradorPresentation_<?php echo $blockId; ?>" class="tab-pane fade">

                <div class="row">
                    <div class="col-sm-2">
                    </div>
                    <?php
                    echo '<div class="col-sm-2 mp py">';
                    echo $context->Form->input(
                        'procurador_imp_costas',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de costas letrado',
                        'label' => 'Imp. Costas',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_costas_' . $blockId,
                        'value' => $impCostsPro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp py">';
                    echo $context->Form->input(
                        'procurador_imp_perito',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de perito',
                        'label' => 'Imp. Perito',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_perito_' . $blockId,
                        'value' => $proficientAmountPro
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp py">';
                    echo $context->Form->input(
                        'procurador_imp_testigo',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'class' => 'form-control',
                        'placeholder' => 'Ingrese el importe de testigo',
                        'label' => 'Imp. Testigo',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_testigo_' . $blockId,
                        'value' => $witnessAmountPro
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp py">';
                    echo $context->Form->input(
                        'procurador_fecha_calculo',
                        [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker form-control',
                        'label' => 'Fecha de Cálculo',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_calculo_' . $blockId,
                        'value' => $calculationDatePro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp py">';
                    echo $context->Form->input(
                        'procurador_imp_tasa',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de tasa',
                        'label' => 'Imp. Tasa',
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',
                        'id' => 'procurador_imp_tasa_' . $blockId,
                        'value' => $TaxAmountPro
                        ]
                    );
                    echo '</div>';
                    ?>
                </div>

            </div>

        </div>
    </div>
<?php
    return ob_get_clean();
}




function transfer($context, $blockId, $cost = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-traslado-<?php echo $blockId; ?>">

        <?php

        $transferDate = isset($cost['transfer_date']) ? $cost['transfer_date']->format('d/m/Y') : '';

        //arraiga variables
        //first row
        $taxationResult  = isset($cost->lawyer_court_costs[$blockId - 1]->taxation_result) ? $cost->lawyer_court_costs[$blockId - 1]->taxation_result : '';
        $lajAmount  = isset($cost->lawyer_court_costs[$blockId - 1]->laj_amount) ? $cost->lawyer_court_costs[$blockId - 1]->laj_amount : '';
        $lajRequestedDiff  = isset($cost->lawyer_court_costs[$blockId - 1]->laj_requested_difference_amount) ? $cost->lawyer_court_costs[$blockId - 1]->laj_requested_difference_amount : '';
        $icaReport  = (isset($cost->lawyer_court_costs[$blockId - 1]->ica_report) && $cost->lawyer_court_costs[$blockId - 1]->ica_report == 1) ? 1 : 0;
        $icaDate = isset($cost->lawyer_court_costs[$blockId - 1]->ica_report_date) ? $cost->lawyer_court_costs[$blockId - 1]->ica_report_date->format('d/m/Y') : '';
        //second row
        $ownChallenge = (isset($cost->lawyer_court_costs[$blockId - 1]->own_challenge) && $cost->lawyer_court_costs[$blockId - 1]->own_challenge == 1) ? 1 : 0;
        $ownChallengeType = isset($cost->lawyer_court_costs[$blockId - 1]->own_challenge_type) ? $cost->lawyer_court_costs[$blockId - 1]->own_challenge_type : '';
        $ownChallengeDate = isset($cost->lawyer_court_costs[$blockId - 1]->own_challenge_date) ? $cost->lawyer_court_costs[$blockId - 1]->own_challenge_date->format('d/m/Y') : '';
        //third row
        $opponnentChallenge  = (isset($cost->lawyer_court_costs[$blockId - 1]->opponent_challenge) && $cost->lawyer_court_costs[$blockId - 1]->opponent_challenge == 1) ? 1 : 0;
        $opponentChallengeType = isset($cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_type) ? $cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_type : '';
        $opponentChallengeDate = isset($cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_date) ? $cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_date->format('d/m/Y') : '';
        //fourth
        $opponentChallengeAcceptance = isset($cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_acceptance)
            ? ($cost->lawyer_court_costs[$blockId - 1]->opponent_challenge_acceptance ? 'yes' : 'no')
            : '';


        //procurador variables
        //first row
        $taxationResultPro  = isset($cost->attorney_court_costs[$blockId - 1]->taxation_result) ? $cost->attorney_court_costs[$blockId - 1]->taxation_result : '';
        $lajAmountPro  = isset($cost->attorney_court_costs[$blockId - 1]->laj_amount) ? $cost->attorney_court_costs[$blockId - 1]->laj_amount : '';
        $lajRequestedDiffPro  = isset($cost->attorney_court_costs[$blockId - 1]->laj_requested_difference_amount) ? $cost->attorney_court_costs[$blockId - 1]->laj_requested_difference_amount : '';
        //second row
        $ownChallengePro = (isset($cost->attorney_court_costs[$blockId - 1]->own_challenge) && $cost->attorney_court_costs[$blockId - 1]->own_challenge == 1) ? 1 : 0;
        $ownChallengeTypePro = isset($cost->attorney_court_costs[$blockId - 1]->own_challenge_type) ? $cost->attorney_court_costs[$blockId - 1]->own_challenge_type : '';
        $ownChallengeDatePro = isset($cost->attorney_court_costs[$blockId - 1]->own_challenge_date) ? $cost->attorney_court_costs[$blockId - 1]->own_challenge_date->format('d/m/Y') : '';
        //third row
        $opponnentChallengePro  = (isset($cost->attorney_court_costs[$blockId - 1]->opponent_challenge) && $cost->attorney_court_costs[$blockId - 1]->opponent_challenge == 1);
        $opponentChallengeTypePro = isset($cost->attorney_court_costs[$blockId - 1]->opponent_challenge_type) ? $cost->attorney_court_costs[$blockId - 1]->opponent_challenge_type : '';
        $opponentChallengeDatePro = isset($cost->attorney_court_costs[$blockId - 1]->opponent_challenge_date) ? $cost->attorney_court_costs[$blockId - 1]->opponent_challenge_date->format('d/m/Y') : '';
        //fourth
        $opponentChallengeAcceptancePro  = isset($cost->attorney_court_costs[$blockId - 1]->opponent_challenge_acceptance) ? $cost->attorney_court_costs[$blockId - 1]->opponent_challenge_acceptance : '';






        ?>
        <div class="row mb">
            <div class="col-sm-2"><br>
                <h4>Traslado</h4>
            </div>

            <?php

            echo '<div class="col-sm-2">';
            echo $context->Form->input(
                'fecha_traslado',
                [
                'type' => 'text',
                'class' => 'form-control datepicker',
                'class' => 'datepicker',
                'placeholder' => '__/__/____',
                'label' => 'F. Traslado',
                'id' => 'fecha_traslado_' .  $blockId,
                'value' => $transferDate,
                'autocomplete' => 'off',
                'required' => true
                ]
            );
            echo '</div>';


            ?>
        </div>
        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Transfer arriaga asociados -->
                <li>
                    <a class="bg-orange" data-toggle="tab" href="#arriagaTransfer_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Transfer Procurador-->
                <li>
                    <a class="bg-blue" data-toggle="tab" href="#procuradorTransfer_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
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
                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'resultado_tasacion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_tasacion_' .  $blockId,
                        'value' => $taxationResult,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'importe_laj',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe LAJ',
                        'label' => 'Importe LAJ',
                        'id' => 'importe_laj_' .  $blockId,
                        'value' => $lajAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'laj_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'laj_dif_' .  $blockId,
                        'value' => $lajRequestedDiff,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa"><br>';
                    echo $context->Form->input(
                        'informe_ica',
                        [
                        'type' => 'checkbox',
                        'label' => 'Informe ICA',
                        'id' => 'informe_ica_' .  $blockId,
                        'checked' => $icaReport,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'fecha_ica',
                        [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'Fecha ICA',
                        'autocomplete' => 'off',
                        'value' => $icaDate,
                        'id' => 'fecha_ica_' .  $blockId
                        ]
                    );
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 ma pa"><br>';
                    echo $context->Form->input(
                        'impugnacion_traslado',
                        [
                        'type' => 'checkbox',
                        'label' => 'Impugnación',
                        'id' => 'impugnacion_traslado_' .  $blockId,
                        'checked' => $ownChallenge,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'tipo_impugnacion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo de Impugnación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tipo_impugnacion_' .  $blockId,
                        'value' => $ownChallengeType,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo '</div>';

                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'fecha_impugnacion',
                        [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'F. Impug. (CS)',
                        'autocomplete' => 'off',
                        'value' => $ownChallengeDate,
                        'id' => 'fecha_impugnacion_' .  $blockId
                        ]
                    );
                    echo '</div>';
                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 ma pa"><br>';
                    echo $context->Form->input(
                        'imp_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Imp. Contrario',
                        'id' => 'imp_contrario_' .  $blockId,
                        'checked' => $opponnentChallenge,
                        ]
                    );
                    echo '</div>';



                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'tipo_impugnacion_contrario',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo Imp. Contrario',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'tipo_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeType,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-2 ma pa">';
                    echo '</div>';
                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'fecha_impugnacion_contrario',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Imp. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'fecha_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeDate,
                        ]
                    );
                    echo '</div>';



                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-8 ma pa">';
                    echo '</div>';
                    echo '<div class="col-sm-2 ma">';
                    echo $context->Form->input(
                        'conformidad_imp_contrario',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.yesNo'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'conformidad_imp_contrario_' .  $blockId,
                        'value' => $opponentChallengeAcceptance,

                        ]
                    );
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
                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_resultado_tasacion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_tasacion_' .  $blockId,
                        'value' => $taxationResultPro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_importe_laj',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe LAJ',
                        'label' => 'Importe LAJ',
                        'id' => 'procurador_importe_laj_' .  $blockId,
                        'value' => $lajAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';


                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_laj_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_laj_dif_' .  $blockId,
                        'value' => $lajRequestedDiffPro,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 mp pa">';
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_impugnacion_traslado',
                        [
                        'type' => 'checkbox',
                        'label' => 'Impugnación',
                        'id' => 'procurador_impugnacion_traslado_' .  $blockId,
                        'checked' => $ownChallengePro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_tipo_impugnacion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo de Impugnación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_tipo_impugnacion_' .  $blockId,
                        'value' => $ownChallengeTypePro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_impugnacion',
                        [
                        'type' => 'text',
                        'placeholder' => '__/__/____',
                        'class' => 'datepicker',
                        'label' => 'F. Impug. (CS)',
                        'autocomplete' => 'off',
                        'value' => $ownChallengeDatePro,
                        'id' => 'procurador_fecha_impugnacion_' .  $blockId
                        ]
                    );
                    echo '</div>';
                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_imp_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Imp. Contrario',
                        'id' => 'procurador_imp_contrario_' .  $blockId,
                        'checked' => $opponnentChallengePro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_tipo_impugnacion_contrario',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.challenge_type'),
                        'label' => 'Tipo Imp. Contrario',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_tipo_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeTypePro,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-2 mp pa">';
                    echo '</div>';
                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_impugnacion_contrario',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Imp. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_impugnacion_contrario_traslado_' .  $blockId,
                        'value' => $opponentChallengeDatePro,
                        ]
                    );
                    echo '</div>';





                    ?>

                </div>

                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';

                    echo '<div class="col-sm-8 mp pa">';
                    echo '</div>';
                    echo '<div class="col-sm-2 mp">';
                    echo $context->Form->input(
                        'procurador_conformidad_imp_contrario',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.yesNo'),
                        'label' => 'Resultado Tasación',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_conformidad_imp_contrario_' .  $blockId,
                        'value' => $opponentChallengeAcceptancePro,

                        ]
                    );
                    echo '</div>';
                    ?>

                </div>

            </div>


        </div>





    </div>




<?php
































    return ob_get_clean();
}
function decree($context, $blockId, $cost = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-decreto-<?php echo $blockId; ?>">

        <?php

        $decreeDate = isset($cost['decree_date']) ? $cost['decree_date']->format('d/m/Y') : '';

        //arriaga variables
        //first row
        $decreeResult  = isset($cost->lawyer_court_costs[$blockId - 1]->decree_result) ? $cost->lawyer_court_costs[$blockId - 1]->decree_result : '';
        $decreeAmount  = isset($cost->lawyer_court_costs[$blockId - 1]->decree_amount) ? $cost->lawyer_court_costs[$blockId - 1]->decree_amount : '';
        $difReqDecree  = isset($cost->lawyer_court_costs[$blockId - 1]->decree_requested_difference_amount) ? $cost->lawyer_court_costs[$blockId - 1]->decree_requested_difference_amount : '';
        //second block
        $decreeOwnAppeal  = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_own_appeal) && $cost->lawyer_court_costs[$blockId - 1]->decree_own_appeal == 1);
        $decreeOwnAppealDate = isset($cost->lawyer_court_costs[$blockId - 1]->decree_own_appeal_date) ? $cost->lawyer_court_costs[$blockId - 1]->decree_own_appeal_date->format('d/m/Y') : '';
        $decreeOpponentChallengeToOwnAppeal  = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal) && $cost->lawyer_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal == 1);
        $decreeOpponentChallengeToOwnAppealDate = isset($cost->lawyer_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal_date) ? $cost->lawyer_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal_date->format('d/m/Y') : '';
        //third block
        $decreeOpponentAppeal  = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_opponent_appeal) && $cost->lawyer_court_costs[$blockId - 1]->decree_opponent_appeal == 1);
        $decreeOpponentAppealDate = isset($cost->lawyer_court_costs[$blockId - 1]->decree_opponent_appeal_date) ? $cost->lawyer_court_costs[$blockId - 1]->decree_opponent_appeal_date->format('d/m/Y') : '';
        $decreeOwnChallengeToOpponentAppeal   = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal) && $cost->lawyer_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal == 1);
        $decreeOwnChallengeToOpponentAppealDate = isset($cost->lawyer_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal_date) ? $cost->lawyer_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal_date->format('d/m/Y') : '';
        //fourth block
        $decreeFavorableCosts  = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_favorable_costs) && $cost->lawyer_court_costs[$blockId - 1]->decree_favorable_costs == 1);
        $DecreeAgainstCosts   = (isset($cost->lawyer_court_costs[$blockId - 1]->decree_against_costs) && $cost->lawyer_court_costs[$blockId - 1]->decree_against_costs == 1);
        $ownAllegationsToOpponentChallenge  = (isset($cost->lawyer_court_costs[$blockId - 1]->own_allegations_to_opponent_challenge) && $cost->lawyer_court_costs[$blockId - 1]->own_allegations_to_opponent_challenge == 1);


        //procurador variables
        //first row
        $decreeResultPro  = isset($cost->attorney_court_costs[$blockId - 1]->decree_result) ? $cost->attorney_court_costs[$blockId - 1]->decree_result : '';
        $decreeAmountPro  = isset($cost->attorney_court_costs[$blockId - 1]->decree_amount) ? $cost->attorney_court_costs[$blockId - 1]->decree_amount : '';
        $difReqDecreePro  = isset($cost->attorney_court_costs[$blockId - 1]->decree_requested_difference_amount) ? $cost->attorney_court_costs[$blockId - 1]->decree_requested_difference_amount : '';
        //second block
        $decreeOwnAppealPro  = (isset($cost->attorney_court_costs[$blockId - 1]->decree_own_appeal) && $cost->attorney_court_costs[$blockId - 1]->decree_own_appeal == 1);
        $decreeOwnAppealDatePro = isset($cost->attorney_court_costs[$blockId - 1]->decree_own_appeal_date) ? $cost->attorney_court_costs[$blockId - 1]->decree_own_appeal_date->format('d/m/Y') : '';
        $decreeOpponentChallengeToOwnAppealPro  = (isset($cost->attorney_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal) && $cost->attorney_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal == 1);
        $decreeOpponentChallengeToOwnAppealDatePro = isset($cost->attorney_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal_date) ? $cost->attorney_court_costs[$blockId - 1]->decree_opponent_challenge_to_own_appeal_date->format('d/m/Y') : '';
        //third block
        $decreeOpponentAppealPro  = (isset($cost->attorney_court_costs[$blockId - 1]->decree_opponent_appeal) && $cost->attorney_court_costs[$blockId - 1]->decree_opponent_appeal == 1);
        $decreeOpponentAppealDatePro = isset($cost->attorney_court_costs[$blockId - 1]->decree_opponent_appeal_date) ? $cost->attorney_court_costs[$blockId - 1]->decree_opponent_appeal_date->format('d/m/Y') : '';
        $decreeOwnChallengeToOpponentAppealPro   = (isset($cost->attorney_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal) && $cost->attorney_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal == 1);
        $decreeOwnChallengeToOpponentAppealDatePro = isset($cost->attorney_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal_date) ? $cost->attorney_court_costs[$blockId - 1]->decree_own_challenge_to_opponent_appeal_date->format('d/m/Y') : '';
        //fourth block
        $decreeFavorableCostsPro  = (isset($cost->attorney_court_costs[$blockId - 1]->decree_favorable_costs) && $cost->attorney_court_costs[$blockId - 1]->decree_favorable_costs == 1);
        $DecreeAgainstCostsPro   = (isset($cost->attorney_court_costs[$blockId - 1]->decree_against_costs) && $cost->attorney_court_costs[$blockId - 1]->decree_against_costs == 1);
        $ownAllegationsToOpponentChallengePro  = (isset($cost->attorney_court_costs[$blockId - 1]->own_allegations_to_opponent_challenge) && $cost->attorney_court_costs[$blockId - 1]->own_allegations_to_opponent_challenge == 1);
        ?>
        <div class="row mb">
            <div class="col-sm-2"><br>
                <h4>Decreto</h4>
            </div>
            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input(
                'fecha_decreto',
                [
                'type' => 'text',
                'class' => 'datepicker',
                'placeholder' => '__/__/____',
                'label' => 'Fecha Decreto',
                'id' => 'fecha_decreto_' .  $blockId,
                'value' => $decreeDate,
                'autocomplete' => 'off',
                'required' => true
                ]
            );
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Decree arriaga asociados -->
                <li>
                    <a class="bg-orange" data-toggle="tab" href="#arriagaDecree_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Decree Procurador-->
                <li>
                    <a class="bg-blue" data-toggle="tab" href="#procuradorDecree_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
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
                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'resultado_decreto',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Decreto',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_decreto_' .  $blockId,
                        'value' => $decreeResult,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'importe_decreto',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese importe de decreto',
                        'label' => 'Importe Decreto',
                        'id' => 'importe_decreto_' .  $blockId,
                        'value' => $decreeAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'decree_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'decree_dif_' .  $blockId,
                        'value' => $difReqDecree,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 ma pa">';
                    echo '</div>';


                    ?>

                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 ma pa"><br>';
                    echo $context->Form->input(
                        'recurso',
                        [
                        'type' => 'checkbox',
                        'label' => 'Recurso',
                        'id' => 'recurso_' .  $blockId,
                        'checked' => $decreeOwnAppeal,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'fecha_recurso',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso (CS)',
                        'autocomplete' => 'off',
                        'id' => 'fecha_recurso_' .  $blockId,
                        'value' => $decreeOwnAppealDate
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 ma pa"><br>';
                    echo $context->Form->input(
                        'impugnacion_contrario_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Impugnación Contrario',
                        'id' => 'impugnacion_contrario_decreto_' .  $blockId,
                        'checked' => $decreeOpponentChallengeToOwnAppeal,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'fecha_impugnacion_contrario_decreto',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'fecha_impugnacion_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentChallengeToOwnAppealDate,
                        ]
                    );
                    echo '</div>';




                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2  ma pa"><br>';
                    echo $context->Form->input(
                        'recurso_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Recurso Contrario',
                        'id' => 'recurso_contrario_' .  $blockId,
                        'checked' => $decreeOpponentAppeal,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2  ma pa">';
                    echo $context->Form->input(
                        'fecha_recurso_contrario_decreto',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso Contrario',
                        'id' => 'fecha_recurso_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentAppealDate,
                        'autocomplete' => 'off',
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3  ma pa"><br>';
                    echo $context->Form->input(
                        'impugnacion_rc',
                        [
                        'type' => 'checkbox',
                        'label' => 'impugnación rc',
                        'id' => 'impugnacion_rc_' .  $blockId,
                        'checked' => $decreeOwnChallengeToOpponentAppeal
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3  ma pa">';
                    echo $context->Form->input(
                        'fecha_impugnacion_rc',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug RC (CS)',
                        'id' => 'fecha_impugnacion_rc_' .  $blockId,
                        'value' => $decreeOwnChallengeToOpponentAppealDate,
                        'autocomplete' => 'off',
                        ]
                    );
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-1 ma pa">';
                    echo '</div>';
                    echo '<div class="col-sm-2  ma pa"><br>';
                    echo $context->Form->input(
                        'costas_incidente_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'costas_incidente_decreto_' .  $blockId,
                        'checked' => $decreeFavorableCosts,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-3  ma pa"><br>';
                    echo $context->Form->input(
                        'costas_ec_incidente_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'costas_ec_incidente_decreto_' .  $blockId,
                        'checked' => $DecreeAgainstCosts,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 ma pa"><br>';
                    echo $context->Form->input(
                        'alegaciones_imp_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Alegaciones Impugnación Contrario',
                        'id' => 'alegaciones_imp_contrario_' .  $blockId,
                        'checked' => $ownAllegationsToOpponentChallenge,
                        ]
                    );
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
                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_resultado_decreto',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Decreto',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_decreto_' .  $blockId,
                        'value' => $decreeResultPro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_importe_decreto',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese importe de decreto',
                        'label' => 'Importe Decreto',
                        'id' => 'procurador_importe_decreto_' .  $blockId,
                        'value' => $decreeAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_decree_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_decree_dif_' .  $blockId,
                        'value' => $difReqDecreePro,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 mp pa">';
                    echo '</div>';

                    ?>

                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_recurso',
                        [
                        'type' => 'checkbox',
                        'label' => 'Recurso',
                        'id' => 'procurador_recurso_' .  $blockId,
                        'checked' => $decreeOwnAppealPro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_recurso',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso (CS)',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_recurso_' .  $blockId,
                        'value' => $decreeOwnAppealDatePro
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_impugnacion_contrario_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Impugnación Contrario',
                        'id' => 'procurador_impugnacion_contrario_decreto_' .  $blockId,
                        'checked' => $decreeOpponentChallengeToOwnAppealPro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_impugnacion_contrario_decreto',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug. Contrario',
                        'autocomplete' => 'off',
                        'id' => 'procurador_fecha_impugnacion_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentChallengeToOwnAppealDatePro,
                        ]
                    );
                    echo '</div>';


                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-2 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_recurso_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Recurso Contrario',
                        'id' => 'procurador_recurso_contrario_' .  $blockId,
                        'checked' => $decreeOpponentAppealPro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_recurso_contrario_decreto',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Recurso Contrario',
                        'id' => 'procurador_fecha_recurso_contrario_decreto_' .  $blockId,
                        'value' => $decreeOpponentAppealDatePro,
                        'autocomplete' => 'off',
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_impugnacion_rc',
                        [
                        'type' => 'checkbox',
                        'label' => 'impugnación rc',
                        'id' => 'procurador_impugnacion_rc_' .  $blockId,
                        'checked' => $decreeOwnChallengeToOpponentAppealPro
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_fecha_impugnacion_rc',
                        [
                        'type' => 'text',
                        'class' => 'datepicker',
                        'placeholder' => '__/__/____',
                        'label' => 'F. Impug RC (CS)',
                        'id' => 'procurador_fecha_impugnacion_rc_' .  $blockId,
                        'value' => $decreeOwnChallengeToOpponentAppealDatePro,
                        'autocomplete' => 'off',
                        ]
                    );
                    echo '</div>';

                    ?>
                </div>
                <div class="row">
                    <?php
                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-1 mp pa">';
                    echo '</div>';
                    echo '<div class="col-sm-2  mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_costas_incidente_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'procurador_costas_incidente_decreto_' .  $blockId,
                        'checked' => $decreeFavorableCostsPro,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-3 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_costas_ec_incidente_decreto',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'procurador_costas_ec_incidente_decreto_' .  $blockId,
                        'checked' => $DecreeAgainstCostsPro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 mp pa"><br>';
                    echo $context->Form->input(
                        'procurador_alegaciones_imp_contrario',
                        [
                        'type' => 'checkbox',
                        'label' => 'Alegaciones Impugnación Contrario',
                        'id' => 'procurador_alegaciones_imp_contrario_' .  $blockId,
                        'checked' => $ownAllegationsToOpponentChallengePro,
                        ]
                    );
                    echo '</div>';

                    ?>
                </div>

            </div>





        </div>
    </div>
<?php

    return ob_get_clean();
}

function rulling($context, $blockId, $cost = null)
{
    ob_start();
?>

    <hr>
    <div id="bloque-resolucion-<?php echo $blockId; ?>">

        <?php

        $rulingDate = isset($cost['ruling_date']) ? $cost['ruling_date']->format('d/m/Y') : '';

        //arriaga variables
        $rulingResult  = isset($cost->lawyer_court_costs[$blockId - 1]->ruling_result) ? $cost->lawyer_court_costs[$blockId - 1]->ruling_result : '';
        $rulingAmount  = isset($cost->lawyer_court_costs[$blockId - 1]->ruling_amount) ? $cost->lawyer_court_costs[$blockId - 1]->ruling_amount : '';
        $difReqResult  = isset($cost->lawyer_court_costs[$blockId - 1]->ruling_requested_difference_amount) ? $cost->lawyer_court_costs[$blockId - 1]->ruling_requested_difference_amount : '';
        $rulingFavorableCosts  = (isset($cost->lawyer_court_costs[$blockId - 1]->ruling_favorable_costs) && $cost->lawyer_court_costs[$blockId - 1]->ruling_favorable_costs == 1);
        $rulingAgainstCosts  = (isset($cost->lawyer_court_costs[$blockId - 1]->ruling_against_costs) && $cost->lawyer_court_costs[$blockId - 1]->ruling_against_costs == 1);



        //procurador variables
        $rulingResultPro  = isset($cost->attorney_court_costs[$blockId - 1]->ruling_result) ? $cost->attorney_court_costs[$blockId - 1]->ruling_result : '';
        $rulingAmountPro  = isset($cost->attorney_court_costs[$blockId - 1]->ruling_amount) ? $cost->attorney_court_costs[$blockId - 1]->ruling_amount : '';
        $difReqResultPro  = isset($cost->attorney_court_costs[$blockId - 1]->ruling_requested_difference_amount) ? $cost->attorney_court_costs[$blockId - 1]->ruling_requested_difference_amount : '';
        $rulingFavorableCostsPro  = (isset($cost->attorney_court_costs[$blockId - 1]->ruling_favorable_costs) && $cost->attorney_court_costs[$blockId - 1]->ruling_favorable_costs == 1);
        $rulingAgainstCostsPro = (isset($cost->attorney_court_costs[$blockId - 1]->ruling_against_costs) && $cost->attorney_court_costs[$blockId - 1]->ruling_against_costs == 1);



        ?>
        <div class="row mb">
            <div class="col-sm-2"><br>
                <h4>Resolución</h4>
            </div>
            <?php
            echo '<div class="col-sm-2">';
            echo $context->Form->input(
                'fecha_resolucion',
                [
                'type' => 'text',
                'class' => 'datepicker',
                'placeholder' => '__/__/____',
                'id' => 'fecha_resolucion_' .  $blockId,
                'value' => $rulingDate,
                'label' => 'Fecha Resolución',
                'autocomplete' => 'off',
                'required' => true
                ]
            );
            echo '</div>';
            ?>
        </div>

        <div class="row">
            <div class="col-sm-2">
            </div>
            <ul class="nav nav-tabs">
                <!-- Pestaña 1: Ruling arriaga asociados -->
                <li>
                    <a class="bg-orange" data-toggle="tab" href="#arriagaRuling_<?php echo $blockId; ?>"><strong>Arriaga Asociados</strong></a>
                </li>
                <!-- Pestaña 2: Ruling Procurador-->
                <li>
                    <a class="bg-blue" data-toggle="tab" href="#procuradorRuling_<?php echo $blockId; ?>"><strong>Procurador</strong></a>
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

                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'resultado_resolucion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Resolución',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'resultado_Resolucion_' .  $blockId,
                        'value' => $rulingResult,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'importe_resolucion',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de resolución',
                        'label' => 'Imp. Resolución',
                        'id' => 'importe_resolucion_' .  $blockId,
                        'value' => $rulingAmount,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 ma pa">';
                    echo $context->Form->input(
                        'ruling_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'ruling_dif_' .  $blockId,
                        'value' => $difReqResult,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 ma pa">';
                    echo '</div>';

                    ?>

                </div>

                <div class="row">
                    <?php

                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'costas_incidente',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'costas_incidente_' .  $blockId,
                        'checked' => $rulingFavorableCosts,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-3 ma pa">';
                    echo $context->Form->input(
                        'costas_ec_incidente',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'costas_ec_incidente_' .  $blockId,
                        'checked' => $rulingAgainstCosts,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 ma pa">';
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

                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_resultado_resolucion',
                        [
                        'type' => 'select',
                        'options' => jsonc('Records.appraisal'),
                        'label' => 'Resultado Resolución',
                        'empty' => '-- Seleccione una opción --',
                        'id' => 'procurador_resultado_Resolucion_' .  $blockId,
                        'value' => $rulingResultPro,

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_importe_resolucion',
                        [
                        'type' => 'number',
                        'step' => '0.01',
                        'min' => 0,
                        'placeholder' => 'Ingrese el importe de resolución',
                        'label' => 'Imp. Resolución',
                        'id' => 'procurador_importe_resolucion_' .  $blockId,
                        'value' => $rulingAmountPro,
                        'title' => 'Ingrese un número decimal válido (hasta 2 decimales).',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-2 mp pa">';
                    echo $context->Form->input(
                        'procurador_ruling_dif',
                        [
                        'type' => 'number',
                        'label' => 'Dif. Total Solicitado',
                        'id' => 'procurador_ruling_dif_' .  $blockId,
                        'value' => $difReqResultPro,
                        'readonly',

                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-3 mp pa">';
                    echo '</div>';

                    ?>

                </div>

                <div class="row">
                    <?php

                    echo '<div class="col-sm-2">';
                    echo '</div>';
                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_costas_incidente',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas Incidente',
                        'id' => 'procurador_costas_incidente_' .  $blockId,
                        'checked' => $rulingFavorableCostsPro,
                        ]
                    );
                    echo '</div>';
                    echo '<div class="col-sm-3 mp pa">';
                    echo $context->Form->input(
                        'procurador_costas_ec_incidente',
                        [
                        'type' => 'checkbox',
                        'label' => 'Costas EC Incidente',
                        'id' => 'procurador_costas_ec_incidente_' .  $blockId,
                        'checked' => $rulingAgainstCostsPro,
                        ]
                    );
                    echo '</div>';

                    echo '<div class="col-sm-4 mp pa">';
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