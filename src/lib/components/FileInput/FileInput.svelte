<script lang="ts">
	import { createWorker } from 'tesseract.js';
	import { toast } from 'svelte-sonner';
	import { Button } from '$lib/components/ui/button/index.js';

	let dropZone: HTMLElement;

	let loading: boolean = $state(false);
	let recognizedText: string = $state('');
	$effect(() => {
		['dragenter', 'dragover', 'dragleave', 'drop'].forEach((eventName) => {
			dropZone.addEventListener(
				eventName,
				(e: Event) => {
					e.preventDefault();
					e.stopPropagation();
					if (eventName === 'dragenter' || eventName === 'dragover') {
						dropZone.classList.add('opacity-60');
					} else {
						dropZone.classList.remove('opacity-60');
					}
				},
				false
			);
		});

		dropZone.addEventListener(
			'drop',
			async (e: DragEvent) => {
				e.preventDefault();
				e.stopPropagation();
				loading = true;
				let dt = e.dataTransfer;
				if (dt) {
					let files = dt.files;
					if (files) {
						if (files.length > 1) {
							toast.error(
								'Multiple file upload is not supported. Please upload one image at a time.'
							);
							loading = false;
							return;
						}
						let firstFile = files.item(0);
						if (firstFile) {
							if (/^image\//.test(firstFile.type)) {
								const grayscaleFile = await convertToGrayscale(firstFile);

								if (grayscaleFile) {
									(async () => {
										const worker = await createWorker('eng');
										const ret = await worker.recognize(grayscaleFile);
										recognizedText = ret.data.text;
										loading = false;
										await worker.terminate();
									})();
								}
							} else {
								toast.error(`Uploaded file is not an image.`);
								loading = false;
							}
						}
					}
				}
			},
			false
		);
	});

	async function convertToGrayscale(fileInput: File): Promise<File> {
		return new Promise<File>((resolve, reject) => {
			const img = new Image();
			const reader = new FileReader();

			reader.onload = function (e: ProgressEvent<FileReader>) {
				if (e.target) {
					img.src = e.target.result as string;
					img.onload = function () {
						const canvas = document.createElement('canvas');
						const ctx = canvas.getContext('2d');

						if (ctx) {
							canvas.width = img.width;
							canvas.height = img.height;

							ctx.drawImage(img, 0, 0);

							const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
							const data = imageData.data;

							// Grayscale conversion
							for (let i = 0; i < data.length; i += 4) {
								const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
								data[i] = avg; // R
								data[i + 1] = avg; // G
								data[i + 2] = avg; // B
							}

							// Contrast adjustment
							const contrast = 1.3;
							const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));

							for (let i = 0; i < data.length; i += 4) {
								data[i] = clamp(factor * (data[i] - 128) + 128); // R
								data[i + 1] = clamp(factor * (data[i + 1] - 128) + 128); // G
								data[i + 2] = clamp(factor * (data[i + 2] - 128) + 128); // B
							}

							ctx.putImageData(imageData, 0, 0);

							canvas.toBlob((blob) => {
								if (blob) {
									const grayscaleFile = new File([blob], fileInput.name, {
										type: 'image/png',
										lastModified: Date.now()
									});
									resolve(grayscaleFile);
								} else {
									toast.error('Grayscale conversion failed.');
									loading = false;
									reject(new Error('Grayscale conversion failed.'));
								}
							}, 'image/png');
						}
					};
				}
			};

			reader.onerror = function () {
				reject(new Error('Failed to read image file.'));
				loading = false;
			};

			reader.readAsDataURL(fileInput);

			function clamp(value: number): number {
				return Math.max(0, Math.min(Math.round(value), 255));
			}
		});
	}

	function reset() {
		window.location.reload();
	}
</script>

<main class="flex w-full justify-center p-4 md:mt-0">
	{#if loading}
		<span class="loader"></span>
	{:else if !loading && recognizedText}
		<div class="flex flex-col items-center justify-center gap-4">
			<div class="min-h-[50vh] w-full rounded-xl border-2 xl:w-[50vw]">
				<div class="flex h-full justify-center p-4 text-xl">
					{recognizedText}
				</div>
			</div>
			<Button onclick={reset}>Reset</Button>
		</div>
	{:else}
		<div
			bind:this={dropZone}
			class="dropzone cursor min-h-[50vh] w-full rounded-xl border-2 transition-opacity xl:w-[50vw]"
		>
			<div class="flex h-full items-center justify-center text-xl opacity-50">
				Drop your image here...
			</div>
		</div>
	{/if}
</main>

<style>
	.loader {
		width: 48px;
		height: 48px;
		border: 4px solid #4d4d4d;
		border-bottom-color: #ffffff;
		border-radius: 50%;
		display: inline-block;
		box-sizing: border-box;
		animation: rotation 1s linear infinite;
	}

	@keyframes rotation {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}
</style>
