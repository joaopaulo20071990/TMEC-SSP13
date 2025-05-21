<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Contador com Senha e Envio</title>
  <style>
    /* Reset básico */
    * {
      box-sizing: border-box;
    }

    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
      font-family: Arial, sans-serif;
      background-color: white;
    }

    body {
      display: flex;
      justify-content: center;  /* horizontal */
      align-items: center;      /* vertical */
      min-height: 100vh;
      padding: 20px;
    }

    .container {
      width: 100%;
      max-width: 480px;
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #f7f9fc;
      padding: 30px 40px;
      border-radius: 10px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.1);
    }

    #contador {
      font-size: 96px;
      font-weight: bold;
      color: #2c3e50;
      font-variant-numeric: tabular-nums;
      margin-bottom: 30px;
      user-select: none;
    }

    .inputs-container {
      width: 100%;
      margin-bottom: 30px;
    }

    label {
      font-weight: bold;
      font-size: 16px;
      color: #2c3e50;
      display: block;
      margin-bottom: 6px;
    }

    input[type="text"] {
      width: 100%;
      padding: 8px 12px;
      font-size: 16px;
      border: 1px solid #ccc;
      border-radius: 6px;
      margin-bottom: 16px;
      transition: border-color 0.3s;
    }

    input[type="text"]:focus {
      border-color: #3498db;
      outline: none;
      box-shadow: 0 0 5px rgba(52, 152, 219, 0.5);
    }

    .buttons-container {
      display: flex;
      justify-content: center;
      gap: 15px;
      margin-bottom: 25px;
      flex-wrap: wrap;
    }

    button {
      flex: 1 1 100px;
      font-size: 18px;
      padding: 12px 16px;
      cursor: pointer;
      border: none;
      border-radius: 6px;
      background-color: #3498db;
      color: white;
      transition: background-color 0.3s;
      user-select: none;
      min-width: 100px;
    }

    button:disabled {
      background-color: #999;
      cursor: not-allowed;
    }

    button:hover:not(:disabled) {
      background-color: #2980b9;
    }

    #dadosInseridos {
      width: 100%;
      font-size: 18px;
      color: #34495e;
      line-height: 1.5;
      white-space: pre-wrap;
      text-align: center;
      min-height: 70px;
      user-select: none;
    }

    @media(max-width: 400px) {
      .container {
        padding: 20px 25px;
      }
      #contador {
        font-size: 72px;
      }
      button {
        min-width: 80px;
        font-size: 16px;
      }
    }
  </style>
</head>
<body>

  <div class="container">
    <div id="contador">00:00:00</div>

    <div class="inputs-container">
      <label for="placa">Placa:</label>
      <input type="text" id="placa" placeholder="Digite a placa" />

      <label for="doca">Doca:</label>
      <input type="text" id="doca" placeholder="Digite a doca" />

      <label for="transportadora">Transportadora:</label>
      <input type="text" id="transportadora" placeholder="Digite a transportadora" />
    </div>

    <div class="buttons-container">
      <button id="iniciarBtn" disabled>Iniciar</button>
      <button id="pararBtn" disabled>Parar</button>
      <button id="resetarBtn" disabled>Resetar</button>
    </div>

    <div id="dadosInseridos"></div>
  </div>

  <script>
    const URL_WEB_APP = "URL_DO_SEU_WEB_APP"; // substitua aqui

    const SENHA_CORRETA = "MELI123";

    let totalSegundos = 0;
    const maxSegundos = 40 * 60; // 40 minutos
    const contadorElement = document.getElementById('contador');
    const iniciarBtn = document.getElementById('iniciarBtn');
    const pararBtn = document.getElementById('pararBtn');
    const resetarBtn = document.getElementById('resetarBtn');

    const placaInput = document.getElementById('placa');
    const docaInput = document.getElementById('doca');
    const transportadoraInput = document.getElementById('transportadora');

    const dadosInseridos = document.getElementById('dadosInseridos');

    let intervalo = null;

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

    function atualizarFundo(segundos) {
      if (segundos <= 15) {
        document.body.style.backgroundColor = 'green';
      } else if (segundos <= 30) {
        document.body.style.backgroundColor = 'yellow';
      } else {
        document.body.style.backgroundColor = 'red';
      }
    }

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

    function validarCampos() {
      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();

      iniciarBtn.disabled = !(placa && doca && transportadora);
    }

    placaInput.addEventListener('input', validarCampos);
    docaInput.addEventListener('input', validarCampos);
    transportadoraInput.addEventListener('input', validarCampos);

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

    pararBtn.addEventListener('click', () => {
      if (!intervalo) return;

      const senha = prompt("Digite a senha para parar:");
      if (senha !== SENHA_CORRETA) {
        alert("Senha incorreta! Operação cancelada.");
        return;
      }

      clearInterval(intervalo);
      intervalo = null;

      enviarDados();

      iniciarBtn.disabled = true;
      pararBtn.disabled = true;
      resetarBtn.disabled = false;
    });

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
    iniciarBtn.disabled = true;
    pararBtn.disabled = true;
    resetarBtn.disabled = true;
  </script>

</body>
</html>
