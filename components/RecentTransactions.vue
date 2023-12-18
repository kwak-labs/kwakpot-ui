<template>
	<div class="max-h-[200px] overflow-auto border border-1 border-gray-500 rounded-lg mt-2">
		<table class="min-w-full divide-y divide-gray-200">
			<thead class="bg-gray-900">
				<tr>
					<th class="px-6 py-3 text-left border-r border-gray-500">TxId</th>
					<th class="px-6 py-3 text-left">Tickets Purchased</th>
				</tr>
			</thead>
			<tbody class="divide-y divide-gray-900">
				<tr v-for="transaction in transactions" :key="transaction.tx">
					<td class="px-6 py-4 whitespace-nowrap border-r border-gray-5z00">
						{{ transaction.tx.substring(0, 40) }}...{{
							transaction.tx.substring(transaction.tx.length - 4)
						}}
					</td>
					<td class="px-6 py-4 whitespace-nowrap">{{ transaction.tickets }}</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<script setup>
import { useTransactions } from '../composables/useState';

let transactions = useTransactions();
</script>

<style scoped>
.card {
	background-color: #333;
}
</style>
