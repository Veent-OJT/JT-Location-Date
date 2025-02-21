<script lang="ts">
	import { onMount } from 'svelte';
	import 'leaflet/dist/leaflet.css';
	import 'leaflet-geosearch/dist/geosearch.css';

	// date picker
	import AirDatepicker, {
		type AirDatepickerOptions,
		type AirDatepickerPosition
	} from 'air-datepicker';
	import 'air-datepicker/air-datepicker.css';
	import localeEn from 'air-datepicker/locale/en';

	// font family poppins from fontsource
	import '@fontsource/poppins/400.css';
	import '@fontsource/poppins/500.css';
	// remixicon
	import 'remixicon/fonts/remixicon.css';

	import {
		validateDateTimeRange,
		formatDateTimeRange,
		DEFAULT_DATE_FORMAT
	} from '$lib/utils/datetime';

	// toast sooner
	import { Toaster, toast } from 'svelte-sonner';

	let map: any;
	let eventName = '';
	let subdomain = '';
	let selectedLocation = '';
	let description = '';
	let saveAsDraft = false;

	// errors for form validation
	let errors = {
		eventName: '',
		subdomain: '',
		dateTime: '',
		location: '',
		description: ''
	};

	let isFormValid = false;

	// Add validation function
	const validateForm = () => {
		// Reset errors
		errors = {
			eventName: '',
			subdomain: '',
			dateTime: '',
			location: '',
			description: ''
		};

		// Validate event name
		if (!eventName.trim()) {
			errors.eventName = 'Event name is required';
		}

		// Validate subdomain
		if (!subdomain.trim()) {
			errors.subdomain = 'Subdomain is required';
		} else if (!/^[a-z0-9-]+$/.test(subdomain)) {
			errors.subdomain = 'Subdomain can only contain lowercase letters, numbers, and hyphens';
		}

		// Validate date/time
		if (!startDate || !endDate || !startTime || !endTime) {
			errors.dateTime = 'Start and end date/time are required';
		} else {
			const startDateTime = new Date(`${startDate}T${startTime}`);
			const endDateTime = new Date(`${endDate}T${endTime}`);

			if (endDateTime <= startDateTime) {
				errors.dateTime = 'End date/time must be after start date/time';
			}
		}

		// Validate location
		if (!selectedLocation.trim()) {
			errors.location = 'Location is required';
		}

		// Validate description
		if (!description.trim()) {
			errors.description = 'Description is required';
		}

		// Update form validity
		isFormValid = Object.values(errors).every((error) => !error);
	};

	// Add reactive statement to validate form when values change
	$: {
		eventName, subdomain, startDate, endDate, startTime, endTime, selectedLocation, description;
		validateForm();
	}

	export let startDate = '';
	export let endDate = '';
	export let startTime = '09:00';
	export let endTime = '17:00';

	let startDatePicker: AirDatepicker<HTMLElement> | undefined;
	let endDatePicker: AirDatepicker<HTMLElement> | undefined;

	// logo and poster
	let logoFile: File | null = null;
	let posterFile: File | null = null;
	let backgroundImageFile: File | null = null;
	let isDraggingLogo = false;
	let isDraggingPoster = false;
	let isDraggingBackgroundImage = false;

	function handleStartDateSelect({ date }: { date: Date | Date[] }) {
		if (endDatePicker && date) {
			const singleDate = Array.isArray(date) ? date[0] : date;
			endDatePicker.update({
				minDate: singleDate
			});

			const currentEndDate = endDatePicker.selectedDates[0];
			if (currentEndDate && currentEndDate < singleDate) {
				endDatePicker.selectDate(singleDate);
			}
		}
	}

	function handleSubmit() {
		validateForm();

		if (!isFormValid) {
			toast.error('Please fix the errors before submitting');
			return;
		}

		// Create form data object with all fields
		const formData = {
			// Basic Information
			eventName,
			subdomain,
			description,
			saveAsDraft,

			// Date and Time
			dateTime: {
				startDate,
				startTime,
				endDate,
				endTime
			},

			// Location
			location: selectedLocation,

			// Files
			files: {
				logo: logoFile
					? {
							name: logoFile.name,
							type: logoFile.type,
							size: `${(logoFile.size / 1024 / 1024).toFixed(2)}MB`
						}
					: null,
				poster: posterFile
					? {
							name: posterFile.name,
							type: posterFile.type,
							size: `${(posterFile.size / 1024 / 1024).toFixed(2)}MB`
						}
					: null,
				background: backgroundImageFile
					? {
							name: backgroundImageFile.name,
							type: backgroundImageFile.type,
							size: `${(backgroundImageFile.size / 1024 / 1024).toFixed(2)}MB`
						}
					: null
			}
		};

		// Log the complete form data
		console.log('Form Data:', {
			...formData,
			rawDateTime: {
				startDateTime: new Date(`${startDate}T${startTime}`),
				endDateTime: new Date(`${endDate}T${endTime}`)
			}
		});

		toast.success('Event has been created');
	}

	onMount(async () => {
		const L = await import('leaflet');
		const { OpenStreetMapProvider, GeoSearchControl } = await import('leaflet-geosearch');

		const commonConfig: AirDatepickerOptions<HTMLElement> = {
			dateFormat: DEFAULT_DATE_FORMAT,
			minDate: new Date(),
			autoClose: true,
			isMobile: true,
			classes: 'custom-datepicker',
			locale: localeEn
		};

		startDatePicker = new AirDatepicker('#start-date', {
			...commonConfig,
			onSelect: ({ date, formattedDate }) => {
				startDate = formattedDate as string;
				handleStartDateSelect({ date });
			}
		});

		endDatePicker = new AirDatepicker('#end-date', {
			...commonConfig,
			onSelect: ({ formattedDate }) => {
				endDate = formattedDate as string;
			}
		});

		map = L.map('map').setView([12.8797, 121.774], 6);

		L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution: '© OpenStreetMap contributors'
		}).addTo(map);

		const searchControl = new GeoSearchControl({
			provider: new OpenStreetMapProvider(),
			style: 'bar',
			autoComplete: true,
			autoCompleteDelay: 250,
			showMarker: true,
			searchLabel: 'Enter location...',
			notFoundMessage: 'Sorry, that address could not be found.'
		});

		map.addControl(searchControl);

		map.on('geosearch/showlocation', (event: { location: any }) => {
			selectedLocation = `${event.location.label}`;

			// Log all location data
			// console.log('Selected Location Full Data:', {
			// 	label: event.location.label,
			// 	x: event.location.x, // longitude
			// 	y: event.location.y, // latitude
			// 	bounds: event.location.bounds,
			// 	raw: event.location.raw // Contains all raw data from the geocoding service
			// });
		});

		return () => {
			startDatePicker?.destroy();
			endDatePicker?.destroy();
		};
	});

	function handleClearLocation() {
		selectedLocation = '';
		if (map) {
			map.setView([12.8797, 121.774], 6);
		}
	}

	function handleFileUpload(event: Event, type: 'logo' | 'poster' | 'background') {
		const input = event.target as HTMLInputElement;
		if (input.files && input.files[0]) {
			const file = input.files[0];
			switch (type) {
				case 'logo':
					logoFile = file;
					break;
				case 'poster':
					posterFile = file;
					break;
				case 'background':
					backgroundImageFile = file;
					break;
			}
		}
	}

	function handleDragOver(event: DragEvent) {
		event.preventDefault();
		const target = event.currentTarget as HTMLDivElement;
		switch (target.dataset.type) {
			case 'logo':
				isDraggingLogo = true;
				break;
			case 'poster':
				isDraggingPoster = true;
				break;
			case 'background':
				isDraggingBackgroundImage = true;
				break;
		}
	}

	function handleDragLeave(event: DragEvent) {
		event.preventDefault();
		const target = event.currentTarget as HTMLDivElement;
		switch (target.dataset.type) {
			case 'logo':
				isDraggingLogo = false;
				break;
			case 'poster':
				isDraggingPoster = false;
				break;
			case 'background':
				isDraggingBackgroundImage = false;
				break;
		}
	}

	function handleDrop(event: DragEvent, type: 'logo' | 'poster' | 'background') {
		event.preventDefault();
		isDraggingLogo = false;
		isDraggingPoster = false;
		isDraggingBackgroundImage = false;

		const files = event.dataTransfer?.files;
		if (files && files[0] && files[0].type.startsWith('image/')) {
			switch (type) {
				case 'logo':
					logoFile = files[0];
					break;
				case 'poster':
					posterFile = files[0];
					break;
				case 'background':
					backgroundImageFile = files[0];
					break;
			}
		}
	}
