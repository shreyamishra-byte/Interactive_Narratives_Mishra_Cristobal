<script>
	import { onMount, onDestroy } from 'svelte';

	// init variables
	let map;
	let mapContainer;
	let openingBounds;
	let activeState = null;
	let observer;

	const locations = [
		{
			title: 'Fort Ross (Sonoma County)',
			lat: 38.259141,
			lon: -122.491092,
			zoom: 12,
			description:
				'The rocky, sheltered coves of Fort Ross Cove provide a perfect habitat for purple sea urchins to settle.'
		},
		{
			title: 'Shelter Cove (Sonoma County)',
			lat: 40.028591,
			lon: -124.072578,
			zoom: 12,
			description:
				'Shelter Cove is a focal point of this issue because it is located in the heart of this ecological disaster.'
		},
		{
			title: 'Caspar Cove (Mendocino County)',
			lat: 39.363,
			lon: -123.814,
			zoom: 12,
			description:
				'The once-thriving kelp forests of Caspar Cove have been decimated by the voracious appetite of purple sea urchins.'
		},
		{
			title: 'Stillwater Cove (Mendocino County)',
			lat: 39.343,
			lon: -123.821,
			zoom: 12,
			description:
				'Stillwater Cove has been transformed into a desolate expanse due to the unchecked proliferation of purple sea urchins.'
		},
    {
      title: "Point Loma Rocky Reefs (San Diego)",
    lat: 32.6556,
    lon: -117.2425,
    zoom: 10,
    description: "Southern California kelp forest edge habitat with established purple sea urchin populations on rocky reefs."
    },
    {
    title: "La Jolla Cove Reef System",
    lat: 32.8338,
    lon: -117.2713,
    zoom: 10,
    description: "Rocky reef ecosystem with kelp forest influence and consistent purple sea urchin presence."
  }
	];

	// this needs to be in an onMount, or ssr error otherwise
	onMount(async () => {
		const maplibregl = (await import('maplibre-gl')).default;
    // same for css import i believe?
		import('maplibre-gl/dist/maplibre-gl.css');

		// more basemap styles are available
		map = new maplibregl.Map({
			container: mapContainer,
			style: 'https://tiles.openfreemap.org/styles/positron',
			center: [-123, 39],
			zoom: 7
		});

		// interactions
		map.scrollZoom.disable();
		map.dragPan.disable();
		map.dragRotate.disable();

		openingBounds = new maplibregl.LngLatBounds();
		locations.forEach((loc) => {
			const markerElement = document.createElement('div');
			markerElement.className = 'custom-marker';
			new maplibregl.Marker({ element: markerElement }).setLngLat([loc.lon, loc.lat]).addTo(map);
			openingBounds.extend([loc.lon, loc.lat]);
		});

		// scrolly with intersection observer
		observer = new IntersectionObserver(
			(entries) => {
				entries.forEach((entry) => {
					if (entry.isIntersecting) {
						const { type, index } = entry.target.dataset;
						if (type === 'intro') showOpeningOverview();
						else activateLocation(Number(index));
					}
				});
			},
			{
				threshold: 0.6 // trigger when 60% of the card is visible
			}
		);

		document.querySelectorAll('.story-card').forEach((card) => observer.observe(card));

		map.on('load', () => showOpeningOverview(false));
	});

	onDestroy(() => {
		if (observer) observer.disconnect();
		if (map) map.remove();
	});

	function showOpeningOverview(animated = true) {
		// need to check if map is true
		if (!map || activeState === 'intro') return;
		activeState = 'intro';
		map.fitBounds(openingBounds, {
			padding: 100,
			maxZoom: 7,
			duration: animated ? 1500 : 0
		});
	}

	function activateLocation(index) {
		// need to check if map is true
		if (!map || activeState === index) return;
		const loc = locations[index];
		activeState = index;
		map.flyTo({
			center: [loc.lon, loc.lat],
			zoom: loc.zoom,
			speed: 0.8,
			essential: true
		});
	}
</script>

<div class="container-fluid p-0">
	<div class="row g-0 scrolly-container">
		<div class="col-12 col-lg-5 story-column">
			<section class="story-block">
				<div class="story-card" data-type="intro">
					<h1 class="h3 mb-3">Sea Urchins of the California Coast</h1>
					<p>
						The kelp forests of Northern California are disappearing. Scroll down to see the
						affected areas.
					</p>
					<div class="scroll-indicator">↓ Scroll to begin</div>
				</div>
			</section>

			{#each locations as location, index}
				<section class="story-block">
					<div class="story-card" data-type="location" data-index={index}>
						<h2 class="h4 mb-3">{location.title}</h2>
						<p>{location.description}</p>
					</div>
				</section>
			{/each}

			<div class="spacer"></div>
		</div>

		<div class="col-12 col-lg-7 map-column">
			<div class="map-sticky-wrapper">
				<div bind:this={mapContainer} id="map"></div>
			</div>
		</div>
	</div>
</div>

<style>
	/* Reset and Layout */
	:global(body) {
		margin: 0;
		padding: 0;
	}

  :global(.custom-marker) {
  width: 56px;
  height: 40px;
  background-image: url('/Map_elements/uni.jpg');
  background-size: cover;
  background-repeat: no-repeat;
  background-position: center;
  }
	.scrolly-container {
		display: flex;
		align-items: stretch;
		position: relative;
	}

	/* Story Styling */
	.story-column {
		position: relative;
		z-index: 2;
	}

	.story-block {
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 0 2rem;
	}

	.story-card {
		background: rgba(255, 255, 255, 0.95);
		padding: 2rem;
		border-radius: 12px;
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
		max-width: 450px;
		width: 100%;
		border: 1px solid #eee;
	}

	.scroll-indicator {
		margin-top: 2rem;
		font-size: 0.9rem;
		color: #666;
		font-weight: bold;
		text-align: center;
	}

	.spacer {
		height: 50vh;
	}

	/* Map Styling */
	.map-column {
		position: relative;
	}

	.map-sticky-wrapper {
		position: sticky;
		top: 0;
		width: 100%;
		height: 100vh;
		background: #f8f9fa;
	}

	#map {
		width: 100%;
		height: 100%;
	}
  

	/* maybe for mobile needs to be different? */
	@media (max-width: 1000px) {
		.map-column {
			order: -1;
			height: 40vh;
			position: relative;
		}
		.map-sticky-wrapper {
			position: relative;
			height: 40vh;
		}
		.story-block {
			height: auto;
			padding: 4rem 1rem;
		}
	}
</style>
