<script>
	import '../app.css';
	import { Sun, Moon } from 'lucide-svelte';
	import { ModeWatcher, toggleMode } from 'mode-watcher';
	import { Button } from '$lib/components/ui/button/index';
	import { Toaster } from '$lib/components/ui/sonner';
	let { children } = $props();
</script>

<ModeWatcher />
<Toaster />
<nav class="absolute w-full p-4">
	<div class="flex justify-between">
		<a href="/">
			<div class="text-5xl font-medium">PixelText.</div>
			<p class="text-xl">Convert image to text easily.</p>
		</a>

		<Button onclick={toggleMode} variant="outline" size="icon">
			<Sun
				class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
			/>
			<Moon
				class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
			/>
			<span class="sr-only">Toggle theme</span>
		</Button>
	</div>
</nav>
{@render children()}
