<script lang="ts">
	$: coordinates = {
		lat: 8.4827794,
		lng: 124.6560661
	};

	const getMapUrl = (lat: number, lng: number) => {
		return `https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d500!2d${lng}!3d${lat}!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x0!2zOMKwMjgnNTguMCJOIDEyNMKwMzknMjEuOCJF!5e0!3m2!1sen!2sph!4v1739860371902!5m2!1sen!2sph`;
	};
</script>

<div class="h-screen w-full">
	<iframe
		width="100%"
		height="100%"
		src={getMapUrl(coordinates.lat, coordinates.lng)}
		style="border:0"
		loading="lazy"
		referrerpolicy="no-referrer-when-downgrade"
		title="map"
	></iframe>
</div>
