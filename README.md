<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<title>Mini Juego Pelea Anime 2D</title>
<style>
  body {
    margin: 0; background: #111; color: #eee; font-family: Arial, sans-serif;
    overflow: hidden;
  }
  canvas {
    display: block; margin: auto; background: #222; border: 2px solid #555;
  }
  #info {
    text-align: center; margin: 10px; font-size: 14px;
  }
</style>
</head>
<body>
  <h1 style="text-align:center;">Mini Juego Pelea Anime 2D (WASD + E,R,T para atacar)</h1>
  <canvas id="game" width="800" height="400"></canvas>
  <div id="info">
    Controles Jugador 1:<br>
    Mover: A (izquierda), D (derecha), W (saltar), S (agacharse)<br>
    Atacar: E (puñetazo), R (patada), T (ataque especial)
  </div>

<script>
(() => {
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  // Jugador
  class Player {
    constructor(x, color) {
      this.x = x;
      this.y = 300;
      this.vy = 0;
      this.width = 50;
      this.height = 100;
      this.color = color;
      this.speed = 5;
      this.onGround = true;
      this.isCrouch = false;
      this.health = 100;
      this.attackCooldown = 0;
      this.currentAttack = null;
      this.direction = 1; // 1 derecha, -1 izquierda
    }

    draw() {
      ctx.fillStyle = this.color;
      if(this.isCrouch) {
        ctx.fillRect(this.x, this.y + 50, this.width, 50);
      } else {
        ctx.fillRect(this.x, this.y, this.width, this.height);
      }
      // Health bar
      ctx.fillStyle = 'red';
      ctx.fillRect(this.x, this.y - 20, this.health / 2, 10);
      ctx.strokeStyle = 'white';
      ctx.strokeRect(this.x, this.y - 20, 50, 10);
    }

    update() {
      // Gravity
      if(!this.onGround) {
        this.vy += 1.2;
        this.y += this.vy;
        if(this.y > 300) {
          this.y = 300;
          this.vy = 0;
          this.onGround = true;
          this.isCrouch = false;
        }
      }

      // Attack cooldown decrease
      if(this.attackCooldown > 0) this.attackCooldown--;

      // Attack hitbox check
      if(this.currentAttack) {
        if(this.currentAttack.frame > 10 && this.currentAttack.frame < 20) {
          // during attack active frames
          // attack hitbox
          const attackX = this.x + (this.direction * this.width);
          const attackY = this.y + (this.isCrouch ? 80 : 40);
          const attackW = 30;
          const attackH = 30;

          // Simple visualization of attack hitbox (comment if want clean)
          ctx.fillStyle = 'rgba(255,255,0,0.3)';
          ctx.fillRect(attackX, attackY, attackW * this.direction, attackH);

          // Check collision with other player
          let target = this === player1 ? player2 : player1;
          if(target.x < attackX + attackW && target.x + target.width > attackX &&
             target.y < attackY + attackH && target.y + target.height > attackY) {
            target.health -= 0.5; // Damage
            if(target.health < 0) target.health = 0;
          }
        }
        this.currentAttack.frame++;
        if(this.currentAttack.frame > 25) this.currentAttack = null;
      }
    }

    move(dir) {
      this.direction = dir;
      let newX = this.x + dir * this.speed;
      if(newX >= 0 && newX + this.width <= canvas.width) {
        this.x = newX;
      }
    }

    jump() {
      if(this.onGround) {
        this.vy = -20;
        this.onGround = false;
        this.isCrouch = false;
      }
    }

    crouch(flag) {
      if(this.onGround) this.isCrouch = flag;
    }

    attack(type) {
      if(this.attackCooldown === 0) {
        this.currentAttack = { type, frame: 0 };
        this.attackCooldown = 40;
      }
    }
  }

  // Crear jugadores
  const player1 = new Player(100, 'cyan');
  const player2 = new Player(650, 'orange');

  // Controles Jugador 1
  const keys = {};
  window.addEventListener('keydown', e => {
    keys[e.key.toLowerCase()] = true;

    if(e.key.toLowerCase() === 'w') player1.jump();
    if(e.key.toLowerCase() === 's') player1.crouch(true);
    if(e.key.toLowerCase() === 'e') player1.attack('puñetazo');
    if(e.key.toLowerCase() === 'r') player1.attack('patada');
    if(e.key.toLowerCase() === 't') player1.attack('especial');
  });
  window.addEventListener('keyup', e => {
    keys[e.key.toLowerCase()] = false;

    if(e.key.toLowerCase() === 's') player1.crouch(false);
  });

  function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Movimiento horizontal
    if(keys['a']) player1.move(-1);
    else if(keys['d']) player1.move(1);

    player1.update();
    player2.update(); // Por ahora player2 está quieto, pero puedes ampliar para 2 jugadores

    // Dibujar piso
    ctx.fillStyle = '#444';
    ctx.fillRect(0, 400, canvas.width, 10);

    player1.draw();
    player2.draw();

    // Mostrar texto salud
    ctx.fillStyle = 'white';
    ctx.font = '20px Arial';
    ctx.fillText('Jugador 1 Salud: ' + Math.round(player1.health), 20, 30);
    ctx.fillText('Jugador 2 Salud: ' + Math.round(player2.health), 600, 30);

    requestAnimationFrame(gameLoop);
  }
  requestAnimationFrame(gameLoop);
})();
</script>
</body>
</html>
