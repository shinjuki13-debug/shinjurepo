<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Smile Booth</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Unbounded:wght@600&family=Outfit:wght@400;500&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg: #e9e4fa;
      --ink: #2b2347;
      --soft: #7d73a6;
      --pink: #ff6b92;
      --mint: #5fd9b0;
      --white: #ffffff;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      min-height: 100vh;
      font-family: "Outfit", Arial, sans-serif;
      color: var(--ink);
      background: radial-gradient(circle at 15% 10%, #f8d9e6 0, transparent 40%),
                  radial-gradient(circle at 90% 80%, #cfe6f7 0, transparent 40%),
                  var(--bg);
      text-align: center;
      padding: 32px 16px 60px;
    }
    h1 {
      font-family: "Unbounded", Arial, sans-serif;
      font-weight: 600;
      font-size: clamp(2rem, 7vw, 3.4rem);
      margin: 0 0 8px;
      letter-spacing: -0.03em;
    }
    .sub { color: var(--soft); margin: 0 0 28px; font-size: 1.05rem; }

    /* the camera window */
    .frame {
      position: relative;
      width: 100%;
      max-width: 460px;
      aspect-ratio: 4 / 3;
      margin: 0 auto;
      border-radius: 28px;
      background: var(--ink);
      overflow: hidden;
      box-shadow: 0 20px 50px rgba(43, 35, 71, 0.25);
    }
    video {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
      display: block;
    }
    .corner {
      position: absolute;
      width: 34px;
      height: 34px;
      border: 4px solid rgba(255, 255, 255, 0.7);
      transition: border-color 0.25s;
    }
    .tl { top: 16px; left: 16px; border-right: 0; border-bottom: 0; border-top-left-radius: 12px; }
    .tr { top: 16px; right: 16px; border-left: 0; border-bottom: 0; border-top-right-radius: 12px; }
    .bl { bottom: 16px; left: 16px; border-right: 0; border-top: 0; border-bottom-left-radius: 12px; }
    .br { bottom: 16px; right: 16px; border-left: 0; border-top: 0; border-bottom-right-radius: 12px; }
    .frame.found .corner { border-color: var(--mint); }

    .placeholder {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      color: rgba(255, 255, 255, 0.6);
      font-size: 1.1rem;
      padding: 20px;
    }
    #flash {
      position: absolute;
      inset: 0;
      background: #fff;
      opacity: 0;
      pointer-events: none;
    }
    #flash.go { animation: flash 0.5s ease-out; }
    @keyframes flash { from { opacity: 0.9; } to { opacity: 0; } }

    /* status pill */
    .pill {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      margin: 22px 0 6px;
      padding: 10px 20px;
      background: var(--white);
      border-radius: 999px;
      font-weight: 500;
      box-shadow: 0 6px 18px rgba(43, 35, 71, 0.1);
    }
    .dot { width: 12px; height: 12px; border-radius: 50%; background: var(--soft); transition: background 0.25s; }
    .pill.found .dot { background: var(--mint); }

    button {
      font-family: inherit;
      font-size: 1.1rem;
      font-weight: 500;
      color: var(--white);
      background: var(--pink);
      border: none;
      border-radius: 999px;
      padding: 14px 32px;
      cursor: pointer;
      box-shadow: 0 8px 20px rgba(255, 107, 146, 0.4);
    }
    button:hover { transform: translateY(-2px); }
    button:focus-visible { outline: 3px solid var(--ink); outline-offset: 3px; }

    #help {
      max-width: 460px;
      margin: 18px auto 0;
      padding: 16px 20px;
      background: var(--white);
      border-radius: 18px;
      text-align: left;
      line-height: 1.5;
      display: none;
    }
    #help b { color: var(--pink); }

    /* photo strip */
    #gallery {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 22px;
      margin-top: 44px;
    }
    .polaroid {
      background: var(--white);
      padding: 10px 10px 8px;
      border-radius: 6px;
      width: 190px;
      box-shadow: 0 10px 24px rgba(43, 35, 71, 0.18);
      transform: rotate(-3deg);
      animation: drop 0.4s ease-out;
    }
    .polaroid:nth-child(even) { transform: rotate(3deg); }
    .polaroid img { width: 100%; display: block; border-radius: 3px; }
    .polaroid span { display: block; margin-top: 8px; font-size: 0.85rem; color: var(--soft); }
    .polaroid a { color: var(--pink); font-weight: 500; text-decoration: none; font-size: 0.9rem; }
    @keyframes drop { from { opacity: 0; transform: translateY(-20px) scale(0.9); } }

    @media (prefers-reduced-motion: reduce) {
      * { animation: none !important; transition: none !important; }
    }
  </style>
