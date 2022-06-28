<script lang="ts">
	import kaboom from 'kaboom';
	import { onMount } from 'svelte';

	onMount(async () => {
		const g = kaboom({
			width: 1280,
			height: 720,
			root: document.getElementById('game') as HTMLDivElement
		});

		const gameOverScene = scene("gameOver", () => {
			add([
				text("GAME OVER ", {size: 40, font: "sink"}),
				pos(center()),
				g.origin("center"),
				layer("ui"),
			]);
		});

		const gameScene = scene("game", async () => {
			// Render background
			const bgSprite = await loadSprite('bg', '/background_brown.png');
			const bg = add([
				sprite('bg', { tiled: true, width: g.width(), height: g.height() }),
				pos(0, 0),
				fixed()
			]);

			// Render walls
			const walls = [
				add([ // Top
					pos(18, 10),
					'wall',
					rect(g.width() - 36, 8),
					color(Color.fromArray([148, 89, 36])),
					area(),
					body(),
					gravity(0),
					fixed()
				]),
				add([ // Right
					pos(g.width() - 10, 10),
					g.origin('topright'),
					'wall',
					rect(8, g.height() - 20),
					color(Color.fromArray([148, 89, 36])),
					area(),
					body(),
					gravity(0),
					fixed()
				]),
				add([ // Bottom
					pos(18, g.height() - 10),
					'wall',
					g.origin('botleft'),
					rect(g.width() - 36, 8),
					color(Color.fromArray([148, 89, 36])),
					area(),
					body(),
					gravity(0),
					fixed()
				]),
				add([ // Left
					pos(10, 10),
					'wall',
					g.origin('topleft'),
					rect(8, g.height() - 20),
					color(Color.fromArray([148, 89, 36])),
					area(),
					body(),
					fixed(),
					gravity(0),
				])
			];

			// Wall functionality
			onCollide("player", "wall", (player, wall) => {
				go("gameOver");
			});

			// Render player
			const carSprite = await loadSprite('car', '/car.png');
			const car = add([
				sprite('car'),
				pos(center()),
				area(),
				gravity(0),
				body(),
				'player',
				{
					dir: UP,
					speed: 200
				},
				rotate(0),
				g.origin('center')
			]);

			// Player movement + controls
			let isRotating = false;
			let angleAcceleration = 1;
			let angleForce = 0.05;

			const setRotating = (state: boolean) => {
				isRotating = state;

				if (!state) {
					angleAcceleration = 1;
				}
			};

			onTouchStart(() => setRotating(true));
			onMouseDown(() => setRotating(true));

			onTouchEnd(() => setRotating(false));
			onMouseRelease(() => setRotating(false));

			car.onDraw(() => {
				car.move(g.Vec2.fromAngle(car.angle).scale(car.speed));

				if (isRotating) {
					car.angle += Math.min(angleAcceleration, 10);
					angleAcceleration += angleForce;
				}
			});

			// Coin spawning
			const coinSprite = await loadSprite('coin', '/coin.png');

			function spawnCoin() {
				const getDistance = (x1: number, y1: number, x2: number, y2: number) => {
					var a = x1 - x2;
					const b = y1 - y2;
					const c = Math.sqrt( a*a + b*b );

					return c;
				}
				const getPos = (iteration = 0): {x: number; y: number} => {
					const randX = Math.random();
					const randY = Math.random();

					const maxX = g.width() - 400;
					const maxY = g.height() - 400;

					const x = 200 + Math.round(maxX * randX);
					const y = 200 + Math.round(maxY * randY);

					if(iteration >= 100) {
						return {x,y};
					}

					if(getDistance(x,y, car.pos.x, car.pos.y) < 200) {
						return getPos(iteration + 1);
					}

					return {x,y};
				};


				const coinPos = getPos();

				add([
					sprite('coin'),
					pos(coinPos.x, coinPos.y),
					area(),
					gravity(0),
					body(),
					'coin',
					g.origin('center')
				]);
			}

			spawnCoin();

			// Coin functionality
			const onPickupCoin = () => {
				car.speed += 10;
				angleForce += 0.005;
			};

			car.onCollide("coin", (coin) => {
				destroy(coin);
				onPickupCoin();
				setTimeout(() => {
					spawnCoin();
				}, 500);
			});
		});

		go("game");
	});
</script>

<div class="bg"><div class="bg-child" /></div>

<div id="game" />

<style global>
	#game {
		z-index: 30;
		position: relative;
		display: block;
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.bg-child {
		z-index: 20;
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		background-image: url('/background_brown.png');
		background-position: center;
		background-repeat: repeat;
		filter: blur(6px);
		transform: scale(2);
		opacity: 0.5;
	}

	.bg {
		z-index: 10;
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		background-color: black;
	}
</style>
