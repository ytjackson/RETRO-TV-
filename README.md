<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Retro TV OS | Jackson Matos</title>
    
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    
    <style>
        :root {
            --primary: #ffcc00;
            --accent: #00d4ff;
            --bg: #050505;
            --panel: rgba(25, 25, 25, 0.95);
            --border: rgba(255, 255, 255, 0.1);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        body {
            background: var(--bg);
            color: #fff;
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* Scanlines Overlay */
        body::after {
            content: " ";
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.1) 50%);
            background-size: 100% 3px;
            z-index: 9999;
            pointer-events: none;
            opacity: 0.3;
        }

        /* Header */
        header {
            padding: 15px 25px;
            background: #000;
            border-bottom: 1px solid var(--border);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        header h1 { font-size: 16px; letter-spacing: 3px; color: var(--primary); font-weight: 300; }

        /* Main Layout */
        main {
            flex: 1;
            display: grid;
            grid-template-columns: 1fr 320px;
            gap: 1px;
            background: var(--border);
        }

        @media (max-width: 900px) {
            main { grid-template-columns: 1fr; overflow-y: auto; }
            body { overflow-y: auto; }
        }

        /* Player Section */
        .player-container {
            background: #000;
            display: flex;
            flex-direction: column;
            position: relative;
        }

        video { width: 100%; flex: 1; background: #000; outline: none; }

        .video-info {
            padding: 15px;
            background: #111;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-top: 1px solid var(--border);
        }

        /* Sidebar Section */
        .sidebar {
            background: #111;
            display: flex;
            flex-direction: column;
            padding: 20px;
            gap: 20px;
        }

        .channel-list {
            flex: 1;
            overflow-y: auto;
            border: 1px solid var(--border);
            background: #0a0a0a;
            border-radius: 8px;
        }

        .channel-item {
            padding: 15px;
            border-bottom: 1px solid var(--border);
            cursor: pointer;
            transition: all 0.3s;
            font-size: 13px;
            color: #888;
        }

        .channel-item:hover { background: #1a1a1a; color: var(--primary); }
        .channel-item.active { border-left: 4px solid var(--primary); background: #1a1a1a; color: #fff; }

        /* Footer Info */
        .footer-info {
            font-size: 11px;
            line-height: 1.6;
            color: #555;
            padding-top: 15px;
            border-top: 1px solid var(--border);
        }

        .social-links a {
            color: var(--accent);
            text-decoration: none;
            margin-right: 15px;
            font-weight: bold;
        }

        .social-links a:hover { color: #fff; }

        /* Status Blinker */
        .live-tag { color: red; animation: blink 1.5s infinite; font-weight: bold; }
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.2; } 100% { opacity: 1; } }

        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-thumb { background: var(--primary); }
    </style>
</head>
<body>

<header>
    <h1>RETRO TV // SYSTEM v.3.0</h1>
    <div style="font-size: 10px; color: #444;">ID: JACKSON_MATOS_OFFICIAL</div>
</header>

<main>
    <section class="player-container">
        <video id="p" autoplay playsinline controls></video>
        <div class="video-info">
            <div>
                <span id="chName" style="color: var(--primary); font-weight: bold;">AGUARDANDO SINAL...</span>
                <div style="font-size: 10px; color: #555; margin-top: 4px;">DECODIFICANDO STREAM HLS V.4</div>
            </div>
            <div class="live-tag">● LIVE</div>
        </div>
    </section>

    <section class="sidebar">
        <div style="font-size: 12px; letter-spacing: 1px; color: #666;">GRADE DE CANAIS</div>
        <div class="channel-list" id="l"></div>

        <div class="footer-info">
            <strong>CRÉDITOS DO PROJETO:</strong><br>
            Desenvolvido por Jackson Matos.<br>
            Interface otimizada para GitHub Pages.<br><br>
            
            <div class="social-links">
                <a href="https://github.com/ytjackson" target="_blank">GITHUB</a>
                <a href="https://instagram.com/jacksonmatosbr" target="_blank">INSTAGRAM</a>
            </div>
            
            <p style="margin-top: 15px; font-style: italic;">
                "Transformando nostalgia em código funcional."
            </p>
        </div>
    </section>
</main>

<script>
    const v = document.getElementById('p');
    const h = new Hls();
    const chName = document.getElementById('chName');

    // BANCO DE DADOS DE CANAIS
    const channels = [
        { n: "TOP TV", u: "https://isaocorp.cloudecast.com/toptv/index.m3u8" },
        { n: "RETRO TV", u: "https://stmv1.video.brstream.com.br/retrotv/retrotv/playlist.m3u8" },
        { n: "DREAMWORKS", u: "https://cdn-3.nxplay.com.br/DREAMWORKS/index.m3u8?token=ff3ca6a3-1f5e-4ad2-a2fc-609daadc8b88" },
        { n: "CARTOON NETWORK", u: "https://cdn-3.nxplay.com.br/CARTOON_NETWORK_TK/index.m3u8?token=ff3ca6a3-1f5e-4ad2-a2fc-609daadc8b88" },
        { n: "LOADING TV", u: "https://stmv1.srvif.com/loadingtv/loadingtv/playlist.m3u8" }
    ];

    // CARREGAMENTO DINÂMICO
    const list = document.getElementById('l');
    channels.forEach((c, index) => {
        let div = document.createElement('div');
        div.className = 'channel-item';
        div.innerHTML = `<strong>0${index + 1}</strong> // ${c.n}`;
        
        div.onclick = () => {
            // UI Update
            document.querySelectorAll('.channel-item').forEach(i => i.classList.remove('active'));
            div.classList.add('active');
            chName.innerText = c.n;

            // Stream Logic
            if (Hls.isSupported()) {
                h.loadSource(c.u);
                h.attachMedia(v);
                h.on(Hls.Events.MANIFEST_PARSED, () => v.play());
            } else if (v.canPlayType('application/vnd.apple.mpegurl')) {
                v.src = c.u;
                v.play();
            }
        };
        list.appendChild(div);
    });

    // Auto-play primeiro canal
    setTimeout(() => list.firstChild.click(), 1000);
</script>

</body>
</html>