</head>
<body>
  <h1>Smile Booth</h1>
  <p class="sub">Show your face and it takes the photo. Hide, and it waits.</p>

  <div class="frame" id="frame">
    <video id="video" autoplay muted playsinline></video>
    <div class="placeholder" id="placeholder">Camera is off</div>
    <div class="corner tl"></div><div class="corner tr"></div>
    <div class="corner bl"></div><div class="corner br"></div>
    <div id="flash"></div>
  </div>

  <div class="pill" id="pill"><span class="dot"></span><span id="status">Camera is off</span></div>
  <div><button id="startBtn" onclick="start()">Start camera</button></div>

  <div id="help"></div>
  <canvas id="canvas" style="display:none"></canvas>
  <div id="gallery"></div>

  <script src="https://cdn.jsdelivr.net/npm/@vladmandic/face-api@1.7.12/dist/face-api.js"></script>
  <script>
    const MODEL_URL = "https://cdn.jsdelivr.net/npm/@vladmandic/face-api@1.7.12/model/";
    const video = document.getElementById("video");
    const canvas = document.getElementById("canvas");
    const statusText = document.getElementById("status");
    const pill = document.getElementById("pill");
    const frame = document.getElementById("frame");
    const gallery = document.getElementById("gallery");
    const help = document.getElementById("help");

    let lastShot = 0;
    const WAIT_TIME = 3000; // 3 seconds between photos
    let photoCount = 0;

    function showHelp(html) {
      help.innerHTML = html;
      help.style.display = "block";
    }

    async function start() {
      help.style.display = "none";

      if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
        statusText.textContent = "Camera not supported here";
        showHelp("<b>This page can't use the camera.</b><br>Open it in Chrome, Edge or Safari. If you opened it inside another app (like WhatsApp or Telegram), open it in the browser instead.");
        return;
      }

      try {
        statusText.textContent = "Loading face detector...";
        await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
      } catch (e) {
        statusText.textContent = "Couldn't load the detector";
        showHelp("<b>No internet?</b> The face detector downloads from the web. Connect to the internet and try again.");
        return;
      }

      try {
        statusText.textContent = "Waiting for camera permission...";
        const stream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: "user" }
        });
        video.srcObject = stream;
        document.getElementById("placeholder").style.display = "none";
        document.getElementById("startBtn").style.display = "none";
        video.onloadeddata = () => {
          statusText.textContent = "Looking for a face...";
          setInterval(detect, 300);
        };
      } catch (error) {
        statusText.textContent = "Camera blocked";
        if (error.name === "NotAllowedError") {
          showHelp(
            "<b>The browser blocked the camera.</b> Try these in order:<br>" +
            "1. Click the lock or camera icon next to the web address, set Camera to <b>Allow</b>, then refresh.<br>" +
            "2. On Windows: Settings, Privacy, Camera, and turn on camera access for your browser.<br>" +
            "3. If you opened this as a file on your computer, upload it to GitHub Pages and open the https link. Browsers allow cameras most reliably there."
          );
        } else if (error.name === "NotFoundError") {
          showHelp("<b>No camera found.</b> Plug in a webcam or try on a phone.");
        } else if (error.name === "NotReadableError") {
          showHelp("<b>The camera is busy.</b> Close other apps using it (Zoom, Meet, Camera) and try again.");
        } else {
          showHelp("<b>Something went wrong:</b> " + error.message);
        }
      }
    }

    async function detect() {
      const result = await faceapi.detectSingleFace(
        video,
        new faceapi.TinyFaceDetectorOptions()
      );

      if (result) {
        frame.classList.add("found");
        pill.classList.add("found");
        statusText.textContent = "Face found";
        const now = Date.now();
        if (now - lastShot > WAIT_TIME) {
          takePhoto();
          lastShot = now;
        }
      } else {
        frame.classList.remove("found");
        pill.classList.remove("found");
        statusText.textContent = "No face, no photo";
      }
    }

    function takePhoto() {
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      const ctx = canvas.getContext("2d");
      ctx.translate(canvas.width, 0);
      ctx.scale(-1, 1); // mirror, same as what you see
      ctx.drawImage(video, 0, 0);
      const url = canvas.toDataURL("image/jpeg", 0.9);

      // flash effect
      const flash = document.getElementById("flash");
      flash.classList.remove("go");
      void flash.offsetWidth;
      flash.classList.add("go");

      photoCount = photoCount + 1;
      const time = new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
      const card = document.createElement("div");
      card.className = "polaroid";
      card.innerHTML =
        '<img src="' + url + '" alt="Photo ' + photoCount + '">' +
        '<span>' + time + '</span>' +
        '<a href="' + url + '" download="smile-' + photoCount + '.jpg">Download</a>';
      gallery.prepend(card);
    }
  </script>
</body>
</html>
