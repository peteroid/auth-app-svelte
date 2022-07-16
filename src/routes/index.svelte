<script>
	import { Clipboard, Eye, EyeOff, Icon, Key, Link, LockOpen, Pencil } from 'svelte-hero-icons';
	import totp from 'totp-generator';
	import { onDestroy, onMount } from 'svelte';
	import { page } from '$app/stores';

	let secret = '';
	let tokenInput;
	let linkInput;
	let hasInitSecret = false;
	let messages = [];
	let refreshTokenTimer;
	let tokenRefreshRemain = 0;
	let isEditingSecret = false;

	onMount(() => {
		secret =
			$page.url.searchParams.get('secret') || localStorage.getItem('auth_app:last_secret') || '';
		hasInitSecret = true;
		isEditingSecret = !secret;
		refreshToken();
	});

	onDestroy(() => {
		if (refreshTokenTimer) clearTimeout(refreshTokenTimer);
	});

	const refreshToken = () => {
		tokenRefreshRemain = 30 - (new Date().getSeconds() % 30);
		refreshTokenTimer = setTimeout(refreshToken, 1000);
	};

	let token = '';
	let link = '';
	const calculateToken = () => {
		try {
			token = totp(secret);
			link = typeof window !== 'undefined' ? `${window.origin}/?secret=${secret}` : '';
			if (hasInitSecret) localStorage.setItem('auth_app:last_secret', secret);
		} catch {
			token = 'Invalid';
		}
	};

	$: tokenRefreshRemain, calculateToken();

	const toast = (message) => {
		messages = [...messages, message];

		setTimeout(() => {
			const toRemove = messages.indexOf(message);
			messages = messages.filter((m, i) => i !== toRemove);
		}, 3000);
	};

	const copyToken = () => {
		if (!tokenInput) return;
		tokenInput.select();
		tokenInput.setSelectionRange(0, 99999); /* For mobile devices */
		navigator.clipboard.writeText(tokenInput.value);
		toast('Copied the token 🔑');
	};

	const copyLink = () => {
		if (!tokenInput) return;
		linkInput.select();
		linkInput.setSelectionRange(0, 99999); /* For mobile devices */
		navigator.clipboard.writeText(linkInput.value);
		toast('Copied the link 🔗');
	};

	const onClickToken = () => {
		copyToken();
	};

	const onClickLink = () => {
		copyLink();
	};

	const onClickSecret = () => {
		isEditingSecret = !isEditingSecret;
	};
</script>

<div
	class="absolute inset-0 flex items-center justify-center from-gray-900 to-gray-600 bg-gradient-to-br p-8"
>
	<div class="card w-96 shadow-2xl bg-base-100">
		<figure>
			<div style="width:100%;height:0;padding-bottom:54%;position:relative;">
				<iframe
					src="https://giphy.com/embed/81xwEHX23zhvy"
					width="100%"
					height="100%"
					style="position:absolute"
					frameBorder="0"
					class="giphy-embed"
					allowFullScreen
					title="gif"
				/>
			</div>
		</figure>
		<div class="" />

		<div class="card-body">
			<h2 class="card-title mb-4">Auth 🔐</h2>

			<div class="flex flex-col gap-6 mb-6">
				<div class="form-control">
					<label class="input-group">
						<span>
							<Icon src={LockOpen} class="w-6" />
						</span>
						{#if !isEditingSecret}
							<input
								readonly
								type="password"
								placeholder="Secret"
								class="input input-bordered input-md w-full"
								value={secret}
							/>
						{:else}
							<input
								type="text"
								placeholder="Secret"
								class="input input-bordered input-md w-full"
								bind:value={secret}
							/>
						{/if}
						<button class="btn btn-outline btn-accent" on:click={onClickSecret}>
							<Icon src={isEditingSecret ? EyeOff : Pencil} class="w-5" />
						</button>
					</label>
				</div>
				<div class="form-control">
					<label class="input-group">
						<span>
							<Icon src={Key} class="w-6" />
						</span>
						<div class="w-full relative">
							<input
								readonly
								type="text"
								placeholder="Token"
								class="input input-bordered !rounded-none input-md w-full input-disabled cursor-pointer"
								bind:this={tokenInput}
								on:click={onClickToken}
								value={token}
							/>
							<div class="h-0.5 absolute bottom-0 left-0 right-0 rounded-full">
								<div
									class="bg-accent h-full transition-transform origin-left"
									style={`transform:scaleX(${Math.min(
										100,
										Math.floor(((tokenRefreshRemain - 1) / 30) * 100)
									)}%)`}
								/>
							</div>
						</div>
						<button class="btn btn-primary" on:click={onClickToken}>
							<Icon src={Clipboard} class="w-5" />
						</button>
					</label>
				</div>
				<div class="form-control">
					<label class="input-group">
						<span>
							<Icon src={Link} class="w-6" />
						</span>

						<input
							readonly
							type="text"
							placeholder="Link"
							class="input input-bordered input-md w-full input-disabled cursor-pointer"
							bind:this={linkInput}
							on:click={onClickLink}
							value={link}
						/>
						<button class="btn btn-outline btn-secondary" on:click={onClickLink}>
							<Icon src={Clipboard} class="w-5" /></button
						>
					</label>
				</div>
			</div>

			<!-- <div class="card-actions justify-end">
				<button class="btn btn-primary" on:click={onClickLink}>?</button>
			</div> -->
		</div>
	</div>

	<div class="toast">
		{#each messages as message}
			<div class="alert alert-info">
				<div>
					<span>{message}</span>
				</div>
			</div>
		{/each}
	</div>
</div>
