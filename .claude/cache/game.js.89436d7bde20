const canvas = document.getElementById( 'gameCanvas' );
const ctx = canvas.getContext( '2d' );

const PADDLE_WIDTH = 100;
const PADDLE_HEIGHT = 14;

const PADDLE_SPEED = 7;

const paddle = {
  x: ( canvas.width - PADDLE_WIDTH ) / 2,
  y: canvas.height - PADDLE_HEIGHT - 20,
  width: PADDLE_WIDTH,
  height: PADDLE_HEIGHT,
};

const keys = {
  left: false,
  right: false,
};

document.addEventListener( 'keydown', ( e ) => {
  if ( e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A' ) keys.left = true;
  if ( e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D' ) keys.right = true;
} );

document.addEventListener( 'keyup', ( e ) => {
  if ( e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A' ) keys.left = false;
  if ( e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D' ) keys.right = false;
} );

canvas.addEventListener( 'mousemove', ( e ) => {
  const rect = canvas.getBoundingClientRect();
  const mouseX = e.clientX - rect.left;
  paddle.x = mouseX - paddle.width / 2;
  clampPaddle();
} );

function clampPaddle() {
  if ( paddle.x < 0 ) paddle.x = 0;
  if ( paddle.x + paddle.width > canvas.width ) paddle.x = canvas.width - paddle.width;
}

const BALL_RADIUS = 8;
const BALL_SPEED = 5;

const ball = {
  x: canvas.width / 2,
  y: paddle.y - BALL_RADIUS,
  radius: BALL_RADIUS,
  dx: BALL_SPEED,
  dy: -BALL_SPEED,
};

function updateBall() {
  ball.x += ball.dx;
  ball.y += ball.dy;

  if ( ball.x - ball.radius < 0 ) {
    ball.x = ball.radius;
    ball.dx *= -1;
  } else if ( ball.x + ball.radius > canvas.width ) {
    ball.x = canvas.width - ball.radius;
    ball.dx *= -1;
  }

  if ( ball.y - ball.radius < 0 ) {
    ball.y = ball.radius;
    ball.dy *= -1;
  }
}

function update() {
  if ( keys.left ) paddle.x -= PADDLE_SPEED;
  if ( keys.right ) paddle.x += PADDLE_SPEED;
  clampPaddle();
  updateBall();
}

function render() {
  ctx.clearRect( 0, 0, canvas.width, canvas.height );
  drawSprite( ctx, 'paddle', paddle.x, paddle.y, paddle.width, paddle.height );
  drawSprite( ctx, 'ball', ball.x - ball.radius, ball.y - ball.radius, ball.radius * 2, ball.radius * 2 );
}

function loop() {
  update();
  render();
  requestAnimationFrame( loop );
}

loadSpritesheet( () => {
  requestAnimationFrame( loop );
} );
