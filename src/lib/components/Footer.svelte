<script lang="ts">
	import { GithubIcon } from 'lucide-svelte';

	export let name;
	export let link;
</script>

<footer class="flex w-full footer min-h-28 border-t border-muted footer">
  <div class="grid grid-cols-3 w-full">
		<div class="flex items-center justify-center text-muted">
			<p>Powered by ChromaSky</p>
		</div>
		<a href="{link}" class="flex items-center justify-center text-muted">
			Photos by {name} on bsky.app
		</a>
		<a href="https://github.com/kay-xr/chroma-sky" class="flex items-center justify-center text-muted">
			<GithubIcon />
		</a>
	</div>
</footer>

<style>
	.footer {
			background-color: #000000;
	}
</style>