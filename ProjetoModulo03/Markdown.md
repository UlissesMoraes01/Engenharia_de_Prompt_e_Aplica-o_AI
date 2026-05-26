# Arquivo Markdown com Todo o Código Criado
# Imports e Configurações Iniciais
import { useEffect, useRef, useState } from 'react';
/**
 * DESIGN PHILOSOPHY: Retro Arcade Neon
 * - Neon vibrante contra fundo escuro
 * - Feedback visual rico com glow effects
 * - Animações que celebram cada ação
 * - Paleta: Magenta, Ciano, Amarelo contra preto profundo
 */

interface Position {
  x: number;
  y: number;
}

type Direction = 'UP' | 'DOWN' | 'LEFT' | 'RIGHT';

const GRID_SIZE = 20;
const CANVAS_WIDTH = 400;
const CANVAS_HEIGHT = 400;
const CELL_SIZE = CANVAS_WIDTH / GRID_SIZE;
const INITIAL_SPEED = 100;

# Estados do Jogo
export default function Home() {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const [snake, setSnake] = useState<Position[]>([{ x: 10, y: 10 }]);
  const [food, setFood] = useState<Position>({ x: 15, y: 15 });
  const [direction, setDirection] = useState<Direction>('RIGHT');
  const [nextDirection, setNextDirection] = useState<Direction>('RIGHT');
  const [score, setScore] = useState(0);
  const [gameOver, setGameOver] = useState(false);
  const [gameStarted, setGameStarted] = useState(false);
  const [speed, setSpeed] = useState(INITIAL_SPEED);
  const gameLoopRef = useRef<ReturnType<typeof setInterval> | undefined>(undefined);
  const directionChangeRef = useRef<boolean>(false);

# Função: Gerar Nova Comida
const generateFood = (snakePositions: Position[]): Position => {
  let newFood: Position = { x: 0, y: 0 };
  let isOnSnake = true;

  while (isOnSnake) {
    newFood = {
      x: Math.floor(Math.random() * GRID_SIZE),
      y: Math.floor(Math.random() * GRID_SIZE),
    };

    isOnSnake = snakePositions.some(
      (segment) => segment.x === newFood.x && segment.y === newFood.y
    );
  }

  return newFood;
};

# Função: Desenhar no Canvas
const draw = (snakePositions: Position[], foodPosition: Position) => {
  const canvas = canvasRef.current;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  // Fundo preto profundo
  ctx.fillStyle = '#0A0E27';
  ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);

  // Grid de fundo sutil
  ctx.strokeStyle = 'rgba(0, 255, 255, 0.05)';
  ctx.lineWidth = 0.5;
  for (let i = 0; i <= GRID_SIZE; i++) {
    const pos = i * CELL_SIZE;
    ctx.beginPath();
    ctx.moveTo(pos, 0);
    ctx.lineTo(pos, CANVAS_HEIGHT);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(0, pos);
    ctx.lineTo(CANVAS_WIDTH, pos);
    ctx.stroke();
  }

  // Desenhar comida com glow
  const foodX = foodPosition.x * CELL_SIZE + CELL_SIZE / 2;
  const foodY = foodPosition.y * CELL_SIZE + CELL_SIZE / 2;

  // Glow amarelo
  ctx.fillStyle = 'rgba(255, 255, 0, 0.3)';
  ctx.beginPath();
  ctx.arc(foodX, foodY, CELL_SIZE * 0.8, 0, Math.PI * 2);
  ctx.fill();

  // Comida amarela
  ctx.fillStyle = '#FFFF00';
  ctx.beginPath();
  ctx.arc(foodX, foodY, CELL_SIZE * 0.35, 0, Math.PI * 2);
  ctx.fill();

  // Desenhar cobrinha
  snakePositions.forEach((segment, index) => {
    const x = segment.x * CELL_SIZE + 1;
    const y = segment.y * CELL_SIZE + 1;
    const size = CELL_SIZE - 2;

    if (index === 0) {
      // Cabeça com glow magenta
      ctx.fillStyle = 'rgba(255, 0, 255, 0.4)';
      ctx.fillRect(x - 2, y - 2, size + 4, size + 4);

      ctx.fillStyle = '#FF00FF';
      ctx.fillRect(x, y, size, size);

      // Olhos
      ctx.fillStyle = '#00FFFF';
      const eyeSize = 3;
      const eyeOffset = 5;

      if (direction === 'RIGHT') {
        ctx.fillRect(x + eyeOffset, y + 4, eyeSize, eyeSize);
        ctx.fillRect(x + eyeOffset, y + size - 7, eyeSize, eyeSize);
      } else if (direction === 'LEFT') {
        ctx.fillRect(x + size - eyeOffset - eyeSize, y + 4, eyeSize, eyeSize);
        ctx.fillRect(x + size - eyeOffset - eyeSize, y + size - 7, eyeSize, eyeSize);
      } else if (direction === 'UP') {
        ctx.fillRect(x + 4, y + size - eyeOffset - eyeSize, eyeSize, eyeSize);
        ctx.fillRect(x + size - 7, y + size - eyeOffset - eyeSize, eyeSize, eyeSize);
      } else {
        ctx.fillRect(x + 4, y + eyeOffset, eyeSize, eyeSize);
        ctx.fillRect(x + size - 7, y + eyeOffset, eyeSize, eyeSize);
      }
    } else {
      // Corpo com gradiente de cor
      const hue = (index * 10) % 360;
      ctx.fillStyle = `hsl(${hue}, 100%, 50%)`;
      ctx.fillRect(x, y, size, size);

      // Borda ciano
      ctx.strokeStyle = '#00FFFF';
      ctx.lineWidth = 1;
      ctx.strokeRect(x, y, size, size);
    }
  });
};

