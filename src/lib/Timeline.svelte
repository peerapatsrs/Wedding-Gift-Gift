<script lang="ts">
  import { onMount } from 'svelte';
  import Swiper from 'swiper';
  import { Navigation, Pagination } from 'swiper/modules';
  import 'swiper/css';
  import 'swiper/css/navigation';
  import 'swiper/css/pagination';

  export let title: string = '';
  export let slides: { year: string; title: string; text: string; image: string }[] = [];

  let swiper: any;
  let activeIndex = 0;

  $: if (typeof window !== 'undefined') {
    onMount(() => {
      swiper = new Swiper('.swiper', {
        modules: [Navigation, Pagination],
        direction: 'vertical',
        loop: false,
        speed: 900,
        pagination: {
          el: '.swiper-pagination',
          clickable: true,
          renderBullet: (index, className) => {
            const year = slides[index]?.year || '';
            return `<span class="${className}" style="color:#ffd700;font-style:italic;font-weight:700;"><span class="slide-year" style="color:#ffd700;font-style:italic;font-weight:700;">${year}</span></span>`;
          },
        },
        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
        },
        on: {
          slideChange: () => {
            activeIndex = swiper.activeIndex;
          }
        },
        breakpoints: {
          900: {
            direction: 'horizontal',
          }
        }
      });
      activeIndex = swiper.activeIndex;
    });
  }
</script>

<div class="timeline-slider">
  <div class="swiper">
    <div class="swiper-wrapper">
      {#each slides as slide, i}
        <div class="swiper-slide" style="background-image: url({slide.image})" data-year={slide.year}>
          {#if activeIndex === i}
            <div class="slide-content">
              <span class="slide-year">{slide.year}</span>
              <h2 class="slide-title">{slide.title}</h2>
              <p class="slide-text">{slide.text}</p>
            </div>
          {/if}
        </div>
      {/each}
    </div>
    <div class="swiper-button-prev"></div>
    <div class="swiper-button-next"></div>
    <div class="swiper-pagination"></div>
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
  background-size: cover;
  background-position: center center;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
.slide-content {
  position: absolute;
  right: 12vw;
  top: 50%;
  transform: translateY(-50%);
  z-index: 2;
  color: #fff;
  text-align: right;
  max-width: 900px;
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
.slide-year,
.swiper-pagination-bullet,
.swiper-pagination-bullet-active {
  color: #ffd700 !important;
  font-style: italic;
  font-weight: 700;
}
.slide-title {
  font-size: 54px;
  font-weight: 800;
  margin: 0;
  text-shadow: 0 2px 12px #000, 0 0 2px #fff;
}
.slide-text {
  font-size: 22px;
  line-height: 1.5;
  margin: 0;
  text-shadow: 0 2px 12px #000, 0 0 2px #fff;
}
.swiper-pagination {
  position: absolute !important;
  right: 2vw !important;
  top: 0 !important;
  height: 100vh !important;
  display: flex !important;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  font-style: italic;
  font-weight: 700;
  font-size: 18px;
  z-index: 3;
  gap: 10px;
  padding: 10px 10px;
  background: rgba(0,0,0,0.18);
  border-radius: 12px;
  box-shadow: 0 2px 8px 0 rgba(0,0,0,0.12);
}
.swiper-pagination-bullet {
  color: #ffd700;
  font-weight: 700;
  display: flex;
  align-items: center;
  width: auto;
  height: 32px;
  min-width: 60px;
  text-align: left;
  opacity: 1;
  background: transparent;
  margin: 0 !important;
  padding: 0 8px 0 0;
  position: relative;
  font-size: 22px;
  font-style: italic;
  letter-spacing: 1px;
  transition: color 0.3s, font-weight 0.3s, font-size 0.3s, background 0.3s;
  cursor: pointer;
  text-shadow: 0 2px 8px #000, 0 0 2px #fff;
}
.swiper-pagination-bullet-active {
  color: #ffd700;
  font-weight: 700;
  font-size: 26px;
  text-shadow: 0 2px 16px #000, 0 0 2px #fff;
}
.swiper-pagination-bullet::before {
  content: "";
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #ffd700;
  margin-right: 12px;
  opacity: 0.5;
  transition: opacity 0.3s, transform 0.3s, background 0.3s;
}
.swiper-pagination-bullet-active::before {
  opacity: 1;
  transform: scale(1.2);
  background: #ffd700;
}
.swiper-button-prev, .swiper-button-next {
  background: #fff !important;
  border-radius: 50%;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.25);
  width: 64px;
  height: 64px;
  background-size: 40px 40px !important;
  background-repeat: no-repeat !important;
  background-position: center !important;
  opacity: 0.95;
  z-index: 20;
  border: 2px solid #007aff;
  transition: box-shadow 0.2s, border 0.2s;
}
.swiper-button-prev:hover, .swiper-button-next:hover {
  box-shadow: 0 4px 24px 0 rgba(0,0,0,0.35);
  border: 2.5px solid #0051a8;
  opacity: 1;
}
.swiper-button-next {
  transform: scaleX(-1);
}
.swiper-button-prev::after,
.swiper-button-next::after {
  display: none !important;
  content: none !important;
}
.swiper-button-prev {
  right: 5vw;
  left: auto;
  top: 15%;
  transform: rotate(90deg);
}
.swiper-button-next {
  right: 5vw;
  left: auto;
  bottom: 15%;
  top: auto;
  transform: rotate(-90deg);
}
.swiper-pagination-bullet span,
.swiper-pagination-bullet-active span,
.slide-year {
  color: #ffd700 !important;
  font-style: italic;
  font-weight: 700;
}
.slide-content .slide-year {
  font-size: 54px;
}
@media (max-width: 900px) {
  .swiper-pagination {
    position: absolute !important;
    left: 0 !important;
    right: 0 !important;
    bottom: 0 !important;
    top: auto !important;
    height: auto !important;
    width: 100vw !important;
    max-width: 100vw;
    flex-direction: row !important;
    justify-content: center;
    padding: 10px;
    background: rgba(0,0,0,0.5);
    border-radius: 0;
  }

  .swiper-pagination-bullet {
    font-size: 16px;
    height: 24px;
    min-width: 40px;
  }

  .swiper-pagination-bullet-active {
    font-size: 18px;
  }

  .slide-content {
    right: 0;
    left: 0;
    top: auto;
    bottom: 80px;
    transform: none;
    max-width: 100%;
    padding: 20px;
    border-radius: 0;
    background: rgba(0,0,0,0.7);
  }

  .slide-title {
    font-size: 32px;
  }

  .slide-text {
    font-size: 18px;
  }

  .slide-content .slide-year {
    font-size: 32px;
  }

  .swiper-button-prev {
    left: 20px;
    right: auto;
    bottom: 20px;
    top: auto;
    transform: none;
  }
  .swiper-button-next {
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

  .slide-content .slide-year {
    font-size: 24px;
  }

  .swiper-pagination-bullet {
    font-size: 14px;
    height: 20px;
    min-width: 32px;
  }

  .swiper-pagination-bullet-active {
    font-size: 16px;
  }
}
</style> 