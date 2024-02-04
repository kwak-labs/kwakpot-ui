<template>
	<div class="max-h-[200px] overflow-auto border border-1 border-gray-500 rounded-lg mt-2">
		<table class="min-w-full divide-y divide-gray-200">
			<thead class="bg-gray-900">
				<tr>
					<th class="px-6 py-3 text-left border-r border-gray-500">Winner</th>
					<th class="px-6 py-3 text-left border-r border-gray-500">Winnings</th>
					<th class="px-6 py-3 text-left border-r border-gray-500">Chance</th>
					<th class="px-6 py-3 text-left border-r border-gray-500">Date</th>
				</tr>
			</thead>
			<tbody class="divide-y divide-gray-900">
				<tr v-for="game in allGames" :key="game.Winner">
					<td class="px-6 py-4 whitespace-nowrap border-r border-gray-500">
						{{ game.Winner }}
					</td>
					<td class="px-6 py-4 whitespace-nowrap border-r border-gray-500">
						{{ game.Winnings }} Elys
					</td>
					<td class="px-6 py-4 whitespace-nowrap border-r border-gray-500">
						{{ game.Chance.toFixed(2).replace(/[.,]00$/, '') }}%
					</td>
					<td class="px-6 py-4 whitespace-nowrap border-r border-gray-500">
						{{ timeSince(game.Date) }}
					</td>
				</tr>
			</tbody>
		</table>
	</div>
</template>

<script setup>
import gameConfig from '../config/game.json';
import Arweave from 'arweave';

const arweave = Arweave.init({
	host: 'ar-io.net',
	port: 443,
	protocol: 'https',
	logging: false,
});

let allGames = ref([]);

onMounted(async () => {
	await loadGames();
	setInterval(async () => {
		await loadGames();
	}, 120000);
});

async function loadGames() {
	allGames.value = [];
	let games = await (
		await $fetch(gameConfig.arweaveGateway + '/graphql', {
			body: JSON.stringify({
				query: `query {
  transactions(owners:["${gameConfig.arweaveAddress}"],sort:HEIGHT_ASC, first:100,tags:[{
  name: "kwakpot-game",
  values: ["${gameConfig.queryTag}"]
}]) {
        edges {
            node {
                id
            }
        }
    }
}
`,
			}),
			method: 'POST',
			headers: { 'Content-type': 'application/json' },
		})
	).data.transactions.edges;

	games = games.reverse();

	for (let game of games) {
		let txId = game.node.id;

		let data;

		try {
			data = JSON.parse(
				await arweave.transactions.getData(txId, {
					decode: true,
					string: true,
				})
			);

			console.log(data.date);

			let Winner = data.winner.address;
			let Chance = data.winner.chanceToWin;
			let Date = data.date;
			let Winnings = data.totalPot / gameConfig.denom;

			allGames.value.push({
				Winner,
				Chance,
				Winnings,
				Date,
			});
			// console.log(allGames);
		} catch (e) {
			console.log(e);
		}
	}
}

function timeSince(date) {
	var seconds = Math.floor((new Date() - new Date(date)) / 1000);

	var interval = seconds / 31536000;

	if (interval > 1) {
		return Math.floor(interval) + ' years ago';
	}
	interval = seconds / 2592000;
	if (interval > 1) {
		return Math.floor(interval) + ' months ago';
	}
	interval = seconds / 86400;
	if (interval > 1) {
		return Math.floor(interval) + ' days ago';
	}
	interval = seconds / 3600;
	if (interval > 1) {
		return Math.floor(interval) + ' hours ago';
	}
	interval = seconds / 60;
	if (interval > 1) {
		return Math.floor(interval) + ' minutes ago';
	}
	return Math.floor(seconds) + ' seconds ago';
}
</script>

<style scoped>
.table-col {
	max-height: 200px; /* Adjust this height to your preference */
	overflow-y: auto;
}
</style>
