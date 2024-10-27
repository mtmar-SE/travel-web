<template>
  <div ref="canvasContainer" class="canvas-container">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
import Matter from "matter-js";
import enfj from "../assets/member/enfj-protagonist.svg";
import infj from "../assets/member/infj-advocate.svg";
import infp from "../assets/member/infp-mediator.svg";
import isfj from "../assets/member/isfj-defender.svg";
import motoki from "../assets/member/motoki.png";
import konno from "../assets/member/konno.png";
import enzo from "../assets/member/enzo.png";
import taiga from "../assets/member/taiga.png";

const images = [enfj, infj, infp, isfj, motoki, konno, enzo, taiga];

export default {
  name: "HomePage",
  data() {
    return {
      engine: null,
      world: null,
      balls: [],
      ground: null,
      leftWall: null,
      rightWall: null,
      ballInterval: null,
      firstDropBall: 4,
      maxBalls: 12, // 最大個数を12に変更
      usedImages: [],
      imageElements: [],
    };
  },
  mounted() {
    this.loadImages();
    this.initMatter();
    this.run();
    window.addEventListener("resize", this.resizeCanvas);
    this.resizeCanvas();
    this.createInitialBalls();
    this.startBallRain();
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.resizeCanvas);
    clearInterval(this.ballInterval);
  },
  methods: {
    loadImages() {
      this.imageElements = images.map((src) => {
        const img = new Image();
        img.src = src;
        return img;
      });
    },
    initMatter() {
      this.engine = Matter.Engine.create();
      this.world = this.engine.world;
      this.createStaticBodies();
      this.resizeCanvas();
      this.context = this.$refs.canvas.getContext("2d");
    },
    resizeCanvas() {
      const canvas = this.$refs.canvas;
      const container = this.$refs.canvasContainer;
      canvas.width = container.clientWidth;
      canvas.height = container.clientHeight;

      // 画面幅が600px以下の場合、スマホ表示としてボールのサイズを小さくする
      this.isMobile = canvas.width <= 600;
      this.createStaticBodies();
    },
    createStaticBodies() {
      if (this.ground) Matter.World.remove(this.world, this.ground);
      if (this.leftWall) Matter.World.remove(this.world, this.leftWall);
      if (this.rightWall) Matter.World.remove(this.world, this.rightWall);

      const canvasWidth = this.$refs.canvas.width;
      const canvasHeight = this.$refs.canvas.height;

      this.ground = Matter.Bodies.rectangle(
        canvasWidth / 2,
        canvasHeight,
        canvasWidth,
        30,
        { isStatic: true }
      );

      this.leftWall = Matter.Bodies.rectangle(
        0,
        canvasHeight / 2,
        1,
        canvasHeight,
        { isStatic: true }
      );

      this.rightWall = Matter.Bodies.rectangle(
        canvasWidth,
        canvasHeight / 2,
        1,
        canvasHeight,
        { isStatic: true }
      );

      Matter.World.add(this.world, [
        this.ground,
        this.leftWall,
        this.rightWall,
      ]);
    },
    createInitialBalls() {
      const initialBalls = Math.floor(Math.random() * 2) + 2; // 2〜3個のボールを落とす
      for (let i = 0; i < initialBalls; i++) {
        this.addBall();
      }
    },

    startBallRain() {
      this.ballInterval = setInterval(() => {
        this.dropBalls();
      }, 800);
    },
    dropBalls() {
      // 現在のボール数が最大数に達している場合は終了
      if (this.balls.length >= this.maxBalls) {
        clearInterval(this.ballInterval);
        return;
      }

      // 2〜4個のボールをランダムに追加
      const ballsToAdd = Math.floor(Math.random() * 3) + 2; // 2から4までのランダム数
      for (let i = 0; i < ballsToAdd; i++) {
        if (this.balls.length < this.maxBalls) {
          this.addBall();
        }
      }
    },
    getRandomColor() {
      const letters = "0123456789ABCDEF";
      let color = "#";
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
      }
      return color;
    },
    addBall() {
      if (this.balls.length >= this.maxBalls) return;

      // スマホの場合は半径を小さくする
      const baseRadius = this.isMobile ? 20 : 50;
      let radius = Math.random() * 30 + baseRadius;
      const x = Math.random() * (this.$refs.canvas.width - radius * 2) + radius;
      const y = -radius;

      const availableImages = images.filter(
        (img) => !this.usedImages.includes(img)
      );

      let imgSrc;
      let color = this.getRandomColor();

      if (availableImages.length > 0) {
        imgSrc =
          availableImages[Math.floor(Math.random() * availableImages.length)];
        this.usedImages.push(imgSrc);
        radius += this.isMobile ? 30 : 50; // スマホの場合は半径を控えめに追加

        const imageName = imgSrc.split("/").pop().split("-")[0];
        console.log(imageName);
        if (imageName === "isfj") {
          color = "#d9eaf0";
        } else if (["enfj", "infj", "infp"].includes(imageName)) {
          color = "#d6ece3";
        }
      } else {
        color = this.getRandomColor();
      }

      const ball = Matter.Bodies.circle(x, y, radius, {
        restitution: 0.7,
        render: {
          fillStyle: color,
          imgSrc: imgSrc,
          strokeStyle: "#000",
          lineWidth: 2,
        },
      });

      Matter.World.add(this.world, ball);
      this.balls.push(ball);
    },
    run() {
      Matter.Engine.update(this.engine);
      this.draw();
      requestAnimationFrame(this.run);
    },
    draw() {
      const canvas = this.$refs.canvas;
      if (!canvas) return;

      this.context.clearRect(0, 0, canvas.width, canvas.height);

      this.balls.forEach((ball) => {
        // 背景色の円を描画
        this.context.fillStyle = ball.render.fillStyle; // 背景色
        this.context.beginPath();
        this.context.arc(
          ball.position.x,
          ball.position.y,
          ball.circleRadius,
          0,
          2 * Math.PI
        );
        this.context.fill();

        // SVG画像を描画
        const imgElement = this.imageElements.find((img) =>
          img.src.includes(ball.render.imgSrc)
        );
        if (imgElement) {
          // 回転を考慮した描画
          this.context.save();
          this.context.translate(ball.position.x, ball.position.y);
          this.context.rotate(ball.angle); // ボールの角度で回転

          // 80%のサイズで描画するためのサイズ計算
          const imageSize = ball.circleRadius * 2 * 0.8;
          this.context.drawImage(
            imgElement,
            -imageSize / 2, // 中心に描画するためのオフセット
            -imageSize / 2, // 中心に描画するためのオフセット
            imageSize,
            imageSize
          );
          this.context.restore();
        } else {
          // 画像がない場合の色
          this.context.fillStyle = ball.render.fillStyle;
          this.context.beginPath();
          this.context.arc(
            ball.position.x,
            ball.position.y,
            ball.circleRadius,
            0,
            2 * Math.PI
          );
          this.context.fill();
        }
      });

      const padding = canvas.width * 0.05;
      this.context.beginPath();
      this.context.moveTo(padding, this.ground.position.y - 14);
      this.context.lineTo(canvas.width - padding, this.ground.position.y - 14);
      this.context.strokeStyle = "#000";
      this.context.lineWidth = 1;
      this.context.stroke();
      this.context.closePath();
    },
  },
};
</script>

<style scoped>
.canvas-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
