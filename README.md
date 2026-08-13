<!DOCTYPE html>
<html lang="th">
<head>
  <meta name="viewport"
        content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta charset="UTF-8">

  <title>ผู้พิทักษ์จิ๋ว - AR เกมป้องกันสิ่งเสพติด</title>

  <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>

  <style>
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      margin: 0;
      overflow: hidden;
      font-family: Tahoma, sans-serif;
      background: #000;
    }

    #ui-container {
      position: fixed;
      top: 15px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 9999;

      background: rgba(0, 0, 0, 0.75);
      color: #fff;

      padding: 10px 20px;
      border-radius: 25px;

      font-size: 18px;
      font-weight: bold;

      text-align: center;
      white-space: nowrap;

      box-shadow: 0 3px 15px rgba(0,0,0,.4);
    }

    #instructions {
      position: fixed;
      bottom: 25px;
      left: 50%;
      transform: translateX(-50%);

      z-index: 9999;

      color: #fff;
      background: #dc3545;

      padding: 12px 20px;
      border-radius: 30px;

      font-size: 16px;
      font-weight: bold;

      text-align: center;

      width: calc(100% - 40px);
      max-width: 500px;

      pointer-events: none;

      box-shadow: 0 3px 15px rgba(0,0,0,.3);
    }

    #start-screen {
      position: fixed;
      inset: 0;

      z-index: 10000;

      display: flex;
      align-items: center;
      justify-content: center;

      background:
        linear-gradient(
          rgba(0,0,0,.65),
          rgba(0,0,0,.75)
        );

      color: #fff;

      text-align: center;
    }

    #start-box {
      width: calc(100% - 40px);
      max-width: 400px;

      padding: 30px 20px;

      background: rgba(20,20,20,.9);
      border-radius: 25px;

      box-shadow: 0 10px 40px rgba(0,0,0,.5);
    }

    #start-box h1 {
      margin: 0 0 10px;
      font-size: 28px;
    }

    #start-box p {
      line-height: 1.6;
      color: #ddd;
    }

    #start-button {
      border: 0;
      background: #28a745;
      color: white;

      padding: 15px 30px;

      border-radius: 30px;

      font-size: 20px;
      font-weight: bold;

      cursor: pointer;

      width: 100%;
    }

    #start-button:active {
      transform: scale(.96);
    }

    #debug {
      position: fixed;
      left: 10px;
      top: 70px;
      z-index: 9999;

      color: #fff;
      font-size: 11px;

      background: rgba(0,0,0,.5);
      padding: 4px 8px;

      border-radius: 10px;

      pointer-events: none;
    }

    .hidden {
      display: none !important;
    }
  </style>
</head>

