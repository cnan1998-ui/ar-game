<!DOCTYPE html>
<html lang="th">
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <meta charset="UTF-8">
  <title>ผู้พิทักษ์จิ๋ว - AR เกมป้องกันสิ่งเสพติด</title>
  <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>
  <style>
    body { margin: 0; overflow: hidden; font-family: 'Tahoma', sans-serif; }
    #ui-container { position: absolute; top: 15px; left: 50%; transform: translateX(-50%); z-index: 999; background: rgba(0, 0, 0, 0.7); color: #fff; padding: 10px 20px; border-radius: 20px; font-size: 18px; text-align: center; min-width: 220px; }
    #instructions { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); z-index: 999; color: #fff; background: #dc3545; padding: 10px 20px; border-radius: 25px; font-size: 16px; font-weight: bold; text-align: center; pointer-events: none; }
  </style>
</head>
<body>
  <div id="ui-container">
    <span>🏆 คะแนน: <b id="score-text">0</b></span> | 
    <span>พลัง: <span id="lives-text">❤️❤️❤️</span></span>
  </div>
  <div id="instructions">🔍 กำลังสแกนใบหน้า... กรุณาหันหน้าเข้าหากล้อง</div>

  <a-scene mindar-face embedded color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
    <a-camera active="false" position="0 0 0"></a-camera>
    
    <a-entity id="face-target" mindar-face-target="anchorIndex: 1">
      <a-sphere id="player-shield" position="0 -0.25 -0.1" radius="0.07" color="#00d2ff" opacity="0.8">
        <a-ring color="#ffffff" radius-inner="0.07" radius-outer="0.09"></a-ring>
      </a-sphere>
    </a-entity>

    <a-entity id="items-container" position="0 0 -1"></a-entity>
  </a-scene>

  <script>
    let score = 0, lives = 3, gameActive = true, faceDetected = false;
    const faceTarget = document.getElementById('face-target');
    const playerShield = document.getElementById('player-shield');
    const itemsContainer = document.getElementById('items-container');
    const scoreText = document.getElementById('score-text');
    const livesText = document.getElementById('lives-text');
    const instructions = document.getElementById('instructions');
    let shieldPosX = 0;

    // สร้างภาพไอคอน 2D จาก Emoji อัตโนมัติ
    function createEmojiTexture(emojiText) {
      const canvas = document.createElement('canvas');
      canvas.width = 128;
      canvas.height = 128;
      const ctx = canvas.getContext('2d');
      ctx.font = '90px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(emojiText, 64, 64);
      return canvas.toDataURL();
    }

    const goodTextures = [createEmojiTexture('🥛'), createEmojiTexture('🍏'), createEmojiTexture('🥦')];
    const badTextures = [createEmojiTexture('🚬'), createEmojiTexture('💊'), createEmojiTexture('🍷')];

    faceTarget.addEventListener("targetFound", () => {
      faceDetected = true;
      instructions.innerText = "👈 เอียงคอ ซ้าย-ขวา เพื่อรับนม/ผลไม้ และหลบบุหรี่ 👉";
      instructions.style.background = "#28a745";
    });

    faceTarget.addEventListener("targetLost", () => {
      faceDetected = false;
      instructions.innerText = "🔍 ไม่พบใบหน้า กรุณาหันหน้าเข้าหากล้อง";
      instructions.style.background = "#dc3545";
    });

    function updateHeadTilt() {
      if (gameActive && faceDetected && faceTarget.object3D) {
        const tiltZ = faceTarget.object3D.rotation.z;
        if (tiltZ > 0.08) shieldPosX = -0.3;
        else if (tiltZ < -0.08) shieldPosX = 0.3;
        else shieldPosX = 0;
        playerShield.setAttribute('position', `${shieldPosX} -0.25 -0.1`);
      }
      requestAnimationFrame(updateHeadTilt);
    }

    function spawnItem() {
      if (!gameActive || !faceDetected) return;

      const item = document.createElement('a-entity');
      const isGood = Math.random() > 0.4;
      const positionsX = [-0.3, 0, 0.3];
      const startX = positionsX[Math.floor(Math.random() * positionsX.length)];
      let startY = 0.7;

      item.setAttribute('position', `${startX} ${startY} 0`);
      item.setAttribute('geometry', 'primitive: plane; width: 0.18; height: 0.18');

      if (isGood) {
        const randomGood = goodTextures[Math.floor(Math.random() * goodTextures.length)];
        item.setAttribute('material', `src: ${randomGood}; transparent: true; shader: flat`);
        item.dataset.type = 'good';
      } else {
        const randomBad = badTextures[Math.floor(Math.random() * badTextures.length)];
        item.setAttribute('material', `src: ${randomBad}; transparent: true; shader: flat`);
        item.dataset.type = 'bad';
      }

      itemsContainer.appendChild(item);

      const dropInterval = setInterval(() => {
        if (!gameActive) {
          clearInterval(dropInterval);
          if (item.parentNode) item.remove();
          return;
        }

        startY -= 0.018;
        item.setAttribute('position', `${startX} ${startY} 0`);

        if (startY <= -0.2 && startY >= -0.38 && Math.abs(startX - shieldPosX) < 0.15) {
          handleCollision(item.dataset.type);
          clearInterval(dropInterval);
          if (item.parentNode) item.remove();
        }

        if (startY < -0.7) {
          clearInterval(dropInterval);
          if (item.parentNode) item.remove();
        }
      }, 30);
    }

    updateHeadTilt();
    setInterval(spawnItem, 1600);

    function handleCollision(type) {
      if (type === 'good') {
        score += 10;
        scoreText.innerText = score;
      } else {
        lives -= 1;
        updateLivesUI();
        if (lives <= 0) gameOver();
      }
    }

    function updateLivesUI() {
      let hearts = '';
      for (let i = 0; i < lives; i++) hearts += '❤️';
      livesText.innerText = hearts || '💀';
    }

    function gameOver() {
      gameActive = false;
      alert(`จบเกม! คุณได้คะแนน: ${score} คะแนน\nคุณคือผู้พิทักษ์จิ๋วที่เก่งมาก!`);
      location.reload();
    }
  </script>
</body>
</html>
