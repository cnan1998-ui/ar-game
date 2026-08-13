<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"
  >

  <meta name="apple-mobile-web-app-capable" content="yes">

  <title>ผู้พิทักษ์จิ๋ว - AR เกมป้องกันสิ่งเสพติด</title>

  <!-- A-Frame -->
  <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>

  <!-- MindAR Face -->
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>

  <style>

    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    html,
    body {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background: #000;
      font-family: Tahoma, Arial, sans-serif;
    }

    /* =========================
       คะแนน
    ========================= */

    #ui-container {

      position: fixed;

      top: 15px;
      left: 50%;

      transform: translateX(-50%);

      z-index: 99999;

      padding: 12px 22px;

      background: rgba(20,20,20,0.82);

      color: white;

      border-radius: 22px;

      font-size: 19px;

      font-weight: bold;

      text-align: center;

      white-space: nowrap;

      box-shadow:
        0 5px 20px rgba(0,0,0,0.35);

    }

    /* =========================
       ข้อความด้านล่าง
    ========================= */

    #instructions {

      position: fixed;

      bottom: 25px;
      left: 50%;

      transform: translateX(-50%);

      z-index: 99999;

      width: calc(100% - 40px);

      max-width: 430px;

      padding: 13px 20px;

      border-radius: 25px;

      background: #28a745;

      color: white;

      font-size: 17px;

      font-weight: bold;

      text-align: center;

      line-height: 1.4;

      pointer-events: none;

      box-shadow:
        0 5px 20px rgba(0,0,0,0.3);

    }

    /* =========================
       หน้าจอเริ่มเกม
    ========================= */

    #start-screen {

      position: fixed;

      inset: 0;

      z-index: 100000;

      display: flex;

      align-items: center;

      justify-content: center;

      padding: 20px;

      background:
        linear-gradient(
          rgba(0,0,0,0.7),
          rgba(0,0,0,0.82)
        );

      color: white;

      text-align: center;

    }

    #start-box {

      width: 100%;

      max-width: 390px;

      padding: 30px 22px;

      border-radius: 25px;

      background: rgba(25,25,25,0.95);

      box-shadow:
        0 10px 40px rgba(0,0,0,0.6);

    }

    #start-box h1 {

      margin: 0 0 15px;

      font-size: 28px;

    }

    #start-box p {

      margin: 0 0 25px;

      color: #eee;

      line-height: 1.7;

      font-size: 16px;

    }

    #start-button {

      width: 100%;

      border: 0;

      border-radius: 30px;

      padding: 15px;

      background: #28a745;

      color: white;

      font-size: 20px;

      font-weight: bold;

    }

    #start-button:active {

      transform: scale(0.96);

    }

    /* =========================
       Debug
    ========================= */

    #debug {

      position: fixed;

      top: 72px;

      left: 10px;

      z-index: 99998;

      padding: 4px 8px;

      border-radius: 10px;

      background: rgba(0,0,0,0.5);

      color: white;

      font-size: 11px;

      pointer-events: none;

    }

    .hidden {
      display: none !important;
    }

  </style>
</head>