<body>

  <!-- หน้าเริ่มเกม -->
  <div id="start-screen">

    <div id="start-box">

      <h1>🛡️ ผู้พิทักษ์จิ๋ว</h1>

      <p>
        เกม AR ป้องกันสิ่งเสพติด
        <br><br>
        🥛🍏🥦 รับของดี
        <br>
        🚬💊🍷 หลบของไม่ดี
        <br><br>
        เอียงศีรษะซ้าย-ขวาเพื่อบังคับโล่
      </p>

      <button id="start-button">
        🎮 เริ่มเกม
      </button>

    </div>

  </div>


  <!-- คะแนน -->
  <div id="ui-container">

    🏆 คะแนน:
    <b id="score-text">0</b>

    &nbsp; | &nbsp;

    พลัง:
    <span id="lives-text">❤️❤️❤️</span>

  </div>


  <!-- คำแนะนำ -->
  <div id="instructions">
    👆 กด "เริ่มเกม" เพื่อเปิดกล้อง
  </div>


  <!-- Debug -->
  <div id="debug">
    กำลังเตรียมเกม...
  </div>


  <!-- AR -->
  <a-scene
    id="ar-scene"

    mindar-face

    embedded

    color-space="sRGB"

    renderer="
      colorManagement: true;
      physicallyCorrectLights: true;
      antialias: true
    "

    vr-mode-ui="enabled: false"

    device-orientation-permission-ui="enabled: false"

    loading-screen="enabled: false"
  >

    <a-camera
      active="false"
      position="0 0 0"
    ></a-camera>


    <!-- จุดจับใบหน้า -->
    <a-entity
      id="face-target"
      mindar-face-target="anchorIndex: 1"
    >

      <!-- โล่ของผู้เล่น -->
      <a-ring
        id="player-shield"
        position="0 -0.25 -0.1"

        radius-inner="0.075"
        radius-outer="0.105"

        color="#00d9ff"

        material="
          shader: flat;
          opacity: 0.95;
          transparent: true
        "
      ></a-ring>

      <a-sphere
        position="0 -0.25 -0.1"

        radius="0.06"

        color="#00d9ff"

        material="
          shader: flat;
          opacity: 0.75;
          transparent: true
        "
      ></a-sphere>

    </a-entity>


    <!-- พื้นที่สำหรับไอเทม -->
    <a-entity
      id="items-container"
      position="0 0 -1"
    ></a-entity>

  </a-scene>


