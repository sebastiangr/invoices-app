<script>
	import Header from '$lib/components/Header.svelte';
  import '../app.postcss'
</script>

<Header />

<div class="container mx-auto px-4 py-8 grid md:grid-cols-2 gap-8 mt-4">
  <slot />
</div>


