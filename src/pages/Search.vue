<template>
  <div class="bg-primary flex-1">
    <div class="container mx-auto px-4">
      <div class="text-3xl font-bold mt-16">
        Tag search for "{{ tag }}"
      </div>
      <div class="grid lg:grid-cols-2 gap-5 mt-8 mb-16">
        <entry-tile
          v-for="(entry, index) in searchEntries"
          :key="entry.url"
          :entry="entry"
          :index="index"
          :change-bookmarks="changeBookmarks"
        />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, watchEffect } from 'vue'
import useQuery from '/@/use/query'
import apis, {EntryDetail} from '/@/lib/apis'
import EntryTile from '/@/components/EntryTile.vue'

export default defineComponent({
  name: 'Search',
  components: {
    EntryTile
  },
  setup() {
    const tag = useQuery('tag')
    // const title = useQuery('title')
    // const text = useQuery('text')
    
    const searchEntries = ref<EntryDetail[]>([])
    const changeBookmarks = (index: number) => {
      searchEntries.value[index].count += searchEntries.value[index].isBookmark? -1: 1
      searchEntries.value[index].isBookmark = !searchEntries.value[index].isBookmark
    }
    watchEffect(async () => {
      const res = await apis.getSearchEntry(tag.value)
      searchEntries.value = res.data
    })
    return {
      tag, searchEntries, changeBookmarks
    }
  }
})
</script>
