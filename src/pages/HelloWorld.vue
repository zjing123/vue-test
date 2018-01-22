<template>
  <div>
   <vue-tags-input
     class="add"
     v-model="tag"
     :tags="tags"
     @tags-changed="newTags => tags = newTags"
     :allowEditTags="true"
   />
 </div>
</template>

<script>
import VueTagsInput from '@johmun/vue-tags-input';

export default {
  components: {
    VueTagsInput,
  },
  data() {
    return {
      tag: '',
      tags: [],
    }
  },
  watch : {
    'tags' () {
      console.log(this.tags)
    }
  }
};
</script>
