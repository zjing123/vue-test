<template>
  <div class="tags-textarea">
   <input-tags :tags="tags" @tags-changed="newTags => tags = newTags" @before-deleting-tag="beforeDelete">
   </input-tags>
 </div>
</template>

<script>
import InputTags from '@/components/InputTags';

export default {
  components: {
    InputTags,
  },
  data() {
    return {
      tag: '',
      tags: [1,2,3,4,5,6,7,8,9,10,11,12,12,14,15,165,161,1241,15151,151],
    }
  },
  methods : {
    beforeDelete (paras) {
      console.log(paras)
    }
  },
  watch : {
    'tags' () {
      console.log('pareent',this.tags)
    }
  }
};
</script>

<style>
  .tags-textarea {
    border: 1px solid orange;
    height: 200px;
    text-align: left;
    padding:4px;
  }
</style>
