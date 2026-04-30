<script lang="ts">
  export let src: string;
  export let title: string = '';
  export let overlayContent: string | undefined = undefined;
</script>

<section class="video-background-section">
  <video 
    src={src}
    autoplay 
    muted 
    loop 
    playsinline
    class="video-bg"
    title={title}
  >
    <track kind="captions" srclang="en" label="English" />
  </video>
  
  {#if overlayContent}
    <div class="overlay-content">
      {@html overlayContent}
    </div>
  {/if}
</section>

<style>
  .video-background-section {
    position: relative;
    width: 100%;
    height: 100vh;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .video-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    z-index: 1;
  }

  .overlay-content {
    position: relative;
    z-index: 2;
    text-align: center;
    color: white;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
    max-width: 80%;
    margin: 0 auto;
  }

  :global(.overlay-content h1) {
    font-size: clamp(2rem, 8vw, 4rem);
    font-weight: bold;
    margin-bottom: 1rem;
  }

  :global(.overlay-content p) {
    font-size: clamp(1rem, 3vw, 1.5rem);
    line-height: 1.6;
  }
</style>
