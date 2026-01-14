# chess-web.in
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Classical Chess — Single File</title>

  <!-- Chessboard.js CSS -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css">

  <style>
    :root {
      --bg: #0f1115;
      --panel: #171b23;
      --text: #e6edf3;
      --muted: #9aa4b2;
      --accent: #d4a373;
      --wood-light: #e0c097;
      --wood-dark: #8b5a2b;
      --highlight: rgba(102, 204, 255, 0.35);
      --lastmove: rgba(255, 230, 150, 0.45);
      --dot: rgba(50, 150, 200, 0.75);
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      background: radial-gradient(1200px 600px at 20% 10%, #12151c 0%, #0b0d12 60%, #07080c 100%);
      color: var(--text);
      font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial;
    }

    .app { max-width: 1200px; margin: 24px auto; padding: 0 16px; }

    .header {
      display: flex; align-items: baseline; justify-content: space-between;
      margin-bottom: 16px;
    }
    .header h1 { margin: 0; font-size: 28px; letter-spacing: 0.5px; }
    .status { display: flex; gap: 16px; color: var(--muted); }
    .fen { max-width: 520px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

    .layout {
      display: grid; grid-template-columns: 1fr 340px; gap: 18px; align-items: start;
    }

    .board-wrap {
      position: relative;
      display: grid;
      grid-template-areas:
        "board";
      grid-template-columns: minmax(480px, 720px);
      justify-content: center;
    }

    .board-frame {
      grid-area: board;
      position: relative;
      padding: 12px;
      background: linear-gradient(135deg, #4a3422, #2e2014);
      border: 2px solid #2a1d12;
      border-radius: 12px;
      box-shadow: inset 0 0 0 2px #1f140c, 0 12px 28px rgba(0,0,0,0.45);
    }

    .board {
      width: 100%;
      max-width: 720px;
      aspect-ratio: 1 / 1;
      border-radius: 8px;
      overflow: hidden;
    }

    /* Classical wood squares */
    .white-1e1d7 { background: var(--wood-light) !important; }
    .black-3c85d { background: var(--wood-dark) !important; }

    /* Overlay for highlights */
    .overlay {
      position: absolute; inset: 12px; pointer-events: none;
      display: grid; grid-template-columns: repeat(8, 1fr); grid-template-rows: repeat(8, 1fr);
    }
    .overlay .cell { position: relative; }
    .overlay .cell .dot {
      width: 22%; height: 22%; border-radius: 50%;
      background: var(--dot); margin: auto; position: absolute; inset: 0;
    }
    .overlay .cell .ring {
      position: absolute; inset: 0; border: 3px solid var(--dot); border-radius: 6px;
    }
    .overlay .cell .last {
      position: absolute; inset: 0; background: var(--lastmove);
    }
    .overlay .cell .select {
      position: absolute; inset: 0; background: var(--highlight);
    }

    /* Side panel */
    .panel {
      background: var(--panel);
      border: 1px solid #222836;
      border-radius: 12px;
      padding: 14px;
      display: flex; flex-direction: column; gap: 14px;
      box-shadow: 0 10px 24px rgba(0,0,0,0.35);
    }

    .controls { display: flex; gap: 8px; flex-wrap: wrap; }
    button {
      background: #22293a; color: var(--text);
      border: 1px solid #2b3347; border-radius: 8px;
      padding: 8px 12px; cursor: pointer;
    }
    button:hover { border-color: var(--accent); }

    .moves h3, .info h3 { margin: 0 0 8px 0; font-size: 16px; color: var(--muted); }
    #moveList { margin: 0; padding-left: 18px; max-height: 360px; overflow: auto; }
    .info ul { margin: 0; padding-left: 18px; display: grid; gap: 6px; }

    /* Modal */
    .modal {
      position: fixed; inset: 0; display: grid; place-items: center;
      background: rgba(0,0,0,0.55);
    }
    .modal.hidden { display: none; }
    .modal-content {
      background: #1a1f2a; border: 1px solid #2b3347; border-radius: 12px;
      padding: 16px; width: 320px; box-shadow: 0 12px 28px rgba(0,0,0,0.45);
    }
    .modal-content h3 { margin: 0 0 12px 0; color: var(--text); }
    .promo-choices { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; margin-bottom: 12px; }
    .promo-choices button { padding: 10px; }
    .cancel { width: 100%; background: #2a3347; }

    /* Footer */
    .footer { margin-top: 16px; color: var(--muted); text-align: center; }

    /* Responsive */
    @media (max-width: 980px) {
      .layout { grid-template-columns: 1fr; }
      .panel { order: -1; }
      .board-wrap { grid-template-columns: minmax(360px, 1fr); }
    }
  </style>
</head>
<body>
  <div class="app">
    <header class="header">
      <h1>Classical Chess</h1>
      <div class="status">
        <span id="turn">White to move</span>
        <span id="state">Ready</span>
        <span id="fen" class="fen"></span>
      </div>
    </header>

    <main class="layout">
      <section class="board-wrap">
        <div class="board-frame">
          <div id="board" class="board"></div>
          <div id="overlay" class="overlay"></div>
        </div>
      </section>

      <aside class="panel">
        <div class="controls">
          <button id="newGame">New game</button>
          <button id="undo">Undo</button>
          <button id="redo">Redo</button>
          <button id="flip">Flip board</button>
          <button id="exportPGN">Export PGN</button>
        </div>

        <div class="moves">
          <h3>Move list</h3>
          <ol id="moveList"></ol>
        </div>

        <div class="info">
          <h3>Game status</h3>
          <ul>
            <li><strong>Check:</strong> <span id="checkFlag">No</span></li>
            <li><strong>Checkmate:</strong> <span id="mateFlag">No</span></li>
            <li><strong>Draw:</strong> <span id="drawFlag">No</span></li>
          </ul>
        </div>
      </aside>
    </main>

    <footer class="footer">
      <small>Staunton pieces • chess.js + chessboard.js • Classical wood theme</small>
    </footer>
  </div>

  <!-- Promotion modal -->
  <div id="promotionModal" class="modal hidden">
    <div class="modal-content">
      <h3>Choose promotion</h3>
      <div class="promo-choices">
        <button data-piece="q">Queen</button>
        <button data-piece="r">Rook</button>
        <button data-piece="b">Bishop</button>
        <button data-piece="n">Knight</button>
      </div>
      <button id="cancelPromotion" class="cancel">Cancel</button>
    </div>
  </div>

  <!-- Dependencies -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>

  <script>
    // Core state
    let game = new Chess();
    let board = null;
    let isFlipped = false;
    let pendingPromotion = null; // { from, to, color }
    const redoStack = [];

    // Elements
    const turnEl = document.getElementById("turn");
    const stateEl = document.getElementById("state");
    const fenEl = document.getElementById("fen");
    const moveListEl = document.getElementById("moveList");
    const overlayEl = document.getElementById("overlay");
    const checkFlagEl = document.getElementById("checkFlag");
    const mateFlagEl = document.getElementById("mateFlag");
    const drawFlagEl = document.getElementById("drawFlag");
    const modalEl = document.getElementById("promotionModal");
    const cancelPromotionEl = document.getElementById("cancelPromotion");

    // Utilities
    function algebraToIndex(square) {
      const file = square.charCodeAt(0) - 97; // a->0
      const rank = 8 - parseInt(square[1], 10); // 1->7, 8->0
      return { file, rank };
    }
    function clearOverlay() { overlayEl.innerHTML = ""; }
    function makeOverlayCell(file, rank) {
      const cell = document.createElement("div");
      cell.className = "cell";
      cell.style.gridColumn = (file + 1);
      cell.style.gridRow = (rank + 1);
      return cell;
    }

    // Status updates
    function updateStatus() {
      const moveColor = game.turn() === "w" ? "White" : "Black";
      let status = `${moveColor} to move`;

      checkFlagEl.textContent = game.in_check() ? "Yes" : "No";
      mateFlagEl.textContent = game.in_checkmate() ? "Yes" : "No";
      drawFlagEl.textContent = game.in_draw() ? "Yes" : "No";

      if (game.in_checkmate()) status = `Checkmate — ${moveColor === "White" ? "Black" : "White"} wins`;
      else if (game.in_stalemate()) status = "Stalemate";
      else if (game.in_threefold_repetition()) status = "Draw — threefold repetition";
      else if (game.insufficient_material()) status = "Draw — insufficient material";
      else if (game.in_draw()) status = "Draw";

      turnEl.textContent = status;
      stateEl.textContent = game.in_check() ? "Check" : "Ready";
      fenEl.textContent = game.fen();
    }

    // Move list
    function resetMoves() { moveListEl.innerHTML = ""; }
    function addMoveToList(move) {
      const li = document.createElement("li");
      li.textContent = move.san;
      moveListEl.appendChild(li);
      moveListEl.scrollTop = moveListEl.scrollHeight;
    }
    function rebuildMoveList() {
      resetMoves();
      const history = game.history({ verbose: true });
      history.forEach(m => addMoveToList(m));
    }

    // Highlights
    let selectedSquare = null;
    let lastMove = null;

    function highlightSelected(square) {
      if (!square) return;
      const { file, rank } = algebraToIndex(square);
      const cell = makeOverlayCell(file, rank);
      const sel = document.createElement("div");
      sel.className = "select";
      cell.appendChild(sel);
      overlayEl.appendChild(cell);
    }

    function highlightLastMove() {
      if (!lastMove) return;
      const from = algebraToIndex(lastMove.from);
      const to = algebraToIndex(lastMove.to);

      [from, to].forEach(({ file, rank }) => {
        const cell = makeOverlayCell(file, rank);
        const lm = document.createElement("div");
        lm.className = "last";
        cell.appendChild(lm);
        overlayEl.appendChild(cell);
      });
    }

    function highlightLegalMoves(square) {
      const moves = game.moves({ square, verbose: true });
      moves.forEach(m => {
        const { file, rank } = algebraToIndex(m.to);
        const cell = makeOverlayCell(file, rank);
        if (m.flags.includes("c")) {
          const ring = document.createElement("div");
          ring.className = "ring";
          cell.appendChild(ring);
        } else {
          const dot = document.createElement("div");
          dot.className = "dot";
          cell.appendChild(dot);
        }
        overlayEl.appendChild(cell);
      });
    }

    function refreshHighlights() {
      clearOverlay();
      highlightSelected(selectedSquare);
      highlightLastMove();
      if (selectedSquare) highlightLegalMoves(selectedSquare);
    }

    // Promotion modal
    function showPromotion(from, to, color) {
      pendingPromotion = { from, to, color };
      modalEl.classList.remove("hidden");
    }
    function hidePromotion() {
      pendingPromotion = null;
      modalEl.classList.add("hidden");
    }
    document.querySelectorAll(".promo-choices button").forEach(btn => {
      btn.addEventListener("click", () => {
        if (!pendingPromotion) return;
        const piece = btn.dataset.piece; // q r b n
        const move = game.move({
          from: pendingPromotion.from,
          to: pendingPromotion.to,
          promotion: piece
        });
        hidePromotion();
        if (move) {
          lastMove = move;
          redoStack.length = 0;
          board.position(game.fen());
          rebuildMoveList();
          updateStatus();
          refreshHighlights();
        }
      });
    });
    cancelPromotionEl.addEventListener("click", hidePromotion);

    // Board interactions
    function onDragStart(source, piece) {
      if (game.game_over()) return false;
      if ((game.turn() === "w" && piece.startsWith("b")) ||
          (game.turn() === "b" && piece.startsWith("w"))) return false;

      selectedSquare = source;
      refreshHighlights();
    }

    function onDrop(source, target) {
      const movingPiece = game.get(source);
      const isPawn = movingPiece && movingPiece.type === "p";
      const targetRank = target[1];

      if (isPawn && (targetRank === "8" || targetRank === "1")) {
        showPromotion(source, target, movingPiece.color);
        return "snapback";
      }

      const move = game.move({ from: source, to: target, promotion: "q" });
      if (move === null) return "snapback";

      lastMove = move;
      redoStack.length = 0;
      addMoveToList(move);
      updateStatus();
    }

    function onSnapEnd() {
      board.position(game.fen());
      selectedSquare = null;
      refreshHighlights();
    }

    function onSquareClick(square) {
      selectedSquare = selectedSquare === square ? null : square;
      refreshHighlights();
    }

    // Initialize board
    function initBoard() {
      board = Chessboard("board", {
        position: "start",
        draggable: true,
        onDragStart,
        onDrop,
        onSnapEnd,
        onSquareClick
      });
      overlayEl.innerHTML = "";
      for (let r = 0; r < 8; r++) {
        for (let f = 0; f < 8; f++) {
          overlayEl.appendChild(makeOverlayCell(f, r));
        }
      }
      updateStatus();
      refreshHighlights();
    }

    // Controls
    document.getElementById("newGame").addEventListener("click", () => {
      game.reset();
      lastMove = null;
      selectedSquare = null;
      redoStack.length = 0;
      board.start();
      resetMoves();
      updateStatus();
      refreshHighlights();
    });

    document.getElementById("undo").addEventListener("click", () => {
      const undone = game.undo();
      if (undone) {
        redoStack.push(undone);
        if (moveListEl.lastChild) moveListEl.removeChild(moveListEl.lastChild);
        lastMove = null;
        board.position(game.fen());
        updateStatus();
        refreshHighlights();
      }
    });

    document.getElementById("redo").addEventListener("click", () => {
      const move = redoStack.pop();
      if (!move) return;
      const redone = game.move(move);
      if (redone) {
        addMoveToList(redone);
        lastMove = redone;
        board.position(game.fen());
        updateStatus();
        refreshHighlights();
      } else {
        redoStack.push(move);
      }
    });

    document.getElementById("flip").addEventListener("click", () => {
      isFlipped = !isFlipped;
      board.flip();
    });

    document.getElementById("exportPGN").addEventListener("click", () => {
      const pgn = game.pgn();
      navigator.clipboard.writeText(pgn).then(() => {
        stateEl.textContent = "PGN copied";
        setTimeout(() => stateEl.textContent = game.in_check() ? "Check" : "Ready", 1500);
      });
    });

    // Boot
    window.addEventListener("load", initBoard);
  </script>
</body>
</html>
