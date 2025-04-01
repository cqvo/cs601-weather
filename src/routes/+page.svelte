<script lang="ts">
	import { Popover } from '@skeletonlabs/skeleton-svelte';
	import IconX from '@lucide/svelte/icons/x';

	let openState = $state(false);
	let zipHistory = $state(new Set());
	let selectedZip = $state('');
	let forecastToday = $state();
	let forecastFiveDay = $state();

	const popoverClose = () => {
		openState = false;
	};
	const submitHandler = async () => {
		forecastToday = await fetchForecastToday(selectedZip);
		forecastFiveDay = await fetchForecastFiveDay(selectedZip);
		zipHistory.add(selectedZip);
	};
	const historyHandler = async (zip: string) => {
		selectedZip = zip;
		submitHandler();
		popoverClose();
	};
	$inspect(forecastFiveDay);

	const API_KEY = '7251bbce56845e24f83613a84ac285b4';
	const fetchForecastToday = async (zip: string) => {
		const response = await fetch(
			`https://api.openweathermap.org/data/2.5/weather?zip=${zip},US&appid=${API_KEY}&units=imperial`
		);
		return response.json();
	};
	const fetchForecastFiveDay = async (zip: string) => {
		const response = await fetch(
			`https://api.openweathermap.org/data/2.5/forecast?zip=${zip},US&appid=${API_KEY}&units=imperial`
		);
		return response.json();
	};
</script>

<main class="space-y-4 p-4">
	<div class="mx-auto w-3/4">
		<label class="label">
			<span class="label-text">ZIP Code</span>
			<input class="input" type="text" bind:value={selectedZip} />
		</label>
	</div>
	<div class="flex justify-evenly">
		<button class="btn preset-filled" onclick={submitHandler}>Search</button>
		<Popover
			open={openState}
			onOpenChange={(e) => (openState = e.open)}
			positioning={{ placement: 'top' }}
			triggerBase="btn preset-tonal"
			contentBase="card bg-surface-200-800 p-4 space-y-4 max-w-[320px]"
			arrow
			arrowBackground="!bg-surface-200 dark:!bg-surface-800"
		>
			{#snippet trigger()}History{/snippet}
			{#snippet content()}
				<header class="flex justify-between">
					<p class="text-xl font-bold">ZIP Codes</p>
					<button class="btn-icon hover:preset-tonal" onclick={popoverClose}><IconX /></button>
				</header>
				<article>
					{#each zipHistory as zip}
						<p class="opacity-60">
							<button
								class="anchor"
								onclick={(e) => {
									historyHandler(e.target.innerText);
								}}>{zip}</button
							>
						</p>
					{/each}
				</article>
			{/snippet}
		</Popover>
	</div>
</main>
{#if forecastToday}
	<div
		class="card preset-filled-surface-50-950 border-surface-200-800 card-hover divide-surface-200-800 block max-w-full divide-y overflow-hidden border-[1px]"
	>
		<header>
			<div>
				<h2 class="h6">Forecast for</h2>
				<h3 class="h3">{forecastToday.name}</h3>
			</div>
		</header>
		<article class="space-y-4 p-4">
			<section>
				<h2 class="h6">Today</h2>
				<h3 class="h3">{new Date(forecastToday.dt * 1000).toLocaleDateString()}</h3>
			</section>

			<section class="flex flex-wrap content-start justify-start">
				<div class="content-center">
					<img
						src="https://openweathermap.org/img/wn/{forecastToday.weather[0].icon}@2x.png"
						alt={forecastToday.weather[0].description}
					/>
				</div>
				<div class="content-center">
					<p class="opacity-60">
						{forecastToday.weather[0].description}, feels like {forecastToday.main.feels_like}
					</p>
					<p class="opacity-60">
						High: {forecastToday.main.temp_max}, Low: {forecastToday.main.temp_min}
					</p>
				</div>
			</section>
		</article>
		{#if forecastFiveDay && forecastFiveDay.city}
			<article class="space-y-4 p-4">
				<div>
					<h3 class="h3">5-Day Forecast</h3>
				</div>
				{#each forecastFiveDay.list as item}
					<h6 class="h6">{item.dt_txt}</h6>
					<section class="flex flex-wrap content-start justify-start">
						<div class="content-center">
							<img
								src="https://openweathermap.org/img/wn/{item.weather[0].icon}.png"
								alt={forecastToday.weather[0].description}
							/>
						</div>
						<div class="content-center">
							<p class="opacity-60">
								{item.weather[0].description}, feels like {item.main.feels_like}
							</p>
							<p class="opacity-60">High: {item.main.temp_max}, Low: {item.main.temp_min}</p>
						</div>
					</section>
				{/each}
			</article>
		{/if}
	</div>{/if}
