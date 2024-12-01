<script lang="ts">
  import SlideEditor from "./components/SlideEditor.svelte";
  import SlidePresenter from "./components/SlidePresenter.svelte";
  import ControlButtons from "./components/ControlButtons.svelte";

  let currentSlide = 0;
  let slides = [{ elements: [], appState: {}, files: {} }];
  let isPresenting = false;

  const savedSlides = localStorage.getItem("slides");
  if (savedSlides) {
    slides = JSON.parse(savedSlides);
  }
  $: if (slides) {
    localStorage.setItem(
      "slides",
      JSON.stringify(
        slides.map((slide) => {
          return {
            elements: slide.elements,
            appState: {
              ...slide.appState,
              collaborators: undefined,
            },
            files: slide.files,
          };
        }),
      ),
    );
  }

  function deleteSlide(index: number) {
    if (slides.length <= 1) {
      // Don't delete the last slide
      slides = [{ elements: [], appState: {}, files: {} }];
    } else {
      slides = slides.filter((_, i) => i !== index);
      if (currentSlide >= slides.length) {
        currentSlide = slides.length - 1;
      }
    }
  }

  function clearAllSlides() {
    if (confirm("Are you sure you want to clear all slides?")) {
      slides = [{ elements: [], appState: {}, files: {} }];
      currentSlide = 0;
    }
  }

  function switchSlide(index: number) {
    currentSlide = index;
  }

  function handleSlideChange(elements: any, appState: any, files: any) {
    slides[currentSlide] = { elements, appState, files };
    slides = slides;
  }

  function handleKeydown(event: KeyboardEvent) {
    if (!isPresenting) return;

    if (event.key === "ArrowRight" || event.key === "Space") {
      if (currentSlide < slides.length - 1) {
        switchSlide(currentSlide + 1);
      }
    } else if (event.key === "ArrowLeft") {
      if (currentSlide > 0) {
        switchSlide(currentSlide - 1);
      }
    } else if (event.key === "Escape") {
      isPresenting = false;
    }
  }
</script>

<svelte:window on:keydown={handleKeydown} />

{#if !isPresenting}
  <SlideEditor
    {currentSlide}
    {slides}
    onSlideChange={handleSlideChange}
    onSlideSwitch={switchSlide}
    onDeleteSlide={deleteSlide}
  />
{:else}
  <SlidePresenter {currentSlide} {slides} />
{/if}

<ControlButtons
  {isPresenting}
  onPresentingChange={(value) => (isPresenting = value)}
  {slides}
  onClearAll={clearAllSlides}
/>
