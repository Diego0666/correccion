<template>
  <div class="game-container">
    <h1>🎮 Juego del Ahorcado</h1>
    
    <!-- Selección de Nivel de Dificultad -->
    <div v-if="gameState === 'selectLevel'" class="level-selection">
      <h2>Selecciona el Nivel de Dificultad</h2>
      <div class="level-buttons">
        <button @click="selectLevel('facil')" class="level-btn easy">
          🟢 Fácil
          <span class="level-desc">4-5 letras</span>
        </button>
        <button @click="selectLevel('medio')" class="level-btn medium">
          🟡 Medio
          <span class="level-desc">6-7 letras</span>
        </button>
        <button @click="selectLevel('dificil')" class="level-btn hard">
          🔴 Difícil
          <span class="level-desc">8+ letras</span>
        </button>
      </div>
    </div>

    <!-- Juego Principal -->
    <div v-if="gameState === 'playing'" class="game-area">
      <div class="game-info">
        <p><strong>Nivel:</strong> {{ currentLevel }}</p>
        <p><strong>Intentos restantes:</strong> {{ attemptsLeft }}</p>
        <p><strong>Categoría:</strong> {{ currentCategory }}</p>
      </div>

      <!-- Dibujo del Ahorcado SVG -->
      <div class="hangman-container">
        <svg width="200" height="250" viewBox="0 0 200 250" class="hangman-svg">
          <!-- Base -->
          <line x1="10" y1="230" x2="150" y2="230" stroke="#8B4513" stroke-width="4"/>
          
          <!-- Poste vertical -->
          <line x1="30" y1="230" x2="30" y2="20" stroke="#8B4513" stroke-width="4"/>
          
          <!-- Poste horizontal -->
          <line x1="30" y1="20" x2="120" y2="20" stroke="#8B4513" stroke-width="4"/>
          
          <!-- Cuerda -->
          <line x1="120" y1="20" x2="120" y2="50" stroke="#654321" stroke-width="3"/>
          
          <!-- Cabeza -->
          <circle 
            v-if="wrongGuesses >= 1" 
            cx="120" 
            cy="65" 
            r="15" 
            stroke="#2c3e50" 
            stroke-width="3" 
            fill="none"
            class="hangman-part"
          />
          
          <!-- Cara triste -->
          <g v-if="wrongGuesses >= 1" class="hangman-part">
            <!-- Ojos -->
            <circle cx="115" cy="60" r="2" fill="#e74c3c"/>
            <circle cx="125" cy="60" r="2" fill="#e74c3c"/>
            <!-- Boca triste -->
            <path d="M 112 72 Q 120 68 128 72" stroke="#e74c3c" stroke-width="2" fill="none"/>
          </g>
          
          <!-- Cuerpo -->
          <line 
            v-if="wrongGuesses >= 2" 
            x1="120" 
            y1="80" 
            x2="120" 
            y2="160" 
            stroke="#2c3e50" 
            stroke-width="3"
            class="hangman-part"
          />
          
          <!-- Brazo izquierdo -->
          <line 
            v-if="wrongGuesses >= 3" 
            x1="120" 
            y1="100" 
            x2="90" 
            y2="130" 
            stroke="#2c3e50" 
            stroke-width="3"
            class="hangman-part"
          />
          
          <!-- Brazo derecho -->
          <line 
            v-if="wrongGuesses >= 4" 
            x1="120" 
            y1="100" 
            x2="150" 
            y2="130" 
            stroke="#2c3e50" 
            stroke-width="3"
            class="hangman-part"
          />
          
          <!-- Pierna izquierda -->
          <line 
            v-if="wrongGuesses >= 5" 
            x1="120" 
            y1="160" 
            x2="90" 
            y2="200" 
            stroke="#2c3e50" 
            stroke-width="3"
            class="hangman-part"
          />
          
          <!-- Pierna derecha -->
          <line 
            v-if="wrongGuesses >= 6" 
            x1="120" 
            y1="160" 
            x2="150" 
            y2="200" 
            stroke="#2c3e50" 
            stroke-width="3"
            class="hangman-part"
          />
        </svg>
      </div>

      <!-- Palabra a Adivinar -->
      <div class="word-display">
        <span v-for="letter in displayWord" :key="letter.id" class="letter-box">
          {{ letter.char }}
        </span>
      </div>

      <!-- Letras Usadas -->
      <div class="used-letters" v-if="usedLetters.length > 0">
        <p><strong>Letras usadas:</strong></p>
        <span v-for="letter in usedLetters" :key="letter" 
              :class="['used-letter', { 'correct': correctLetters.includes(letter) }]">
          {{ letter }}
        </span>
      </div>

      <!-- Input manual para escribir letras -->
      <div class="input-area">
        <input 
          v-model="currentGuess" 
          @keyup.enter="guessLetter"
          @input="currentGuess = currentGuess.toLowerCase()"
          maxlength="1"
          placeholder="Escribe una letra"
          :disabled="gameState !== 'playing'"
          ref="letterInput"
          class="letter-input"
        >
        <button @click="guessLetter" :disabled="!currentGuess || gameState !== 'playing'" class="guess-btn">
          Adivinar
        </button>
      </div>

      <!-- Botón para Nueva Palabra -->
      <button @click="newWord" class="new-word-btn">Nueva Palabra</button>
    </div>

    <!-- Pantalla de Victoria/Derrota -->
    <div v-if="gameState === 'won' || gameState === 'lost'" class="game-over">
      <div v-if="gameState === 'won'" class="victory-animation">
        <h2>🎉 ¡Felicidades! ¡Ganaste! 🎉</h2>
        <div class="confetti">🎊</div>
      </div>
      <div v-if="gameState === 'lost'" class="defeat-animation">
        <h2>💀 ¡Perdiste! 💀</h2>
        <p class="defeat-message">¡El ahorcado está completo!</p>
      </div>
      <p class="reveal-word"><strong>La palabra era:</strong> {{ currentWord }}</p>
      <div class="game-over-buttons">
        <button @click="newWord" class="play-again-btn">Jugar de Nuevo</button>
        <button @click="backToLevelSelection" class="level-btn">Cambiar Nivel</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

