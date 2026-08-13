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

  <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>

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

    #ui-container {
      position: fixed;
      top: 15px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 99999;

      background: rgba(20,20,20,0.82);
      color: white;

      padding: 12px 20px;
      border-radius: 24px;

      font-size: 18px;
      font-weight: bold;

      text-align: center;
      white-space: nowrap;

      box-shadow: 0 5px 20px rgba(0,0,0,0.35);
    }

    #instructions {
      position: fixed;

      bottom: 25px;
      left: 50%;

      transform: translateX(-50%);

      z-index: 99999;

      width: calc(100% - 40px);
      max-width: 450px;

      padding: 13px 18px;

      border-radius: 26px;

      background: #28a745;
      color: white;

      font-size: 16px;
      font-weight: bold;

      text-align: center;
      line-height: 1.45;

      pointer-events: none;

      box-shadow: 0 5px 20px rgba(0,0,0,0.3);
    }

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
          rgba(0,0,0,0.72),
          rgba(0,0,0,0.85)
        );

      color: white;
      text-align: center;
    }

    #start-box {
      width: 100%;
      max-width: 390px;

      padding: 28px 22px;

      border-radius: 25px;

      background: rgba(25,25,25,0.96);

      box-shadow:
        0 10px 40px rgba(0,0,0,0.6);
    }

    #start-box h1 {
      margin: 0 0 12px;
      font-size: 28px;
    }

    #start-box p {
      margin: 0 0 24px;

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
       START SCREEN
  ========================= -->

  <div id="start-screen">

    <div id="start-box">

      <h1>🛡️ ผู้พิทักษ์จิ๋ว</h1>

      <p>
        เกม AR ป้องกันสิ่งเสพติด

        <br><br>

        🥛 🍎 🥦 💧
        <br>
        รับของมีประโยชน์ <b>+10 คะแนน</b>

        <br><br>

        🚬 💊 🍺
        <br>
        หลีกเลี่ยงสิ่งเสพติด <b>-10 คะแนน</b>

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
       SCORE
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
       INSTRUCTIONS
  ========================= -->

  <div id="instructions">
    👆 กด "เริ่มเกม"
  </div>


  <!-- =========================
       DEBUG
  ========================= -->

  <div id="debug">
    กำลังเตรียมเกม...
  </div>


  <!-- =========================
       AR SCENE
  ========================= -->

  <a-scene

    id="ar-scene"

    mindar-face

    embedded

    color-space="sRGB"

    renderer="
      colorManagement: true;
      antialias: true;
    "

    vr-mode-ui="enabled: false"

    device-orientation-permission-ui="enabled: false"

    loading-screen="enabled: false"

  >

    <a-camera
      active="false"
      position="0 0 0"
    ></a-camera>


    <!-- =========================
         FACE TARGET
    ========================= -->

    <a-entity
      id="face-target"
      mindar-face-target="anchorIndex: 1"
    >

      <!-- โล่ -->
      <a-ring

        id="player-shield"

        position="0 -0.25 -0.05"

        radius-inner="0.075"

        radius-outer="0.105"

        material="
          color: #00d9ff;
          shader: flat;
          opacity: 1;
        "

      ></a-ring>

      <a-circle

        position="0 -0.25 -0.05"

        radius="0.055"

        material="
          color: #ffffff;
          shader: flat;
          opacity: 0.9;
        "

      ></a-circle>

    </a-entity>


    <!-- =========================
         ITEMS
    ========================= -->

    <a-entity
      id="items-container"
      position="0 0 -0.8"
    ></a-entity>

  </a-scene>


<script>


/* =====================================================
   GAME VARIABLES
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
   DEBUG
===================================================== */

function debugMessage(message) {

  debug.innerText = message;

  console.log(
    "[AR GAME]",
    message
  );

}


/* =====================================================
   SVG IMAGE CREATOR
===================================================== */

function makeSVG(svg) {

  return "data:image/svg+xml;charset=utf-8," +
    encodeURIComponent(svg);

}


/* =====================================================
   ITEM IMAGES
===================================================== */

/*
 * ของมีประโยชน์
 */

