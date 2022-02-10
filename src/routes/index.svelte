<script>
	import { onMount } from 'svelte';
	import getUAuth from '$lib/uauth';
	import Arweave from 'arweave';

	// Thoughts publish as blogs, send, save, remind in future.

	let userPromise, ar;

	onMount(() => {
		userPromise = getUAuth(window).then(async (uauth) => uauth.user());
		ar = Arweave.init({});
	});

	const handleLogin = () => {
		getUAuth(window)
			.then(async (uauth) => {
				await uauth.loginWithPopup();
				userPromise = uauth.user();
			})
			.catch((error) => {
				console.error(error);
				alert(String(error));
			});
	};

	const handleLogout = () => {
		getUAuth(window)
			.then(async (uauth) => {
				await uauth.logout({ rpInitiatedLogout: false });
				userPromise = Promise.reject();
			})
			.catch((error) => {
				console.error(error);
				alert(String(error));
			});
	};

	const addThought = async (data, addr) => {
		const tx = await ar.createTransaction({ data: data }, import.meta.env.VITE_ARWEAVE_KEY);
		tx.addTag('App-Name', 'memoir');
		tx.addTag('App-Version', '0.0.1');
		tx.addTag('Content-Type', 'application/json');
		tx.addTag('User', addr);
		await ar.transactions.sign(tx, import.meta.env.VITE_ARWEAVE_KEY);
		const uploader = await ar.transactions.getUploader(tx);
		while (!uploader.isComplete) {
			await uploader.uploadChunk();
			console.log(
				`${uploader.pctComplete}% complete, ${uploader.uploadedChunks}/${uploader.totalChunks}`
			);
		}
	};
</script>

<svelte:head>
	<title>Home</title>
</svelte:head>

{#await userPromise then user}
	<pre>{JSON.stringify(user, null, 2)}</pre>
	<button on:click={handleLogout}>Logout</button>
{:catch error}
	<button on:click={handleLogin}>Login with Unstoppable</button>
{/await}

Writings

<style>
</style>
