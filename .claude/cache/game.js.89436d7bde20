const canvas = document.getElementById( 'gameCanvas' );
const ctx = canvas.getContext( '2d' );

const SOUNDS = {
  ballBounce: 'assets/sounds/ball-bounce.mp3',
  brickBreak: 'assets/sounds/break-sound.mp3',
};

function playSound( src ) {
  new Audio( src ).play();
}

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

let currentLevel = 1;

function currentLevelSpeed() {
  return BALL_SPEED * LEVELS[ currentLevel - 1 ].speedMultiplier;
}

const ball = {
  x: canvas.width / 2,
  y: paddle.y - BALL_RADIUS,
  radius: BALL_RADIUS,
  dx: BALL_SPEED,
  dy: -BALL_SPEED,
};

let lives = 3;

function resetBall() {
  const speed = currentLevelSpeed();
  ball.x = canvas.width / 2;
  ball.y = paddle.y - ball.radius;
  ball.dx = speed;
  ball.dy = -speed;
}

function updateBall() {
  ball.x += ball.dx;
  ball.y += ball.dy;

  if ( ball.x - ball.radius < 0 ) {
    ball.x = ball.radius;
    ball.dx *= -1;
    playSound( SOUNDS.ballBounce );
  } else if ( ball.x + ball.radius > canvas.width ) {
    ball.x = canvas.width - ball.radius;
    ball.dx *= -1;
    playSound( SOUNDS.ballBounce );
  }

  if ( ball.y - ball.radius < 0 ) {
    ball.y = ball.radius;
    ball.dy *= -1;
    playSound( SOUNDS.ballBounce );
  }

  checkPaddleCollision();
  checkBrickCollision();

  if ( ball.y - ball.radius > canvas.height ) {
    lives -= 1;
    if ( lives <= 0 ) lives = 3;
    resetBall();
  }
}

const MAX_BOUNCE_ANGLE = Math.PI / 3; // 60°

function checkPaddleCollision() {
  const withinX = ball.x + ball.radius > paddle.x && ball.x - ball.radius < paddle.x + paddle.width;
  const withinY = ball.y + ball.radius > paddle.y && ball.y - ball.radius < paddle.y + paddle.height;

  if ( !( withinX && withinY && ball.dy > 0 ) ) return;

  const paddleCenter = paddle.x + paddle.width / 2;
  const hitOffset = ( ball.x - paddleCenter ) / ( paddle.width / 2 );
  const clampedOffset = Math.max( -1, Math.min( 1, hitOffset ) );
  const angle = clampedOffset * MAX_BOUNCE_ANGLE;
  const speed = Math.hypot( ball.dx, ball.dy );

  ball.dx = speed * Math.sin( angle );
  ball.dy = -speed * Math.cos( angle );
  ball.y = paddle.y - ball.radius;
  playSound( SOUNDS.ballBounce );
}

const BRICK_COLS = 8;
const BRICK_ROWS = 5;
const BRICK_WIDTH = 80;
const BRICK_HEIGHT = 24;
const BRICK_PADDING = 6;
const BRICK_OFFSET_TOP = 40;
const BRICK_OFFSET_LEFT = ( canvas.width - ( BRICK_COLS * ( BRICK_WIDTH + BRICK_PADDING ) - BRICK_PADDING ) ) / 2;

const BRICK_COLORS = [ 'red', 'yellow', 'cyan', 'magenta', 'hotpink', 'green', 'gray' ];

const LEVEL_SPEED_STEP = 0.15;

function levelSpeedMultiplier( level ) {
  return 1 + LEVEL_SPEED_STEP * ( level - 1 );
}

const LEVELS = [
  {
    // Nivel 1: grid completo, todas las filas.
    grid: Array.from( { length: BRICK_ROWS }, ( _, row ) =>
      Array.from( { length: BRICK_COLS }, () => BRICK_COLORS[ row % BRICK_COLORS.length ] )
    ),
    speedMultiplier: levelSpeedMultiplier( 1 ),
  },
  {
    // Nivel 2: patrón en damero (huecos alternados).
    grid: Array.from( { length: BRICK_ROWS }, ( _, row ) =>
      Array.from( { length: BRICK_COLS }, ( _, col ) =>
        ( row + col ) % 2 === 0 ? BRICK_COLORS[ row % BRICK_COLORS.length ] : null
      )
    ),
    speedMultiplier: levelSpeedMultiplier( 2 ),
  },
  {
    // Nivel 3: columnas pares huecas.
    grid: Array.from( { length: BRICK_ROWS }, ( _, row ) =>
      Array.from( { length: BRICK_COLS }, ( _, col ) =>
        col % 2 === 0 ? null : BRICK_COLORS[ ( row + 2 ) % BRICK_COLORS.length ]
      )
    ),
    speedMultiplier: levelSpeedMultiplier( 3 ),
  },
  {
    // Nivel 4: forma de diamante (huecos en las esquinas de cada fila).
    grid: Array.from( { length: BRICK_ROWS }, ( _, row ) => {
      const distFromCenter = Math.abs( row - ( BRICK_ROWS - 1 ) / 2 );
      const margin = Math.round( distFromCenter );
      return Array.from( { length: BRICK_COLS }, ( _, col ) =>
        col < margin || col >= BRICK_COLS - margin ? null : BRICK_COLORS[ ( row + 4 ) % BRICK_COLORS.length ]
      );
    } ),
    speedMultiplier: levelSpeedMultiplier( 4 ),
  },
  {
    // Nivel 5: grid completo con colores invertidos (más difícil de leer, misma dimensión).
    grid: Array.from( { length: BRICK_ROWS }, ( _, row ) =>
      Array.from( { length: BRICK_COLS }, () => BRICK_COLORS[ ( BRICK_ROWS - 1 - row ) % BRICK_COLORS.length ] )
    ),
    speedMultiplier: levelSpeedMultiplier( 5 ),
  },
];

