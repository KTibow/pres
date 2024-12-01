<script lang="ts">
  import { exportToSvg } from "@excalidraw/excalidraw";

  export let currentSlide: number;
  export let slides: Array<{ elements: any[]; appState: any; files: any }>;

  let container: HTMLDivElement;

  async function renderCurrentSlide() {
    if (!container) return;

    container.innerHTML = "";
    const svg = await exportToSvg({
      elements: slides[currentSlide].elements,
      appState: {
        ...slides[currentSlide].appState,
        viewBackgroundColor: "#fff",
        exportWithDarkMode: false,
      },
      files: slides[currentSlide].files,
    });

    svg.setAttribute("width", "100%");
    svg.setAttribute("height", "100%");
    container.appendChild(svg);
  }

  $: if (container && currentSlide >= 0) {
    renderCurrentSlide();
  }
</script>

<div class="presentation-mode" bind:this={container}></div>

<style>
  .presentation-mode {
    width: 100%;
    height: 100%;
    background: #fff;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
