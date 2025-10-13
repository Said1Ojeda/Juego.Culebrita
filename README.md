# Juego.Culebrita
JUGAR/culebra
[Culebra.html](https://github.com/user-attachments/files/22890583/Culebra.html)
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake: Obra de Arte Bioluminiscente</title>
    <!-- Carga de fuente moderna para el HUD -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=Playfair+Display:ital@1&display=swap" rel="stylesheet">
    <style>
        /* =========================================================
        ESTILOS CSS: ESTÉTICA INMERSIVA Y RESPONSIVE (TAILWIND SIMULADO)
        =========================================================
        */
        :root {
            --bg-dark: #020205; /* Fondo más oscuro */
            --snake-head: #00ffff; /* Cian ultra-brillante */
            --snake-body: #0099cc; /* Azul eléctrico */
            --food-color-standard: #ffaa00; /* Oro pulsante */
            --food-color-speed: #39ff14; /* Verde neón para velocidad */
            --board-glow: #101018;
            --font-main: 'Inter', sans-serif;
            --font-display: 'Playfair Display', serif;
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-color: var(--bg-dark);
            color: white;
            font-family: var(--font-main);
            overflow: hidden;
            padding: 1rem;
        }

        /* Contenedor principal para centrar y aplicar sombra de inmersión */
        .game-container {
            position: relative;
            /* Resplandor más intenso para un efecto de luz profunda */
            box-shadow: 0 0 60px rgba(0, 153, 204, 0.6); 
            border-radius: 1.5rem;
            overflow: hidden;
            max-width: 90vw;
            width: 800px; 
            aspect-ratio: 1 / 1;
            background-color: var(--board-glow);
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
            /* Sombra interior para simular profundidad o un borde elevado */
            box-shadow: inset 0 0 40px rgba(0, 0, 0, 0.9);
            border-radius: 1.5rem;
        }

        /* HUD: Puntuación y Mensajes */
        .hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 1.5rem;
            box-sizing: border-box;
            pointer-events: none;
            text-align: right;
            text-shadow: 0 0 10px rgba(0, 255, 255, 0.6); /* Sombra de texto cian */
        }

        #score {
            font-size: 1.8rem;
            font-weight: 700;
            color: var(--snake-head);
            opacity: 1;
        }

        /* Mensaje de Game Over */
        #message-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(2, 2, 5, 0.95); /* Fondo casi negro */
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.5s ease;
            text-align: center;
            border-radius: 1.5rem;
            z-index: 10;
        }

        #message-overlay.active {
            opacity: 1;
            visibility: visible;
        }

        .game-over-text {
            font-family: var(--font-display);
            font-size: 4.5rem;
            font-style: italic;
            color: #ff6666; 
            text-shadow: 0 0 25px #ff4444, 0 0 50px rgba(255, 68, 68, 0.7);
            margin-bottom: 1.5rem;
            animation: pulse 1.5s infinite alternate;
        }

        @keyframes pulse {
            from { transform: scale(1); opacity: 0.8; }
            to { transform: scale(1.05); opacity: 1; }
        }

        .restart-text {
            font-size: 1.4rem;
            color: var(--snake-head);
            padding: 0.75rem 1.5rem;
            border: 2px solid var(--snake-head);
            border-radius: 0.75rem;
            cursor: pointer;
            transition: background 0.3s, color 0.3s, box-shadow 0.3s;
            box-shadow: 0 0 10px rgba(0, 255, 255, 0.5);
        }

        .restart-text:hover {
            background-color: var(--snake-head);
            color: var(--bg-dark);
            box-shadow: 0 0 20px var(--snake-head);
        }

        /* Estilos responsivos */
        @media (max-width: 600px) {
            .game-over-text {
                font-size: 3rem;
            }
            .restart-text {
                font-size: 1.1rem;
            }
            .hud {
                padding: 1rem;
            }
            #score {
                font-size: 1.5rem;
            }
        }
    </style>
