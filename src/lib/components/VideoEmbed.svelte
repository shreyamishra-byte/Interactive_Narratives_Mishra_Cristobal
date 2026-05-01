<!-- src/lib/components/VideoEmbed.svelte -->
<script lang="ts">
  export let src: string | undefined;

  export let title: string = '';
  export let size: 'full' | 'large' | 'fit' = 'large';
  export let caption: string | undefined;

  export let autoplay: boolean | string | undefined = false;
  export let controls: boolean | string | undefined = true;
  export let playsinline: boolean | string | undefined = true;
  export let mute: boolean | string | undefined = false;

  export let captions: string | undefined;
  export let srclang: string = 'en';
  export let label: string = 'English';

  function toBool(v: boolean | string | undefined, fallback: boolean) {
    if (typeof v === 'boolean') return v;
    if (typeof v === 'string') {
      const s = v.trim().toLowerCase();
      if (['true', '1', 'yes', 'y', 'on'].includes(s)) return true;
      if (['false', '0', 'no', 'n', 'off'].includes(s)) return false;
    }
    return fallback;
  }

  function safeUrl(raw: string): URL | null {
    try {
      return new URL(raw);
    } catch {
      return null;
    }
  }

  function isYouTubeHost(host: string) {
    return host === 'youtube.com' || host.endsWith('.youtube.com') || host === 'youtu.be';
  }

  function isVimeoHost(host: string) {
    return host === 'vimeo.com' || host.endsWith('.vimeo.com') || host === 'player.vimeo.com';
  }

  function extractYouTubeId(u: URL): string | null {
    if (u.hostname === 'youtu.be') {
      return u.pathname.split('/').filter(Boolean)[0] || null;
    }
    if (u.pathname === '/watch') return u.searchParams.get('v');
    const embedMatch = u.pathname.match(/^\/embed\/([^/]+)/);
    if (embedMatch) return embedMatch[1];
    const shortsMatch = u.pathname.match(/^\/shorts\/([^/]+)/);
    if (shortsMatch) return shortsMatch[1];
    const liveMatch = u.pathname.match(/^\/live\/([^/]+)/);
    if (liveMatch) return liveMatch[1];
    return null;
  }

  function extractVimeoId(u: URL): string | null {
    const playerMatch = u.pathname.match(/^\/video\/(\d+)/);
    if (playerMatch) return playerMatch[1];
    const simpleMatch = u.pathname.match(/^\/(\d+)/);
    if (simpleMatch) return simpleMatch[1];
    const anyIdMatch = u.pathname.match(/\/(\d+)(?:$|\/)/);
    return anyIdMatch ? anyIdMatch[1] : null;
  }

  function isDirectVideoFile(raw: string) {
    return /\.(mp4|webm|mov)(\?.*)?$/i.test(raw.trim());
  }

  function normalizeStaticPath(raw: string) {
    if (/^https?:\/\//i.test(raw)) return raw;
    return raw.startsWith('/') ? raw : '/' + raw;
  }

  type Resolved =
    | { kind: 'youtube'; embedUrl: string }
    | { kind: 'vimeo'; embedUrl: string }
    | { kind: 'file'; fileUrl: string };

  function resolveVideo(rawSrc: string): Resolved | null {
    const s = rawSrc.trim();
    if (!s) return null;

    if (isDirectVideoFile(s)) {
      return { kind: 'file', fileUrl: normalizeStaticPath(s) };
    }

    const u = safeUrl(s);
    if (!u) return null;

    const autoplayBool = toBool(autoplay, false);
    const controlsBool = toBool(controls, true);
    const playsinlineBool = toBool(playsinline, true);
    const muteBool = toBool(mute, false);

    if (isYouTubeHost(u.hostname)) {
      const id = extractYouTubeId(u);
      if (!id) return null;

      const params = new URLSearchParams();
      if (autoplayBool) params.set('autoplay', '1');
      params.set('controls', controlsBool ? '1' : '0');
      if (playsinlineBool) params.set('playsinline', '1');
      if (muteBool) params.set('mute', '1');
      params.set('rel', '0');

      return {
        kind: 'youtube',
        embedUrl: `https://www.youtube-nocookie.com/embed/${id}?${params.toString()}`
      };
    }

    if (isVimeoHost(u.hostname)) {
      const id = extractVimeoId(u);
      if (!id) return null;

      const params = new URLSearchParams();
      if (autoplayBool) params.set('autoplay', '1');
      if (!controlsBool) params.set('controls', '0');
      if (muteBool) params.set('muted', '1');

      return {
        kind: 'vimeo',
        embedUrl: `https://player.vimeo.com/video/${id}?${params.toString()}`
      };
    }

    return null;
  }

  $: resolved = src ? resolveVideo(src) : null;

  $: autoplayBool = toBool(autoplay, false);
  $: controlsBool = toBool(controls, true);
  $: playsinlineBool = toBool(playsinline, true);
  $: muteBool = toBool(mute, false);

  $: captionsSrc = captions ? normalizeStaticPath(captions) : undefined;
</script>

{#if resolved}
  <figure class="my-3">

    <div class="ratio ratio-16x9">

      {#if resolved.kind === 'file'}
        <video
          src={resolved.fileUrl}
          title={title}
          class="position-absolute top-0 start-0 w-100 h-100"
          autoplay={autoplayBool}
          controls={controlsBool}
          muted={muteBool}
          playsinline={playsinlineBool}
        >
          {#if captionsSrc}
            <track kind="captions" src={captionsSrc} srclang={srclang} label={label} default />
          {/if}
        </video>

      {:else}
        <iframe
          src={resolved.embedUrl}
          title={title}
          class="position-absolute top-0 start-0 w-100 h-100 border-0"
          allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
          allowfullscreen
        ></iframe>
      {/if}

    </div>

    {#if caption}
      <figcaption class="mt-2 text-muted small text-center">
        {caption}
      </figcaption>
    {/if}

  </figure>
{/if}
