<script>
  import * as PIXI from "pixi.js";

  let renderer;
  let stage;
  let gameOver = false;

  let widthGrowStuff = 0;
  let heightGrowStuff = 0;

  // Create the application helper and add its render target to the page
  let app = new PIXI.Application({
    resizeTo: window,
    backgroundColor: 0x0000,
  });
  document.body.appendChild(app.view);

  app.stage.interactive = true;
  app.stage.hitArea = app.screen;

  // get app screen size
  const { width, height } = app.screen;

  function createGraphic(color, size) {
    const graphic = new PIXI.Graphics();
    graphic.beginFill(color);
    graphic.drawRect(0, 0, size.w, size.y);
    // graphic.filters = [new PIXI.filters.GlowFilter(15, 2, 1, 0xff0000, 0.5)];
    graphic.endFill();
    // make it glow
    return graphic;
  }

  function createOutlineGraphic(color, size) {
    const graphic = new PIXI.Graphics();
    graphic.lineStyle(4, color, 1);
    graphic.drawRect(0, 0, size.w, size.y);
    graphic.endFill();
    // make it glow
    return graphic;
  }

  const magicShape = app.stage.addChild(
    createOutlineGraphic(0xffff, { w: 60, y: 260 })
  );

  magicShape.position.set(
    Math.random() * width - 25,
    Math.random() * height - 25
  );

  const eatable = app.stage.addChild(createGraphic(0xb76535, { w: 10, y: 10 }));
  eatable.position.set(100, 100);

  const heightEatable = app.stage.addChild(
    createGraphic(0xb00555, { w: 10, y: 10 })
  );
  heightEatable.position.set(300, 400);

  const enemy = app.stage.addChild(createGraphic(0x007500, { w: 30, y: 30 }));
  enemy.position.set(Math.random() * width, Math.random() * height);

  const vector = app.stage.addChild(createGraphic(0xffff00, { w: 50, y: 50 }));

  const minion1 = app.stage.addChild(createGraphic(0xb00005, { w: 15, y: 15 }));

  // position minion1 on the top right corner
  minion1.position.set(width - 50, Math.random() * height);

  let baseSpeed = 20;

  window.addEventListener("keydown", (e) => {
    console.log(e);
    // check if the key pressed is the right arrow
    if (e.key === "ArrowRight" || e.key === "d") {
      // move the sprite to the right
      vector.x += baseSpeed;
    }
    // check if the key pressed is the left arrow
    if (e.key === "ArrowLeft" || e.key === "a") {
      // move the sprite to the left
      vector.x -= baseSpeed;
    }
    // check if the key pressed is the up arrow
    if (e.key === "ArrowUp" || e.key === "w") {
      // move the sprite to the up
      vector.y -= baseSpeed;
    }
    // check if the key pressed is the down arrow
    if (e.key === "ArrowDown" || e.key === "s") {
      // move the sprite to the down
      vector.y += baseSpeed;
    }

    // c increases height, v increases width, x decreases height, z decreases width

    // check if c is pressed
    if (e.key === "c") {
      // move the sprite to the down
      if (heightGrowStuff > 0) {
        vector.height += 50;
        heightGrowStuff -= 1;
      }
    }

    // if x is pressed decrease the height
    if (e.key === "x") {
      if (heightGrowStuff > 0) {
        vector.height -= 50;
        heightGrowStuff -= 1;
      }
    }

    // if v is pressed
    if (e.key === "v") {
      // move the sprite to the down
      if (widthGrowStuff > 0) {
        vector.width += 50;
        widthGrowStuff -= 1;
      }
    }

    // if z is pressed
    if (e.key === "z") {
      if (widthGrowStuff > 0) {
        vector.width -= 50;
        widthGrowStuff -= 1;
      }
    }
  });

  // Add a ticker callback to move the sprite back and forth
  let elapsed = 0.0;
  app.ticker.add((delta) => {
    elapsed += delta;
    vector.x = vector.x + 0.3;

    //  check if vector is colliding with eatable
    if (eatable.visible) {
      if (
        vector.x < eatable.x + eatable.width &&
        vector.x + vector.width > eatable.x &&
        vector.y < eatable.y + eatable.height &&
        vector.y + vector.height > eatable.y
      ) {
        // collision detected!
        console.log("collision detected");
        // if collision detected increase the size of the vector and hide the eatable
        // vector.width += 50;

        widthGrowStuff += 1;
        // change eatable position
        eatable.x = Math.floor(Math.random() * 1000);
        eatable.y = Math.floor(Math.random() * 1000);
        moveIncreaseDecreaceItem({ widht: true });
      }
    }

    if (enemy.visible) {
      if (
        vector.x < enemy.x + enemy.width &&
        vector.x + vector.width > enemy.x &&
        vector.y < enemy.y + enemy.height &&
        vector.y + vector.height > enemy.y
      ) {
        // if collision detected increase the size of the vector and hide the eatable
        if (vector.widht > 50) {
          vector.width -= 50;
        } else if (vector.height > 50) {
          vector.height -= 50;
        } else {
          gameOver = true;

          alert("se acabó");
          // reset the game
        }

        enemy.x = Math.floor(Math.random() * 1000);
        enemy.y = Math.floor(Math.random() * 1000);
      }
    }

    if (heightEatable.visible) {
      if (
        vector.x < heightEatable.x + heightEatable.width &&
        vector.x + vector.width > heightEatable.x &&
        vector.y < heightEatable.y + heightEatable.height &&
        vector.y + vector.height > heightEatable.y
      ) {
        // collision detected!
        console.log("collision detected");
        // if collision detected increase the size of the vector and hide the eatable
        // vector.width += 50;

        heightGrowStuff += 1;
        // change eatable position
        heightEatable.x = Math.floor(Math.random() * 1000);
        heightEatable.y = Math.floor(Math.random() * 1000);
        moveIncreaseDecreaceItem({ height: true });
      }
    }
  });

  app.ticker.add(() => {
    //  check if vector is the same width and height as magic box and if at least 90% of it is inside magicbox
    if (
      vector.width == magicShape.width &&
      vector.height == magicShape.height &&
      vector.x >= magicShape.x &&
      vector.y >= magicShape.y &&
      vector.x + vector.width <= magicShape.x + magicShape.width &&
      vector.y + vector.height <= magicShape.y + magicShape.height
    ) {
      alert("next level");
      // if so, show the next level button
    }
  });

  let secondsDelta = 0;
  let start = Date.now();
  setInterval(function () {
    var delta = Date.now() - start; // milliseconds elapsed since start

    secondsDelta = Math.floor(delta / 1000); // in seconds
  }, 1000); // update about every second

  app.ticker.add((delta) => {
    // make minion1 follow the vector

    minion1.x = minion1.x - 5;

    // check if minion1 collides with enemy
    if (enemy.visible) {
      if (
        minion1.x < enemy.x + enemy.width &&
        minion1.x + minion1.width > enemy.x &&
        minion1.y < enemy.y + enemy.height &&
        minion1.y + minion1.height > enemy.y
      ) {
        minion1.x = width;
        minion1.y = vector.y + 20;
      }
    }

    // check if minion1 collides with vector
    if (
      minion1.x < vector.x + vector.width &&
      minion1.x + minion1.width > vector.x &&
      minion1.y < vector.y + vector.height &&
      minion1.y + minion1.height > vector.y
    ) {
      if (vector.widht > 50) {
        vector.width -= 50;
      } else if (vector.height > 50) {
        vector.height -= 50;
      } else {
        gameOver = true;
        vector.width = 50;
        vector.height = 50;
        vector.x = 0;
        vector.y = 0;
        enemy.width = 50;
        enemy.height = 50;
        enemy.visible = true;
        heightEatable.visible = true;
        eatable.visible = true;
        minion1.visible = true;
        secondsDelta = 0;
        // start = Date.now();
        alert("se acabó");
      }
    }

    if (minion1.x < 0) {
      minion1.x = width;
      minion1.y = vector.y + 20;
    }
  });

  function moveIncreaseDecreaceItem({ widht = false, height = false }) {
    enemy.visible = true;
    if (widht) {
      enemy.width += 50;
    }
    if (height) {
      enemy.height += 50;
    }
    enemy.x = Math.floor(Math.random() * 1000);
    enemy.y = Math.floor(Math.random() * 1000);
  }
