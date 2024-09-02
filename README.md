<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cloudy Comics</title>
    <link href="https://fonts.googleapis.com/css2?family=Lobster&display=swap" rel="stylesheet">
    <style>
        body, html {
            margin: 0;
            padding: 0;
            font-family: 'Lobster', cursive;
            background-color: #000000;
        }
        .container {
            width: 80%;
            margin: auto;
            padding-top: 20px;
        }
        header {
            background-color: #ed586e; /* Updated color */
            color: #fff;
            text-align: center;
            padding: 10px 0;
        }
        h1 {
            font-family: 'Lobster', cursive; /* Apply Lobster font to the title */
        }
        .episode-list {
            margin-top: 20px;
        }
        .episode-item {
            display: flex;
            align-items: center;
            background: #000000;
            padding: 15px;
            margin-bottom: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            cursor: pointer;
            transition: opacity 0.3s;
        }
        .episode-item img {
            width: 60px;
            height: 60px;
            object-fit: cover;
            margin-right: 10px;
            border-radius: 5px;
        }
        .episode-item p {
            margin: 0;
            font-size: 16px;
            color: #ffffff;
        }
        .dull {
            opacity: 0.5;
        }
        .form-container {
            background: #000000;
            padding: 15px;
            margin-top: 20px;
            border: 1px solid #ddd;
            border-radius: 5px;
            color: #ffffff;
        }
        .form-container input,
        .form-container button {
            margin-bottom: 10px;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            width: 100%;
            box-sizing: border-box;
        }
        #episode-title {
            background-color: #000000;
            color: #ffffff;
        }
        .form-container button {
            background-color: #ed586e; /* Updated color */
            color: #fff;
            border: none;
            cursor: pointer;
        }
        .modal {
            display: none;
            position: fixed;
            z-index: 1;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            overflow: hidden;
        }
        .modal-content {
            position: relative;
            margin: auto;
            width: 100%;
            max-width: 100%;
            height: 100%;
            background-color: #fefefe;
            border: 1px solid #888;
            border-radius: 5px;
            overflow-y: auto;
        }
        .modal-content img {
            width: 100%;
            height: auto;
        }
        .modal-footer {
            position: sticky;
            bottom: 0;
            background-color: #000000;
            border-top: 1px solid #ddd;
            padding: 10px;
            text-align: center;
        }
        .modal-footer button {
            margin: 10px;
            padding: 10px 20px;
            background-color: #ed586e; /* Updated color */
            color: #fff;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .close {
            color: #aaa;
            position: absolute;
            top: 10px;
            right: 25px;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close:hover,
        .close:focus {
            color: #000;
            text-decoration: none;
        }
    </style>
</head>
<body>
    <header>
        <h1>Cloudy Comics</h1>
    </header>
    <div class="container">
        <div class="episode-list" id="episode-list">
            <!-- Episodes will be added here -->
        </div>
        <div class="form-container">
            <h2>Add New Episode</h2>
            <form id="add-episode-form">
                <input type="text" id="episode-title" name="episode-title" placeholder="Episode Title" required>
                <input type="file" id="episode-image" name="episode-image" accept=".jpg, .jpeg" required>
                <input type="file" id="thumbnail-image" name="thumbnail-image" accept=".jpg, .jpeg" required>
                <button type="submit">Add Episode</button>
            </form>
        </div>
    </div>

    <!-- Modal for displaying episodes -->
    <div id="myModal" class="modal">
        <div class="modal-content">
            <span class="close">&times;</span>
            <img id="modal-image" src="" alt="Episode Image">
            <div class="modal-footer">
                <button id="previous-episode">Previous Episode</button>
                <button id="next-episode">Next Episode</button>
                <button id="back-home">Back to Homepage</button>
            </div>
        </div>
    </div>

    <script>
        let currentEpisodeIndex = null;
        const episodeList = JSON.parse(localStorage.getItem('episodeList')) || [];

        document.getElementById('add-episode-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const title = document.getElementById('episode-title').value;
            const episodeImageFile = document.getElementById('episode-image').files[0];
            const thumbnailImageFile = document.getElementById('thumbnail-image').files[0];

            const episodeImageReader = new FileReader();
            const thumbnailImageReader = new FileReader();

            episodeImageReader.onload = function(e) {
                const episodeImageBase64 = e.target.result;
                thumbnailImageReader.onload = function(e) {
                    const thumbnailImageBase64 = e.target.result;
                    const episode = { title, episodeImageBase64, thumbnailImageBase64, viewed: false };
                    episodeList.push(episode);
                    localStorage.setItem('episodeList', JSON.stringify(episodeList));
                    addEpisodeToList(episodeList.length - 1, episode);
                };
                thumbnailImageReader.readAsDataURL(thumbnailImageFile);
            };
            episodeImageReader.readAsDataURL(episodeImageFile);

            document.getElementById('add-episode-form').reset();
        });

        function addEpisodeToList(index, episode) {
            const episodeListContainer = document.getElementById('episode-list');
            const newEpisode = document.createElement('div');
            newEpisode.className = 'episode-item' + (episode.viewed ? ' dull' : '');
            newEpisode.dataset.index = index;
            newEpisode.innerHTML = `<img src="${episode.thumbnailImageBase64}" alt="Episode Thumbnail"><p>${episode.title}</p>`;
            episodeListContainer.appendChild(newEpisode);
        }

        function updateEpisodeList() {
            const episodeListContainer = document.getElementById('episode-list');
            episodeListContainer.innerHTML = '';
            episodeList.forEach((episode, index) => {
                addEpisodeToList(index, episode);
            });
        }

        const modal = document.getElementById("myModal");
        const img = document.getElementById("modal-image");
        const span = document.getElementsByClassName("close")[0];
        const previousEpisodeBtn = document.getElementById("previous-episode");
        const nextEpisodeBtn = document.getElementById("next-episode");
        const backHomeBtn = document.getElementById("back-home");

        document.getElementById('episode-list').addEventListener('click', function(event) {
            if (event.target.closest('.episode-item')) {
                currentEpisodeIndex = parseInt(event.target.closest('.episode-item').dataset.index);
                const episode = episodeList[currentEpisodeIndex];
                img.src = episode.episodeImageBase64;
                modal.style.display = "block";
                document.body.style.overflow = 'hidden';
                document.documentElement.style.overflow = 'hidden';
                episode.viewed = true; // Mark as viewed
                localStorage.setItem('episodeList', JSON.stringify(episodeList)); // Save the updated state
                event.target.closest('.episode-item').classList.add('dull'); // Dull the episode item after viewing
            }
        });

        span.onclick = function() {
            modal.style.display = "none";
            document.body.style.overflow = '';
            document.documentElement.style.overflow = '';
        }

        window.onclick = function(event) {
            if (event.target == modal) {
                modal.style.display = "none";
                document.body.style.overflow = '';
                document.documentElement.style.overflow = '';
            }
        }

        previousEpisodeBtn.onclick = function() {
            if (currentEpisodeIndex !== null && currentEpisodeIndex > 0) {
                currentEpisodeIndex--;
                const episode = episodeList[currentEpisodeIndex];
                img.src = episode.episodeImageBase64;
            }
        }

        nextEpisodeBtn.onclick = function() {
            if (currentEpisodeIndex !== null && currentEpisodeIndex < episodeList.length - 1) {
                currentEpisodeIndex++;
                const episode = episodeList[currentEpisodeIndex];
                img.src = episode.episodeImageBase64;
            }
        }

        backHomeBtn.onclick = function() {
            modal.style.display = "none";
            document.body.style.overflow = '';
            document.documentElement.style.overflow = '';
        }

        // Initialize the episode list on page load
        updateEpisodeList();
    </script>
</body>
</html>
