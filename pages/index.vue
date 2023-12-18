<template>
	<div class="grid grid-cols-1 justify-center">
		<div class="card w-4/5 mx-auto">
			<h1 class="text-8xl font-bold text-center kwak-text mt-5">kwak Pot</h1>

			<div class="grid grid-cols-1 md:grid-cols-3 justify-center mt-2">
				<div class="flex flex-col items-center mx-auto mt-5">
					<h1 class="text-5xl text-white text-center font-bold sm:pt-6 md:pt-0">Total Pot:</h1>
					<h2 class="mt-3">
						<div>
							<img v-if="totalPotNumber == null" src="/loader.svg" />
							<span v-else class="mt-5 text-5xl kwak-text font-bold"
								>{{ totalPotNumber }} <span style="color: white">Elys</span></span
							>
						</div>
					</h2>
				</div>
				<div class="flex flex-col items-center mx-auto mt-5">
					<h1 class="text-5xl text-white font-bold">Your Pot:</h1>
					<h2 class="mt-3">
						<div>
							<img v-if="yourPotNumber == null" src="/loader.svg" />
							<span v-else class="mt-5 text-5xl kwak-text font-bold"
								>{{ yourPotNumber }} <span style="color: white">Elys</span></span
							>
						</div>
					</h2>
				</div>
				<div class="flex flex-col items-center mx-auto mt-5">
					<h1 class="text-5xl text-white font-bold">Chance To Win:</h1>
					<h2 class="mt-3">
						<div>
							<span class="mt-5 text-5xl kwak-text font-bold">{{ chanceToWin }}%</span>
						</div>
					</h2>
				</div>
			</div>
			<div class="grid grid-cols-1 justify-center mt-5">
				<div class="flex flex-col items-center mx-auto">
					<h1 class="text-5xl text-white font-bold">Ends In:</h1>
					<h2 class="mt-3">
						<div>
							<img v-if="endTime == null" src="/loader.svg" />
							<span v-else class="mt-5 text-5xl kwak-text font-bold">{{ endTime }}</span>
						</div>
					</h2>
				</div>
			</div>
			<div class="grid grid-cols-1 justify-center mt-5">
				<div class="flex flex-col items-center mx-auto">
					<button
						v-if="!wallet"
						@click="connectKeplr()"
						class="btn btn-md btn-warning text-xl mb-5 transform transition-all duration-200 hover:scale-105 flex flex-col items-center mx-auto"
					>
						Connect Wallet
					</button>
					<label
						v-else
						for="buy-modal"
						class="btn btn-md btn-warning text-xl mb-5 transform transition-all duration-200 hover:scale-105 flex flex-col items-center mx-auto"
					>
						Buy Tickets
					</label>
				</div>
			</div>
		</div>
	</div>

	<div class="w-4/5 mx-auto mt-5 md:grid md:grid-cols-2 md:gap-x-12">
		<div class="p-6 text-white card rounded-lg text-center">
			<h1 class="font-bold text-2xl">Your Purchases</h1>
			<UsersTransactions />
		</div>
		<div class="p-6 text-white card rounded-lg text-center">
			<h1 class="font-bold text-2xl">Recent Transactions</h1>
			<RecentTransactions />
		</div>
	</div>

	<div class="w-4/5 mx-auto mt-5 md:grid md:grid-cols-1 md:gap-x-12">
		<div class="p-6 text-white card rounded-lg text-center">
			<h1 class="font-bold text-2xl">Last Winners</h1>
			<RecentGames />
		</div>
	</div>

	<input type="checkbox" id="buy-modal" class="modal-toggle" :checked="false" v-model="buyModal" />
	<div class="modal">
		<div class="modal-box w-full h-2/4 relative flex flex-col">
			<label for="buy-modal" class="btn btn-sm absolute right-2 top-2">✕</label>
			<h3 class="text-2xl font-bold text-lg text-center">Buy Tickets</h3>
			<div class="m-2">
				<div class="row p-3">
					<div class="col-md-4 text-center">
						<input
							class="input input-bordered p-2 w-full bg-opacity-20 bg-white border border-gray-300 text-white text-4xl"
							v-model="tokenAmount"
							type="number"
							@input="updateTokenAmount"
							required
						/>
						<span class="text-3xl font-bold text-white">Elys</span>
					</div>
					<div class="col-md-4 flex items-center justify-center">
						<img class="w-14 text-white transform rotate-90 m-3" src="/arrow.svg" />
					</div>
					<div class="col-md-4 text-center">
						<input
							class="input input-bordered p-2 w-full bg-opacity-20 bg-white border border-gray-300 text-white text-4xl"
							type="number"
							disabled
							v-bind:value="(tokenAmount * gameConfig.denom) / ticketPrice"
						/>
						<span class="text-3xl font-bold text-white">Tickets</span>
					</div>
				</div>
				<div class="row">
					<div class="col-md-12">
						<div
							class="btn btn-md btn-warning text-xl mb-5 transform transition-all duration-200 hover:scale-105 flex flex-col items-center mt-5"
							@click="purchaseTickets"
						>
							Buy Tickets
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script setup>
import { useWallet, useAddress, useTransactions } from '../composables/useState';
import gameConfig from '../config/game.json';
import walletConfig from '../config/wallet.json';
import { SigningStargateClient } from '@cosmjs/stargate';