// Variables reactivas
const gameState = ref('selectLevel')
const currentLevel = ref('')
const currentWord = ref('')
const currentCategory = ref('')
const displayWord = ref([])
const usedLetters = ref([])
const correctLetters = ref([])
const attemptsLeft = ref(6)
const currentGuess = ref('')
const letterInput = ref(null)

// Computed para errores
const wrongGuesses = computed(() => 6 - attemptsLeft.value)

// Palabras por nivel de dificultad
const words = {
  facil: {
    animales: ['GATO', 'PERRO', 'PATO', 'LEON', 'OSO', 'PEZ', 'RATA', 'LOBO'],
    frutas: ['PERA', 'UVA', 'KIWI', 'MANGO', 'LIMA', 'COCO'],
    colores: ['ROJO', 'AZUL', 'ROSA', 'GRIS', 'ORO'],
    objetos: ['MESA', 'SILLA', 'LIBRO', 'LAPIZ', 'RELOJ']
  },
  medio: {
    animales: ['CABALLO', 'JIRAFA', 'TIGRE', 'CONEJO', 'ARDILLA', 'DELFIN', 'BALLENA'],
    frutas: ['NARANJA', 'PLATANO', 'CEREZA', 'LIMON', 'SANDIA', 'MELON'],
    paises: ['MEXICO', 'BRASIL', 'ESPAÑA', 'ITALIA', 'FRANCIA', 'CANADA'],
    deportes: ['FUTBOL', 'TENNIS', 'NATACION', 'CICLISMO', 'BOXEO']
  },
  dificil: {
    animales: ['ELEFANTE', 'RINOCERONTE', 'HIPOPOTAMO', 'COCODRILO', 'MURCIELAGO', 'SALAMANDRA'],
    profesiones: ['INGENIERO', 'ARQUITECTO', 'PSICOLOGO', 'VETERINARIO', 'ASTRONAUTA', 'PALEONTOGO'],
    tecnologia: ['COMPUTADORA', 'PROGRAMACION', 'ALGORITMO', 'JAVASCRIPT', 'INTELIGENCIA', 'CIBERSEGURIDAD'],
    ciencias: ['ASTRONOMIA', 'BIOQUIMICA', 'MATEMATICAS', 'ANTROPOLOGIA', 'NEUROCIENCIA']
  }
}

// Métodos
const selectLevel = (level) => {
  currentLevel.value = level
  gameState.value = 'playing'
  newWord()
}

