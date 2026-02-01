# moviz
a free movie app
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>StreamLite - Movie App</title>
    <style>
        :root {
            --primary: #e50914;
            --dark: #141414;
            --light: #f5f5f5;
        }

        body {
            font-family: 'Arial', sans-serif;
            background-color: var(--dark);
            color: var(--light);
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* Header */
        header {
            padding: 20px;
            background: linear-gradient(to bottom, rgba(0,0,0,0.7) 10%, rgba(0,0,0,0));
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        h1 { margin: 0; color: var(--primary); letter-spacing: 1px; }

        /* Main Content */
        .container {
            padding: 20px;
            overflow-y: auto;
            flex: 1;
        }

        /* Movie Grid */
        .movie-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 20px;
        }

        .movie-card {
            background-color: #222;
            border-radius: 8px;
            overflow: hidden;
            transition: transform 0.3s;
            cursor: pointer;
            position: relative;
        }

        .movie-card:hover { transform: scale(1.05); }

        .movie-thumb {
            width: 100%;
            height: 300px;
            object-fit: cover;
            background-color: #333;
        }

        .movie-info { padding: 10px; }
        .movie-title { font-weight: bold; font-size: 1.1em; margin-bottom: 5px; }
        .movie-meta { font-size: 0.8em; color: #aaa; display: flex; justify-content: space-between; }

        /* Download Status */
        .download-badge {
            position: absolute;
            top: 10px;
            right: 10px;
            background-color: var(--primary);
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.7em;
            display: none; /* Hidden by default */
        }

        .download-badge.active { display: block; }

        /* Video Player Overlay */
        #player-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: black;
            z-index: 100;
            display: none;
            flex-direction: column;
        }

        video { width: 100%; height: 80%; outline: none; }
        
        .controls {
            padding: 20px;
            display: flex;
            gap: 15px;
            background: #111;
            height: 20%;
            align-items: center;
        }

        button {
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }

        .btn-primary { background-color: var(--primary); color: white; }
        .btn-secondary { background-color: #444; color: white; }
        .btn-close { position: absolute; top: 20px; right: 20px; background: rgba(0,0,0,0.5); color: white; border-radius: 50%; width: 40px; height: 40px; font-size: 20px; z-index: 101; display: flex; align-items: center; justify-content: center; }

        /* Progress Bar */
        .progress-container {
            width: 100%;
            background-color: #333;
            height: 5px;
            margin-top: 10px;
            border-radius: 5px;
            overflow: hidden;
            display: none;
        }
        .progress-bar {
            height: 100%;
            background-color: var(--primary);
            width: 0%;
            transition: width 0.2s;
        }

    </style>
</head>
<body>

    <header>
        <h1>STREAMLITE</h1>
        <div>My Library</div>
    </header>

    <div class="container">
        <h2>Trending Now</h2>
        <div class="movie-grid" id="movieGrid">
            </div>
    </div>

    <div id="player-overlay">
        <button class="btn-close" onclick="closePlayer()">×</button>
        <video id="mainVideo" controls>
            <source src="" type="video/mp4">
            Your browser does not support the video tag.
        </video>
        <div class="controls">
            <div style="flex: 1;">
                <h2 id="playerTitle">Movie Title</h2>
                <div class="progress-container" id="dlProgressContainer">
                    <div class="progress-bar" id="dlProgressBar"></div>
                </div>
                <small id="dlStatusText" style="color: #aaa;"></small>
            </div>
            <button class="btn-secondary" onclick="simulateDownload()" id="btnDownload">⬇ Download</button>
        </div>
    </div>

    <script>
        // DATA: Using Open Source Blender Movies (Public Domain / CC)
        const movies = [
            {
                id: 1,
                title: "Big Buck Bunny",
                year: 2008,
                duration: "10m",
                thumb: "https://peach.blender.org/wp-content/uploads/title_anouncement.jpg?x11217",
                src: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
            },
            {
                id: 2,
                title: "Elephant Dream",
                year: 2006,
                duration: "11m",
                thumb: "https://orange.blender.org/wp-content/themes/orange/images/media/gallery/s1_proog.jpg",
                src: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4"
            },
            {
                id: 3,
                title: "Sintel",
                year: 2010,
                duration: "15m",
                thumb: "https://durian.blender.org/wp-content/themes/durian/images/media/01_sintel.jpg",
                src: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/Sintel.mp4"
            },
            {
                id: 4,
                title: "Tears of Steel",
                year: 2012,
                duration: "12m",
                thumb: "https://mango.blender.org/wp-content/uploads/2013/05/01_thom_celia_bridge.jpg",
                src: "http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/TearsOfSteel.mp4"
            }
        ];

        let currentMovie = null;

        // 1. RENDER MOVIES
        const grid = document.getElementById('movieGrid');
        
        function renderMovies() {
            grid.innerHTML = '';
            movies.forEach(movie => {
                const isDownloaded = localStorage.getItem(`downloaded_${movie.id}`);
                
                const card = document.createElement('div');
                card.className = 'movie-card';
                card.onclick = () => openPlayer(movie);
                card.innerHTML = `
                    <div class="download-badge ${isDownloaded ? 'active' : ''}" id="badge-${movie.id}">Downloaded</div>
                    <img src="${movie.thumb}" class="movie-thumb" alt="${movie.title}">
                    <div class="movie-info">
                        <div class="movie-title">${movie.title}</div>
                        <div class="movie-meta">
                            <span>${movie.year}</span>
                            <span>${movie.duration}</span>
                        </div>
                    </div>
                `;
                grid.appendChild(card);
            });
        }

        // 2. PLAYER LOGIC
        const player = document.getElementById('player-overlay');
        const videoElement = document.getElementById('mainVideo');
        const playerTitle = document.getElementById('playerTitle');
        const btnDownload = document.getElementById('btnDownload');
        const dlProgress = document.getElementById('dlProgressBar');
        const dlContainer = document.getElementById('dlProgressContainer');
        const dlStatus = document.getElementById('dlStatusText');

        function openPlayer(movie) {
            currentMovie = movie;
            playerTitle.innerText = movie.title;
            videoElement.src = movie.src;
            player.style.display = 'flex';
            videoElement.play();

            // Check download status
            const isDownloaded = localStorage.getItem(`downloaded_${movie.id}`);
            resetDownloadUI(isDownloaded);
        }

        function closePlayer() {
            player.style.display = 'none';
            videoElement.pause();
            videoElement.src = ""; // Stop buffering
            renderMovies(); // Refresh grid to show new badges
        }

        // 3. DOWNLOAD SIMULATION
        function resetDownloadUI(isDownloaded) {
            dlContainer.style.display = 'none';
            dlProgress.style.width = '0%';
            if (isDownloaded) {
                btnDownload.innerText = "✓ Saved to Device";
                btnDownload.disabled = true;
                btnDownload.style.backgroundColor = "#2ecc71";
                dlStatus.innerText = "Available for offline playback";
            } else {
                btnDownload.innerText = "⬇ Download";
                btnDownload.disabled = false;
                btnDownload.style.backgroundColor = "#444";
                dlStatus.innerText = "";
            }
        }

        function simulateDownload() {
            if (!currentMovie) return;

            btnDownload.disabled = true;
            btnDownload.innerText = "Downloading...";
            dlContainer.style.display = 'block';
            dlStatus.innerText = "Downloading assets...";

            let width = 0;
            const interval = setInterval(() => {
                if (width >= 100) {
                    clearInterval(interval);
                    finishDownload();
                } else {
                    width += 2; // Speed of simulation
                    dlProgress.style.width = width + '%';
                }
            }, 50);
        }

        function finishDownload() {
            localStorage.setItem(`downloaded_${currentMovie.id}`, 'true');
            btnDownload.innerText = "✓ Saved to Device";
            btnDownload.style.backgroundColor = "#2ecc71";
            dlStatus.innerText = "Download complete.";
        }

        // Initial Render
        renderMovies();

    </script>
</body>
</html>