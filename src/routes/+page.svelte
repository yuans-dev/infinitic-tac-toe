<script>
	import Board from '../lib/Board.svelte';
	import Mark from '../lib/Mark.svelte';

	let disabled = false;
	let board;
	let winner;
	let end = false;
	let currentPlayer = 0;
	let winsX = 0;
	let winsO = 0;
	function handleWin(event) {
		winner = event.detail.winner;
		if (winner == 'x') {
			winsX++;
		} else {
			winsO++;
		}
		end = true;
	}
	function handlePlayAgain() {
		end = false;
		board.retry();
	}
</script>

<div class="app-proper">
	{#if end}
		<span class="winner"><Mark value={winner}></Mark> wins</span>
	{:else}
		<span class="turn"><Mark value={currentPlayer == 0 ? 'x' : 'o'}></Mark>'s turn</span>
	{/if}
	<span class="score">{winsX} - {winsO}</span>
	<Board bind:this={board} bind:currentPlayer {disabled} on:win={handleWin}></Board>

	<button on:click={handlePlayAgain} class="play-again-button"
		>{end ? 'Play again' : 'Reset'}</button
	>
</div>

<style>
	@import '../styles/global.css';
	.app-proper {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		color: white;
		gap: 4vh;
	}
	.winner,
	.turn {
		font-size: 2rem;
	}
	.play-again-button {
		padding: 0.8rem 1.5rem;
		font-family: 'Poppins', sans-serif;
		border: none;
		border-radius: 1rem;
		background-color: rgb(88, 51, 95);
		font-size: 1rem;
		color: white;
		cursor: pointer;
		transition: all 0.3s ease;
	}
	.play-again-button:hover {
		background-color: #ffffff;
		color: black;
	}
</style>
