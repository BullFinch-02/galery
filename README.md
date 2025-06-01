<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Journal de Madrestown</title>
  <style>
    body {
      font-family: 'Courier New', Courier, monospace;
      background-color: #0d0d0d;
      color: #f2f2f2;
      text-align: center;
      padding: 20px;
    }
    .journal {
      border: 4px double #ffd700;
      border-radius: 15px;
      padding: 20px;
      margin: auto;
      max-width: 600px;
      background-color: #1a1a1a;
    }
    .image-container {
      margin: 20px 0;
      position: relative;
    }
    .image-container img {
      max-width: 100%;
      border-radius: 10px;
    }
    .author {
      position: absolute;
      bottom: 8px;
      right: 16px;
      background-color: rgba(0, 0, 0, 0.7);
      padding: 4px 8px;
      border-radius: 5px;
      color: #ffd700;
    }
    .comments {
      margin-top: 20px;
      text-align: left;
    }
    .comment {
      margin-bottom: 10px;
      padding: 10px;
      background-color: #262626;
      border-left: 4px solid #ffd700;
      border-radius: 5px;
    }
    .comment strong {
      color: #ffd700;
    }
    .page-number {
      margin-top: 10px;
      font-size: 18px;
      font-weight: bold;
    }
    button {
      margin: 5px;
      padding: 10px 15px;
      border: none;
      border-radius: 5px;
      background-color: #ffd700;
      color: #000;
      cursor: pointer;
      font-weight: bold;
    }
    input, textarea {
      width: 90%;
      margin-top: 10px;
      padding: 10px;
      border-radius: 5px;
      border: none;
    }
    #drop-zone {
      margin-top: 10px;
      padding: 20px;
      border: 2px dashed #ffd700;
      border-radius: 10px;
      color: #ffd700;
    }
    #drop-zone.dragover {
      background-color: #333;
    }

    #delete-button[disabled] {
      background-color: #999;
      cursor: not-allowed;
    }
  </style>
</head>
<body>
  <h1>Journal de Madrestown</h1>
  <div class="journal">
    <div id="drop-zone">Dépose une image ici</div>
    <input type="text" id="new-author" placeholder="Nom de l'auteur" />
    <button onclick="addImage()">Ajouter</button>

    <div class="image-container">
      <img id="main-image" src="" alt="Image" />
      <div class="author" id="image-author"></div>
    </div>

    <div class="page-number" id="page-number"></div>

    <div>
      <button onclick="prevPage()">←</button>
      <button onclick="nextPage()">→</button>
    </div>

    <div class="comments" id="comments"></div>

    <input type="text" id="comment-name" placeholder="Ton nom" />
    <textarea id="comment-text" placeholder="Ton commentaire"></textarea>
    <button onclick="addComment()">Envoyer</button>

    <button id="delete-button" onclick="deleteImage()" disabled>Supprimer l’image</button>
  </div>

  <script>
    let images = [];
    let comments = {};
    let currentIndex = 0;
    let droppedImage = null;
    let isAdmin = false; // 🔒 Accès admin initialisé à false
    const adminPassword = "admin123"; // 🔒 Change ce mot de passe ici si besoin

    const mainImage = document.getElementById('main-image');
    const imageAuthor = document.getElementById('image-author');
    const pageNumber = document.getElementById('page-number');
    const commentsContainer = document.getElementById('comments');
    const deleteButton = document.getElementById('delete-button'); // 🔒

    function render() {
      if (images.length === 0) {
        mainImage.src = "";
        imageAuthor.textContent = "";
        pageNumber.textContent = "Aucune image";
        commentsContainer.innerHTML = "";
        return;
      }

      const current = images[currentIndex];
      mainImage.src = current.url;
      imageAuthor.textContent = current.author;
      pageNumber.textContent = `Page ${currentIndex + 1} / ${images.length}`;

      const currentComments = comments[current.url] || [];
      commentsContainer.innerHTML = currentComments.map(
        c => `<div class="comment"><strong>${c.name}</strong>: ${c.text}</div>`
      ).join('');
    }

    function addImage() {
      const author = document.getElementById('new-author').value.trim();
      if (!droppedImage || !author) {
        alert("Ajoute une image et un nom d'auteur.");
        return;
      }
      images.push({ url: droppedImage, author });
      currentIndex = images.length - 1;
      comments[droppedImage] = [];
      droppedImage = null;
      document.getElementById('new-author').value = '';
      render();
    }

    function prevPage() {
      if (currentIndex > 0) {
        currentIndex--;
        render();
      }
    }

    function nextPage() {
      if (currentIndex < images.length - 1) {
        currentIndex++;
        render();
      }
    }

    function addComment() {
      const name = document.getElementById('comment-name').value.trim();
      const text = document.getElementById('comment-text').value.trim();
      if (!name || !text) return;

      const url = images[currentIndex]?.url;
      if (!url) return;

      comments[url].push({ name, text });
      document.getElementById('comment-name').value = '';
      document.getElementById('comment-text').value = '';
      render();
    }

    function deleteImage() {
      if (!isAdmin) {
        alert("Action réservée à l'administrateur.");
        return;
      }

      const url = images[currentIndex]?.url;
      if (!url) return;

      images.splice(currentIndex, 1);
      delete comments[url];
      if (currentIndex >= images.length) currentIndex = images.length - 1;
      render();
    }

    const dropZone = document.getElementById('drop-zone');
    dropZone.addEventListener('dragover', e => {
      e.preventDefault();
      dropZone.classList.add('dragover');
    });

    dropZone.addEventListener('dragleave', () => {
      dropZone.classList.remove('dragover');
    });

    dropZone.addEventListener('drop', e => {
      e.preventDefault();
      dropZone.classList.remove('dragover');
      const file = e.dataTransfer.files[0];
      if (file && file.type.startsWith('image/')) {
        const reader = new FileReader();
        reader.onload = () => {
          droppedImage = reader.result;
          dropZone.textContent = "Image prête ! Clique sur Ajouter.";
        };
        reader.readAsDataURL(file);
      } else {
        alert("Ce fichier n’est pas une image.");
      }
    });

    // 🔒 Activer admin avec Alt+R
    window.addEventListener('keydown', e => {
      if (e.altKey && e.key.toLowerCase() === 'r') {
        const pwd = prompt("Mot de passe admin :");
        if (pwd === adminPassword) {
          isAdmin = true;
          deleteButton.disabled = false;
          alert("Mode admin activé.");
        } else {
          alert("Mot de passe incorrect.");
        }
      }
    });

    render();
  </script>
</body>
</html>
