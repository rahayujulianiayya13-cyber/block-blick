<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Block Blast Web Game</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #1a1a2e;
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 0;
      padding: 20px;
    }

    h1 {
      margin-bottom: 5px;
      color: #00fff5;
    }

    .score-board {
      font-size: 1.5rem;
      margin-bottom: 20px;
      background: #16213e;
      padding: 10px 20px;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }

    #board {
      display: grid;
      grid-template-columns: repeat(8, 40px);
      grid-template-rows: repeat(8, 40px);
      gap: 4px;
      background-color: #0f3460;
      padding: 10px;
      border-radius: 10px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.5);
    }

    .cell {
      width: 40px;
      height: 40px;
      background-color: #1a1a2e;
      border-radius: 4px;
      transition: background-color 0.2s;
    }

    .cell.filled {
      background-color: #e94560;
    }

    #block-container {
      display: flex;
      gap: 20px;
      margin-top: 30px;
      min-height: 100px;
    }

    .shape-preview {
      display: grid;
      gap: 2px;
      cursor: pointer;
      padding: 5px;
      background: #16213e;
      border-radius: 8px;
      border: 2px solid transparent;
    }

    .shape-preview.selected {
      border-color: #00fff5;
    }

    .shape-cell {
      width: 20px;
      height: 20px;
      border-radius: 2px;
    }

    .shape-cell.filled {
      background-color: #e94560;
    }

    /* --- GAYA TAMPILAN TUTORIAL / PETUNJUK --- */
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.85);
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }

    .modal-card {
      background: #16213e;
      border: 2px solid #00fff5;
      border-radius: 15px;
      padding: 25px;
      max-width: 320px;
      width: 85%;
      text-align: center;
      box-shadow: 0 0 20px rgba(0, 255, 245, 0.3);
    }

    .modal-card h2 {
      color: #00fff5;
      margin-top: 0;
    }

    .instructions {
      text-align: left;
      font-size: 0.95rem;
      line-height: 1.6;
      margin: 15px 0;
      padding-left: 20px;
    }

    .start-btn {
      background: #e94560;
      color: white;
      border: none;
      padding: 12px 25px;
      font-size: 1rem;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      width: 100%;
      transition: 0.2s;
    }

    .start-btn:hover {
      background: #ff5277;
    }
  </style>
