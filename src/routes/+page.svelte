<script>
	import dayjs from 'dayjs';
	import * as d3 from 'd3';

	import { onMount } from 'svelte';

	import LineChart from '$lib/components/LineChart.svelte';
	import P5 from 'p5-svelte';

	let width = 700;
	let height = 700;

	let reviewsData = [];
	let reviews = reviewsData.sort((a, b) => a.unixReviewTime - b.unixReviewTime);
	let selectedReviews = reviews;

	let fromValue = 0;
	let toValue = 0;

	const groupBy = (items, key) =>
		items.reduce(
			(result, item) => ({
				...result,
				[item[key]]: [...(result[item[key]] || []), item]
			}),
			{}
		);

	let graph = {
		all: {
			parentId: null,
			id: 'all',
			title: 'All Amazon Products',

			x: width * 0.5,
			y: height - 150,
			color: '#000',
			value: 5000
		},
		lifestyle: {
			parentId: 'all',
			id: 'lifestyle',
			title: 'Lifestyle',

			position: 'left',
			color: '#08316C',
			value: 3000,

			x: width * 0.4,
			y: height - 300
		},
		technical: {
			parentId: 'all',
			id: 'technical',
			title: 'Technical',

			x: width * 0.6,
			y: height - 300,
			color: '#A50F15',
			value: 2000
		},
		generalbeauty: {
			parentId: 'lifestyle',
			id: 'generalbeauty',
			title: 'General Beauty',

			x: width * 0.34,
			y: height - 450,
			color: '#08519C',
			value: 2000
		},
		fashion: {
			parentId: 'lifestyle',
			id: 'fashion',
			title: 'Fashion',
			x: width * 0.46,
			y: height - 450,
			color: '#2171B5',
			value: 1000
		},
		appliances: {
			parentId: 'technical',
			id: 'appliances',
			title: 'Appliances',

			x: width * 0.55,
			y: height - 450,
			color: '#CB181D',
			value: 1000
		},
		industrial: {
			parentId: 'technical',
			id: 'industrial',
			title: 'Industrial',

			x: width * 0.65,
			y: height - 450,
			color: '#EF3B2C',
			value: 1000
		},
		beauty: {
			parentId: 'generalbeauty',
			id: 'beauty',
			position: 'left',
			title: 'Beauty',

			x: width * 0.3,
			y: height - 600,
			color: '#6BAED6',
			value: 1700
		},
		luxbeauty: {
			parentId: 'generalbeauty',
			id: 8,
			title: 'Luxury Beauty',

			x: width * 0.4,
			y: height - 600,
			color: '#C6DBEF',
			value: 300
		}
	};

	$: if (fromValue && toValue) {
		selectedReviews = reviews.filter(
			(review) => review.unixReviewTime >= fromValue && review.unixReviewTime <= toValue
		);

		let groupByCategory = selectedReviews.reduce((result, item) => {
			const groupKey = item['category'];

			// Check if the groupKey already exists in the result object
			if (!result[groupKey]) {
				result[groupKey] = 1;
			} else {
				result[groupKey]++;
			}

			return result;
		}, {});

		for (const category of Object.keys(groupByCategory)) {
			graph[category].value = groupByCategory[category] ?? 0;
		}

		graph['generalbeauty'].value = graph['beauty'].value + graph['luxbeauty'].value;
		graph['lifestyle'].value = graph['generalbeauty'].value + graph['fashion'].value;
		graph['technical'].value = graph['appliances'].value + graph['industrial'].value;

		graph['all'].value = graph['lifestyle'].value + graph['technical'].value;
	}

	const getBezierCoords = (x1, y1, x2, y2) => {
		// Calculate control points to create a slight Bezier curve
		const deltaX = x2 - x1;
		const deltaY = y2 - y1;

		const x3 = x1 + deltaX * 0.5;
		const y3 = y1 + deltaY * 0.1;

		const x4 = x2 - deltaX * 0.5;
		const y4 = y2 - deltaY * 0.1;

		// Return the coordinates for the Bezier curve
		return [x1, y1, x3, y3, x4, y4, x2, y2];
	};

	const sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(width, height);

			for (let nodeIdx = Object.keys(graph).length - 1; nodeIdx >= 0; nodeIdx--) {
				let node = Object.keys(graph).at(nodeIdx);
				// p5.fill(graph[node].color);
				p5.strokeWeight(5);

				p5.stroke(graph[node].color);
				if (nodeIdx == 0) {
					p5.rect(
						graph[node].x - graph[node].value / 500,
						graph[node].y,
						graph[node].value / 250,
						130
					);
				}

				p5.strokeWeight(10);
				p5.line(
					graph[node].x - graph[node].value / 500,
					graph[node].y,
					graph[node].x - graph[node].value / 500 + graph[node].value / 250,
					graph[node].y
				);

				if (graph[graph[node].parentId] ?? false) {
					p5.stroke(graph[node].color);
					p5.strokeWeight(5);

					if (nodeIdx !== 0 && nodeIdx % 2 == 0) {
						let [x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x - graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x +
								graph[graph[node].parentId].value / 500 -
								graph[node].value / 250,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);

						[x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x + graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x + graph[graph[node].parentId].value / 500,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);
					} else {
						let [x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x - graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x - graph[graph[node].parentId].value / 500,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);

						[x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x + graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x -
								graph[graph[node].parentId].value / 500 +
								graph[node].value / 250,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);
					}
				}

				p5.noStroke();
				// p5.textAlign(p5.CENTER, p5.CENTER);
				p5.textStyle(p5.BOLD);

				p5.text(graph[node].title, graph[node].x - graph[node].value / 60, graph[node].y + 20);
				p5.textStyle(p5.BOLDITALIC);
				p5.text(
					`(${graph[node].value.toLocaleString()})`,
					graph[node].x - graph[node].value / 600,
					graph[node].y + 35
				);
			}
		};

		p5.draw = () => {
			p5.background(255);
			for (let nodeIdx = Object.keys(graph).length - 1; nodeIdx >= 0; nodeIdx--) {
				let node = Object.keys(graph).at(nodeIdx);
				// p5.fill(graph[node].color);
				p5.strokeWeight(5);

				p5.stroke(graph[node].color);
				if (nodeIdx == 0) {
					p5.rect(
						graph[node].x - graph[node].value / 500,
						graph[node].y,
						graph[node].value / 250,
						130
					);
				}

				p5.strokeWeight(10);
				p5.line(
					graph[node].x - graph[node].value / 500,
					graph[node].y,
					graph[node].x - graph[node].value / 500 + graph[node].value / 250,
					graph[node].y
				);

				if (graph[graph[node].parentId] ?? false) {
					p5.stroke(graph[node].color);
					p5.strokeWeight(5);

					if (nodeIdx !== 0 && nodeIdx % 2 == 0) {
						let [x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x - graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x +
								graph[graph[node].parentId].value / 500 -
								graph[node].value / 250,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);

						[x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x + graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x + graph[graph[node].parentId].value / 500,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);
					} else {
						let [x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x - graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x - graph[graph[node].parentId].value / 500,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);

						[x1, y1, x2, y2, x3, y3, x4, y4] = getBezierCoords(
							graph[node].x + graph[node].value / 500,
							graph[node].y,
							graph[graph[node].parentId].x -
								graph[graph[node].parentId].value / 500 +
								graph[node].value / 250,
							graph[graph[node].parentId].y
						);

						p5.bezier(x1, y1, x2, y2, x3, y3, x4, y4);
					}
				}

				p5.noStroke();
				// p5.textAlign(p5.CENTER, p5.CENTER);
				p5.textStyle(p5.BOLD);

				p5.text(graph[node].title, graph[node].x - graph[node].value / 600, graph[node].y + 20);
				p5.textStyle(p5.BOLDITALIC);
				p5.text(
					`(${graph[node].value.toLocaleString()})`,
					graph[node].x - graph[node].value / 600,
					graph[node].y + 35
				);
			}
		};
	};

	let loaded = false;
	onMount(async () => {
		reviewsData = await d3.csv('/reviews.csv');
		reviews = reviewsData.sort((a, b) => a.unixReviewTime - b.unixReviewTime);
		selectedReviews = reviews;

		fromValue = reviews.at(0).unixReviewTime;
		toValue = reviews.at(-1).unixReviewTime;

		console.log(reviews);

		loaded = true;
	});
