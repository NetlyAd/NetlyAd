<!DOCTYPE html>
<html>
<head>
  <title>Netly - Movie Upload</title>
  <meta charset="UTF-8" />
  <style>
    body {
      font-family: Arial, sans-serif;
      background: orange;
      color: white;
      padding: 20px;
      margin: 0;
    }
    input, button {
      margin: 5px 0;
      padding: 8px;
      border: none;
      border-radius: 5px;
    }
    video {
      max-width: 100%;
      display: block;
      margin-top: 10px;
    }
    .video-card {
      background: #e65100;
      padding: 10px;
      margin-bottom: 15px;
      border-radius: 8px;
    }
    .auth-section, .upload-section {
      background: #ff8a00;
      padding: 15px;
      border-radius: 10px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>

  <h1>🎬 Netly</h1>
  <p><strong>Status:</strong> <span id="user-status">Not logged in</span></p>

  <div class="auth-section">
    <h3>Login / Sign Up</h3>
    <input type="email" id="email" placeholder="Email" required><br>
    <input type="password" id="password" placeholder="Password" required><br>
    <button onclick="signUp()">Sign Up</button>
    <button onclick="signIn()">Log In</button>
    <button onclick="signOut()">Log Out</button>
  </div>

  <div class="upload-section" id="upload-section" style="display:none;">
    <h3>Upload Video</h3>
    <input type="text" id="title" placeholder="Movie Title" required><br>
    <input type="file" id="video" accept="video/*" required><br>
    <button onclick="uploadVideo()">Upload</button>
  </div>

  <h2>Uploaded Movies</h2>
  <div id="videos"></div>

  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.11.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.11.0/firebase-auth.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.11.0/firebase-storage.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.11.0/firebase-firestore.js"></script>

  <script>
    const firebaseConfig = {
      apiKey: "AIzaSyBRvLjxAvJCvzDopVsA2FzhuwiAiZ6HNG4",
      authDomain: "kill-f3b78.firebaseapp.com",
      projectId: "kill-f3b78",
      storageBucket: "kill-f3b78.appspot.com",
      messagingSenderId: "614261058521",
      appId: "1:614261058521:web:c53f121ffc711ec4e7c7c8",
      measurementId: "G-GT7ENLWFQ7"
    };

    firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const storage = firebase.storage();
    const db = firebase.firestore();

    const ADMIN_EMAIL = "g3324874@gmail.com";

    function showUser(user) {
      document.getElementById("user-status").textContent = user.email;
      document.getElementById("upload-section").style.display = "block";
      loadVideos();
    }

    function hideUser() {
      document.getElementById("user-status").textContent = "Not logged in";
      document.getElementById("upload-section").style.display = "none";
      document.getElementById("videos").innerHTML = "";
    }

    auth.onAuthStateChanged(user => {
      if (user) {
        showUser(user);
      } else {
        hideUser();
      }
    });

    function signUp() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      auth.createUserWithEmailAndPassword(email, password)
        .catch(err => alert(err.message));
    }

    function signIn() {
      const email = document.getElementById("email").value;
      const password = document.getElementById("password").value;
      auth.signInWithEmailAndPassword(email, password)
        .catch(err => alert(err.message));
    }

    function signOut() {
      auth.signOut();
    }

    function uploadVideo() {
      const user = auth.currentUser;
      const file = document.getElementById("video").files[0];
      const title = document.getElementById("title").value;

      if (!file || !title || !user) return alert("Missing fields");

      const filename = Date.now() + "_" + file.name;
      const videoRef = storage.ref(`videos/${filename}`);

      videoRef.put(file, {
        customMetadata: { user: user.email }
      }).then(snapshot => {
        return videoRef.getDownloadURL();
      }).then(url => {
        return db.collection("videos").add({
          title,
          url,
          user: user.email,
          filename,
          timestamp: new Date()
        });
      }).then(() => {
        alert("Uploaded!");
        loadVideos();
      }).catch(err => alert(err.message));
    }

    function loadVideos() {
      db.collection("videos").orderBy("timestamp", "desc").get().then(snapshot => {
        const videosDiv = document.getElementById("videos");
        videosDiv.innerHTML = "";
        snapshot.forEach(doc => {
          const data = doc.data();
          const isAdmin = auth.currentUser && auth.currentUser.email === ADMIN_EMAIL;

          const videoCard = document.createElement("div");
          videoCard.className = "video-card";
          videoCard.innerHTML = `
            <h3>${data.title}</h3>
            <p><small>Uploaded by: ${data.user} on ${data.timestamp.toDate().toLocaleString()}</small></p>
            <video controls src="${data.url}"></video>
            ${isAdmin ? `<button onclick="deleteVideo('${doc.id}', '${data.filename}')">🗑 Delete</button>` : ""}
          `;
          videosDiv.appendChild(videoCard);
        });
      });
    }

    function deleteVideo(docId, filename) {
      if (!confirm("Delete this video?")) return;
      const videoRef = storage.ref(`videos/${filename}`);

      videoRef.delete().then(() => {
        return db.collection("videos").doc(docId).delete();
      }).then(() => {
        alert("Deleted!");
        loadVideos();
      }).catch(err => alert(err.message));
    }
  </script>

</body>
</html>
