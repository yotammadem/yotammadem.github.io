---
name: tiny-phaser-games
description: >
  Build small self-contained 2D browser games directly with Phaser 4.
  Optimized for compact AI-generated games such as platformers, maze games,
  shooters, Breakout/Pong-like games, simple arcade games, collectathons,
  and small puzzle/action games.
---

# Tiny Phaser Games — Phaser 4 Skill

Use Phaser 4 directly. Do not invent a custom game DSL unless the user explicitly asks for one.

The goal is to generate **small, reliable, self-contained 2D games** that can run inside a single HTML page and can later be minified/compressed into a URL payload.

Prefer simple Phaser APIs and compact game logic over abstractions, frameworks, or architecture intended for large games.

---

# 1. Minimal game structure

For most tiny games, use one Scene:

```js
class G extends Phaser.Scene{
  preload(){}
  create(){}
  update(t,dt){}
}

new Phaser.Game({
  type:Phaser.AUTO,
  width:800,
  height:500,
  parent:"game",
  backgroundColor:"#111827",
  physics:{
    default:"arcade",
    arcade:{gravity:{y:0},debug:false}
  },
  scene:G
});
```

Use multiple Scenes only when there is a real need for separate menus, loading screens, overlays, or level transitions.

Scene lifecycle:

```text
init -> preload -> create -> update loop
```

Useful scene operations:

```js
this.scene.restart()
this.scene.start("OtherScene",{score})
this.scene.pause()
this.scene.resume()
```

---

# 2. Prefer Arcade Physics

For the target class of games, prefer **Arcade Physics** unless a mechanic genuinely needs Matter physics.

Dynamic object:

```js
const p=this.physics.add.sprite(100,100,"player");
p.setVelocity(100,0);
p.setBounce(1);
p.setCollideWorldBounds(true);
```

Static object:

```js
const wall=this.physics.add.staticImage(300,200,"wall");
```

Static group:

```js
const walls=this.physics.add.staticGroup();
```

Collision:

```js
this.physics.add.collider(player,walls);
```

Overlap without physical separation:

```js
this.physics.add.overlap(player,coins,(p,c)=>{
  c.destroy();
});
```

Move toward target:

```js
this.physics.moveToObject(enemy,player,100);
```

Important body state:

```js
body.velocity
body.blocked.down
body.touching
body.allowGravity
body.immovable
```

Platformer jump:

```js
if(jumpPressed&&player.body.blocked.down)
  player.setVelocityY(-420);
```

---

# 3. Input

Keyboard:

```js
this.cursors=this.input.keyboard.createCursorKeys();
this.keys=this.input.keyboard.addKeys("W,A,S,D,SPACE");
```

Continuous movement:

```js
if(this.cursors.left.isDown) player.setVelocityX(-160);
else if(this.cursors.right.isDown) player.setVelocityX(160);
else player.setVelocityX(0);
```

Single key press:

```js
if(Phaser.Input.Keyboard.JustDown(this.cursors.space)){
  // fire / jump / action
}
```

Pointer / touch:

```js
this.input.on("pointerdown",p=>{
  console.log(p.worldX,p.worldY);
});
```

Interactive object:

```js
sprite.setInteractive();
sprite.on("pointerdown",()=>{});
```

Prefer controls that work with both arrows and WASD when reasonable.

---

# 4. Visual objects

Use built-in shapes when assets are unnecessary.

Rectangle:

```js
this.add.rectangle(x,y,w,h,0xff0000);
```

Circle:

```js
this.add.circle(x,y,r,0xffff00);
```

Ellipse:

```js
this.add.ellipse(x,y,w,h,0xff66aa);
```

Triangle:

```js
this.add.triangle(x,y,0,s,s/2,0,s,s,0xffffff);
```

Star:

```js
this.add.star(x,y,5,8,16,0xffff00);
```

Text:

```js
this.add.text(10,10,"Score: 0",{
  fontFamily:"system-ui",
  fontSize:"20px",
  color:"#fff"
});
```

Useful visual methods:

```js
o.setPosition(x,y)
o.setScale(1.2)
o.setRotation(radians)
o.setAngle(degrees)
o.setAlpha(.5)
o.setDepth(10)
o.setFlipX(true)
o.setTint(0xff0000)
```

For HUD elements that should stay fixed while the camera moves:

```js
hud.setScrollFactor(0).setDepth(1000);
```

---

# 5. Sprites and assets

Load images in `preload()`:

```js
this.load.image("ship","assets/ship.png");
```

Spritesheet:

```js
this.load.spritesheet("hero","assets/hero.png",{
  frameWidth:32,
  frameHeight:48
});
```

Audio:

```js
this.load.audio("coin","assets/coin.mp3");
```

