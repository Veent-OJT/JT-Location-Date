<script lang="ts">
	import { onMount } from 'svelte';
	import mapboxgl from 'mapbox-gl';
	import MapboxGeocoder from '@mapbox/mapbox-gl-geocoder';
	import '@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css';
	import 'mapbox-gl/dist/mapbox-gl.css';

	let map: mapboxgl.Map;
	let eventName = '';
	let subdomain = '';
	let startDate = '';
	let startTime = '';
	let endDate = '';
	let endTime = '';
	let selectedLocation = '';
	let description = '';
	let logoFile: File | null = null;
	let posterFile: File | null = null;
	let isDraggingLogo = false;
	let isDraggingPoster = false;

	// Replace with your Mapbox access token
	mapboxgl.accessToken =
		'pk.eyJ1IjoiamllY2xhcmtkZXYyNSIsImEiOiJjbTc4a2k2dDMxODYzMmxwdnJib2MzaDI2In0.jitzeX20xN17lvykcKg7aQ';

	onMount(() => {
		map = new mapboxgl.Map({
			container: 'map',
			style: 'mapbox://styles/mapbox/streets-v12',
			center: [121.774, 12.8797], // Philippines
			zoom: 6
		});

		const geocoder = new MapboxGeocoder({
			accessToken: mapboxgl.accessToken,
			mapboxgl: mapboxgl,
			marker: true,
			placeholder: 'Enter location...'
		});

		map.addControl(geocoder);

		geocoder.on('result', (event) => {
			selectedLocation = event.result.place_name;
		});
	});

	function handleClearLocation() {
		selectedLocation = '';
		if (map) {
			map.flyTo({
				center: [121.774, 12.8797],
				zoom: 6
			});
		}
	}

	function handleFileUpload(event: Event, type: 'logo' | 'poster') {
		const input = event.target as HTMLInputElement;
		if (input.files && input.files[0]) {
			const file = input.files[0];
			if (type === 'logo') {
				logoFile = file;
			} else {
				posterFile = file;
			}
		}
	}

	function handleDragOver(event: DragEvent) {
		event.preventDefault();
		const target = event.currentTarget as HTMLDivElement;
		if (target.dataset.type === 'logo') {
			isDraggingLogo = true;
		} else {
			isDraggingPoster = true;
		}
	}

	function handleDragLeave(event: DragEvent) {
		event.preventDefault();
		const target = event.currentTarget as HTMLDivElement;
		if (target.dataset.type === 'logo') {
			isDraggingLogo = false;
		} else {
			isDraggingPoster = false;
		}
	}

	function handleDrop(event: DragEvent, type: 'logo' | 'poster') {
		event.preventDefault();
		isDraggingLogo = false;
		isDraggingPoster = false;

		const files = event.dataTransfer?.files;
		if (files && files[0]) {
			const file = files[0];
			if (file.type.startsWith('image/')) {
				if (type === 'logo') {
					logoFile = file;
				} else {
					posterFile = file;
				}
			}
		}
	}
</script>

