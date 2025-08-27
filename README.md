<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Calculadora Avançada de Despesas Imobiliárias</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 900px;
      margin: 20px auto;
      padding: 20px;
      background-color: #f4f4f4;
      border-radius: 8px;
      box-shadow: 0 0 10px #ccc;
    }
    h2 { text-align: center; }
    label { display: block; margin-top: 12px; }
    input { width: 100%; padding: 8px; margin-top: 4px; box-sizing: border-box; }
    button {
      margin-top: 20px;
      padding: 10px;
      width: 100%;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      font-size: 16px;
    }
    .resultado {
      margin-top: 20px;
      padding: 15px;
      background: #fff;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    hr { margin: 15px 0; }
  </style>
</head>
<body>

<h2>Calculadora Avançada de Despesas</h2>

<form id="formulario">
  <h3>Aluguel</h3>
  <label>Valor mensal do aluguel (R$): <input type="text" id="aluguelMensal" required></label>
  <label>Data início: <input type="date" id="dataInicioAluguel" required></label>
  <label>Data fim: <input type="date" id="dataFimAluguel" required></label>

  <h3>IPTU</h3>
  <label>Valor da parcela (R$): <input type="text" id="iptuParcela"></label>
  <label>Quantidade de parcelas: <input type="text" id="iptuQtdParcelas"></label>
  <label>Total do IPTU (R$): <input type="text" id="iptuTotal" readonly></label>
  <label>Valor já pago (R$): <input type="text" id="iptuPago"></label>
  <label>Data início uso IPTU: <input type="date" id="iptuInicio" required></label>
  <label>Data fim uso IPTU: <input type="date" id="iptuFim" required></label>

  <h3>Seguro Incêndio</h3>
  <label>Valor da parcela (R$): <input type="text" id="seguroParcela"></label>
  <label>Quantidade de parcelas: <input type="text" id="seguroQtdParcelas"></label>
  <label>Valor total (R$): <input type="text" id="seguroTotal" readonly></label>
  <label>Valor pago (R$): <input type="text" id="seguroPago"></label>
  <label>Data início uso seguro: <input type="date" id="seguroInicio" required></label>
  <label>Data fim uso seguro: <input type="date" id="seguroFim" required></label>

  <h3>Água</h3>
  <label>Valor total (R$): <input type="text" id="aguaTotal"></label>
  <label>Valor pago (R$): <input type="text" id="aguaPago"></label>

  <h3>Luz</h3>
  <label>Valor total (R$): <input type="text" id="luzTotal"></label>
  <label>Valor pago (R$): <input type="text" id="luzPago"></label>

  <h3>Condomínio</h3>
  <label>Valor total (R$): <input type="text" id="condTotal"></label>
  <label>Valor pago (R$): <input type="text" id="condPago"></label>

  <h3>Gás</h3>
  <label>Valor total (R$): <input type="text" id="gasTotal"></label>
  <label>Valor pago (R$): <input type="text" id="gasPago"></label>

  <h3>Pintura</h3>
  <label>Valor total (R$): <input type="text" id="pinturaTotal"></label>
  <label>Valor pago (R$): <input type="text" id="pinturaPago"></label>

  <h3>Manutenção</h3>
  <label>Valor estimado (R$): <input type="text" id="manutencao"></label>

  <button type="submit">Calcular</button>
</form>

<div class="resultado" id="resultado" style="display: none;"></div>

<script>
  const formatar = valor => new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL'
  }).format(valor);

  const diasEntreDatas = (inicio, fim) => {
    const i = new Date(inicio);
    const f = new Date(fim);
    return Math.floor((f - i) / (1000 * 60 * 60 * 24)) + 1;
  };

  const lerNumero = id => parseFloat((document.getElementById(id).value || "0").replace(",", "."));

  function atualizarIptuTotal() {
    const parcela = lerNumero('iptuParcela');
    const qtd = lerNumero('iptuQtdParcelas');
    document.getElementById('iptuTotal').value = parcela * qtd;
  }
  document.getElementById('iptuParcela').addEventListener('input', atualizarIptuTotal);
  document.getElementById('iptuQtdParcelas').addEventListener('input', atualizarIptuTotal);

  function atualizarSeguroTotal() {
    const parcela = lerNumero('seguroParcela');
    const qtd = lerNumero('seguroQtdParcelas');
    document.getElementById('seguroTotal').value = parcela * qtd;
  }
  document.getElementById('seguroParcela').addEventListener('input', atualizarSeguroTotal);
  document.getElementById('seguroQtdParcelas').addEventListener('input', atualizarSeguroTotal);

  document.getElementById('formulario').addEventListener('submit', function(e) {
    e.preventDefault();

    let resumo = "<h3>Resumo:</h3>";
    let totalPagar = 0;
    let totalDevolver = 0;

    // Aluguel pró-rata
    const aluguelMensal = lerNumero('aluguelMensal');
    const aluguelDias = diasEntreDatas(document.getElementById('dataInicioAluguel').value, document.getElementById('dataFimAluguel').value);
    const aluguelDiario = aluguelMensal / 30;
    const aluguelTotal = aluguelDiario * aluguelDias;
    resumo += `<p><strong>Aluguel pró-rata (${aluguelDias} dias):</strong> ${formatar(aluguelTotal)}</p>`;
    totalPagar += aluguelTotal;

    // IPTU
    const iptuTotal = lerNumero('iptuTotal');
    const iptuPago = lerNumero('iptuPago');
    const iptuDias = diasEntreDatas(document.getElementById('iptuInicio').value, document.getElementById('iptuFim').value);
    const iptuProporcional = (iptuTotal / 360) * iptuDias;
    const iptuDiferenca = iptuProporcional - iptuPago;
    if (iptuPago !== 0) {
      resumo += `<p><strong>IPTU proporcional (${iptuDias} dias):</strong> ${formatar(iptuProporcional)} - Pago: ${formatar(iptuPago)} → ${iptuDiferenca >= 0 ? "A pagar: " + formatar(iptuDiferenca) : "A devolver: " + formatar(Math.abs(iptuDiferenca))}</p>`;
      if (iptuDiferenca > 0) totalPagar += iptuDiferenca; else totalDevolver += Math.abs(iptuDiferenca);
    }

    // Seguro
    const seguroTotal = lerNumero('seguroTotal');
    const seguroPago = lerNumero('seguroPago');
    const seguroDias = diasEntreDatas(document.getElementById('seguroInicio').value, document.getElementById('seguroFim').value);
    const seguroProporcional = (seguroTotal / 360) * seguroDias;
    const seguroDiferenca = seguroPago - seguroProporcional;
    if (seguroPago !== 0) {
      resumo += `<p><strong>Seguro incêndio (${seguroDias} dias):</strong> ${formatar(seguroProporcional)} - Pago: ${formatar(seguroPago)} → ${seguroDiferenca >= 0 ? "A devolver: " + formatar(seguroDiferenca) : "A pagar: " + formatar(Math.abs(seguroDiferenca))}</p>`;
      if (seguroDiferenca < 0) totalPagar += Math.abs(seguroDiferenca); else totalDevolver += seguroDiferenca;
    }

    // Água
    const aguaTotal = lerNumero('aguaTotal');
    const aguaPago = lerNumero('aguaPago');
    const aguaDiferenca = aguaTotal - aguaPago;
    if (aguaPago !== 0) {
      resumo += `<p><strong>Água:</strong> ${formatar(aguaTotal)} - Pago: ${formatar(aguaPago)} → ${aguaDiferenca >= 0 ? "A pagar: " + formatar(aguaDiferenca) : "A devolver: " + formatar(Math.abs(aguaDiferenca))}</p>`;
      if (aguaDiferenca > 0) totalPagar += aguaDiferenca; else totalDevolver += Math.abs(aguaDiferenca);
    }

    // Luz
    const luzTotal = lerNumero('luzTotal');
    const luzPago = lerNumero('luzPago');
    const luzDiferenca = luzTotal - luzPago;
    if (luzPago !== 0) {
      resumo += `<p><strong>Luz:</strong> ${formatar(luzTotal)} - Pago: ${formatar(luzPago)} → ${luzDiferenca >= 0 ? "A pagar: " + formatar(luzDiferenca) : "A devolver: " + formatar(Math.abs(luzDiferenca))}</p>`;
      if (luzDiferenca > 0) totalPagar += luzDiferenca; else totalDevolver += Math.abs(luzDiferenca);
    }

    // Condomínio
    const condTotal = lerNumero('condTotal');
    const condPago = lerNumero('condPago');
    const condDiferenca = condTotal - condPago;
    if (condPago !== 0) {
      resumo += `<p><strong>Condomínio:</strong> ${formatar(condTotal)} - Pago: ${formatar(condPago)} → ${condDiferenca >= 0 ? "A pagar: " + formatar(condDiferenca) : "A devolver: " + formatar(Math.abs(condDiferenca))}</p>`;
      if (condDiferenca > 0) totalPagar += condDiferenca; else totalDevolver += Math.abs(condDiferenca);
    }

    // Gás
    const gasTotal = lerNumero('gasTotal');
    const gasPago = lerNumero('gasPago');
    const gasDiferenca = gasTotal - gasPago;
    if (gasPago !== 0) {
      resumo += `<p><strong>Gás:</strong> ${formatar(gasTotal)} - Pago: ${formatar(gasPago)} → ${gasDiferenca >= 0 ? "A pagar: " + formatar(gasDiferenca) : "A devolver: " + formatar(Math.abs(gasDiferenca))}</p>`;
      if (gasDiferenca > 0) totalPagar += gasDiferenca; else totalDevolver += Math.abs(gasDiferenca);
    }

    // Pintura
    const pinturaTotal = lerNumero('pinturaTotal');
    const pinturaPago = lerNumero('pinturaPago');
    const pinturaDiferenca = pinturaTotal - pinturaPago;
    if (pinturaPago !== 0 && pinturaDiferenca !== 0) {
      resumo += `<p><strong>Pintura:</strong> ${formatar(pinturaTotal)} - Pago: ${formatar(pinturaPago)} → ${pinturaDiferenca > 0 ? "A pagar: " + formatar(pinturaDiferenca) : "A devolver: " + formatar(Math.abs(pinturaDiferenca))}</p>`;
      if (pinturaDiferenca > 0) totalPagar += pinturaDiferenca; else totalDevolver += Math.abs(pinturaDiferenca);
    }

    // Manutenção
    const manutencaoBase = lerNumero('manutencao');
    const manutencaoTotal = manutencaoBase * 1.2;
    if (manutencaoBase !== 0) {
      resumo += `<p><strong>Manutenção (com 20%):</strong> ${formatar(manutencaoTotal)}</p>`;
      totalPagar += manutencaoTotal;
    }

    resumo += "<hr>";
    resumo += `<p><strong>Total a pagar:</strong> ${formatar(totalPagar)}</p>`;
    if (totalDevolver > 0) resumo += `<p><strong>Total a devolver:</strong> ${formatar(totalDevolver)}</p>`;

    document.getElementById('resultado').innerHTML = resumo;
    document.getElementById('resultado').style.display = 'block';
  });
</script>

</body>
</html>