const goodItems = [

  {
    name: "นม",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256"
           viewBox="0 0 256 256">

        <rect width="256" height="256"
              rx="45"
              fill="#eaf8ff"/>

        <path
          d="M75 60 L181 60 L170 215 L86 215 Z"
          fill="#ffffff"
          stroke="#168bd0"
          stroke-width="8"/>

        <path
          d="M75 60 L92 35 L164 35 L181 60"
          fill="#dff4ff"
          stroke="#168bd0"
          stroke-width="8"/>

        <rect x="90" y="100"
              width="76"
              height="58"
              rx="12"
              fill="#43a5df"/>

        <text
          x="128"
          y="138"
          text-anchor="middle"
          font-size="30"
          font-weight="bold"
          fill="white">
          นม
        </text>

        <circle cx="150" cy="175"
                r="8"
                fill="#168bd0"/>

        <circle cx="105" cy="180"
                r="6"
                fill="#168bd0"/>

      </svg>
    `)
  },

  {
    name: "ผลไม้",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#f2ffe9"/>

        <circle cx="105" cy="135"
                r="62"
                fill="#ef3340"/>

        <circle cx="158" cy="135"
                r="62"
                fill="#ff4d4d"/>

        <path
          d="M128 72 Q140 40 174 45"
          fill="none"
          stroke="#6b3f20"
          stroke-width="13"
          stroke-linecap="round"/>

        <path
          d="M155 55 Q190 35 207 65 Q174 82 155 55"
          fill="#35a853"/>

        <circle cx="82" cy="112"
                r="9"
                fill="#ff8b8b"
                opacity=".7"/>

        <circle cx="176" cy="108"
                r="9"
                fill="#ff9b9b"
                opacity=".7"/>

      </svg>
    `)
  },

  {
    name: "ผัก",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#effff1"/>

        <circle cx="128" cy="140"
                r="65"
                fill="#39a852"/>

        <circle cx="88" cy="120"
                r="42"
                fill="#52bd61"/>

        <circle cx="168" cy="120"
                r="42"
                fill="#48b95a"/>

        <path
          d="M128 72
             Q110 40 75 48
             Q91 82 128 82"
          fill="#228b3b"/>

        <path
          d="M128 72
             Q148 40 181 48
             Q166 82 128 82"
          fill="#2e9944"/>

      </svg>
    `)
  },

  {
    name: "น้ำ",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#eaf8ff"/>

        <path
          d="M105 35
             L151 35
             L151 57
             L165 74
             L165 215
             L91 215
             L91 74
             L105 57 Z"
          fill="#8ed8ff"
          stroke="#168bd0"
          stroke-width="8"/>

        <rect x="105" y="40"
              width="46"
              height="18"
              rx="5"
              fill="#168bd0"/>

        <path
          d="M100 110 Q128 90 156 110
             L156 170 Q128 190 100 170 Z"
          fill="#ffffff"
          opacity=".85"/>

        <text
          x="128"
          y="150"
          text-anchor="middle"
          font-size="26"
          font-weight="bold"
          fill="#168bd0">
          น้ำ
        </text>

      </svg>
    `)
  }

];


/*
 * สิ่งเสพติด / สิ่งที่ควรหลีกเลี่ยง
 */

const badItems = [

  {
    name: "บุหรี่",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#fff1f1"/>

        <rect x="45" y="112"
              width="150"
              height="38"
              rx="18"
              fill="#eeeeee"
              stroke="#555"
              stroke-width="6"/>

        <rect x="145" y="112"
              width="50"
              height="38"
              fill="#ff4b4b"/>

        <path
          d="M198 105 Q220 85 207 66"
          fill="none"
          stroke="#999"
          stroke-width="8"
          stroke-linecap="round"/>

        <circle cx="209" cy="58"
                r="9"
                fill="#aaa"
                opacity=".7"/>

        <path
          d="M128 80 L128 175"
          stroke="#e53935"
          stroke-width="12"/>

        <path
          d="M81 83 L175 177"
          stroke="#e53935"
          stroke-width="12"/>

      </svg>
    `)
  },

  {
    name: "ยาเสพติด",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#fff0f3"/>

        <g transform="rotate(-25 128 130)">

          <rect x="63" y="92"
                width="130"
                height="72"
                rx="36"
                fill="#ffffff"
                stroke="#d51f45"
                stroke-width="8"/>

          <path
            d="M128 92 L128 164"
            stroke="#d51f45"
            stroke-width="8"/>

          <rect x="128" y="92"
                width="65"
                height="72"
                rx="0 36 36 0"
                fill="#e43c62"/>

        </g>

        <path
          d="M75 70 L181 176"
          stroke="#d51f45"
          stroke-width="13"/>

        <path
          d="M181 70 L75 176"
          stroke="#d51f45"
          stroke-width="13"/>

      </svg>
    `)
  },

  {
    name: "แอลกอฮอล์",
    image: makeSVG(`
      <svg xmlns="http://www.w3.org/2000/svg"
           width="256"
           height="256">

        <rect width="256" height="256"
              rx="45"
              fill="#fff6e9"/>

        <path
          d="M91 43 L165 43
             L156 105
             L173 142
             L173 210
             L83 210
             L83 142
             L100 105 Z"
          fill="#f4a641"
          stroke="#a85b12"
          stroke-width="8"/>

        <rect x="101" y="48"
              width="54"
              height="28"
              rx="6"
              fill="#fff"
              opacity=".9"/>

        <path
          d="M90 145
             Q128 128 166 145
             L166 205
             L90 205 Z"
          fill="#f9c46b"/>

        <path
          d="M75 72 L181 178"
          stroke="#d62828"
          stroke-width="13"/>

        <path
          d="M181 72 L75 178"
          stroke="#d62828"
          stroke-width="13"/>

      </svg>
    `)
  }

];