<div class="mx-auto max-w-4xl p-6">
	<h1 class="mb-6 text-2xl font-bold">Create New Event</h1>

	<div class="space-y-6">
		<!-- Event Name -->
		<div class="space-y-2">
			<label class="font-medium text-gray-900">Event Name</label>
			<input
				type="text"
				placeholder="Enter your event name"
				bind:value={eventName}
				class="w-full rounded-lg border border-gray-200 p-3 focus:border-transparent focus:ring-2 focus:ring-blue-500"
			/>
		</div>

		<!-- Subdomain -->
		<div class="space-y-2">
			<label class="font-medium text-gray-900">Subdomain</label>
			<div class="flex items-center">
				<div class="flex w-full items-center rounded-lg border border-gray-200">
					<span class="p-3 text-gray-500">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5"
							viewBox="0 0 20 20"
							fill="currentColor"
						>
							<path
								fill-rule="evenodd"
								d="M10 18a8 8 0 100-16 8 8 0 000 16zM4.332 8.027a6.012 6.012 0 011.912-2.706C6.512 5.73 6.974 6 7.5 6A1.5 1.5 0 019 7.5V8a2 2 0 004 0 2 2 0 011.523-1.943A5.977 5.977 0 0116 10c0 .34-.028.675-.083 1H15a2 2 0 00-2 2v2.197A5.973 5.973 0 0110 16v-2a2 2 0 00-2-2 2 2 0 01-2-2 2 2 0 00-1.668-1.973z"
								clip-rule="evenodd"
							/>
						</svg>
					</span>
					<input
						type="text"
						placeholder="your-event"
						bind:value={subdomain}
						class="flex-1 bg-transparent p-3 focus:outline-none"
					/>
					<span class="p-3 text-gray-500">.veent.co</span>
				</div>
			</div>
		</div>

		<!-- Date/Time Section -->
		<div class="grid grid-cols-1 gap-6 md:grid-cols-2">
			<!-- Start Date/Time -->
			<div class="space-y-2">
				<label class="flex items-center gap-2 font-medium text-gray-900">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-5 w-5 text-gray-500"
						viewBox="0 0 20 20"
						fill="currentColor"
					>
						<path
							fill-rule="evenodd"
							d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z"
							clip-rule="evenodd"
						/>
					</svg>
					Start
				</label>
				<div class="grid grid-cols-2 gap-4">
					<input
						type="date"
						bind:value={startDate}
						class="w-full rounded-lg border border-gray-200 p-3 focus:border-transparent focus:ring-2 focus:ring-blue-500"
					/>
					<input
						type="time"
						bind:value={startTime}
						class="w-full rounded-lg border border-gray-200 p-3 focus:border-transparent focus:ring-2 focus:ring-blue-500"
					/>
				</div>
			</div>

			<!-- End Date/Time -->
			<div class="space-y-2">
				<label class="flex items-center gap-2 font-medium text-gray-900">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-5 w-5 text-gray-500"
						viewBox="0 0 20 20"
						fill="currentColor"
					>
						<path
							fill-rule="evenodd"
							d="M6 2a1 1 0 00-1 1v1H4a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V6a2 2 0 00-2-2h-1V3a1 1 0 10-2 0v1H7V3a1 1 0 00-1-1zm0 5a1 1 0 000 2h8a1 1 0 100-2H6z"
							clip-rule="evenodd"
						/>
					</svg>
					End
				</label>
				<div class="grid grid-cols-2 gap-4">
					<input
						type="date"
						bind:value={endDate}
						class="w-full rounded-lg border border-gray-200 p-3 focus:border-transparent focus:ring-2 focus:ring-blue-500"
					/>
					<input
						type="time"
						bind:value={endTime}
						class="w-full rounded-lg border border-gray-200 p-3 focus:border-transparent focus:ring-2 focus:ring-blue-500"
					/>
				</div>
			</div>
		</div>

		<!-- Location -->
		<div class="space-y-2">
			<label class="flex items-center gap-2 font-medium text-gray-900">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-5 w-5 text-gray-500"
					viewBox="0 0 20 20"
					fill="currentColor"
				>
					<path
						fill-rule="evenodd"
						d="M5.05 4.05a7 7 0 119.9 9.9L10 18.9l-4.95-4.95a7 7 0 010-9.9zM10 11a2 2 0 100-4 2 2 0 000 4z"
						clip-rule="evenodd"
					/>
				</svg>
				Location
			</label>
			<div class="relative">
				<input
					type="text"
					placeholder="Add event location"
					value={selectedLocation}
					readonly
					class="w-full cursor-pointer rounded-lg border border-gray-200 bg-white p-3"
				/>
				{#if selectedLocation}
					<button
						class="absolute top-1/2 right-2 -translate-y-1/2 text-gray-500 hover:text-gray-700"
						on:click={handleClearLocation}
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5"
							viewBox="0 0 20 20"
							fill="currentColor"
						>
							<path
								fill-rule="evenodd"
								d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
								clip-rule="evenodd"
							/>
						</svg>
					</button>
				{/if}
			</div>
			<div id="map" class="h-64 w-full rounded-lg border border-gray-200" />
		</div>

		<!-- Description -->
		<div class="space-y-2">
			<label class="font-medium text-gray-900">Description</label>
			<div class="overflow-hidden rounded-lg border border-gray-200">
				<div class="flex gap-2 border-b border-gray-200 p-2">
					<button class="rounded p-1 hover:bg-gray-100">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5"
							viewBox="0 0 20 20"
							fill="currentColor"
						>
							<path
								fill-rule="evenodd"
								d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z"
								clip-rule="evenodd"
							/>
						</svg>
					</button>
				</div>
				<textarea
					bind:value={description}
					placeholder="Describe your event..."
					class="min-h-[200px] w-full p-3 focus:ring-2 focus:ring-blue-500 focus:outline-none"
				/>
			</div>
		</div>

		<!-- Event Assets -->
		<div class="space-y-4">
			<h2 class="font-medium text-gray-900">Event Assets</h2>

			<!-- Company Logo Upload -->
			<div class="space-y-2">
				<label class="text-gray-700">Company Logo</label>
				<div
					role="button"
					tabindex="0"
					class="rounded-lg border-2 border-dashed {isDraggingLogo
						? 'border-red-500 bg-red-50'
						: 'border-gray-200'} p-8 transition-colors hover:border-gray-500"
					data-type="logo"
					on:dragover={handleDragOver}
					on:dragleave={handleDragLeave}
					on:drop={(e) => handleDrop(e, 'logo')}
				>
					<div class="flex flex-col items-center">
						{#if logoFile}
							<div class="flex items-center gap-2">
								<i class="ri-check-line text-green-500"></i>
								<span class="text-sm text-gray-600">{logoFile.name}</span>
								<button class="text-red-500 hover:text-red-700" on:click={() => (logoFile = null)}>
									<i class="ri-close-line"></i>
								</button>
							</div>
						{:else}
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="h-12 w-12 text-gray-400"
								fill="none"
								viewBox="0 0 24 24"
								stroke="currentColor"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
								/>
							</svg>
							<p class="mt-2">
								<label class="cursor-pointer text-red-500">
									Upload a file
									<input
										type="file"
										class="hidden"
										accept="image/*"
										on:change={(e) => handleFileUpload(e, 'logo')}
									/>
								</label>
								or drag and drop
							</p>
							<p class="mt-1 text-sm text-gray-500">PNG, JPG, GIF up to 10MB</p>
						{/if}
					</div>
				</div>
			</div>

			<!-- Event Poster Upload -->
			<div class="space-y-2">
				<label class="text-gray-700">Event Poster</label>
				<div
					role="button"
					tabindex="0"
					class="rounded-lg border-2 border-dashed {isDraggingPoster
						? 'border-red-500 bg-red-50'
						: 'border-gray-200'} p-8 transition-colors hover:border-gray-500"
					data-type="poster"
					on:dragover={handleDragOver}
					on:dragleave={handleDragLeave}
					on:drop={(e) => handleDrop(e, 'poster')}
				>
					<div class="flex flex-col items-center">
						{#if posterFile}
							<div class="flex items-center gap-2">
								<i class="ri-check-line text-green-500"></i>
								<span class="text-sm text-gray-600">{posterFile.name}</span>
								<button
									class="text-red-500 hover:text-red-700"
									on:click={() => (posterFile = null)}
								>
									<i class="ri-close-line"></i>
								</button>
							</div>
						{:else}
							<svg
								xmlns="http://www.w3.org/2000/svg"
								class="h-12 w-12 text-gray-400"
								fill="none"
								viewBox="0 0 24 24"
								stroke="currentColor"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"
								/>
							</svg>
							<p class="mt-2">
								<label class="cursor-pointer text-red-500">
									Upload a file
									<input
										type="file"
										class="hidden"
										accept="image/*"
										on:change={(e) => handleFileUpload(e, 'poster')}
									/>
								</label>
								or drag and drop
							</p>
							<p class="mt-1 text-sm text-gray-500">PNG, JPG, GIF up to 10MB</p>
						{/if}
					</div>
				</div>
			</div>
		</div>

		<!-- Create Button -->
		<button class="w-full rounded-lg bg-red-500 py-3 text-white transition-colors hover:bg-red-600">
			Create Event
		</button>
	</div>
</div>

<style>
	:global(.mapboxgl-ctrl-geocoder) {
		width: 100% !important;
		max-width: none !important;
		margin: 10px 0 !important;
	}

	:global(.mapboxgl-map) {
		width: 100%;
		height: 100%;
		min-height: 300px;
	}
</style>