Create:

```js
this.add.image(x,y,"ship");
this.add.sprite(x,y,"hero");
```

If the game can be expressed clearly with shapes, prefer shapes for prototypes because they remove asset dependencies.

---

# 6. Animation

Spritesheet animation:

```js
this.anims.create({
  key:"walk",
  frames:this.anims.generateFrameNumbers("hero",{start:0,end:7}),
  frameRate:10,
  repeat:-1
});

player.play("walk");
```

For simple movement or visual effects, prefer tweens:

```js
this.tweens.add({
  targets:coin,
  y:"-=8",
  duration:400,
  yoyo:true,
  repeat:-1,
  ease:"Sine.easeInOut"
});
```

Useful tween properties:

```text
x y scale scaleX scaleY alpha angle rotation
```

Pulse:

```js
this.tweens.add({
  targets:enemy,
  scaleX:1.1,
  scaleY:.9,
  duration:180,
  yoyo:true,
  repeat:-1
});
```

---

# 7. Groups

Use groups for collections such as enemies, coins, bullets, and bricks.

Dynamic physics group:

```js
this.enemies=this.physics.add.group();
```

Static group:

```js
this.walls=this.physics.add.staticGroup();
```

Get children:

```js
this.enemies.getChildren()
```

Object pooling is useful for bullets:

```js
const bullet=bullets.get(x,y);
if(bullet){
  bullet.setActive(true).setVisible(true);
}
```

For small games, destroying objects is acceptable unless there are many rapidly-created projectiles.

---

# 8. Camera

Follow player:

```js
const cam=this.cameras.main;
cam.startFollow(player,true,.1,.1);
cam.setBounds(0,0,worldW,worldH);
```

World bounds:

```js
this.physics.world.setBounds(0,0,worldW,worldH);
```

Effects:

```js
cam.shake(120,.01);
cam.fadeOut(500);
cam.fadeIn(500);
cam.setZoom(1.5);
```

For platformers and scrolling games, usually set both physics world bounds and camera bounds.

---

# 9. Timers

One-shot:

```js
this.time.delayedCall(1000,()=>{
  // action
});
```

Loop:

```js
this.time.addEvent({
  delay:1000,
  loop:true,
  callback:()=>spawnEnemy()
});
```

Use timers for:

- spawning
- cooldowns
- temporary invulnerability
- enemy decisions
- level events
- delayed restart

---

# 10. Sound

Fire-and-forget effect:

```js
this.sound.play("coin");
```

Music:

```js
this.music=this.sound.add("music",{
  loop:true,
  volume:.4
});
this.music.play();
```

Remember that browsers may block audio until the first user interaction.

---

# 11. Compact maps

For maze/platform/grid games, a string map is often the most compact representation:

```js
const map=[
"################",
"#P....#....E...#",
"#.##..#........#",
"################"
];
```

Parse it:

```js
for(let y=0;y<map.length;y++)
for(let x=0;x<map[y].length;x++){
  const c=map[y][x];
  const px=x*TILE+TILE/2;
  const py=y*TILE+TILE/2;

  if(c=="#") addWall(px,py);
  if(c=="P") addPlayer(px,py);
  if(c=="E") addEnemy(px,py);
}
```

Prefer this over verbose arrays of coordinates when the level is naturally tile-based.

Use Phaser Tilemaps only when they provide real value. For tiny AI-generated games, a string grid is often simpler and smaller.

---

# 12. Useful game patterns

## Four-way arcade movement

```js
let vx=0,vy=0;

if(left)vx=-speed;
else if(right)vx=speed;
else if(up)vy=-speed;
else if(down)vy=speed;

player.setVelocity(vx,vy);
```

## Platformer

```js
player.setVelocityX(left?-speed:right?speed:0);

if(jump&&player.body.blocked.down)
  player.setVelocityY(-jumpSpeed);
```

## Chasing enemy

```js
this.physics.moveToObject(enemy,player,speed);
```

Do not use pure chasing for maze enemies unless that behavior is intended; it can cause enemies to pile up or stick against walls.

## Maze enemy decisions

A good maze-enemy pattern is:

1. Continue in the current corridor direction.
2. At a decision point, find legal directions.
3. Avoid immediate reversal unless necessary.
4. Usually choose the legal direction that moves closer to the player.
5. Sometimes choose another legal direction to create unpredictability.

Example:

```js
function chooseDir(enemy,player,legal){
  const reverse={L:"R",R:"L",U:"D",D:"U"}[enemy.dir];
  let a=legal.filter(d=>d!==reverse);
  if(!a.length)a=legal;

  if(Math.random()<.3)
    return Phaser.Utils.Array.GetRandom(a);

  return a.sort((x,y)=>
    distAfter(enemy,x,player)-distAfter(enemy,y,player)
  )[0];
}
```

