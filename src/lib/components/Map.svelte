

  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
  <link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />

  
  <div class="container-fluid">
    <div class="row">
      
      <!-- Story cards holder-->
      <div class="col-12 col-lg-5 story-column" id="story">

        <!-- the first story block hard-coded. The others are generated -->
        <section class="story-block intro-block">
          <div class="story-card" data-card-type="intro">
            <h2 class="h4 mb-3 text-center">Welcome to My Story Map</h2>
            <p class="mb-0 text-center">Let me take you on a journey</p>
          </div>
        </section>

      </div>

      <!-- Map holder -->
      <div class="col-12 map-column">
        <div class="map-fixed">
          <div id="map"></div>
        </div>
      </div>
    </div>
  </div>

  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
    }

    .story-block {
      min-height: 100vh;
      display: flex;
      align-items: center;
      padding: 1rem 2rem;
    }

    .story-card {
      width: 100%;
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 0.75rem;
      padding: 1.5rem;
      box-shadow: 0 0.5rem 1rem rgba(0,0,0,0.08);
    }

    .map-column {
      position: relative;
      min-height: 100vh;
    }

    .map-fixed {
      position: fixed;
      top: 0;
      right: 0;
      z-index: -1;
      width: 100%;
      height: 100vh;
    }

    #map {
      width: 100%;
      height: 100%;
    }
  </style>

  <script>

  import * as maplibregl from 'maplibre-gl';

    const locations = [
      {
        title: "Fort Ross (Sonoma County)",
        lat: 38.259141,
        lon: -122.491092,
        zoom: 8,
        description: "The rocky, sheltered coves of Fort Ross Cove provide a perfect habitat for purple sea urchins to settle, leading to dense concentrations that locals are now attempting to harvest."
      }, 
      {
        title: "Shelter Cove (Sonoma County)",
        lat: 40.028591,
        lon: -124.072578,
        zoom: 8,
        description: "Shelter Cove is a focal point of this issue because it is located in the heart of this ecological disaster, where a massive, once-abundant kelp forest has been replaced by a carpet of hungry purple urchins."
      },
      {
        title: "Caspar Cove (Mendocino County)",
        lat: 39.363,
        lon: -123.814,
        zoom: 8,
        description: "The once-thriving kelp forests of Caspar Cove have been decimated by the voracious appetite of purple sea urchins, leaving behind a barren underwater landscape that has disrupted local ecosystems and livelihoods."
      },
      {
        title: "Stillwater Cove (Mendocino County)",
        lat: 39.343,
        lon: -123.821,
        zoom: 8,
        description: "Stillwater Cove, once a vibrant marine habitat, has been transformed into a desolate expanse due to the unchecked proliferation of purple sea urchins, which have overgrazed the kelp forests and left behind a stark, barren seascape."
      }
    ]; //YOU NEED TO EDIT THIS LINE FOR THE MAP TO WORK


    //Check out other basemap options here: https://madewithmaplibre.com/basemaps/gallery
    const map = new maplibregl.Map({
        container: "map",
        style: "https://tiles.openfreemap.org/styles/liberty",

      center: [-98.5795, 39.8283],
      zoom: 3
    });

    //disable all interactions with the map
    map.scrollZoom.disable();
    map.dragPan.disable();
    map.dragRotate.disable();
    map.touchZoomRotate.disable();
    map.doubleClickZoom.disable();
    map.keyboard.disable();

    // Add markers
    locations.forEach(location => {
      new maplibregl.Marker()
        .setLngLat([location.lon, location.lat])
        .addTo(map);
    });

    // Build bounds for opening overview
    const openingBounds = new maplibregl.LngLatBounds();
    locations.forEach(location => {
      openingBounds.extend([location.lon, location.lat]);
    });

    const storyContainer = document.querySelector("#story");

    locations.forEach((location, index) => {
      const block = document.createElement("section");
      block.className = "story-block";

      block.innerHTML = `
        <div class="story-card" data-card-type="location" data-location-index="${index}">
          <h2 class="h4 mb-3">${location.title}</h2>
          <p class="mb-0">${location.description}</p>
        </div>
      `;

      storyContainer.appendChild(block);
    });

    const introCard = document.querySelector('.story-card[data-card-type="intro"]');
    const allCards = document.querySelectorAll("#story .story-card");

    let activeState = null;

    function showOpeningOverview(animated = true) {
      if (activeState === "intro") return;
      activeState = "intro";

      map.fitBounds(openingBounds, {
        padding: { top: 80, right: 80, bottom: 80, left: 80 },
        maxZoom: 5,
        duration: animated ? 1200 : 0
      });
    }

    function activateLocation(locationIndex) {
      if (activeState === locationIndex) return;

      const location = locations[locationIndex];
      if (!location) return;

      activeState = locationIndex;

      map.flyTo({
        center: [location.lon, location.lat],
        zoom: location.zoom,
        speed: 0.9,
        curve: 1.2,
        essential: true
      });
    }

    function getClosestCardToViewportMiddle() {
      const viewportMid = window.innerHeight / 2;
      let closestCard = null;
      let closestDistance = Infinity;

      allCards.forEach(card => {
        const rect = card.getBoundingClientRect();
        const cardCenter = rect.top + (rect.height / 2);
        const distance = Math.abs(cardCenter - viewportMid);

        if (distance < closestDistance) {
          closestDistance = distance;
          closestCard = card;
        }
      });

      return closestCard;
    }

    function updateMapFromScroll() {
      const closestCard = getClosestCardToViewportMiddle();
      if (!closestCard) return;

      const cardType = closestCard.dataset.cardType;

      if (cardType === "intro") {
        showOpeningOverview();
        return;
      }

      if (cardType === "location") {
        const locationIndex = Number(closestCard.dataset.locationIndex);
        activateLocation(locationIndex);
      }
    }

  
    const observer = new IntersectionObserver(
      entries => {
        for (const entry of entries) {
          if (entry.isIntersecting) {
            updateMapFromScroll();
            break;
          }
        }
      },
      {
        root: null,
        rootMargin: "-45% 0px -45% 0px",
        threshold: 0
      }
    );

    allCards.forEach(card => observer.observe(card));

    window.addEventListener("scroll", updateMapFromScroll, { passive: true });
    window.addEventListener("resize", updateMapFromScroll);

    map.on("load", () => {
      showOpeningOverview(false);
    });
  </script>