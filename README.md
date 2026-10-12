
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema Autónomo de Guiagem DSMAC/ATR - VANT & Arduino</title>
    <style>
        :root {
            --bg-color: #0b0f19;
            --panel-bg: #111827;
            --accent-green: #10b981;
            --accent-amber: #f59e0b;
            --accent-red: #ef4444;
            --text-main: #f3f4f6;
            --text-muted: #9ca3af;
            --border-color: #1f2937;
        }
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Courier New', Courier, monospace;
        }
        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            padding: 15px;
        }
        header {
            background: var(--panel-bg);
            border: 1px solid var(--border-color);
            padding: 15px 20px;
            margin-bottom: 15px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-radius: 6px;
        }
        header h1 {
            font-size: 1.15rem;
            color: var(--accent-green);
            letter-spacing: 1px;
        }
        .status-badge {
            background: rgba(16, 185, 129, 0.1);
            color: var(--accent-green);
            border: 1px solid var(--accent-green);
            padding: 6px 12px;
            border-radius: 4px;
            font-size: 0.75rem;
            font-weight: bold;
        }
        .main-container {
            display: grid;
            grid-template-columns: 1fr 400px;
            gap: 15px;
            flex: 1;
        }
        @media(max-width: 1024px) {
            .main-container {
                grid-template-columns: 1fr;
            }
        }
        .panel {
            background: var(--panel-bg);
            border: 1px solid var(--border-color);
            border-radius: 6px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        .panel-title {
            font-size: 0.9rem;
            color: var(--accent-amber);
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 8px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        .canvas-wrapper {
            position: relative;
            background: #000;
            border: 1px solid var(--border-color);
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            border-radius: 4px;
            flex: 1;
            min-height: 420px;
        }
        canvas {
            display: block;
            background: #05070a;
            cursor: crosshair;
        }
        .controls-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
        }
        button {
            background: #1f2937;
            color: var(--text-main);
            border: 1px solid var(--border-color);
            padding: 9px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.8rem;
            transition: all 0.2s;
        }
        button:hover {
            background: #374151;
            border-color: var(--accent-green);
            color: var(--accent-green);
        }
        button.active {
            background: rgba(16, 185, 129, 0.2);
            border-color: var(--accent-green);
            color: var(--accent-green);
        }
        .telemetry-box {
            background: #070a0f;
            border: 1px solid var(--border-color);
            padding: 10px;
            border-radius: 4px;
            font-size: 0.8rem;
            display: flex;
            flex-direction: column;
            gap: 6px;
        }
        .telemetry-row {
            display: flex;
            justify-content: space-between;
        }
        .serial-console {
            background: #030406;
            color: var(--accent-green);
            border: 1px solid var(--border-color);
            padding: 8px;
            border-radius: 4px;
            font-size: 0.72rem;
            height: 110px;
            overflow-y: auto;
            white-space: pre-wrap;
        }
        .report-section {
            font-size: 0.78rem;
            line-height: 1.35;
            color: var(--text-muted);
            background: #0a0e14;
            padding: 10px;
            border-radius: 4px;
            border-left: 3px solid var(--accent-amber);
        }
        .report-section h3 {
            color: var(--text-main);
            font-size: 0.82rem;
            margin-bottom: 4px;
        }
    </style>
</head>
<body>

    <header>
        <div>
            <h1>SISTEMA AUTÓNOMO DE GUIAGEM DSMAC / ATR</h1>
            <p style="font-size: 0.72rem; color: var(--text-muted);">OpenCV CSRT Tracker (Python) + Arduino Uno Servo Matrix (C++) Unificados</p>
        </div>
        <div class="status-badge" id="system-status">MODO: DSMAC AUTÔNOMO [ATIVO]</div>
    </header>

    <div class="main-container">
        <!-- Coluna Esquerda: Visão da Câmera IA & Canvas 2D -->
        <div class="panel">
            <div class="panel-title">1. Camada de Decisão e Percepção (IA - Óptico / Térmico / DSMAC)</div>
            <div class="canvas-wrapper">
                <canvas id="simCanvas" width="640" height="440"></canvas>
            </div>
            <div class="controls-grid">
                <button id="btnLock" class="active">Lock Tracker (CSRT)</button>
                <button id="btnThermal">Alternar Visão Térmica/IR</button>
                <button id="btnAutonomous">Modo Anti-Intercepção (Sem RTH)</button>
                <button id="btnReset">Reposicionar Alvo</button>
            </div>
            <p style="font-size: 0.72rem; color: var(--text-muted);">* Dica: Clique em qualquer ponto do radar/canvas para designar um novo alvo móvel instantaneamente.</p>
        </div>

        <!-- Coluna Direita: Relatório Técnico & Camada de Atuação Física -->
        <div class="panel">
            <div class="panel-title">Relatório Técnico & Memorial Descritivo</div>
            <div class="report-section">
                <h3>2.1 Sumário Executivo do Sistema</h3>
                <p>Elimina a dependência de reabertura de canais de comunicação ou rotinas previsíveis de Retorno para Casa (RTH), prevenindo interceptações eletrônicas e perda de sinal após o engajamento.</p>
            </div>
            <div class="report-section">
                <h3>2.2 Arquitetura Lógica e Mecânica</h3>
                <p><strong>Camada 1 (IA):</strong> Processamento digital 3D/óptico e térmico em telescópio rastreador.<br><strong>Camada 2 (Arduino Uno):</strong> Atuação física via servos mestre-escravo (`map` de -320/320 para 45°/135°).</p>
            </div>

            <div class="panel-title" style="margin-top: 2px;">Telemetria Física (Arduino Uno - 115200 baud)</div>
            <div class="telemetry-box">
                <div class="telemetry-row"><span>Vetor Erro X:</span> <span id="valErroX" style="color:var(--accent-amber)">0 px</span></div>
                <div class="telemetry-row"><span>Vetor Erro Y:</span> <span id="valErroY" style="color:var(--accent-amber)">0 px</span></div>
                <div class="telemetry-row"><span>Servo Eixo X (Pin 9):</span> <span id="valServoX" style="color:var(--accent-green)">90°</span></div>
                <div class="telemetry-row"><span>Servo Eixo Y (Pin 10):</span> <span id="valServoY" style="color:var(--accent-green)">90°</span></div>
            </div>

            <div class="panel-title">Barramento Serial Local</div>
            <div class="serial-console" id="serialConsole">[SYSTEM] Serial.begin(115200) conectado com sucesso.
[AI] LocalTargetTracker (OpenCV CSRT) pronto.
[DSMAC] Guiagem autônoma por correlação de área ativa.
[ARDUINO] Servo Eixo X e Y acoplados nos pinos 9 e 10.</div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');

        // Estados do Sistema
        let isThermal = false;
        let isAutonomous = true;
        let targetLocked = true;

        // Alvo móvel simulado no espaço
        let target = {
            x: 320,
            y: 220,
            vx: 1.8,
            vy: 1.2,
            w: 48,
            h: 38
        };

        // Bounding Box do Rastreador CSRT simulado
        let bbox = { x: target.x - 24, y: target.y - 19, w: 48, h: 38 };
        
        // Posições dos Servos (45° a 135°, centro em 90°)
        let servoPosX = 90;
        let servoPosY = 90;

        // Elementos da UI
        const btnLock = document.getElementById('btnLock');
        const btnThermal = document.getElementById('btnThermal');
        const btnAutonomous = document.getElementById('btnAutonomous');
        const btnReset = document.getElementById('btnReset');
        const valErroX = document.getElementById('valErroX');
        const valErroY = document.getElementById('valErroY');
        const valServoX = document.getElementById('valServoX');
        const valServoY = document.getElementById('valServoY');
        const serialConsole = document.getElementById('serialConsole');
        const systemStatus = document.getElementById('system-status');

        function logSerial(msg) {
            serialConsole.textContent += "\n" + msg;
            serialConsole.scrollTop = serialConsole.scrollHeight;
        }

        btnThermal.addEventListener('click', () => {
            isThermal = !isThermal;
            btnThermal.classList.toggle('active', isThermal);
            logSerial(isThermal ? "[AI] Sensor acoplado: Espectro Térmico / Infravermelho ativo." : "[AI] Sensor acoplado: Visão Óptica RGB padrão.");
        });

        btnLock.addEventListener('click', () => {
            targetLocked = !targetLocked;
            btnLock.classList.toggle('active', targetLocked);
            logSerial(targetLocked ? "[AI] lock_target(): Assinatura digital tridimensional indexada." : "[AI] Alvo destravado pelo operador.");
        });

        btnAutonomous.addEventListener('click', () => {
            isAutonomous = !isAutonomous;
            btnAutonomous.classList.toggle('active', isAutonomous);
            systemStatus.textContent = isAutonomous ? "MODO: DSMAC AUTÔNOMO [ATIVO]" : "MODO: MANUAL [RTH BLOQUEADO]";
            logSerial(isAutonomous ? "[DSMAC] Guiagem autônoma por contraste e correlação de área ativa." : "[SYSTEM] Alerta: Modo manual sem rotina vulnerável de Retorno para Casa.");
        });

        btnReset.addEventListener('click', () => {
            target.x = 120 + Math.random() * 400;
            target.y = 100 + Math.random() * 240;
            target.vx = (Math.random() - 0.5) * 3.5;
            target.vy = (Math.random() - 0.5) * 3.5;
            logSerial("[AI] Nova varredura mecânica: Alvo reposicionado no espaço.");
        });

        // Clique para designar alvo no Canvas
        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            target.x = e.clientX - rect.left;
            target.y = e.clientY - rect.top;
            targetLocked = true;
            btnLock.classList.add('active');
            logSerial(`[AI] Novo alvo designado por operador: X=${Math.round(target.x)}, Y=${Math.round(target.y)}`);
        });

        function update() {
            // Movimentação dinâmica do alvo
            target.x += target.vx;
            target.y += target.vy;

            // Colisão com as bordas da área de rastreamento
            if (target.x < 40 || target.x > canvas.width - 40) target.vx *= -1;
            if (target.y < 40 || target.y > canvas.height - 40) target.vy *= -1;

            if (targetLocked) {
                // Simulação da lógica update_tracking do OpenCV CSRT
                bbox.x += ((target.x - target.w / 2) - bbox.x) * 0.25;
                bbox.y += ((target.y - target.h / 2) - bbox.y) * 0.25;

                const centro_x = Math.round(bbox.x + bbox.w / 2);
                const centro_y = Math.round(bbox.y + bbox.h / 2);

                // Cálculo do vetor de erro espacial centralizado local (Centro da tela = 320, 220)
                const erro_x = centro_x - 320;
                const erro_y = centro_y - 220;

                // Simulação do código C++ Arduino Uno (map function)
                // int posX = map(erro_x, -320, 320, 45, 135);
                // int posY = map(erro_y, -220, 220, 45, 135);
                servoPosX = Math.round(45 + (erro_x + 320) * (90 / 640));
                servoPosY = Math.round(45 + (erro_y + 220) * (90 / 440));

                servoPosX = Math.max(45, Math.min(135, servoPosX));
                servoPosY = Math.max(45, Math.min(135, servoPosY));

                // Atualização dos indicadores na interface
                valErroX.textContent = `${erro_x > 0 ? '+' : ''}${erro_x} px`;
                valErroY.textContent = `${erro_y > 0 ? '+' : ''}${erro_y} px`;
                valServoX.textContent = `${servoPosX}°`;
                valServoY.textContent = `${servoPosY}°`;

                if (Math.random() < 0.12) {
                    logSerial(`[ARDUINO Serial] erro_x=${erro_x} erro_y=${erro_y} -> servoX=${servoPosX}° servoY=${servoPosY}°`);
                }
            } else {
                valErroX.textContent = `0 px (LIVRE)`;
                valErroY.textContent = `0 px (LIVRE)`;
            }
        }

        function draw() {
            // Renderização do fundo (Visualizador Óptico / Térmico)
            if (isThermal) {
                ctx.fillStyle = '#180000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            } else {
                ctx.fillStyle = '#05070a';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // Grid tático e linhas de referência do telescópio
                ctx.strokeStyle = '#1f2937';
                ctx.lineWidth = 1;
                for (let x = 0; x < canvas.width; x += 40) {
                    ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
                }
                for (let y = 0; y < canvas.height; y += 40) {
                    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
                }
            }

            // Cruz central de mira fixa (Eixos ópticos)
            ctx.strokeStyle = isAutonomous ? 'rgba(16, 185, 129, 0.45)' : 'rgba(245, 158, 11, 0.45)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(320, 0); ctx.lineTo(320, 440);
            ctx.moveTo(0, 220); ctx.lineTo(640, 220);
            ctx.stroke();

            // Desenho do alvo móvel
            ctx.fillStyle = isThermal ? '#ff4500' : '#3b82f6';
            ctx.beginPath();
            ctx.arc(target.x, target.y, target.w / 2, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#ffffff';
            ctx.font = '9px Courier New';
            ctx.fillText("ATR TARGET", target.x - 26, target.y - 24);

            // Desenho do Bounding Box do OpenCV CSRT Tracker
            if (targetLocked) {
                ctx.strokeStyle = '#10b981';
                ctx.lineWidth = 2;
                ctx.strokeRect(bbox.x, bbox.y, bbox.w, bbox.h);

                // Cantos de engajamento do rastreador
                const len = 8;
                ctx.strokeStyle = '#f59e0b';
                ctx.lineWidth = 3;
                // Top-Left
                ctx.beginPath(); ctx.moveTo(bbox.x, bbox.y + len); ctx.lineTo(bbox.x, bbox.y); ctx.lineTo(bbox.x + len, bbox.y); ctx.stroke();
                // Top-Right
                ctx.beginPath(); ctx.moveTo(bbox.x + bbox.w - len, bbox.y); ctx.lineTo(bbox.x + bbox.w, bbox.y); ctx.lineTo(bbox.x + bbox.w, bbox.y + len); ctx.stroke();
                // Bottom-Left
                ctx.beginPath(); ctx.moveTo(bbox.x, bbox.y + bbox.h - len); ctx.lineTo(bbox.x, bbox.y + bbox.h); ctx.lineTo(bbox.x + len, bbox.y + bbox.h); ctx.stroke();
                // Bottom-Right
                ctx.beginPath(); ctx.moveTo(bbox.x + bbox.w - len, bbox.y + bbox.h); ctx.lineTo(bbox.x + bbox.w, bbox.y + bbox.h); ctx.lineTo(bbox.x + bbox.w, bbox.y + bbox.h - len); ctx.stroke();

                // Vetor de erro espacial centralizado
                ctx.strokeStyle = 'rgba(245, 158, 11, 0.65)';
                ctx.setLineDash([3, 3]);
                ctx.beginPath();
                ctx.moveTo(320, 220);
                ctx.lineTo(bbox.x + bbox.w / 2, bbox.y + bbox.h / 2);
                ctx.stroke();
                ctx.setLineDash([]);
            }

            // HUD Overlays de Telemetria Visual
            ctx.fillStyle = isAutonomous ? '#10b981' : '#f59e0b';
            ctx.font = '10px Courier New';
            ctx.fillText(`DSMAC: ${isAutonomous ? 'ENGAGED (ATR ACTIVE)' : 'STANDBY'}`, 12, 22);
            ctx.fillText(`TRACKER: OpenCV CSRT | EIXO X: ${Math.round(bbox.x + bbox.w/2 - 320)}`, 12, 38);
        }

        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        requestAnimationFrame(loop);
    </script>
</body>
</html>