const newWord = () => {
  const levelWords = words[currentLevel.value]
  const categories = Object.keys(levelWords)
  const randomCategory = categories[Math.floor(Math.random() * categories.length)]
  const categoryWords = levelWords[randomCategory]
  
  currentWord.value = categoryWords[Math.floor(Math.random() * categoryWords.length)]
  currentCategory.value = randomCategory
  displayWord.value = currentWord.value.split('').map((char, index) => ({
    id: index,
    char: '_'
  }))
  
  usedLetters.value = []
  correctLetters.value = []
  attemptsLeft.value = 6
  currentGuess.value = ''
  gameState.value = 'playing'
  
  nextTick(() => {
    if (letterInput.value) {
      letterInput.value.focus()
    }
  })
}

const guessLetter = () => {
  const letter = currentGuess.value.toUpperCase()
  
  if (!letter || letter.length !== 1) return
  if (usedLetters.value.includes(letter)) {
    alert('Ya usaste esa letra')
    currentGuess.value = ''
    return
  }
  if (!/^[A-ZÁÉÍÓÚÑ]$/.test(letter)) {
    alert('Por favor ingresa solo letras')
    currentGuess.value = ''
    return
  }

  usedLetters.value.push(letter)

  if (currentWord.value.includes(letter)) {
    correctLetters.value.push(letter)
    for (let i = 0; i < currentWord.value.length; i++) {
      if (currentWord.value[i] === letter) {
        displayWord.value[i].char = letter
      }
    }
    
    if (displayWord.value.every(letter => letter.char !== '_')) {
      gameState.value = 'won'
    }
  } else {
    attemptsLeft.value--
    if (attemptsLeft.value === 0) {
      gameState.value = 'lost'
      for (let i = 0; i < currentWord.value.length; i++) {
        displayWord.value[i].char = currentWord.value[i]
      }
    }
  }

  currentGuess.value = ''
  
  // Mantener el foco en el input para continuar jugando
  nextTick(() => {
    if (letterInput.value && gameState.value === 'playing') {
      letterInput.value.focus()
    }
  })
}

const backToLevelSelection = () => {
  gameState.value = 'selectLevel'
  currentLevel.value = ''
}
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.game-container {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 20px 40px rgba(0,0,0,0.1);
  max-width: 700px;
  width: 100%;
  text-align: center;
  margin: 0 auto;
}

h1 {
  color: #333;
  margin-bottom: 30px;
  font-size: 2.5em;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
}

/* Selección de Nivel */
.level-selection h2 {
  color: #555;
  margin-bottom: 30px;
}

.level-buttons {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.level-btn {
  padding: 20px 30px;
  border: none;
  border-radius: 15px;
  font-size: 1.2em;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 150px;
}

.level-btn.easy {
  background: linear-gradient(135deg, #4CAF50, #45a049);
  color: white;
}

.level-btn.medium {
  background: linear-gradient(135deg, #FF9800, #f57c00);
  color: white;
}

.level-btn.hard {
  background: linear-gradient(135deg, #F44336, #d32f2f);
  color: white;
}

.level-btn:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.2);
}

.level-desc {
  font-size: 0.9em;
  opacity: 0.9;
}

/* Área de Juego */
.game-info {
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 20px;
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 10px;
  border: 2px solid #dee2e6;
}

.game-info p {
  margin: 5px 0;
  color: #555;
  font-weight: 600;
}

/* Dibujo del Ahorcado SVG */
.hangman-container {
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  padding: 20px;
  border-radius: 15px;
  margin: 20px 0;
  border: 3px solid #dee2e6;
  display: flex;
  justify-content: center;
}

.hangman-svg {
  filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.1));
}

.hangman-part {
  animation: drawPart 0.5s ease-in-out;
}

