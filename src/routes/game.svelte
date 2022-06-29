<script lang="ts">
	import kaboom from 'kaboom';
	import { onMount } from 'svelte';
	import seedrandom from "seedrandom";

	onMount(async () => {
		let currentFrame = 0;
		let randomCycle = 0;

		const g = kaboom({
			width: 1280,
			height: 720,
			root: document.getElementById('game') as HTMLDivElement
		});

		// Preload sprites
		await Promise.all([
			loadSprite('bg_blue', '/background_blue.png'),
			loadSprite('bg_red', '/background_red.png'),
			loadSprite('bg_brown', '/background_brown.png'),
			loadSprite('car', '/car.png'),
			loadSprite('coin', '/coin.png')
		]);

		const menuScene = scene("menu", async () => {
			// Render background
			const bg = add([
				sprite('bg_blue', { tiled: true, width: g.width(), height: g.height() }),
				pos(0, 0),
				fixed()
			]);

			// Render title
			add([
				text("Welcome!", {size: 48, font: "sink"}),
				pos(center().x, center().y - 100),
				g.origin("center"),
				layer("ui"),
			]);

			add([
				text("Click anywhere\nto start a game", {size: 32, font: "sink"}),
				pos(center().x, center().y + 100),
				g.origin("center"),
				layer("ui"),
			]);

			onMousePress(() => {
				go("game");
			});
		});

		const gameOverScene = scene("gameOver", async (seed, score, moves) => {
			console.log(seed, score, moves);

			// Render background
			const bg = add([
				sprite('bg_red', { tiled: true, width: g.width(), height: g.height() }),
				pos(0, 0),
				fixed()
			]);

			// Render title
			add([
				text("Welcome!", {size: 48, font: "sink"}),
				pos(center().x, center().y - 100),
				g.origin("center"),
				layer("ui"),
			]);

			add([
				text("Click anywhere\nto play again", {size: 32, font: "sink"}),
				pos(center().x, center().y + 100),
				g.origin("center"),
				layer("ui"),
			]);

			onMousePress(() => {
				go("game");
			});

			// Show score
			const scoreText = add([
				pos(32, 32),
				text(score + " pts", {
					size: 32,
					width: 600,
					font: "sink", // there're 4 built-in fonts: "apl386", "apl386o", "sink", and "sinko"
				}),
			]);
		});

		const gameScene = scene("game", async () => {
			currentFrame = 0;
			randomCycle = 0;
			let moves: any = [];

			// Secure seeding logic
			const seed = "appwrite"; // TODO: Random seed
			const getRandomNumber = () => {
				const generator = seedrandom(seed + "_" + currentFrame + "_" + randomCycle);
				const randomNumber = generator();

				randomCycle++;
				return randomNumber;
			}

			// Render background
			const bg = add([
				sprite('bg_brown', { tiled: true, width: g.width(), height: g.height() }),
				pos(0, 0),
				fixed()
			]);

			// Render walls
			const walls = [
				add([ // Top
					pos(18, 10),
					'wall',
					rect(g.width() - 36, 8),
					color(g.Color.fromArray([148, 89, 36])),
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
					color(g.Color.fromArray([148, 89, 36])),
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
					color(g.Color.fromArray([148, 89, 36])),
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
					color(g.Color.fromArray([148, 89, 36])),
					area(),
					body(),
					fixed(),
					gravity(0),
				])
			];

			// Wall functionality
			onCollide("player", "wall", (player, wall) => {
				go("gameOver", seed, score, moves);
			});

			// Render player
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
			let score = 0;

			let isRotating = false;
			let angleAcceleration = 1;
			let angleForce = 0.05;

			const setRotating = (state: boolean) => {
				if(state !== isRotating) {
					moves.push(`${currentFrame};rotating;${state}`);
				}

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
			function spawnCoin() {
				const getDistance = (x1: number, y1: number, x2: number, y2: number) => {
					var a = x1 - x2;
					const b = y1 - y2;
					const c = Math.sqrt( a*a + b*b );

					return c;
				}
				const getPos = (): {x: number; y: number} => {
					const randX = getRandomNumber();
					const randY = getRandomNumber();

					const maxX = g.width() - 400;
					const maxY = g.height() - 400;

					const x = 200 + Math.round(maxX * randX);
					const y = 200 + Math.round(maxY * randY);

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
				score++;
				scoreText.text = score + " pts";
			};

			car.onCollide("coin", (coin) => {
				destroy(coin);
				onPickupCoin();
				setTimeout(() => {
					spawnCoin();
				}, 200);
			});

			// Show score
			const scoreText = add([
				pos(32, 32),
				text("0 pts", {
					size: 32,
					width: 600,
					font: "sink", // there're 4 built-in fonts: "apl386", "apl386o", "sink", and "sinko"
				}),
			]);

			// afterInit
			onUpdate(() => {
				currentFrame++;
			});
		});

		go("menu");
	});
</script>

<a href="/" class="back">
	<svg xmlns="http://www.w3.org/2000/svg" style="width: 20px; margin-right: 8px;" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
		<path stroke-linecap="round" stroke-linejoin="round" d="M15 19l-7-7 7-7" />
	  </svg>
	<span>Back to Home</span>
</a>

<div class="bg"><div class="bg-child" /></div>

<div id="game" />

<style global>
	.back {
		position: absolute;
		left: 16px;
		top: 16px;
		color: white;
		z-index: 40;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	#game {
		user-select: none;
		-moz-user-select: none;
		-webkit-user-select: none;
		-ms-user-select: none;

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
