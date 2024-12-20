---
layout: base
title: Snake
---

<style>
    * {
        box-sizing: border-box;
    }

    body {
        font-family: 'Comic Sans MS', sans-serif;
        background: #f1f1f1;
        margin: 0;
        padding: 0;
        overflow: hidden;
    }

    .wrap {
        margin: 0 auto;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
    }

    canvas {
        border: 8px solid #FFFFFF;
        border-radius: 20px;
        background: radial-gradient(circle, rgba(0, 0, 0, 0.8), rgb(0, 158, 84)); 
        box-shadow: 0px 0px 30px rgba(0, 0, 0, 0.5);
        animation: fadeIn 1s ease-in-out;
    }

    canvas:focus {
        outline: none;
    }

    h2 {
        font-size: 3rem;
        text-align: center;
        color: #2d3e50;
        margin-top: 20px;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
    }

    #score_value {
        font-size: 2.5rem;
        color: #ff5733;
    }

    #menu, #gameover, #setting {
        font-size: 1.5rem;
        color: #ffeb3b;
        animation: slideIn 1s ease-in-out;
    }

    #menu p, #gameover p, #setting p {
        font-size: 1.8rem;
    }

    #menu a, #gameover a, #setting a {
        font-size: 2.2rem;
        display: block;
        color: #ffeb3b;
        text-decoration: none;
        text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        transition: transform 0.3s, color 0.3s;
    }

    #menu a:hover, #gameover a:hover, #setting a:hover {
        transform: scale(1.1);
        color: #03a9f4;
    }

    #setting label {
        font-size: 1.4rem;
        cursor: pointer;
        padding: 8px 15px;
        background: rgb(18, 138, 68);
        color: #000;
        border-radius: 8px;
        margin: 0 5px;
        transition: background 0.3s, color 0.3s;
    }

    #setting input:checked + label {
        background: #03a9f4;
        color: white;
    }

    @keyframes fadeIn {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    @keyframes slideIn {
        from {
            transform: translateY(-30px);
            opacity: 0;
        }
        to {
            transform: translateY(0);
            opacity: 1;
        }
    }
        img {
        image-rendering: smooth;
    }
</style>



<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">New Game</a>
            <a id="setting_menu" class="link-alert">Settings</a>
        </div>
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">New Game</a>
            <a id="setting_menu1" class="link-alert">Settings</a>
        </div>
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">New Game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="120" checked />
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75" />
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35" />
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked />
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0" />
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
(function () {
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");
    const ele_score = document.getElementById("score_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");
    const SCREEN_MENU = -1, SCREEN_SNAKE = 0, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2;
    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");
    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");
    const BLOCK = 20;
    const ROWS = 16;
    const COLS = 16;
    let SCREEN = SCREEN_MENU;
    let snake;
    let snake_dir = 2; // Initial direction: down (facing down)
    let snake_next_dir = 2;
    let snake_speed = 120;
    let food = { x: 0, y: 0 };
    let score = 0;
    let wall = 1; // Wall on by default

    // Load fruit and snake head images
    const fruitImage = new Image();
    fruitImage.src = '{{site.baseurl}}/images/applepng.ico'; // Replace with your image path
    
    const snakeHeadImage = new Image();
    snakeHeadImage.src = '{{site.baseurl}}/images/snakehead.png'; // Replace with your image path

    const showScreen = (screen_opt) => {
        SCREEN = screen_opt;
        screen_menu.style.display = screen_opt === SCREEN_MENU ? "block" : "none";
        screen_game_over.style.display = screen_opt === SCREEN_GAME_OVER ? "block" : "none";
        screen_setting.style.display = screen_opt === SCREEN_SETTING ? "block" : "none";
        canvas.style.display = screen_opt === SCREEN_SNAKE ? "block" : "none";
    };

    window.onload = () => {
        showScreen(SCREEN_MENU);  // Show the menu on page load
        button_new_game.onclick = button_new_game1.onclick = button_new_game2.onclick = newGame;
        button_setting_menu.onclick = button_setting_menu1.onclick = () => showScreen(SCREEN_SETTING);

        for (let i = 0; i < speed_setting.length; i++) {
            speed_setting[i].addEventListener("click", () => {
                snake_speed = parseInt(speed_setting[i].value);
            });
        }

        for (let i = 0; i < wall_setting.length; i++) {
            wall_setting[i].addEventListener("click", () => {
                wall = parseInt(wall_setting[i].value);
            });
        }

        // Prevent the page from scrolling when arrow keys are pressed
        window.addEventListener("keydown", function(e) {
            if (e.keyCode >= 37 && e.keyCode <= 40) {
                e.preventDefault();
            }
        });
    };

    const newGame = () => {
        showScreen(SCREEN_SNAKE);
        canvas.focus();
        score = 0;
        ele_score.textContent = score;
        snake = [{ x: 5, y: 5 }];
        snake_next_dir = 2;
        addFood();
        window.addEventListener("keydown", changeDirection);
        gameLoop();
    };

    const gameLoop = () => {
        if (SCREEN !== SCREEN_SNAKE) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Draw grid
        for (let i = 0; i < ROWS; i++) {
            for (let j = 0; j < COLS; j++) {
                ctx.fillStyle = (i + j) % 2 === 0 ? "#76d7c4" : "#58b3a5"; // Light green tones
                ctx.fillRect(j * BLOCK, i * BLOCK, BLOCK, BLOCK);
            }
        }

        // Move snake
        moveSnake();

        // Draw food (with image)
        ctx.drawImage(fruitImage, food.x * BLOCK, food.y * BLOCK, BLOCK, BLOCK);

        // Draw snake head with rotation
        drawRotatedHead(snake[0].x * BLOCK, snake[0].y * BLOCK, snake_dir);

        // Draw snake body (still using rectangles)
        for (let i = 1; i < snake.length; i++) {
            ctx.fillStyle = "#4594b2"; // Light blue body
            ctx.fillRect(snake[i].x * BLOCK, snake[i].y * BLOCK, BLOCK, BLOCK);
        }

        setTimeout(gameLoop, snake_speed);
    };

    const drawRotatedHead = (x, y, direction) => {
        ctx.save();
        ctx.translate(x + BLOCK / 2, y + BLOCK / 2); // Move to the center of the block
        let angle = 0;

        // Set rotation based on direction
        switch (direction) {
            case 0: angle = -Math.PI / 2; break; // Up
            case 1: angle = 0; break; // Right
            case 2: angle = Math.PI / 2; break; // Down
            case 3: angle = Math.PI; break; // Left
        }

        ctx.rotate(angle); // Apply rotation
        ctx.drawImage(snakeHeadImage, -BLOCK / 2, -BLOCK / 2, BLOCK, BLOCK); // Draw the head
        ctx.restore();
    };

    const moveSnake = () => {
        const head = { ...snake[0] };
        snake_dir = snake_next_dir;

        switch (snake_dir) {
            case 0: head.y--; break; // Up
            case 1: head.x++; break; // Right
            case 2: head.y++; break; // Down
            case 3: head.x--; break; // Left
        }

        // Check wall collisions
        if (wall === 1 && (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS)) {
            showScreen(SCREEN_GAME_OVER);
            return;
        }

        // Wrap around if wall is off
        head.x = (head.x + COLS) % COLS;
        head.y = (head.y + ROWS) % ROWS;

        // Check self-collision
        if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
            showScreen(SCREEN_GAME_OVER);
            return;
        }

        // Add new head
        snake.unshift(head);

        // Check for food
        if (head.x === food.x && head.y === food.y) {
            score++;
            ele_score.textContent = score;
            addFood();
        } else {
            snake.pop(); // Remove tail
        }
    };

    const addFood = () => {
        food.x = Math.floor(Math.random() * COLS);
        food.y = Math.floor(Math.random() * ROWS);
    };

    const changeDirection = (event) => {
        switch (event.keyCode) {
            case 37: if (snake_dir !== 1) snake_next_dir = 3; break; // Left
            case 38: if (snake_dir !== 2) snake_next_dir = 0; break; // Up
            case 39: if (snake_dir !== 3) snake_next_dir = 1; break; // Right
            case 40: if (snake_dir !== 0) snake_next_dir = 2; break; // Down
            case 32: if (SCREEN !== SCREEN_SNAKE) newGame(); break;  // Space
        }
    };
})();
</script>