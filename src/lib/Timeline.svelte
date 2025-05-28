<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import Swiper from 'swiper';
  import { Navigation, Pagination } from 'swiper/modules';
  import 'swiper/css';
  import 'swiper/css/navigation';
  import 'swiper/css/pagination';

  export let title: string = '';
  export let slides: { year: string; title: string; text: string; image: string }[] = [];

  let swiperInstance: any; // Renamed to avoid conflict with the DOM element class
  let activeIndex = 0;
  const swiperContainerId = 'swiper-container-' + Math.random().toString(36).substring(2);
  const slideIdBase = 'swiper-slide-' + Math.random().toString(36).substring(2) + '-';

  onMount(() => {
    swiperInstance = new Swiper(`#${swiperContainerId} .swiper`, { // Use ID for specificity
      modules: [Navigation, Pagination],
      direction: 'vertical',
      loop: false,
      speed: 900,
      pagination: {
        el: `#${swiperContainerId} .swiper-pagination`, // Use ID for specificity
        clickable: true,
        renderBullet: (index, className) => {
          const slide = slides[index];
          if (!slide) return `<span class="${className}"></span>`; // Should not happen with loop: false
          const year = slide.year || '';
          // Swiper usually handles aria-selected and tabindex on clickable bullets.
          // We add aria-label and aria-controls.
          return `<span class="${className}" role="tab" aria-label="Go to slide ${index + 1}, Year ${year}" aria-controls="${slideIdBase}${index}">
                    <span class="timeline-year-style">${year}</span>
                  </span>`;
        },
      },
      navigation: {
        nextEl: `#${swiperContainerId} .swiper-button-next`, // Use ID for specificity
        prevEl: `#${swiperContainerId} .swiper-button-prev`, // Use ID for specificity
      },
      on: {
        slideChange: (swiper) => { // swiper here is the instance
          activeIndex = swiper.activeIndex;
        }
      },
      observer: true, // Good for Svelte reactivity
      observeParents: true, // Good for Svelte reactivity
      breakpoints: {
        900: {
          direction: 'horizontal',
        }
      }
    });
    activeIndex = swiperInstance.activeIndex;
  });

  onDestroy(() => {
    if (swiperInstance) {
      swiperInstance.destroy(true, true);
    }
  });

  // Reactive statement to update aria-selected on bullets when activeIndex changes
  // Swiper might do this, but this is a fallback.
  $: if (typeof window !== 'undefined' && swiperInstance && swiperInstance.pagination && swiperInstance.pagination.bullets) {
    swiperInstance.pagination.bullets.forEach((bullet: HTMLElement, index: number) => {
      bullet.setAttribute('aria-selected', activeIndex === index ? 'true' : 'false');
    });
  }

</script>

