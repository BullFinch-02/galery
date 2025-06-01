<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Journal de Madrestown</title>
  <style>
    /* ... (ton CSS ici, inchangé) ... */
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
  let isAdmin = false;
  const adminPassword = "admin123";

  const webhookUrl = "https://discord.com/api/webhooks/1378675075598778430/0e04rMnv6J7OPdCs-rWlZccnr4Vr1XfYASCdCGY9-nljP4sT1EWJaxTC-haY9R7RK83O"; // <- mets ton URL webhook ici
  const imgurClientId = "bf8f7df1ccfb23a"; // <- mets ton Client-ID Imgur ici

  const mainImage = document.getElementById('main-image');
  const imageAuthor = document.getElementById('image-author');
  const pageNumber = document.getElementById('page-number');
  const commentsContainer = document.getElementById('comments');
  const deleteButton = document.getElementById('delete-button');

  function render() {
    if (images.length === 0) {
      mainImage.src = "";
      imageAuthor.textContent = "";
      pageNumber.textContent = "Aucune image";
      commentsContainer.innerHTML = "";
      deleteButton.disabled = true;
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

    deleteButton.disabled = !isAdmin;
  }

  function addImage() {
    const author = document.getElementById('new-author').value.trim();
    if (!droppedImage || !author) {
      alert("Ajoute une image et un nom d'auteur.");
      return;
    }
    // Upload sur Imgur
    uploadToImgur(droppedImage).then(imgurUrl => {
      images.push({ url: imgurUrl, author });
      currentIndex = images.length - 1;
      comments[imgurUrl] = [];
      droppedImage = null;
      document.getElementById('new-author').value = '';
      document.getElementById('drop-zone').textContent = "Dépose une image ici";

      render();
      sendToDiscordWebhook(author, imgurUrl);
    }).catch(err => {
      alert("Erreur lors de l'upload de l'image.");
      console.error(err);
    });
  }

  function uploadToImgur(base64Image) {
    return new Promise((resolve, reject) => {
      // Supprime le préfixe data:image/png;base64,
      const base64Data = base64Image.split(',')[1];

      fetch("https://api.imgur.com/3/image", {
        method: "POST",
        headers: {
          Authorization: `Client-ID ${imgurClientId}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          image: base64Data,
          type: "base64"
        }),
      })
      .then(response => response.json())
      .then(data => {
        if (data.success) {
          resolve(data.data.link);
        } else {
          reject(data);
        }
      })
      .catch(reject);
    });
  }

  function sendToDiscordWebhook(author, imageUrl) {
    const payload = {
      content: `Nouvelle image ajoutée par **${author}**`,
      embeds: [
        {
          image: {
            url: imageUrl
          }
        }
      ]
    };

    fetch(webhookUrl, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
    }).then(response => {
      if (!response.ok) {
        console.error("Erreur en envoyant au webhook Discord");
      }
    }).catch(console.error);
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

  window.addEventListener('keydown', e => {
    if (e.altKey && e.key.toLowerCase() === 'r') {
      const pwd = prompt("Mot de passe admin :");
      if (pwd === adminPassword) {
        isAdmin = true;
        deleteButton.disabled = false;
        alert("Mode admin activé.");
        render();
      } else {
        alert("Mot de passe incorrect.");
      }
    }
  });

  render();
</script>
</body>
</html>
