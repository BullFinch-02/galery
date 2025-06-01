<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Journal de Madrestown</title>
<style>
  /* Reset */
  * {
    box-sizing: border-box;
  }
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #f2f6ff, #dce6f7);
    color: #333;
    margin: 0; padding: 20px;
    display: flex;
    justify-content: center;
  }
  .journal {
    max-width: 600px;
    width: 100%;
    background: white;
    border-radius: 12px;
    padding: 25px 30px 40px 30px;
    box-shadow: 0 12px 25px rgba(0,0,0,0.12);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 18px;
  }
  h1 {
    margin-bottom: 10px;
    color: #0a3d62;
    font-weight: 700;
    text-align: center;
    letter-spacing: 1.5px;
  }
  #drop-zone {
    border: 3px dashed #1e90ff;
    border-radius: 10px;
    padding: 25px;
    width: 100%;
    text-align: center;
    color: #1e90ff;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease;
    user-select: none;
  }
  #drop-zone.dragover {
    background-color: #1e90ff;
    color: white;
    border-color: #174ea6;
  }
  input[type="text"], textarea {
    width: 100%;
    padding: 12px 15px;
    font-size: 1rem;
    border-radius: 8px;
    border: 2px solid #ddd;
    transition: border-color 0.3s ease;
    font-family: inherit;
    resize: vertical;
  }
  input[type="text"]:focus, textarea:focus {
    outline: none;
    border-color: #1e90ff;
    box-shadow: 0 0 8px #7ab8ff99;
  }
  textarea {
    min-height: 60px;
    max-height: 120px;
  }
  button {
    background-color: #1e90ff;
    color: white;
    border: none;
    padding: 12px 25px;
    border-radius: 8px;
    cursor: pointer;
    font-weight: 700;
    font-size: 1rem;
    transition: background-color 0.25s ease, transform 0.2s ease;
  }
  button:disabled {
    background-color: #9ec9ff88;
    cursor: not-allowed;
    transform: none;
  }
  button:not(:disabled):hover {
    background-color: #174ea6;
    transform: scale(1.05);
  }

  .image-container {
    position: relative;
    width: 100%;
    max-height: 400px;
    overflow: hidden;
    border-radius: 12px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.1);
  }
  #main-image {
    width: 100%;
    border-radius: 12px;
    object-fit: contain;
    max-height: 400px;
    opacity: 0;
    transform: translateY(20px);
    transition: opacity 0.5s ease, transform 0.5s ease;
  }
  #main-image.visible {
    opacity: 1;
    transform: translateY(0);
  }

  .author {
    position: absolute;
    bottom: 10px;
    right: 12px;
    background: rgba(30,144,255,0.85);
    color: white;
    padding: 5px 15px;
    border-radius: 20px;
    font-weight: 600;
    font-size: 0.9rem;
    user-select: none;
    box-shadow: 0 2px 8px rgba(0,0,0,0.2);
  }

  .page-number {
    font-weight: 600;
    font-size: 1rem;
    color: #555;
  }

  .comments {
    width: 100%;
    max-height: 160px;
    overflow-y: auto;
    background: #f7faff;
    border-radius: 8px;
    padding: 12px 15px;
    box-shadow: inset 0 0 6px #cde1ff;
  }
  .comment {
    margin-bottom: 10px;
    border-bottom: 1px solid #cde1ff;
    padding-bottom: 6px;
    font-size: 0.9rem;
    line-height: 1.3;
  }
  .comment:last-child {
    border: none;
  }

  .navigation-buttons {
    width: 100%;
    display: flex;
    justify-content: space-between;
    gap: 15px;
  }
  .navigation-buttons button {
    flex-grow: 1;
  }

  /* Responsive */
  @media (max-width: 480px) {
    .journal {
      padding: 20px 18px 30px 18px;
    }
    #drop-zone {
      padding: 18px;
    }
  }
</style>
</head>
<body>
  <div class="journal">
    <h1>Journal de Madrestown</h1>

    <div id="drop-zone">Glisse une image ici ou clique pour choisir</div>
    <input type="text" id="new-author" placeholder="Nom de l'auteur" autocomplete="off" />
    <button id="add-btn" disabled>Ajouter</button>

    <div class="image-container">
      <img id="main-image" src="" alt="Image" />
      <div class="author" id="image-author"></div>
    </div>

    <div class="page-number" id="page-number">Aucune image</div>

    <div class="navigation-buttons">
      <button id="prev-btn">← Précédent</button>
      <button id="next-btn">Suivant →</button>
    </div>

    <div class="comments" id="comments"></div>

    <input type="text" id="comment-name" placeholder="Ton nom" autocomplete="off" />
    <textarea id="comment-text" placeholder="Ton commentaire"></textarea>
    <button id="comment-btn">Envoyer</button>

    <button id="delete-button" disabled>Supprimer l’image</button>
  </div>

  <!-- Firebase SDK -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-storage-compat.js"></script>