<div class="timeline-slider" id={swiperContainerId} role="region" aria-label={title || 'Timeline of Events'} aria-roledescription="carousel">
  <div class="swiper"> {/* This is the main swiper container targeted by new Swiper() */}
    <div class="swiper-wrapper" aria-live="polite">
      {#each slides as slide, i (slide.year + slide.title)}
        <div
          class="swiper-slide"
          id="{slideIdBase}{i}"
          data-year={slide.year}
          role="group"
          aria-label={"Slide " + (i + 1) + ": " + slide.title + " (" + slide.year + ")"}
          aria-roledescription="slide"
          aria-hidden={activeIndex !== i ? 'true' : undefined}
        >
          {#if slide.image}
            <div class="slide-bg-blur" style="background-image: url('{slide.image}')" aria-hidden="true"></div>
            <img src={slide.image} alt={slide.title} class="slide-img" />
          {/if}
          {#if activeIndex === i}
            <div class="slide-content">
              <span class="slide-year timeline-year-style">{slide.year}</span>
              <h2 class="slide-title">{slide.title}</h2>
              <p class="slide-text">{slide.text}</p>
            </div>
          {/if}
        </div>
      {/each}
    </div>
    <div class="swiper-button-prev" role="button" aria-label="Previous Slide" aria-controls={swiperContainerId}></div>
    <div class="swiper-button-next" role="button" aria-label="Next Slide" aria-controls={swiperContainerId}></div>
    <div class="swiper-pagination" role="tablist" aria-label="Timeline Navigation"></div>
  </div>
</div>

<style>
.timeline-slider {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: #fff;
}
.swiper {
  width: 100vw;
  height: 100vh;
}
.swiper-slide {
  width: 100vw;
  height: 100vh;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  overflow: hidden;
  background: #222;
}
.slide-bg-blur {
  position: absolute;
  left: 0; top: 0; width: 100%; height: 100%;
  background-size: cover;
  background-position: center;
  filter: blur(18px) brightness(0.7);
  z-index: 0;
}
.slide-img {
  position: absolute;
  left: 0; top: 0; width: 100%; height: 100%;
  object-fit: contain;
  z-index: 1;
  pointer-events: none;
}
.slide-content {
  position: absolute;
  right: clamp(60px, 12vw, 200px); /* Adjusted right positioning */
  top: 50%;
  transform: translateY(-50%);
  z-index: 2;
  color: #fff;
  text-align: right;
  max-width: 800px; /* Adjusted max-width */
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 18px;
  background: rgba(0,0,0,0.25);
  border-radius: 18px;
  padding: 32px 40px 32px 32px;
  box-sizing: border-box;
}
/* Common style for year text */
.timeline-year-style {
  color: #ffd700;
  font-style: italic;
  font-weight: 700;
}

.slide-year { /* Applied to the year text in the main slide content - now mostly for structure, styling via .timeline-year-style */
  /* font-size is now handled by .slide-content .timeline-year-style directly or its clamp function */
}
/* Apply common year style to pagination bullets via the new class inside renderBullet */
.swiper-pagination-bullet span.timeline-year-style { /* Made selector more specific for direct styling */
  /* Styles for year text within bullets are now primarily through .timeline-year-style */
  /* Specific adjustments for bullet context if any, can be added here */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: 100%; /* Ensure it doesn't overflow bullet padding */
  display: inline-block; /* Allows max-width to work as expected */
}

.swiper-pagination-bullet-active .timeline-year-style {
  /* Styles for active year text within bullets */
  /* font-size adjustment for active bullet can be here if needed, or handled by .swiper-pagination-bullet-active overall */
}

.slide-title {
  font-size: clamp(30px, 4vw + 1rem, 54px); /* Smoother scaling */
  font-weight: 800;
  margin: 0;
  text-shadow: 0 2px 12px #000, 0 0 2px #fff;
}
.slide-text {
  font-size: clamp(16px, 2vw + 0.5rem, 22px); /* Smoother scaling */
  line-height: 1.5;
  margin: 0;
  text-shadow: 0 2px 12px #000, 0 0 2px #fff;
}
.slide-content .timeline-year-style { /* Styling for the large year text in the content block */
  font-size: clamp(30px, 4vw + 1rem, 54px); /* Smoother scaling */
  /* Other .timeline-year-style properties (color, font-style, font-weight) are inherited */
}

/* Swiper Pagination custom styling */
.timeline-slider .swiper-pagination { /* Increased specificity */
  position: absolute;
  right: 2vw; /* This remains for desktop, media query handles mobile */
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  /* font-style, font-weight, font-size for container text if any, not individual bullets */
  z-index: 3;
  gap: 10px;
  padding: 10px 10px;
  background: rgba(0,0,0,0.18);
  border-radius: 12px;
  box-shadow: 0 2px 8px 0 rgba(0,0,0,0.12);
}

.timeline-slider .swiper-pagination-bullet { /* Increased specificity */
  display: flex;
  align-items: center;
  width: auto;
  height: 32px;
  min-width: 60px;
  text-align: left;
  opacity: 1;
  background: transparent;
  margin: 0;
  padding: 0 8px 0 0;
  position: relative;
  font-size: 18px; /* Adjusted bullet font size */
  /* font-style and font-weight for bullet text now comes from .timeline-year-style */
  letter-spacing: 1px; /* Keep if desired, or adjust */
  transition: color 0.3s, font-weight 0.3s, font-size 0.3s, background 0.3s;
  cursor: pointer;
  text-shadow: 0 2px 8px #000, 0 0 2px #fff;
  min-width: 50px; /* Adjusted min-width */
  /* color for the bullet text itself is inherited or set by .timeline-year-style */
}

.timeline-slider .swiper-pagination-bullet-active { /* Increased specificity */
  /* color from .timeline-year-style */
  /* font-weight from .timeline-year-style */
  font-size: 22px; /* Adjusted active bullet font size */
  text-shadow: 0 2px 16px #000, 0 0 2px #fff;
}

.timeline-slider .swiper-pagination-bullet::before { /* Increased specificity */
  content: "";
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #ffd700; /* Dot color */
  margin-right: 12px;
  opacity: 0.5;
  transition: opacity 0.3s, transform 0.3s, background 0.3s;
}

.timeline-slider .swiper-pagination-bullet-active::before { /* Increased specificity */
  opacity: 1;
  transform: scale(1.2);
  background: #ffd700; /* Active dot color */
}

.timeline-slider .swiper-button-prev,
.timeline-slider .swiper-button-next { /* Increased specificity */
  background-color: #fff;
  border-radius: 50%;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.25);
  width: 64px; /* Base size, will be overridden in media query */
  height: 64px; /* Base size, will be overridden in media query */
  background-size: 40px 40px; /* Base size */
  background-repeat: no-repeat;
  background-position: center;
  opacity: 0.95;
  z-index: 20;
  border: 2px solid #007aff; /* Base border width */
  transition: box-shadow 0.2s, border 0.2s, width 0.2s, height 0.2s, background-size 0.2s; /* Added transitions for size changes */
}
.timeline-slider .swiper-button-prev:hover,
.timeline-slider .swiper-button-next:hover { /* Increased specificity */
  box-shadow: 0 4px 24px 0 rgba(0,0,0,0.35);
  border: 2.5px solid #0051a8;
  opacity: 1;
}
.timeline-slider .swiper-button-next { /* Increased specificity */
  /* transform: scaleX(-1); */
}
.timeline-slider .swiper-button-prev::after,
.timeline-slider .swiper-button-next::after { /* Increased specificity */
  display: none;
  content: none;
}
.timeline-slider .swiper-button-prev { /* Increased specificity */
  right: 5vw; /* Base positioning */
  left: auto;
  top: 15%; /* Base positioning */
  transform: rotate(90deg); /* Base transform */
}
.timeline-slider .swiper-button-next { /* Increased specificity */
  right: 5vw; /* Base positioning */
  left: auto;
  bottom: 15%; /* Base positioning */
  top: auto;
  transform: rotate(-90deg); /* Base transform */
}

@media (max-width: 900px) {
  .timeline-slider .swiper-pagination { /* Increased specificity */
    left: 0;
    right: 0;
    bottom: 0;
    top: auto;
    height: auto;
    width: 100vw;
    max-width: 100vw;
    flex-direction: row;
    justify-content: center;
    padding: 10px;
    background: rgba(0,0,0,0.5);
    border-radius: 0;
  }

  .timeline-slider .swiper-pagination-bullet { /* Increased specificity */
    font-size: 16px; /* Bullet container size */
    height: 24px;
    min-width: 40px;
  }
  /* .timeline-year-style within bullets will inherit or can be specifically styled for this breakpoint if needed */

  .timeline-slider .swiper-pagination-bullet-active { /* Increased specificity */
    font-size: 18px; /* Active bullet container size */
  }

  .slide-content {
    right: 0;
    left: 0;
    top: auto;
    bottom: 80px;
    transform: none;
    max-width: 100%;
    padding: 20px; /* Base padding for <900px */
    border-radius: 0;
    background: rgba(0,0,0,0.7);
    gap: 18px; /* Base gap for <900px */
  }

  /* Font sizes for .slide-title, .slide-text, .slide-content .timeline-year-style are handled by clamp() */
  /* No need to redefine them here unless clamp() max values are too large for this breakpoint */

  .timeline-slider .swiper-button-prev { /* Increased specificity */
    left: 20px;
    right: auto;
    bottom: 20px;
    top: auto;
    transform: none;
  }
  .timeline-slider .swiper-button-next { /* Increased specificity */
    right: 20px;
    left: auto;
    bottom: 20px;
    top: auto;
    transform: none;
  }
}

@media (max-width: 480px) {
  .slide-title {
    font-size: 24px;
  }

  .slide-text {
    font-size: 16px;
  }

  .slide-content .slide-year.timeline-year-style { /* Increased specificity */
    /* font-size: 24px; */ /* clamp() handles this */
  }

  .timeline-slider .swiper-pagination-bullet { /* Increased specificity */
    font-size: 14px; /* Bullet container font size */
    height: 20px;
    min-width: 32px;
  }
  /* .timeline-year-style within bullets will inherit or can be specifically styled for this breakpoint if needed */

  .timeline-slider .swiper-pagination-bullet-active { /* Increased specificity */
    font-size: 16px; /* Active bullet container font size */
  }

  .timeline-slider .swiper-button-prev,
  .timeline-slider .swiper-button-next {
    width: 52px; /* Reduced size for small screens */
    height: 52px; /* Reduced size for small screens */
    background-size: 30px 30px; /* Proportional background size */
    border-width: 1.5px; /* Slightly thinner border */
  }
  .slide-content { /* Specific adjustments for very small screens */
    gap: 12px; /* Reduced gap */
    padding: 16px; /* Reduced padding */
  }
}

/* Media query for short viewports (mobile landscape) */
@media (max-height: 500px) and (max-width: 900px) {
  .slide-content {
    bottom: 40px; /* Reduced bottom spacing */
    padding-top: 15px; /* Reduced vertical padding */
    padding-bottom: 15px; /* Reduced vertical padding */
    gap: 10px; /* Further reduced gap for very short screens */
  }
  .slide-title {
    font-size: clamp(20px, 3vw + 0.5rem, 28px); /* Further reduce title font for short screens */
  }
  .slide-text {
    font-size: clamp(12px, 1.5vw + 0.5rem, 14px); /* Further reduce text font for short screens */
    line-height: 1.3;
  }
  .slide-content .timeline-year-style {
     font-size: clamp(20px, 3vw + 0.5rem, 28px); /* Further reduce year font for short screens */
  }
  .timeline-slider .swiper-pagination {
    padding: 5px; /* Reduce pagination padding */
  }
  .timeline-slider .swiper-pagination-bullet {
    height: 20px; /* Reduce bullet height */
    min-width: 30px; /* Reduce bullet min-width */
    font-size: 12px; /* Reduce bullet font size */
  }
   .timeline-slider .swiper-pagination-bullet-active {
    font-size: 14px; /* Reduce active bullet font size */
  }
  .timeline-slider .swiper-button-prev,
  .timeline-slider .swiper-button-next {
    width: 44px; /* Smaller buttons for short screens */
    height: 44px;
    background-size: 25px 25px;
    bottom: 10px; /* Adjust button position */
  }
  .timeline-slider .swiper-button-prev {
    left: 10px;
  }
  .timeline-slider .swiper-button-next {
    right: 10px;
  }
}
</style>