let bricks = [];

function createBricks() {
  bricks = [];
  const grid = LEVELS[ currentLevel - 1 ].grid;
  for ( let row = 0; row < grid.length; row++ ) {
    for ( let col = 0; col < grid[ row ].length; col++ ) {
      const color = grid[ row ][ col ];
      if ( color === null ) continue;
      bricks.push( {
        x: BRICK_OFFSET_LEFT + col * ( BRICK_WIDTH + BRICK_PADDING ),
        y: BRICK_OFFSET_TOP + row * ( BRICK_HEIGHT + BRICK_PADDING ),
        width: BRICK_WIDTH,
        height: BRICK_HEIGHT,
        color: color,
        alive: true,
      } );
    }
  }
}

createBricks();

const POINTS_PER_BRICK = 10;
let score = 0;

let explosions = [];

function checkBrickCollision() {
  let closestBrick = null;
  let closestDist = Infinity;

  for ( const brick of bricks ) {
    if ( !brick.alive ) continue;

    const withinX = ball.x + ball.radius > brick.x && ball.x - ball.radius < brick.x + brick.width;
    const withinY = ball.y + ball.radius > brick.y && ball.y - ball.radius < brick.y + brick.height;
    if ( !( withinX && withinY ) ) continue;

    const brickCenterX = brick.x + brick.width / 2;
    const brickCenterY = brick.y + brick.height / 2;
    const dist = Math.hypot( ball.x - brickCenterX, ball.y - brickCenterY );

    if ( dist < closestDist ) {
      closestDist = dist;
      closestBrick = brick;
    }
  }

  if ( !closestBrick ) return;

  closestBrick.alive = false;
  score += POINTS_PER_BRICK;
  playSound( SOUNDS.brickBreak );

  explosions.push( {
    x: closestBrick.x,
    y: closestBrick.y,
    width: closestBrick.width,
    height: closestBrick.height,
    color: closestBrick.color,
    startTime: performance.now(),
  } );

  const overlapLeft = ( ball.x + ball.radius ) - closestBrick.x;
  const overlapRight = ( closestBrick.x + closestBrick.width ) - ( ball.x - ball.radius );
  const overlapTop = ( ball.y + ball.radius ) - closestBrick.y;
  const overlapBottom = ( closestBrick.y + closestBrick.height ) - ( ball.y - ball.radius );

  const minOverlapX = Math.min( overlapLeft, overlapRight );
  const minOverlapY = Math.min( overlapTop, overlapBottom );

  if ( minOverlapX < minOverlapY ) {
    ball.dx *= -1;
  } else {
    ball.dy *= -1;
  }
}

function updateExplosions() {
  explosions = explosions.filter( ( explosion ) => performance.now() - explosion.startTime < EXPLOSION_DURATION );

  if ( explosions.length === 0 && bricks.every( ( brick ) => !brick.alive ) ) {
    currentLevel = currentLevel >= LEVELS.length ? 1 : currentLevel + 1;
    createBricks();
  }
}

function update() {
  if ( keys.left ) paddle.x -= PADDLE_SPEED;
  if ( keys.right ) paddle.x += PADDLE_SPEED;
  clampPaddle();
  updateBall();
  updateExplosions();
}

function render() {
  ctx.clearRect( 0, 0, canvas.width, canvas.height );
  drawSprite( ctx, 'paddle', paddle.x, paddle.y, paddle.width, paddle.height );
  drawSprite( ctx, 'ball', ball.x - ball.radius, ball.y - ball.radius, ball.radius * 2, ball.radius * 2 );
  bricks.forEach( ( brick ) => {
    if ( !brick.alive ) return;
    drawSprite( ctx, `block_${ brick.color }`, brick.x, brick.y, brick.width, brick.height );
  } );

  explosions.forEach( ( explosion ) => {
    const frameIndex = Math.min( 3, Math.floor( ( performance.now() - explosion.startTime ) / ( EXPLOSION_DURATION / 4 ) ) );
    drawFrame( ctx, EXPLOSION_FRAMES[ explosion.color ][ frameIndex ], explosion.x, explosion.y, explosion.width, explosion.height );
  } );

  ctx.fillStyle = '#fff';
  ctx.font = '16px sans-serif';
  ctx.textBaseline = 'top';
  ctx.fillText( `Puntaje: ${ score }`, 10, 10 );
  ctx.fillText( `Nivel: ${ currentLevel }`, 10, 32 );

  const lifeIconSize = 16;
  const lifeIconGap = 6;
  let lifeIconX = canvas.width - 10 - lifeIconSize;
  for ( let i = 0; i < lives; i++ ) {
    drawSprite( ctx, 'ball', lifeIconX, 10, lifeIconSize, lifeIconSize );
    lifeIconX -= lifeIconSize + lifeIconGap;
  }
}

function loop() {
  update();
  render();
  requestAnimationFrame( loop );
}

loadSpritesheet( () => {
  requestAnimationFrame( loop );
} );
