<template>
  <div ref="canvasContainer" class="canvas-container" @click="addBall">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script>
import Matter from "matter-js"; // Matter.jsライブラリのインポート

export default {
  name: "HomePage", // コンポーネント名
  data() {
    return {
      engine: null, // Matter.jsのエンジン
      world: null, // Matter.jsのワールド
      balls: [], // ボールの配列
    };
  },
  mounted() {
    // コンポーネントがマウントされた後に初期化
    this.initMatter(); // Matter.jsの初期化
    this.run(); // ゲームの実行
    window.addEventListener("resize", this.updateCanvasSize); // リサイズイベントの追加
    this.updateCanvasSize(); // 初期サイズを設定
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.updateCanvasSize); // イベントリスナーを削除
  },
  methods: {
    initMatter() {
      // Matter.jsのエンジンとワールドの作成
      this.engine = Matter.Engine.create();
      this.world = this.engine.world;

      // 地面を作成（静的オブジェクト）
      const ground = Matter.Bodies.rectangle(400, 580, 810, 60, {
        isStatic: true, // 静的なオブジェクトとして設定
      });
      const leftWall = Matter.Bodies.rectangle(0, 300, 60, 600, {
        isStatic: true, // 左壁
      });
      const rightWall = Matter.Bodies.rectangle(800, 300, 60, 600, {
        isStatic: true, // 右壁
      });

      // ワールドにオブジェクトを追加
      Matter.World.add(this.world, [ground, leftWall, rightWall]);

      // canvasサイズ設定
      const canvas = this.$refs.canvas; // canvasの参照を取得
      this.context = canvas.getContext("2d"); // 2Dコンテキストを取得
    },
    addBall(event) {
      // クリックした位置にボールを追加するメソッド
      if (this.balls.length >= 16) return; // 最大ボール数を制限

      const radius = this.getBallRadius(); // ボールの半径を動的に取得
      const x = event.clientX; // クリック位置のX座標
      const y = 0; // ボールを画面の上から落とす
      const color = this.getRandomColor(); // ランダムな色を取得

      // ボールを作成
      const ball = Matter.Bodies.circle(x, y, radius, {
        restitution: 0.8, // バウンス設定
        render: {
          fillStyle: color, // ボールの色
        },
      });
      Matter.World.add(this.world, ball); // ワールドにボールを追加
      this.balls.push(ball); // ボールを配列に追加
    },
    getBallRadius() {
      const canvasWidth = this.$refs.canvas.width; // canvasの幅を取得
      console.log(canvasWidth);
      const maxBalls = 16; // 最大ボール数
      const area = (canvasWidth * 0.8) / maxBalls; // 使用可能な面積を計算
      const radius = Math.sqrt(area / Math.PI); // 面積から半径を計算
      return radius; // 計算した半径を返す
    },
    getRandomColor() {
      // ランダムな色を生成するメソッド
      const letters = "0123456789ABCDEF"; // 16進数の色
      let color = "#"; // 色の初期値
      for (let i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)]; // ランダムに色を生成
      }
      return color; // 生成した色を返す
    },
    run() {
      // ゲームの実行ループ
      Matter.Engine.update(this.engine); // エンジンを更新
      this.draw(); // 画面を描画
      requestAnimationFrame(this.run); // 次のフレームを要求
    },
    draw() {
      // 画面の描画を行うメソッド
      const canvas = this.$refs.canvas; // canvasの参照を取得
      if (!canvas) return; // canvasが存在しない場合は何もしない

      // 画面をクリア
      this.context.clearRect(0, 0, canvas.width, canvas.height);

      // ボールを描画
      this.balls.forEach((ball) => {
        this.context.beginPath(); // 新しいパスを開始
        this.context.arc(
          ball.position.x, // ボールのX位置
          ball.position.y, // ボールのY位置
          ball.circleRadius, // ボールの半径
          0, // 開始角度
          Math.PI * 2 // 終了角度
        );
        this.context.fillStyle = ball.render.fillStyle; // ボールの色
        this.context.fill(); // 塗りつぶし
        this.context.closePath(); // パスを閉じる
      });
    },
    updateCanvasSize() {
      // canvasサイズを更新するメソッド
      const canvas = this.$refs.canvas; // canvasの参照を取得
      canvas.width = this.$refs.canvasContainer.clientWidth; // 親コンテナの幅を設定
      canvas.height = 600; // 高さを600pxに設定
    },
  },
};
</script>

<style scoped>
.canvas-container {
  position: relative;
  width: 100%; /* 親コンテナの幅に応じてサイズを変更 */
  height: 600px; /* 高さは固定 */
  overflow: hidden;
  border: 1px solid #000;
}
</style>
