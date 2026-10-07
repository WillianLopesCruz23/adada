<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Painel IoT - Central de Iluminação</title>
  <script src="https://unpkg.com/mqtt/dist/mqtt.min.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    body { background-color: #e0f2fe; color: #1e293b; display: flex; justify-content: center; align-items: center; min-height: 100vh; padding: 20px; }
    .container { background: #ffffff; width: 100%; max-width: 480px; padding: 28px; border-radius: 20px; box-shadow: 0 15px 35px rgba(0,111,255,0.1); }
    h1 { font-size: 1.5rem; text-align: center; color: #0369a1; margin-bottom: 20px; }
    
    .pot-card { background: #f0f9ff; border: 2px solid #bae6fd; border-radius: 14px; padding: 20px; text-align: center; margin-bottom: 20px; }
    .pot-val { font-size: 2.8rem; font-weight: 800; color: #0284c7; margin: 5px 0; }
    .progress-bar { width: 100%; height: 12px; background: #e2e8f0; border-radius: 6px; overflow: hidden; margin-top: 10px; }
    .progress-fill { height: 100%; width: 0%; background: #0284c7; transition: width 0.2s ease; }

    .card { background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 12px; padding: 16px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; }
    .card-info { display: flex; align-items: center; gap: 12px; font-weight: 600; }
    
    .status-badge { padding: 6px 12px; border-radius: 20px; font-size: 0.85rem; font-weight: bold; background: #cbd5e1; color: #475569; transition: all 0.3s; }
    .status-badge.active { background: #22c55e; color: #ffffff; box-shadow: 0 0 10px rgba(34,197,94,0.4); }
  </style>
</head>
<body>

<div class="container">
  <h1>🌐 Iluminação Inteligente IoT</h1>

  <!-- Leitura do Potenciômetro A0 -->
  <div class="pot-card">
    <span style="font-weight: 600; color: #0369a1;">🎛️ Potenciômetro (Pino A0)</span>
    <div class="pot-val"><span id="pot-value">0</span>%</div>
    <div class="progress-bar">
      <div id="pot-bar" class="progress-fill"></div>
    </div>
  </div>

  <!-- LED 1: Sala (Pino 8) -->
  <div class="card">
    <div class="card-info">
      <span>🛋️ Sala (Pino 8)</span>
    </div>
    <div id="badge-sala" class="status-badge">Desligado</div>
  </div>

  <!-- LED 2: Quarto (Pino 10 - PWM) -->
  <div class="card">
    <div class="card-info">
      <span>🛏️ Quarto (Pino 10)</span>
    </div>
    <div id="badge-quarto" class="status-badge">0%</div>
  </div>

  <!-- LED 3: Cozinha (Pino 13) -->
  <div class="card">
    <div class="card-info">
      <span>🍳 Cozinha (Pino 13)</span>
    </div>
    <div id="badge-cozinha" class="status-badge">Desligado</div>
  </div>
</div>

<script>
  // 🚨 USE O MESMO NOME DE BANCADA DEFINIDO NO PYTHON
  const BANCADA = "bancada_automa_2026"; 

  // Conexão via WebSocket Seguro (wss) na porta 8884
  const client = mqtt.connect('wss://broker.hivemq.com:8884/mqtt', {
    clientId: 'web_dashboard_' + Math.random().toString(16).substring(2, 8),
    clean: true,
    connectTimeout: 4000
  });

  client.on('connect', () => {
    console.log('✅ Dashboard conectado ao Broker MQTT via WSS!');
    client.subscribe(`${BANCADA}/casa/#`);
  });

  client.on('error', (err) => {
    console.error('❌ Erro na Conexão MQTT:', err);
  });

  client.on('message', (topic, message) => {
    const valor = message.toString();

    if (topic === `${BANCADA}/casa/potenciometro`) {
      document.getElementById('pot-value').innerText = valor;
      document.getElementById('pot-bar').style.width = valor + '%';
    } 
    else if (topic === `${BANCADA}/casa/sala`) {
      const badge = document.getElementById('badge-sala');
      if (valor === "1") {
        badge.innerText = "Ligado";
        badge.className = "status-badge active";
      } else {
        badge.innerText = "Desligado";
        badge.className = "status-badge";
      }
    } 
    else if (topic === `${BANCADA}/casa/quarto`) {
      const badge = document.getElementById('badge-quarto');
      if (parseInt(valor) > 0) {
        badge.innerText = "Brilho: " + valor + "%";
        badge.className = "status-badge active";
      } else {
        badge.innerText = "Desligado";
        badge.className = "status-badge";
      }
    } 
    else if (topic === `${BANCADA}/casa/cozinha`) {
      const badge = document.getElementById('badge-cozinha');
      if (valor === "1") {
        badge.innerText = "Ligado";
        badge.className = "status-badge active";
      } else {
        badge.innerText = "Desligado";
        badge.className = "status-badge";
      }
    }
  });
</script>
</body>
</html># adada