# Função: Atualizar Jogo
const updateGame = () => {
  if (!gameStarted || gameOver) return;

  setSnake((prevSnake) => {
    const newSnake = [...prevSnake];
    const head = { ...newSnake[0] };

    // Aplicar próxima direção se for válida
    if (
      (nextDirection === 'UP' && direction !== 'DOWN') ||
      (nextDirection === 'DOWN' && direction !== 'UP') ||
      (nextDirection === 'LEFT' && direction !== 'RIGHT') ||
      (nextDirection === 'RIGHT' && direction !== 'LEFT')
    ) {
      setDirection(nextDirection);
      directionChangeRef.current = false;
    }

    // Mover cabeça
    switch (direction) {
      case 'UP':
        head.y = (head.y - 1 + GRID_SIZE) % GRID_SIZE;
        break;
      case 'DOWN':
        head.y = (head.y + 1) % GRID_SIZE;
        break;
      case 'LEFT':
        head.x = (head.x - 1 + GRID_SIZE) % GRID_SIZE;
        break;
      case 'RIGHT':
        head.x = (head.x + 1) % GRID_SIZE;
        break;
    }

    newSnake.unshift(head);

    // Verificar colisão com corpo
    for (let i = 1; i < newSnake.length; i++) {
      if (head.x === newSnake[i].x && head.y === newSnake[i].y) {
        setGameOver(true);
        return prevSnake;
      }
    }

    // Verificar se comeu
    if (head.x === food.x && head.y === food.y) {
      setScore((prev) => prev + 10);
      setFood(generateFood(newSnake));
      setSpeed((prev) => Math.max(50, prev - 2));
    } else {
      newSnake.pop();
    }

    return newSnake;
  });
};

#Game Loop
useEffect(() => {
  if (gameLoopRef.current) clearInterval(gameLoopRef.current);

  if (gameStarted && !gameOver) {
    gameLoopRef.current = setInterval(updateGame, speed);
  }

  return () => {
    if (gameLoopRef.current) clearInterval(gameLoopRef.current);
  };
}, [gameStarted, gameOver, speed, direction, nextDirection]);

#Controle por Teclado

useEffect(() => {
  const handleKeyPress = (e: KeyboardEvent) => {
    if (!gameStarted && (e.key === ' ' || e.key === 'Enter')) {
      e.preventDefault();
      setGameStarted(true);
      return;
    }

    if (!directionChangeRef.current) {
      switch (e.key.toUpperCase()) {
        case 'ARROWUP':
        case 'W':
          e.preventDefault();
          setNextDirection('UP');
          directionChangeRef.current = true;
          break;
        case 'ARROWDOWN':
        case 'S':
          e.preventDefault();
          setNextDirection('DOWN');
          directionChangeRef.current = true;
          break;
        case 'ARROWLEFT':
        case 'A':
          e.preventDefault();
          setNextDirection('LEFT');
          directionChangeRef.current = true;
          break;
        case 'ARROWRIGHT':
        case 'D':
          e.preventDefault();
          setNextDirection('RIGHT');
          directionChangeRef.current = true;
          break;
      }
    }
  };

  window.addEventListener('keydown', handleKeyPress);
  return () => window.removeEventListener('keydown', handleKeyPress);
}, [gameStarted]);

