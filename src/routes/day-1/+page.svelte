<script lang="ts">
	import { derived, writable, type Readable } from 'svelte/store';
	import Icon from '@iconify/svelte';
	import { onMount } from 'svelte';

	type Child = {
		name: string;
		tally: number;
	};

	let childList = writable<Child[]>([]);
	let tableHeader = writable<string[]>([]);
	let sortable = writable<{ type: string; value: boolean }>();
	let search = writable<string>('');

	let filterList: Readable<Child[]> = derived(
		[search, childList],
		([$val, $child]: [string, Child[]], set: Function) => {
			let timeoutId = setTimeout(() => {
				const lowercasedSearchTerm = $val.toLowerCase().trim();
				const filter = $childList.filter(
					(child) =>
						child.name.includes(lowercasedSearchTerm) ||
						child.tally.toString().includes(lowercasedSearchTerm)
				);
				set(filter || $child);
			}, 0);
			return () => {
				clearTimeout(timeoutId);
			};
		},
		[]
	);

	onMount(async () => {
		const response = await fetch('https://advent.sveltesociety.dev/data/2023/day-one.json');
		if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
		$childList = await response.json();
		if ($childList.length > 0) $tableHeader = Object.keys($childList[0]);
	});

	const sort = (header: string) => {
		if ($sortable?.type === header) {
			$sortable.value = !$sortable.value;
		} else {
			$sortable = { type: header, value: false };
		}
	};

	$: if ($sortable) {
		if ($sortable.type === 'name') {
			if ($sortable.value === true) {
				$childList = $childList.sort((a, b) => b.name.localeCompare(a.name));
			} else {
				$childList = $childList.sort((a, b) => a.name.localeCompare(b.name));
			}
		} else if ($sortable.type === 'tally') {
			if ($sortable.value === true) {
				$childList = $childList.sort((a, b) => a.tally - b.tally);
			} else {
				$childList = $childList.sort((a, b) => b.tally - a.tally);
			}
		}
	}
</script>

<div class="flex flex-col justify-center items-center align-middle h-screen m-auto">
	<input
		type="text"
		bind:value={$search}
		placeholder="Search ..."
		class="input input-ghost w-full max-w-[400px] my-3"
	/>
	<div class="flex justify-center md:max-h-[60dvh] overflow-auto">
		<table class="table table-pin-rows table-pin-cols min-w-[300px] max-h-[600px] max-w-[400px]">
			<thead>
				<tr>
					{#each $tableHeader as header, i (header)}
						<th>
							<div class="flex justify-center">
								<button
									class="flex justify-center items-center align-middle gap-2"
									type="button"
									on:click={() => sort(header)}
								>
									{header?.toUpperCase()}
									{#if $sortable?.type === header}
										{#if $sortable?.value}
											<Icon icon="mdi:arrow-up" />
										{:else}
											<Icon icon="mdi:arrow-down" />
										{/if}
									{/if}
								</button>
							</div>
						</th>
					{/each}
				</tr>
			</thead>
			<tbody>
				{#each $filterList as dear, i (dear)}
					<tr>
						<td class="text-center">{dear.name}</td>
						<td class="text-end">{dear.tally}</td>
					</tr>
				{/each}
				{#if $filterList.length === 0}
					<tr>
						<td class="text-center" colspan="2"> No data to display </td>
					</tr>
				{/if}
			</tbody>
		</table>
	</div>
</div>
