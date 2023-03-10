{% include arcadeSubmenu.html %}

<html lang="en">
<head>
    <meta charset="UTF-8">
    <style>
        h1 {
            font-size: 32pt;
            text-align: center;
            margin-bottom: 20px;
        }
        body{
            background-color: dimgray;
        }
        #startScreen {
            text-align: center;
            font-size: 13.5pt;
            position: absolute;
            width: 74.5%;
            height: 400px;
            background-color: #202020;
            border-radius: 20px;
            padding: 10%;
            z-index: 99;
        }
        #endScreen {
            display: none;
            text-align: center;
            font-size: 13.5pt;
            position: absolute;
            width: 74.5%;
            height: 400px;
            background-color: #202020;
            border-radius: 20px;
            padding: 10%;
            z-index: 99;
        }
        .startGame {
            outline: none;
            -webkit-tap-highlight-color: transparent;
            font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
            font-size: 30px;
            position: inline;
            width: 60%;
            margin-left: 20%;
            margin-right: 20%;
            height: 100px;
            margin-top: 100px;
            margin-bottom: 200px;
            border-radius: 8px;
            background-color: #302f2f;
            color: #f1cc0c;
            border: none;
            transition-duration: 0.3s;
        }
        .startGame:hover {
            color: #242424;
            background-color: #f1cc0c;
        }
        .tokenicon {
            width: 28px;
            margin-top: -5px;
            vertical-align: middle;
        }
        #pong{
            border: 2px solid #f1cc0c;
            position: relative;
            margin :auto;
            top:0;
            right:0;
            left:3.5%;
            bottom:0;
        }
            @keyframes fadeOut {
        from {
            opacity: 1;
        }
        to {
            opacity: 0;
        }
        }
        .animater {
            animation: fadeOut 0.4s forwards;
        }
    </style>
</head>
<body>
<div class="outer">
    <h1>Pong</h1>
    <div id="startScreen">
        <p>This game costs <span style="color: #f1cc0c;">10 tokens</span> to play. <br> First to 7 points wins. If you win, you earn <span style="color: #f1cc0c;">25 tokens</span> (<span style="color: #f1cc0c;">15 token</span> profit). <br> User paddle is on the left. AI paddle is on the right. <br> Control your paddle with mouse movements up and down in the yellow box!</p>
        <button type="button" class="startGame" id="startGame" value="" onclick="startGame()">Start Game for 10 <img class="tokenicon" src="{{ site.baseurl }}/images/AJToken_60x60.png"></button>
    </div>
    <div id="endScreen">
        <p id='endText'></p>
        <button type="button" class="startGame" id='tryAgain' value="" onclick="location.reload()">Play Again</button>
    </div>
    <canvas id="pong" width="600" height="400"></canvas>
