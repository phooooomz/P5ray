const TILE = 32;
const MAP_W = 40;
const MAP_H = 30;
const SPEED = 140;

// สุ่มแบบกำหนดค่าเริ่มต้น ทำให้แผนที่เหมือนเดิมทุกครั้ง
function mulberry32(a) {
  return function () {
    a |= 0; a = (a + 0x6d2b79f5) | 0;
    let t = Math.imul(a ^ (a >>> 15), 1 | a);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}

class WorldScene extends Phaser.Scene {
  constructor() {
    super('World');
    this.joy = { active: false, id: null, ox: 0, oy: 0, dx: 0, dy: 0 };
  }

  makeTextures() {
    let g = this.make.graphics({ x: 0, y: 0, add: false });

    // หญ้า
    g.fillStyle(0x3f8f3f, 1).fillRect(0, 0, TILE, TILE);
    g.fillStyle(0x4aa04a, 1);
    g.fillRect(4, 6, 2, 2); g.fillRect(20, 12, 2, 2);
    g.fillRect(10, 24, 2, 2); g.fillRect(26, 26, 2, 2);
    g.generateTexture('grass', TILE, TILE);
    g.clear();

    // ต้นไม้
    g.fillStyle(0x3f8f3f, 1).fillRect(0, 0, TILE, TILE);
    g.fillStyle(0x6b4423, 1).fillRect(13, 18, 6, 12);
    g.fillStyle(0x1f5f2a, 1).fillCircle(16, 14, 12);
    g.fillStyle(0x2a7a38, 1).fillCircle(12, 11, 5);
    g.generateTexture('tree', TILE, TILE);
    g.clear();

    // ผู้เล่น
    g.fillStyle(0x000000, 0.35).fillEllipse(14, 24, 20, 8);
    g.fillStyle(0x2f6fdb, 1).fillCircle(14, 12, 10);
    g.lineStyle(2, 0xffffff, 1).strokeCircle(14, 12, 10);
    g.fillStyle(0xffffff, 1).fillCircle(10, 10, 2).fillCircle(18, 10, 2);
    g.generateTexture('player', 28, 28);
    g.destroy();
  }

  create() {
    this.makeTextures();
    const rng = mulberry32(12345);
    const worldW = MAP_W * TILE;
    const worldH = MAP_H * TILE;

    this.trees = this.physics.add.staticGroup();

    // วางพื้นหญ้าและต้นไม้
    for (let y = 0; y < MAP_H; y++) {
      for (let x = 0; x < MAP_W; x++) {
        const px = x * TILE + TILE / 2;
        const py = y * TILE + TILE / 2;
        const border = x === 0 || y === 0 || x === MAP_W - 1 || y === MAP_H - 1;
        const spawnArea = Math.abs(x - MAP_W / 2) < 3 && Math.abs(y - MAP_H / 2) < 3;
        if (border || (!spawnArea && rng() < 0.08)) {
          const t = this.trees.create(px, py, 'tree');
          t.body.setSize(TILE * 0.6, TILE * 0.6).updateFromGameObject();
        } else {
          this.add.image(px, py, 'grass');
        }
      }
    }

    // ผู้เล่น
    this.player = this.physics.add.sprite(worldW / 2, worldH / 2, 'player');
    this.player.body.setSize(16, 14).setOffset(6, 14);
    this.player.setCollideWorldBounds(true);
    this.physics.world.setBounds(0, 0, worldW, worldH);
    this.physics.add.collider(this.player, this.trees);

    this.nameLabel = this.add.text(0, 0, 'Hero', {
      fontFamily: 'sans-serif', fontSize: '12px', color: '#ffffff',
      stroke: '#000000', strokeThickness: 3
    }).setOrigin(0.5, 1);

    // กล้อง
    this.cameras.main.setBounds(0, 0, worldW, worldH);
    this.cameras.main.startFollow(this.player, true, 0.15, 0.15);
    this.cameras.main.setZoom(1.5);

    // คีย์บอร์ด
    this.cursors = this.input.keyboard.createCursorKeys();
    this.wasd = this.input.keyboard.addKeys('W,A,S,D');

    // HUD
    this.hud = this.add.text(8, 8, '', {
      fontFamily: 'monospace', fontSize: '12px', color: '#ffffff',
      backgroundColor: '#00000088', padding: { x: 6, y: 4 }
    }).setScrollFactor(0).setDepth(1000);

    this.setupJoystick();
  }

  setupJoystick() {
    this.input.addPointer(2);
    this.joyBase = this.add.circle(0, 0, 50, 0xffffff, 0.15)
      .setScrollFactor(0).setDepth(1000).setVisible(false);
    this.joyThumb = this.add.circle(0, 0, 22, 0xffffff, 0.4)
      .setScrollFactor(0).setDepth(1001).setVisible(false);

    this.input.on('pointerdown', (p) => {
      if (this.joy.active) return;
      if (p.x > this.scale.width / 2) return; // จอยอยู่ครึ่งซ้ายของจอ
      this.joy.active = true;
      this.joy.id = p.id;
      this.joy.ox = p.x; this.joy.oy = p.y;
      this.joyBase.setPosition(p.x, p.y).setVisible(true);
      this.joyThumb.setPosition(p.x, p.y).setVisible(true);
    });

    this.input.on('pointermove', (p) => {
      if (!this.joy.active || p.id !== this.joy.id) return;
      let dx = p.x - this.joy.ox;
      let dy = p.y - this.joy.oy;
      const len = Math.hypot(dx, dy);
      const max = 50;
      if (len > max) { dx = (dx / len) * max; dy = (dy / len) * max; }
      this.joy.dx = dx / max;
      this.joy.dy = dy / max;
      this.joyThumb.setPosition(this.joy.ox + dx, this.joy.oy + dy);
    });

    const release = (p) => {
      if (!this.joy.active || p.id !== this.joy.id) return;
      this.joy.active = false; this.joy.id = null;
      this.joy.dx = 0; this.joy.dy = 0;
      this.joyBase.setVisible(false);
      this.joyThumb.setVisible(false);
    };
    this.input.on('pointerup', release);
    this.input.on('pointerupoutside', release);
  }

  update() {
    let vx = 0, vy = 0;
    if (this.cursors.left.isDown || this.wasd.A.isDown) vx -= 1;
    if (this.cursors.right.isDown || this.wasd.D.isDown) vx += 1;
    if (this.cursors.up.isDown || this.wasd.W.isDown) vy -= 1;
    if (this.cursors.down.isDown || this.wasd.S.isDown) vy += 1;

    if (this.joy.active && (Math.abs(this.joy.dx) > 0.15 || Math.abs(this.joy.dy) > 0.15)) {
      vx = this.joy.dx;
      vy = this.joy.dy;
    }

    // ทำให้ความเร็วทแยงไม่เร็วกว่าเดินตรง
    const len = Math.hypot(vx, vy);
    if (len > 1) { vx /= len; vy /= len; }

    this.player.setVelocity(vx * SPEED, vy * SPEED);
    this.nameLabel.setPosition(this.player.x, this.player.y - 18);
    this.hud.setText(
      'Eternal Realm Online\n' +
      'X:' + Math.round(this.player.x) + ' Y:' + Math.round(this.player.y) +
      '  FPS:' + Math.round(this.game.loop.actualFps)
    );
  }
}

new Phaser.Game({
  type: Phaser.AUTO,
  parent: 'game',
  backgroundColor: '#0b0f14',
  pixelArt: true,
  scale: { mode: Phaser.Scale.RESIZE, width: '100%', height: '100%' },
  physics: { default: 'arcade', arcade: { debug: false } },
  scene: [WorldScene]
});
