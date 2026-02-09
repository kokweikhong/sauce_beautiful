<script lang="ts">
    import { onMount } from 'svelte';

    type Item = {
        name: string;
        value: number;
        amountInCents: number;
    };
    
    const defaultItems: Item[] = [
        { name: "猪肠粉", value: 0, amountInCents: 120 },
        { name: "料130", value: 0, amountInCents: 130 },
        { name: "料180", value: 0, amountInCents: 180 },
        { name: "料220", value: 0, amountInCents: 220 },
        { name: "料250", value: 0, amountInCents: 250 },
        { name: "咖喱", value: 0, amountInCents: 100 },
        { name: "咖喱猪皮", value: 0, amountInCents: 200 },
        { name: "香菇肉碎", value: 0, amountInCents: 450 },
        { name: "甜酱", value: 0, amountInCents: 0 },
        { name: "红酱", value: 0, amountInCents: 0 },
        { name: "打包", value: 0, amountInCents: 50 },
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
            items: defaultItems.map(item => ({ ...item })),
            totalAmountInCents: 0,
        };
        orderRecords = [orderData, ...orderRecords];
        saveOrderRecords();
        return orderData;
    }

    function saveOrderRecords() {
        localStorage.setItem('orderRecords', JSON.stringify(orderRecords));
    }

    function deleteOrderRecord(date: string) {
        orderRecords = orderRecords.filter(record => record.date !== date);
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

<div class="min-h-screen bg-gray-100 p-8 max-w-md mx-auto w-full">
        <h1 class="text-4xl font-bold text-gray-900 mb-8">Order Records</h1>
        <div class="bg-white rounded-lg shadow-md p-4 sm:p-6 mb-8">
            <label for="newDate" class="block text-gray-700 font-semibold mb-4">Add New Record</label>
            <div class="flex flex-col sm:flex-row gap-2 sm:gap-4">
            <button 
                on:click={() => {
                const date = new Date(newDate);
                date.setDate(date.getDate() - 1);
                newDate = date.toISOString().split('T')[0];
                }}
                class="bg-gray-500 hover:bg-gray-600 text-white font-semibold py-2 px-3 sm:px-4 rounded transition text-sm sm:text-base"
            >
                ← Prev
            </button>
            <input 
                id="newDate"
                type="date" 
                bind:value={newDate}
                class="flex-1 border border-gray-300 rounded px-3 sm:px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500 text-sm sm:text-base"
            />
            <button 
                on:click={() => {
                const date = new Date(newDate);
                date.setDate(date.getDate() + 1);
                newDate = date.toISOString().split('T')[0];
                }}
                class="bg-gray-500 hover:bg-gray-600 text-white font-semibold py-2 px-3 sm:px-4 rounded transition text-sm sm:text-base"
            >
                Next →
            </button>
            <button 
                on:click={() => createNewOrderDate(newDate)}
                class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 sm:px-6 rounded transition text-sm sm:text-base"
            >
                Add Record
            </button>
            </div>
        </div>

        {#if orderRecords.length === 0}
            <p class="text-gray-600 text-lg">No records found.</p>
        {:else}
            <div class="space-y-6 mb-8">
                {#each orderRecords as record (record.date)}
                    <div class="bg-white rounded-lg shadow-md p-6">
                        <h2 class="text-2xl font-semibold text-gray-800 mb-4">{record.date}</h2>
                        <ul class="space-y-2 mb-4">
                            {#each record.items as item}
                                <li class="flex justify-between items-center text-gray-700">
                                    <span>{item.name}</span>
                                    <div class="flex items-center gap-3">
                                        <span>{item.value} (RM{(item.amountInCents / 100).toFixed(2)})</span>
                                        <button 
                                            on:click={() => {
                                                item.value--;
                                                record.totalAmountInCents -= item.amountInCents;
                                                saveOrderRecords();
                                            }}
                                            class="bg-red-500 hover:bg-red-600 text-white font-semibold py-1 px-2 rounded"
                                        >
                                            −
                                        </button>
                                        <button 
                                            on:click={() => {
                                                item.value++;
                                                record.totalAmountInCents += item.amountInCents;
                                                saveOrderRecords();
                                            }}
                                            class="bg-green-500 hover:bg-green-600 text-white font-semibold py-1 px-2 rounded"
                                        >
                                            +
                                        </button>
                                    </div>
                                </li>
                            {/each}
                        </ul>
                        <div class="border-t pt-4 mb-4">
                            <p class="text-lg font-semibold text-gray-900">Total: RM{(record.totalAmountInCents / 100).toFixed(2)}</p>
                        </div>
                        <button 
                            on:click={() => deleteOrderRecord(record.date)}
                            class="bg-red-500 hover:bg-red-600 text-white font-semibold py-2 px-4 rounded transition"
                        >
                            Delete
                        </button>
                    </div>
                {/each}
            </div>
        {/if}
</div>