<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Journal de Madrestown</title>
  <style>
    body {
      font-family: 'Courier New', Courier, monospace;
      background: linear-gradient(135deg, #0d0d0d, #1a1a1a);
      color: #f2f2f2;
      text-align: center;
      padding: 20px;
      min-height: 100vh;
      margin: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 15px;
    }
    h1 {
      color: #ffd700;
      text-shadow: 0 0 8px #ffd700;
    }
    .journal {
      border: 4px double #ffd700;
      border-radius: 15px;
      padding: 20px;
      max-width: 600px;
      background-color: #222;
      box-shadow: 0 0 15px #ffd700aa;
      width: 100%;
    }
    #drop-zone {
      margin: 20px auto;
      padding: 30px;
      border: 3px dashed #ffd700;
      border-radius: 15px;
      color: #ffd700;
      font-weight: bold;
      cursor: pointer;
      transition: background-color 0.3s ease;
      user-select: none;
    }
    #drop-zone.dragover {
      background-color: #444;
      box-shadow: 0 0 20px #ffd700;
    }
    input, textarea {
      width: 90%;
      margin-top: 10px;
      padding: 10px;
      border-radius: 5px;
      border: none;
      font-size: 16px;
    }
    textarea {
      resize: vertical;
      min-height: 60px;
    }
    button {
      margin: 10px 5px 0 5px;
      padding: 10px 20px;
      border: none;
      border-radius: 10px;
      background-color: #ffd700;
      color: #000;
      cursor: pointer;
      font-weight: bold;
      font-size: 16px;
      box-shadow: 0 0 10px #ffd700aa;
      transition: background-color 0.3s ease;
    }
    button:hover:not(:disabled) {
      background-color: #e6c200;
    }
    button:disabled {
      background-color: #999;
      cursor: not-allowed;
      box-shadow: none;
    }
    .image-container {
      margin: 20px 0;
      position: relative;
      border-radius: 12px;
      overflow: hidden;
      box-shadow: 0 0 15px #ffd700aa;
    }
    .image-container img {
      max-width: 100%;
      display: block;
      border-radius: 12px;
    }
    .author {
      position: absolute;
      bottom: 8px;
      right: 16px;
      background-color: rgba(0, 0, 0, 0.7);
      padding: 6px 12px;
      border-radius: 8px;
      color: #ffd700;
      font-weight: bold;
      font-size: 14px;
      text-shadow: 0 0 4px black;
    }
    .page-number {
      margin-top: 10px;
      font-size: 18px;
      font-weight: bold;
      color: #ffd700;
      text-shadow: 0 0 5px #ffd700bb;
    }
    .comments {
      margin-top: 15px;
      text-align: left;
      max-height: 150px;
      overflow-y: auto;
      background: #111;
      border-radius: 10px;
      padding: 10px;
      box-shadow: inset 0 0 10px #ffd70088;
    }
    .comment {
      margin-bottom: 10px;
      padding: 8px 12px;
      background-color: #262626;
      border-left: 4px solid #ffd700;
      border-radius: 5px;
      font-size: 14px;
      color: #eee;
      word-wrap: break-word;
    }
    .comment strong {
      color: #ffd700;
    }
  </style>
</head>
<body>
  <h1>Journal de Madrestown</h1>
  <div class="journal">
    <div id="drop-zone">Glisse une image ici ou clique pour choisir</div>
    <input type="text" id="new-author" placeholder="Nom de l'auteur" autocomplete="off" />
    <button id="add-btn" disabled>Ajouter</button>

    <div class="image-container">
      <img id="main-image" src="" alt="Image" />
      <div class="author" id="image-author"></div>
    </div>

    <div class="page-number" id="page-number">Aucune image</div>

    <div>
      <button id="prev-btn">←</button>
      <button id="next-btn">→</button>
    </div>

    <div class="comments" id="comments"></div>

    <input type="text" id="comment-name" placeholder="Ton nom" autocomplete="off" />
    <textarea id="comment-text" placeholder="Ton commentaire"></textarea>
    <button id="comment-btn">Envoyer</button>

    <button id="delete-button" disabled>Supprimer l’image</button>
  </div>

