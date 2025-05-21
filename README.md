<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Contador com Senha e Envio</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
    }
    body {
      display: flex;
      justify-content: center;  /* centraliza horizontalmente */
      align-items: center;      /* centraliza verticalmente */
      flex-direction: column;
      font-family: Arial, sans-serif;
      transition: background-color 0.5s ease;
      background-color: white;
      min-height: 100vh;
    }
    #contador {
      font-size: 96px;
      font-weight: bold;
      color: #2c3e50;
      font-variant-numeric: tabular-nums;
      margin-bottom: 20px;
      user-select: none;
    }
    label {
      display: inline-block;
      margin-top: 10px;
      margin-bottom: 5px;
      font-weight: bold;
      font-size: 16px;
      color: #2c3e50;
    }
    input[type="text"] {
      font-size: 16px;
      padding: 7px;
      width: 250px;
      border: 1px solid #ccc;
      border-radius: 5px;
      margin-bottom: 10px;
    }
    .inputs-container {
      margin-bottom: 40px;
      text-align: center;
    }
    button {
      font-size: 18px;
      padding: 10px 20px;
      margin: 10px 8px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background-color: #3498db;
      color: white;
      transition: background-color 0.3s;
    }
    button:disabled {
      background-color: #999;
      cursor: not-allowed;
    }
    button:hover:not(:disabled) {
      background-color: #2980b9;
    }
    #dadosInseridos {
      margin-top: 25px;
      font-size: 18px;
      color: #34495e;
      line-height: 1.5;
      max-width: 400px;
      text-align: center;
      white-space: pre-wrap;
      user-select: none;
    }
  </style>
</head>
<body>

  <div class="inputs-container">
    <div>
      <label for="placa">Placa:</label><br/>
      <input type="text" id="placa" placeholder="Digite a placa" />
    </div>
    <div>
      <label for="doca">Doca:</label><br/>
      <input type="text" id="doca" placeholder="Digite a doca" />
    </div>
    <div>
      <label for="transportadora">Transportadora:</label><br/>
      <input type="text" id="transportadora" placeholder="Digite a transportadora" />
    </div>
  </div>

  <div id="contador">00:00:00</div>

  <div>
    <button id="iniciarBtn" disabled>Iniciar</button>
    <button id="pararBtn" disabled>Parar</button>
    <button id="resetarBtn" disabled>Resetar</button>
  </div>

  <div id="dadosInseridos"></div>

  <script>
    const URL_WEB_APP = "https://script.google.com/macros/s/AKfycbxtZp5q79LkTadzus7tq2KBr_Sdg8lNFLP7DfjOTIG2u8cjcgq_yYiHSVexKAlTWsg/exec"; // substitua aqui

    const SENHA_CORRETA = "MELI123";

    let totalSegundos = 0;
    const maxSegundos = 40 * 60; // 40 minutos (ajuste se desejar)
    const contadorElement = document.getElementById('contador');
    const iniciarBtn = document.getElementById('iniciarBtn');
    const pararBtn = document.getElementById('pararBtn');
    const resetarBtn = document.getElementById('resetarBtn');

    const placaInput = document.getElementById('placa');
    const docaInput = document.getElementById('doca');
    const transportadoraInput = document.getElementById('transportadora');

    const dadosInseridos = document.getElementById('dadosInseridos');

    let intervalo = null;

    // Formatar segundos para HH:MM:SS
    function formatarTempo(segundos) {
      const hrs = Math.floor(segundos / 3600);
      const mins = Math.floor((segundos % 3600) / 60);
      const segs = segundos % 60;

      return (
        String(hrs).padStart(2, '0') + ':' +
        String(mins).padStart(2, '0') + ':' +
        String(segs).padStart(2, '0')
      );
    }

    // Atualiza cor do fundo conforme tempo
    function atualizarFundo(segundos) {
      if (segundos <= 15) {
        document.body.style.backgroundColor = 'green';
      } else if (segundos <= 30) {
        document.body.style.backgroundColor = 'yellow';
      } else {
        document.body.style.backgroundColor = 'red';
      }
    }

    // Atualiza contador e fundo a cada segundo
    function atualizarContador() {
      totalSegundos++;
      contadorElement.textContent = formatarTempo(totalSegundos);
      atualizarFundo(totalSegundos);

      if (totalSegundos >= maxSegundos) {
        clearInterval(intervalo);
        intervalo = null;
        iniciarBtn.disabled = false;
        pararBtn.disabled = true;
        resetarBtn.disabled = false;
      }
    }

    // Verifica se campos foram preenchidos para habilitar o iniciar
    function validarCampos() {
      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();

      iniciarBtn.disabled = !(placa && doca && transportadora);
    }

    // Eventos para validar inputs dinamicamente
    placaInput.addEventListener('input', validarCampos);
    docaInput.addEventListener('input', validarCampos);
    transportadoraInput.addEventListener('input', validarCampos);

    // Evento para iniciar o contador
    iniciarBtn.addEventListener('click', () => {
      if (intervalo) return;

      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();

      dadosInseridos.textContent =
        `Dados Iniciados:\nPlaca: ${placa}\nDoca: ${doca}\nTransportadora: ${transportadora}`;

      placaInput.disabled = true;
      docaInput.disabled = true;
      transportadoraInput.disabled = true;

      iniciarBtn.disabled = true;
      pararBtn.disabled = false;
      resetarBtn.disabled = false;

      intervalo = setInterval(atualizarContador, 1000);
    });

    // Evento para parar o contador (solicita senha)
    pararBtn.addEventListener('click', () => {
      if (!intervalo) return;

      const senha = prompt("Digite a senha para parar:");
      if (senha !== SENHA_CORRETA) {
        alert("Senha incorreta! Operação cancelada.");
        return;
      }

      clearInterval(intervalo);
      intervalo = null;

      // Não zera o contador, só para e mantém o valor exibido!

      enviarDados();

      iniciarBtn.disabled = true;  // evitar reinício para manter controle
      pararBtn.disabled = true;
      resetarBtn.disabled = false;
    });

    // Evento para resetar tudo
    resetarBtn.addEventListener('click', () => {
      if (intervalo) {
        clearInterval(intervalo);
        intervalo = null;
      }
      totalSegundos = 0;
      contadorElement.textContent = formatarTempo(totalSegundos);
      document.body.style.backgroundColor = 'white';

      placaInput.value = "";
      docaInput.value = "";
      transportadoraInput.value = "";
      placaInput.disabled = false;
      docaInput.disabled = false;
      transportadoraInput.disabled = false;

      dadosInseridos.textContent = "";

      iniciarBtn.disabled = true;
      pararBtn.disabled = true;
      resetarBtn.disabled = true;
    });

    // Envia os dados para o Google Apps Script
    function enviarDados() {
      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();
      const tempo = formatarTempo(totalSegundos);

      fetch(URL_WEB_APP, {
        method: "POST",
        mode: "cors",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ placa, doca, transportadora, tempo })
      })
      .then(response => {
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
      })
      .then(data => {
        if(data.status === "success") {
          alert("Dados enviados com sucesso!");
        } else {
          alert("Erro ao enviar dados: " + (data.message || "Desconhecido"));
        }
      })
      .catch(error => {
        alert("Falha ao enviar dados: " + error);
      });
    }

    // Inicialização da página
    contadorElement.textContent = formatarTempo(totalSegundos);
    document.body.style.backgroundColor = 'white';
    iniciarBtn.disabled = true;
    pararBtn.disabled = true;
    resetarBtn.disabled = true;
  </script>

</body>
</html>
