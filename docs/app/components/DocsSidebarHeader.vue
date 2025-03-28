<script setup lang="ts">
import { createPerformanceMeasure } from '~/utils/perf'

const activeModule = useModule((await useStats()).value.modules)
const searchTerm = ref('')
const perf = createPerformanceMeasure()
const _nav = await useDocsNav(activeModule)
perf('docs-nav', _nav.value?.navTiming)
perf('docs-search', _nav.value?.searchTiming)
const nav = computed(() => {
  const { top = [], bottom = [] } = _nav.value?.nav.value || {}
  return {
    top,
    bottom,
  }
})
const files = computed(() => _nav.value?.files.value || [])
</script>

<template>
  <div>
    <div class="isolate hidden md:block md:sticky pt-8 rounded bg-white dark:bg-neutral-900 top-0 z-1">
      <ModuleCard :key="activeModule.slug" :module="activeModule" :version="false" class="mb-2" />
    </div>
    <div class="block md:hidden flex items-center gap-1 font-bold mb-3">
      <UIcon v-if="activeModule.icon" dynamic :name="activeModule.icon" class="text-blue-500 dark:text-blue-300" />{{ activeModule.label }}
    </div>
    <div class="flex items-center gap-1 mb-3">
      <USelectMenu :search-input="false" size="sm" :model-value="activeModule.version" :items="activeModule.versions?.map(v => ({ label: v, disabled: v !== activeModule.version }))" class="md:w-full" />

      <UTooltip
        text="Search"
        :kbds="['meta', 'K']"
      >
        <UContentSearchButton class=" opacity-70 hover:opacity-100" />
      </UTooltip>

      <UButton
        title="GitHub"
        aria-label="GitHub"
        :to="`https://github.com/${activeModule.repo}`"
        target="_blank"
        color="neutral"
        variant="ghost"
        class="transition opacity-70 hover:opacity-100"
        icon="i-carbon-logo-github"
      />
      <UButton
        title="NPM"
        aria-label="NPM"
        :to="`https://npmjs.com/package/${activeModule.npm}`"
        target="_blank"
        color="neutral"
        variant="ghost"
        class="transition opacity-70 hover:opacity-100"
        icon="i-carbon-logo-npm"
      />
    </div>
    <nav aria-title="Documentation Navigation">
      <ContentNavigation as="div" class="mb-5" default-open :collapsible="false" :navigation="nav.top" highlight :ui="{ listWithChildren: 'sm:ml-0 my-10' }">
        <template #link-leading="{ link, active }">
          <div v-if="link.icon" class="rounded-md p-1 inline-flex ring-inset ring-1 bg-neutral-100/50 dark:bg-neutral-800/50 ring-neutral-300 dark:ring-neutral-700 group-hover:bg-primary group-hover:ring-primary group-hover:text-background" :class="active ? 'dark:bg-teal-700' : ''">
            <UIcon :name="link.icon" class="w-4 h-4 text-(--ui-text-muted) dark:text-sky-200" />
          </div>
        </template>
      </ContentNavigation>
      <div class="bg-neutral-800 h-[1px] my-5 mr-5" />
      <ContentNavigation as="div" default-open :collapsible="false" :navigation="nav.bottom" highlight :ui="{ listWithChildren: 'sm:ml-0 my-10' }">
        <template #link="{ link }">
          <div v-if="!link.html" class="flex items-center gap-2">
            <UIcon v-if="link.icon" :name="link.icon" class="w-4 h-4 text-(--ui-text-muted) dark:text-sky-200" />
            <div :class="link.children?.length ? 'text-sm font-bold' : ''">
              {{ link.title }}
            </div>
          </div>
          <div v-else v-html="link.title" />
        </template>
      </ContentNavigation>
    </nav>
    <ClientOnly>
      <LazyUContentSearch
        v-model:search-term="searchTerm"
        :files="files"
        :navigation="[{ title: 'Getting Started', _path: `/docs/${activeModule.slug}/getting-started`, path: `/docs/${activeModule.slug}/getting-started`, children: nav.top }, ...nav.bottom]"
        :fuse="{ resultLimit: 42 }"
      />
    </ClientOnly>
  </div>
</template>
