<script lang="ts">
	import { onMount } from 'svelte';

	type Item = {
		name: string;
		value: number;
		amountInCents: number;
	};

	const defaultItems: Item[] = [
		{ name: '猪肠粉', value: 0, amountInCents: 120 },
		{ name: '料130', value: 0, amountInCents: 130 },
		{ name: '料180', value: 0, amountInCents: 180 },
		{ name: '料220', value: 0, amountInCents: 220 },
		{ name: '料250', value: 0, amountInCents: 250 },
		{ name: '咖喱', value: 0, amountInCents: 100 },
		{ name: '咖喱猪皮', value: 0, amountInCents: 200 },
		{ name: '香菇肉碎', value: 0, amountInCents: 450 },
		{ name: '甜酱', value: 0, amountInCents: 0 },
		{ name: '红酱', value: 0, amountInCents: 0 },
		{ name: '打包', value: 0, amountInCents: 50 }
	];

	type OrderRecord = {
		date: string;
		items: Item[];
		totalAmountInCents: number;
	};

	let orderRecords: OrderRecord[] = [];
	let newDate: string = '';

	function createNewOrderDate(date: string): OrderRecord {
		const orderData = {
			date,
			items: defaultItems.map((item) => ({ ...item })),
			totalAmountInCents: 0
		};
		orderRecords = [orderData, ...orderRecords];
		saveOrderRecords();
		return orderData;
	}

	function saveOrderRecords() {
		localStorage.setItem('orderRecords', JSON.stringify(orderRecords));
	}

	function deleteOrderRecord(date: string) {
		orderRecords = orderRecords.filter((record) => record.date !== date);
		saveOrderRecords();
	}

	function loadOrderRecords() {
		const data = localStorage.getItem('orderRecords');
		if (data) {
			orderRecords = JSON.parse(data);
		}
	}

	onMount(() => {
		loadOrderRecords();
	});
</script>

<div class="mx-auto min-h-screen max-w-md bg-gray-100 p-8">
	<h1 class="mb-8 text-4xl font-bold text-gray-900">Order Records</h1>
	<div class="mb-8 rounded-lg bg-white p-4 shadow-md sm:p-6">
		<label for="newDate" class="mb-4 block font-semibold text-gray-700">Add New Record</label>
		<div class="flex flex-col gap-2">
			<button
				on:click={() => {
					const date = new Date(newDate);
					date.setDate(date.getDate() - 1);
					newDate = date.toISOString().split('T')[0];
				}}
				class="rounded bg-gray-500 px-3 py-2 text-sm font-semibold text-white transition hover:bg-gray-600 sm:px-4 sm:text-base"
			>
				← Prev
			</button>
			<input
				id="newDate"
				type="date"
				bind:value={newDate}
				class="flex-1 rounded border border-gray-300 px-3 py-2 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none sm:px-4 sm:text-base"
			/>
			<button
				on:click={() => {
					const date = new Date(newDate);
					date.setDate(date.getDate() + 1);
					newDate = date.toISOString().split('T')[0];
				}}
				class="rounded bg-gray-500 px-3 py-2 text-sm font-semibold text-white transition hover:bg-gray-600 sm:px-4 sm:text-base"
			>
				Next →
			</button>
			<button
				on:click={() => createNewOrderDate(newDate)}
				class="rounded bg-blue-500 px-4 py-2 text-sm font-semibold text-white transition hover:bg-blue-600 sm:px-6 sm:text-base"
			>
				Add Record
			</button>
		</div>
	</div>

	{#if orderRecords.length === 0}
		<p class="text-lg text-gray-600">No records found.</p>
	{:else}
		<div class="mb-8 space-y-6">
			{#each orderRecords as record (record.date)}
				<div class="rounded-lg bg-white p-6 shadow-md">
					<h2 class="mb-4 text-2xl font-semibold text-gray-800">{record.date}</h2>
					<ul class="mb-4 space-y-2">
						{#each record.items as item}
							<li class="flex items-center justify-between text-gray-700">
								<span>{item.name}</span>
								<div class="flex items-center gap-3">
									<span>{item.value} (RM{(item.amountInCents / 100).toFixed(2)})</span>
									<button
										on:click={() => {
											item.value--;
											record.totalAmountInCents -= item.amountInCents;
											saveOrderRecords();
										}}
										class="rounded bg-red-500 px-2 py-1 font-semibold text-white hover:bg-red-600"
									>
										−
									</button>
									<button
										on:click={() => {
											item.value++;
											record.totalAmountInCents += item.amountInCents;
											saveOrderRecords();
										}}
										class="rounded bg-green-500 px-2 py-1 font-semibold text-white hover:bg-green-600"
									>
										+
									</button>
								</div>
							</li>
						{/each}
					</ul>
					<div class="mb-4 border-t pt-4">
						<p class="text-lg font-semibold text-gray-900">
							Total: RM{(record.totalAmountInCents / 100).toFixed(2)}
						</p>
					</div>
					<button
						on:click={() => deleteOrderRecord(record.date)}
						class="rounded bg-red-500 px-4 py-2 font-semibold text-white transition hover:bg-red-600"
					>
						Delete
					</button>
				</div>
			{/each}
		</div>
	{/if}
</div>