</script>

<!-- videogame style counter absolute positioned on top right tailwind -->

<div class="w-scren h-screen">
  <div class="absolute right-10 top-5 text-white font-mono">
    <div class="space-y-2">
      <div class="grid grid-cols-3  border-white border-2 shadow-md ">
        <div class="col-span-1">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            fill="currentColor"
            class="w-6 h-6"
          >
            <path
              fill-rule="evenodd"
              d="M15.97 2.47a.75.75 0 011.06 0l4.5 4.5a.75.75 0 010 1.06l-4.5 4.5a.75.75 0 11-1.06-1.06l3.22-3.22H7.5a.75.75 0 010-1.5h11.69l-3.22-3.22a.75.75 0 010-1.06zm-7.94 9a.75.75 0 010 1.06l-3.22 3.22H16.5a.75.75 0 010 1.5H4.81l3.22 3.22a.75.75 0 11-1.06 1.06l-4.5-4.5a.75.75 0 010-1.06l4.5-4.5a.75.75 0 011.06 0z"
              clip-rule="evenodd"
            />
          </svg>
        </div>
        <p class="text-yellow-300">{widthGrowStuff}</p>
      </div>
      <div class="grid grid-cols-3  border-white border-2 shadow-md">
        <div>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            fill="currentColor"
            class="w-6 h-6"
          >
            <path
              fill-rule="evenodd"
              d="M6.97 2.47a.75.75 0 011.06 0l4.5 4.5a.75.75 0 01-1.06 1.06L8.25 4.81V16.5a.75.75 0 01-1.5 0V4.81L3.53 8.03a.75.75 0 01-1.06-1.06l4.5-4.5zm9.53 4.28a.75.75 0 01.75.75v11.69l3.22-3.22a.75.75 0 111.06 1.06l-4.5 4.5a.75.75 0 01-1.06 0l-4.5-4.5a.75.75 0 111.06-1.06l3.22 3.22V7.5a.75.75 0 01.75-.75z"
              clip-rule="evenodd"
            />
          </svg>
        </div>
        <p class="text-yellow-300">{heightGrowStuff}</p>
      </div>
    </div>
  </div>

  <!-- time counter -->
  <div>
    <div class="absolute right-10 top-20 text-white font-mono">
      <div class="space-y-2">
        <div class="grid grid-cols-3  border-white border-2 shadow-md ">
          <div class="col-span-1">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24"
              fill="currentColor"
              class="w-6 h-6"
            >
              <path
                fill-rule="evenodd"
                d="M12 2a10 10 0 100 20 10 10 0 000-20zm0 1.5a8.5 8.5 0 110 17 8.5 8.5 0 010-17zm0 3.5a.75.75 0 00-.75.75v4.69l-2.22 2.22a.75.75 0 101.06 1.06l2.97-2.97a.75.75 0 00-.53-1.28h-.01V6.25a.75.75 0 00-.75-.75z"
                clip-rule="evenodd"
              />
            </svg>
          </div>
          <p class="text-yellow-300">{secondsDelta}</p>
        </div>
      </div>
    </div>
  </div>
</div>
