<script>
	// ESM (mjs)
	import Chart from 'chart.js/auto';

	import { onMount } from 'svelte';
	let chartElement;
	let chart;

	export let from;
	export let to;

	export let data = [
		{ x: '2016-12-25', y: 20 },
		{ x: '2016-12-26', y: 10 }
	];

	$: if (chart && from && to) {
		console.log(from, to);
		chart.data.datasets[0].data = data;
		chart.update();
	}

	onMount(() => {
		console.log(data);
		chart = new Chart(chartElement, {
			type: 'line',
			data: {
				datasets: [
					{
						label: 'Daily Cumulative # of Reviews',
						data: data,
						fill: true,
						backgroundColor: 'rgba(255, 99, 132, 0.2)',
						borderColor: 'rgb(255, 99, 132)',
						pointBackgroundColor: 'rgb(255, 99, 132)',
						pointBorderColor: '#fff',
						pointHoverBackgroundColor: '#fff',
						pointHoverBorderColor: 'rgb(255, 99, 132)'
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false,
				plugins: {
					legend: {
						position: 'bottom'
					}
				}
			}
		});

		// window.addEventListener('beforeprint', () => {
		// 	ctx.resize(600, 600);
		// });
		// window.addEventListener('afterprint', () => {
		// 	ctx.resize();
		// });
	});
</script>

<div class="  h-40">
	<canvas bind:this={chartElement} />
</div>
