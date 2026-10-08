<script setup lang="ts">
/**
 * slidev-addon-autofit
 *
 * Scales a slide down until its content fits into the layout – like PowerPoint's "shrink text on overflow".
 * It drives Slidev's own per-slide zoom (the `--slidev-slide-zoom-scale` variable that `zoom:` in the frontmatter
 * sets), so layouts that already handle `zoom` behave exactly as with a manual value.
 *
 * Fitting runs synchronously inside ResizeObserver / MutationObserver callbacks, i.e. after layout but before the
 * browser paints. The slide therefore appears at its final size immediately – there is no visible shrinking.
 *
 * Options (headmatter for all slides, frontmatter per slide):
 *   autofit: false            # disable
 *   autofit: { min: 0.5 }     # smallest allowed scale (default 0.4)
 * A manual `zoom:` in the frontmatter always takes precedence.
 */
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
import { configs, useSlideContext } from '@slidev/client'

interface AutofitOptions {
  min?: number
}

const ZOOM_VARIABLE = '--slidev-slide-zoom-scale'
// Set permanently on every auto-fitted slide page; see the global style below.
const ENABLED_CLASS = 'slidev-autofit'
const DEFAULT_MIN = 0.4
const SEARCH_STEPS = 8

const { $frontmatter } = useSlideContext()
const anchor = ref<HTMLElement>()

const options = computed<AutofitOptions | false>(() => {
  const global = (configs as Record<string, unknown>).autofit
  const local = $frontmatter.autofit
  if (global === false || local === false)
    return false

  return {
    ...(typeof global === 'object' ? global as AutofitOptions : {}),
    ...(typeof local === 'object' ? local as AutofitOptions : {}),
  }
})

// A manual zoom in the frontmatter wins; Slidev applies it itself.
const enabled = computed(() => options.value !== false && $frontmatter.zoom === undefined)

let page: HTMLElement | undefined
let resizeObserver: ResizeObserver | undefined
let mutationObserver: MutationObserver | undefined
let fitting = false

function layoutElement(): HTMLElement | null {
  return page?.querySelector<HTMLElement>(':scope > .slidev-layout') ?? null
}

function fits(layout: HTMLElement): boolean {
  return layout.scrollHeight <= layout.clientHeight + 1 && layout.scrollWidth <= layout.clientWidth + 1
}

function setScale(scale: number) {
  if (scale >= 1)
    page!.style.removeProperty(ZOOM_VARIABLE)
  else
    page!.style.setProperty(ZOOM_VARIABLE, String(scale))
}

/**
 * Finds the largest scale at which the content fits. Reading scrollHeight forces a synchronous reflow, so every
 * step is measured with the line wrapping of that scale – nothing is painted in between.
 */
function fit() {
  const layout = layoutElement()
  if (!page || !layout || fitting || !enabled.value)
    return
  // Hidden slides (v-show) have no size; they are fitted when they become visible.
  if (layout.clientHeight === 0)
    return

  fitting = true
  try {
    setScale(1)
    if (fits(layout))
      return

    const min = (options.value as AutofitOptions).min ?? DEFAULT_MIN
    let low = min
    let high = 1
    for (let i = 0; i < SEARCH_STEPS; i++) {
      const mid = (low + high) / 2
      setScale(mid)
      if (fits(layout))
        low = mid
      else
        high = mid
    }
    setScale(Math.floor(low * 100) / 100)
  }
  finally {
    fitting = false
  }
}

function observeChildren() {
  const layout = layoutElement()
  if (!layout || !resizeObserver)
    return
  resizeObserver.observe(layout)
  for (const child of Array.from(layout.children))
    resizeObserver.observe(child)
}

onMounted(() => {
  page = anchor.value?.parentElement ?? undefined
  if (!page || !enabled.value)
    return

  // Must be present before the first scale change and never toggled: every style change caused while measuring
  // would otherwise start a transition of the scale.
  page.classList.add(ENABLED_CLASS)

  resizeObserver = new ResizeObserver(() => fit())
  mutationObserver = new MutationObserver(() => {
    observeChildren()
    fit()
  })

  // The slide component is loaded asynchronously, so the layout may not exist yet: watch the whole slide page.
  mutationObserver.observe(page, { childList: true, subtree: true, characterData: true })
  observeChildren()
  fit()
})

onBeforeUnmount(() => {
  page?.classList.remove(ENABLED_CLASS)
  resizeObserver?.disconnect()
  mutationObserver?.disconnect()
})
</script>

<template>
  <span ref="anchor" class="slidev-autofit-anchor" aria-hidden="true" />
</template>

<style scoped>
.slidev-autofit-anchor {
  display: none;
}
</style>

<style>
/*
 * Slidev's slide transitions use `transition: all`. Without this rule the scale set while a slide enters would be
 * animated as well (scale, width, height), so the slide would visibly shrink during the transition. Limit the
 * transition to the properties the built-in transitions actually move. Custom transitions that animate `scale`,
 * `width` or `height` of the slide page won't animate those properties on auto-fitted slides.
 */
.slidev-page.slidev-autofit {
  transition-property: translate, transform, opacity, rotate, filter, clip-path !important;
}
</style>