<body>


  <!-- =========================
       หน้าเริ่มเกม
  ========================= -->

  <div id="start-screen">

    <div id="start-box">

      <h1>
        🛡️ ผู้พิทักษ์จิ๋ว
      </h1>

      <p>

        เกม AR ป้องกันสิ่งเสพติด

        <br><br>

        🟢 รับของดี

        <br>

        🔴 หลบสิ่งไม่ดี

        <br><br>

        เอียงศีรษะซ้าย-ขวา
        เพื่อควบคุมโล่

      </p>

      <button id="start-button">
        🎮 เริ่มเกม
      </button>

    </div>

  </div>


  <!-- =========================
       คะแนน
  ========================= -->

  <div id="ui-container">

    🏆 คะแนน:
    <span id="score-text">0</span>

    &nbsp; | &nbsp;

    พลัง:
    <span id="lives-text">
      ❤️❤️❤️
    </span>

  </div>


  <!-- =========================
       คำแนะนำ
  ========================= -->

  <div id="instructions">

    👆 กด "เริ่มเกม"

  </div>


  <!-- =========================
       Debug
  ========================= -->

  <div id="debug">

    กำลังเตรียมเกม...

  </div>


  <!-- =========================
       A-FRAME / MINDAR
  ========================= -->

  <a-scene

    id="ar-scene"

    mindar-face

    embedded

    color-space="sRGB"

    renderer="
      colorManagement: true;
      physicallyCorrectLights: true;
      antialias: true;
    "

    vr-mode-ui="enabled: false"

    device-orientation-permission-ui="enabled: false"

    loading-screen="enabled: false"

  >


    <!-- กล้องของ MindAR -->

    <a-camera
      active="false"
      position="0 0 0"
    ></a-camera>


    <!-- =========================
         ใบหน้า
    ========================= -->

    <a-entity
      id="face-target"
      mindar-face-target="anchorIndex: 1"
    >

      <!-- โล่ -->
      <a-ring

        id="player-shield"

        position="0 -0.25 0"

        rotation="0 0 0"

        radius-inner="0.075"

        radius-outer="0.105"

        material="
          color: #00d9ff;
          shader: flat;
          opacity: 1;
          transparent: false;
        "

      ></a-ring>


      <!-- จุดกลางโล่ -->

      <a-circle

        position="0 -0.25 0"

        radius="0.055"

        material="
          color: #ffffff;
          shader: flat;
          opacity: 0.9;
        "

      ></a-circle>

    </a-entity>


    <!-- =========================
         กล่องไอเทม
    ========================= -->

    <a-entity
      id="items-container"
      position="0 0 -1.2"
    ></a-entity>


  </a-scene>