## Separation

If enemies should not stack exactly on top of each other, add mild separation or make slightly different decisions rather than making all enemies continuously chase the exact player position.

## Respawning a round

Store initial positions:

```js
enemy.spawnX=enemy.x;
enemy.spawnY=enemy.y;
```

Reset:

```js
enemy.setPosition(enemy.spawnX,enemy.spawnY);
enemy.body.setVelocity(0,0);
```

If a player death starts a new round, reset all relevant enemies as well as the player.

---

# 13. Game state

For tiny games, simple Scene fields are preferred:

```js
this.score=0;
this.lives=3;
```

When state changes:

```js
this.score++;
scoreText.setText("Score: "+this.score);
```

Avoid introducing state-management libraries.

---

# 14. Win / lose conditions

Examples:

All collectibles gone:

```js
if(coins.countActive(true)===0) win();
```

All enemies destroyed:

```js
if(enemies.countActive(true)===0) win();
```

Lives exhausted:

```js
if(--this.lives<=0) gameOver();
```

Do not require a goal object if the actual game rule is simply completion of collectibles/enemies/etc.

---

# 15. Events and callbacks

Phaser systems are event emitters.

Examples:

```js
this.input.on("pointerdown",fn);
sprite.on("pointerdown",fn);
```

Custom events:

```js
this.events.emit("player-died");
this.events.on("player-died",()=>{});
```

For a tiny single-Scene game, direct functions are often simpler than creating a large event architecture.

---

# 16. Keep generated game code small

The final game may be encoded into a URL, so compactness matters.

However:

1. First write correct, understandable code.
2. Reuse helper functions.
3. Prefer short local names where obvious.
4. Prefer arrays / grids over repeated object declarations.
5. Avoid comments in the final payload unless useful for debugging.
6. Avoid unnecessary whitespace in the final payload.
7. Minify automatically if the host supports it.
8. Compress before Base64URL encoding when possible.

Do **not** sacrifice correctness for manual code-golf tricks.

Good:

```js
const addWall=(x,y)=>{
  const w=this.add.rectangle(x,y,T,T,0x17406d);
  this.physics.add.existing(w,true);
  walls.add(w);
};
```

Avoid generating dozens of repeated wall declarations.

---

# 17. Keep one HTML host, game code as payload

The host should ideally own:

- Phaser loading
- payload decoding
- optional decompression
- error display
- sandboxing
- start/restart controls

The payload should contain only game-specific JavaScript.

Conceptually:

```text
HTML host
  -> decode payload
  -> execute game script
  -> game script creates Phaser.Game
```

For local prototyping it is acceptable to run trusted generated code directly.

For a public site executing arbitrary shared game code, **do not execute payload JavaScript with access to the host page's origin**. Run untrusted game code in a sandboxed iframe or isolated origin and expose only the capabilities the game needs.

---

# 18. Reliability rules for AI-generated games

Before returning a game:

- Ensure every referenced asset key is loaded.
- Ensure every collision target has a physics body.
- Ensure static bodies are refreshed after resizing/scaling if necessary.
- Ensure the player cannot immediately respawn into an enemy.
- Ensure reset logic restores all actors relevant to a new round.
- Ensure a win condition is actually reachable.
- Ensure no required collectible is placed inside a wall.
- Ensure maze enemies have a legal escape from their spawn.
- Ensure a game can restart without duplicate event handlers or stale objects.
- Prefer straightforward code over clever abstractions.

When modifying an existing game, change the game code first. Only propose a host/runtime change when Phaser itself or browser isolation truly requires one.

---

# 19. Target game classes

This skill is optimized for:

- Pac-Man-like maze games
- Mario-like platformers
- Breakout
- Pong
- Space Invaders
- Asteroids-like games
- top-down shooters
- simple tower-defense variants
- collectathons
- Frogger-like games
- simple racing / avoidance games
- small action puzzles

If a requested mechanic does not fit the patterns above, still use Phaser directly and implement it with normal JavaScript rather than inventing a new DSL keyword.

---

# 20. Preferred authoring workflow

When asked to create a game:

1. Clarify only if the core game concept is genuinely ambiguous.
2. Choose the simplest Phaser mechanics that express the request.
3. Write a playable version.
4. Keep it self-contained.
5. Use shapes first unless assets materially improve the game.
6. Test game-state transitions mentally: start, play, collision, death, reset, win.
7. Return the game source / playable URL format expected by the host.
8. When the user asks for a change, modify JavaScript directly rather than extending a custom language.

The guiding principle is:

> Phaser is the game language. JavaScript is the scripting language. Keep the host small.
