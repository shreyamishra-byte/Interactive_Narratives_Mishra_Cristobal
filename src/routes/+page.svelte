<script lang="ts">
  import rawBlocks from '$lib/doc.blocks.json';
  import DocRenderer from '$lib/components/DocRenderer.svelte';
  import VideoBackground from '$lib/components/VideoBackground.svelte';
  import ClickThroughImages from '$lib/components/ClickThroughImages.svelte';
  import type { Block } from '$lib/components/DocRenderer.svelte';
  const blocks = rawBlocks as Block[];

  const videoOverlayText = `
    <h1>Purple people</h1>
    <p>A people's perspective to the purple sea urchin overpopulation in California. </p>
  `;
</script>
<style>
  :global(.overlay-content h1) {
    color: #fff;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
    font-family: 'Georgia', serif;
  }
</style>





<!-- hard-coded custom code that should appear BEFORE Google Doc import can go here. -->
<VideoBackground 
  src="/IMG_7777.MOV"
  title="Opening video"
  overlayContent={videoOverlayText}
/>

<ClickThroughImages />

<!-- Code imported from Google Doc -->
<div class="container-fluid">
  <div class="row justify-content-center">
    <div class="col-12 col-lg-10 col-xxl-8">
      <DocRenderer {blocks} />
    </div>
  </div>
</div>


<!-- hard-coded custom code that should appear AFTER Google Doc import can go here. -->