let wallet = useWallet();
let walletAddress = useAddress();
let transactions = useTransactions();

let game;
let totalPotNumber = ref(null);
let yourPotNumber = ref(null);
let chanceToWin = ref('0');
let endTime = ref(null);

// Buy stuff
let buyModal = ref(false);
let tokenAmount = ref(0.25);
let ticketPrice = ref();

onMounted(async () => {
	await loadData();
	setInterval(loadData, 10000);

	window.addEventListener('keplr_keystorechange', async () => {
		console.log('Wallet Change');
		await connectKeplr();
		await loadData();
	});
});

async function loadData() {
	game = await $fetch(`${gameConfig.api}/game`);
	transactions.value = (await $fetch(`${gameConfig.api}/transactions`)).reverse();

	totalPotNumber.value = game.totalPot / gameConfig.denom;
	ticketPrice.value = game.ticketPrice; // Fetches the current ticket price (denominated)

	let time = calculateTimeUntilBlock(game.currentBlock, game.endingBlock, gameConfig.blockTime);
	endTime.value = time;

	if (wallet.value) {
		let address = await walletAddress.value.bech32Address;

		// Fetches the users pot data
		let account = await $fetch(`${gameConfig.api}/account/${address}`);

		if (!account) {
			yourPotNumber.value = '0';
			chanceToWin.value = '0';
			return;
		}

		yourPotNumber.value = account.amount / gameConfig.denom;
		chanceToWin.value = `${((account.tickets / (game.totalPot / game.ticketPrice)) * 100)
			.toFixed(2)
			.replace(/[.,]00$/, '')}`;
	}
}

async function connectKeplr() {
	if (!window.keplr) {
		alert('Please install keplr extension');
	}

	await window.keplr.enable(walletConfig.chainId);

	let walletData = await window.keplr.getKey(walletConfig.chainId);

	wallet.value = window.keplr;

	walletAddress.value = { ...walletData };

	await loadData();
}

function calculateTimeUntilBlock(currentBlock, futureBlock, blockTimeInMilliseconds) {
	const blocksToWait = futureBlock - currentBlock;
	const timeInMilliseconds = blocksToWait * blockTimeInMilliseconds;
	const timeInSeconds = timeInMilliseconds / 1000; // Convert milliseconds to seconds

	var interval = timeInSeconds / 31536000;

	if (interval > 1) {
		return Math.floor(interval) + ' Years';
	}
	interval = timeInSeconds / 2592000;
	if (interval > 1) {
		return Math.floor(interval) + ' Months';
	}
	interval = timeInSeconds / 86400;
	if (interval > 1) {
		return Math.floor(interval) + ' Days';
	}
	interval = timeInSeconds / 3600;
	if (interval > 1) {
		return Math.floor(interval) + ' Hours';
	}
	interval = timeInSeconds / 60;
	if (interval > 1) {
		return Math.floor(interval) + ' Min';
	}
	return Math.floor(timeInSeconds) + ' Sec';
}

async function purchaseTickets() {
	const offlineSigner = window.getOfflineSigner(walletConfig.chainId);
	const accounts = await offlineSigner.getAccounts();

	const client = await SigningStargateClient.connectWithSigner(walletConfig.rpcUrl, offlineSigner);

	const sendResult = await client.sendTokens(
		accounts[0].address,
		game.address,
		[
			{
				denom: walletConfig.denom,
				amount: (tokenAmount.value * walletConfig.units).toString(),
			},
		],
		{
			amount: [{ denom: walletConfig.denom, amount: '500' }],
			gas: '200000',
		}
	);

	// Indexs the transaction in the game server
	let index = await $fetch(`${gameConfig.api}/index/${sendResult.transactionHash}`);

	buyModal.value = false;
}
</script>

<style scoped>
.section {
	margin-top: 125px;
}
.kwak-text {
	color: #e1b723;
}
.card {
	background-color: oklch(0.278078 0.029596 256.847952/1);
}
</style>
