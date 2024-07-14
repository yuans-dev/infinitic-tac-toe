<script>
	export let value = '';
	export let opacity = 1;
	let defaultOpacity = opacity;
	let color = '00000';
	$: color = (value.charCodeAt(0) ** 5).toString(16).substr(0, 6);
	$: if (!value) {
		opacity = defaultOpacity;
	}
</script>

<i class="fa-solid fa-{value}" style="color: #{color}; opacity:{opacity}"></i>
