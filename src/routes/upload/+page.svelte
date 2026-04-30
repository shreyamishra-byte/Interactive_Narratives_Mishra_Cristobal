<script lang="ts">
  let file: File | null = null;
  let videoUrl = '';
  let error = '';

  const acceptedTypes = ['video/mp4', 'video/webm'];
  const maxFileSize = 250 * 1024 * 1024; // 250MB

  function formatSize(bytes: number) {
    if (bytes < 1024) return `${bytes} B`;
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
    return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
  }

  function handleFile(event: Event) {
    const input = event.currentTarget as HTMLInputElement;
    const selected = input.files?.[0] ?? null;

    if (!selected) {
      file = null;
      videoUrl = '';
      error = '';
      return;
    }

    if (!acceptedTypes.includes(selected.type)) {
      error = 'Please choose an MP4 or WEBM video file.';
      file = null;
      videoUrl = '';
      return;
    }

    if (selected.size > maxFileSize) {
      error = `File is too large. Maximum allowed size is ${formatSize(maxFileSize)}.`;
      file = null;
      videoUrl = '';
      return;
    }

    error = '';
    file = selected;
    if (videoUrl) URL.revokeObjectURL(videoUrl);
    videoUrl = URL.createObjectURL(selected);
  }
</script>

<main class="container py-5">
  <div class="row justify-content-center">
    <div class="col-12 col-lg-8">
      <h1 class="mb-4">Upload a Video</h1>
      <p class="mb-3">Choose a local MP4 or WEBM file to preview it in the browser. This is a client-side preview, not a permanent server upload.</p>

      <div class="mb-3">
        <label class="form-label" for="video-file">Select video</label>
        <input
          id="video-file"
          class="form-control"
          type="file"
          accept="video/mp4,video/webm"
          on:change={handleFile}
        />
      </div>

      {#if error}
        <div class="alert alert-danger">{error}</div>
      {/if}

      {#if file}
        <div class="card mb-4">
          <div class="card-body">
            <h2 class="h5">Preview</h2>
            <p class="mb-2 text-muted">{file.name} · {formatSize(file.size)} · {file.type}</p>
            <div class="ratio ratio-16x9 mb-3">
              <video src={videoUrl} controls playsinline class="w-100 h-100">
                <track kind="captions" srclang="en" label="English" />
              </video>
            </div>
            <p class="small text-muted mb-0">
              Note: this preview only lives in your browser while the page is open. To make the video part of the site permanently, copy it into <code>static/videos/</code> and use that path in your content.
            </p>
          </div>
        </div>
      {/if}
    </div>
  </div>
</main>
