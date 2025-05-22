<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Draw and Animate</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: #f0f8ff;
    }

    canvas {
      border: 2px solid #333;
      background: #fff;
    }

    .button-container {
      margin-bottom: 10px;
    }

    button, input[type="color"], input[type="range"] {
      margin: 0 10px;
      padding: 10px 20px;
      font-size: 16px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    button:hover {
      background: #0056b3;
    }

    input[type="color"] {
      padding: 5px;
      width: 50px;
      height: 40px;
      cursor: pointer;
    }

    input[type="range"] {
      width: 150px;
    }
  </style>
</head>
<body>
  <div class="button-container">
    <button id="animateButton">Animate Drawing</button>
    <button id="eraseButton">Erase Drawing</button>
    <button id="fillButton">Fill Drawing</button>
    <button id="smoothButton">Smooth Drawing</button>
    <button id="imperfectionButton">Add Imperfections</button>
    <button id="moveButton">Move Drawing</button>
    <button id="undoButton">Undo</button>
    <button id="resetButton">Reset</button>
    <label for="colorPicker">Pick a Color:</label>
    <input type="color" id="colorPicker" value="#000000">
    <label for="speedControl">Animation Speed:</label>
    <input type="range" id="speedControl" min="1" max="50" value="10">
  </div>
  <canvas id="drawingCanvas" width="600" height="400"></canvas>

  <script>
    const canvas = document.getElementById('drawingCanvas');
    const ctx = canvas.getContext('2d');
    const animateButton = document.getElementById('animateButton');
    const eraseButton = document.getElementById('eraseButton');
    const fillButton = document.getElementById('fillButton');
    const smoothButton = document.getElementById('smoothButton');
    const imperfectionButton = document.getElementById('imperfectionButton');
    const moveButton = document.getElementById('moveButton');
    const undoButton = document.getElementById('undoButton');
    const resetButton = document.getElementById('resetButton');
    const colorPicker = document.getElementById('colorPicker');
    const speedControl = document.getElementById('speedControl');

    let isDrawing = false;
    let isAnimating = false;
    let isMoving = false;
    let drawingData = [];
    let originalDrawingData = [];
    let currentColor = '#000000';
    let offsetX = 0, offsetY = 0;
    let animationSpeed = 10;

    // Update the drawing color
    colorPicker.addEventListener('input', (e) => {
      currentColor = e.target.value;
      ctx.strokeStyle = currentColor;
    });

    // Update animation speed
    speedControl.addEventListener('input', (e) => {
      animationSpeed = parseInt(e.target.value, 10);
    });

    // Start drawing
    canvas.addEventListener('mousedown', (e) => {
      if (isAnimating || isMoving) return;
      isDrawing = true;
      ctx.beginPath();
      ctx.moveTo(e.offsetX, e.offsetY);
      drawingData.push({ x: e.offsetX, y: e.offsetY, move: true, color: currentColor });
    });

    // Draw on the canvas
    canvas.addEventListener('mousemove', (e) => {
      if (!isDrawing || isAnimating || isMoving) return;
      ctx.lineTo(e.offsetX, e.offsetY);
      ctx.stroke();
      drawingData.push({ x: e.offsetX, y: e.offsetY, move: false, color: currentColor });
    });

    // Stop drawing
    canvas.addEventListener('mouseup', () => {
      isDrawing = false;
    });

    // Animate the drawing
    animateButton.addEventListener('click', () => {
      if (isAnimating) return;
      isAnimating = true;
      let i = 0;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      const interval = setInterval(() => {
        if (i >= drawingData.length) {
          clearInterval(interval);
          isAnimating = false;
          return;
        }
        const point = drawingData[i];
        ctx.strokeStyle = point.color;
        if (point.move) {
          ctx.beginPath();
          ctx.moveTo(point.x, point.y);
        } else {
          ctx.lineTo(point.x, point.y);
          ctx.stroke();
        }
        i++;
      }, animationSpeed);
    });

    // Erase the drawing
    eraseButton.addEventListener('click', () => {
      if (isAnimating || isMoving) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawingData = [];
    });

    // Fill the drawing
    fillButton.addEventListener('click', () => {
      if (isAnimating) return;
      ctx.fillStyle = currentColor;
      ctx.fill();
    });

    // Smooth the drawing
    smoothButton.addEventListener('click', () => {
      if (isAnimating || drawingData.length === 0) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      for (let i = 0; i < drawingData.length - 1; i++) {
        const current = drawingData[i];
        const next = drawingData[i + 1];
        if (current.move) {
          ctx.moveTo(current.x, current.y);
        } else {
          const midX = (current.x + next.x) / 2;
          const midY = (current.y + next.y) / 2;
          ctx.quadraticCurveTo(current.x, current.y, midX, midY);
        }
      }
      ctx.stroke();
    });

    // Add imperfections
    imperfectionButton.addEventListener('click', () => {
      if (isAnimating || drawingData.length === 0) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      drawingData.forEach((point) => {
        const randomOffsetX = (Math.random() - 0.5) * 5;
        const randomOffsetY = (Math.random() - 0.5) * 5;
        ctx.strokeStyle = point.color;
        if (point.move) {
          ctx.moveTo(point.x + randomOffsetX, point.y + randomOffsetY);
        } else {
          ctx.lineTo(point.x + randomOffsetX, point.y + randomOffsetY);
        }
      });
      ctx.stroke();
    });

    // Move the drawing
    moveButton.addEventListener('click', () => {
      if (drawingData.length === 0) return;
      originalDrawingData = JSON.parse(JSON.stringify(drawingData));
      isMoving = true;
      canvas.style.cursor = 'grab';
      canvas.addEventListener('mousedown', startMove);
      canvas.addEventListener('mousemove', moveDrawing);
      canvas.addEventListener('mouseup', stopMove);
    });

    function startMove(e) {
      offsetX = e.offsetX;
      offsetY = e.offsetY;
    }

    function moveDrawing(e) {
      if (!isMoving) return;
      const dx = e.offsetX - offsetX;
      const dy = e.offsetY - offsetY;
      offsetX = e.offsetX;
      offsetY = e.offsetY;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      drawingData.forEach((point) => {
        point.x += dx;
        point.y += dy;
        ctx.strokeStyle = point.color;
        if (point.move) {
          ctx.moveTo(point.x, point.y);
        } else {
          ctx.lineTo(point.x, point.y);
        }
      });
      ctx.stroke();
    }

    function stopMove() {
      isMoving = false;
      canvas.style.cursor = 'default';
      canvas.removeEventListener('mousedown', startMove);
      canvas.removeEventListener('mousemove', moveDrawing);
      canvas.removeEventListener('mouseup', stopMove);
    }

    // Undo last stroke
    undoButton.addEventListener('click', () => {
      if (drawingData.length > 0) {
        drawingData.pop();
        redrawCanvas();
      }
    });

    // Reset to original drawing
    resetButton.addEventListener('click', () => {
      drawingData = JSON.parse(JSON.stringify(originalDrawingData));
      redrawCanvas();
    });

    function redrawCanvas() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.beginPath();
      drawingData.forEach((point) => {
        ctx.strokeStyle = point.color;
        if (point.move) {
          ctx.moveTo(point.x, point.y);
        } else {
          ctx.lineTo(point.x, point.y);
        }
      });
      ctx.stroke();
    }
  </script>
</body>
</html>
