<script lang="ts">
  import { onMount } from "svelte";
  import { Excalidraw } from "@excalidraw/excalidraw";
  import * as React from "react";
  import { createRoot } from "react-dom/client";

  export let currentSlide: number;
  export let slides: Array<{ elements: any[]; appState: any }>;
  export let onSlideChange: (elements: any, appState: any, files: any) => void;
  export let onSlideSwitch: (index: number) => void;
  export let onDeleteSlide: (index: number) => void;

  let container: HTMLDivElement;
  let excalidrawAPI: any;

  onMount(() => {
    const root = createRoot(container);
    root.render(
      React.createElement(Excalidraw, {
        onChange: (elements: any, appState: any, files: any) => {
          onSlideChange(elements, appState, files);
        },
        initialData: slides[currentSlide],
        viewModeEnabled: false,
        zenModeEnabled: false,
        gridModeEnabled: false,
        theme: window.matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light",
        excalidrawAPI: (api: any) => {
          excalidrawAPI = api;
        },
      }),
    );

    return () => {
      root.unmount();
    };
  });
</script>

<div class="slide-editor" bind:this={container}></div>
<div class="filmstrip">
  {#each slides as _, i}
    <div class="slide-wrapper">
      <button
        class="slide-thumbnail"
        class:active={i === currentSlide}
        on:click={() => {
          if (excalidrawAPI) {
            excalidrawAPI.updateScene(slides[i]);
          }
          onSlideSwitch(i);
        }}
      >
        Slide {i + 1}
      </button>
      <button
        class="delete-button"
        on:click={(e) => {
          e.stopPropagation();
          onDeleteSlide(i);
        }}
      >
        ×
      </button>
    </div>
  {/each}
  <button
    class="slide-thumbnail"
    on:click={() => {
      slides = [...slides, { elements: [], appState: {} }];

      const i = slides.length - 1;
      // Focus the new slide after it's added
      onSlideSwitch(i);
      excalidrawAPI.updateScene(slides[i]);
    }}
  >
    +
  </button>
</div>

<style>
  .slide-editor {
    flex: 1;
    min-height: 0;
  }

  .filmstrip {
    height: 8rem;
    background: rgb(var(--m3-scheme-surface-container-low));
    display: flex;
    gap: 0.5rem;
    padding: 0.5rem;
    overflow-x: auto;
  }

  .slide-wrapper {
    position: relative;
  }

  .slide-thumbnail {
    width: 12rem;
    height: 6.75rem;
    background: rgb(var(--m3-scheme-surface-container-high));
    display: flex;
    align-items: center;
    justify-content: center;
    border: 2px solid transparent;
    border-radius: 0.5rem;
  }

  .delete-button {
    position: absolute;
    top: -0.5rem;
    right: -0.5rem;
    width: 1.25rem;
    height: 1.25rem;
    border-radius: 1.25rem;
    background: rgb(var(--m3-scheme-error-container));
    color: rgb(var(--m3-scheme-on-error-container));
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.875rem;
    padding: 0;
  }

  .slide-thumbnail.active {
    border-color: rgb(var(--m3-scheme-primary));
  }
</style>
