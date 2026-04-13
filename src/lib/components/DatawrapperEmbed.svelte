<script>
  export let id; // This receives the "tDcgV" from your Google Doc
</script>

<div class="datawrapper-container">
  <iframe
    title="Datawrapper Chart"
    aria-label="Interactive chart"
    src="https://datawrapper.dwcdn.net/{id}/"
    scrolling="no"
    frameborder="0"
    style="width: 0; min-width: 100% !important; border: none;"
    height="400"
  ></iframe>
</div>

<style>
  .datawrapper-container {
    margin: 2rem auto;
    width: 100%;
    max-width: 800px; /* Adjust based on your layout */
  }
  
  iframe {
    transition: height 0.2s;
  }
</style>