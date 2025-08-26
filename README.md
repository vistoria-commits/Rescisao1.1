<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Calculadora Avançada de Despesas Imobiliárias</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
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
  <label>Valor mensal do aluguel (R$): <input type="number" id="aluguelMensal" required></label>
  <label>Data início: <input type="date" id="dataInicioAluguel" required></label>
  <label>Data fim: <input type="date" id="dataFimAluguel" required></label>

  <h3>IPTU</h3>
  <label>Total do IPTU (R$): <input type="number" id="iptuTotal" required></label>
  <label>Valor já pago (R$): <input type="number" id="iptuPago" required></label>
  <label>Data início uso IPTU: <input type="date" id="iptuInicio" required></label>
  <label>Data fim uso IPTU: <input type="date" id="iptuFim" required></label>

  <h3>Seguro Incêndio</h3>
  <label>Valor total (R$): <input type="number" id="seguroTotal" required></label>
  <label>Valor pago (R$): <input type="number" id="seguroPago" required></label>
  <label>Data início uso seguro: <input type="date" id="seguroInicio" required></label>
  <label>Data fim uso seguro: <input type="date" id="seguroFim" required></label>

  <h3>Pintura</h3>
  <label>Valor total (R$): <input type="number" id="pinturaTotal" required></label>
  <label>Parcelas pagas: <input type="number" id="pinturaParcelas" required></label>

  <h3>Manutenção</h3>
  <label>Valor estimado (R$): <input type="number" id="manutencao" required></label>

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

  document.getElementById('formulario').addEventListener('submit', function(e) {
    e.preventDefault();

    // Aluguel pró-rata
    const aluguelMensal = parseFloat(document.getElementById('aluguelMensal').value);
    const aluguelDias = diasEntreDatas(
      document.getElementById('dataInicioAluguel').value,
      document.getElementById('dataFimAluguel').value
    );
    const aluguelDiario = aluguelMensal / 30;
    const aluguelTotal = aluguelDiario * aluguelDias;

    // IPTU proporcional
    const iptuTotal = parseFloat(document.getElementById('iptuTotal').value);
    const iptuPago = parseFloat(document.getElementById('iptuPago').value);
    const iptuDias = diasEntreDatas(
      document.getElementById('iptuInicio').value,
      document.getElementById('iptuFim').value
    );
    const iptuProporcional = (iptuTotal / 360) * iptuDias;
    const iptuDiferenca = iptuProporcional - iptuPago;

    // Seguro incêndio
    const seguroTotal = parseFloat(document.getElementById('seguroTotal').value);
    const seguroPago = parseFloat(document.getElementById('seguroPago').value);
    const seguroDias = diasEntreDatas(
      document.getElementById('seguroInicio').value,
      document.getElementById('seguroFim').value
    );
    const seguroProporcional = (seguroTotal / 360) * seguroDias;
    const seguroDiferenca = seguroPago - seguroProporcional;

    // Pintura
    const pinturaTotal = parseFloat(document.getElementById('pinturaTotal').value);
    const pinturaParcelas = parseInt(document.getElementById('pinturaParcelas').value);

    // Manutenção
    const manutencaoBase = parseFloat(document.getElementById('manutencao').value);
    const manutencaoTotal = manutencaoBase * 1.2;

    // Total final
    const totalAPagar = 
      aluguelTotal +
      (iptuDiferenca > 0 ? iptuDiferenca : 0) +
      (seguroDiferenca < 0 ? Math.abs(seguroDiferenca) : 0) +
      pinturaTotal +
      manutencaoTotal;

    const resultado = document.getElementById('resultado');
    resultado.innerHTML = `
      <h3>Resumo:</h3>
      <p><strong>Aluguel pró-rata (${aluguelDias} dias):</strong> ${formatar(aluguelTotal)}</p>
      <p><strong>IPTU proporcional (${iptuDias} dias):</strong> ${formatar(iptuProporcional)} - Pago: ${formatar(iptuPago)} 
      → ${iptuDiferenca >= 0 ? "A pagar: " + formatar(iptuDiferenca) : "A devolver: " + formatar(Math.abs(iptuDiferenca))}</p>
      <p><strong>Seguro incêndio (${seguroDias} dias):</strong> ${formatar(seguroProporcional)} - Pago: ${formatar(seguroPago)} 
      → ${seguroDiferenca >= 0 ? "A devolver: " + formatar(seguroDiferenca) : "A pagar: " + formatar(Math.abs(seguroDiferenca))}</p>
      <p><strong>Pintura:</strong> ${formatar(pinturaTotal)} (${pinturaParcelas} parcelas pagas)</p>
      <p><strong>Manutenção (com 20%):</strong> ${formatar(manutencaoTotal)}</p>
      <hr>
      <strong>Total a pagar (exceto devoluções): ${formatar(totalAPagar)}</strong>
    `;
    resultado.style.display = 'block';
  });
</script>

</body>
</html>
