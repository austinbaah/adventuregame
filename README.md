<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Tile Map Maze Game</title>
  <style>
     /* Global Styles */
    body {
      overflow: hidden;
      margin: 0;
      padding: 0;
      background-color: #15ff66;
    }

    #scene {
      overflow: hidden;
      width: 100%;
      height: 100%;
      position: absolute;
      background-image: url('data:image/svg+xml,%3Csvg width="12" height="24" viewBox="0 0 12 24" xmlns="http://www.w3.org/2000/svg"%3E%3Cg fill="none" fill-rule="evenodd"%3E%3Cg fill="%239C92AC" fill-opacity="0.4"%3E%3Cpath d="M2 0h2v12H2V0zm1 20c1.105 0 2-.895 2-2s-.895-2-2-2-2 .895-2 2 .895 2 2 2zM9 8c1.105 0 2-.895 2-2s-.895-2-2-2-2 .895-2 2 .895 2 2 2zm-1 4h2v12H8V12z"/%3E%3C/g%3E%3C/g%3E%3C/svg%3E');
    }

    /* Layout Containers */
    #gameboard {
      position: absolute;
    }
    #inventory, #controls {
      z-index: 1000;
    }
    #inventory {
      position: fixed;
      bottom: 0;
      left: 0;
      width: 100%;
      background-color: #000;
      color: #fff;
      font-family: 'Courier New', monospace;
      font-size: 16px;
      padding: 5px;
      box-sizing: border-box;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #controls {
      position: fixed;
      top: 0;
      left: 0;
      padding: 5px;
      background-color: rgba(0,0,0,0.5);
      color: #fff;
      font-family: 'Courier New', monospace;
      font-size: 16px;
    }

    /* Debug Hitbox */
    #blue-hitbox {
      position: absolute;
      top: 5px;
      left: 15px;
      width: 40px;
      height: 45px;
      border: 2px solid blue;
      pointer-events: none;
    }

    /* Character Styles */
    #character {
      --pixelsize: 1; /* Scaling Factor */
      position: relative;
      top: 0;
      left: 0;
      background-image: url('https://raw.githubusercontent.com/austinbaah/adventuregame/main/CharacterMusclular2.png');
      width: calc(var(--pixelsize) * 32px);
      height: calc(var(--pixelsize) * 32px);
      background-size: calc(18 * var(--pixelsize) * 32px) calc(62 * var(--pixelsize) * 32px);
      image-rendering: pixelated;
      border: 2px solid red;
      z-index: 1000;
    }

    /* Game Over / Win Overlay */
    #overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.75);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      text-align: center;
      z-index: 3000;
      font-family: Arial, sans-serif;
      font-size: 48px;
      color: #fff;
    }

    /* Animations for Movement */
    .walk-up { animation: walk-up 0.6s steps(9) infinite; }
    .walk-left { animation: walk-left 0.6s steps(9) infinite; }
    .walk-down { animation: walk-down 0.6s steps(9) infinite; }
    .walk-right { animation: walk-right 0.6s steps(9) infinite; }

    .idle-up { background-position: 0 calc(-8 * var(--pixelsize) * 32px); }
    .idle-left { background-position: 0 calc(-9 * var(--pixelsize) * 32px); }
    .idle-down { background-position: 0 calc(-10 * var(--pixelsize) * 32px); }
    .idle-right { background-position: 0 calc(-11 * var(--pixelsize) * 32px); }

    /* Animations for Shooting */
    .shoot-up { animation: shoot-up 0.2s steps(13); }
    .shoot-left { animation: shoot-left 0.2s steps(13); }
    .shoot-down { animation: shoot-down 0.2s steps(13); }
    .shoot-right { animation: shoot-right 0.2s steps(13); }

    /* Animations for Conjuring */
    .conjure-up { animation: conjure-up 0.6s steps(7); }
    .conjure-left { animation: conjure-left 0.6s steps(7); }
    .conjure-down { animation: conjure-down 0.6s steps(7); }
    .conjure-right { animation: conjure-right 0.6s steps(7); }

    /* Keyframe Animations */
    @keyframes walk-up {
      from { background-position: 0 calc(-8 * var(--pixelsize) * 32px); }
      to { background-position: calc(-9 * var(--pixelsize) * 32px) calc(-8 * var(--pixelsize) * 32px); }
    }
    @keyframes walk-left {
      from { background-position: 0 calc(-9 * var(--pixelsize) * 32px); }
      to { background-position: calc(-9 * var(--pixelsize) * 32px) calc(-9 * var(--pixelsize) * 32px); }
    }
    @keyframes walk-down {
      from { background-position: 0 calc(-10 * var(--pixelsize) * 32px); }
      to { background-position: calc(-9 * var(--pixelsize) * 32px) calc(-10 * var(--pixelsize) * 32px); }
    }
    @keyframes walk-right {
      from { background-position: 0 calc(-11 * var(--pixelsize) * 32px); }
      to { background-position: calc(-9 * var(--pixelsize) * 32px) calc(-11 * var(--pixelsize) * 32px); }
    }
    @keyframes shoot-up {
      from { background-position: 0 calc(-16 * var(--pixelsize) * 32px); }
      to { background-position: calc(-13 * var(--pixelsize) * 32px) calc(-16 * var(--pixelsize) * 32px); }
    }
    @keyframes shoot-left {
      from { background-position: 0 calc(-17 * var(--pixelsize) * 32px); }
      to { background-position: calc(-13 * var(--pixelsize) * 32px) calc(-17 * var(--pixelsize) * 32px); }
    }
    @keyframes shoot-down {
      from { background-position: 0 calc(-18 * var(--pixelsize) * 32px); }
      to { background-position: calc(-13 * var(--pixelsize) * 32px) calc(-18 * var(--pixelsize) * 32px); }
    }
    @keyframes shoot-right {
      from { background-position: 0 calc(-19 * var(--pixelsize) * 32px); }
      to { background-position: calc(-13 * var(--pixelsize) * 32px) calc(-19 * var(--pixelsize) * 32px); }
    }
    @keyframes conjure-up {
      from { background-position: 0 0; }
      to { background-position: calc(-7 * var(--pixelsize) * 32px) 0; }
    }
    @keyframes conjure-left {
      from { background-position: 0 calc(-1 * var(--pixelsize) * 32px); }
      to { background-position: calc(-7 * var(--pixelsize) * 32px) calc(-1 * var(--pixelsize) * 32px); }
    }
    @keyframes conjure-down {
      from { background-position: 0 calc(-2 * var(--pixelsize) * 32px); }
      to { background-position: calc(-7 * var(--pixelsize) * 32px) calc(-2 * var(--pixelsize) * 32px); }
    }
    @keyframes conjure-right {
      from { background-position: 0 calc(-3 * var(--pixelsize) * 32px); }
      to { background-position: calc(-7 * var(--pixelsize) * 32px) calc(-3 * var(--pixelsize) * 32px); }
    }

    .spell-cast {
      position: absolute;
      width: 32px;
      height: 32px;
      background: url("https://physicsland.net/spritesheets/Purple_Spell_Cast.png") 0 0 no-repeat;
      background-size: 1024px 32px;
      transform: scale(3);
      transform-origin: top left;
      pointer-events: none;
      z-index: 999;
      animation: spellCast 0.6s steps(31) forwards;
    }
    @keyframes spellCast {
      0%   { background-position: 0 0; }
      100% { background-position: -992px 0; }
    }
  </style>
