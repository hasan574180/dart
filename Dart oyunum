<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pro Live Dart 501 - Fix</title>
    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
    <style>
        body { margin: 0; background: #0a0f14; color: white; font-family: sans-serif; overflow: hidden; touch-action: none; }
        #lobby { position: absolute; inset: 0; background: rgba(0,0,0,0.95); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 1000; padding: 20px; }
        .card { background: #1a252f; padding: 20px; border-radius: 15px; border: 2px solid #3498db; margin-bottom: 20px; width: 90%; max-width: 350px; text-align: center;}
        #game-ui { position: absolute; top: 0; width: 100%; display: flex; justify-content: space-around; padding: 15px 0; background: rgba(0,0,0,0.5); pointer-events: none; }
        .player { text-align: center; padding: 10px; border-radius: 10px; min-width: 120px; border: 2px solid #444; position: relative; }
        .active { border-color: #f1c40f; background: rgba(241, 196, 15, 0.1); }
        .score { font-size: 2.2rem; font-weight: bold; }
        .minus-score { position: absolute; bottom: -30px; left: 50%; transform: translateX(-50%); color: #e74c3c; font-weight: bold; font-size: 1.5rem; opacity: 0; transition: 0.5s; }
        #reaction-msg { position: absolute; top: 150px; width: 100%; text-align: center; font-size: 2rem; font-weight: bold; text-shadow: 2px 2px 5px #000; pointer-events: none; }
        canvas { display: block; }
    </style>
</head>
<body>

    <div id="lobby">
        <div class="card">
            <h1>DART 501 LIVE</h1>
            <p>ID'niz: <strong id="my-id">...</strong></p>
            <input type="text" id="peer-id-input" placeholder="Arkadaş ID Yapıştır" style="padding:10px; width:80%">
            <br><br>
            <button onclick="connectToPeer()" style="padding:10px 20px; background:#3498db; color:white; border:none; border-radius:5px;">BAĞLAN</button>
            <button onclick="startAI()" style="padding:10px 20px; background:#27ae60; color:white; border:none; border-radius:5px; margin-top:10px;">YAPAY ZEKA</button>
        </div>
    </div>

    <div id="game-ui" style="display:none">
        <div id="p1-box" class="player active">SİZ<span id="s1" class="score">501</span><div id="minus1" class="minus-score"></div></div>
        <div id="p2-box" class="player">RAKİP<span id="s2" class="score">501</span><div id="minus2" class="minus-score"></div></div>
    </div>

    <div id="reaction-msg"></div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const sectors = [20, 1, 18, 4, 13, 6, 10, 15, 2, 17, 3, 19, 7, 16, 8, 11, 14, 9, 12, 5];

        let w, h, cx, cy, rad;
        let p1Score = 501, p2Score = 501, myTurn = true, dartsLeft = 3;
        let peer, conn, isAI = false, isHost = true;
        let dart = { x: 0, y: 0, flying: false, tx: 0, ty: 0, scale: 2.5 };
        let remoteAim = { x: 0, y: 0, active: false };
        let localAim = { x: 0, y: 0, active: false };
        let hits = [];

        function initPeer() {
            peer = new Peer();
            peer.on('open', id => document.getElementById('my-id').innerText = id);
            peer.on('connection', c => { conn = c; isHost = true; setupConn(); startGame(); });
        }

        function connectToPeer() {
            const id = document.getElementById('peer-id-input').value;
            conn = peer.connect(id);
            isHost = false;
            setupConn();
            startGame();
        }

        function setupConn() {
            conn.on('data', data => {
                if (data.type === 'aim') { remoteAim = { x: data.x, y: data.y, active: data.active }; }
                if (data.type === 'fire') { 
                    dart.flying = true; dart.tx = data.tx; dart.ty = data.ty;
                    remoteAim.active = false;
                }
            });
        }

        function startGame() {
            document.getElementById('lobby').style.display = 'none';
            document.getElementById('game-ui').style.display = 'flex';
            myTurn = isHost;
            initCanvas();
            animate();
        }

        function startAI() { isAI = true; isHost = true; startGame(); }

        function initCanvas() {
            w = window.innerWidth; h = window.innerHeight;
            canvas.width = w; canvas.height = h;
            cx = w / 2; cy = h * 0.45; rad = Math.min(w, h) * 0.35;
            resetDart();
        }

        function resetDart() {
            dart.x = w / 2; dart.y = h - 120; dart.scale = 2.5; dart.flying = false;
        }

        function drawBoard() {
            // Siyah Dış Kenar
            ctx.fillStyle = "black";
            ctx.beginPath(); ctx.arc(cx, cy, rad * 1.15, 0, Math.PI * 2); ctx.fill();

            sectors.forEach((num, i) => {
                const a = (i * 18 - 99) * Math.PI / 180;
                const next = a + (18 * Math.PI / 180);
                
                // Ana Dilimler
                ctx.beginPath();
                ctx.moveTo(cx, cy);
                ctx.arc(cx, cy, rad, a, next);
                ctx.fillStyle = (i % 2 === 0) ? "#1a1a1a" : "#eee";
                ctx.fill();
                ctx.strokeStyle = "#444";
                ctx.stroke();

                // Double Halkası (Dış)
                drawRing(rad * 0.92, rad, a, next, (i % 2 === 0) ? "#27ae60" : "#e74c3c");
                
                // Triple Halkası (İç)
                drawRing(rad * 0.55, rad * 0.63, a, next, (i % 2 === 0) ? "#27ae60" : "#e74c3c");

                // Sayılar (Siyah zeminde beyaz yazı)
                ctx.fillStyle = "white";
                ctx.font = `bold ${rad * 0.12}px Arial`;
                ctx.textAlign = "center";
                ctx.textBaseline = "middle";
                const labelX = cx + Math.cos(a + 0.15) * (rad * 1.08);
                const labelY = cy + Math.sin(a + 0.15) * (rad * 1.08);
                ctx.fillText(num, labelX, labelY);
            });

            // Bullseye
            ctx.beginPath(); ctx.arc(cx, cy, rad * 0.12, 0, 7); ctx.fillStyle = "#27ae60"; ctx.fill(); ctx.stroke();
            ctx.beginPath(); ctx.arc(cx, cy, rad * 0.05, 0, 7); ctx.fillStyle = "#e74c3c"; ctx.fill(); ctx.stroke();
        }

        function drawRing(inR, outR, s, e, col) {
            ctx.beginPath();
            ctx.arc(cx, cy, outR, s, e);
            ctx.arc(cx, cy, inR, e, s, true);
            ctx.fillStyle = col;
            ctx.fill();
            ctx.strokeStyle = "#222";
            ctx.lineWidth = 1;
            ctx.stroke();
        }

        function showReaction(pts) {
            const msg = document.getElementById('reaction-msg');
            if (pts <= 10) { msg.innerText = "Ahh! Kaçırdın!"; msg.style.color = "#e74c3c"; }
            else if (pts < 40) { msg.innerText = "Bir dahaki sefere!"; msg.style.color = "#3498db"; }
            else { msg.innerText = "MÜKEMMEL!"; msg.style.color = "#27ae60"; }
            setTimeout(() => msg.innerText = "", 1500);
        }

        function applyScore(pts, isDouble) {
            let targetP = myTurn ? "1" : "2";
            let score = myTurn ? p1Score : p2Score;
            
            if (score - pts === 0 && isDouble) score = 0;
            else if (score - pts <= 1) { pts = 0; } 
            else score -= pts;

            const minusEl = document.getElementById('minus' + targetP);
            minusEl.innerText = "-" + pts;
            minusEl.style.opacity = 1;
            
            setTimeout(() => {
                if (myTurn) p1Score = score; else p2Score = score;
                minusEl.style.opacity = 0;
                updateUI();
            }, 800);

            showReaction(pts);
        }

        function checkScore(x, y) {
            const dx = x - cx, dy = y - cy, d = Math.sqrt(dx*dx + dy*dy);
            let pts = 0, isD = false;
            if (d < rad) {
                if (d < rad * 0.05) { pts = 50; isD = true; }
                else if (d < rad * 0.12) pts = 25;
                else {
                    let a = Math.atan2(dy, dx) * 180 / Math.PI + 99;
                    if (a < 0) a += 360;
                    pts = sectors[Math.floor(a / 18) % 20];
                    if (d > rad * 0.92) { pts *= 2; isD = true; }
                    else if (d > rad * 0.55 && d < rad * 0.63) pts *= 3;
                }
                hits.push({x, y});
            }
            applyScore(pts, isD);
            dartsLeft--;
            if (dartsLeft === 0) {
                setTimeout(() => { myTurn = !myTurn; dartsLeft = 3; hits = []; updateUI(); if(isAI && !myTurn) aiTurn(); }, 1500);
            } else if (isAI && !myTurn) { setTimeout(aiTurn, 1000); }
        }

        function aiTurn() {
            let tx = cx + (Math.random()-0.5)*rad, ty = cy + (Math.random()-0.5)*rad;
            dart.flying = true; dart.tx = tx; dart.ty = ty;
        }

        let sx, sy, isDrag = false;
        canvas.addEventListener('pointerdown', e => { if(!myTurn || dart.flying) return; isDrag = true; sx = e.clientX; sy = e.clientY; });
        window.addEventListener('pointermove', e => {
            if(!isDrag) return;
            dart.x = e.clientX; dart.y = e.clientY;
            localAim.x = e.clientX + (e.clientX - sx) * 1.5;
            localAim.y = e.clientY - (sy - e.clientY) * 2.5;
            localAim.active = true;
            if(conn) conn.send({type: 'aim', x: localAim.x, y: localAim.y, active: true});
        });
        window.addEventListener('pointerup', e => {
            if(!isDrag) return; isDrag = false; localAim.active = false;
            if (sy - e.clientY > 60) {
                dart.flying = true; dart.tx = localAim.x; dart.ty = localAim.y;
                if(conn) conn.send({type: 'fire', tx: dart.tx, ty: dart.ty});
            } else resetDart();
            if(conn) conn.send({type: 'aim', active: false});
        });

        function updateUI() {
            document.getElementById('s1').innerText = p1Score;
            document.getElementById('s2').innerText = p2Score;
            document.getElementById('p1-box').className = "player" + (myTurn ? " active" : "");
            document.getElementById('p2-box').className = "player" + (!myTurn ? " active" : "");
        }

        function animate() {
            ctx.clearRect(0, 0, w, h);
            drawBoard();
            
            // Atış İzleri
            hits.forEach(h => { ctx.beginPath(); ctx.arc(h.x, h.y, 5, 0, 7); ctx.fillStyle = "cyan"; ctx.fill(); });
            
            // Nişangahlar
            if (localAim.active) { ctx.beginPath(); ctx.arc(localAim.x, localAim.y, 15, 0, 7); ctx.strokeStyle = "yellow"; ctx.lineWidth=2; ctx.stroke(); }
            if (remoteAim.active) { ctx.beginPath(); ctx.arc(remoteAim.x, remoteAim.y, 15, 0, 7); ctx.strokeStyle = "red"; ctx.lineWidth=2; ctx.stroke(); }

            if (dart.flying) {
                dart.x += (dart.tx - dart.x) * 0.15; dart.y += (dart.ty - dart.y) * 0.15; dart.scale -= 0.08;
                if (dart.scale < 0.7) { checkScore(dart.x, dart.y); resetDart(); }
            }
            
            // Oku Çiz
            ctx.save(); ctx.translate(dart.x, dart.y); ctx.scale(dart.scale, dart.scale);
            ctx.fillStyle = "gold"; ctx.fillRect(-3, 0, 6, 20);
            ctx.fillStyle = "red"; ctx.beginPath(); ctx.moveTo(0, 5); ctx.lineTo(-12, 25); ctx.lineTo(12, 25); ctx.fill();
            ctx.restore();
            requestAnimationFrame(animate);
        }

        initPeer(); window.addEventListener('resize', initCanvas);
    </script>
</body>
</html>