@keyframes drawPart {
  from {
    opacity: 0;
    transform: scale(0.5);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* Palabra Display */
.word-display {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin: 30px 0;
  flex-wrap: wrap;
}

.letter-box {
  width: 45px;
  height: 45px;
  border: 3px solid #007bff;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8em;
  font-weight: bold;
  background: linear-gradient(135deg, #ffffff, #f8f9fa);
  color: #333;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.letter-box:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

/* Letras Usadas */
.used-letters {
  margin: 20px 0;
  padding: 15px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-radius: 10px;
  border: 2px solid #dee2e6;
}

.used-letter {
  display: inline-block;
  margin: 5px;
  padding: 8px 12px;
  background: linear-gradient(135deg, #dc3545, #bd2130);
  color: white;
  border-radius: 8px;
  font-weight: bold;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.used-letter.correct {
  background: linear-gradient(135deg, #28a745, #1e7e34);
}

/* Input Area - CON CONTRASTE MEJORADO */
.input-area {
  display: flex;
  gap: 15px;
  justify-content: center;
  align-items: center;
  margin: 30px 0;
  padding: 20px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-radius: 15px;
  border: 2px solid #dee2e6;
  flex-wrap: wrap;
}

.letter-input {
  padding: 15px 25px;
  border: 3px solid #007bff;
  border-radius: 12px;
  font-size: 1.5em;
  text-align: center;
  width: 100px;
  height: 60px;
  text-transform: uppercase;
  background: #ffffff; /* FONDO BLANCO SÓLIDO */
  font-weight: bold;
  color: #2c3e50; /* COLOR DE TEXTO OSCURO */
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.letter-input:focus {
  outline: none;
  border-color: #0056b3;
  background: #ffffff; /* MANTENER FONDO BLANCO */
  box-shadow: 0 0 15px rgba(0, 123, 255, 0.4);
  transform: scale(1.05);
  color: #1a252f; /* COLOR MÁS OSCURO AL HACER FOCUS */
}

.letter-input:disabled {
  background: #f8f9fa; /* FONDO CLARO CUANDO ESTÁ DESHABILITADO */
  border-color: #6c757d;
  cursor: not-allowed;
  opacity: 0.6;
  color: #6c757d; /* COLOR DE TEXTO GRIS CUANDO ESTÁ DESHABILITADO */
}

.letter-input::placeholder {
  color: #6c757d; /* COLOR DEL PLACEHOLDER */
  opacity: 0.7;
}

.guess-btn {
  padding: 15px 30px;
  background: linear-gradient(135deg, #007bff, #0056b3);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 1.2em;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  height: 60px;
}

.guess-btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #0056b3, #004085);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

.guess-btn:disabled {
  background: #6c757d;
  cursor: not-allowed;
  opacity: 0.6;
}

.new-word-btn {
  background: linear-gradient(135deg, #28a745, #1e7e34);
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 10px;
  font-size: 1.2em;
  cursor: pointer;
  margin-top: 20px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.new-word-btn:hover {
  background: linear-gradient(135deg, #1e7e34, #155724);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

/* Pantalla de Fin de Juego */
.game-over {
  text-align: center;
}

.victory-animation h2 {
  color: #28a745;
  margin-bottom: 20px;
  font-size: 2.2em;
  animation: bounce 1s infinite;
}

.defeat-animation h2 {
  color: #dc3545;
  margin-bottom: 20px;
  font-size: 2.2em;
  animation: shake 0.5s ease-in-out;
}

.defeat-message {
  color: #6c757d;
  font-style: italic;
  margin-bottom: 10px;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-10px);
  }
  60% {
    transform: translateY(-5px);
  }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

.confetti {
  font-size: 3em;
  animation: confettifall 2s ease-in-out infinite;
}

@keyframes confettifall {
  0% { transform: translateY(-20px) rotate(0deg); }
  100% { transform: translateY(20px) rotate(360deg); }
}

.reveal-word {
  font-size: 1.4em;
  margin-bottom: 30px;
  color: #555;
  padding: 15px;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  border-radius: 10px;
  border: 2px solid #dee2e6;
}

.game-over-buttons {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}

.play-again-btn {
  background: linear-gradient(135deg, #28a745, #1e7e34);
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 10px;
  font-size: 1.2em;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.play-again-btn:hover {
  background: linear-gradient(135deg, #1e7e34, #155724);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

/* Responsive */
@media (max-width: 768px) {
  .game-container {
    padding: 20px;
    margin: 10px;
  }
  
  .level-buttons {
    flex-direction: column;
    align-items: center;
  }
  
  .level-btn {
    width: 100%;
    max-width: 250px;
  }
  
  .word-display {
    gap: 5px;
  }
  
  .letter-box {
    width: 35px;
    height: 35px;
    font-size: 1.3em;
  }
  
  .hangman-svg {
    width: 150px;
    height: 180px;
  }
  
  .input-area {
    flex-direction: column;
    gap: 15px;
  }
  
  .letter-input {
    width: 80px;
    height: 50px;
    font-size: 1.3em;
  }
  
  .guess-btn {
    width: 100%;
    max-width: 200px;
    height: 50px;
  }
}
</style>
