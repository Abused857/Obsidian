procedure man ctp  comunicacion entre pestañas


```php
<div class="row">
    <div class="col-md-2">
        <?php echo $this->Form->input(
            'importe_a_ingresar', [
            'label' => 'Importe a Ingresar',
            'type' => 'number',
            'id' => 'importe_a_ingresar',
            'value' => !empty($record) && !empty($record->get('importe_a_ingresar')) ? $record->get('importe_a_ingresar') : 0,
            'readonly'
            ]
        ); ?>
    </div>
    <div class="col-md-2">
        <?php echo $this->Form->input(
            'costas_arr_cantidad1', [
            'label' => 'Importe Ingresado',
            'type' => 'number',
            'step' => 'any',
            'readonly'
            ]
        ); ?>
    </div>

    <div class="col-md-2">
        <?php echo $this->Form->input(
            'costas_arr_cantidad1', [
            'label' => 'Importe Pte. Ingreso',
            'type' => 'number',
            'step' => 'any',
            'readonly'
            ]
        ); ?>
    </div>

    <div class="col-md-2">
        <?php echo $this->Form->input(
            'costas_arr_cantidad1', [
            'label' => 'Importe Facturado',
            'type' => 'number',
            'step' => 'any',
            'readonly'
            ]
        ); ?>
    </div>

    <div class="col-md-2">
        <?php echo $this->Form->input(
            'costas_arr_cantidad1', [
            'label' => 'Importe Cobrado',
            'type' => 'number',
            'step' => 'any',
            'readonly'
            ]
        ); ?>
    </div>

    <div class="col-md-2">
        <?php echo $this->Form->input(
            'costas_arr_cantidad1', [
            'label' => 'Importe Pte. Cobro',
            'type' => 'number',
            'step' => 'any',
            'readonly'
            ]
        ); ?>
    </div>
</div>

<br>
<br>

<ul class="nav nav-tabs">
    <li class="active"><a data-toggle="tab" href="#costas"><strong>Tasación de Costas</strong></a></li>
    <li><a data-toggle="tab" href="#intereses"><strong>Liquidación de Intereses</strong></a></li>
    <li><a data-toggle="tab" href="#ejecucion"><strong>Demanda de Ejecución</strong></a></li>
    <li><a data-toggle="tab" href="#ingresos"><strong>Gestión de Ingresos</strong></a></li>
    <li><a data-toggle="tab" href="#operaciones"><strong>Operaciones Financieras</strong></a></li>
    <li><a data-toggle="tab" href="#historico"><strong>Datos Anteriores</strong></a></li>
</ul>

<div class="tab-content">
    <div id="costas" class="tab-pane fade in active">
        <?php echo $this->element('Records.record_edit/economics/tab_court_costs'); ?>
    </div>

    <div id="intereses" class="tab-pane fade">
        <?php echo $this->element('Records.record_edit/economics/tab_interests_settlement'); ?>
    </div>

    <div id="ejecucion" class="tab-pane fade">
        <div class="row">
            <div class="col-md-12">
                <div class="alert alert-warning">
                    <span><em>Desarrollo futuro</em></span>
                </div>
            </div>
        </div>
    </div>
    <div id="historico" class="tab-pane fade">
        <?php echo $this->element('Records.record_edit/economics/tab_historical'); ?>
    </div>
    <div id="ingresos" class="tab-pane fade">
        <br>
        <div class="row">
            <div id="spinner" style="display: flex;
                    justify-content: center;
                    align-items: center;
                    position: fixed;
                    padding: 10px;
                    background-color: rgba(0, 0, 0, 0.5);
                    color: white;
                    position: fixed;
                    top: 50%;
                    left: 50%;
                    transform: translate(-50%, -50%);
                    border-radius: 5px;
                    z-index: 9999;
                    display: none;">
                <i class="fa fa-spinner fa-spin"></i> Cargando...
            </div>
        </div>
        <div class="row">
            <div class="col-md-6"></div>
            <div class="col-md-6" style="text-align:right !important; margin-bottom: 15px;">
                <button type="button" class="btn btn-success add-man">
                    Añadir Fase de Mandamiento
                </button>
            </div>
        </div>
        <?php foreach ((array)$record->get('payment_orders') as $man_pago): ?>
            <?php $bills = (array)$man_pago->get('bills'); ?>
            <div class="panel panel-default">
                <div class="panel-heading">
                    <div class="row">
                        <div class="col-md-6">
                            <b>Procedimiento:</b> Fase mandamientos de pago (<b><?php echo $man_pago['ordering'] ?></b>) .Id MP: <b><?php echo $man_pago['id'] ?></b>
                        </div>
                        <div class="col-md-2">
                            <?php if (!empty($man_pago->get('pago_cantidad')) && $man_pago->get('pago_cantidad') <> 0) :
                                $sumaCobros = $man_pago->get('cobro_arriaga') + $man_pago->get('cobro_cliente') + $man_pago->get('cobro_procurador') + $man_pago->get('cobro_juzgado');
                                $saldo = $man_pago['pago_cantidad'] - $sumaCobros;
                                if ($saldo != 0) : ?>
                                    <b>Saldo: </b><b><?php echo number_format($saldo, 2, ',', '.'); ?></b><b>€</b>
                                <?php endif; ?>
                            <?php endif; ?>

                        </div>
                        <div class="col-md-4 text-right">
                            <span><b>ARRIAGA - </b></span>
                            <span><b>Nº Factura: </b></span>
                            <?php echo !empty($man_pago->get('fac_num_arr')) ? str_pad($man_pago->get('fac_num_arr'), 8, "0", STR_PAD_LEFT) : '---' ?> /
                            <span><b>Fecha Factura: </b></span>
                            <?php echo !empty($man_pago->get('fac_fecha_arr')) ? $man_pago->get('fac_fecha_arr')->format('d-m-Y') : '---' ?><br />
                            <span><b>PROCURADOR - </b></span>
                            <span><b>Nº Factura: </b></span>
                            <?php echo !empty($man_pago->get('fac_num_proc')) ? str_pad($man_pago->get('fac_num_proc'), 8, "0", STR_PAD_LEFT) : '---' ?> /
                            <span><b>Fecha Factura: </b></span>
                            <?php echo !empty($man_pago->get('fac_fecha_proc')) ? $man_pago->get('fac_fecha_proc')->format('d-m-Y') : '---' ?>
                        </div>
                    </div>
                </div>
                <div class="panel-body">
                    <div class="row">
                        <div class="col-md-5">
                            <p>
                                <span><b>Fecha consignación: </b></span>
                                <?php echo !empty($man_pago->get('cons_fecha')) ? $man_pago->get('cons_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <p>
                                <span><b>Consignación, importe: </b></span>
                                <?php echo !empty($man_pago->get('cons_cantidad')) ? $man_pago->get('cons_cantidad') . ' €' : '---' ?>
                            </p>
                            <p>
                                <span><b>Descripción: </b></span>
                                <?php echo !empty($man_pago->get('cons_notas')) ? $man_pago->get('cons_notas') : '---' ?>
                            </p>
                            <p>
                                <span><b>Concepto pago: </b></span>
                                <?php echo !empty($man_pago->get('cons_lugar')) ? $man_pago->get('cons_lugar') : '---' ?>
                            </p>
                            <p>
                                <span><b>Estimación honorarios Arriaga: </b></span>
                                <?php echo !empty($man_pago->get('est_honorarios_arriaga')) ? $man_pago->get('est_honorarios_arriaga') . ' €' : '---' ?>
                            </p>
                            <p>
                                <span><b>Fecha cobrado Arriaga: </b></span>
                                <?php echo !empty($man_pago->get('cobro_arriaga_fecha')) ? $man_pago->get('cobro_arriaga_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <div class="row">
                                <div class="col-md-8">
                                    <p>
                                        <span><b>Cobro Arriaga, importe: </b></span>
                                        <?php echo !empty($man_pago->get('cobro_arriaga')) ? $man_pago->get('cobro_arriaga') . ' €' : '---' ?>
                                    </p>
                                </div>

                                <div class="col-md-4">
                                    <?php
                                    if (!empty($man_pago->get('cobro_arriaga')) && $man_pago->get('cobro_arriaga') <> 0) :
                                        $facturaVieja = $record->files(['filter' => 'files/facturas/man-pago' . $man_pago['ordering'] . '/*factura_arriaga.pdf'])->first();

                                        if (!empty($facturaVieja)) : ?>
                                            <?php echo $this->Html->link(
                                                '',
                                                [
                                                    'controller' => 'Files',
                                                    'action' => 'download',
                                                    $record->get('id'),
                                                    'file' => $facturaVieja->get('path')
                                                ],
                                                [
                                                    'title' => 'Ver factura',
                                                    'class' => 'btn btn-success btn-file fa fa-file-text-o',
                                                    'target' => '_blank'
                                                ]
                                            ); ?>
                                        <?php else : ?>
                                            <?php echo
                                            $this->Html->link(
                                                '<i class="fa fa-info-circle"></i>', [], [
                                                'data-target' => '#cobro-arriaga-factura-help',
                                                'data-toggle' => 'collapse',
                                                // 'class' => 'center-block',
                                                'escape' => false,
                                                'onclick' => 'return false;'
                                                ]
                                            );
                                            ?>
                                            <em class="help-block collapse" id="cobro-arriaga-factura-help">
                                                <small>La facturación de este importe se realiza en la parte inferior de la página.</small>
                                            </em>
                                        <?php endif; ?>
                                    <?php endif; ?>
                                </div>
                            </div>
                            <p>
                                <span><b>Banco cobrado Arriaga: </b></span>
                                <?php $arrBancos = jsonc('Records.banco-cobro'); ?>
                                <?php echo !empty($man_pago->get('man_banco')) ? $arrBancos[$man_pago->get('man_banco')] : '---' ?>
                            </p>
                            <p>
                                <span><b>Paga a Arriaga: </b></span>
                                <?php $arrPagador = jsonc('Records.pagador'); ?>
                                <?php echo !empty($man_pago->get('pagador')) ? $arrPagador[$man_pago->get('pagador')] : '---' ?>
                            </p>
                            <p>
                                <span><b>Fecha cobrado procurador: </b></span>
                                <?php echo !empty($man_pago->get('cobro_procurador_fecha')) ? $man_pago->get('cobro_procurador_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <div class="row">
                                <div class="col-md-8">
                                    <p>
                                        <span><b>Cobro procurador, importe: </b></span>
                                        <?php echo !empty($man_pago->get('cobro_procurador')) ? $man_pago->get('cobro_procurador') . ' €' : '---' ?>
                                    </p>
                                </div>
                                <div class="col-md-4"></div>
                            </div>
                            <p id="tipoFactura">
                                <span><b>Tipo Factura: </b></span>
                                <?php echo !empty($man_pago->get('tipo')) ? $man_pago->get('tipo') : '---' ?>
                            </p>
                            <p>
                                <span><b>Fecha devolución juzgado: </b></span>
                                <?php echo !empty($man_pago->get('cobro_juzgado_fecha')) ? $man_pago->get('cobro_juzgado_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <p>
                                <span><b>Devolución juzgado, importe: </b></span>
                                <span class="bg-danger">
                                    <?php echo !empty($man_pago->get('cobro_juzgado')) ? $man_pago->get('cobro_juzgado') . ' €' : '---' ?>
                                </span>
                            </p>
                        </div>
                        <div class="col-md-5">
                            <p>
                                <span><b>Fecha mandamiento de pago: </b></span>
                                <?php echo !empty($man_pago->get('pago_fecha')) ? $man_pago->get('pago_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <p>
                                <span><b>Mandamiento de pago, importe: </b></span>
                                <?php echo !empty($man_pago->get('pago_cantidad')) ? $man_pago->get('pago_cantidad') . ' €' : '---' ?>
                            </p>
                            <p>
                                <span><b>Notas mandamiento de pago: </b></span>
                                <?php echo !empty($man_pago->get('pago_notas')) ? $man_pago->get('pago_notas') : '---' ?>
                            </p>
                            <p>
                                <span><b>Fecha cobrado cliente: </b></span>
                                <?php echo !empty($man_pago->get('cobro_cliente_fecha')) ? $man_pago->get('cobro_cliente_fecha')->format('d-m-Y') : '---' ?>
                            </p>
                            <div class="row">
                                <div class="col-md-8">
                                    <p>
                                        <span><b>Cobro cliente, importe: </b></span>
                                        <?php echo !empty($man_pago->get('cobro_cliente')) ? $man_pago->get('cobro_cliente') . ' €' : '---' ?>
                                    </p>
                                </div>
                                <div class="col-md-4">
                                    <?php
                                    if (!empty($man_pago->get('cobro_cliente'))) :
                                        $justifi = $record->files(['filter' => 'files/facturas/man-pago' . $man_pago['ordering'] . '/justificante_cliente.*'])->first();
                                        if (!empty($justifi)) : ?>
                                            <span
                                                class="file-input btn btn-success btn-file fa fa-exchange"
                                                title="Ver el justificante">
                                                <?php echo
                                                $this->Html->link(
                                                    '', [
                                                    'controller' => 'Files',
                                                    'action' => 'download',
                                                    $record->get('id'),
                                                    'file' => $justifi->get('path')
                                                    ], ['target' => '_blank']
                                                );
                                                ?>
                                            </span>
                                        <?php else : ?>
                                            <span class="file-input btn btn-danger btn-file fa fa-exchange" title="Subir justificante cliente">
                                                <?php echo
                                                $this->Html->link(
                                                    '<i class="fa fa-exchange"></i>', '#', [
                                                    'class' => 'simple-upload',
                                                    'data-record-id' => $record->get('id'),
                                                    'data-handler' => 'SubirMandamiento',
                                                    'data-extra' => base64_encode(Security::encrypt($man_pago['ordering'] . '::justificante_cliente', Security::salt())),
                                                    'escape' => false
                                                    ]
                                                );
                                                ?>
                                            </span>
                                        <?php endif;
                                    endif; ?>
                                </div>
                            </div>
                            <p>
                                <span><b>MP Gestionado: </b></span>
                                <?php echo $man_pago->get('gestionado') ? 'SI' : 'NO' ?>
                            </p>
                            <p>
                                <span><b>Ubicación del mandamiento: </b></span>
                                <?php echo !empty($man_pago->get('ubicacion')) ? $man_pago->get('ubicacion') : '---' ?>
                            </p>
                            <p>
                                <span><b>Ubic. actualizada: </b></span>
                                <?php echo !empty($man_pago->get('user_ubicacion')) ? $man_pago->get('user_ubicacion') : '---' ?>
                            </p>
                            <p id="conceptoFactura">
                                <span><b>Concepto factura: </b></span>
                                <?php echo !empty($man_pago->get('concepto')) ? $man_pago->get('concepto') : '---' ?>
                            </p>
                            <p>
                                <span><b>Número de orden: </b></span>
                                <?php echo !empty($man_pago->get('order_number')) ? $man_pago->get('order_number') : 'Q-' ?>
                            </p>
                            <?php
                            $justifi = $record->files(['filter' => 'files/facturas/man-pago' . $man_pago['ordering'] . '/doc_mandamiento.*'])->first();
                            if (!empty($justifi)) : ?>
                                <div
                                    class="docbuttons file2button"
                                    data-file-name="/files/facturas/man-pago<?php echo $man_pago['ordering'] ?>/.*doc_mandamiento.*|54af54|Ver mandamiento"
                                    data-record-id="<?php echo $record->get('id'); ?>"></div>
                            <?php else : ?>
                                <span class="file-input btn btn-primary btn-file fa fa-upload" title="Subir documentación"> Subir doc mandamiento
                                    <?php echo
                                    $this->Html->link(
                                        '<i class="fa fa-exchange"></i>', '#', [
                                        'class' => 'simple-upload',
                                        'data-record-id' => $record->get('id'),
                                        'data-handler' => 'SubirMandamiento',
                                        'data-extra' => base64_encode(Security::encrypt($man_pago['ordering'] . '::doc_mandamiento', Security::salt())),
                                        'escape' => false
                                        ]
                                    );
                                    ?>
                                </span>

                            <?php endif; ?>
                        </div>
                        <div class="col-md-2">
                            <div class="btn-group">
                                <?php
                                if (!$this->Form->viewMode() 
                                    && !empty($man_pago->get("cons_fecha")) 
                                    && empty($man_pago->get("pago_fecha"))
                                ) :
                                ?>
                                    <?php echo $this->Html->link(
                                        '', [
                                        'plugin' => 'Records',
                                        'controller' => 'Documents',
                                        'action' => 'create',
                                        $record->get('id'),
                                        'escritoJuzgado',
                                        'data' => base64_encode(Security::encrypt(serialize(['paymentOrderId' => $man_pago->get('id')]), Security::salt()))
                                        ], [
                                        'title' => 'Crear escrito juzgado',
                                        'class' => 'btn btn-danger doc-link fa fa-file-text'
                                        ]
                                    ); ?>
                                <?php endif; ?>
                                <button type="button"
                                    class="btn btn-default edit-man fa fa-edit"
                                    title="Editar mandamiento de pago"
                                    data-man-id=<?php echo $man_pago->get('id') ?>></button>
                            </div>
                            <button
                                type="button"
                                class="btn btn-success fa add-reminder"
                                title="Enviar notificación al procurador de mandamiento listo para transferencia"
                                data-fase-id=<?php echo $man_pago->get('ordering') ?>
                                data-record-id=<?php echo $record->get('id') ?>>
                                <i class="fa fa-bell-o"></i>
                            </button>
                        </div>
                    </div>
                    <!-- Facturas ya propuestas -->
                    <?php if (!empty($bills)) : ?>
                        <?php $importeTotalBills = 0; ?>
                        <?php foreach ($bills as $bill): ?>
                            <?php
                            $importeTotalBills += $bill->get('total');
                            ?>
                        <?php endforeach; ?>
                        <hr>
                        <label for="selectCliente">Facturas para Cobro Arriaga:</label>
                        <table>
                            <tr>
                                <td>
                                    <div class="cuadrado" style="background:#bdecb6; width: 10px; height: 10px; border: 1px solid #555;"> </div>
                                </td>
                                <td>
                                    &nbsp;<i><small>Factura confirmada</small></i>&nbsp;&nbsp;&nbsp;
                                </td>
                                <td>
                                    <div class="cuadrado" style="background:#b9935a; width: 10px; height: 10px; border: 1px solid #555;"> </div>
                                </td>
                                <td>
                                    &nbsp;<i><small>Factura sin confirmar</small></i>&nbsp;&nbsp;&nbsp;
                                </td>
                                <td>
                                    <div class="cuadrado" style="background:#ff9f9f; width: 10px; height: 10px; border: 1px solid #555;"> </div>
                                </td>
                                <td>
                                    &nbsp;<i><small>Factura rectificada</small></i>&nbsp;&nbsp;&nbsp;
                                </td>
                            </tr>
                        </table>
                        <div class="row">
                            <?php foreach ($bills as $bill): ?>
                                <?php $nombre = ''; ?>
                                <?php $fechaFactura = !empty($bill->get('fecha_factura')) ? true : false; ?>
                                <?php $rectificada = !empty($bill->get('rectificada')) ? true : false; ?>
                                <?php $importeOverCobro = ($man_pago->get('cobro_arriaga') < $importeTotalBills) ? true : false; ?>
                                <?php
                                if ($fechaFactura) {
                                    if ($rectificada || ($bill->get('fac_num') && ((strpos($bill->get('fac_num'), 'aAB-') !== false) || ($bill->get('tipo') && strpos($bill->get('tipo'), 'ABONO') !== false)))) {
                                        $colorVariable = '#ff9f9f';
                                    } else {
                                        $colorVariable = '#bdecb6';
                                    }
                                } else {
                                    $colorVariable = '#b9935a';
                                }
                                ?>
                                <?php $codigoFactura = !empty($bill->get('fac_num')) ? substr($bill->get('fac_num'), 0, 3) : ''; ?>
                                <?php $abono = ($codigoFactura == 'aAB') ? true : false ?>
                                <?php foreach ($clientes as $client): ?>
                                    <?php $cliente_factura = null; ?>
                                    <?php if ($client['id'] == $bill->get('client_id')) : ?>
                                        <?php
                                        $cliente_factura = $client;
                                        break;
                                        ?>
                                    <?php endif; ?>
                                <?php endforeach; ?>
                                <?php
                                if (empty($cliente_factura)) {
                                    $cliente_factura = TableRegistry::get('Clients.Clients')->get($bill->get('client_id'), ['fields' => ['name', 'lastname']]);
                                }
                                ?>
                                <?php $billLines = $bill->get('bill_lines'); ?>
                                <?php $factura = $record->files(['filter' => 'files/facturas/man-pago' . $man_pago['ordering'] . '/*factura_arriaga' . $bill->get('fac_num') . '.pdf'])->first(); ?>
                                <div class="col-md-12" style="background-color: <?php echo $colorVariable ?>; border-radius: 5px; padding: 10px; border: 1px solid #669; margin-top:5px; margin-left:5px; margin-right:5px;">
                                    <div class="col-md-8">
                                        <div class="col-md-5">
                                            <p>
                                                <span><b>Cliente: </b></span>
                                                <?php $nombre .= $client->get('empresa') ? $client->get('empresa-nombre-juridico') : $cliente_factura->get('name') . ' ' . $cliente_factura->get('lastname'); ?>
                                                <?php echo !empty($nombre) ? $nombre : '---' ?>
                                            </p>
                                            <p>
                                                <span><b>Tipo factura: </b></span>
                                                <?php if (!$abono) : ?>
                                                    <?php echo !empty($bill->get('tipo')) ? $bill->get('tipo') : '---' ?>
                                                <?php else : ?>
                                                    ABONO FACTURA
                                                <?php endif; ?>
                                            </p>
                                        </div>
                                        <?php if (!empty($factura)) : ?>
                                            <div class="col-md-5">
                                                <p>
                                                    <span><b>Nº factura: </b></span>
                                                    <?php echo $bill->get('fac_num') ?>
                                                </p>
                                                <p>
                                                    <span><b>Fecha factura: </b></span>
                                                    <?php echo $bill->get('fecha_factura')->format('d-m-Y') ?>
                                                </p>
                                            </div>
                                        <?php endif; ?>
                                    </div>
                                    <?php if (empty($factura) && user()->isAllowed('#facturacion-team')) : ?>
                                        <div class="col-md-4">
                                            <button type="button" class="btn btn-danger fa fa-trash-o remove-bill float-right" style="position: absolute; top: 0; right: 0;" title="Borrar propuesta de factura" data-bill-id=<?php echo $bill->get('id') ?>>&nbsp;&nbsp;Borrar propuesta</button>
                                        </div>
                                    <?php endif; ?>
                                    <div class="col-md-12 linea-horizontal" style="border-bottom: 1px solid #000000; margin: 20px 0;"></div>
                                    <div class="col-md-12" style="margin-bottom: 20px;">
                                        <table style="width:100%;">
                                            <colgroup>
                                                <col style="width: 10%;">
                                                <col style="width: 50%;">
                                                <col style="width: 10%;">
                                                <col style="width: 15%;">
                                                <col style="width: 15%;">
                                            </colgroup>
                                            <tr>
                                                <th></th>
                                                <th>Conceptos</th>
                                                <th>Impuesto</th>
                                                <th>Base impositiva</th>
                                                <th>Base imponible</th>
                                            </tr>
                                            <?php foreach ($billLines as $billLine): ?>
                                                <tr style="margin-bottom: 10px;">
                                                    <td>
                                                        <?php if (!$fechaFactura && user()->isAllowed('#facturacion-team')) : ?>
                                                            <button type="button"
                                                                class="btn btn-sm edit-line fa fa-edit"
                                                                title="Editar concepto"
                                                                style="background-color: transparent; border: none;"
                                                                data-bill-id=<?php echo $bill->get('id') ?>;
                                                                data-bill-line-id=<?php echo $billLine->get('id') ?>></button>

                                                            <button type="button"
                                                                class="btn btn-sm remove-line fa fa-trash-o"
                                                                title="Eliminar concepto"
                                                                style="background-color: transparent; border: none;"
                                                                data-bill-line-id=<?php echo $billLine->get('id') ?>></button>
                                                        <?php endif; ?>
                                                    </td>
                                                    <td style="padding: 10px;">
                                                        <?php echo !empty($billLine->get('concepto')) ? $billLine->get('concepto') : '---' ?>
                                                    </td>
                                                    <td>
                                                        <?php echo !empty($billLine->get('impuesto')) ? $billLine->get('impuesto') : '---' ?>
                                                    </td>
                                                    <td>
                                                        <?php echo !empty($billLine->get('base_impositiva')) ? Number::currency($billLine->get('base_impositiva')) : '---' ?>
                                                    </td>
                                                    <td>
                                                        <?php echo !empty($billLine->get('base_imponible')) ? Number::currency($billLine->get('base_imponible')) : '---' ?>
                                                    </td>
                                                </tr>
                                            <?php endforeach; ?>
                                        </table>
                                    </div>
                                    <br>
                                    <div class="col-md-12 linea-horizontal" style="border-bottom: 1px solid #000000; margin: 20px 0;"></div>
                                    <div class="col-md-12">
                                        <div class="col-md-9 text-left">
                                            <?php if (!$fechaFactura && user()->isAllowed('#facturacion-team')) : ?>
                                                <p>
                                                    <a href="#procedure-man" id="" class="btn btn-primary btn-file fa fa-plus-circle add-lines" data-bill-id="<?php echo $bill->get('id'); ?>">&nbsp;Añadir nuevo concepto</a>
                                                </p>
                                                <?php //Si no hay fecha de cobro o ha pasado menos de un mes desde que se haya puesto 
                                                ?>
                                                <?php if (empty($man_pago->get('cobro_arriaga_fecha')) || (strtotime('+ 1 month', $man_pago->get('cobro_arriaga_fecha')->toUnixString()) >= strtotime(date('Y-m-d')))) : ?>

                                                    <?php
                                                    $classExist = 'btn-danger';
                                                    $title = 'Crear factura PROFORMA';
                                                    $ruta = [
                                                        'plugin' => 'Records',
                                                        'controller' => 'Documents',
                                                        'action' => 'create',
                                                        $record->get('id'),
                                                        'facturaMandamientoProforma',
                                                        'data' => base64_encode(Security::encrypt(serialize(['paymentOrderId' => $man_pago->get('id'), 'invoiceType' => 'arriaga']), Security::salt()))
                                                    ];

                                                    if (!empty($docsPayment[$man_pago->get('id')]) && !empty($docsPayment[$man_pago->get('id')]['facturaMandamientoProforma'])) {
                                                        $classExist = 'btn-success';
                                                        $title = 'Editar factura PROFORMA';
                                                        $facturaProformaId = $docsPayment[$man_pago->get('id')]['facturaMandamientoProforma'][0];

                                                        $ruta = [
                                                            'plugin' => 'Records',
                                                            'controller' => 'Documents',
                                                            'action' => 'edit',
                                                            $facturaProformaId
                                                        ];
                                                    }
                                                    ?>

                                                    <?php echo $this->Html->link(
                                                        '', $ruta, [
                                                        'title' => $title,
                                                        'class' => 'btn ' . $classExist . ' doc-link fa fa-file-powerpoint-o'
                                                        ]
                                                    ); ?>
                                                    <?php if (!$importeOverCobro && user()->isAllowed('#facturacion-team')) : ?>
                                                        <?php echo $this->Html->link(
                                                            '', [
                                                            'plugin' => 'Records',
                                                            'controller' => 'Documents',
                                                            'action' => 'create',
                                                            $record->get('id'),
                                                            'facturaMandamiento',
                                                            'data' => base64_encode(Security::encrypt(serialize(['paymentOrderId' => $man_pago->get('id'), 'billId' => $bill->get('id'), 'abono' => false, 'invoiceType' => 'arriaga']), Security::salt()))
                                                            ], [
                                                            'title' => 'Crear factura',
                                                            'class' => 'btn btn-danger doc-link fa fa-file-text-o'
                                                            ]
                                                        ); ?>
                                                    <?php endif; ?>
                                                <?php endif; ?>
                                                <?php $justifi = $record->files(['filter' => 'files/facturas/man-pago' . $man_pago['ordering'] . '/justificante_arriaga*'])->first();

                                                if (!empty($justifi)) : ?>
                                                    <span
                                                        class="file-input btn btn-success btn-file fa fa-exchange"
                                                        title="Ver el justificante">
                                                        <?php echo
                                                        $this->Html->link(
                                                            '', [
                                                            'controller' => 'Files',
                                                            'action' => 'download',
                                                            $record->get('id'),
                                                            'file' => $justifi->get('path')
                                                            ], ['target' => '_blank']
                                                        );
                                                        ?>
                                                    </span>
                                                <?php else : ?>
                                                    <span class="file-input btn btn-danger btn-file fa fa-exchange" title="Subir justificante arriaga">
                                                        <?php echo
                                                        $this->Html->link(
                                                            '<i class="fa fa-exchange"></i>', '#', [
                                                            'class' => 'simple-upload',
                                                            'data-record-id' => $record->get('id'),
                                                            'data-handler' => 'SubirMandamiento',
                                                            'data-extra' => base64_encode(Security::encrypt($man_pago['ordering'] . '::justificante_arriaga', Security::salt())),
                                                            'escape' => false
                                                            ]
                                                        );
                                                        ?>
                                                    </span>
                                                <?php endif; ?>
                                            <?php else : ?>
                                                <?php if (!empty($bill->get('total')) && $bill->get('total') <> 0) : ?>

                                                    <?php if (!empty($factura)) : ?>
                                                        <?php echo $this->Html->link(
                                                            '',
                                                            [
                                                                'controller' => 'Files',
                                                                'action' => 'download',
                                                                $record->get('id'),
                                                                'file' => $factura->get('path')
                                                            ],
                                                            [
                                                                'title' => 'Ver factura',
                                                                'class' => 'btn btn-success btn-file fa fa-file-text-o',
                                                                'target' => '_blank'
                                                            ]
                                                        ); ?>
                                                        <!-- Aquí va la FACTURA RECTIFICADA -->
                                                        &nbsp;&nbsp;
                                                        <?php  ?>
                                                        <?php if (!$abono && !$rectificada && user()->isAllowed('#facturacion-team')) : ?>
                                                            <?php echo
                                                            $this->Html->link(
                                                                '', [
                                                                'plugin' => 'Records',
                                                                'controller' => 'Documents',
                                                                'action' => 'create',
                                                                $record->get('id'),
                                                                'facturaMandamiento',
                                                                'data' => base64_encode(Security::encrypt(serialize(['paymentOrderId' => $man_pago->get('id'), 'billId' => $bill->get('id'), 'abono' => true, 'invoiceType' => 'arriaga']), Security::salt()))
                                                                ], [
                                                                'title' => 'Crear factura rectificativa',
                                                                'class' => 'btn btn-danger doc-link fa fa-recycle'
                                                                ]
                                                            );
                                                            ?>
                                                        <?php endif; ?>
                                                        <!-- Aquí va la FACTURA RECTIFICADA DEL ABONO-->
                                                        &nbsp;&nbsp;
                                                        <?php  ?>
                                                        <?php if ($abono && !$rectificada && user()->isAllowed('#facturacion-team')) : ?>
                                                            <?php echo
                                                            $this->Html->link(
                                                                '', [
                                                                'plugin' => 'Records',
                                                                'controller' => 'Documents',
                                                                'action' => 'create',
                                                                $record->get('id'),
                                                                'facturaMandamiento',
                                                                'data' => base64_encode(Security::encrypt(serialize(['paymentOrderId' => $man_pago->get('id'), 'billId' => $bill->get('id'), 'abono' => true, 'invoiceType' => 'arriaga']), Security::salt()))
                                                                ], [
                                                                'title' => 'Crear factura rectificativa',
                                                                'class' => 'btn btn-danger doc-link fa fa-recycle'
                                                                ]
                                                            );
                                                            ?>
                                                        <?php endif; ?>
                                                    <?php endif; ?>
                                                <?php endif; ?>
                                            <?php endif; ?>
                                        </div>
                                        <div class="col-md-3 text-right">
                                            <p><span><b>TOTALES: </b></span></p>
                                            <p>
                                                <span><b>Base imponible: </b></span>
                                                <?php echo !empty($bill->get('base_imponible')) ? Number::currency($bill->get('base_imponible')) : '---' ?>
                                            </p>
                                            <p>
                                                <span><b>Base impositiva: </b></span>
                                                <?php echo !empty($bill->get('base_impositiva')) ? Number::currency($bill->get('base_impositiva')) : '---' ?>
                                            </p>
                                            <?php if ($importeOverCobro && !$fechaFactura) : ?>
                                                <p style="color: red;">
                                                    <?php echo
                                                    $this->Html->link(
                                                        '<i class="fa fa-info-circle"></i>', [], [
                                                        'data-target' => '#factura-help',
                                                        'data-toggle' => 'collapse',
                                                        // 'class' => 'center-block',
                                                        'escape' => false,
                                                        'onclick' => 'return false;'
                                                        ]
                                                    );
                                                    ?>
                                                    <em class="help-block collapse" id="factura-help">
                                                        <small>El importe de esta factura es superior a "Cobro Arriaga", revísela para poder generar la factura.<br>Hasta entonces no aparecerá la opción de "Crear Factura"</small>
                                                    </em>
                                                <?php else : ?>
                                                <p>
                                                <?php endif; ?>
                                                <span>
                                                    <b>Importe Total: </b>
                                                </span>
                                                <b id="importe_cifra_bill<?php echo $man_pago['id'] ?>" data-total="<?php echo $bill->get('total') ?>"><?php echo !empty($bill->get('total')) ? Number::currency($bill->get('total')) : '---' ?></b>
                                                </p>
                                        </div>
                                    </div>
                                </div>
                            <?php endforeach; ?>
                        </div>
                    <?php endif; ?>
                    <!-- Nuevas facturas -->
                    <br><br>
                    <hr>
                    <?php if (empty($facturaVieja)) : ?>
                        <div class="row">
                            <div class="col-md-12">
                                <?php if (user()->isAllowed('#facturacion-team')) : ?>
                                    <div class="col-md-4">
                                        <!-- Agregar el desplegable con el nombre de los clientes -->
                                        <label for="selectCliente">Seleccionar Cliente para facturación:</label>
                                        <select id="selectCliente" class="form-control col-md-8">
                                            <option value="empty">-- Selecciona cliente --</option>
                                            <?php foreach ($clientes as $key => $client): ?>
                                                <?php $clientRole = $client->get('_joinData')->get('client_role'); ?>
                                                <?php if (strpos(mb_strtolower($clientRole), 'contrario') === false) : ?>
                                                    <?php
                                                    $clientId = $client->get('id');
                                                    $clientName = $client->get('empresa') ? $client->get('empresa-nombre-juridico') : $client->get('full_name');
                                                    $provinciaJpi = $client->get('province') ? $client->get('province') : '---';
                                                    $provinciaCanaria = ($provinciaJpi == 'SANTA CRUZ DE TENERIFE' || $provinciaJpi == 'PALMAS (LAS)') ? true : false;
                                                    $client->provinciaCanaria = $provinciaCanaria;
                                                    ?>
                                                    <option value="<?php echo $clientId ?>" data-man-pago-id="<?php echo $man_pago['id'] ?>"><?php echo $clientName ?> <i>(<?php echo $clientRole ?>) </i></option>
                                                <?php endif; ?>
                                            <?php endforeach; ?>
                                        </select>
                                    </div>
                                <?php endif; ?>
                            </div>
                            <div id="clienteTarjetonContainer<?php echo $man_pago['id'] ?>" class="col-md-12" style="display: none;" data-man-pago-cobro-arriaga="<?php echo $man_pago['cobro_arriaga']; ?>" data-man-pago-id="<?php echo $man_pago['id'] ?>">
                                <!-- Aquí se mostrará el tarjetón del cliente seleccionado -->
                                <script>
                                    var manPagoId = <?php echo $man_pago['id'] ?>;
                                    var cobroArriaga<?php echo $man_pago['id'] ?> = <?php echo !empty($man_pago['cobro_arriaga']) ? $man_pago['cobro_arriaga'] : 0 ?>;
                                </script>
                            </div>
                        </div>
                    <?php endif; ?>
                </div>
            </div>
        <?php endforeach; ?>
    </div>

    <div id="operaciones" class="tab-pane fade">
        <div class="row">
            <div class="col-md-12">
                <div class="alert alert-warning">
                    <span><em>Desarrollo futuro</em></span>
                </div>
            </div>
        </div>
    </div>
</div>

<div class="modal fade" id="add-man">
</div>




<script>
    function actualizarNumero(importePorcentual = null, id = null, manPagoId) {
        var importeTotal = window['cobroArriaga' + manPagoId] ? window['cobroArriaga' + manPagoId] : 0;
        var fraccionPorcentaje = parseFloat(importePorcentual);
        var numericoCalculado = (fraccionPorcentaje / 100) * importeTotal;
        var numericoCalculadoDecimales = numericoCalculado.toFixed(2);
        $('#importe_cifra_' + id).val(numericoCalculadoDecimales);
    }

    function eraseImportes() {
        $('[id^="importe_cifra_"]').val('');
        $('[id^="importe_porcentaje_"]').val('');
    }

    function mostrarConfirmacion(campo, valorAnterior) {
        var idCliente = obtenerIdClienteDesdeCampo(campo);
        var valorAnterior = campo.value;

        if (!cambiosPorCliente.hasOwnProperty(idCliente)) {
            if (confirm('¿Está usted seguro de cambiar el tipo impositivo predeterminado por el sistema por el tipo ' + campo.value + ' seleccionado?')) {
                campo.setAttribute('data-valor-anterior', campo.value);
                // Actualizar el objeto de cambios por cliente
                cambiosPorCliente[idCliente] = true;
            } else {
                campo.value = valorAnterior;
            }
        }
    }

    // Función para obtener el ID del cliente desde el campo
    function obtenerIdClienteDesdeCampo(campo) {
        var partesId = campo.id.split('_');
        return partesId[partesId.length - 1];
    }

    function mostrarTarjetonCliente(cliente, manPagoId) {
        var tarjetonHTML = construirTarjetonHTML(cliente, clients, manPagoId);
        // Agregar el nuevo tarjetón sin borrar el contenido anterior
        var index = 0; // Supongamos que queremos operar sobre el contenedor 0
        var containerId = 'clienteTarjetonContainer' + manPagoId;
        var container = document.getElementById(containerId);
        container.insertAdjacentHTML('beforeend', tarjetonHTML);

        container.style.display = 'block';
    }

    function construirTarjetonHTML(selectedId, clients, manPagoId) {
        selectedId = parseInt(selectedId, 10);

        var clienteSeleccionado;
        for (var i = 0; i < clients.length; i++) {
            if (clients[i].id === selectedId) {
                clienteSeleccionado = clients[i];
                break; // Salir del bucle una vez que se encuentra el cliente
            }
        }

        var clienteRole = clienteSeleccionado._joinData.client_role;
        var provinciaCanaria = clienteSeleccionado.provinciaCanaria;

        var icon = '';
        var clientName = '';
        var clientId = clienteSeleccionado.id;
        var manPagoId = manPagoId


        if (!clienteSeleccionado.empresa) {
            clientName = clienteSeleccionado.name + " " + clienteSeleccionado.lastname;
            icon = clienteRole.includes('primer titular') ? '<i class="fa fa-user-circle-o"></i>' : '<i class="fa fa-user-o"></i>';
        } else {
            clientName = clienteSeleccionado['empresa-nombre-juridico'];
            icon = '<i class="fa fa-building"></i>';
        }

        if (clienteSeleccionado.deceased) {
            icon += '<span class="fa-stack fa-lg" title="fallecido"><i class="fa fa-minus fa-rotate-90 fa-stack-1x" aria-hidden="true" style="left: -1px;"></i><i class="fa fa-plus fa-stack-1x" style="top: -2px;"></i></span>';
        }

        var tipoFacturaElemento = document.getElementById('tipoFactura');
        var tipoFacturaValorCompleto = tipoFacturaElemento.textContent.trim();
        var indiceSeparador = tipoFacturaValorCompleto.indexOf(':');
        var tipoFacturaValor = tipoFacturaValorCompleto.substring(indiceSeparador + 1).trim();

        var tipoInput = `
            <label for="tipo_${clientId}">Tipo factura</label>
            <br>
            <select id="tipo_${clientId}" name="tipo_${clientId}" value="">
                <option value=""></option>
                ${Object.keys(arrTipo).map(function(key) {
                    return `<option value="${key}" ${tipoFacturaValor === arrTipo[key] ? 'selected' : ''}>${arrTipo[key]}</option>`;
                }).join('')}
            </select>
        `;
        var impuestoInput = `
            <label for="impuesto_${clientId}">Impuesto</label>
            <select id="impuesto_${clientId}" name="impuesto_${clientId}" onchange="mostrarConfirmacion(this, '${provinciaCanaria ? 'IGIC' : 'IVA ORDINARIO'}')">
                <option value="IVA" ${!provinciaCanaria ? 'selected' : ''}>IVA ORDINARIO</option>
                <option value="IGIC" ${provinciaCanaria ? 'selected' : ''}>IGIC</option>
                <option value="NINGUNO">EXENTO</option>
            </select>
        `;
        var importeCifraInput = `
            <label for="importe_cifra_${clientId}">Importe (€)</label>
            <input type="number" id="importe_cifra_${clientId}" name="importe_cifra_${clientId}" data-man-pago-id="${manPagoId}" step="any" value="">
        `;
        var importePorcentajeInput = `
            <label for="importe_porcentaje_${clientId}">Importe (%)</label>
            <input type="number" id="importe_porcentaje_${clientId}" name="importe_porcentaje_${clientId}" data-man-pago-id="${manPagoId}" step="any" value="" formatter="percent">
        `;

        var conceptoFacturaElemento = document.getElementById('conceptoFactura');
        var conceptoFacturaValorCompleto = conceptoFacturaElemento.textContent.trim();
        var conceptoindiceSeparador = conceptoFacturaValorCompleto.indexOf(':');
        var conceptoFacturaValor = conceptoFacturaValorCompleto.substring(conceptoindiceSeparador + 1).trim();

        var conceptoInput = `
            <label for="concepto_${clientId}">Concepto factura</label>
            <br>
            <textarea id="concepto_${clientId}" name="concepto_${clientId}" rows="6" style="resize:none; width: 100%;"">${conceptoFacturaValor}</textarea>
        `;

        var tarjetonHTML = `
            <div id="factura_${clientId}" class="col-md-12" style="background-color: #f5f5dc; border-radius: 5px; padding: 10px; border: 1px solid #669; margin-top:5px; margin-bottom:5px;">
                <div class="row">
                    <div class="col-md-4">
                        <em><small>(${clienteRole})</small></em><br>${icon} <strong>${clientName}</strong>
                    </div>
                    <div class="col-md-3">
                        ${tipoInput}
                    </div>
                    <div class="col-md-2">
                        ${impuestoInput}
                    </div>
                    <div class="col-md-3">
                        ${importeCifraInput}
                    </div>
                </div>
                <div class="row">
                    <div class="col-md-9">
                        ${conceptoInput}
                    </div>
                    <div class="col-md-3" style="margin-bottom: 10px;">
                        ${importePorcentajeInput}
                    </div>
                    <div class="col-md-3">
                        <button type="button" class="btn btn-success fa fa-floppy-o add-bill" title="Guardar propuesta de factura individual" data-client-id="${clientId}" data-man-pago-id="${manPagoId}"> Guardar </button>
                    </div>
                </div>
            </div>
        `;

        return tarjetonHTML;
    }

    $(document).ready(function() {
        //cambio de pestaña
        var urlParams = new URLSearchParams(window.location.search);
        if (urlParams.has('tab')) {
            var tab = urlParams.get('tab');
            var myUrl = window.location.href;
            myUrl = myUrl.replace('procedure-man', tab);
            window.location = myUrl;
        }
        $(document).on('change', 'select[id^="selectCliente"]', function() {
            var selectedId = $(this).val();
            if (selectedId !== 'empty') {
                var manPagoId = $(this).find('option:selected').data('man-pago-id');
                mostrarTarjetonCliente(selectedId, manPagoId);
            }
        });

        $(document).on('change', 'select[id^="tipo_"]', function() {
            var select = $(this);
            var numero = this.id.replace('tipo_', '');
            // Construir el id del campo concepto correspondiente
            var idConcepto = 'concepto_' + numero;

            var valorTipo = select.val();

            if (select.val() === '') {
                $('#' + idConcepto).val(null);
            } else {
                concepto = conceptos[select.val()]
                    .replace('[PROC]', procedimiento)
                    .replace('[APE]', apelacion)
                    .replace('[CAS]', casacion)
                    .replace('[NOMBRE JUZGADO]', juzgado)
                    .replace('[NOMBRE AUDIENCIA]', audiencia)
                    .replace('[SEC]', seccion)
                    .replace('[PARTIDO JUDICIAL]', ciudad_juzgado);

                $('#' + idConcepto).val(concepto);
            }
        });

        $(document).on('change', 'input[type="number"][id^="importe_cifra_"]', function() {
            var numberInput = $(this);
            var numeroCifra = this.id.replace('importe_cifra_', '');
            var idImporteCifra = 'importe_cifra_' + numeroCifra;
            var importeCifra = parseFloat($('#' + idImporteCifra).val());
            var manPagoId = numberInput.data('man-pago-id');

            if (!importeCifra) {
                importeCifra = 0;
            }

            var importeConDecimales = importeCifra.toFixed(2);
            $('#' + idImporteCifra).val(importeConDecimales);

            actualizarPorcentaje(importeCifra, numeroCifra, manPagoId);
        });

        $(document).on('change', 'input[type="number"][id^="importe_porcentaje_"]', function() {
            var porcentajeInput = $(this);
            var numeroPorcentual = this.id.replace('importe_porcentaje_', '');
            var idImportePorcentual = 'importe_porcentaje_' + numeroPorcentual;
            var importePorcentual = parseFloat($('#' + idImportePorcentual).val());
            var manPagoId = porcentajeInput.data('man-pago-id');

            if (!importePorcentual) {
                importePorcentual = 0;
            }

            var importePorcentualDecimales = importePorcentual.toFixed(2);
            $('#' + idImportePorcentual).val(importePorcentualDecimales);

            actualizarNumero(importePorcentual, numeroPorcentual, manPagoId);
        });

        $(document).on('click', 'button[class*=add-bill]', function() {
            var $btn = $(this);
            $('#spinner').show();
            $btn.attr("readonly", true);
            var clientId = $btn.data('client-id');
            var manPagoId = $btn.data('man-pago-id');
            var tipo = $('#tipo_' + clientId).val();
            var impuesto = $('#impuesto_' + clientId).val();
            var importe = $('#importe_cifra_' + clientId).val();
            var concepto = $('#concepto_' + clientId).val();

            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'addBill'], true); ?>',
                    dataType: 'json',
                    data: {
                        payment_order: manPagoId,
                        client_id: clientId,
                        record_id: '<?php echo $record->get("id"); ?>',
                        tipo: tipo,
                        impuesto: impuesto,
                        importe: importe,
                        concepto: concepto,
                    },
                })
                .success(function(data) {
                    $('#spinner').hide();
                    location.reload();
                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    $('#spinner').hide();
                    alert(jqXHR.responseJSON.message);
                    $('#tipo_' + clientId).val('');
                    $('#impuesto_' + clientId).val('');
                    $('#importe_cifra_' + clientId).val('');
                    $('#importe_porcentaje_' + clientId).val('');
                    $('#concepto_' + clientId).val('');
                });

            return false;
        });

        $('.docbuttons.file2button[data-file-name][data-record-id]').file2button();
        $('.est-success').hide();
        $('.est-warning').hide();
        $('.est-danger').hide();
        $('.num-cuenta').removeClass('has-error').removeClass('has-success');
        $('.iban-fail').hide();
        $('.iban-success').hide();
        $('.iban-fail').tooltip('toggle');

        $('button.refresh').on('click', function() {
            var $btn = $(this);
            $btn.attr("readonly", true);
            $btn.button('calculando');

            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'refreshCostas', $record->get('id'),], true); ?>',
                    dataType: 'json',
                })
                .done(function(data) {
                    if (!data.costas && !data.costasProc) {
                        $flashMsg = $(".est-danger");
                    } else if ((data.costas != 0 && !data.costas) && data.costasProc > 0) {
                        $flashMsg = $(".est-warning");
                    } else {
                        $flashMsg = $(".est-success");
                    }

                    if (data.costasProc > 0) {
                        $('#est-costas-proc').val(data.costasProc);
                    }
                    if (data.costas > 0) {
                        $('#importe-a-ingresar').val(data.costas);
                    }

                    $flashMsg.html(data.flash);
                    $flashMsg.show(1000);
                    setTimeout(function() {
                        $flashMsg.hide(1000);
                    }, 9000);

                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    alert(jqXHR.responseJSON.message);
                })
                .always(function() {
                    $btn.button('reset');
                    $btn.attr("readonly", false);
                });

            return false;
        });

        $('button.add-reminder').on('click', function() {
            var $btn = $(this);
            $btn.button('calculando');
            var fase = $btn.data('fase-id');
            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'addRemind'], true); ?>',
                    dataType: 'json',
                    data: {
                        msg_alert: 'Mandamiento de pago FASE ' + fase + ' preparado para transferencia',
                        user_alert: 709,
                        record_id: '<?php echo $record->get("id"); ?>'
                    },
                })
                .done(function(data) {
                    $flashMsg = $(".est-success");
                    alert('Alerta enviada satisfactoriamente (FASE ' + fase + ')');

                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    alert(jqXHR.responseJSON.message);
                })
                .always(function() {
                    $btn.button('reset');
                    $btn.attr("readonly", false);
                });

            return false;
        });

        $('.view-lines').on('click', function() {
            var $btn = $(this);
            $btn.button('añadiendo');

            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'renderFormLines', $record->get('id'),], true); ?>',
                    dataType: 'edit',
                    data: {
                        type: 'edit'
                    },
                })
                .done(function(data) {
                    $modal = $('#view-lines');
                    $modal.html(data);
                    $modal.modal('show');
                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    alert(jqXHR.responseJSON.message);
                });
        });

        $('.add-lines').on('click', function() {
            event.preventDefault();
            var $btn = $(this);
            $btn.button('añadiendo');
            var billId = $btn.data('bill-id');
            $('#spinner').show();

            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'renderFormLines', $record->get('id'),], true); ?>',
                    dataType: 'html',
                    data: {
                        type: 'add',
                        bill_id: billId
                    },
                })
                .done(function(data) {
                    $('#spinner').hide();
                    $modal = $('#add-lines');
                    $modal.html(data);
                    $modal.modal('show');
                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    $('#spinner').hide();
                    alert(jqXHR.responseJSON.message);
                });
        });

        $('button.edit-line').on('click', function() {
            event.preventDefault();
            var $btn = $(this);
            $btn.button('editando');
            var billId = $btn.data('bill-id');
            var billLineId = $btn.data('bill-line-id');
            $('#spinner').show();

            $.ajax({
                    type: 'POST',
                    url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'renderFormLines', $record->get('id'),], true); ?>',
                    dataType: 'html',
                    data: {
                        type: 'edit',
                        bill_id: billId,
                        bill_line_id: billLineId
                    },
                })
                .done(function(data) {
                    $('#spinner').hide();
                    $modal = $('#edit-lines');
                    $modal.html(data);
                    $modal.modal('show');
                })
                .fail(function(jqXHR, textStatus, errorThrown) {
                    $('#spinner').hide();
                    alert(jqXHR.responseJSON.message);
                });
        });

        $('button.remove-line').on('click', function() {
            event.preventDefault();
            var $btn = $(this);

            if (confirm("¿Estás seguro de que quieres eliminar este concepto?")) {
                $btn.button('añadiendo');
                var billLineId = $btn.data('bill-line-id');
                $('#spinner').show();

                $.ajax({
                        type: 'POST',
                        url: '<?php echo $this->Url->build(['plugin' => 'Records', 'controller' => 'Ajax', 'action' => 'removeBillLine'], true); ?>',
                        dataType: 'json',
                        data: {
                            bill_line_id: billLineId
                        },
                    })
                    .done(function(data) {
                        $('#spinner').hide();
                        location.reload();
                    })
                    .fail(function(jqXHR, textStatus, errorThrown) {
                        $('#spinner').hide();
                        alert(jqXHR.responseJSON.data.message);
                        $btn.button('reset');
                    });
            }


            $('#importe_a_ingresar').val(total.toFixed(2));
        });

        updateImporteIngresar();

        $('#costas').on('input change', 'input[type="number"], select', function() {
            updateImporteIngresar();
        });


        $('#costas').on('change', 'select[id^="tipo-select"], select[id^="fase-select"]', updateImporteIngresar);


        $('#costas').on('blur', 'input[type="number"]', function() {
            updateImporteIngresar();
        });

        function updateImporteIngresar() {
            const tipoSelect = $('select[id^="tipo-select"]');
            const selectedTipo = tipoSelect.length ? tipoSelect.val().toLowerCase() : '';


            if (selectedTipo === 'a favor') {
                let total = 0;


                $('.cost-content').each(function() {
                    const faseSelect = $(this).find('select[id^="fase-select"]');
                    const selectedFase = faseSelect.length ? faseSelect.val().toLowerCase() : '';


                    if (selectedFase === 'presentacion') {
                        total += parseFloat($(this).find('input[id^="imp_costas_"]').val()) || 0;
                        total += parseFloat($(this).find('input[id^="imp_perito_"]').val()) || 0;
                        total += parseFloat($(this).find('input[id^="imp_testigo_"]').val()) || 0;
                    } else if (selectedFase === 'traslado') {
                        total += parseFloat($(this).find('input[id^="procurador_importe_laj_"]').val()) || 0;
                        total += parseFloat($(this).find('input[id^="importe_laj_"]').val()) || 0;
                    } else if (selectedFase === 'decreto') {
                        total += parseFloat($(this).find('input[id^="importe_decreto_"]').val()) || 0;
                        total += parseFloat($(this).find('input[id^="procurador_importe_decreto_"]').val()) || 0;
                    } else if (selectedFase === 'resolucion') {
                        total += parseFloat($(this).find('input[id^="importe_resolucion_"]').val()) || 0;
                        total += parseFloat($(this).find('input[id^="procurador_importe_resolucion_"]').val()) || 0;
                    }
                });


                $('#importe_a_ingresar').val(total.toFixed(2));
            }
        }



    });
</script>
```
[[Programación]]
