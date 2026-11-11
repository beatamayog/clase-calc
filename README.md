[gemini-code-1794329164619.html](https://github.com/user-attachments/files/34694086/gemini-code-1794329164619.html)
# clase-calc
IA que analiza un proyecto de Dipolos en 2D
<!DOCTYPE html>
<html lang="es" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulación y Análisis de Dipolo 2D</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        darkBg: '#050000', 
                        panelBg: 'rgba(15, 2, 2, 0.65)', 
                        accentRed: '#ff0033',
                        accentWhite: '#ffffff'
                    },
                    fontFamily: {
                        'display': ['"Space Grotesk"', 'sans-serif'],
                        'body': ['"Inter"', 'sans-serif'],
                        'mono': ['"JetBrains Mono"', 'monospace']
                    }
                }
            }
        }
    </script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500&family=JetBrains+Mono:wght@400;500&family=Space+Grotesk:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #030000;
            color: #f8fafc;
            margin: 0;
            overflow-x: hidden;
        }

        /* --- Canvas Espacial con Vectores Blancos --- */
        #dipoleCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 0; 
            opacity: 0.9; 
            pointer-events: none; 
        }

        /* --- Cuadrícula HUD --- */
        .bg-grid {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: 1;
            background-image: 
                linear-gradient(rgba(255, 255, 255, 0.03) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255, 255, 255, 0.03) 1px, transparent 1px);
            background-size: 40px 40px;
            pointer-events: none;
        }
        
        /* Glassmorphism Negro/Rojo */
        .glass-panel {
            background: rgba(10, 2, 2, 0.55);
            backdrop-filter: blur(16px);
            -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 50, 50, 0.1);
            border-top: 1px solid rgba(255, 255, 255, 0.15);
            border-left: 1px solid rgba(255, 255, 255, 0.08);
            box-shadow: 0 20px 40px -10px rgba(0,0,0,1);
            border-radius: 24px;
        }

        /* Brillos ambientales rojos */
        .bg-glow {
            position: fixed; 
            border-radius: 50%;
            filter: blur(150px);
            opacity: 0.3;
            z-index: 1; 
            pointer-events: none;
            animation: pulseGlow 12s infinite alternate ease-in-out;
        }
        
        @keyframes pulseGlow {
            0% { transform: scale(0.9) translate(0, 0); opacity: 0.2; }
            100% { transform: scale(1.2) translate(20px, -20px); opacity: 0.4; }
        }

        /* Nodos del Workflow */
        .agent-card { 
            transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1);
            background: rgba(5, 0, 0, 0.7);
            border: 1px solid rgba(255, 255, 255, 0.05);
        }
        
        .agent-processing {
            transform: translateY(-8px) scale(1.02);
            background: rgba(30, 5, 5, 0.9);
        }
        
        /* Tonos rojos y blancos para cuando se activan */
        .agent-processing.red-glow { border-color: #ff0033; box-shadow: 0 0 25px -5px rgba(255, 0, 51, 0.4); }
        .agent-processing.white-glow { border-color: #ffffff; box-shadow: 0 0 25px -5px rgba(255, 255, 255, 0.4); }

        .main-ui { position: relative; z-index: 10; }
    </style>
</head>
<body class="min-h-screen relative overflow-x-hidden flex flex-col">

    <!-- CANVAS ESPACIAL Y CUADRÍCULA -->
    <canvas id="dipoleCanvas"></canvas>
    <div class="bg-grid"></div>

    <!-- Luces de fondo (Rojo profundo y Carmesí) -->
    <div class="bg-glow w-[600px] h-[600px] bg-red-700 top-[-150px] left-[-150px]"></div>
    <div class="bg-glow w-[700px] h-[700px] bg-red-900 bottom-[-200px] right-[-200px]" style="animation-delay: -5s;"></div>
    <div class="bg-glow w-[400px] h-[400px] bg-red-800 top-[30%] left-[40%]" style="animation-delay: -2s;"></div>

    <div class="container mx-auto px-4 py-8 flex-grow max-w-7xl main-ui">
        
        <!-- Header -->
        <header class="text-center mb-10">
            <div class="inline-block px-4 py-1.5 rounded-full border border-red-500/30 bg-red-500/10 backdrop-blur-md mb-4 text-xs font-mono text-red-400 tracking-widest uppercase shadow-[0_0_15px_rgba(255,0,0,0.3)]">
                <i class="fa-solid fa-satellite-dish mr-2"></i>Saptiva AI Network
            </div>
            <h1 class="text-4xl md:text-5xl font-display font-bold tracking-tight mb-2">
                <span class="text-white drop-shadow-[0_0_20px_rgba(255,255,255,0.4)]">
                    Simulación y Análisis de Dipolo 2D
                </span>
            </h1>
            <p class="text-xl font-display text-slate-400 font-light mt-2">Interfaz de Agentes Saptiva</p>
        </header>

        <!-- Main Layout -->
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
            
            <!-- PANEL IZQUIERDO: INPUT -->
            <div class="lg:col-span-5 w-full">
                <div class="glass-panel p-8 relative overflow-hidden group">
                    <div class="flex items-center space-x-4 mb-8 relative z-10">
                        <div class="w-12 h-12 rounded-xl bg-white/10 border border-white/20 flex items-center justify-center shadow-[0_0_15px_rgba(255,255,255,0.1)]">
                            <i class="fa-solid fa-terminal text-white text-xl"></i>
                        </div>
                        <div>
                            <h2 class="text-xl font-display font-bold text-white tracking-wide">Agente Triage</h2>
                            <p class="text-xs text-red-400 font-mono">ID: NODE-01 // ENTRADA</p>
                        </div>
                    </div>

                    <div class="relative z-10 space-y-6">
                        <div class="relative">
                            <textarea 
                                id="user-input" 
                                rows="7" 
                                class="w-full bg-[#030000]/80 border border-red-500/20 rounded-xl p-5 text-sm text-slate-200 placeholder-slate-600 focus:outline-none focus:border-red-500 focus:ring-1 focus:ring-red-500 transition-all font-mono resize-none shadow-inner"
                                placeholder="> INICIAR CONSULTA...&#10;> Ej: m = -6.28, alpha = 90. ¿Cómo afecta esto a la singularidad en el origen?"></textarea>
                            
                            <div class="absolute bottom-4 right-4 text-xs font-mono text-red-600">INPUT_READY</div>
                        </div>
                        
                        <button 
                            id="submit-btn"
                            onclick="startWorkflow()"
                            class="w-full py-4 rounded-xl bg-white hover:bg-red-50 text-black font-display font-bold text-sm tracking-widest uppercase shadow-[0_0_20px_rgba(255,255,255,0.4)] transition-all transform hover:-translate-y-1 flex items-center justify-center space-x-3 relative">
                            <span id="btn-text">Ejecutar Simulación</span>
                            <i id="btn-icon" class="fa-solid fa-arrow-right group-hover:translate-x-1 transition-transform"></i>
                            
                            <svg id="btn-spinner" class="animate-spin -ml-1 mr-2 h-5 w-5 text-black hidden absolute" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                            </svg>
                        </button>
                    </div>
                </div>
            </div>

            <!-- PANEL DERECHO: PIPELINE DE AGENTES -->
            <div class="lg:col-span-7 w-full flex flex-col items-center">
                
                <div class="w-full flex items-center justify-between mb-8 border-b border-red-500/20 pb-3">
                    <h3 class="text-xs font-display font-semibold text-slate-400 uppercase tracking-[0.2em]">Red de Procesamiento</h3>
                    <span class="text-[10px] font-mono text-red-500 flex items-center">
                        <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse mr-2"></span> SYSTEM_ONLINE
                    </span>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-3 gap-5 w-full relative">
                    <div class="hidden sm:block absolute top-[110%] left-1/2 w-[80%] h-[1px] bg-gradient-to-r from-transparent via-red-500/30 to-transparent -translate-x-1/2"></div>
                    
                    <!-- Nodos -->
                    <div id="agent-1" class="agent-card rounded-2xl p-6 flex flex-col items-center text-center opacity-80">
                        <i class="fa-solid fa-square-root-variable text-2xl text-slate-300 mb-4"></i>
                        <h4 class="font-display font-bold text-white text-sm">Física-Matemática</h4>
                        <div class="mt-4 px-3 py-1.5 rounded-md bg-white/5 border border-white/10 text-[9px] font-mono text-slate-400 flex items-center status-badge w-full justify-center">
                            <span class="w-1.5 h-1.5 rounded-full bg-slate-500 mr-2 status-dot"></span> <span class="status-text">STANDBY</span>
                        </div>
                    </div>

                    <div id="agent-2" class="agent-card rounded-2xl p-6 flex flex-col items-center text-center opacity-80">
                        <i class="fa-solid fa-code text-2xl text-slate-300 mb-4"></i>
                        <h4 class="font-display font-bold text-white text-sm">Ing. Software</h4>
                        <div class="mt-4 px-3 py-1.5 rounded-md bg-white/5 border border-white/10 text-[9px] font-mono text-slate-400 flex items-center status-badge w-full justify-center">
                            <span class="w-1.5 h-1.5 rounded-full bg-slate-500 mr-2 status-dot"></span> <span class="status-text">STANDBY</span>
                        </div>
                    </div>

                    <div id="agent-3" class="agent-card rounded-2xl p-6 flex flex-col items-center text-center opacity-80">
                        <i class="fa-solid fa-chart-network text-2xl text-slate-300 mb-4"></i>
                        <h4 class="font-display font-bold text-white text-sm">Análisis Dinámico</h4>
                        <div class="mt-4 px-3 py-1.5 rounded-md bg-white/5 border border-white/10 text-[9px] font-mono text-slate-400 flex items-center status-badge w-full justify-center">
                            <span class="w-1.5 h-1.5 rounded-full bg-slate-500 mr-2 status-dot"></span> <span class="status-text">STANDBY</span>
                        </div>
                    </div>
                </div>

                <!-- Nodo Final -->
                <div id="agent-4" class="agent-card mt-12 rounded-2xl p-6 flex flex-col items-center text-center opacity-80 w-full max-w-sm relative bg-[#050000]/80 backdrop-blur-md">
                    <div class="absolute -top-3 -right-3 w-8 h-8 bg-white text-black rounded-full flex items-center justify-center shadow-[0_0_20px_rgba(255,255,255,0.8)] opacity-0 transition-opacity" id="a4-badge">
                        <i class="fa-solid fa-check text-sm font-bold"></i>
                    </div>
                    <i class="fa-solid fa-chalkboard-user text-3xl text-white mb-4"></i>
                    <h4 class="font-display font-bold text-white text-lg">Coach de Pitch</h4>
                    <span class="text-xs text-red-400 font-mono mt-1">SÍNTESIS FINAL</span>
                    <div class="mt-5 px-5 py-2 rounded-md bg-white/10 border border-white/20 text-[10px] font-mono text-slate-300 flex items-center status-badge w-full justify-center">
                        <span class="w-2 h-2 rounded-full bg-slate-500 mr-2 status-dot"></span> <span class="status-text">ESPERANDO DATOS</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- OUTPUT SECTION (TERMINAL STYLE) -->
        <div id="output-section" class="mt-12 hidden opacity-0 transition-all duration-700 ease-in-out">
            <div class="glass-panel p-1 overflow-hidden relative border-t-red-500/30">
                <div class="relative bg-[#020000]/95 rounded-[20px] overflow-hidden border border-red-500/10 shadow-[0_0_30px_rgba(255,0,0,0.15)]">
                    <div class="bg-white/5 px-6 py-4 flex items-center justify-between border-b border-red-500/20">
                        <div class="flex gap-1.5">
                            <div class="w-3 h-3 rounded-full bg-[#ff5f56]"></div>
                            <div class="w-3 h-3 rounded-full bg-[#ffbd2e]"></div>
                            <div class="w-3 h-3 rounded-full bg-[#27c93f]"></div>
                        </div>
                        <div class="text-xs font-mono text-slate-400 font-bold">bash ~ saptiva-output</div>
                    </div>
                    <div class="p-8">
                        <div id="final-output-text" class="font-mono text-sm leading-relaxed text-slate-300 space-y-4">
                            <!-- El texto se inyecta aquí -->
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- JavaScript -->
    <script>
        // --- 1. LÓGICA DE LOS VECTORES BLANCOS (Fondo Negro/Rojo) ---
        const canvas = document.getElementById('dipoleCanvas');
        const ctx = canvas.getContext('2d');
        let width, height;
        let vectors = [];

        function resize() {
            width = canvas.width = window.innerWidth;
            height = canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        class VectorFieldParticle {
            constructor() { this.reset(); }
            reset() {
                this.x = Math.random() * width;
                this.y = Math.random() * height;
                this.life = Math.random() * 150 + 50;
                this.maxLife = this.life;
                this.speed = Math.random() * 0.4 + 0.15; 
            }
            update() {
                const dx = this.x - (width / 2);
                const dy = this.y - (height / 2);
                const r2 = dx*dx + dy*dy || 1;
                
                const vx = (dx*dx - dy*dy) / (r2*r2) * 50000;
                const vy = (2*dx*dy) / (r2*r2) * 50000;

                this.vx = vx * this.speed;
                this.vy = vy * this.speed;
                this.x += this.vx;
                this.y += this.vy;
                this.life--;

                if (this.life <= 0 || this.x < 0 || this.x > width || this.y < 0 || this.y > height) {
                    this.reset();
                }
            }
            draw() {
                const alpha = (this.life / this.maxLife) * 0.9; 
                const angle = Math.atan2(this.vy, this.vx);
                const length = 18; 
                
                const endX = this.x + Math.cos(angle) * length;
                const endY = this.y + Math.sin(angle) * length;

                ctx.beginPath();
                ctx.moveTo(this.x, this.y);
                ctx.lineTo(endX, endY);
                
                ctx.strokeStyle = `rgba(255, 255, 255, ${alpha})`;
                ctx.lineWidth = 1.5;
                ctx.shadowBlur = 8;
                ctx.shadowColor = '#ffffff';
                ctx.stroke();
            }
        }

        for (let i = 0; i < 400; i++) { vectors.push(new VectorFieldParticle()); }

        function animate() {
            ctx.fillStyle = 'rgba(5, 0, 0, 0.2)'; 
            ctx.fillRect(0, 0, width, height);
            vectors.forEach(v => { v.update(); v.draw(); });
            requestAnimationFrame(animate);
        }
        animate(); 

        // --- 2. LÓGICA DE INTERFAZ Y CONEXIÓN REAL ---
        const delay = ms => new Promise(res => setTimeout(res, ms));

        async function startWorkflow() {
            const input = document.getElementById('user-input');
            const btn = document.getElementById('submit-btn');
            const btnText = document.getElementById('btn-text');
            const btnIcon = document.getElementById('btn-icon');
            const btnSpinner = document.getElementById('btn-spinner');
            const outputSection = document.getElementById('output-section');

            if (input.value.trim() === '') {
                input.classList.add('border-red-500', 'shadow-[0_0_15px_rgba(255,0,0,0.5)]');
                setTimeout(() => input.classList.remove('border-red-500', 'shadow-[0_0_15px_rgba(255,0,0,0.5)]'), 1000);
                return;
            }

            input.disabled = true;
            btn.disabled = true;
            btnText.classList.add('hidden');
            btnIcon.classList.add('hidden');
            btnSpinner.classList.remove('hidden');
            
            outputSection.classList.add('hidden', 'opacity-0');

            await delay(500);
            activateAgent('agent-1', 'white-glow', 'CALCULANDO...', 'text-white', 'bg-white');
            await delay(300);
            activateAgent('agent-2', 'red-glow', 'ANALIZANDO...', 'text-red-400', 'bg-red-500');
            await delay(400);
            activateAgent('agent-3', 'white-glow', 'GENERANDO...', 'text-white', 'bg-white');

            // ⚠️ === CONEXIÓN REAL AL WEBHOOK === ⚠️
            let textoReal = "";
            try {
                // RECUERDA CAMBIAR ESTA URL POR LA TUYA
                const webhookURL = "https://api.saptiva.com/webhook/TU_ID_AQUI"; 
                
                const respuesta = await fetch(webhookURL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ mensaje: input.value }) 
                });

                const datos = await respuesta.json();
                textoReal = datos.respuesta || datos.texto || JSON.stringify(datos, null, 2);
            } catch (error) {
                console.error("Error de conexión:", error);
                textoReal = "🚨 Error de red. Revisa la consola o asegúrate de haber colocado la URL correcta del Webhook en el código.";
            }

            completeAgent('agent-1', 'text-white');
            completeAgent('agent-2', 'text-white');
            completeAgent('agent-3', 'text-white');

            await delay(500);
            activateAgent('agent-4', 'red-glow', 'SINTETIZANDO...', 'text-red-400', 'bg-red-500');
            
            await delay(1000);
            completeAgent('agent-4', 'text-white');
            document.getElementById('a4-badge').classList.remove('opacity-0');

            document.getElementById('final-output-text').innerHTML = `
                <div class="text-red-400 font-bold mb-4">root@saptiva-coach:~$ ./generar_pitch.sh</div>
                <pre class="whitespace-pre-wrap font-mono text-sm">${textoReal}</pre>
                <div class="mt-8 text-slate-600">========================================= <span class="animate-pulse text-white inline-block w-2 h-4 bg-white ml-2 align-middle"></span></div>
            `;

            outputSection.classList.remove('hidden');
            void outputSection.offsetWidth;
            outputSection.classList.remove('opacity-0');

            btnSpinner.classList.add('hidden');
            btnText.classList.remove('hidden');
            btnIcon.classList.remove('hidden');
            btnText.innerText = "NUEVA SIMULACIÓN";
            btn.disabled = false;
            input.disabled = false;

            outputSection.scrollIntoView({ behavior: 'smooth', block: 'start' });
        }

        function activateAgent(id, glowClass, text, textColor, dotColor) {
            const card = document.getElementById(id);
            card.classList.remove('opacity-80', 'border-slate-700');
            card.classList.add('agent-processing', glowClass);
            
            const badge = card.querySelector('.status-badge');
            badge.classList.remove('bg-white/5', 'border-white/10', 'text-slate-400');
            badge.classList.add('bg-black/60', 'border-red-500/30', textColor);
            
            const dot = card.querySelector('.status-dot');
            dot.className = `w-1.5 h-1.5 rounded-full mr-2 status-dot animate-pulse ${dotColor}`;
            
            card.querySelector('.status-text').innerText = text;
        }

        function completeAgent(id, textColor) {
            const card = document.getElementById(id);
            card.querySelector('.status-dot').classList.remove('animate-pulse');
            card.querySelector('.status-text').innerText = "COMPLETADO";
            card.querySelector('.status-text').classList.add(textColor);
        }
    </script>
</body>
</html>