/* =====================================================
   START GAME
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


  try {

    const mindar =
      scene.systems["mindar-face"];

    if (mindar) {

      mindar.start();

    }

  }
  catch(error) {

    console.error(error);

    debugMessage(
      "ไม่สามารถเริ่ม AR"
    );

  }


  /*
   * เริ่มปล่อยไอเทม
   */

  startItemSpawner();

}


/* =====================================================
   FACE FOUND
===================================================== */

faceTarget.addEventListener(
  "targetFound",
  function() {

    faceDetected = true;


    instructions.innerText =
      "👈 เอียงคอ ซ้าย-ขวา เพื่อรับของดี / หลบสิ่งเสพติด 👉";


    instructions.style.background =
      "#28a745";


    debugMessage(
      "👤 พบใบหน้า"
    );

  }
);


/* =====================================================
   FACE LOST
===================================================== */

faceTarget.addEventListener(
  "targetLost",
  function() {

    faceDetected = false;


    instructions.innerText =
      "🔍 กรุณาหันหน้าเข้ากล้อง";


    instructions.style.background =
      "#dc3545";


  }
);


/* =====================================================
   HEAD CONTROL
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

      shieldPosX = -0.38;

    }
    else if (z < -0.08) {

      shieldPosX = 0.38;

    }
    else {

      shieldPosX = 0;

    }


    shield.setAttribute(
      "position",
      `${shieldPosX} -0.25 -0.05`
    );

  }


  requestAnimationFrame(
    updateHeadTilt
  );

}


/* =====================================================
   ITEM SPAWNER
===================================================== */

function startItemSpawner() {

  if (spawnTimer) {

    clearInterval(
      spawnTimer
    );

  }


  /*
   * ชิ้นแรก
   */

  setTimeout(
    function() {

      spawnItem();

    },
    700
  );


  /*
   * ทุก 1.3 วินาที
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
   SPAWN ITEM
===================================================== */

function spawnItem() {

  if (!gameActive) {
    return;
  }


  itemNumber++;


  /*
   * 60% ของมีประโยชน์
   * 40% สิ่งเสพติด
   */

  const isGood =
    Math.random() < 0.60;


  const itemList =
    isGood
      ? goodItems
      : badItems;


  const itemData =
    itemList[
      Math.floor(
        Math.random() *
        itemList.length
      )
    ];


  /*
   * ตำแหน่งซ้าย / กลาง / ขวา
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


  let y = 0.75;


  /*
   * A-IMAGE
   */

  const item =
    document.createElement(
      "a-image"
    );


  item.dataset.type =
    isGood
      ? "good"
      : "bad";


  item.dataset.name =
    itemData.name;


  item.setAttribute(
    "src",
    itemData.image
  );


  /*
   * ขนาดใหญ่
   */

  item.setAttribute(
    "width",
    "0.28"
  );


  item.setAttribute(
    "height",
    "0.28"
  );


  item.setAttribute(
    "position",
    `${x} ${y} 0`
  );


  item.setAttribute(
    "material",
    `
      shader: flat;
      transparent: true;
      opacity: 1;
      side: double;
    `
  );


  itemsContainer.appendChild(
    item
  );


  debugMessage(
    `${isGood ? "🟢" : "🔴"} ${itemData.name} กำลังตก`
  );


  /*
   * ความเร็ว
   */

  const speed =
    0.013;


  /*
   * Animation
   */

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


        y -= speed;


        item.setAttribute(
          "position",
          `${x} ${y} 0`
        );


        /*
         * หมุนเล็กน้อย
         */

        item.object3D.rotation.z +=
          0.025;


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
            distance < 0.20
          ) {

            handleCollision(
              item.dataset.type,
              item.dataset.name
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
         * ตกพ้น
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
   REMOVE ITEM
===================================================== */

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


/* =====================================================
   COLLISION
===================================================== */

function handleCollision(
  type,
  itemName
) {

  if (
    type === "good"
  ) {

    /*
     * ของมีประโยชน์ +10
     */

    score += 10;


    scoreText.innerText =
      score;


    instructions.innerText =
      `🎉 ${itemName} +10 คะแนน`;


    instructions.style.background =
      "#28a745";

  }

  else {

    /*
     * สิ่งเสพติด -10
     */

    score =
      Math.max(
        0,
        score - 10
      );


    scoreText.innerText =
      score;


    instructions.innerText =
      `🚫 ${itemName} -10 คะแนน`;


    instructions.style.background =
      "#dc3545";

  }

}


/* =====================================================
   UPDATE LIVES
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
   GAME OVER
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
   START HEAD TRACKING
===================================================== */

updateHeadTilt();


/* =====================================================
   SCENE LOADED
===================================================== */

scene.addEventListener(
  "loaded",
  function() {

    debugMessage(
      "ระบบ AR พร้อมใช้งาน"
    );

  }
);

</script>

</body>
</html>
