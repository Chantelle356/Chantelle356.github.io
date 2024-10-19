<!doctype html>
<html lang="en">
<head>
    <link rel="manifest" href="manifest.json">
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
            background-color: #ed586e;
            color: #fff;
            text-align: center;
            padding: 10px 0;
            position: relative;
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
            background-color: #ed586e;
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
            display: block;
            margin-bottom: 10px;
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
            background-color: #ed586e;
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
        #delete-mode-btn {
            background-color: #ed586e;
            color: #fff;
            border: none;
            padding: 10px;
            cursor: pointer;
            position: absolute;
            top: 10px;
            left: 10px;
            border-radius: 5px;
        }
        #delete-mode-btn.delete-active {
            background-color: red;
        }
    </style>
</head>
<body>
    <header>
        <h1>Cloudy Comics</h1>
        <button id="delete-mode-btn">Delete Mode</button>
    </header>
    <div class="container">
        <div class="episode-list" id="episode-list">
            <!-- Episodes will be added here -->
        </div>
        <div class="form-container">
            <h2>Add New Episode</h2>
            <form id="add-episode-form">
                <input type="text" id="episode-title" name="episode-title" placeholder="Episode Title" required>
                <input type="file" id="episode-image" name="episode-image" accept=".jpg, .jpeg" multiple required>
                <input type="file" id="thumbnail-image" name="thumbnail-image" accept=".jpg, .jpeg" required>
                <button type="submit">Add Episode</button>
            </form>
        </div>
    </div>

    <!-- Modal for displaying episodes -->
    <div id="myModal" class="modal">
        <div class="modal-content">
            <span class="close">&times;</span>
            <div id="modal-image-container"></div>
            <div class="modal-footer">
                <button id="previous-episode">Previous Episode</button>
                <button id="next-episode">Next Episode</button>
                <button id="back-home">Back to Homepage</button>
            </div>
        </div>
    </div>

    <script>
        let currentEpisodeIndex = null;
        const episodeList = JSON.parse(localStorage.getItem('episodes')) || [];
        let deleteMode = false;

        function saveEpisodes() {
            localStorage.setItem('episodes', JSON.stringify(episodeList));
        }

        function convertFileToBase64(file, callback) {
            const reader = new FileReader();
            reader.onloadend = function() {
                callback(reader.result);
            };
            reader.readAsDataURL(file);
        }

        function addEpisodeToList(title, thumbnailImageUrl, isDull) {
            const episodeListContainer = document.getElementById('episode-list');
            const newEpisode = document.createElement('div');
            newEpisode.className = 'episode-item';
            newEpisode.dataset.index = episodeList.length - 1;
            newEpisode.innerHTML = `<img src="${thumbnailImageUrl}" alt="Episode Thumbnail"><p>${title}</p>`;
            if (isDull) {
                newEpisode.classList.add('dull');
            }
            episodeListContainer.appendChild(newEpisode);
        }

        function updateEpisodeList() {
            const episodeListContainer = document.getElementById('episode-list');
            episodeListContainer.innerHTML = '';
            episodeList.forEach((episode, index) => {
                addEpisodeToList(episode.title, episode.thumbnailImageUrl, episode.isDull);
            });
        }

        document.getElementById('add-episode-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const title = document.getElementById('episode-title').value;
            const episodeImageFiles = document.getElementById('episode-image').files;
            const thumbnailImageFile = document.getElementById('thumbnail-image').files[0];

            const episodeImages = [];

            for (let i = 0; i < episodeImageFiles.length; i++) {
                convertFileToBase64(episodeImageFiles[i], (imageBase64) => {
                    episodeImages.push(imageBase64);

                    if (episodeImages.length === episodeImageFiles.length) {
                        convertFileToBase64(thumbnailImageFile, (thumbnailImageUrl) => {
                            episodeList.push({ title, episodeImages, thumbnailImageUrl, isDull: false });
                            addEpisodeToList(title, thumbnailImageUrl, false);
                            saveEpisodes();
                            document.getElementById('add-episode-form').reset();
                        });
                    }
                });
            }
        });

        document.getElementById('delete-mode-btn').addEventListener('click', function() {
            deleteMode = !deleteMode;
            this.classList.toggle('delete-active', deleteMode);
        });

        document.getElementById('episode-list').addEventListener('click', function(event) {
            const episodeItem = event.target.closest('.episode-item');
            if (!episodeItem) return;

            if (deleteMode) {
                const episodeIndex = parseInt(episodeItem.dataset.index);
                episodeList.splice(episodeIndex, 1);
                saveEpisodes();
                updateEpisodeList();
            } else {
                currentEpisodeIndex = parseInt(episodeItem.dataset.index);
                const episode = episodeList[currentEpisodeIndex];
                displayEpisodeImages(episode);
                document.getElementById('myModal').style.display = "block";
                document.body.style.overflow = 'hidden';
                document.documentElement.style.overflow = 'hidden';
                episodeItem.classList.add('dull');
                episodeList[currentEpisodeIndex].isDull = true;
                saveEpisodes();
            }
        });

        function displayEpisodeImages(episode) {
            const modalImageContainer = document.getElementById('modal-image-container');
            modalImageContainer.innerHTML = '';
            episode.episodeImages.forEach(imageUrl => {
                const img = document.createElement('img');
                img.src = imageUrl;
                modalImageContainer.appendChild(img);
            });
        }

        document.querySelector('.close').addEventListener('click', function() {
            closeModal();
        });

        document.getElementById('back-home').addEventListener('click', function() {
            closeModal();
        });

        function closeModal() {
            document.getElementById('myModal').style.display = "none";
            document.body.style.overflow = 'auto';
            document.documentElement.style.overflow = 'auto';
            if (currentEpisodeIndex !== null) {
                const episodeItem = document.querySelector(`.episode-item[data-index="${currentEpisodeIndex}"]`);
                if (episodeItem) {
                    episodeItem.classList.remove('dull');
                }
            }
        }

        document.getElementById('previous-episode').addEventListener('click', function() {
            if (currentEpisodeIndex > 0) {
                currentEpisodeIndex--;
                displayEpisodeImages(episodeList[currentEpisodeIndex]);
            }
        });

        document.getElementById('next-episode').addEventListener('click', function() {
            if (currentEpisodeIndex < episodeList.length - 1) {
                currentEpisodeIndex++;
                displayEpisodeImages(episodeList[currentEpisodeIndex]);
            }
        });

        // Initialize the episode list on page load
        updateEpisodeList();
    </script>
</body>
</html>
