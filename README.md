<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Live Trading Signal</title>
  <style>
    body {
      margin: 0;
      font-family: monospace;
      background-color: #000;
      overflow: hidden;
    }

    canvas {
      position: fixed;
      top: 0;
      left: 0;
      z-index: -1;
    }

    .center-container {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
    }

    .signal {
      font-size: 2em;
      margin-top: 20px;
    }

    .denger {
      font-weight: bold;
      animation: flash 1s infinite;
    }

    @keyframes flash {
      0% {opacity: 1;}
      50% {opacity: 0.2;}
      100% {opacity: 1;}
    }

    .top-title {
      position: absolute;
      top: 10px;
      width: 100%;
      text-align: center;
      font-size: 1.5em;
      color: rgba(255, 255, 255, 0.3);
      font-weight: bold;
    }

    .bottom-credit {
      position: absolute;
      bottom: 10px;
      right: 10px;
      font-size: 0.9em;
      color: rgba(255, 255, 255, 0.2);
    }
  </style>
</head>
<body>
  <canvas id="network"></canvas>
  <div class="top-title">XAUUSD</div>

  <div class="center-container">
    <div id="time">Time: --:--:--</div>
    <div id="price">Price: Rp 0</div>
    <div id="rsi">RSI: 0</div>
    <div id="signal" class="signal">Signal: -</div>
    <div id="estimate">Next update in: --:--</div>
    <div id="alert" class="signal"></div>
  </div>

  <div class="bottom-credit">by Kenzz</div>

  <audio id="sound">
    <source src="https://www.soundjay.com/buttons/sounds/beep-07.mp3" type="audio/mpeg" />
  </audio>

  <script>
    // Canvas Background (Connected cells)
    const canvas = document.getElementById('network');
    const ctx = canvas.getContext('2d');
    let width, height;
    let points = [];

    const colors = ['#00FFAA', '#00DDFF', '#0099FF', '#6633FF', '#FF33CC', '#FF6666'];
    let colorIndex = 0;
    let currentColor = colors[colorIndex];

    function resizeCanvas() {
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
    }

    function createPoints(count) {
      points = [];
      for (let i = 0; i < count; i++) {
        points.push({
          x: Math.random() * width,
          y: Math.random() * height,
          vx: (Math.random() - 0.5) * 0.5,
          vy: (Math.random() - 0.5) * 0.5
        });
      }
    }

    function draw() {
      ctx.clearRect(0, 0, width, height);

      for (let i = 0; i < points.length; i++) {
        let p = points[i];
        p.x += p.vx;
        p.y += p.vy;

        if (p.x < 0 || p.x > width) p.vx *= -1;
        if (p.y < 0 || p.y > height) p.vy *= -1;

        ctx.beginPath();
        ctx.arc(p.x, p.y, 2, 0, Math.PI * 2);
        ctx.fillStyle = currentColor;
        ctx.fill();

        for (let j = i + 1; j < points.length; j++) {
          let q = points[j];
          let dx = p.x - q.x;
          let dy = p.y - q.y;
          let dist = Math.sqrt(dx * dx + dy * dy);

          if (dist < 100) {
            ctx.beginPath();
            ctx.moveTo(p.x, p.y);
            ctx.lineTo(q.x, q.y);
            ctx.strokeStyle = currentColor;
            ctx.globalAlpha = 1 - dist / 100;
            ctx.stroke();
            ctx.globalAlpha = 1;
          }
        }
      }

      requestAnimationFrame(draw);
    }

    function changeColor() {
      colorIndex = (colorIndex + 1) % colors.length;
      currentColor = colors[colorIndex];

      // Update all signal-related text colors
      document.querySelectorAll(
        '.center-container, .signal, #alert, #time, #price, #rsi, #signal, #estimate'
      ).forEach(el => {
        el.style.color = currentColor;
      });
    }

    window.addEventListener('resize', () => {
      resizeCanvas();
      createPoints(100);
    });

    resizeCanvas();
    createPoints(100);
    draw();
    setInterval(changeColor, 5000);
  </script>

  <script>
    // Trading signal logic
    const priceEl = document.getElementById("price");
    const rsiEl = document.getElementById("rsi");
    const signalEl = document.getElementById("signal");
    const estimateEl = document.getElementById("estimate");
    const timeEl = document.getElementById("time");
    const alertEl = document.getElementById("alert");
    const sound = document.getElementById("sound");

    let price = 1000000;
    let rsi = 50;
    let prevPrice = price;
    let nextUpdate = randomDelay();

    function randomDelay() {
      return Math.random() < 0.32 ? randomInt(120, 360) : randomInt(20, 120);
    }

    function randomInt(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    function updateDisplay() {
      prevPrice = price;
      price += randomInt(-3000, 3000);
      rsi += randomInt(-2, 2);
      rsi = Math.max(0, Math.min(100, rsi));

      priceEl.textContent = `Price: Rp ${price.toLocaleString()}`;
      rsiEl.textContent = `RSI: ${rsi}`;

      let signal = "Stabil";
      if (rsi < 40 && price > prevPrice) signal = "Buy";
      else if (rsi > 60 && price < prevPrice) signal = "Sell";

      signalEl.textContent = `Signal: ${signal}`;

      if (Math.random() < 0.24 && signal !== "Stabil") {
        alertEl.textContent = "‼ DENGER ‼";
        alertEl.classList.add("denger");
        sound.play();
      } else {
        alertEl.textContent = "";
        alertEl.classList.remove("denger");
      }

      nextUpdate = randomDelay();
    }

    function updateTime() {
      const now = new Date();
      const hh = now.getHours().toString().padStart(2, '0');
      const mm = now.getMinutes().toString().padStart(2, '0');
      const ss = now.getSeconds().toString().padStart(2, '0');
      timeEl.textContent = `Time: ${hh}:${mm}:${ss}`;

      let minutes = Math.floor(nextUpdate / 60);
      let seconds = nextUpdate % 60;
      estimateEl.textContent = `Next update in: ${minutes.toString().padStart(2,'0')}:${seconds.toString().padStart(2,'0')}`;

      nextUpdate--;
      if (nextUpdate <= 0) updateDisplay();
    }

    updateDisplay();
    setInterval(updateTime, 1000);
  </script>
</body>
</html>
