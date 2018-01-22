<template>
  <div class="tags-textarea">
   <input-tags :tags="tags">
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
      tags: [{text:'abd'}, {text:'efg'}],
    }
  },
  watch : {
    'tags' () {
      console.log(this.tags)
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
