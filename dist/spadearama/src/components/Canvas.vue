<script setup>
import { onMounted, ref,reactive } from 'vue';

const canvas = ref(null);
let context, canvasWidth, canvasHeight;
let isMouseUsed = false;


class Sprite {
    constructor(id,height,width){
        this.id = id;
        this.active = true;
        this.positionX = 0;
        this.positionY = 0;
        this.rotation = 0;
        this.height = height;
        this.width = width;
        this.velocityX = 0;
        this.velocityY = 0;
        this.angVel = 0
    }
    setActive(active){
        this.active = active;
    }
    setPosition(x,y){
        this.positionX = x;
        this.positionY = y;
    }
    setRotation(rotation){
        this.rotation = rotation;
    }
    setSize(height,width){
        this.height = height;
        this.width = width;
    }
    update(deltaTime){
        this.setPosition(
            this.positionX + (deltaTime * this.velocityX),
            this.positionY + (deltaTime * this.velocityY)
        );
        if(this.angVel != 0){
            this.setRotation(
                this.rotation + (this.angVel * deltaTime)
            );
        }
    }
}

let gameState = reactive({
  mouseX: 0,
  mouseY: 0,
  followBall: new Sprite(0, 20, 20),
  hue: 0, 
});

onMounted(() => {
    context = canvas.value.getContext('2d');
    canvasWidth = canvas.value.width;
    canvasHeight = canvas.value.height;

    context.fillStyle = '#000';
    context.fillRect(0, 0, canvasWidth, canvasHeight);

    canvas.value.addEventListener('mousemove', updateMousePos, false);
    canvas.value.addEventListener("mouseenter", () => isMouseUsed = true, false);
    canvas.value.addEventListener("mouseleave", () => isMouseUsed = false, false);
    canvas.value.addEventListener("click", () => {
    
    });
    gameState.followBall.setPosition(0,0);
});

var updateMousePos = function(event){
    //console.log('updateMousePos : mouseX', event.offsetX, 'mouseY',event.offsetY);
    gameState.mouseX =event.offsetX;
    gameState.mouseY =event.offsetY;
}

var onUpdate = function(deltaTime){
    gameState.followBall.update(deltaTime);
}

var drawBall = function(ball,color = '#FFF'){
    if (ball.active) {
        context.beginPath();
        context.arc(
            ball.positionX,
            ball.positionY,
            ball.height / 2,
            0,
            2 * Math.PI
        );
        context.fillStyle = color;
        context.fill();
    }
}

//rendering loop
var lastTime = 0;
//time is ms since landed on page  (float)
var vsyncLoop = function (time) {

    requestAnimationFrame(vsyncLoop);

    var deltaTime = (time-lastTime)/1000; //fractions of a second
    lastTime = time;

    onUpdate(deltaTime);

    //Clear old content
    //context.fillStyle = '#0002';
    //context.fillRect(0, 0, canvasWidth, canvasHeight);
    
    if(Math.abs(gameState.followBall.positionX - gameState.mouseX) > .005 || Math.abs(gameState.followBall.positionY - gameState.mouseY) > .005 ){
        gameState.followBall.setPosition(gameState.mouseX,gameState.mouseY);
        gameState.hue += 60 * deltaTime;
        if (gameState.hue >= 360) 
        {
            gameState.hue -= 360;
        }
    }
    const rainbowColor = `hsl(${gameState.hue}, 100%, 50%)`;
    drawBall(gameState.followBall,rainbowColor);
};

requestAnimationFrame(vsyncLoop);

</script>

<template>
    
  <div>
    <h1 v-if="gameState.hue<180">What you make will be beautiful</h1>
    <h1 v-else>What you make is garbage
    </h1>
    <p v-if="gameState.hue<40">"I feel like this is one of those art pieces in a gallery that you interact with for about 30 seconds, then say 'oh that's cool' then you never touch it again"-The designers</p>
    <p v-else-if="gameState.hue<80">"It’s not a bug, it’s an unexpected feature in an abstract experience." - The developers </p>
    <p v-else-if="gameState.hue<120">"True art is supposed to leave you feeling… vaguely confused." - The critics</p>
    <p v-else-if="gameState.hue<160">"Remember, if it’s incomprehensible, it’s probably profound." - The gallery curator</p>
    <p v-else-if="gameState.hue<200">"I think this piece symbolizes the artist’s struggle with deadlines." - The art school graduate</p>
    <p v-else-if="gameState.hue<240">"Ah, another masterpiece. Though it probably makes more sense if you tilt your head slightly." - The art connoisseur</p>
    <p v-else-if="gameState.hue < 280">"It's minimalist because… well, we ran out of ideas." - The creative director</p>
    <p v-else-if="gameState.hue < 320">"This piece speaks to the raw power of confusion in modern art." - The avant-garde enthusiast</p>
    <p v-else>"An exploration of colors that don't quite match but insist on getting along." - The color theorist</p>
</div>
    <canvas ref="canvas" id="game-canvas" width="512" height="384"></canvas>
</template>

<style scoped>

</style>
