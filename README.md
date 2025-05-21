<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8" />
  <title>Contador e Dados de Entrada</title>
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
    }
  </style>
</head>
<body>

  <h1>Controle de Tempo e Cadastro</h1>

  <div>
    <label for="placa">Placa:</label><br/>
    <input type="text" id="placa" name="placa" placeholder="Digite a placa" />
  </div>
  <div>
    <label for="doca">Doca:</label><br/>
    <input type="text" id="doca" name="doca" placeholder="Digite a doca" />
  </div>
  <div>
    <label for="transportadora">Transportadora:</label><br/>
    <input type="text" id="transportadora" name="transportadora" placeholder="Digite a transportadora" />
  </div>

  <div>
    <button id="iniciarBtn" disabled>Iniciar</button>
    <button id="pararBtn" disabled>Parar</button>
    <button id="resetarBtn" disabled>Resetar</button>
  </div>

  <div id="dadosInseridos"></div>

  <div id="contador">00:00:00</div>

  <script>
    // Variáveis do contador
    let totalSegundos = 0;
    const maxSegundos = 40 * 60; // 40 minutos
    const contadorElement = document.getElementById('contador');
    const iniciarBtn = document.getElementById('iniciarBtn');
    const pararBtn = document.getElementById('pararBtn');
    const resetarBtn = document.getElementById('resetarBtn');

    // Inputs do formulário para validar
    const placaInput = document.getElementById('placa');
    const docaInput = document.getElementById('doca');
    const transportadoraInput = document.getElementById('transportadora');

    let intervalo = null;

    // Função para formatar segundos em HH:MM:SS
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

    // Atualiza cor de fundo conforme o tempo
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

    // Função que valida os campos e habilita/desabilita o botão Iniciar
    function validarCampos() {
      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();

      if (placa !== '' && doca !== '' && transportadora !== '') {
        iniciarBtn.disabled = false;
      } else {
        iniciarBtn.disabled = true;
      }
    }

    // Monitora mudanças nos inputs para habilitar o botão Iniciar
    placaInput.addEventListener('input', validarCampos);
    docaInput.addEventListener('input', validarCampos);
    transportadoraInput.addEventListener('input', validarCampos);

    iniciarBtn.addEventListener('click', () => {
      if (intervalo) return; // não permite iniciar duas vezes seguidas

      // Exibe os dados inseridos
      const placa = placaInput.value.trim();
      const doca = docaInput.value.trim();
      const transportadora = transportadoraInput.value.trim();

      const dadosInseridos = document.getElementById('dadosInseridos');
      dadosInseridos.innerHTML = `
        <h3>Dados Iniciados:</h3>
        <p><strong>Placa:</strong> ${placa}</p>
        <p><strong>Doca:</strong> ${doca}</p>
        <p><strong>Transportadora:</strong> ${transportadora}</p>
      `;

      // Desabilita inputs para evitar alterações durante o contador
      placaInput.disabled = true;
      docaInput.disabled = true;
      transportadoraInput.disabled = true;

      // Desabilita o botão iniciar e habilita parar e resetar
      iniciarBtn.disabled = true;
      pararBtn.disabled = false;
      resetarBtn.disabled = false;

      // Inicia o contador
      intervalo = setInterval(atualizarContador, 1000);
    });

    pararBtn.addEventListener('click', () => {
      if (intervalo) {
        clearInterval(intervalo);
        intervalo = null;
        iniciarBtn.disabled = false;  // Permite continuar
        pararBtn.disabled = true;
      }
    });

    resetarBtn.addEventListener('click', () => {
      if (intervalo) {
        clearInterval(intervalo);
        intervalo = null;
      }
      totalSegundos = 0;
      contadorElement.textContent = formatarTempo(totalSegundos);
      document.body.style.backgroundColor = '';

      // Limpa dados exibidos
      document.getElementById('dadosInseridos').innerHTML = '';

      // Limpa e habilita inputs
      placaInput.value = '';
      docaInput.value = '';
      transportadoraInput.value = '';
      placaInput.disabled = false;
      docaInput.disabled = false;
      transportadoraInput.disabled = false;

      // Ajusta botões
      iniciarBtn.disabled = true;
      pararBtn.disabled = true;
      resetarBtn.disabled = true;
    });

    // Inicializa estado dos botões e contador
    contadorElement.textContent = formatarTempo(totalSegundos);
    document.body.style.backgroundColor = '';
    iniciarBtn.disabled = true;
    pararBtn.disabled = true;
    resetarBtn.disabled = true;
  </script>

</body>
</html>
