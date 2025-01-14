<template>
  <div class="bg-primary flex-1">
    <div
      v-if="entry"
      class="container mx-auto px-12"
    >
      <div class="mt-12 h-36 flex max-w-screen-lg mx-auto">
        <button
          class="focus:outline-none h-18 w-18 my-auto rounded-lg bg-white flex"
          @click="changeBookmark"
        >
          <img
            class="h-10 w-10 m-auto"
            :src="BookmarkLogo"
          >
        </button>
        <div class="flex-1 flex flex-col justify-center mx-4">
          <div class="h-14 text-xl font-bold mb-1 line-clamp-2 hover:text-accent duration-200">
            <a
              :href="entry.url"
              target="_blank"
            >
              {{ entry.title }}
            </a>
          </div>
          <a
            :href="entry.url"
            target="_blank"
          >
            <div class="ml-1 text-secondary flex pr-2">
              <img
                class="mr-1 h-4 my-auto"
                :src="LinkIcon"
              >
              {{ entry.url }}
            </div>
          </a>
        </div>
        <div class="ml-auto hidden md:block">
          <img
            class="h-36 min-w-36"
            :src="entry.thumbnail"
            alt=""
            onerror="this.onerror = null; this.src='';"
          >
        </div>
      </div>
      <div class="mx-8 mt-12">
        <!-- <div class="flex mb-1">
          <img
            class="mr-2 h-4 my-auto"
            :src="TagIcon"
          >
          タグ
        </div> -->
        <div class="bg-white flex flex-col flex-wrap px-6 py-5 rounded-lg max-w-screen-lg mx-auto">
          <div class="flex flex-wrap">
            <edit-tag
              v-for="tag in entry.tags"
              :key="tag"
              :name="tag"
              :entry-url="url"
            />
            <!-- <button class="ml-auto my-auto focus:outline-none">
              <img
                class="w-6 h-6"
                :src="AddButton"
              >
            </button> -->
          </div>
          <form
            class="py-1 px-2 mt-3 flex"
            @submit="onSubmit"
          >
            <img
              class="mr-2 h-4 my-auto"
              :src="TagIcon"
            >
            <input
              v-model="addValue"
              class="outline-none border-b-2 border-transparent duration-100 hover:border-gray-300 focus:border-gray-300"
              type="text"
              placeholder="タグを追加"
            >
            <button 
              v-if="addValue.length > 0"
              class="ml-2 my-auto border-none focus:outline-none"
            >
              <img
                :src="AddButton"
              >
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, ref, watch, watchEffect } from 'vue'
import useQuery from '/@/use/query'
import apis, {EntryDetail} from '/@/lib/apis'
import EditTag from '/@/components/EditTag.vue'
import BookmarkEmpty from '/@/assets/bookmark_empty.svg'
import BookmarkFill from '/@/assets/bookmark_fill.svg'
import LinkIcon from '/@/assets/link.svg'
import TagIcon from '/@/assets/tag.svg'
import AddButton from '/@/assets/addbutton.svg'
import sha256 from 'js-sha256'

export default defineComponent({
  name: 'Entry',
  components: {
    EditTag
  },
  props: {
    value: {
      type: String,
      default: ''
    },
  },
  setup(props) {
    const url = useQuery('url')
    const addValue = ref(props.value)
    watch(
      () => props.value,
      v => {
        addValue.value = v
      }
    )
    const entry = ref<EntryDetail>()
    watchEffect(async () => {
      if (url.value == undefined) return
      const entryId = sha256.sha256(decodeURI(url.value))
      try {
        const res = await apis.getEntryDetail(entryId)
        entry.value = res.data
      } catch (e) {
        console.error(url.value, e)
      }
    })

    const BookmarkLogo = computed(() => 
      entry.value?.isBookmark ? BookmarkFill : BookmarkEmpty
    )

    const onSubmit = async (e: Event) => {
      e.preventDefault()
      if (addValue.value === '' || url.value === undefined) return
      const entryId = sha256.sha256(decodeURI(url.value))
      try {
        await apis.postTags(entryId, addValue.value)
        const res = await apis.getEntryDetail(entryId)
        entry.value = res.data
      } catch (e) {
        console.error(url.value, e)
      }
      addValue.value = ''
    }

    const changeBookmark = async(e: Event) => {
      e.preventDefault()
      if (url.value === undefined || entry.value === undefined) return
      const entryId = sha256.sha256(decodeURI(url.value))
      if (entry.value.isBookmark === true) {
        // ブックマークの削除
        try {
          entry.value.isBookmark = false
          const res = await apis.deleteBookmark(entryId)
          if (res.status !== 204){
            entry.value.isBookmark = true
          }
        } catch (e) {
          entry.value.isBookmark = true
          console.error(url.value, e)
        }
      }else if (entry.value.isBookmark === false) {
        // ブックマークの追加
        try {
          entry.value.isBookmark = true
          const res = await apis.putEntry({
            url: url.value
          })
          if (res.status !== 201) {
            entry.value.isBookmark = false
          }
        } catch (e) {
          entry.value.isBookmark = false
          console.error(url.value, e)
        }
      }
    }
    
    return {
      url, entry, BookmarkLogo, LinkIcon, TagIcon, EditTag, AddButton, onSubmit, addValue, changeBookmark
    }
  }
})
</script>