<script>
  const IMGUR_CLIENT_ID = "bf8f7df1ccfb23a";
  const DISCORD_WEBHOOK_URL = "https://discord.com/api/webhooks/1378675075598778430/0e04rMnv6J7OPdCs-rWlZccnr4Vr1XfYASCdCGY9-nljP4sT1EWJaxTC-haY9R7RK83O";

  let images = [];
  let comments = {};
  let currentIndex = 0;
  let droppedImage = null;
  let uploadedImageUrl = null;
  let isAdmin = false;
  const adminPassword = "admin123";

  const dropZone = document.getElementById('drop-zone');
  const addBtn = document.getElementById('add-btn');
  const mainImage = document.getElementById('main-image');
  const imageAuthor = document.getElementById('image-author');
  const pageNumber = document.getElementById('page-number');
  const commentsContainer = document.getElementById('comments');
  const deleteButton = document.getElementById('delete-button');
  const prevBtn = document.getElementById('prev-btn');
  const nextBtn = document.getElementById('next-btn');
  const commentBtn = document.getElementById('comment-btn');

  // Affiche la galerie + commentaires
  function render() {
    if (images.length === 0) {
      mainImage.src = "";
      imageAuthor.textContent = "";
      pageNumber.textContent = "Aucune image";
      commentsContainer.innerHTML = "";
      deleteButton.disabled = true;
      addBtn.disabled = !droppedImage;
      return;
    }
    const current = images[currentIndex];
    mainImage.src = current.url;
    imageAuthor.textContent = current.author;
    pageNumber.textContent = `Page ${currentIndex + 1} / ${images.length}`;

    const currentComments = comments[current.url] || [];
    commentsContainer.innerHTML = currentComments.map(
      c => `<div class="comment"><strong>${escapeHtml(c.name)}</strong>: ${escapeHtml(c.text)}</div>`
    ).join('');
    deleteButton.disabled = !isAdmin;
    addBtn.disabled = !!droppedImage; // Disable add if already have dropped image to add
  }

  // Sécurise texte pour éviter injection HTML
  function escapeHtml(text) {
    return text.replace(/[&<>"']/g, m => {
      return {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#39;'
      }[m];
    });
  }

  // Upload image sur Imgur et récupère URL
  async function uploadToImgur(base64Data) {
    try {
      const response = await fetch('https://api.imgur.com/3/image', {
        method: 'POST',
        headers: {
          Authorization: 'Client-ID ' + IMGUR_CLIENT_ID,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          image: base64Data.split(',')[1], // Supprime data:image/...;base64,
          type: 'base64'
        })
      });
      const data = await response.json();
      if (data.success) {
        return data.data.link;
      } else {
        alert("Erreur lors de l'upload Imgur: " + (data.data.error || 'Erreur inconnue'));
        return null;
      }
    } catch (e) {
      alert("Erreur réseau ou API Imgur");
      return null;
    }
  }

  // Envoie URL au webhook Discord
  async function sendToDiscord(url, author) {
    try {
      await fetch(DISCORD_WEBHOOK_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: `Nouvelle image ajoutée par **${author}** : ${url}`
        })
      });
    } catch (e) {
      console.warn("Erreur en envoyant au webhook Discord", e);
    }
  }

  // Ajouter image dans la galerie
  async function addImage() {
    const author = document.getElementById('new-author').value.trim();
    if (!droppedImage) {
      alert("Dépose une image avant d'ajouter.");
      return;
    }
    if (!author) {
      alert("Entre un nom d'auteur.");
      return;
    }

    addBtn.disabled = true;
    dropZone.textContent = "Upload en cours...";

    const uploadedUrl = await uploadToImgur(droppedImage);
    if (!uploadedUrl) {
      dropZone.textContent = "Dépose une image ici ou clique pour choisir";
      addBtn.disabled = false;
      return;
    }
    uploadedImageUrl = uploadedUrl;
    images.push({ url: uploadedUrl, author });
    comments[uploadedUrl] = [];
    currentIndex = images.length - 1;
    await sendToDiscord(uploadedUrl, author);
    droppedImage = null;
    uploadedImageUrl = null;
    document.getElementById('new-author').value = '';
    dropZone.textContent = "Dépose une image ici ou clique pour choisir";
    addBtn.disabled = true;
    render();
  }

  // Naviguer pages
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

  // Ajouter commentaire
  function addComment() {
    const name = document.getElementById('comment-name').value.trim();
    const text = document.getElementById('comment-text').value.trim();
    if (!name || !text) return;
    if (images.length === 0) {
      alert("Pas d'image pour commenter.");
      return;
    }
    const url = images[currentIndex].url;
    comments[url].push({ name, text });
    document.getElementById('comment-name').value = '';
    document.getElementById('comment-text').value = '';
    render();
  }

  // Supprimer image (admin seulement)
  function deleteImage() {
    if (!isAdmin) {
      alert("Action réservée à l'administrateur.");
      return;
    }
    if (images.length === 0) return;

    const url = images[currentIndex].url;
    images.splice(currentIndex, 1);
    delete comments[url];
    if (currentIndex >= images.length) currentIndex = images.length - 1;
    render();
  }

  // Drag & Drop gestion
  dropZone.addEventListener('dragover', e => {
    e.preventDefault();
    dropZone.classList.add('dragover');
  });
  dropZone.addEventListener('dragleave', e => {
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
        addBtn.disabled = false;
      };
      reader.readAsDataURL(file);
    } else {
      alert("Ce fichier n’est pas une image.");
    }
  });

  // Click sur zone drop pour ouvrir explorateur fichiers
  dropZone.addEventListener('click', () => {
    const fileInput = document.createElement('input');
    fileInput.type = 'file';
    fileInput.accept = 'image/*';
    fileInput.onchange = () => {
      const file = fileInput.files[0];
      if (file && file.type.startsWith('image/')) {
        const reader = new FileReader();
        reader.onload = () => {
          droppedImage = reader.result;
          dropZone.textContent = "Image prête ! Clique sur Ajouter.";
          addBtn.disabled = false;
        };
        reader.readAsDataURL(file);
      } else {
        alert("Ce fichier n’est pas une image.");
      }
    };
    fileInput.click();
  });

  // Admin mode Alt+R
  window.addEventListener('keydown', e => {
    if (e.altKey && e.key.toLowerCase() === 'r') {
      const pwd = prompt("Mot de passe admin :");
      if (pwd === adminPassword) {
        isAdmin = true;
        deleteButton.disabled = false;
        alert("Mode admin activé !");
        render();
      } else {
        alert("Mot de passe incorrect.");
      }
    }
  });

  addBtn.addEventListener('click', addImage);
  prevBtn.addEventListener('click', prevPage);
  nextBtn.addEventListener('click', nextPage);
  commentBtn.addEventListener('click', addComment);
  deleteButton.addEventListener('click', deleteImage);

  render();
</script>
</body>
</html>
