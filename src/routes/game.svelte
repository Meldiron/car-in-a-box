<script lang="ts">
	import kaboom from 'kaboom';
	import { onMount } from 'svelte';

	onMount(async () => {
		const g = kaboom({
			width: 1280,
			height: 720,
			root: document.getElementById('game') as HTMLDivElement
		});

		// TODO: Put into "game" scene
		// TODO: Make game over

		// Render background
		const bgSprite = await loadSprite('bg', '/background_brown.png');
		const bg = add([
			sprite('bg', { tiled: true, width: g.width(), height: g.height() }),
			pos(0, 0),
			fixed()
		]);

		// Render walls
		const walls = [
			add([
				pos(10, 10),
				'wall',
				rect(g.width() - 28, 8),
				color(Color.fromArray([148, 89, 36])),
				area(),
				body(),
				fixed()
			]),
			add([
				pos(g.width() - 10, 10),
				g.origin('topright'),
				'wall',
				rect(8, g.height() - 20),
				color(Color.fromArray([148, 89, 36])),
				area(),
				body(),
				fixed()
			]),
			add([
				pos(10, g.height() - 10),
				'wall',
				g.origin('botleft'),
				rect(g.width() - 28, 8),
				color(Color.fromArray([148, 89, 36])),
				area(),
				body(),
				fixed()
			]),
			add([
				pos(10, 10),
				'wall',
				g.origin('topleft'),
				rect(8, g.height() - 20),
				color(Color.fromArray([148, 89, 36])),
				area(),
				body(),
				fixed()
			])
		];

		// Render player
		const carSprite = await loadSprite('car', '/car.png');
		const car = add([
			sprite('car'),
			pos(center()),
			area({
				width: 61,
				shape: 'circle',
				height: 36
			}),
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
				onPickupCoin();
			}
		};

		onTouchStart(() => setRotating(true));
		onMouseDown(() => setRotating(true));

		onTouchEnd(() => setRotating(false));
		onMouseRelease(() => setRotating(false));

		car.onDraw(() => {
			console.log(g.debug.fps());
			car.move(g.Vec2.fromAngle(car.angle).scale(car.speed));

			if (isRotating) {
				car.angle += Math.min(angleAcceleration, 10);
				angleAcceleration += angleForce;
			}
		});

		// Coin spawning

		// Coin functionality
		const onPickupCoin = () => {
			car.speed += 20;
			angleForce += 0.005;
		};
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
