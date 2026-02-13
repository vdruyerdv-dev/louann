<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1.0" />
  <title>Invitation Saint-Valentin</title>

  <style>
    :root{
      --pad: 16px;
      --card-max: 520px;
      --btn-pad-y: 12px;
      --btn-pad-x: 18px;
    }

    *{ box-sizing: border-box; }

    body{
      font-family: Arial, sans-serif;
      background-color: #f9f9f9;
      text-align: center;
      margin: 0;
      padding: clamp(16px, 5vw, 50px);
      min-height: 100dvh;
      display: grid;
      place-items: center;
    }

    .invitation{
      background-color: #fff;
      border-radius: 12px;
      padding: clamp(16px, 4.5vw, 22px);
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
      position: relative;
      width: min(var(--card-max), 100%);
      margin: 0 auto;
      overflow: hidden;
    }

    h2{
      margin: 0 0 10px;
      font-size: clamp(18px, 4.8vw, 22px);
      line-height: 1.2;
    }

    p{
      margin: 10px 0;
      font-size: clamp(14px, 3.8vw, 16px);
      line-height: 1.35;
    }

    .actions{
      position: relative;
      margin-top: 14px;
      padding-bottom: 180px; /* 🔥 zone de déplacement agrandie */
      display: flex;
      justify-content: center;
      gap: 10px;
      flex-wrap: wrap;
    }

    .button{
      background-color: #ff6b6b;
      color: white;
      border: none;
      border-radius: 10px;
      padding: var(--btn-pad-y) var(--btn-pad-x);
      cursor: pointer;
      text-decoration: none;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      user-select: none;
      font-size: 16px;
      min-width: 110px;
      touch-action: manipulation;
    }

    .button.disabled{
      background-color: #ccc;
      cursor: not-allowed;
    }

    #nonButton{
      position: absolute;
      transition: left 0.2s ease, top 0.2s ease;
      left: 50%;
      top: 0;
      transform: translateX(-50%);
    }

    .overlay{
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.55);
      display: none;
      align-items: center;
      justify-content: center;
      z-index: 100;
      padding: 16px;
    }

    .modal{
      width: min(680px, 100%);
      background: #fff;
      border-radius: 14px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.25);
      padding: 22px 18px;
      position: relative;
      z-index: 101;
      text-align: center;
      transform: translateY(10px) scale(0.98);
      opacity: 0;
      animation: pop 260ms ease-out forwards;
    }

    .close{
      position: absolute;
      top: 10px;
      right: 10px;
      width: 38px;
      height: 38px;
      border-radius: 10px;
      border: 0;
      cursor: pointer;
      background: #f1f1f1;
      font-size: 18px;
      line-height: 38px;
    }

    @keyframes pop{
      to{ transform: translateY(0) scale(1); opacity: 1; }
    }

    #fx{
      position: fixed;
      inset: 0;
      width: 100%;
      height: 100%;
      pointer-events: none;
      display: none;
      z-index: 60;
    }

    @media (max-width: 420px){
      .button{ min-width: 130px; font-size: 16px; }
      .actions{ padding-bottom: 200px; }
    }

    @media (prefers-reduced-motion: reduce){
      #nonButton{ transition: none; }
      .modal{ animation: none; opacity: 1; transform: none; }
    }
  </style>
</head>