<script>

  /* =====================================================
     ตัวแปรเกม
  ===================================================== */

  let score = 0;

  let lives = 3;

  let gameActive = false;

  let faceDetected = false;

  let shieldPosX = 0;

  let spawnTimer = null;

  let itemNumber = 0;


  /* =====================================================
     DOM
  ===================================================== */

  const scene =
    document.getElementById("ar-scene");

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


  /* =====================================================
     Debug
  ===================================================== */

  function debugMessage(message) {

    debug.innerText = message;

    console.log(
      "[AR GAME]",
      message
    );

  }


  /* =====================================================
     เริ่มเกม
  ===================================================== */

  startButton.addEventListener(
    "click",
    startGame
  );


  function startGame() {

    startButton.disabled = true;

    startButton.innerText =
      "⏳ กำลังเปิดกล้อง...";


    startScreen.classList.add(
      "hidden"
    );


    instructions.innerText =
      "🔍 กำลังค้นหาใบหน้า...";


    instructions.style.background =
      "#dc3545";


    gameActive = true;


    debugMessage(
      "เริ่มเกมแล้ว"
    );


    /*
     * เริ่ม MindAR
     */

    try {

      const mindar =
        scene.systems["mindar-face"];

      if (mindar) {

        mindar.start();

        debugMessage(
          "เปิดระบบ AR แล้ว"
        );

      }

    }
    catch(error) {

      console.error(error);

      debugMessage(
        "AR start error"
      );

    }


    /*
     * เริ่มสร้างไอเทม
     */

    startItemSpawner();

  }


  /* =====================================================
     ตรวจพบใบหน้า
  ===================================================== */

  faceTarget.addEventListener(
    "targetFound",
    function() {

      faceDetected = true;


      instructions.innerText =
        "👈 เอียงคอ ซ้าย-ขวา เพื่อเก็บของดี / หลบสิ่งไม่ดี 👉";


      instructions.style.background =
        "#28a745";


      debugMessage(
        "👤 พบใบหน้า"
      );

    }
  );


  /* =====================================================
     ไม่พบใบหน้า
  ===================================================== */

  faceTarget.addEventListener(
    "targetLost",
    function() {

      faceDetected = false;


      instructions.innerText =
        "🔍 กรุณาหันหน้าเข้ากล้อง";


      instructions.style.background =
        "#dc3545";


      debugMessage(
        "ไม่พบใบหน้า"
      );

    }
  );


  /* =====================================================
     ควบคุมโล่ด้วยการเอียงศีรษะ
  ===================================================== */

  function updateHeadTilt() {

    if (
      gameActive &&
      faceDetected &&
      faceTarget.object3D
    ) {

      const z =
        faceTarget.object3D.rotation.z;


      if (z > 0.08) {

        shieldPosX = -0.3;

      }
      else if (z < -0.08) {

        shieldPosX = 0.3;

      }
      else {

        shieldPosX = 0;

      }


      shield.setAttribute(
        "position",
        `${shieldPosX} -0.25 0`
      );

    }


    requestAnimationFrame(
      updateHeadTilt
    );

  }


  /* =====================================================
     เริ่มระบบไอเทม
  ===================================================== */

  function startItemSpawner() {

    if (spawnTimer) {

      clearInterval(
        spawnTimer
      );

    }


    /*
     * ไอเทมชิ้นแรก
     */

    setTimeout(
      function() {

        spawnItem();

      },
      700
    );


    /*
     * ไอเทมทุก 1.3 วินาที
     */

    spawnTimer =
      setInterval(
        function() {

          if (gameActive) {

            spawnItem();

          }

        },
        1300
      );

  }


  /* =====================================================
     สร้างไอเทม
  ===================================================== */

  function spawnItem() {

    if (!gameActive) {
      return;
    }


    itemNumber++;


    /*
     * ของดี 60%
     */

    const isGood =
      Math.random() < 0.60;


    /*
     * ตำแหน่ง
     */

    const positions = [
      -0.38,
      0,
      0.38
    ];


    const x =
      positions[
        Math.floor(
          Math.random() *
          positions.length
        )
      ];


    /*
     * จุดเริ่มต้นด้านบน
     */

    let y = 0.75;


    /*
     * สร้าง Entity หลัก
     */

    const item =
      document.createElement(
        "a-entity"
      );


    item.setAttribute(
      "position",
      `${x} ${y} 0`
    );


    /*
     * ชนิด
     */

    item.dataset.type =
      isGood
        ? "good"
        : "bad";


    /* =================================================
       วงกลมสี
    ================================================= */

    const circle =
      document.createElement(
        "a-circle"
      );


    circle.setAttribute(
      "radius",
      "0.105"
    );


    circle.setAttribute(
      "material",
      isGood
        ? "color: #20c997; shader: flat;"
        : "color: #ff3b30; shader: flat;"
    );


    circle.setAttribute(
      "position",
      "0 0 0"
    );


    item.appendChild(
      circle
    );


    /* =================================================
       วงแหวน
    ================================================= */

    const ring =
      document.createElement(
        "a-ring"
      );


    ring.setAttribute(
      "radius-inner",
      "0.105"
    );


    ring.setAttribute(
      "radius-outer",
      "0.125"
    );


    ring.setAttribute(
      "material",
      "color: #ffffff; shader: flat;"
    );


    ring.setAttribute(
      "position",
      "0 0 0.005"
    );


    item.appendChild(
      ring
    );


    /* =================================================
       สัญลักษณ์ตรงกลาง
       
       ใช้รูปทรงแทน Emoji
       เพื่อให้ iPhone / Android เห็นเหมือนกัน
    ================================================= */

    if (isGood) {

      createGoodIcon(
        item
      );

    }
    else {

      createBadIcon(
        item
      );

    }


    /* =================================================
       เพิ่มเข้า Scene
    ================================================= */

    itemsContainer.appendChild(
      item
    );


    debugMessage(
      `🎁 ไอเทม ${itemNumber} กำลังตก`
    );


    /*
     * ทำให้เด่นขึ้น
     */

    item.setAttribute(
      "scale",
      "1.15 1.15 1.15"
    );


    /* =================================================
       Animation
    ================================================= */

    const speed =
      0.013;


    const timer =
      setInterval(
        function() {

          if (!gameActive) {

            clearInterval(
              timer
            );

            removeItem(
              item
            );

            return;

          }


          /*
           * ตกลง
           */

          y -= speed;


          item.setAttribute(
            "position",
            `${x} ${y} 0`
          );


          /*
           * ตรวจชน
           */

          if (
            y <= -0.14 &&
            y >= -0.39
          ) {

            const distance =
              Math.abs(
                x -
                shieldPosX
              );


            if (
              distance < 0.19
            ) {

              handleCollision(
                item.dataset.type
              );


              clearInterval(
                timer
              );


              removeItem(
                item
              );


              return;

            }

          }


          /*
           * ตกพ้นหน้าจอ
           */

          if (
            y < -0.85
          ) {

            clearInterval(
              timer
            );


            removeItem(
              item
            );

          }

        },
        30
      );

  }


  /* =====================================================
     สร้างไอคอนของดี
  ===================================================== */

  function createGoodIcon(
    parent
  ) {

    /*
     * สร้างเครื่องหมาย +
     */

    const vertical =
      document.createElement(
        "a-box"
      );


    vertical.setAttribute(
      "width",
      "0.025"
    );


    vertical.setAttribute(
      "height",
      "0.09"
    );


    vertical.setAttribute(
      "depth",
      "0.015"
    );


    vertical.setAttribute(
      "color",
      "#ffffff"
    );


    vertical.setAttribute(
      "position",
      "0 0 0.02"
    );


    parent.appendChild(
      vertical
    );


    const horizontal =
      document.createElement(
        "a-box"
      );


    horizontal.setAttribute(
      "width",
      "0.09"
    );


    horizontal.setAttribute(
      "height",
      "0.025"
    );


    horizontal.setAttribute(
      "depth",
      "0.015"
    );


    horizontal.setAttribute(
      "color",
      "#ffffff"
    );


    horizontal.setAttribute(
      "position",
      "0 0 0.02"
    );


    parent.appendChild(
      horizontal
    );

  }


  /* =====================================================
     สร้างไอคอนสิ่งไม่ดี
  ===================================================== */

  function createBadIcon(
    parent
  ) {

    /*
     * กากบาท
     */

    const bar1 =
      document.createElement(
        "a-box"
      );


    bar1.setAttribute(
      "width",
      "0.025"
    );


    bar1.setAttribute(
      "height",
      "0.10"
    );


    bar1.setAttribute(
      "depth",
      "0.015"
    );


    bar1.setAttribute(
      "color",
      "#ffffff"
    );


    bar1.setAttribute(
      "rotation",
      "0 0 45"
    );


    bar1.setAttribute(
      "position",
      "0 0 0.02"
    );


    parent.appendChild(
      bar1
    );


    const bar2 =
      document.createElement(
        "a-box"
      );


    bar2.setAttribute(
      "width",
      "0.025"
    );


    bar2.setAttribute(
      "height",
      "0.10"
    );


    bar2.setAttribute(
      "depth",
      "0.015"
    );


    bar2.setAttribute(
      "color",
      "#ffffff"
    );


    bar2.setAttribute(
      "rotation",
      "0 0 -45"
    );


    bar2.setAttribute(
      "position",
      "0 0 0.02"
    );


    parent.appendChild(
      bar2
    );

  }


  /* =====================================================
     ลบไอเทม
  ===================================================== */

  function removeItem(
    item
  ) {

    if (
      item &&
      item.parentNode
    ) {

      item.parentNode.removeChild(
        item
      );

    }

  }


  /* =====================================================
     ตรวจชน
  ===================================================== */

  function handleCollision(
    type
  ) {

    if (
      type === "good"
    ) {

      score += 10;


      scoreText.innerText =
        score;


      instructions.innerText =
        "🎉 เก่งมาก! +10 คะแนน";


      instructions.style.background =
        "#28a745";

    }
    else {

      lives--;


      updateLives();


      instructions.innerText =
        "💥 โดนสิ่งไม่ดี!";


      instructions.style.background =
        "#dc3545";


      if (
        lives <= 0
      ) {

        gameOver();

      }

    }

  }


  /* =====================================================
     อัปเดตพลัง
  ===================================================== */

  function updateLives() {

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


  /* =====================================================
     Game Over
  ===================================================== */

  function gameOver() {

    gameActive = false;


    if (spawnTimer) {

      clearInterval(
        spawnTimer
      );

      spawnTimer = null;

    }


    instructions.innerText =
      `💀 GAME OVER | คะแนน ${score}`;


    instructions.style.background =
      "#343a40";


    setTimeout(
      function() {

        const again =
          confirm(
            `จบเกม!\n\nคะแนน ${score} คะแนน\n\nเล่นอีกครั้งไหม?`
          );


        if (again) {

          location.reload();

        }

      },
      300
    );

  }


  /* =====================================================
     เริ่มตรวจหัว
  ===================================================== */

  updateHeadTilt();


  /* =====================================================
     Scene โหลดเสร็จ
  ===================================================== */

  scene.addEventListener(
    "loaded",
    function() {

      debugMessage(
        "ระบบ AR พร้อม"
      );

    }
  );


</script>

</body>
</html>
