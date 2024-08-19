<script lang="ts">
	import { onMount } from 'svelte';
	import mammoth from 'mammoth';

	let fileInput: HTMLInputElement;
	let convertedText: string = '';
	let isDragging = false;

	function handleFileChange(event: Event): void {
		const target = event.target as HTMLInputElement;
		const file: File | undefined = target.files?.[0];
		if (file) {
			convertFile(file);
		}
	}

	function convertFile(file: File): void {
		const reader = new FileReader();
		reader.onload = function (e: ProgressEvent<FileReader>) {
			const arrayBuffer = e.target?.result as ArrayBuffer;
			mammoth
				.convertToMarkdown({ arrayBuffer: arrayBuffer })
				.then((result: { value: string }) => {
					convertedText = result.value;
				})
				.catch((error: Error) => {
					console.error('Conversion error:', error);
				});
		};
		reader.readAsArrayBuffer(file);
	}

	function handleDragOver(event: DragEvent): void {
		event.preventDefault();
		isDragging = true;
	}

	function handleDragLeave(): void {
		isDragging = false;
	}

	function handleDrop(event: DragEvent): void {
		event.preventDefault();
		isDragging = false;
		const file = event.dataTransfer?.files[0];
		if (file && (file.name.endsWith('.doc') || file.name.endsWith('.docx'))) {
			convertFile(file);
		}
	}

	function handleBrowseClick(): void {
		fileInput.click();
	}

	onMount(() => {
		if (typeof window !== 'undefined') {
			const element = document.getElementById('file-input');
			if (element instanceof HTMLInputElement) {
				fileInput = element;
			}
		}
	});
</script>

<div
	class="bg-background flex min-h-[100dvh] flex-col items-center justify-center px-4 py-12 sm:px-6 lg:px-8"
>
	<div class="mx-auto w-full max-w-2xl space-y-6">
		<div class="space-y-2 text-center">
			<h1 class="text-3xl font-bold tracking-tighter sm:text-4xl md:text-5xl">
				Doc to Markdown Converter
			</h1>
			<p class="text-muted-foreground md:text-xl">
				Easily convert your .doc and .docx files to clean, formatted Markdown.
			</p>
		</div>
		<!-- svelte-ignore a11y-no-static-element-interactions -->
		<div
			class="bg-card text-card-foreground border-input rounded-lg border-2 border-dashed p-6 shadow-sm sm:p-8"
			class:border-primary={isDragging}
			on:dragover={handleDragOver}
			on:dragleave={handleDragLeave}
			on:drop={handleDrop}
		>
			<div class="flex flex-col items-center justify-center space-y-4 p-6 text-center">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="24"
					height="24"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
					class="text-primary h-12 w-12"
				>
					<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path>
					<polyline points="17 8 12 3 7 8"></polyline>
					<line x1="12" x2="12" y1="3" y2="15"></line>
				</svg>
				<div class="space-y-1 text-center">
					<h3 class="text-lg font-medium">Drag and drop your file</h3>
					<p class="text-muted-foreground">
						or{' '}
						<button
							on:click={handleBrowseClick}
							class="ring-offset-background focus-visible:ring-ring text-primary inline-flex h-10 items-center justify-center whitespace-nowrap rounded-md px-4 py-2 text-sm font-medium underline-offset-4 transition-colors hover:underline focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50"
						>
							browse
						</button>
						to select a file
					</p>
				</div>
				<input
					class="border-input bg-background ring-offset-background placeholder:text-muted-foreground focus-visible:ring-ring hidden h-10 w-full rounded-md border px-3 py-2 text-sm file:border-0 file:bg-transparent file:text-sm file:font-medium focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50"
					accept=".doc,.docx"
					id="file-input"
					type="file"
					on:change={handleFileChange}
				/>
			</div>
		</div>
		{#if convertedText}
			<div class="space-y-2">
				<h2 class="text-xl font-bold">Markdown Output</h2>
				<div class="bg-card text-card-foreground rounded-lg border shadow-sm" data-v0-t="card">
					<div class="max-h-[400px] overflow-auto p-6">
						<pre class="whitespace-pre-wrap break-words">
							{convertedText}
						</pre>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>