<script>

  /*********************************
   * ตัวแปรเกม
   *********************************/

  let score = 0;
  let lives = 3;

  let gameActive = false;
  let faceDetected = false;

  let shieldPosX = 0;

  let spawnTimer = null;

  let itemCount = 0;


  /*********************************
   * DOM
   *********************************/

  const scene = document.getElementById("ar-scene");

  const faceTarget =
    document.getElementById("face-target");

  const shield =
    document.getElementById("player-shield");

  const itemsContainer =
    document.getElementById("items-container");

  const scoreText =
    document.getElementById("score-text");

  const livesText =
    document.getElementById("lives-text");

  const instructions =
    document.getElementById("instructions");

  const debug =
    document.getElementById("debug");

  const startScreen =
    document.getElementById("start-screen");

  const startButton =
    document.getElementById("start-button");


  /*********************************
   * สร้าง Emoji เป็นรูปภาพ
   *********************************/

  function createEmojiTexture(emoji) {

    const canvas =
      document.createElement("canvas");

    canvas.width = 256;
    canvas.height = 256;

    const ctx =
      canvas.getContext("2d");

    ctx.clearRect(
      0,
      0,
      256,
      256
    );

    ctx.font =
      "180px Arial, sans-serif";

    ctx.textAlign = "center";
    ctx.textBaseline = "middle";

    ctx.fillText(
      emoji,
      128,
      135
    );

    return canvas.toDataURL(
      "image/png"
    );
  }


  /*********************************
   * รูปไอเทม
   *********************************/

  const goodTextures = [

    createEmojiTexture("🥛"),
    createEmojiTexture("🍏"),
    createEmojiTexture("🥦")

  ];

  const badTextures = [

    createEmojiTexture("🚬"),
    createEmojiTexture("💊"),
    createEmojiTexture("🍷")

  ];


  /*********************************
   * Debug
   *********************************/

  function setDebug(text) {

    debug.innerText = text;

    console.log(
      "[GAME]",
      text
    );
  }


  /*********************************
   * เริ่มเกม
   *********************************/

  startButton.addEventListener(
    "click",
    startGame
  );


  function startGame() {

    startButton.disabled = true;

    startButton.innerText =
      "⏳ กำลังเปิดกล้อง...";

    setDebug(
      "กำลังเริ่ม AR..."
    );

    /*
     * เริ่มระบบ MindAR
     */

    const mindarSystem =
      scene.systems["mindar-face"];

    if (mindarSystem) {

      try {

        mindarSystem.start();

        setDebug(
          "เปิดกล้องแล้ว กำลังหาใบหน้า..."
        );

      } catch (error) {

        console.error(error);

        setDebug(
          "ไม่สามารถเปิด AR ได้"
        );

      }

    } else {

      setDebug(
        "ไม่พบระบบ MindAR"
      );

    }

    startScreen.classList.add(
      "hidden"
    );

    instructions.innerText =
      "🔍 กำลังค้นหาใบหน้า...";

    instructions.style.background =
      "#dc3545";

    /*
     * สำคัญ:
     * เริ่มเกมทันที ไม่รอ faceDetected
     */

    gameActive = true;

    startSpawning();

  }


  /*********************************
   * ตรวจพบใบหน้า
   *********************************/

  faceTarget.addEventListener(
    "targetFound",
    () => {

      faceDetected = true;

      setDebug(
        "👤 พบใบหน้า | ไอเทมกำลังตก"
      );

      instructions.innerText =
        "👈 เอียงหน้า ซ้าย-ขวา เพื่อเก็บของดี";

      instructions.style.background =
        "#28a745";

    }
  );


  /*********************************
   * ไม่พบใบหน้า
   *********************************/

  faceTarget.addEventListener(
    "targetLost",
    () => {

      faceDetected = false;

      setDebug(
        "ไม่พบใบหน้า แต่เกมยังทำงาน"
      );

      instructions.innerText =
        "🔍 กรุณาหันหน้าเข้ากล้อง";

      instructions.style.background =
        "#dc3545";

    }
  );


  /*********************************
   * บังคับโล่ด้วยการเอียงหัว
   *********************************/

  function updateHeadTilt() {

    if (
      gameActive &&
      faceDetected &&
      faceTarget.object3D
    ) {

      const rotationZ =
        faceTarget.object3D.rotation.z;


      if (rotationZ > 0.08) {

        shieldPosX = -0.3;

      }
      else if (rotationZ < -0.08) {

        shieldPosX = 0.3;

      }
      else {

        shieldPosX = 0;

      }


      shield.setAttribute(
        "position",
        `${shieldPosX} -0.25 -0.1`
      );

    }

    requestAnimationFrame(
      updateHeadTilt
    );
  }


  /*********************************
   * เริ่มสร้างไอเทม
   *********************************/

  function startSpawning() {

    if (spawnTimer) {

      clearInterval(
        spawnTimer
      );

    }


    /*
     * สร้างทันที 1 ชิ้น
     */

    setTimeout(
      () => {

        spawnItem();

      },
      500
    );


    /*
     * จากนั้นสร้างทุก 1.2 วินาที
     */

    spawnTimer =
      setInterval(
        () => {

          if (gameActive) {

            spawnItem();

          }

        },
        1200
      );

  }


  /*********************************
   * สร้างไอเทม
   *********************************/

  function spawnItem() {

    if (!gameActive) {
      return;
    }


    itemCount++;


    /*
     * สุ่มของดี/ของเสีย
     *
     * ของดี 60%
     * ของเสีย 40%
     */

    const isGood =
      Math.random() < 0.60;


    /*
     * ตำแหน่งซ้าย กลาง ขวา
     */

    const positionsX = [
      -0.3,
      0,
      0.3
    ];


    const startX =
      positionsX[
        Math.floor(
          Math.random() *
          positionsX.length
        )
      ];


    /*
     * เริ่มจากด้านบน
     */

    let currentY = 0.75;


    /*
     * สร้าง a-image โดยตรง
     *
     * จุดนี้แก้จากโค้ดเดิม
     */

    const item =
      document.createElement(
        "a-image"
      );


    item.setAttribute(
      "position",
      `${startX} ${currentY} 0`
    );


    item.setAttribute(
      "width",
      "0.20"
    );


    item.setAttribute(
      "height",
      "0.20"
    );


    /*
     * เลือกรูป
     */

    let texture;

    if (isGood) {

      texture =
        goodTextures[
          Math.floor(
            Math.random() *
            goodTextures.length
          )
        ];

      item.dataset.type =
        "good";

    }
    else {

      texture =
        badTextures[
          Math.floor(
            Math.random() *
            badTextures.length
          )
        ];

      item.dataset.type =
        "bad";

    }


    /*
     * ใส่รูป
     */

    item.setAttribute(
      "src",
      texture
    );


    /*
     * ทำให้รูปสว่างชัด
     */

    item.setAttribute(
      "material",
      `
        shader: flat;
        transparent: true;
        opacity: 1
      `
    );


    /*
     * ใส่ไอเทมเข้า Scene
     */

    itemsContainer.appendChild(
      item
    );


    setDebug(
      `🎁 ไอเทม #${itemCount} ตกลงมา`
    );


    /*
     * ความเร็วตก
     */

    const speed = 0.012;


    /*
     * animation loop
     */

    const dropInterval =
      setInterval(
        () => {

          if (!gameActive) {

            clearInterval(
              dropInterval
            );

            removeItem(
              item
            );

            return;

          }


          /*
           * เลื่อนลง
           */

          currentY -= speed;


          item.setAttribute(
            "position",
            `${startX} ${currentY} 0`
          );


          /*
           * ตรวจชนกับโล่
           *
           * ช่วง Y ประมาณโล่
           */

          if (
            currentY <= -0.16 &&
            currentY >= -0.38
          ) {

            const distanceX =
              Math.abs(
                startX -
                shieldPosX
              );


            if (
              distanceX < 0.17
            ) {

              handleCollision(
                item.dataset.type
              );


              clearInterval(
                dropInterval
              );


              removeItem(
                item
              );


              return;

            }

          }


          /*
           * ตกพ้นจอ
           */

          if (
            currentY < -0.85
          ) {

            clearInterval(
              dropInterval
            );

            removeItem(
              item
            );

          }

        },
        30
      );

  }


  /*********************************
   * ลบไอเทม
   *********************************/

  function removeItem(item) {

    if (
      item &&
      item.parentNode
    ) {

      item.parentNode.removeChild(
        item
      );

    }

  }


  /*********************************
   * ตรวจชน
   *********************************/

  function handleCollision(type) {

    if (type === "good") {

      score += 10;

      scoreText.innerText =
        score;


      /*
       * เอฟเฟกต์
       */

      instructions.innerText =
        "🎉 เยี่ยมมาก! +10 คะแนน";

    }
    else {

      lives--;

      updateLivesUI();


      instructions.innerText =
        "💥 ระวัง! เสียพลัง";


      if (lives <= 0) {

        gameOver();

      }

    }

  }


  /*********************************
   * อัปเดตหัวใจ
   *********************************/

  function updateLivesUI() {

    let hearts = "";

    for (
      let i = 0;
      i < lives;
      i++
    ) {

      hearts += "❤️";

    }


    livesText.innerText =
      hearts || "💀";

  }


  /*********************************
   * Game Over
   *********************************/

  function gameOver() {

    gameActive = false;


    if (spawnTimer) {

      clearInterval(
        spawnTimer
      );

    }


    instructions.innerText =
      `💀 GAME OVER — ${score} คะแนน`;


    instructions.style.background =
      "#343a40";


    setTimeout(
      () => {

        const playAgain =
          confirm(
            `จบเกม!\n\nคะแนนของคุณ: ${score} คะแนน\n\nเล่นอีกครั้งไหม?`
          );


        if (playAgain) {

          location.reload();

        }

      },
      500
    );

  }


  /*********************************
   * เริ่มตรวจการเอียงหัว
   *********************************/

  updateHeadTilt();


  /*********************************
   * เมื่อ Scene โหลดเสร็จ
   *********************************/

  scene.addEventListener(
    "loaded",
    () => {

      setDebug(
        "ระบบ AR พร้อมใช้งาน"
      );

    }
  );


</script>

</body>
</html>
