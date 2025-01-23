<script>
	import Header from '$lib/components/Header.svelte';
  import '../app.postcss'
  import '../styles.css'
</script>

<Header />

<div class="container mx-auto px-4 py-8 grid gap-8 mt-4">
  <slot />
</div>


