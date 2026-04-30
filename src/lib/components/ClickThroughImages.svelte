<script lang="ts">
  const images = [
    '/scrolly/1uni.jpg',
    '/scrolly/2uni.jpg',
    '/scrolly/3uni.jpg',
    '/scrolly/4uni.jpg'
  ];

  let currentIndex = 0;

  function nextImage() {
    currentIndex = (currentIndex + 1) % images.length;
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      nextImage();
    }
  }
</script>

<div
  class="click-through-images"
  role="button"
  tabindex="0"
  aria-label="Click to advance to the next image"
  on:click={nextImage}
  on:keydown={handleKeydown}
>
  <div class="image-panel">
    <img
      src={images[currentIndex]}
      alt={`Scrolly image ${currentIndex + 1}`}
      class="image-content"
      loading="eager"
      decoding="async"
    />

    <div class="image-overlay">
      <div>
        <h2>Click to advance</h2>
        <p>{currentIndex + 1} of {images.length}</p>
      </div>
    </div>
  </div>
</div>

<style>
  .click-through-images {
    cursor: url('/scrolly/handdd-cursor-3x.png') 65 , pointer;
    margin: 0;
    width: 100%;
    outline: none;
    background: transparent;
  }



  .image-panel {
    position: relative;
    width: 100%;
    height: 100vh;
    overflow: hidden;
  }

  .image-content {
    width: 100%;
    height: 100%;
    
    /* KEY FIX */
    object-fit: contain; /* prevents ugly upscaling blur */

    object-position: center;
    display: block;

    /* improves rendering sharpness slightly */
    transform: translateZ(0);
  }

  .click-through-images:hover .image-panel {
    transform: scale(1.01);
    transition: transform 0.35s ease;
  }

  .image-overlay {
    position: absolute;
    inset: 0;
    background: transparent;

    display: flex;
    align-items: flex-end;
    justify-content: center;
    padding: 2rem;
    color: white;
    text-align: center;
  }

  .image-overlay h2 {
    margin: 0;
    font-size: clamp(1.75rem, 3vw, 2.75rem);
  }

  .image-overlay p {
    margin: 0.75rem 0 0;
    font-size: clamp(1rem, 2vw, 1.25rem);
    opacity: 0.92;
  }
</style>