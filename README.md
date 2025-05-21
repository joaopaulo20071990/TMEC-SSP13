<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Contador com Senha e Envio</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin-top: 30px;
      transition: background-color 0.5s ease;
    }
    #contador {
      font-size: 48px;
      font-weight: bold;
      color: #2c3e50;
      margin-bottom: 20px;
      font-variant-numeric: tabular-nums;
    }
    label {
      display: inline-block;
      margin-top: 10px;
      margin-bottom: 5px;
      font-weight: bold;
    }
    input[type="text"] {
      font-size: 16px;
      padding: 7px;
      width: 250px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    button {
      font-size: 18px;
      padding: 10px 20px;
      margin: 15px 10px 10px 10px;
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
      margin-top: 30px;
      font-size: 18px;
      color: #34495e;
      line-height: 1.5;
      max-width: 400px;
      margin-left: auto;
      margin-right: auto;
      text-align: left;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>

  <h1>TEMPO MÉDIO DE CARREGAMENTO SSP13</h1>

  <div id="contador">00:00:00</div>

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

  <div>
    <button id="iniciarBtn" disabled>Iniciar</button>
    <button id="pararBtn" disabled>Parar</button>
    <button id="resetarBtn" disabled>Resetar</button>
  </div>

  <div id="dadosInseridos"></div>

  <script>
    const URL_WEB_APP = "URL_DO_SEU_WEB_APP"; // coloque aqui sua URL do Apps Script

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

      const valido = placa !== "" && doca !== "" && transportadora !== "";
      iniciarBtn.disabled = !valido;
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
      document.body.style.backgroundColor = '';

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
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          placa,
          doca,
          transportadora,
          tempo
        })
      })
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
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

    // Inicialização
    contadorElement.textContent = formatarTempo(totalSegundos);
    document.body.style.backgroundColor = '';
    iniciarBtn.disabled = true;
    pararBtn.disabled = true;
    resetarBtn.disabled = true;
  </script>

</body>
</html>
