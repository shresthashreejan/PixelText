<script lang="ts">
	let dropZone: HTMLElement;

	$effect(() => {
		['dragenter', 'dragover', 'dragleave', 'drop'].forEach((eventName) => {
			dropZone.addEventListener(
				eventName,
				(e: Event) => {
					e.preventDefault();
					e.stopPropagation();
					if (eventName === 'dragenter' || eventName === 'dragover') {
						dropZone.classList.add('opacity-50');
					} else {
						dropZone.classList.remove('opacity-50');
					}
				},
				false
			);
		});

		dropZone.addEventListener(
			'drop',
			(e: DragEvent) => {
				e.preventDefault();
				e.stopPropagation();
				let dt = e.dataTransfer;
				if (dt) {
					let files = dt.files;
					if (files && files.length > 0) {
						let firstFile = files.item(0);
						if (firstFile) {
							if (/^image\//.test(firstFile.type)) {
								console.log(firstFile);
							} else {
								throw new Error(`First file ${firstFile.name} is not an image.`);
							}
						}
					}
				}
			},
			false
		);
	});
</script>

<main class="flex justify-center p-4">
	<div
		bind:this={dropZone}
		class="dropzone cursor min-h-[50vh] min-w-full rounded-xl border-2 transition-opacity md:min-w-[50vw]"
	>
		<div class="flex h-full items-center justify-center opacity-50">Drop your image here...</div>
	</div>
</main>