# Reset do Jogo
const resetGame = () => {
  setSnake([{ x: 10, y: 10 }]);
  setFood({ x: 15, y: 15 });
  setDirection('RIGHT');
  setNextDirection('RIGHT');
  setScore(0);
  setGameOver(false);
  setGameStarted(false);
  setSpeed(INITIAL_SPEED);
};

# Animação Neon Pulse
@keyframes neon-pulse {
  0%, 100% {
    opacity: 1;
    filter: drop-shadow(0 0 20px rgba(255, 0, 255, 0.6)) 
            drop-shadow(0 0 40px rgba(0, 255, 255, 0.3));
  }
  50% {
    opacity: 0.8;
    filter: drop-shadow(0 0 10px rgba(255, 0, 255, 0.4)) 
            drop-shadow(0 0 20px rgba(0, 255, 255, 0.2));
  }
}

# Animação Glitch

@keyframes glitch {
  0%, 100% {
    transform: translate(0);
  }
  20% {
    transform: translate(-2px, 2px);
  }
  40% {
    transform: translate(-2px, -2px);
  }
  60% {
    transform: translate(2px, 2px);
  }
  80% {
    transform: translate(2px, -2px);
  }
}

# Efeitos de Brilho
.neon-glow {
  text-shadow: 
    0 0 10px rgba(255, 0, 255, 0.8),
    0 0 20px rgba(0, 255, 255, 0.6),
    0 0 30px rgba(255, 255, 0, 0.4);
}

.neon-border {
  border-color: rgba(0, 255, 255, 0.8);
  box-shadow: 
    0 0 10px rgba(0, 255, 255, 0.5),
    inset 0 0 10px rgba(0, 255, 255, 0.1);
}

# Transições de Botões
button {
  transition: all 0.2s cubic-bezier(0.23, 1, 0.32, 1);
}

button:active {
  transform: scale(0.95);
}

button:hover:not(:disabled) {
  transform: scale(1.05);
  filter: brightness(1.1);
}

# Fluxo do Jogo
1. Usuário clica em START GAME
   ↓
2. gameStarted = true
   ↓
3. Game Loop começa (setInterval a cada 100ms)
   ↓
4. updateGame() é chamado:
   - Valida próxima direção
   - Move cabeça
   - Verifica colisão com corpo
   - Verifica se comeu
   - Remove último segmento (se não comeu)
   ↓
5. draw() renderiza o estado atual
   ↓
6. Se comeu:
   - Score += 10
   - Speed -= 2 (máximo 50ms)
   - Nova comida é gerada
   ↓
7. Se colidiu:
   - gameOver = true
   - Game Loop para
   - Modal de Game Over aparece
  
   # Wraparound (Saída por um lado = Entrada pelo outro)
   // Quando sai pela direita, entra pela esquerda
head.x = (head.x + 1) % GRID_SIZE;

// Quando sai pela esquerda, entra pela direita
head.x = (head.x - 1 + GRID_SIZE) % GRID_SIZE;

# Detecção de Colisão
// Verifica se cabeça colidiu com algum segmento do corpo
for (let i = 1; i < newSnake.length; i++) {
  if (head.x === newSnake[i].x && head.y === newSnake[i].y) {
    setGameOver(true);
    return prevSnake;
  }
}

# Crescimento da Cobrinha
// Se comeu, não remove o último segmento
if (head.x === food.x && head.y === food.y) {
  setScore((prev) => prev + 10);
  setFood(generateFood(newSnake));
  setSpeed((prev) => Math.max(50, prev - 2));
} else {
  // Se não comeu, remove o último segmento
  newSnake.pop();
}

# Estrutura HTML/React
return (
  <div className="min-h-screen bg-gradient-to-b from-[#0A0E27] to-[#1a1f3a] flex items-center justify-center p-4">
    <div className="w-full max-w-md">
      {/* Título */}
      <h1>SNAKE GAME</h1>
      
      {/* Canvas */}
      <canvas ref={canvasRef} width={400} height={400} />
      
      {/* Placar */}
      <div>SCORE e SPEED</div>
      
      {/* Controles */}
      <div>Instruções</div>
      
      {/* Botões */}
      <button>START GAME</button>
      <button>RESET</button>
      
      {/* Modal Game Over */}
      {gameOver && <div>GAME OVER Modal</div>}
    </div>
  </div>
);
