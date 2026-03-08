<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Retro Experience TV | Open Source Project</title>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        :root {
            --primary: #ffcc00;
            --bg: #0f0f0f;
            --glass: rgba(255, 255, 255, 0.05);
            --border: rgba(255, 255, 255, 0.1);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', sans-serif; }

        body {
            background: var(--bg);
            color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }

        /* Scanlines Effect */
        body::before {
            content: "";
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.1) 50%);
            background-size: 100% 4px;
            z-index: 10;
            pointer-events: none;
        }

        .container {
            width: 95%;
            max-width: 1100px;
            display: grid;
            grid-template-columns: 1fr 350px;
            gap: 20px;
            z-index: 2;
        }

        @media (max-width: 900px) {
            .container { grid-template-columns: 1fr; height: 95vh; overflow-y: auto; }
            body { overflow-y: auto; }
        }

        /* Video Player Section */
        .player-section {
            background: #000;
            border-radius: 20px;
            border: 1px solid var(--border);
            overflow: hidden;
            position: relative;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            display: flex;
            flex-direction: column;
        }

        video { width: 100%; flex: 1; background: #000; }

        .player-footer {
            padding: 15px;
            background: var(--glass);
            backdrop-filter: blur(10px);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        /* Sidebar Section */
        .sidebar {
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .card {
            background: var(--glass);
            backdrop-filter: blur(15px);
            border: 1px solid var(--border);
            border-radius: 20px;
            padding: 20px;
        }

        .channel-list {
            max-height: 300px;
            overflow-y: auto;
            margin-top: 10px;
        }

        .channel-item {
            padding: 12px;
            border-radius: 10px;
            margin-bottom: 8px;
            cursor: pointer;
            transition: 0.3s;
            border: 1px solid transparent;
            font-size: 14px;
        }

        .channel-item:hover {
            background: rgba(255, 204, 0, 0.1);
            border-color: var(--primary);
            color: var(--primary);
        }

        .active-ch { background: var(--primary) !important; color: #000 !important; font-weight: bold; }

        .btn-link {
            text-decoration: none;
            color: var(--primary);
            font-size: 13px;
            font-weight: 600;
            margin-right: 15px;
        }

        .btn-link:hover { text-decoration: underline; }

        h3 { font-size: 16px; text-transform: uppercase; letter-spacing: 1px; color: var(--primary); }

        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-thumb { background: var(--primary); border-radius: 10px; }
    </style>
</head>
<body>

<div class="container">
    <main class="player-section">
        <video id="videoPlayer" autoplay playsinline controls></video>
        <div class="player-footer">
            <div>
                <span id="currentTitle" style="font-weight: bold;">SELECIONE UM CANAL</span>
                <p style="font-size: 10px; color: #888;">LIVE STREAMING HLS</p>
            </div>
            <div style="text-align: right;">
                <span style="color: red; animation: blink 1s infinite;">●</span> <small>REC</small>
            </div>
        </div>
    </main>

    <aside class="sidebar">
        <div class="card">
            <h3>📡 Canais Disponíveis</h3>
            <div class="channel-list" id="list"></div>
        </div>

        <div class="card">
            <h3>👤 Desenvolvedor</h3>
            <p style="font-size: 14px; margin: 10px 0; color: #ccc;">Jackson Matos</p>
            <div style="margin-top: 10px;">
                <a href="https://github.com/ytjackson" target="_blank" class="btn-link">GITHUB</a>
                <a href="https://instagram.com/jacksonmatosbr" target="_blank" class="btn-link">INSTAGRAM</a>
            </div>
        </div>

        <div class="card" style="font-size: 11px; color: #777;">
            <p>© 2026 Retro TV Project. Todos os direitos reservados. Este projeto é open-source para fins educacionais.</p>
        </div>
    </aside>
</div>

<script>
    const video = document.getElementById('videoPlayer');
    const hls = new Hls();
    const title = document.getElementById('currentTitle');

    const channels = [
        { n: "TOP TV", u: "https://isaocorp.cloudecast.com/toptv/index.m3u8" },
        { n: "RETRO TV", u: "https://stmv1.video.brstream.com.br/retrotv/retrotv/playlist.m3u8" },
        { n: "DREAMWORKS", u: "https://cdn-3.nxplay.com.br/DREAMWORKS/index.m3u8?token=ff3ca6a3-1f5e-4ad2-a2fc-609daadc8b88" },
        { n: "CARTOON NETWORK", u: "https://cdn-3.nxplay.com.br/CARTOON_NETWORK_TK/index.m3u8?token=ff3ca6a3-1f5e-4ad2-a2fc-609daadc8b88" },
        { n: "LOADING TV", u: "https://stmv1.srvif.com/loadingtv/loadingtv/playlist.m3u8" }
    ];

    const listDiv = document.getElementById('list');
    channels.forEach((ch, i) => {
        const item = document.createElement('div');
        item.className = 'channel-item';
        item.innerText = `${i+1}. ${ch.n}`;
        item.onclick = () => {
            document.querySelectorAll('.channel-item').forEach(el => el.classList.remove('active-ch'));
            item.classList.add('active-ch');
            title.innerText = ch.n;
            if (Hls.isSupported()) {
                hls.loadSource(ch.u);
                hls.attachMedia(video);
                hls.on(Hls.Events.MANIFEST_PARSED, () => video.play());
            }
        };
        listDiv.appendChild(item);
    });

    const style = document.createElement('style');
    style.innerHTML = "@keyframes blink { 0% {opacity:1} 50% {opacity:0} 100% {opacity:1} }";
    document.head.appendChild(style);
</script>

</body>
</html>