</head>
<body>

  <div id="tutorial-modal" class="modal-overlay">
    <div class="modal-card">
      <h2>🎮 Cara Bermain</h2>
      <ol class="instructions">
        <li><strong>Pilih Balok:</strong> Ketuk salah satu dari 3 bentuk balok di bawah papan.</li>
        <li><strong>Pasang Balok:</strong> Ketuk kotak kosong di papan 8x8 untuk memasang balok pilihanmu.</li>
        <li><strong>Hancurkan Baris:</strong> Isi 1 baris atau 1 kolom secara penuh untuk menghancurkannya dan dapat skor!</li>
      </ol>
      <button class="start-btn" onclick="closeTutorial()">Saya Paham, Mulai Main!</button>
    </div>
  </div>

  <h1>Block Blast Lite</h1>
  <div class="score-board">Skor: <span id="score">0</span></div>

  <div id="board"></div>

  <div id="block-container"></div>

  <script>
    const BOARD_SIZE = 8;
    const boardElement = document.getElementById('board');
    const scoreElement = document.getElementById('score');
    const blockContainer = document.getElementById('block-container');

    let grid = Array(BOARD_SIZE).fill(null).map(() => Array(BOARD_SIZE).fill(0));
    let score = 0;
    let selectedShape = null;

    // Bentuk-bentuk blok
    const SHAPES = [
      [[1]], // 1x1
      [[1, 1]], // 1x2 Horizontal
      [[1], [1]], // 2x1 Vertikal
      [[1, 1, 1]], // 1x3
      [[1, 1], [1, 1]], // 2x2 Kotak
      [[1, 1, 1], [0, 1, 0]] // Bentuk T
    ];

    // Fungsi Tutup Pop-up Tutorial
    function closeTutorial() {
      document.getElementById('tutorial-modal').style.display = 'none';
    }

    // Inisialisasi Papan
    function createBoard() {
      boardElement.innerHTML = '';
      for (let r = 0; r < BOARD_SIZE; r++) {
        for (let c = 0; c < BOARD_SIZE; c++) {
          const cell = document.createElement('div');
          cell.classList.add('cell');
          cell.dataset.row = r;
          cell.dataset.col = c;
          cell.addEventListener('click', () => handleCellClick(r, c));
          boardElement.appendChild(cell);
        }
      }
    }

    // Tampilkan Bentuk Acak
    function generateShapes() {
      blockContainer.innerHTML = '';
      for (let i = 0; i < 3; i++) {
        const randomIndex = Math.floor(Math.random() * SHAPES.length);
        const shape = SHAPES[randomIndex];
        
        const preview = document.createElement('div');
        preview.classList.add('shape-preview');
        preview.style.gridTemplateColumns = `repeat(${shape[0].length}, 20px)`;

        shape.forEach(row => {
          row.forEach(val => {
            const cell = document.createElement('div');
            cell.classList.add('shape-cell');
            if (val) cell.classList.add('filled');
            preview.appendChild(cell);
          });
        });

        preview.addEventListener('click', () => {
          document.querySelectorAll('.shape-preview').forEach(p => p.classList.remove('selected'));
          preview.classList.add('selected');
          selectedShape = { matrix: shape, element: preview };
        });

        blockContainer.appendChild(preview);
      }
    }

    // Pasang Blok ke Papan
    function handleCellClick(row, col) {
      if (!selectedShape) return;

      const matrix = selectedShape.matrix;
      
      if (canPlace(matrix, row, col)) {
        placeShape(matrix, row, col);
        selectedShape.element.remove();
        selectedShape = null;

        checkClears();
        
        if (blockContainer.children.length === 0) {
          generateShapes();
        }
      }
    }

    function canPlace(matrix, r, c) {
      for (let row = 0; row < matrix.length; row++) {
        for (let col = 0; col < matrix[row].length; col++) {
          if (matrix[row][col]) {
            let targetR = r + row;
            let targetC = c + col;
            if (targetR >= BOARD_SIZE || targetC >= BOARD_SIZE || grid[targetR][targetC]) {
              return false;
            }
          }
        }
      }
      return true;
    }

    function placeShape(matrix, r, c) {
      for (let row = 0; row < matrix.length; row++) {
        for (let col = 0; col < matrix[row].length; col++) {
          if (matrix[row][col]) {
            grid[r + row][c + col] = 1;
          }
        }
      }
      renderBoard();
    }

    function renderBoard() {
      const cells = boardElement.children;
      for (let r = 0; r < BOARD_SIZE; r++) {
        for (let c = 0; c < BOARD_SIZE; c++) {
          const index = r * BOARD_SIZE + c;
          if (grid[r][c] === 1) {
            cells[index].classList.add('filled');
          } else {
            cells[index].classList.remove('filled');
          }
        }
      }
    }

    // Cek Garis Penuh
    function checkClears() {
      let rowsToClear = [];
      let colsToClear = [];

      for (let r = 0; r < BOARD_SIZE; r++) {
        if (grid[r].every(val => val === 1)) rowsToClear.push(r);
      }

      for (let c = 0; c < BOARD_SIZE; c++) {
        let full = true;
        for (let r = 0; r < BOARD_SIZE; r++) {
          if (grid[r][c] === 0) full = false;
        }
        if (full) colsToClear.push(c);
      }

      rowsToClear.forEach(r => grid[r].fill(0));
      colsToClear.forEach(c => {
        for (let r = 0; r < BOARD_SIZE; r++) grid[r][c] = 0;
      });

      let totalCleared = rowsToClear.length + colsToClear.length;
      if (totalCleared > 0) {
        score += totalCleared * 10;
        scoreElement.textContent = score;
        renderBoard();
      }
    }

    createBoard();
    generateShapes();
  </script>
</body>
</html>