<script>
  const firebaseConfig = {
    apiKey: "AIzaSyC_ktxCLNGAijAtlRFa2-t45_EBzDGJeF0",
    authDomain: "madrestown-galery.firebaseapp.com",
    projectId: "madrestown-galery",
    storageBucket: "madrestown-galery.appspot.com",
    messagingSenderId: "1003744187403",
    appId: "1:1003744187403:web:52f9392d6453bcf670e30e"
  };

  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();
  const storage = firebase.storage();

  auth.signInAnonymously().catch(console.error);

  let images = [];
  let comments = {};
  let currentIndex = 0;
  let droppedImage = null;
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

  // Chargement des images depuis Firestore
  async function loadImages() {
    const snapshot = await db.collection('images').orderBy('createdAt').get();
    images = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    if(images.length > 0) currentIndex = 0;
    await loadComments();
    render();
  }

  // Chargement des commentaires
  async function loadComments() {
    comments = {};
    const snapshot = await db.collection('comments').get();
    snapshot.docs.forEach(doc => {
      comments[doc.id] = doc.data().list || [];
    });
  }

  // Affichage des données courantes
  function render() {
    if (images.length === 0) {
      mainImage.src = "";
      imageAuthor.textContent = "";
      pageNumber.textContent = "Aucune image";
      commentsContainer.innerHTML = "";
      deleteButton.disabled = true;
      addBtn.disabled = !droppedImage;
      mainImage.classList.remove("visible");
      return;
    }
    const current = images[currentIndex];
    mainImage.classList.remove("visible");

    setTimeout(() => {
      mainImage.src = current.url;
      mainImage.onload = () => {
        mainImage.classList.add("visible");
      }
    }, 100);

    imageAuthor.textContent = current.author;
    pageNumber.textContent = `Page ${currentIndex + 1} / ${images.length}`;

    const currentComments = comments[current.id] || [];
    commentsContainer.innerHTML = currentComments.map(
      c => `<div class="comment"><strong>${escapeHtml(c.name)}</strong>: ${escapeHtml(c.text)}</div>`
    ).join('');

    deleteButton.disabled = !isAdmin;
    addBtn.disabled = !droppedImage || !document.getElementById('new-author').value.trim();
  }

  function escapeHtml(text) {
    return text.replace(/[&<>"']/g, m => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
    }[m]));
  }

  // Ajouter une image dans Firebase Storage et Firestore avec logs et suivi upload
  async function addImage() {
    const author = document.getElementById('new-author').value.trim();
    if (!droppedImage || !author) return alert("Image et auteur requis");

    addBtn.disabled = true;
    dropZone.textContent = "Upload en cours...";
    console.log("[addImage] Début upload");

    try {
      const res = await fetch(droppedImage);
      const blob = await res.blob();
      console.log("[addImage] Blob créé :", blob);

      const fileName = `${Date.now()}_${Math.random().toString(36).slice(2, 10)}.jpg`;
      const storageRef = storage.ref().child(`images/${fileName}`);
      const uploadTask = storageRef.put(blob);

      uploadTask.on('state_changed',
        snapshot => {
          const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
          dropZone.textContent = `Téléchargement: ${progress.toFixed(1)}%`;
          console.log(`[addImage] Upload ${progress.toFixed(1)}%`);
        },
        error => {
          console.error("[addImage] Erreur upload :", error);
          alert("Erreur lors du téléchargement de l'image.");
          addBtn.disabled = false;
          dropZone.textContent = "Glisse une image ici ou clique pour choisir";
        },
        async () => {
          const downloadURL = await uploadTask.snapshot.ref.getDownloadURL();
          console.log("[addImage] Upload terminé, URL :", downloadURL);

          const docRef = await db.collection('images').add({
            url: downloadURL,
            author: author,
            createdAt: firebase.firestore.FieldValue.serverTimestamp(),
          });
          console.log("[addImage] Image ajoutée en Firestore, ID:", docRef.id);

          images.push({ id: docRef.id, url: downloadURL, author: author });
          currentIndex = images.length - 1;
          droppedImage = null;
          document.getElementById('new-author').value = "";
          dropZone.textContent = "Glisse une image ici ou clique pour choisir";
          render();
        }
      );
    } catch (err) {
      console.error("[addImage] Erreur inattendue:", err);
      alert("Erreur lors de l'ajout de l'image.");
      addBtn.disabled = false;
      dropZone.textContent = "Glisse une image ici ou clique pour choisir";
    }
  }

  // Supprimer image
  async function deleteImage() {
    if (!isAdmin || images.length === 0) return;
    const current = images[currentIndex];
    if (!confirm("Voulez-vous vraiment supprimer cette image ?")) return;

    try {
      // Supprimer dans storage
      const fileRef = storage.refFromURL(current.url);
      await fileRef.delete();
      console.log("[deleteImage] Image supprimée du storage");

      // Supprimer dans Firestore
      await db.collection('images').doc(current.id).delete();
      console.log("[deleteImage] Image supprimée de Firestore");

      // Supprimer commentaires liés
      await db.collection('comments').doc(current.id).delete();
      delete comments[current.id];

      images.splice(currentIndex, 1);
      if (currentIndex >= images.length) currentIndex = images.length - 1;
      render();
    } catch (err) {
      console.error("[deleteImage] Erreur:", err);
      alert("Erreur lors de la suppression.");
    }
  }

  // Navigation
  prevBtn.onclick = () => {
    if (images.length === 0) return;
    currentIndex = (currentIndex - 1 + images.length) % images.length;
    render();
  };

  nextBtn.onclick = () => {
    if (images.length === 0) return;
    currentIndex = (currentIndex + 1) % images.length;
    render();
  };

  // Gestion commentaires
  commentBtn.onclick = async () => {
    const name = document.getElementById('comment-name').value.trim();
    const text = document.getElementById('comment-text').value.trim();
    if (!name || !text || images.length === 0) return alert("Nom, commentaire et image sont requis");

    const current = images[currentIndex];
    const commentList = comments[current.id] || [];

    commentList.push({ name, text });
    comments[current.id] = commentList;

    try {
      await db.collection('comments').doc(current.id).set({ list: commentList });
      document.getElementById('comment-name').value = "";
      document.getElementById('comment-text').value = "";
      render();
    } catch (err) {
      console.error("Erreur ajout commentaire:", err);
      alert("Erreur lors de l'ajout du commentaire.");
    }
  };

  // Gestion drag and drop
  dropZone.addEventListener('dragover', e => {
    e.preventDefault();
    dropZone.classList.add('dragover');
  });
  dropZone.addEventListener('dragleave', e => {
    e.preventDefault();
    dropZone.classList.remove('dragover');
  });
  dropZone.addEventListener('drop', e => {
    e.preventDefault();
    dropZone.classList.remove('dragover');

    if(e.dataTransfer.files.length === 0) return;

    const file = e.dataTransfer.files[0];
    if (!file.type.startsWith('image/')) {
      alert("Seules les images sont acceptées");
      return;
    }

    const reader = new FileReader();
    reader.onload = (evt) => {
      droppedImage = evt.target.result;
      dropZone.textContent = "Image prête à être ajoutée";
      addBtn.disabled = !document.getElementById('new-author').value.trim();
    };
    reader.readAsDataURL(file);
  });

  // Click drop zone to open file selector
  dropZone.addEventListener('click', () => {
    const input = document.createElement('input');
    input.type = 'file';
    input.accept = 'image/*';
    input.onchange = e => {
      const file = e.target.files[0];
      if (!file) return;
      if (!file.type.startsWith('image/')) {
        alert("Seules les images sont acceptées");
        return;
      }
      const reader = new FileReader();
      reader.onload = (evt) => {
        droppedImage = evt.target.result;
        dropZone.textContent = "Image prête à être ajoutée";
        addBtn.disabled = !document.getElementById('new-author').value.trim();
      };
      reader.readAsDataURL(file);
    };
    input.click();
  });

  // Activer/désactiver bouton Ajouter selon input author
  document.getElementById('new-author').addEventListener('input', e => {
    addBtn.disabled = !droppedImage || !e.target.value.trim();
  });

  // Ajouter image au clic
  addBtn.addEventListener('click', addImage);

  // Supprimer image au clic
  deleteButton.addEventListener('click', deleteImage);

  // Authentification admin simple
  window.addEventListener('load', () => {
    const pwd = prompt("Mot de passe admin (laisse vide pour utilisateur normal)");
    if (pwd === adminPassword) {
      isAdmin = true;
      alert("Mode administrateur activé");
    } else {
      isAdmin = false;
    }
  });

  // Initial load
  loadImages();
</script>

</body>
</html>