</script>

{#if loaded && reviews.length > 0}
	<div class=" flex">
		<div>
			<P5 {sketch} />
			<div>
				<div class=" wrap relative mt-2">
					<input
						id="fromSlider"
						type="range"
						class=" absolute"
						bind:value={fromValue}
						min={reviews.at(0).unixReviewTime}
						max={reviews.at(-1).unixReviewTime}
					/>
					<input
						type="range"
						class=" absolute"
						bind:value={toValue}
						min={reviews.at(0).unixReviewTime}
						max={reviews.at(-1).unixReviewTime}
					/>
				</div>
			</div>
			<div class=" mt-5 flex justify-between">
				<div>
					{dayjs.unix(fromValue).format('YYYY-MMM-DD')}
				</div>

				<div class=" font-bold">
					{selectedReviews.length} Reviews Selected
				</div>

				<div>
					{dayjs.unix(toValue).format('YYYY-MMM-DD')}
				</div>
			</div>

			<LineChart
				data={selectedReviews.reduce((acc, entry) => {
					const date = new Date(entry.unixReviewTime * 1000);
					const firstDayOfMonth = new Date(date.getFullYear(), date.getMonth(), 1);
					const formattedDate = `${firstDayOfMonth.getFullYear()}/${(firstDayOfMonth.getMonth() + 1)
						.toString()
						.padStart(2, '0')}/01`;

					acc[formattedDate] = (acc[formattedDate] || 0) + 1;

					return acc;
				}, {})}
				from={fromValue}
				to={toValue}
			/>
		</div>

		<div>dsfs</div>
	</div>
{/if}

<style>
	#fromSlider {
		height: 0;
		z-index: 1;
	}
	input[type='range']::-webkit-slider-thumb {
		-webkit-appearance: none;
		pointer-events: all;
		width: 24px;
		height: 24px;
		background-color: #fff;
		border-radius: 50%;
		box-shadow: 0 0 0 1px #c6c6c6;
		cursor: pointer;
	}

	input[type='range']::-moz-range-thumb {
		-webkit-appearance: none;
		pointer-events: all;
		width: 24px;
		height: 24px;
		background-color: #fff;
		border-radius: 50%;
		box-shadow: 0 0 0 1px #c6c6c6;
		cursor: pointer;
	}

	input[type='range']::-webkit-slider-thumb:hover {
		background: #f7f7f7;
	}

	input[type='range']::-webkit-slider-thumb:active {
		box-shadow:
			inset 0 0 3px #387bbe,
			0 0 9px #387bbe;
		-webkit-box-shadow:
			inset 0 0 3px #387bbe,
			0 0 9px #387bbe;
	}

	input[type='range'] {
		-webkit-appearance: none;
		appearance: none;
		height: 2px;
		width: 100%;
		position: absolute;
		background-color: #c6c6c6;
		pointer-events: none;
	}
</style>
