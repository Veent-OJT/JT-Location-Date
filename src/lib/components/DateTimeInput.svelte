<script lang="ts">
	export let label: string;
	export let id: string;
	export let value: string;
	export let type: 'date' | 'time' = 'date';
	export let readonly = false;
	export let placeholder = '';
	export let onChange: () => void = () => {};
</script>

<div class="flex-1">
	<label for={id} class="mb-2 block font-medium">{label}</label>
	<input
		{type}
		{id}
		{placeholder}
		bind:value
		{readonly}
		on:change={onChange}
		class="w-full rounded-md border border-gray-300 p-2 text-base"
	/>
</div>
