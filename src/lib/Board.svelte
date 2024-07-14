<script>
	import { createEventDispatcher, onMount } from 'svelte';
	import ClickableTile from './ClickableTile.svelte';
	import { draw } from 'svelte/transition';
	const dispatch = createEventDispatcher();

	export let width = 3;
	export let height = 3;
	export let disabled = false;
	export let currentPlayer = 0;
	export function retry() {
		board = [[]];
		moves = [];
		disabled = false;
		canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height);
		initializeArray(board);
	}
	let board = [[]];
	let markArray = [[]];
	let size = Math.min(width, height);
	let moves = [];
	let boardElement, boardWidth, boardHeight;
	let canvas;
	let drawAudio;

	initializeArray(board);
	initializeArray(markArray);

	onMount(() => {
		boardWidth = boardElement.getBoundingClientRect().width;
		boardHeight = boardElement.getBoundingClientRect().height;
	});

	function initializeArray(array) {
		for (let i = 0; i < height; i++) {
			array[i] = [];
			for (let j = 0; j < width; j++) {
				array[i][j] = '';
			}
		}
	}

	function handleClick(event) {
		currentPlayer = (currentPlayer + 1) % 2;
		moves.push(event.detail);
		checkForMoves(moves);

		let result = checkWinner(board, event.detail.row, event.detail.column);
		if (result) {
			disabled = true;
			console.log(result);
			drawWinningLine(result.start, result.end);
			dispatch('win', { winner: result.winner, start: result.start, end: result.end });
		}
	}
	function drawWinningLine(start, end) {
		let ctx = canvas.getContext('2d');
		let startCoords = markArray[start[0]][start[1]].getBounds();
		let endCoords = markArray[end[0]][end[1]].getBounds();
		let canvasCoords = canvas.getBoundingClientRect();

		ctx.lineWidth = 12;
		ctx.lineCap = 'round';
		ctx.strokeStyle = 'white';

		const startX = startCoords.left + startCoords.width / 2 - canvasCoords.left;
		const startY = startCoords.top + startCoords.height / 2 - canvasCoords.top;
		const endX = endCoords.left + endCoords.width / 2 - canvasCoords.left;
		const endY = endCoords.top + endCoords.height / 2 - canvasCoords.top;

		drawAudio.playbackRate = 2;
		drawAudio.play();
		animateLine(startX, startY, endX, endY, 300);
	}
	function easeOutQuad(t) {
		return t * (2 - t);
	}
	function animateLine(startX, startY, endX, endY, duration) {
		let startTime = null;

		function drawFrame(time) {
			let ctx = canvas.getContext('2d');
			if (!startTime) startTime = time;
			const elapsed = time - startTime;
			const progress = Math.min(elapsed / duration, 1);
			const easedProgress = easeOutQuad(progress);

			const currentX = startX + (endX - startX) * easedProgress;
			const currentY = startY + (endY - startY) * easedProgress;

			ctx.clearRect(0, 0, canvas.width, canvas.height);

			ctx.lineWidth = 12;
			ctx.strokeStyle = 'white';
			ctx.beginPath();
			ctx.moveTo(startX, startY);
			ctx.lineTo(currentX, currentY);
			ctx.stroke();

			if (progress < 1) {
				requestAnimationFrame(drawFrame);
			}
		}

		requestAnimationFrame(drawFrame);
	}
	function checkForMoves(array) {
		console.log('size', array.length, size);

		let moveLimit = size * 2;
		if (array.length > moveLimit) {
			let removed = array.shift();
			removeValue(removed.row, removed.column);
		}
		if (array.length == moveLimit) {
			let aboutToDisappear = markArray[array[0].row][array[0].column];
			aboutToDisappear.aboutToDisappear();
		}
	}
	function removeValue(row, column) {
		board[row][column] = '';
	}
	function checkWinner(board, lastMoveRow, lastMoveCol) {
		const player = board[lastMoveRow][lastMoveCol];

		// Check the last move's row
		let rowWin = true;
		for (let j = 0; j < size; j++) {
			if (board[lastMoveRow][j] !== player) {
				rowWin = false;
				break;
			}
		}
		if (rowWin) {
			return { winner: player, start: [lastMoveRow, 0], end: [lastMoveRow, size - 1] };
		}

		// Check the last move's column
		let colWin = true;
		for (let i = 0; i < size; i++) {
			if (board[i][lastMoveCol] !== player) {
				colWin = false;
				break;
			}
		}
		if (colWin) {
			return { winner: player, start: [0, lastMoveCol], end: [size - 1, lastMoveCol] };
		}

		// Check the main diagonal if necessary
		if (lastMoveRow === lastMoveCol) {
			let diagWin = true;
			for (let i = 0; i < size; i++) {
				if (board[i][i] !== player) {
					diagWin = false;
					break;
				}
			}
			if (diagWin) {
				return { winner: player, start: [0, 0], end: [size - 1, size - 1] };
			}
		}

		// Check the anti-diagonal if necessary
		if (lastMoveRow + lastMoveCol === size - 1) {
			let antiDiagWin = true;
			for (let i = 0; i < size; i++) {
				if (board[i][size - 1 - i] !== player) {
					antiDiagWin = false;
					break;
				}
			}
			if (antiDiagWin) {
				return { winner: player, start: [0, size - 1], end: [size - 1, 0] };
			}
		}

		// No winner
		return null;
	}
</script>

{#key retry}
	<div bind:this={boardElement} class="board">
		<canvas height={boardHeight} width={boardWidth} bind:this={canvas}></canvas>
		<div class="board-proper">
			{#each { length: height } as row, i}
				<div class="row">
					{#each { length: width } as cell, j}
						<ClickableTile
							on:click={handleClick}
							{disabled}
							{currentPlayer}
							bind:value={board[i][j]}
							bind:this={markArray[i][j]}
							row={i}
							column={j}
						></ClickableTile>
					{/each}
				</div>
			{/each}
		</div>
	</div>
	<audio bind:this={drawAudio} src="../audio/draw.mp3" type="audio/mp3"> </audio>
{/key}

<style>
	.board {
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
	}
	.board-proper {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		border-radius: 1rem;
		overflow: hidden;
		gap: 0.5rem;
		padding: 0.5rem;
	}
	canvas {
		position: absolute;
		pointer-events: none;
	}
	.row {
		display: flex;
		flex-direction: row;
		gap: 0.5rem;
	}
</style>