</head>
<body>
  <div id="scene">
    <div id="gameboard">
      <canvas id="gameCanvas" width="576" height="576" style="position:absolute; top:0; left:0; z-index:100;"></canvas>
      <div id="character" class="idle-down"></div>
    </div>
  </div>
  <!-- Inventory Container -->
  <div id="inventory">
    Keys x 0&nbsp;&nbsp;&nbsp;Coins x 0&nbsp;&nbsp;&nbsp;Potion x 0&nbsp;&nbsp;&nbsp;Ammo x 0
  </div>
  <!-- Controls Container -->
  <div id="controls">
    Opacity: <input type="range" id="opacitySlider" min="0" max="1" step="0.05" value="0.5">
    <label for="disableOpacitySlider">Fixed (Left)</label>
    <input type="checkbox" id="disableOpacitySlider" checked>
  </div>
  <div id="overlay" style="display: none;"></div>

  <script>
     // Background and Tile Styling
    const backgroundImageURL = 'https://raw.githubusercontent.com/austinbaah/adventuregame/main/map3.png';
    let tileOpacity = 0.5; // Initial opacity

    // Starting Position (Tile Coordinates)
    const startTile = { x: 0, y: 1 };
    
     // Tile Types
    const TILE_TYPES = {
      FLOOR: 0,         // Light bluish gray (#D0E4F5)
      WALL: 1,          // Dark charcoal (#2B2B2B)
      TREASURE: 2,      // Ruby red (#E0115F)
      COIN: 3,          // Gold (#FFD700)
      KEY: 4,           // Green (#00FF00)
      AMMO: 5,          // Blue (#0000FF)
      DOOR: 6,          // Brown (#8B4513)
      normalEnemy: 7,   // Normal enemy tile
      bossEnemy: 8,     // Boss enemy tile
      heart: 9,         // Heart collectible tile
      pulseEnemy: 'x'   // Pulse enemy (represented by 'x')
    };
    
     // Tile Map and Caching Elements
    const tileMap = [
      [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 5, 0],
      [0, 0, 5, 0, 0, 3, 0, 0, 0, 0, 0, 0, 0, 0, 3, 0, 0, 9],
      [1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 1, 1, 1, 1, 1, 1, 1, 1],
      [5, 0, 9, 1, 0, 0, 0, 1, 0, 2, 1, 0, 0, 0, 1, 0, 0, 0],
      [0, 0, 9, 1, 0, 0, 0, 0, 'x', 'x', 0, 3, 0, 7, 1, 4, 0, 4],
      [0, 0, 1, 1, 0, 0, 0, 0, 'x', 'x', 0, 0, 0, 0, 1, 1, 6, 6],
      [7, 0, 7, 7, 7, 0, 0, 0, 1, 1, 0, 0, 7, 0, 7, 0, 0, 0], 
      [0, 0, 0, 0, 0, 0, 3, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0],
      [1, 1, 1, 1, 1, 1, 'x', 'x', 1, 1, 'x', 'x', 1, 1, 1, 1, 1, 1],
      [0, 0, 1, 1, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 1, 5, 0],
      [4, 0, 1, 2, 0, 7, 0, 0, 1, 1, 0, 0, 0, 4, 0, 1, 0, 0],
      [7, 0, 4, 0, 0, 1, 0, 0, 7, 0, 5, 0, 1, 0, 0, 4, 0, 0],
      [0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 7, 0],
      [1, 1, 6, 6, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 6, 6, 1, 1],
      [0, 0, 0, 0, 0, 1, 0, 0, 1, 1, 7, 0, 1, 0, 0, 0, 0, 0],
      [0, 5, 3, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      [0, 0, 0, 0, 0, 0, 7, 0, 0, 0, 7, 0, 0, 0, 5, 0, 0, 0],
      [0, 0, 8, 0, 0, 0, 0, 0, 0, 8, 0, 0, 0, 7, 0, 0, 0, 0]
    ];
    
    // Enemy Game Asset Images
    const pulseEnemyImage = new Image();
    pulseEnemyImage.src = 'https://physicsland.net/GD2Game1/fireball.png';
    
    // Pulse Enemy Timing
    const pulse_frequency = 4000; // milliseconds (3s on/3s off)

    const normalEnemyImage = new Image();
    normalEnemyImage.src = 'https://physicsland.net/GD2Game1/enemy1.gif';

    const bossEnemyImage = new Image();
    bossEnemyImage.src = 'https://physicsland.net/GD2Game1/skull.png';
    
     // Arrow Constants
    const arrowSpeed = 5;              // Arrow speed
    const arrowOriginOffsetX = 16;     // Arrow origin offset X (e.g., half the character width)
    const arrowOriginOffsetY = 16;     // Arrow origin offset Y (e.g., half the character height)
    
    // Game Constants
    const TILE_SIZE = 32; // pixels
    const MOVE_SPEED = 4; // adjustable move speed

    // Hitbox Parameters (Customizable)
    let hitboxWidth = 12;
    let hitboxHeight = 16;
    let hitboxOffsetX = 11;
    let hitboxOffsetY = 10;
    
     // Activate temporary invulnerability for the character
    function activateInvulnerability(duration = 6000) { //Set to 6 seconds of invulnerability
      isInvulnerable = true;
      character.style.opacity = 0.4;
      setTimeout(() => {
        isInvulnerable = false;
        character.style.opacity = 1;
      }, duration);
    }
    
    // ==============================
    // Configurable Constants and Settings
    // ==============================
    // Canvas and Context
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    // Arrow Images
    const arrowImages = {
      up: new Image(),
      down: new Image(),
      left: new Image(),
      right: new Image()
    };
    arrowImages.up.src = 'https://physicsland.net/GD2Game1/arrow_up_32x5.png';
    arrowImages.down.src = 'https://physicsland.net/GD2Game1/arrow_down_32x5.png';
    arrowImages.left.src = 'https://physicsland.net/GD2Game1/arrow_left_32x5.png';
    arrowImages.right.src = 'https://physicsland.net/GD2Game1/arrow_right_32x5.png';

    // ==============================
    // Global Variables and DOM Elements
    // ==============================
    let enemies = [];
    let arrows = [];
    let gameOver = false;
    let isPerformingAction = false;
    let isOnCooldown = false;
    let isInvulnerable = false;
    let canTakeDamage = true;

    const scene = document.getElementById('scene');
    const gameboard = document.getElementById('gameboard');
    const character = document.getElementById('character');
    const inventoryDisplay = document.getElementById('inventory');

    // Character State
    let characterPosition = calculatePosition(startTile.x, startTile.y);
    let currentDirection = 'down';
    const allClasses = ['walk-up', 'walk-down', 'walk-left', 'walk-right', 'idle-up', 'idle-left', 'idle-down', 'idle-right'];

    // Camera Variables
    let camOffset = { x: window.innerWidth / 2, y: window.innerHeight / 2 };

    // Player Inventory
    let inventory = {
      keys: 0,
      coins: 0,
      treasures: 0,
      ammo: 0,
      hearts: 3
    };
   
    // Deep copy for resetting the tile map
    const initialTileMap = JSON.parse(JSON.stringify(tileMap));

    const tileElements = []; // Cache for tile DOM elements
    const itemElements = {}; // Cache for item DOM elements

    // ==============================
    // Utility Functions
    // ==============================
    // Calculate pixel position from tile coordinates
    function calculatePosition(xTile, yTile) {
      return { x: TILE_SIZE * xTile, y: TILE_SIZE * yTile };
    }

    // Update the hitbox visual position and dimensions
    function updateHitboxVisual() {
      hitboxVisual.style.width = hitboxWidth + 'px';
      hitboxVisual.style.height = hitboxHeight + 'px';
      hitboxVisual.style.left = (characterPosition.x + hitboxOffsetX) + 'px';
      hitboxVisual.style.top = (characterPosition.y + hitboxOffsetY) + 'px';
    }

    // Check for collision between two rectangles
    function isColliding(a, b) {
      return a.x < b.x + b.width &&
             a.x + a.width > b.x &&
             a.y < b.y + b.height &&
             a.y + a.height > b.y;
    }

    // ==============================
    // DOM and Game Elements Initialization
    // ==============================
    // Create a visual hitbox for debugging
    const hitboxVisual = document.createElement('div');
    hitboxVisual.style.position = 'absolute';
    hitboxVisual.style.border = '1px solid blue'; //set to zero to remove
    hitboxVisual.style.pointerEvents = 'none';
    hitboxVisual.style.zIndex = '200';
    gameboard.appendChild(hitboxVisual);

    // ==============================
    // Game Logic Functions
    // ==============================
    // Update enemy positions and behavior
    function updateEnemies() { 
      enemies.forEach((enemy) => {
        if (enemy.type === 'pulse') {
          // Toggle visibility based on pulse frequency
          if (Date.now() - enemy.lastPulseToggle >= pulse_frequency) {
            enemy.visible = !enemy.visible;
            enemy.lastPulseToggle = Date.now();
          }
        } else {
          let nextX = enemy.x + enemy.vx;
          let nextY = enemy.y + enemy.vy;
          // Horizontal movement check
          let tileY = Math.floor(enemy.y / TILE_SIZE);
          let tileX = enemy.vx > 0 
            ? Math.floor((nextX + enemy.width - 1) / TILE_SIZE)
            : Math.floor(nextX / TILE_SIZE);
          if (
            tileX < 0 ||
            tileX >= tileMap[0].length ||
            tileMap[tileY][tileX] === TILE_TYPES.WALL ||
            tileMap[tileY][tileX] === TILE_TYPES.DOOR
          ) {
            enemy.vx = -enemy.vx;
          } else {
            enemy.x = nextX;
          }
          // Vertical movement check
          let tileXForVertical = Math.floor(enemy.x / TILE_SIZE);
          let tileYVertical = enemy.vy > 0 
            ? Math.floor((nextY + enemy.height - 1) / TILE_SIZE)
            : Math.floor(nextY / TILE_SIZE);
          if (
            tileYVertical < 0 ||
            tileYVertical >= tileMap.length ||
            tileMap[tileYVertical][tileXForVertical] === TILE_TYPES.WALL ||
            tileMap[tileYVertical][tileXForVertical] === TILE_TYPES.DOOR
          ) {
            enemy.vy = -enemy.vy;
          } else {
            enemy.y = nextY;
          }
        }
      });
    }

    // Render enemies on the canvas
    function renderEnemies() {
      enemies.forEach(enemy => {
        if (enemy.type === 'boss') {
          ctx.drawImage(bossEnemyImage, enemy.x, enemy.y, enemy.width, enemy.height);
          // Draw health bar above the boss
          const healthBarWidth = enemy.width;
          const healthBarHeight = 5;
          const healthRatio = enemy.health / 3;
          ctx.fillStyle = 'red';
          ctx.fillRect(enemy.x, enemy.y - 10, healthBarWidth * healthRatio, healthBarHeight);
          ctx.strokeStyle = 'black';
          ctx.strokeRect(enemy.x, enemy.y - 10, healthBarWidth, healthBarHeight);
        } else if (enemy.type === 'pulse') {
          if (enemy.visible) {
            ctx.drawImage(pulseEnemyImage, enemy.x, enemy.y, enemy.width, enemy.height);
          }
        } else {
          ctx.drawImage(normalEnemyImage, enemy.x, enemy.y, enemy.width, enemy.height);
        }
      });
    }

    // Reset the game to its initial state
    function resetGame() {
      gameOver = false;
      inventory = {
        keys: 0,
        coins: 0,
        treasures: 0,
        ammo: 0,
        hearts: 3
      };
      updateInventoryDisplay();
      characterPosition = calculatePosition(startTile.x, startTile.y);
      currentDirection = 'down';
      character.className = 'idle-down';
      enemies = [];
      arrows = [];
      tileElements.length = 0;
      for (let key in itemElements) {
        delete itemElements[key];
      }
      tileMap.splice(0, tileMap.length, ...JSON.parse(JSON.stringify(initialTileMap)));
      while (gameboard.firstChild) {
        gameboard.removeChild(gameboard.firstChild);
      }
      gameboard.appendChild(canvas);
      gameboard.appendChild(character);
      gameboard.appendChild(hitboxVisual);
      generateMap();
      const overlay = document.getElementById('overlay');
      overlay.style.display = "none";
      updateCameraPosition();
      requestAnimationFrame(gameLoop);
    }

    // Create a spell cast effect at a given position
    function createSpellCastEffect(x, y, duration = 1000) {
      const spellCast = document.createElement('div');
      spellCast.classList.add('spell-cast');
      spellCast.style.left = (x - 28) + 'px';
      spellCast.style.top = (y - 32) + 'px';
      spellCast.style.animationDuration = (duration / 1000) + 's';
      gameboard.appendChild(spellCast);
      setTimeout(() => {
        if (spellCast.parentNode) {
          spellCast.parentNode.removeChild(spellCast);
        }
      }, duration);
    }

    // Shoot an arrow in the current direction
    function shootArrow() {
      let arrowWidth, arrowHeight;
      if (currentDirection === 'up' || currentDirection === 'down') {
        arrowWidth = 5;
        arrowHeight = 32;
      } else {
        arrowWidth = 32;
        arrowHeight = 5;
      }
      let arrow = {
        x: characterPosition.x + arrowOriginOffsetX,
        y: characterPosition.y + arrowOriginOffsetY,
        width: arrowWidth,
        height: arrowHeight,
        direction: currentDirection,
        speed: arrowSpeed,
        isVisible: true
      };
      arrows.push(arrow);
    }

    // Check if an arrow collides with an obstacle
    function checkArrowCollision(arrow) {
      let tileX = Math.floor(arrow.x / TILE_SIZE);
      let tileY = Math.floor(arrow.y / TILE_SIZE);
      if (tileY < 0 || tileY >= tileMap.length || tileX < 0 || tileX >= tileMap[0].length) {
        return true;
      }
      const tileValue = tileMap[tileY][tileX];
      return tileValue === TILE_TYPES.WALL || tileValue === TILE_TYPES.DOOR;
    }

    // Render arrows on the canvas
    function renderArrows() {
      arrows.forEach((arrow) => {
        if (arrow.isVisible) {
          ctx.drawImage(arrowImages[arrow.direction], arrow.x, arrow.y, arrow.width, arrow.height);
        }
      });
    }

    // Check for collisions between the character and enemies
    function checkCharacterEnemyCollision() {
      enemies.forEach(enemy => {
        const characterHitbox = {
          x: characterPosition.x + hitboxOffsetX,
          y: characterPosition.y + hitboxOffsetY,
          width: hitboxWidth,
          height: hitboxHeight
        };
        if (isColliding(characterHitbox, enemy)) {
          if (enemy.type === 'pulse') {
            if (enemy.visible && (Date.now() - enemy.lastDamageTime >= 1000)) { //Pulse enemy damage cool down
              inventory.hearts -= 1;
              updateInventoryDisplay();
              if (inventory.hearts <= 0) {
                showDeathMessage();
              }
              enemy.lastDamageTime = Date.now();
            }
          } else {
            if (canTakeDamage) {
              if (enemy.type === 'boss' || (!isInvulnerable)) {
                inventory.hearts -= enemy.type === 'boss' ? 3 : 1;  // boss subtracts 3 normal enemy 1
                updateInventoryDisplay();
                if (inventory.hearts <= 0) {
                  showDeathMessage();
                }
              }
              canTakeDamage = false;
              setTimeout(() => { canTakeDamage = true; }, 1000); //Boss cool down
            }
          }
        }
      });
    }

    // Update the inventory display
    function updateInventoryDisplay() {
      inventoryDisplay.innerHTML = `Keys x ${inventory.keys}&nbsp;&nbsp;&nbsp;Coins x ${inventory.coins}&nbsp;&nbsp;&nbsp;Potion x ${inventory.treasures}&nbsp;&nbsp;&nbsp;Ammo x ${inventory.ammo}&nbsp;&nbsp;&nbsp;Hearts x ${inventory.hearts}`;
    }

    // Update the character's on-screen position
    function updateCharacterPosition() {
      character.style.left = characterPosition.x + 'px';
      character.style.top = characterPosition.y + 'px';
    }

    // Update arrow positions and handle collisions
    function updateArrows() {
      arrows.forEach((arrow) => {
        switch (arrow.direction) {
          case 'up': arrow.y -= arrow.speed; break;
          case 'down': arrow.y += arrow.speed; break;
          case 'left': arrow.x -= arrow.speed; break;
          case 'right': arrow.x += arrow.speed; break;
        }
        enemies.forEach((enemy, index) => {
          if (isColliding(arrow, enemy)) {
            arrow.isVisible = false;
            if (enemy.type === 'normal') {
              enemies.splice(index, 1);
            } else if (enemy.type === 'boss') {
              enemy.health -= 1;
              if (enemy.health <= 0) {
                enemies.splice(index, 1);
              }
            }
          }
        });
        if (checkArrowCollision(arrow)) {
          arrow.isVisible = false;
        }
      });
      arrows = arrows.filter(arrow => arrow.isVisible);
    }

    // Set character animation based on movement direction
    function setAnimation(direction) {
      character.classList.remove(...allClasses);
      character.classList.add('walk-' + direction);
    }

    // Move the character by a specified offset
    function moveCharacter(dx, dy) {
      const newX = characterPosition.x + dx;
      const newY = characterPosition.y + dy;
      if (canMoveTo(newX, newY)) {
        characterPosition.x = newX;
        characterPosition.y = newY;
        handleTileInteraction();
      }
    }

    // Check if the character can move to a position (collision detection)
    function canMoveTo(x, y) {
      const hitboxX = x + hitboxOffsetX;
      const hitboxY = y + hitboxOffsetY;
      const pointsToCheck = [
        { x: hitboxX, y: hitboxY },
        { x: hitboxX + hitboxWidth - 1, y: hitboxY },
        { x: hitboxX, y: hitboxY + hitboxHeight - 1 },
        { x: hitboxX + hitboxWidth - 1, y: hitboxY + hitboxHeight - 1 }
      ];
      return pointsToCheck.every(point => {
        const tileX = Math.floor(point.x / TILE_SIZE);
        const tileY = Math.floor(point.y / TILE_SIZE);
        if (tileY >= 0 && tileY < tileMap.length && tileX >= 0 && tileX < tileMap[0].length) {
          const tileValue = tileMap[tileY][tileX];
          if (tileValue === TILE_TYPES.WALL) return false;
          if (tileValue === TILE_TYPES.DOOR && inventory.keys <= 0) return false;
          return true;
        } else {
          return false;
        }
      });
    }

    // Handle interactions when the character steps on a tile
    function handleTileInteraction() {
      const hitboxX = characterPosition.x + hitboxOffsetX;
      const hitboxY = characterPosition.y + hitboxOffsetY;
      const tileX = Math.floor(hitboxX / TILE_SIZE);
      const tileY = Math.floor(hitboxY / TILE_SIZE);
      if (tileY >= 0 && tileY < tileMap.length && tileX >= 0 && tileX < tileMap[0].length) {
        const tileValue = tileMap[tileY][tileX];
        if (tileValue === TILE_TYPES.heart) {
          inventory.hearts++;
          updateInventoryDisplay();
          removeItem(tileX, tileY);
        } else if (tileValue === TILE_TYPES.TREASURE) {
          inventory.treasures += 1;
          removeItem(tileX, tileY);
          updateInventoryDisplay();
        } else if (tileValue === TILE_TYPES.COIN) {
          inventory.coins += 1;
          removeItem(tileX, tileY);
          updateInventoryDisplay();
        } else if (tileValue === TILE_TYPES.KEY) {
          inventory.keys += 1;
          removeItem(tileX, tileY);
          updateInventoryDisplay();
        } else if (tileValue === TILE_TYPES.AMMO) {
          inventory.ammo += 10;
          removeItem(tileX, tileY);
          updateInventoryDisplay();
        } else if (tileValue === TILE_TYPES.DOOR) {
          if (inventory.keys > 0) {
            inventory.keys -= 1;
            removeItem(tileX, tileY);
            updateInventoryDisplay();
          }
        }
      }
    }

    // Remove an item from the gameboard and update the tile
    function removeItem(x, y) {
      const tileKey = `${x}_${y}`;
      if (itemElements[tileKey]) {
        gameboard.removeChild(itemElements[tileKey]);
        delete itemElements[tileKey];
      }
      tileMap[y][x] = TILE_TYPES.FLOOR;
      updateTileAt(x, y);
    }

    // Update a single tile's appearance based on its type
    function updateTileAt(x, y) {
      const tileIndex = y * tileMap[0].length + x;
      const tile = tileElements[tileIndex];
      const tileValue = tileMap[y][x];
      if (tileValue === TILE_TYPES.WALL) {
        tile.style.backgroundColor = '#2B2B2B';
      } else if (tileValue === TILE_TYPES.TREASURE) {
        tile.style.backgroundColor = '#E0115F';
      } else if (tileValue === TILE_TYPES.COIN) {
        tile.style.backgroundColor = '#FFD700';
      } else if (tileValue === TILE_TYPES.KEY) {
        tile.style.backgroundColor = '#00FF00';
      } else if (tileValue === TILE_TYPES.AMMO) {
        tile.style.backgroundColor = '#0000FF';
      } else if (tileValue === TILE_TYPES.DOOR) {
        tile.style.backgroundColor = '#8B4513';
      } else {
        tile.style.backgroundColor = '#D0E4F5';
      }
      tile.style.opacity = tileOpacity;
    }

    // Update the camera position to center on the character
    function updateCameraPosition() {
      const camX = characterPosition.x - window.innerWidth / 2 + TILE_SIZE / 2;
      const camY = characterPosition.y - window.innerHeight / 2 + TILE_SIZE / 2;
      gameboard.style.transform = `translate(${-camX}px, ${-camY}px)`;
    }

    // Show the victory message overlay
    function showVictoryMessage() {
      const overlay = document.getElementById('overlay');
      overlay.innerHTML = "You Defeated the Maze!!!";
      overlay.style.display = "flex";
      gameOver = true;
    }

    // Show the death message overlay
    function showDeathMessage() {
      const overlay = document.getElementById('overlay');
      overlay.innerHTML = `
        <div>You died!!!!!!!!!!!!!!!!!</div>
        <div style="font-size: 24px; margin-top: 20px;">Press the Space Bar to restart</div>
      `;
      overlay.style.display = "flex";
      gameOver = true;
    }

    // ==============================
    // Map Generation Functions
    // ==============================
    // Generate the map, placing tiles, items, and enemy objects
    function generateMap() {
      gameboard.style.width = TILE_SIZE * tileMap[0].length + 'px';
      gameboard.style.height = TILE_SIZE * tileMap.length + 'px';
      gameboard.style.backgroundImage = `url('${backgroundImageURL}')`;
      gameboard.style.backgroundSize = '100% 100%';

      for (let y = 0; y < tileMap.length; y++) {
        for (let x = 0; x < tileMap[0].length; x++) {
          const tileValue = tileMap[y][x];
          // Create enemy objects based on tile type
          if (tileValue === TILE_TYPES.normalEnemy) {
            let enemy = {
              x: x * TILE_SIZE,
              y: y * TILE_SIZE,
              type: 'normal',
              width: TILE_SIZE,
              height: TILE_SIZE,
              vx: 2,
              vy: 2
            };
            enemies.push(enemy);
          } else if (tileValue === TILE_TYPES.bossEnemy) {
            let enemy = {
              x: x * TILE_SIZE,
              y: y * TILE_SIZE,
              type: 'boss',
              width: TILE_SIZE * 2,
              height: TILE_SIZE * 2,
              vx: 1.5,
              vy: 1.5,
              health: 3
            };
            enemies.push(enemy);
          } else if (tileValue === TILE_TYPES.pulseEnemy) {
            let enemy = {
              x: x * TILE_SIZE,
              y: y * TILE_SIZE,
              type: 'pulse',
              width: TILE_SIZE,
              height: TILE_SIZE,
              visible: true,
              lastPulseToggle: Date.now(),
              lastDamageTime: 0
            };
            enemies.push(enemy);
          }
          
          // Create heart element if tile is heart
          if (tileValue === TILE_TYPES.heart) {
            const heartElement = document.createElement('img');
            heartElement.src = 'https://physicsland.net/GD2Game1/heart.png';
            heartElement.style.position = 'absolute';
            heartElement.style.width = TILE_SIZE + 'px';
            heartElement.style.height = TILE_SIZE + 'px';
            heartElement.style.left = x * TILE_SIZE + 'px';
            heartElement.style.top = y * TILE_SIZE + 'px';
            heartElement.style.zIndex = '50';
            gameboard.appendChild(heartElement);
            itemElements[`${x}_${y}`] = heartElement;
          }

          // Create tile element
          const tile = document.createElement('div');
          tile.style.position = 'absolute';
          tile.style.width = TILE_SIZE + 'px';
          tile.style.height = TILE_SIZE + 'px';
          tile.style.left = x * TILE_SIZE + 'px';
          tile.style.top = y * TILE_SIZE + 'px';
          tile.style.zIndex = '10';

          if (tileValue === TILE_TYPES.WALL) {
            tile.style.backgroundColor = '#2B2B2B';
          } else if (tileValue === TILE_TYPES.TREASURE) {
            tile.style.backgroundColor = '#E0115F';
          } else if (tileValue === TILE_TYPES.COIN) {
            tile.style.backgroundColor = '#FFD700';
          } else if (tileValue === TILE_TYPES.KEY) {
            tile.style.backgroundColor = '#00FF00';
          } else if (tileValue === TILE_TYPES.AMMO) {
            tile.style.backgroundColor = '#0000FF';
          } else if (tileValue === TILE_TYPES.DOOR) {
            tile.style.backgroundColor = '#8B4513';
          } else {
            tile.style.backgroundColor = '#D0E4F5';
          }

          tile.style.opacity = tileOpacity;
          gameboard.appendChild(tile);
          tileElements.push(tile);

          // Create item image for specific tile types
          if ([TILE_TYPES.TREASURE, TILE_TYPES.COIN, TILE_TYPES.KEY, TILE_TYPES.AMMO, TILE_TYPES.DOOR].includes(tileValue)) {
            const item = document.createElement('img');
            item.style.position = 'absolute';
            item.style.width = TILE_SIZE + 'px';
            item.style.height = TILE_SIZE + 'px';
            item.style.left = x * TILE_SIZE + 'px';
            item.style.top = y * TILE_SIZE + 'px';
            item.style.zIndex = '50';
            if (tileValue === TILE_TYPES.TREASURE) {
              item.src = 'https://physicsland.net/GD2Game1/potion.png';
            } else if (tileValue === TILE_TYPES.COIN) {
              item.src = 'https://physicsland.net/GD2Game1/coin.png';
            } else if (tileValue === TILE_TYPES.KEY) {
              item.src = 'https://physicsland.net/GD2Game1/key.png';
            } else if (tileValue === TILE_TYPES.AMMO) {
              item.src = 'https://physicsland.net/GD2Game1/ammo.png';
            } else if (tileValue === TILE_TYPES.DOOR) {
              item.src = 'https://physicsland.net/GD2Game1/lock.png';
            }
            gameboard.appendChild(item);
            itemElements[`${x}_${y}`] = item;
          }
        }
      }
    }

    // Update opacity for all tiles
    function updateAllTilesOpacity() {
      tileElements.forEach(tile => {
        tile.style.opacity = tileOpacity;
      });
    }

    // ==============================
    // Event Listeners
    // ==============================
    // Restart the game when the Space key is pressed after death
    document.addEventListener('keydown', function(event) {
      if (event.code === "Space") {
        const overlay = document.getElementById('overlay');
        if (overlay.style.display !== "none" && inventory.hearts <= 0) {
          resetGame();
        }
      }
    });

    // Handle movement and action key presses
    let keysPressed = {};
    document.addEventListener('keydown', function (event) {
      if (event.key === 'ArrowUp' || event.key === 'Up') {
        keysPressed['up'] = true;
        setAnimation('up');
      } else if (event.key === 'ArrowDown' || event.key === 'Down') {
        keysPressed['down'] = true;
        setAnimation('down');
      } else if (event.key === 'ArrowLeft' || event.key === 'Left') {
        keysPressed['left'] = true;
        setAnimation('left');
      } else if (event.key === 'ArrowRight' || event.key === 'Right') {
        keysPressed['right'] = true;
        setAnimation('right');
      } else if (event.key.toLowerCase() === 'z') {
        if (inventory.ammo > 0 && !isPerformingAction && !isOnCooldown) {
          inventory.ammo--;
          updateInventoryDisplay();
          isPerformingAction = true;
          isOnCooldown = true;
          character.classList.remove(...allClasses);
          character.classList.add("shoot-" + currentDirection);
          setTimeout(function () {
            character.classList.remove("shoot-" + currentDirection);
            character.classList.add("idle-" + currentDirection);
            isPerformingAction = false;
            // Spawn arrow after shooting animation
            shootArrow();
          }, 200);   //Arrow delay after pressing Z key
          setTimeout(function () {
            isOnCooldown = false;
          }, 200); //Cool down before shooting another arrow
        }
      } else if (event.key.toLowerCase() === 'x') {
        if (inventory.treasures > 0 && !isPerformingAction && !isOnCooldown) {
          inventory.treasures--;
          updateInventoryDisplay();
          isPerformingAction = true;
          isOnCooldown = true;
          character.classList.remove(...allClasses);
          character.classList.add("conjure-" + currentDirection);
          createSpellCastEffect(characterPosition.x, characterPosition.y, 600);
          activateInvulnerability(4000); //duration of invulnerability
          setTimeout(function () {
            character.classList.remove("conjure-" + currentDirection);
            character.classList.add("idle-" + currentDirection);
            isPerformingAction = false;
          }, 600);
          setTimeout(function () {
            isOnCooldown = false;
          }, 600);
        }
      }
    });

    // Reset movement when keys are released
    document.addEventListener('keyup', function (event) {
      if (event.key === 'ArrowUp' || event.key === 'Up') {
        keysPressed['up'] = false;
      } else if (event.key === 'ArrowDown' || event.key === 'Down') {
        keysPressed['down'] = false;
      } else if (event.key === 'ArrowLeft' || event.key === 'Left') {
        keysPressed['left'] = false;
      } else if (event.key === 'ArrowRight' || event.key === 'Right') {
        keysPressed['right'] = false;
      }
      if (!keysPressed['up'] && !keysPressed['down'] && !keysPressed['left'] && !keysPressed['right']) {
        character.classList.remove(...allClasses);
        character.classList.add('idle-' + currentDirection);
      }
    });

    // Opacity slider control
    const opacitySlider = document.getElementById('opacitySlider');
    const disableOpacitySlider = document.getElementById('disableOpacitySlider');

    opacitySlider.addEventListener('input', function() {
      if (!disableOpacitySlider.checked) {
        tileOpacity = parseFloat(this.value);
        updateAllTilesOpacity();
      }
    });

    if (disableOpacitySlider.checked) {
      opacitySlider.disabled = true;
      tileOpacity = parseFloat(opacitySlider.min);
      updateAllTilesOpacity();
    }

    disableOpacitySlider.addEventListener('change', function() {
      if (this.checked) {
        opacitySlider.disabled = true;
        tileOpacity = parseFloat(opacitySlider.min);
        updateAllTilesOpacity();
      } else {
        opacitySlider.disabled = false;
        tileOpacity = parseFloat(opacitySlider.value);
        updateAllTilesOpacity();
      }
    });

    // ==============================
    // Game Loop
    // ==============================
    function gameLoop() {
      if (gameOver) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      updateArrows();
      updateEnemies();
      renderArrows();
      renderEnemies();
      checkCharacterEnemyCollision();
      let moved = false;
      let dx = 0;
      let dy = 0;
      if (keysPressed['up']) {
        dy -= MOVE_SPEED;
        currentDirection = 'up';
        moved = true;
      }
      if (keysPressed['down']) {
        dy += MOVE_SPEED;
        currentDirection = 'down';
        moved = true;
      }
      if (keysPressed['left']) {
        dx -= MOVE_SPEED;
        currentDirection = 'left';
        moved = true;
      }
      if (keysPressed['right']) {
        dx += MOVE_SPEED;
        currentDirection = 'right';
        moved = true;
      }
      if (moved) {
        moveCharacter(dx, dy);
      }
      updateCharacterPosition();
      updateHitboxVisual();
      updateCameraPosition();
      if (!gameOver && enemies.filter(e => e.type === 'boss').length === 0) {
        showVictoryMessage();
      }
      requestAnimationFrame(gameLoop);
    }

    // ==============================
    // Initialize Game
    // ==============================
    generateMap();
    updateCameraPosition();
    updateInventoryDisplay();
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
