<svelte:options accessors />

<script>
	import { createEventDispatcher } from 'svelte';
	import '@fortawesome/fontawesome-free/css/all.min.css';
	import Mark from './Mark.svelte';
	import Page from '../routes/+page.svelte';
	const dispatch = createEventDispatcher();
	export let currentPlayer;
	export let size = 7;
	export let value;
	export let row;
	export let column;
	export let disabled;
	let preview = '';
	let focused = false;
	let opacity = 1;
	export function aboutToDisappear() {
		opacity = 0.2;
		console.log(row, column);
	}
	let element;
	export function getBounds() {
		return element.getBoundingClientRect();
	}
	let clickAudio, errorAudio;
	function handleClick() {
		if (value) {
			errorAudio.play();
			return;
		}
		if (currentPlayer == 0) {
			value = 'x';
		} else {
			value = 'o';
		}
		clickAudio.play();
		dispatch('click', { row: row, column: column, value: value });
	}
	function handleFocus() {
		if (currentPlayer == 0) {
			preview = 'x';
		} else {
			preview = 'o';
		}
	}
	function handleBlur() {
		preview = '';
	}
</script>

<button
	bind:this={element}
	{disabled}
	on:click={handleClick}
	on:focus={handleFocus}
	on:mouseenter={handleFocus}
	on:mouseleave={handleBlur}
	class="cell"
	style="width: {size * 12}px; font-size: {size * 6}px"
>
	{#if !value}
		<div class="preview">
			<Mark opacity="0.2" value={preview}></Mark>
		</div>
	{/if}

	<Mark bind:opacity {value}></Mark>
</button>
<audio bind:this={clickAudio} src="../audio/click.wav" type="audio/wav"> </audio>
<audio bind:this={errorAudio} src="../audio/error1.wav" type="audio/wav"> </audio>

<style>
	.cell {
		display: flex;
		aspect-ratio: 1;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		padding: 8px;
		cursor: pointer;
		transition: all 0.3s ease;
		border: none;
		background-color: rgba(86, 86, 86, 0.236);
		border-radius: 1rem;
	}
	.cell:hover {
		background-color: rgba(124, 124, 124, 0.236);
	}
</style>
