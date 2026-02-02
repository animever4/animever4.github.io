<?php
// 1. TMDB Settings
$tmdbKey = "eef0f9c16ca61d6c843e7d43ddc5e1d8";
$animeID = $_GET['id'] ?? '';

// Default values agar link galat ho
$title = "AnimeVerse - Watch Online";
$desc = "Watch your favorite anime in Hindi dubbed for free in high quality.";
$poster = "https://your-website.com/default-banner.jpg"; 

// 2. Fetch Data for Telegram Preview (Server-side)
if (!empty($animeID)) {
    $searchQuery = str_replace('-', ' ', $animeID);
    $apiUrl = "https://api.themoviedb.org/3/search/multi?api_key=$tmdbKey&query=" . urlencode($searchQuery);
    
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $apiUrl);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    $response = curl_exec($ch);
    curl_close($ch);
    
    $data = json_decode($response, true);
    if (!empty($data['results'][0])) {
        $item = $data['results'][0];
        $title = ($item['name'] ?? $item['title']) . " Hindi Dubbed - AnimeVerse";
        $desc = $item['overview'] ?? $desc;
        $poster = "https://image.tmdb.org/t/p/w500" . ($item['poster_path'] ?? '');
    }
}
?>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <title><?php echo $title; ?></title>
    <meta name="description" content="<?php echo $desc; ?>">
    <meta property="og:title" content="<?php echo $title; ?>">
    <meta property="og:description" content="<?php echo $desc; ?>">
    <meta property="og:image" content="<?php echo $poster; ?>">
    <meta property="og:type" content="video.other">
    <meta name="twitter:card" content="summary_large_image">

    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@600&family=Poppins:wght@300;500&display=swap" rel="stylesheet">
    <style>
        :root { --neon: #f47e21; --bg: #0b0e11; --card: #161b22; --glow: rgba(244, 126, 33, 0.3); }
        body { background: var(--bg); color: #fff; font-family: 'Poppins', sans-serif; margin: 0; padding: 0; }
        header { display: flex; justify-content: space-between; align-items: center; padding: 15px 20px; background: rgba(13, 22, 33, 0.9); border-bottom: 1px solid #333; }
        .logo-box { font-family: 'Orbitron'; color: var(--neon); font-size: 1.2rem; }
        .back-btn { background: #1a263f; color: #fff; border: 1px solid #333; padding: 5px 15px; border-radius: 5px; cursor: pointer; }
        .main-container { max-width: 900px; margin: auto; padding: 15px; }
        .title-container { text-align: center; background: var(--card); padding: 10px; border-radius: 8px; border: 1px solid #333; margin-bottom: 15px; }
        .anime-display-title { font-family: 'Orbitron'; color: var(--neon); font-size: 14px; margin: 0; }
        .info-section { display: grid; grid-template-columns: 120px 1fr; gap: 15px; margin-bottom: 20px; }
        .poster-box img { width: 100%; border-radius: 8px; border: 1px solid var(--neon); box-shadow: 0 0 10px var(--glow); background: #111; }
        .about-box { background: var(--card); padding: 12px; border-radius: 8px; border: 1px solid #333; font-size: 12px; color: #ccc; }
        #video-area { position: relative; width: 100%; aspect-ratio: 16/9; background: #000; border-radius: 12px; overflow: hidden; border: 1px solid var(--neon); margin-bottom: 10px; }
        iframe { width: 100%; height: 100%; border: none; }
        #error-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.9); display: none; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 20px; }
        .controls-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .custom-dropdown { position: relative; width: 160px; z-index: 100; }
        .selected-option { background: var(--card); border: 1px solid var(--neon); padding: 10px; border-radius: 8px; cursor: pointer; font-size: 12px; display: flex; justify-content: space-between; }
        .options-list { position: absolute; top: 110%; left: 0; width: 100%; background: var(--card); border: 1px solid var(--neon); border-radius: 8px; max-height: 0; opacity: 0; transition: 0.3s; overflow: hidden; }
        .options-list.active { max-height: 200px; opacity: 1; overflow-y: auto; }
        .option-item { padding: 10px; cursor: pointer; border-bottom: 1px solid #333; font-size: 12px; }
        .download-btn { padding: 8px 15px; background: var(--neon); color: #000; border-radius: 5px; text-decoration: none; font-weight: bold; font-size: 11px; }
        #ep-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(50px, 1fr)); gap: 10px; }
        .ep-card { background: #1a263f; padding: 12px 0; text-align: center; border-radius: 8px; cursor: pointer; font-size: 13px; }
        .ep-card.active { background: var(--neon); color: #000; font-weight: bold; }
        .tg-link { display: block; text-align: center; background: #0088cc; color: #fff; padding: 12px; border-radius: 10px; text-decoration: none; margin: 25px 0; }
    </style>
</head>
<body>

    <header>
        <div class="logo-box">AnimeVerse</div>
        <button class="back-btn" onclick="history.back()">← BACK</button>
    </header>

    <div class="main-container">
        <div class="title-container">
            <h2 class="anime-display-title" id="display-title"><?php echo strtoupper(str_replace('-', ' ', $animeID)); ?></h2>
        </div>

        <div class="info-section">
            <div class="poster-box"><img id="anime-poster" src="<?php echo $poster; ?>" alt="Poster"></div>
            <div class="about-box">
                <h3>About</h3>
                <p id="anime-desc"><?php echo $desc; ?></p>
            </div>
        </div>

        <div id="video-area">
            <div id="error-overlay">
                <p style="color:var(--neon); font-weight:bold;">Wait for Update</p>
                <span style="font-size:12px; color:#bbb;">Iska link jald hi update hoga.</span>
            </div>
            <iframe id="main-player" src="" allowfullscreen></iframe>
        </div>

        <div class="controls-row">
            <div class="custom-dropdown">
                <div class="selected-option" onclick="document.getElementById('drop-menu').classList.toggle('active')">
                    <span id="current-season">Season 1</span>
                    <span>▼</span>
                </div>
                <div class="options-list" id="drop-menu"></div>
            </div>
            <a id="dl-link" href="#" class="download-btn" target="_blank">📥 DOWNLOAD</a>
        </div>

        <div id="ep-grid"></div>
        <a href="https://t.me/animeverse648" class="tg-link">Join our Telegram Channel</a>
    </div>

    <script>
        const csvURL = "https://docs.google.com/spreadsheets/d/1VLcid14lgDfdDvYzfJLpLsbgD_jpY1CGzutgpEGhZdA/export?format=csv";
        let animeData = [];

        async function init() {
            const params = new URLSearchParams(window.location.search);
            const animeID = params.get('id');
            if(!animeID) return;

            const res = await fetch(csvURL);
            const text = await res.text();
            const rows = text.split('\n').slice(1);
            
            animeData = rows.map(r => {
                const c = r.split(',');
                return { id: c[0]?.trim(), name: c[1]?.trim(), url: c[2]?.trim(), bot: c[3]?.trim() };
            }).filter(item => item.id === animeID);

            const seasons = [...new Set(animeData.map(item => item.name.split(' - ')[0]))];
            document.getElementById('drop-menu').innerHTML = seasons.map(s => 
                `<div class="option-item" onclick="selectSeason('${s}')">${s.replace('S', 'Season ')}</div>`
            ).join('');
            
            if(seasons.length > 0) selectSeason(seasons[0]);
        }

        function selectSeason(season) {
            document.getElementById('current-season').innerText = season.replace('S', 'Season ');
            document.getElementById('drop-menu').classList.remove('active');
            const grid = document.getElementById('ep-grid');
            grid.innerHTML = "";
            
            const filtered = animeData.filter(ep => ep.name.startsWith(season));

            filtered.forEach((ep, i) => {
                const btn = document.createElement('div');
                btn.className = 'ep-card';
                btn.innerText = i + 1;
                btn.onclick = () => {
                    const player = document.getElementById('main-player');
                    const errorOverlay = document.getElementById('error-overlay');
                    
                    if (!ep.url || ep.url.length < 5) {
                        player.style.display = "none";
                        errorOverlay.style.display = "flex";
                    } else {
                        player.style.display = "block";
                        errorOverlay.style.display = "none";
                        player.src = ep.url;
                    }

                    document.getElementById('dl-link').href = `https://t.me/verse1_8bot?start=${ep.bot}`;
                    document.querySelectorAll('.ep-card').forEach(c => c.classList.remove('active'));
                    btn.classList.add('active');
                };
                grid.appendChild(btn);
            });
            if(grid.firstChild) grid.firstChild.click();
        }

        init();
    </script>
</body>
</html>