</script>

<div class="mx-auto max-w-[1200px] p-4 md:p-8">
	<form on:submit|preventDefault={handleSubmit} class="grid grid-cols-1 gap-8 lg:grid-cols-2">
		<!-- Left Column -->
		<div class="space-y-6">
			<!-- Event Name -->
			<div class="space-y-2">
				<label for="subdomain" class="block text-sm font-medium text-gray-700">Event Name</label>
				<input
					type="text"
					placeholder="Enter your event name"
					bind:value={eventName}
					class="w-full rounded-[12px] border border-gray-200 px-4 py-4 text-[16px] text-gray-500 placeholder-gray-500 transition-colors focus:border-2 focus:border-red-500 focus:outline-none"
				/>
				{#if errors.eventName}
					<p class="text-sm text-red-500">{errors.eventName}</p>
				{/if}
			</div>

			<!-- Subdomain -->
			<div class="mt-8 space-y-2">
				<label for="subdomain" class="block text-sm font-medium text-gray-700">Subdomain</label>
				<div class="flex items-center overflow-hidden rounded-[12px] border border-gray-200">
					<span class="p-4">
						<i class="ri-global-line text-xl text-gray-400"></i>
					</span>
					<input
						type="text"
						placeholder="your-event"
						bind:value={subdomain}
						class="flex-1 px-2 py-4 text-[16px] text-gray-500 transition-colors focus:border-2 focus:border-red-500 focus:outline-none"
					/>
					<span class="p-4 text-gray-500">.veent.co</span>
				</div>
				{#if errors.subdomain}
					<p class="text-sm text-red-500">{errors.subdomain}</p>
				{/if}
			</div>

			<!-- Date/Time Section -->
			<div class="mt-8 space-y-6">
				<!-- Start Date/Time -->
				<div class="space-y-2">
					<div class="flex items-center gap-2">
						<i class="ri-calendar-line text-lg text-gray-400"></i>
						<label for="startDate" class="text-sm font-medium text-gray-700">Start</label>
					</div>
					<div class="grid grid-cols-2 gap-4">
						<div class="relative">
							<input
								id="start-date"
								type="text"
								placeholder="Select start date"
								class="w-full appearance-none rounded-[12px] border border-gray-200 px-4 py-4 text-gray-500 focus:outline-none"
							/>
						</div>
						<div class="relative">
							<label for="startTime" class="sr-only">Start Time</label>

							<input
								id="startTime"
								type="time"
								bind:value={startTime}
								placeholder="Select end time"
								class="w-full appearance-none rounded-[12px] border border-gray-200 px-4 py-4 text-gray-500 focus:outline-none"
							/>
						</div>
					</div>
				</div>

				<!-- End Date/Time -->
				<div class="space-y-2">
					<div class="flex items-center gap-2">
						<i class="ri-time-line text-lg text-gray-400"></i>
						<label for="endDate" class="text-sm font-medium text-gray-700">End</label>
					</div>
					<div class="grid grid-cols-2 gap-4">
						<div class="relative">
							<input
								id="end-date"
								type="text"
								placeholder="Select end date"
								class="w-full appearance-none rounded-[12px] border border-gray-200 px-4 py-4 text-gray-500 focus:outline-none"
							/>
						</div>
						<div class="relative">
							<label for="endTime" class="sr-only">End Time</label>

							<input
								id="endTime"
								type="time"
								bind:value={endTime}
								placeholder="Select end time"
								class="w-full appearance-none rounded-[12px] border border-gray-200 px-4 py-4 text-gray-500 focus:outline-none"
							/>
						</div>
					</div>
					{#if errors.dateTime}
						<p class="text-sm text-red-500">{errors.dateTime}</p>
					{/if}
				</div>

				<!-- Location (Keep existing map implementation) -->
				<div class="space-y-2">
					<!-- Location -->
					<div class="space-y-2">
						<div class="flex items-center gap-2">
							<i class="ri-map-pin-line text-lg text-gray-400"></i>
							<label for="location" class="text-sm font-medium text-gray-700">Location</label>
						</div>
						<div class="relative">
							<input
								type="text"
								placeholder="Add event location"
								value={selectedLocation}
								readonly
								class="w-full cursor-pointer rounded-lg border border-gray-200 bg-white px-4 py-3 pr-10"
							/>
							{#if selectedLocation}
								<button
									aria-labelledby="clear-location"
									class="absolute top-1/2 right-2 -translate-y-1/2 rounded-full p-1 text-gray-500 transition-all duration-200 hover:bg-gray-100 hover:text-gray-700"
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
						{#if errors.location}
							<p class="text-sm text-red-500">{errors.location}</p>
						{/if}
						<div id="map" class="h-64 w-full rounded-lg border border-gray-200"></div>
					</div>
				</div>

				<!-- Description -->
				<div class="space-y-2">
					<label for="description" class="text-gray-700">Description</label>
					<div class="overflow-hidden rounded-[12px] border border-gray-200 bg-white">
						<!-- Toolbar -->
						<div class="flex items-center gap-4 border-b border-gray-200 px-4 py-2">
							<button class="rounded p-1 font-semibold hover:bg-gray-100">B</button>
							<button class="rounded p-1 italic hover:bg-gray-100">I</button>
							<button class="rounded p-1 underline hover:bg-gray-100">U</button>
							<div class="h-5 w-[1px] bg-gray-200"></div>
							<button aria-labelledby="list-unordered" class="rounded p-1 hover:bg-gray-100">
								<i class="ri-list-unordered text-lg"></i>
							</button>
							<button aria-labelledby="list-ordered" class="rounded p-1 hover:bg-gray-100">
								<i class="ri-list-ordered text-lg"></i>
							</button>
							<button aria-labelledby="link" class="rounded p-1 hover:bg-gray-100">
								<i class="ri-link text-lg"></i>
							</button>
						</div>

						<!-- Textarea -->
						<textarea
							bind:value={description}
							placeholder="Describe your event..."
							class="min-h-[200px] w-full resize-none px-4 py-3 text-gray-600 placeholder-gray-400 focus:outline-none"
						></textarea>
					</div>
					{#if errors.description}
						<p class="text-sm text-red-500">{errors.description}</p>
					{/if}
				</div>
			</div>
		</div>

		<!-- Right Column -->
		<div class="space-y-6">
			<div class="rounded-lg border border-gray-200 bg-white p-6 shadow-sm">
				<!-- Preview Button -->
				<div class="mb-6 flex justify-end">
					<button
						class="flex items-center gap-2 text-gray-600 transition-colors hover:text-gray-900"
					>
						<i class="ri-eye-line"></i>
						<span>Preview</span>
					</button>
				</div>

				<!-- Company Logo -->
				<div class="mb-2 space-y-2">
					<label for="logo" class="mb-2 block text-sm font-medium text-gray-700">Company Logo</label
					>
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
									<!-- remove logo -->
									<button
										aria-labelledby="remove-logo"
										class="text-red-500 hover:text-red-700"
										on:click={() => (logoFile = null)}
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
								<p class="mt-4">
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
								<p class="mt-2 text-sm text-gray-500">PNG, JPG, GIF up to 10MB</p>
							{/if}
						</div>
					</div>
				</div>

				<!-- Event Poster -->
				<div class="space-y-2">
					<label for="poster" class="mb-2 block text-sm font-medium text-gray-700"
						>Event Poster</label
					>
					<div
						role="button"
						tabindex="0"
						data-type="poster"
						class="rounded-lg border-2 border-dashed {isDraggingPoster
							? 'border-red-500 bg-red-50'
							: 'border-gray-200'} p-8 transition-colors hover:border-gray-500"
						on:dragover={handleDragOver}
						on:dragleave={handleDragLeave}
						on:drop={(e) => handleDrop(e, 'poster')}
					>
						<div class="flex flex-col items-center">
							{#if posterFile}
								<div class="flex items-center gap-2">
									<i class="ri-check-line text-green-500"></i>
									<span class="text-sm text-gray-600">{posterFile.name}</span>
									<!-- remove poster -->
									<button
										aria-labelledby="remove-poster"
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
								<p class="mt-4">
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
								<p class="mt-2 text-sm text-gray-500">PNG, JPG, GIF up to 10MB</p>
							{/if}
						</div>
					</div>
				</div>
				<!-- Background Image -->
				<div class="mt-2 space-y-2">
					<label for="background" class="mb-2 block text-sm font-medium text-gray-700">
						Background Image
					</label>
					<div
						role="button"
						tabindex="0"
						data-type="background"
						class="rounded-lg border-2 border-dashed {isDraggingBackgroundImage
							? 'border-red-500 bg-red-50'
							: 'border-gray-200'} p-8 transition-colors hover:border-gray-500"
						on:dragover={handleDragOver}
						on:dragleave={handleDragLeave}
						on:drop={(e) => handleDrop(e, 'background')}
					>
						<div class="flex flex-col items-center">
							{#if backgroundImageFile}
								<div class="flex items-center gap-2">
									<i class="ri-check-line text-green-500"></i>
									<span class="text-sm text-gray-600">{backgroundImageFile.name}</span>
									<button
										aria-labelledby="remove-background"
										class="text-red-500 hover:text-red-700"
										on:click={() => (backgroundImageFile = null)}
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
								<p class="mt-4">
									<label class="cursor-pointer text-red-500">
										Upload a file
										<input
											type="file"
											class="hidden"
											accept="image/*"
											on:change={(e) => handleFileUpload(e, 'background')}
										/>
									</label>
									or drag and drop
								</p>
								<p class="mt-2 text-sm text-gray-500">PNG, JPG, GIF up to 10MB</p>
							{/if}
						</div>
					</div>
				</div>
				<!-- Event Options -->
				<div class="mt-5 space-y-4">
					<div class="flex items-center">
						<input
							type="checkbox"
							id="saveAsDraft"
							bind:checked={saveAsDraft}
							class="text-primary focus:ring-primary h-4 w-4 rounded border-gray-300"
						/>
						<label for="saveAsDraft" class="ml-2 block text-sm text-gray-700">Save as Draft</label>
					</div>
				</div>
				<!-- Create Button -->
				<div class="mt-8">
					<button
						type="submit"
						disabled={!isFormValid}
						class="w-full rounded-lg bg-red-500 py-4 text-white transition-colors {isFormValid
							? 'hover:bg-red-600'
							: 'cursor-not-allowed opacity-50'}"
					>
						Create Event
					</button>
				</div>
			</div>
		</div>
	</form>
</div>
<Toaster position="top-right" richColors />

<style>
	:global(.custom-datepicker) {
		--adp-background-color: #fff;
		--adp-accent-color: #4a90e2;
		--adp-color: #333;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
		border-radius: 4px;
		padding: 1rem;
		z-index: 9999; /* Add this line */
		position: relative; /* Add this line */
	}

	:global(.custom-datepicker .air-datepicker-cell.-disabled-) {
		color: #ccc;
		cursor: not-allowed;
	}

	:global(.leaflet-control-geosearch form) {
		background: white;
		padding: 4px;
		border-radius: 4px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	:global(.leaflet-control-geosearch form input) {
		width: 100%;
		padding: 8px;
		border: 1px solid #e2e8f0;
		border-radius: 4px;
		outline: none;
	}

	:global(.leaflet-control-geosearch form input:focus) {
		border-color: #cbd5e0;
		box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.5);
	}

	:global(.leaflet-control-geosearch.bar) {
		width: 100%;
		max-width: none;
		margin: 10px 0;
	}

	:global(.leaflet-touch .leaflet-control-geosearch.bar) {
		width: 100%;
		max-width: none;
		margin: 10px 0;
	}
</style>