</head>
<body>

    <div class="game-container">
        <canvas id="gameCanvas"></canvas>
        <div class="hud">
            Puntuación: <span id="score">0</span>
        </div>
        <div id="message-overlay">
            <div class="game-over-text">JAJAJJAJAJ que bot</div>
            <div id="restart-button" class="restart-text">Presiona ESPACIO o haz clic para Reiniciar</div>
        </div>
    </div>

    <script>
        /* =========================================================
        JAVASCRIPT AVANZADO: LÓGICA DE JUEGO Y RENDERIZADO VISUAL
        =========================================================
        */
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const messageOverlay = document.getElementById('message-overlay');
        const restartButton = document.getElementById('restart-button');
        
        // Colores obtenidos directamente del CSS para ser usados en Canvas.
        const ROOT_STYLES = getComputedStyle(document.documentElement);
        const FOOD_COLOR_STANDARD = ROOT_STYLES.getPropertyValue('--food-color-standard').trim(); // '#ffaa00'
        const FOOD_COLOR_SPEED = ROOT_STYLES.getPropertyValue('--food-color-speed').trim();   // '#39ff14'
        const SNAKE_HEAD_COLOR = ROOT_STYLES.getPropertyValue('--snake-head').trim(); 
        const SNAKE_BODY_COLOR = ROOT_STYLES.getPropertyValue('--snake-body').trim();

        // --- CONSTANTES DEL JUEGO ---
        const GRID_SIZE = 25; 
        let TILE_SIZE;        

        // --- ESTADO DEL JUEGO ---
        let snake = [];
        let food = {}; // Ahora incluye 'type': 'standard' o 'speed'
        let dx = 1; 
        let dy = 0; 
        let score = 0;
        let isGameOver = false;
        let lastUpdateTime = 0;
        let baseSpeed = 150; // Milisegundos entre actualizaciones (velocidad inicial)
        let gameSpeed = baseSpeed; // Velocidad actual, afectada por power-ups
        let speedBoostDuration = 0; // Contador de frames para el boost de velocidad
        let frameCount = 0; 
        let soundContext;
        let pendingDirection = null; 
        
        // Sistema de Partículas para el rastro
        let particles = [];
        const MAX_PARTICLES = 100;

        // --- UTILIDADES DE INICIALIZACIÓN Y CONFIGURACIÓN ---

        function resizeCanvas() {
            const container = canvas.parentElement;
            const size = Math.min(container.clientWidth, container.clientHeight);
            canvas.width = size;
            canvas.height = size;
            
            TILE_SIZE = canvas.width / GRID_SIZE;
        }

        function initGame() {
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);
            document.addEventListener('keydown', handleKeyPress);
            restartButton.addEventListener('click', startGame);

            snake = [
                { x: Math.floor(GRID_SIZE / 2), y: Math.floor(GRID_SIZE / 2) },
                { x: Math.floor(GRID_SIZE / 2) - 1, y: Math.floor(GRID_SIZE / 2) }
            ];
            
            dx = 1; dy = 0;
            score = 0;
            isGameOver = false;
            lastUpdateTime = 0;
            frameCount = 0;
            pendingDirection = null;
            particles = [];
            gameSpeed = baseSpeed;
            speedBoostDuration = 0;

            scoreElement.textContent = score;
            messageOverlay.classList.remove('active');
            
            placeFood();
            
            requestAnimationFrame(gameLoop);
        }

        // --- SISTEMA DE SONIDO (WEB AUDIO API NATIVO) ---

        function initAudio() {
            try {
                if (!soundContext) {
                    soundContext = new (window.AudioContext || window.webkitAudioContext)();
                }
            } catch (e) {
                console.error('Web Audio API not supported. Sounds disabled.');
            }
        }

        // Genera sonido para la comida estándar (tono alto)
        function playEatSound() {
            if (!soundContext) return;
            try {
                const oscillator = soundContext.createOscillator();
                const gainNode = soundContext.createGain();

                oscillator.type = 'sine'; 
                oscillator.frequency.setValueAtTime(440, soundContext.currentTime);
                oscillator.frequency.linearRampToValueAtTime(880, soundContext.currentTime + 0.05); 
                
                gainNode.gain.setValueAtTime(0.5, soundContext.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.001, soundContext.currentTime + 0.15); 

                oscillator.connect(gainNode);
                gainNode.connect(soundContext.destination);
                
                oscillator.start();
                oscillator.stop(soundContext.currentTime + 0.15);
            } catch (e) {}
        }

        // Genera sonido para el boost de velocidad (tono bajo y vibrante)
        function playSpeedBoostSound() {
            if (!soundContext) return;
            try {
                const oscillator = soundContext.createOscillator();
                const gainNode = soundContext.createGain();

                oscillator.type = 'square'; // Onda cuadrada para un sonido más "digital"
                oscillator.frequency.setValueAtTime(200, soundContext.currentTime); 
                oscillator.frequency.linearRampToValueAtTime(400, soundContext.currentTime + 0.1); 
                
                gainNode.gain.setValueAtTime(0.7, soundContext.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.001, soundContext.currentTime + 0.3); 

                oscillator.connect(gainNode);
                gainNode.connect(soundContext.destination);
                
                oscillator.start();
                oscillator.stop(soundContext.currentTime + 0.3);
            } catch (e) {}
        }

        function playGameOverSound() {
            if (!soundContext) return;
            try {
                const oscillator = soundContext.createOscillator();
                const gainNode = soundContext.createGain();

                oscillator.type = 'sawtooth';
                oscillator.frequency.setValueAtTime(100, soundContext.currentTime);
                oscillator.frequency.linearRampToValueAtTime(50, soundContext.currentTime + 0.5);
                
                gainNode.gain.setValueAtTime(0.6, soundContext.currentTime);
                gainNode.gain.linearRampToValueAtTime(0, soundContext.currentTime + 1.0);

                oscillator.connect(gainNode);
                gainNode.connect(soundContext.destination);
                
                oscillator.start();
                oscillator.stop(soundContext.currentTime + 1.0);
            } catch (e) {}
        }

        // --- LÓGICA DE JUEGO ---

        function placeFood() {
            let newFood;
            // 20% de probabilidad de que sea un potenciador de velocidad
            const type = Math.random() < 0.2 ? 'speed' : 'standard'; 
            
            do {
                newFood = {
                    x: Math.floor(Math.random() * GRID_SIZE),
                    y: Math.floor(Math.random() * GRID_SIZE),
                    type: type // Asigna el tipo de comida
                };
            } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));
            
            food = newFood;
        }
        
        // Añade una partícula al rastro de la serpiente
        function addParticle(head) {
            const centerX = head.x * TILE_SIZE + TILE_SIZE / 2;
            const centerY = head.y * TILE_SIZE + TILE_SIZE / 2;
            
            // Posición inicial ligeramente aleatoria
            const x = centerX + (Math.random() - 0.5) * TILE_SIZE * 0.5;
            const y = centerY + (Math.random() - 0.5) * TILE_SIZE * 0.5;
            
            // Velocidad aleatoria en dirección opuesta al movimiento de la serpiente
            const vx = -(dx * 1.5) + (Math.random() - .5) * 0.3;
            const vy = -(dy * 1.5) + (Math.random() - 0.5) * 0.3;

            particles.push({
                x, y,
                vx: vx,
                vy: vy,
                life: 1.0, // Vida de 1.0 a 0.0
                color: SNAKE_HEAD_COLOR,
                size: TILE_SIZE * 0.15
            });

            // Limitar el número de partículas
            if (particles.length > MAX_PARTICLES) {
                particles.shift();
            }
        }
        
        // Actualiza la posición de las partículas
        function updateParticles() {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.life -= 0.02; // Desvanecimiento
                p.size *= 0.98; // Encogimiento

                if (p.life <= 0) {
                    particles.splice(i, 1);
                }
            }
        }

        function updateGame() {
            if (isGameOver) return;

            // Aplica la dirección pendiente
            if (pendingDirection) {
                dx = pendingDirection.dx;
                dy = pendingDirection.dy;
                pendingDirection = null;
            }

            // Mueve la cabeza
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            
            // Añade partículas en la posición anterior de la cabeza
            addParticle(snake[0]);

            // 1. Colisión con los bordes
            if (head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE) {
                return gameOver();
            }

            // 2. Colisión con su propio cuerpo
            if (snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)) {
                return gameOver();
            }

            // Añade la nueva cabeza
            snake.unshift(head);

            // 3. Comió la comida
            if (head.x === food.x && head.y === food.y) {
                if (food.type === 'standard') {
                    score++;
                    playEatSound();
                } else if (food.type === 'speed') {
                    score += 5; // Puntuación extra por power-up
                    speedBoostDuration = 100; // 100 frames de boost (aprox. 1.5 - 2 segundos)
                    playSpeedBoostSound();
                }

                scoreElement.textContent = score;
                placeFood();
            } else {
                // Si no come, elimina el último segmento para simular movimiento
                snake.pop();
            }
        }

        function gameOver() {
            isGameOver = true;
            playGameOverSound();
            messageOverlay.classList.add('active');
        }

        function startGame() {
            if (isGameOver) {
                initAudio(); // Asegura el contexto de audio antes de iniciar
                initGame();
            }
        }

        function handleKeyPress(event) {
            if (!soundContext) initAudio(); // Inicializa el audio al interactuar

            if (isGameOver) {
                if (event.code === 'Space' || event.code === 'Enter') {
                    startGame();
                }
                return;
            }

            let newDx = dx, newDy = dy;

            switch (event.key) {
                case 'ArrowUp':
                case 'w':
                    if (dy !== 1) { newDx = 0; newDy = -1; }
                    break;
                case 'ArrowDown':
                case 's':
                    if (dy !== -1) { newDx = 0; newDy = 1; }
                    break;
                case 'ArrowLeft':
                case 'a':
                    if (dx !== 1) { newDx = -1; newDy = 0; }
                    break;
                case 'ArrowRight':
                case 'd':
                    if (dx !== -1) { newDx = 1; newDy = 0; }
                    break;
                default:
                    return;
            }

            if (newDx !== dx || newDy !== dy) {
                pendingDirection = { dx: newDx, dy: newDy };
            }
        }

        // --- FUNCIONES DE RENDERIZADO (LA OBRA DE ARTE) ---

        function drawBoard() {
            ctx.fillStyle = 'var(--bg-dark)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.save();
            ctx.globalAlpha = 0.05; 

            // Patrón de cuadrícula bioluminiscente
            for (let i = 0; i < GRID_SIZE; i++) {
                for (let j = 0; j < GRID_SIZE; j++) {
                    const x = i * TILE_SIZE;
                    const y = j * TILE_SIZE;
                    
                    ctx.fillStyle = (i % 2 === j % 2) ? '#003344' : '#001a22';
                    ctx.fillRect(x, y, TILE_SIZE, TILE_SIZE);
                    
                    const cx = x + TILE_SIZE / 2;
                    const cy = y + TILE_SIZE / 2;

                    const gradient = ctx.createRadialGradient(cx, cy, 0, cx, cy, TILE_SIZE * 0.4);
                    gradient.addColorStop(0, `rgba(0, 200, 255, 0.2)`);
                    gradient.addColorStop(1, `rgba(0, 0, 0, 0)`);
                    ctx.fillStyle = gradient;
                    ctx.beginPath();
                    ctx.arc(cx, cy, TILE_SIZE * 0.4, 0, Math.PI * 2);
                    ctx.fill();
                }
            }
            ctx.restore();
        }

        // Dibuja un segmento de la serpiente con un efecto de esfera detallado
        function drawSegment(segment, index) {
            const x = segment.x * TILE_SIZE;
            const y = segment.y * TILE_SIZE;
            const radius = TILE_SIZE * 0.45;
            
            // Movimiento ondulado vertical para efecto de natación
            const wobble = Math.sin(index * 0.5 + frameCount * 0.15) * TILE_SIZE * 0.08;
            
            const baseColor = (index === 0) ? SNAKE_HEAD_COLOR : SNAKE_BODY_COLOR; 
            // La transparencia de la cola es más sutil
            const alpha = 1.0 - (index / snake.length) * 0.15; 
            
            ctx.save();
            
            const centerY = y + TILE_SIZE / 2 + wobble;
            const centerX = x + TILE_SIZE / 2;

            // --- FÍSICA AVANZADA: 3 BOLAS SOLAPADAS PARA DAR PROFUNDIDAD Y BRILLO ---
            
            // 1. Bola de Sombra/Profundidad
            ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
            ctx.beginPath();
            ctx.arc(centerX + TILE_SIZE * 0.05, centerY + TILE_SIZE * 0.05, radius, 0, Math.PI * 2);
            ctx.fill();

            // 2. Bola de Color Base (con Resplandor)
            ctx.shadowColor = baseColor;
            ctx.shadowBlur = TILE_SIZE * 0.5;
            ctx.shadowOffsetX = 0;
            ctx.shadowOffsetY = 0;
            
            const mainGradient = ctx.createRadialGradient(centerX, centerY, radius * 0.2, centerX, centerY, radius);
            mainGradient.addColorStop(0, 'white'); 
            mainGradient.addColorStop(0.5, baseColor);
            mainGradient.addColorStop(1, 'rgba(0, 0, 0, 0.8)');
            
            ctx.fillStyle = mainGradient;
            ctx.globalAlpha = alpha;
            ctx.beginPath();
            ctx.arc(centerX, centerY, radius * 0.95, 0, Math.PI * 2);
            ctx.fill();

            // 3. Resplandor Interior (Highlight)
            ctx.shadowBlur = 0;
            ctx.globalAlpha = 0.5;
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(centerX - radius * 0.3, centerY - radius * 0.3, radius * 0.2, 0, Math.PI * 2);
            ctx.fill();

            ctx.restore();
        }

        // Dibuja la comida estándar o la esfera de velocidad
        function drawFood() {
            const isSpeed = food.type === 'speed';
            const color = isSpeed ? FOOD_COLOR_SPEED : FOOD_COLOR_STANDARD;
            
            const x = food.x * TILE_SIZE + TILE_SIZE / 2;
            const y = food.y * TILE_SIZE + TILE_SIZE / 2;
            
            const baseRadius = TILE_SIZE * 0.3;
            // Pulso más visible para ambos, pero especialmente para el de velocidad
            const pulseIntensity = isSpeed ? 0.08 : 0.05;
            const pulse = Math.sin(frameCount * 0.3) * TILE_SIZE * pulseIntensity;
            const currentRadius = baseRadius + pulse;
            
            ctx.save();

            // --- EFECTO DE RESPLANDOR (GLOW) ---
            ctx.shadowColor = color;
            ctx.shadowBlur = TILE_SIZE * 1.0 + pulse * 4; // Resplandor más grande e intenso
            ctx.shadowOffsetX = 0;
            ctx.shadowOffsetY = 0;

            // --- JOYAS/ESFERA (Gradiente interno brillante) ---
            const innerGradient = ctx.createRadialGradient(x, y, 0, x, y, currentRadius);
            innerGradient.addColorStop(0, 'white'); 
            innerGradient.addColorStop(0.5, color);
            innerGradient.addColorStop(1, 'rgba(0, 0, 0, 0.8)');

            ctx.fillStyle = innerGradient;
            ctx.beginPath();
            ctx.arc(x, y, currentRadius, 0, Math.PI * 2);
            ctx.fill();
            
            // Añadir un símbolo sutil para el speed boost
            if (isSpeed) {
                ctx.restore();
                ctx.save();
                ctx.font = `${TILE_SIZE * 0.6}px Arial`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';
                ctx.fillText('⚡', x + 2, y + 2); // Sombra del rayo
                ctx.fillStyle = 'white';
                ctx.fillText('⚡', x, y); // Rayo
            }

            ctx.restore();
        }

        // Dibuja el rastro de partículas
        function drawParticles() {
            ctx.save();
            ctx.globalCompositeOperation = 'lighter'; // Hace que las partículas se sumen y brillen más

            for (const p of particles) {
                const alpha = p.life > 0.5 ? 1.0 : p.life * 2; // Brilla más al principio
                ctx.globalAlpha = alpha;
                
                ctx.fillStyle = p.color;
                ctx.shadowColor = p.color;
                ctx.shadowBlur = p.size * 2; // Resplandor de la partícula

                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
            }

            ctx.restore();
        }

        function drawSnake() {
            for (let i = 0; i < snake.length; i++) {
                drawSegment(snake[i], i);
            }
        }

        // Loop principal del juego
        function gameLoop(timestamp) {
            frameCount++;
            
            // Lógica para el boost de velocidad
            if (speedBoostDuration > 0) {
                speedBoostDuration--;
                // Super velocidad: la mitad de la velocidad normal
                gameSpeed = baseSpeed * 0.5; 
            } else {
                // Aumento de velocidad gradual con la puntuación (hasta un límite)
                const scoreFactor = 1.0 - Math.min(score, 100) / 200; // Máximo 50% de aceleración
                gameSpeed = baseSpeed * scoreFactor;
            }

            // Controla el tick de la lógica del juego
            if (timestamp - lastUpdateTime > gameSpeed && !isGameOver) {
                updateGame();
                lastUpdateTime = timestamp;
            }

            // Actualiza las partículas independientemente del tick de la lógica
            updateParticles();

            // --- FASE DE DIBUJO ---
            drawBoard();
            drawParticles(); // Dibuja el rastro antes que la serpiente y la comida
            
            if (!isGameOver) {
                drawFood();
                drawSnake();
            } else {
                // Animación de "muerte" más dramática
                const opacity = 0.5 + Math.sin(frameCount * 0.1) * 0.5;
                ctx.globalAlpha = opacity;
                drawSnake();
                ctx.globalAlpha = 1.0;
            }
            // --- FIN FASE DE DIBUJO ---

            requestAnimationFrame(gameLoop);
        }

        window.onload = initGame;
    </script>
</body>
</html>