</div>
<script>
// gets the amount of tokens the user has
function getTokens() {
    const id = localStorage.getItem('currentUser');
    fetch('https://ajarcade.duckdns.org/api/players/')
        .then(response => {
            // trap error response from Web API
            if (response.status !== 200) {
                const message = 'Fetch error: ' + response.status + " " + response.statusText;
                alert(message);
                return;
            }
            // Valid response will contain json data
            response.json().then(data => {
                // iterate through the whole database and find a record that matches the uid
                for (i in data) {
                    if (data[i].uid == id) {
                        localStorage.setItem('tokenAmt', data[i].tokens);
                    }
                }
            })
        })
}
// removes 'amt' tokens from the user's tokens attribute (in this case, amt = 10)
function remTokens(amt) {
    const id = localStorage.getItem('currentUser');
    // update the user's token amount
    getTokens();
    var tokenAmt = localStorage.getItem('tokenAmt');
    newAmt = tokenAmt-amt;
    fetch('https://ajarcade.duckdns.org/api/players/update', {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            "uid": id,
            "data": {"tokens": newAmt}
        })
    })
        .then(res => {
            return res.json()
        })
        .then(data => console.log(data))
    setTimeout(function() {
        getTokens();
    }, 500);
}
// adds 'amt' tokens to the user's tokens attribute (called after the game is over and only if user won) (In this case, amt = 25)
function addTokens(amt) {
    const id = localStorage.getItem('currentUser');
    // update the user's token amount
    getTokens();
    tokenAmt = localStorage.getItem('tokenAmt');
    newAmt = parseFloat(tokenAmt) + parseFloat(amt);
    fetch('https://ajarcade.duckdns.org/api/players/update', {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            "uid": id,
            "data": {"tokens": newAmt}
        })
        }).then(res => {
        return res.json()
        })
        .then(data => console.log(data))
    setTimeout(function() {
        getTokens();
    }, 500);
}
// select canvas element
const canvas = document.getElementById("pong");
// getContext of canvas = methods and properties to draw and do a lot of thing to the canvas
const ctx = canvas.getContext('2d');
// load sounds
let hit = new Audio();
let wall = new Audio();
let userScore = new Audio();
let comScore = new Audio();
hit.src = "sounds/hit.mp3";
wall.src = "sounds/wall.mp3";
comScore.src = "sounds/comScore.mp3";
userScore.src = "sounds/userScore.mp3";
// Ball object
const ball = {
    x : canvas.width/2,
    y : canvas.height/2,
    radius : 10,
    velocityX : 5,
    velocityY : 5,
    speed : 7,
    color : "WHITE"
}
// User Paddle
const user = {
    x : 0, // left side of canvas
    y : (canvas.height - 100)/2, // -100 the height of paddle
    width : 10,
    height : 100,
    score : 0,
    color : "WHITE"
}
// COM Paddle
const com = {
    x : canvas.width - 10, // - width of paddle
    y : (canvas.height - 100)/2, // -100 the height of paddle
    width : 10,
    height : 100,
    score : 0,
    color : "WHITE"
}
// NET
const net = {
    x : (canvas.width - 2)/2,
    y : 0,
    height : 10,
    width : 2,
    color : "WHITE"
}
// draw a rectangle, will be used to draw paddles
function drawRect(x, y, w, h, color){
    ctx.fillStyle = color;
    ctx.fillRect(x, y, w, h);
}
// draw circle, will be used to draw the ball
function drawArc(x, y, r, color){
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.arc(x,y,r,0,Math.PI*2,true);
    ctx.closePath();
    ctx.fill();
}
// listening to the mouse
canvas.addEventListener("mousemove", getMousePos);
function getMousePos(evt){
    let rect = canvas.getBoundingClientRect();
    user.y = evt.clientY - rect.top - user.height/2;
}
// when COM or USER scores, we reset the ball
function resetBall(){
    ball.x = canvas.width/2;
    ball.y = canvas.height/2;
    ball.velocityX = -ball.velocityX;
    ball.speed = 7;
}
// draw the net
function drawNet(){
    for(let i = 0; i <= canvas.height; i+=15){
        drawRect(net.x, net.y + i, net.width, net.height, net.color);
    }
}
// draw text
function drawText(text,x,y){
    ctx.fillStyle = "#FFF";
    ctx.font = "75px fantasy";
    ctx.fillText(text, x, y);
}
// collision detection
function collision(b,p){
    p.top = p.y;
    p.bottom = p.y + p.height;
    p.left = p.x;
    p.right = p.x + p.width;
    b.top = b.y - b.radius;
    b.bottom = b.y + b.radius;
    b.left = b.x - b.radius;
    b.right = b.x + b.radius;
    return p.left < b.right && p.top < b.bottom && p.right > b.left && p.bottom > b.top;
}
// update function, the function that does all calculations
function update(){
    // Establishes a key/value pair to store in local storage
    localStorage.setItem('pongStatus', null)
    // change the score of players, if the ball goes to the left "ball.x<0" computer win, else if "ball.x > canvas.width" the user win
    if( ball.x - ball.radius < 0 ){
        com.score++;
        comScore.play();
        resetBall();
        if (com.score > 6) {
            // We can use this item to tell who won the game (false = computer won, and vice versa)
            localStorage.setItem('pongStatus', 'false');
        }
        else {
            // We don't want to edit local storage if the game isn't really over
            localStorage.setItem('pongStatus', null);
        }
    }
    else if( ball.x + ball.radius > canvas.width){
        user.score++;
        userScore.play();
        resetBall();
        if (user.score > 6) {
            // We can use this item to tell who won the game (true = user won, and vice versa)
            localStorage.setItem('pongStatus', 'true');
        }
        else {
            // We don't want to edit local storage if the game isn't really over
            localStorage.setItem('pongStatus', null);
        }
    }
    // the ball has a velocity
    ball.x += ball.velocityX;
    ball.y += ball.velocityY;
    // computer plays for itself, and we must be able to beat it
    // simple AI
    com.y += ((ball.y - (com.y + com.height/2)))*0.1;
    // when the ball collides with bottom and top walls we inverse the y velocity.
    if(ball.y - ball.radius < 0 || ball.y + ball.radius > canvas.height){
        ball.velocityY = -ball.velocityY;
        wall.play();
    }
    // we check if the paddle hit the user or the com paddle
    let player = (ball.x + ball.radius < canvas.width/2) ? user : com;
    // if the ball hits a paddle
    if(collision(ball,player)){
        // play sound
        hit.play();
        // we check where the ball hits the paddle
        let collidePoint = (ball.y - (player.y + player.height/2));
        // normalize the value of collidePoint, we need to get numbers between -1 and 1.
        // -player.height/2 < collide Point < player.height/2
        collidePoint = collidePoint / (player.height/2);
        // when the ball hits the top of a paddle we want the ball, to take a -45degees angle
        // when the ball hits the center of the paddle we want the ball to take a 0degrees angle
        // when the ball hits the bottom of the paddle we want the ball to take a 45degrees
        // Math.PI/4 = 45degrees
        let angleRad = (Math.PI/4) * collidePoint;
        // change the X and Y velocity direction
        let direction = (ball.x + ball.radius < canvas.width/2) ? 1 : -1;
        ball.velocityX = direction * ball.speed * Math.cos(angleRad);
        ball.velocityY = ball.speed * Math.sin(angleRad);
        // speed up the ball everytime a paddle hits it.
        ball.speed += 0.5;
    }
}
// render function, the function that does al the drawing
function render(){
    // clear the canvas
    drawRect(0, 0, canvas.width, canvas.height, "#000");
    // draw the user score to the left
    drawText(user.score,canvas.width/4,canvas.height/5);
    // draw the COM score to the right
    drawText(com.score,3*canvas.width/4,canvas.height/5);
    // draw the net
    drawNet();
    // draw the user's paddle
    drawRect(user.x, user.y, user.width, user.height, user.color);
    // draw the COM's paddle
    drawRect(com.x, com.y, com.width, com.height, com.color);
    // draw the ball
    drawArc(ball.x, ball.y, ball.radius, ball.color);
}
// number of frames per second
let framePerSecond = 50;
// Declare a global variable for 'loop'. This will contain the gameloop and make it accessible to game() in order to stop the gameloop when the game is over
var loop = null
//Hide start screen and call the game function 50 times every 1 Sec; save the gameloop to global variable 'loop'
function startGame() {
    let div = document.getElementById('startScreen')
    // starts animation to fade out the start screen
    div.classList.add('animater')
    // Re-assign loop to be the actual gameloop
    loop = setInterval(game,1000/framePerSecond)
    // Tells the program how many tokens the user currently has
    getTokens()
    // Takes away a cost of 10 tokens from the user's balance
    remTokens(10)
    // Reset the start screen
    setTimeout(function() {
        div.style.display = "none";
        div.classList.remove("animater");
    }, 500);
}
// This thing makes the game work (tells the game how to handle when the ball hits a wall, paddle, when someone scores, etc...)
function game(){
    update();
    render();
    // Declaring variables to HTML elements for ease-of-use
    const endscreen = document.getElementById('endScreen');
    const endtext = document.getElementById('endText');
    // local storage will tell us who wins. See definition of update() above. (pongStatus => true means user won, and false means computer won.)
    if (localStorage.getItem('pongStatus') === 'true') {
        // This is saying if the user won^^, then...
        // Stop the game
        clearInterval(loop)
        // add 25 tokens to the user's balance (this will come out to a net 15 tokens gained after paying 10 tokens)
        addTokens(25)
        // Save the new number of tokens to local storage
        getTokens()
        // Make the end screen visible
        endscreen.style.display = 'block'
        // Write text on end screen
        endtext.innerHTML = 'You won! :) <br> Congrats on earning 15 tokens!'
    }
    if (localStorage.getItem('pongStatus') === 'false') {
        // This is saying if the computer won^^, then...
        // Stop the game
        clearInterval(loop)
        // Make the end screen visible
        endscreen.style.display = 'block'
        // Write text on end screen
        endtext.innerHTML = 'You lost :( <br> 10 tokens down the drain';
        // NOTE: we don't reward the user tokens if they lose! (aka they wasted 10 tokens)
    }
}
</script>
</body>
</html>