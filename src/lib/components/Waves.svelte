<script>
	import { onMount } from 'svelte';

	let canvas;
	let ctx;
	let bounding;

	const mouse = {
		x: -10,
		y: 0,
		lx: 0,
		ly: 0,
		sx: 0,
		sy: 0,
		v: 0,
		vs: 0,
		a: 0,
		set: false,
	};

	let lines = [];

	class Grad {
		constructor(x, y, z) {
			this.x = x;
			this.y = y;
			this.z = z;
		}

		dot2(x, y) {
			return this.x * x + this.y * y;
		}
	}

	class Noise {
		constructor(seed = 0) {
			this.grad3 = [
				new Grad(1, 1, 0),
				new Grad(-1, 1, 0),
				new Grad(1, -1, 0),
				new Grad(-1, -1, 0),
				new Grad(1, 0, 1),
				new Grad(-1, 0, 1),
				new Grad(1, 0, -1),
				new Grad(-1, 0, -1),
				new Grad(0, 1, 1),
				new Grad(0, -1, 1),
				new Grad(0, 1, -1),
				new Grad(0, -1, -1),
			];

			this.p = Array.from({ length: 256 }, (_, i) => i);
			this.perm = new Array(512);
			this.gradP = new Array(512);

			this.seed(seed);
		}

		seed(seed) {
			if (seed > 0 && seed < 1) seed *= 65536;
			seed = Math.floor(seed);
			if (seed < 256) seed |= seed << 8;

			for (let i = 0; i < 256; i++) {
				const v = i & 1 ? this.p[i] ^ (seed & 255) : this.p[i] ^ ((seed >> 8) & 255);
				this.perm[i] = this.perm[i + 256] = v;
				this.gradP[i] = this.gradP[i + 256] = this.grad3[v % 12];
			}
		}

		fade(t) {
			return t * t * t * (t * (t * 6 - 15) + 10);
		}

		lerp(a, b, t) {
			return (1 - t) * a + t * b;
		}

		perlin2(x, y) {
			let X = Math.floor(x);
			let Y = Math.floor(y);

			x -= X;
			y -= Y;

			X = X & 255;
			Y = Y & 255;

			const n00 = this.gradP[X + this.perm[Y]].dot2(x, y);
			const n01 = this.gradP[X + this.perm[Y + 1]].dot2(x, y - 1);
			const n10 = this.gradP[X + 1 + this.perm[Y]].dot2(x - 1, y);
			const n11 = this.gradP[X + 1 + this.perm[Y + 1]].dot2(x - 1, y - 1);

			const u = this.fade(x);
			const v = this.fade(y);

			return this.lerp(this.lerp(n00, n10, u), this.lerp(n01, n11, u), v);
		}
	}

	let noise = new Noise(Math.random());

	function setSize() {
		bounding = canvas.getBoundingClientRect();
		canvas.width = window.innerWidth;
		canvas.height = window.innerHeight;
	}

	function setLines() {
		const { width, height } = canvas;

		lines = [];
		const xGap = 10;
		const yGap = 32;

		const totalLines = Math.ceil(width / xGap);
		const totalPoints = Math.ceil(height / yGap);

		for (let i = 0; i <= totalLines; i++) {
			const points = [];
			for (let j = 0; j <= totalPoints; j++) {
				points.push({
					x: i * xGap,
					y: j * yGap,
					wave: { x: 0, y: 0 },
					cursor: { x: 0, y: 0, vx: 0, vy: 0 },
				});
			}
			lines.push(points);
		}
	}

	function movePoints(time) {
		lines.forEach((points) => {
			points.forEach((p) => {
				const move =
					noise.perlin2(
						(p.x + time * 0.0125) * 0.002,
						(p.y + time * 0.005) * 0.0015
					) * 12;
				p.wave.x = Math.cos(move) * 32;
				p.wave.y = Math.sin(move) * 16;

				const dx = p.x - mouse.sx;
				const dy = p.y - mouse.sy;
				const d = Math.hypot(dx, dy);
				const l = Math.max(175, mouse.vs);

				if (d < l) {
					const s = 1 - d / l;
					const f = Math.cos(d * 0.001) * s;

					p.cursor.vx += Math.cos(mouse.a) * f * l * mouse.vs * 0.00065;
					p.cursor.vy += Math.sin(mouse.a) * f * l * mouse.vs * 0.00065;
				}

				p.cursor.vx += (0 - p.cursor.x) * 0.005;
				p.cursor.vy += (0 - p.cursor.y) * 0.005;

				p.cursor.vx *= 0.925;
				p.cursor.vy *= 0.925;

				p.cursor.x += p.cursor.vx * 2;
				p.cursor.y += p.cursor.vy * 2;

				p.cursor.x = Math.min(100, Math.max(-100, p.cursor.x));
				p.cursor.y = Math.min(100, Math.max(-100, p.cursor.y));
			});
		});
	}

	function drawLines() {
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		ctx.beginPath();
		ctx.strokeStyle = '#1f1f1f';

		lines.forEach((points) => {
			ctx.moveTo(points[0].x, points[0].y);
			points.forEach((p) => {
				ctx.lineTo(p.x + p.wave.x + p.cursor.x, p.y + p.wave.y + p.cursor.y);
			});
		});

		ctx.stroke();
	}

	function tick(time) {
		mouse.sx += (mouse.x - mouse.sx) * 0.1;
		mouse.sy += (mouse.y - mouse.sy) * 0.1;

		const dx = mouse.x - mouse.lx;
		const dy = mouse.y - mouse.ly;
		const d = Math.hypot(dx, dy);

		mouse.v = d;
		mouse.vs += (d - mouse.vs) * 0.1;
		mouse.vs = Math.min(100, mouse.vs);

		mouse.lx = mouse.x;
		mouse.ly = mouse.y;

		mouse.a = Math.atan2(dy, dx);

		movePoints(time);
		drawLines();
		requestAnimationFrame(tick);
	}

	function onMouseMove(e) {
		mouse.x = e.clientX;
		mouse.y = e.clientY;

		if (!mouse.set) {
			mouse.sx = mouse.x;
			mouse.sy = mouse.y;
			mouse.lx = mouse.x;
			mouse.ly = mouse.y;
			mouse.set = true;
		}
	}

	onMount(() => {
		ctx = canvas.getContext('2d');
		setSize();
		setLines();

		window.addEventListener('resize', setSize);
		window.addEventListener('mousemove', onMouseMove);

		requestAnimationFrame(tick);
	});
</script>

<canvas bind:this={canvas}></canvas>

<style>
    canvas {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        z-index: -1;
        pointer-events: none;
        background: #000000;
    }
</style>