<body>

  <div class="invitation">
    <h2>🌹 Invitation à la Saint-Valentin 🌹</h2>
    <p>Chère Louann,</p> 
    <p>J'aimerais passer une soirée spéciale avec toi pour célébrer la Saint-Valentin. Que dirais-tu d'une soirée en tête à tête, juste toi et moi ?</p> 

    <div class="actions" id="actions">
      <a href="#" class="button" id="yesButton">Oui</a>
      <button class="button disabled" id="nonButton" type="button">Non</button>
    </div>
  </div>

  <div class="overlay" id="overlay">
    <div class="modal">
      <button class="close" id="closeModal" aria-label="Fermer">✕</button>
      <h2>🎉 Félicitations !</h2>
      <p>Toutes mes félicitations, vous avez fait le bon choix mon petit chat d'amour !</p> 
    </div>
  </div>

  <canvas id="fx"></canvas>

  <script>
    const actions = document.getElementById("actions");
    const nonButton = document.getElementById("nonButton");
    const yesButton = document.getElementById("yesButton");
    const overlay = document.getElementById("overlay");
    const closeModal = document.getElementById("closeModal");
    const canvas = document.getElementById("fx");
    const ctx = canvas.getContext("2d");

    // 🔥 Déplacement amélioré du bouton NON
    function moveNon() {
      const pad = 8;

      const area = actions.getBoundingClientRect();
      const btn = nonButton.getBoundingClientRect();

      const maxX = area.width - btn.width - pad;
      const maxY = area.height - btn.height - pad;

      const x = Math.random() * Math.max(0, maxX);
      const y = Math.random() * Math.max(0, maxY);

      nonButton.style.left = `${x}px`;
      nonButton.style.top  = `${y}px`;
      nonButton.style.transform = "none";
    }

    nonButton.addEventListener("mouseenter", moveNon);
    nonButton.addEventListener("touchstart", (e) => { e.preventDefault(); moveNon(); }, { passive: false });
    nonButton.addEventListener("pointerdown", (e) => { if (e.pointerType !== "mouse") { e.preventDefault(); moveNon(); } });

    yesButton.addEventListener("click", (e) => {
      e.preventDefault();
      overlay.style.display = "flex";
      startFireworks();
    });

    closeModal.addEventListener("click", () => {
      overlay.style.display = "none";
      stopFireworks();
    });

    overlay.addEventListener("click", (e) => {
      if (e.target === overlay) {
        overlay.style.display = "none";
        stopFireworks();
      }
    });

    let W, H, rafId = null;
    let running = false;
    const particles = [];

    function resize() {
      W = canvas.width = window.innerWidth;
      H = canvas.height = window.innerHeight;
    }
    window.addEventListener("resize", resize);
    resize();

    function rand(min, max) {
      return Math.random() * (max - min) + min;
    }

    function spawnBurst() {
      const x = rand(W * 0.2, W * 0.8);
      const y = rand(H * 0.1, H * 0.5);

      const count = Math.floor(rand(30, 60));
      for (let i = 0; i < count; i++) {
        const angle = rand(0, Math.PI * 2);
        const speed = rand(1.5, 4);

        particles.push({
          x, y,
          vx: Math.cos(angle) * speed,
          vy: Math.sin(angle) * speed - rand(1,2),
          life: rand(40, 80),
          size: rand(1,2.5)
        });
      }
    }

    function step() {
      if (!running) return;

      ctx.fillStyle = "rgba(0,0,0,0.15)";
      ctx.fillRect(0, 0, W, H);

      if (Math.random() < 0.25) spawnBurst();

      for (let i = particles.length - 1; i >= 0; i--) {
        const p = particles[i];

        p.vy += 0.05;
        p.x += p.vx;
        p.y += p.vy;
        p.life--;

        ctx.beginPath();
        ctx.fillStyle = `rgba(255,120,120,${p.life/80})`;
        ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
        ctx.fill();

        if (p.life <= 0) particles.splice(i, 1);
      }

      rafId = requestAnimationFrame(step);
    }

    function startFireworks() {
      if (running) return;
      running = true;
      canvas.style.display = "block";
      ctx.fillStyle = "rgba(0,0,0,0.8)";
      ctx.fillRect(0, 0, W, H);
      for (let i = 0; i < 3; i++) spawnBurst();
      step();
    }

    function stopFireworks() {
      running = false;
      if (rafId) cancelAnimationFrame(rafId);
      ctx.clearRect(0, 0, W, H);
      canvas.style.display = "none";
      particles.length = 0;
    }

    window.addEventListener("load", () => {
      nonButton.style.left = "50%";
      nonButton.style.top = "0px";
      nonButton.style.transform = "translateX(-50%)";
    });
  </script>

</body>
</html>
