<script lang="ts">
  import { exportToSvg } from "@excalidraw/excalidraw";

  export let isPresenting: boolean;
  export let onPresentingChange: (value: boolean) => void;
  export let slides: Array<{ elements: any[]; appState: any; files: any }>;
  export let onClearAll: () => void;

  function importFromJson() {
    const input = prompt("Paste your slides JSON here:");
    if (!input) {
      alert("Input something");
      return;
    }

    try {
      const newSlides = JSON.parse(input);
      if (Array.isArray(newSlides) && newSlides.length > 0) {
        localStorage.slides = input;
        location.reload();
      } else {
        alert("Invalid slides format");
      }
    } catch (e) {
      alert("Invalid JSON format");
    }
  }

  async function exportToPDF() {
    const printWindow = window.open("", "_blank");
    if (!printWindow) {
      alert("Please allow popups to export PDF");
      return;
    }

    printWindow.document.write("<html><body>");

    for (let i = 0; i < slides.length; i++) {
      const svg = await exportToSvg({
        elements: slides[i].elements,
        appState: {
          ...slides[i].appState,
          viewBackgroundColor: "#fff",
          exportWithDarkMode: false,
        },
        files: slides[i].files,
      });

      svg.style.width = "100%";
      svg.style.height = "9in";
      svg.style.pageBreakAfter = "always";
      printWindow.document.body.appendChild(svg);
    }

    printWindow.document.write("</body></html>");
    printWindow.document.write(`<style>
@page {
size: 16in 9in;
margin: 0;
}
</style>`);
    printWindow.document.close();

    setTimeout(() => {
      printWindow.print();
      printWindow.close();
    }, 500);
  }
</script>

<div class="button-container" class:hidden={isPresenting}>
  <button class="present-button" on:click={() => onPresentingChange(!isPresenting)}>
    {isPresenting ? "Exit" : "Present"}
  </button>
  <button class="export-button" on:click={exportToPDF}> Export PDF </button>
  <button class="import-button" on:click={importFromJson}> Import </button>
  <button class="clear-button" on:click={onClearAll}> Clear All </button>
</div>

<style>
  .button-container {
    position: fixed;
    bottom: 1rem;
    right: 1rem;
    z-index: 1;
    display: flex;
    height: 2.5rem;
    gap: 0.5rem;
  }

  .hidden {
    display: none;
  }

  button {
    border-radius: 2.5rem;
    font-weight: 500;
    background-color: rgb(var(--m3-scheme-secondary-container));
    color: rgb(var(--m3-scheme-on-secondary-container));
  }

  .present-button,
  .export-button,
  .import-button {
    padding: 0.5rem 1rem;
  }

  .clear-button {
    padding: 0.5rem 1rem;
    background: rgb(var(--m3-scheme-error-container));
    color: rgb(var(--m3-scheme-on-error-container));
  }
</style>
