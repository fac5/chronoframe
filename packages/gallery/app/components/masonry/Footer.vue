<script lang="ts" setup>
defineProps<{
  summary: string
  photos: Photo[]
}>()

const copyrightName = computed(
  () => getSetting('app:author') || getSetting('app:title'),
)

const projectHref = 'https://github.com/fac5/chronoframe'

const filings = [
  {
    label: '黔ICP备案 2022009864号-1',
    href: 'https://beian.miit.gov.cn/',
    icon: 'tabler:building-bank',
  },
  {
    label: '黔公安备案 52262702000070',
    href: 'https://beian.mps.gov.cn/#/query/webSearch?code=52262702000070',
    icon: 'mdi:police-badge-outline',
  },
]
</script>

<template>
  <footer class="masonry-footer">
    <div class="masonry-footer-pattern" />
    <MasonryFooterGlow :photos />
    <div class="masonry-footer-timeline">
      <span class="masonry-footer-timeline-rail" />
      <span class="masonry-footer-timeline-node" />
      <span class="masonry-footer-timeline-rail" />
    </div>
    <div class="masonry-footer-content">
      <p
        v-if="summary"
        class="text-xs font-medium text-neutral-600 dark:text-neutral-400"
      >
        {{ summary }}
      </p>
      <p class="text-[11px] text-neutral-500 dark:text-neutral-400">
        <template v-if="copyrightName">
          © {{ $dayjs().format('YYYY') }} {{ copyrightName }}
          <span
            class="opacity-40"
            aria-hidden="true"
            >·</span
          >
        </template>
        <a
          :href="projectHref"
          target="_blank"
          rel="noopener noreferrer"
          class="transition-colors hover:text-neutral-800 dark:hover:text-neutral-200"
        >
          Powered by <span class="font-semibold">ChronoFrame</span>
        </a>
      </p>
      <div
        class="flex flex-wrap items-center justify-center gap-x-3 gap-y-1.5 pt-0.5 text-[11px] leading-5"
      >
        <a
          v-for="filing in filings"
          :key="filing.href"
          :href="filing.href"
          target="_blank"
          rel="noopener noreferrer"
          class="inline-flex items-center gap-1 text-neutral-500/80 underline decoration-neutral-400/40 underline-offset-2 transition-colors hover:text-neutral-800 hover:decoration-neutral-600 dark:text-neutral-400 dark:decoration-neutral-500/40 dark:hover:text-neutral-200 dark:hover:decoration-neutral-300"
        >
          <Icon
            :name="filing.icon"
            mode="svg"
            class="size-3 shrink-0"
            aria-hidden="true"
          />
          {{ filing.label }}
        </a>
      </div>
    </div>
  </footer>
</template>

<style scoped>
.masonry-footer {
  position: relative;
  display: flex;
  flex: 1 0 auto;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  padding: 3.5rem 1rem 2rem;
  color: var(--ui-text-muted, #737373);
}

/* Extends upward into the ragged tail of the wall, behind the photos. */
.masonry-footer-pattern {
  position: absolute;
  inset: -22rem 0 0;
  z-index: -1;
  pointer-events: none;
  background-image: radial-gradient(
    circle at center,
    currentColor 1px,
    transparent 1.5px
  );
  background-size: 18px 18px;
  opacity: 0.22;
  mask-image: linear-gradient(to bottom, transparent, #000 70%);
}

/* A film-ruler axis: ticked rails fade out from a centred ring node. */
.masonry-footer-timeline {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  width: min(20rem, 80%);
  margin-bottom: 1.25rem;
  opacity: 0.6;
}

.masonry-footer-timeline-rail {
  flex: 1;
  height: 7px;
  background:
    linear-gradient(currentColor, currentColor) center / 100% 1px no-repeat,
    repeating-linear-gradient(90deg, currentColor 0 1px, transparent 1px 10px)
      center / 100% 5px no-repeat;
}

.masonry-footer-timeline-rail {
  mask-image: linear-gradient(to right, transparent, #000);
}

.masonry-footer-timeline-rail:last-child {
  transform: scaleX(-1);
}

.masonry-footer-timeline-node {
  display: grid;
  flex: none;
  place-items: center;
  width: 12px;
  height: 12px;
  border: 1px solid currentColor;
  border-radius: 9999px;
}

.masonry-footer-timeline-node::after {
  width: 4px;
  height: 4px;
  content: '';
  background: currentColor;
  border-radius: 9999px;
}

.masonry-footer-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.375rem;
  text-align: center;
}
</style>
