<template>
  <div>
    <span class="tag" v-for="(tag, index) in tagsCopy" :key="index">
      <span class="content" ref="tagCenter">
        <span
          @click="performEditTag(index, tag)"
          :class="{ hidden: tagsEditStatus[index] }">{{ tag.text }}</span>
        <input
          type="text"
          class="tag-input"
          v-if="tagsEditStatus[index]"
          :maxlength="maxlength"
          size="1"
          ref="tagInput"
          v-model="tag.text"
          @input="createdChangedTag(index, tag)"
          @blur="cancelEdit(index)"
          @keydown.enter="performSaveTag(index, tag)"
        />
      </span>
      <span class="actions">
        <i
          @click="cancelEdit(index)"
          v-if="!$scopedSlots.tagActions"
          v-show="tagsEditStatus[index]"
          class="icon-undo">
        </i>
        <i
          @click="performDeleteTag(index, tag)"
          v-if="!$scopedSlots.tagActions"
          v-show="!tagsEditStatus[index]"
          class="icon-close">
        </i>
      </span>
    </span>
    <span>
      <input
        class="new-tag-input"
        v-bind="$attrs"
        type="text"
        ref="newTagInput"
        :placeholder="placeholder"
        v-model="newTag"
        @keydown.enter="performAddTags(newTag)"
        @keydown.8="invokeDelete"
        @keydown.38="selectItem($event, 'before')"
        @keydown.40="selectItem($event, 'after')"
        @input="updateNewTag"
        @focus="focused=true"
        :disabled="disabled"
        />
    </span>
  </div>
</template>

<script>
import { createTag, createTags, createClasses } from './create-tags';

export default {
  name: 'input-tags',
  props: {
    tags: {
      type: Array,
      default: []
    },
    placeholder: {
      type: String,
      default: ''
    },
    readOnly: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    maxlength: {
      type: Number,
      default: 1
    },
    separators: {
      type: Array,
      default: () => [';'],
      validator (value) {
        return !value.some(s => {
          const invalidType = typeof s !== 'string';
          if(invalidType) console.warn('[component][input-tags][separators]Separators must be type of string.Found:', s);
          return invalidType;
        })
      }
    },
    validation: {
      type: Array,
      default: () => [],
      validator (value) {
        return !value.some(v => {
          const missingRule = !v.rule;
          if(missingRule) console.warn('[component][input-tags][validation]Property "rule" is missing:', v);

          const invalidRule = v.rule && typeof v.rule !== 'string';
          if(invalidRule) console.warn('[component][input-tags][validation]A rule must be type of string. Found:', v);

          const missingType = !v.type;
          if(missingType) console.warn('[component][input-tags][validation]Property "type" is missing:', v);

          const invalidType = v.type && typeof v.type !== 'string';
          if(invalidType) console.warn('[component][input-tags][validation]Property "type" must be type of sting. Found:', v);

          return invalidRule || missingRule || invalidType || missingType;
        });
      }
    },
    isTrim: {
      type: Boolean,
      default: true
    },
    avoidAddingDuplicates: {
      type: Boolean,
      default: true
    },
    allowEditTags: {
      type: Boolean,
      default: true
    },
    maxTag: {
      type: Number,
    }
  },
  data () {
    return {
      newTag: null,
      tagsCopy: [],
      tagsEditStatus: null,
    }
  },
  methods: {
    performAddTags (tag) {
      if(this.disabled) return;
      if(typeof tag === 'string' && tag.length === 0) return;
      let tags = [];
      if(typeof tag === 'object') tags = [tag];
      if(typeof tag === 'string') tags = this.createTagTexts(tag);
      tags.forEach(tag => {
        tag = createTag(tag, this.tags, this.validation, false);
        if(!this._events['before-adding-tag']) this.addTag(tag);
        this.$emit('before-adding-tag', {
          tag,
          addTag: (goOn) => this.addTag(tag, goOn),
        })
      })
    },
    addTag(tag, goOn) {
      if(goOn === false) return;
      const maxItemReached  = this.maxTags && this.maxTags === this.tagsCopy.length;
      if(maxItemReached) return this.$emit('max-tags-reached');
      const dup = this.avoidAddingDuplicates && this.tagsCopy.map(t => t.text).includes(tag.text);
      if(dup) {
        this.newTag = '';
        return this.$emit('adding-duplicate', tag);
      }
      if(!tag.valid && this.hasForbiddingAddRule(tag.tiClasses)) return;
      this.tagsCopy.push(tag);
      this.newTag = '';
      this.$emit('input', '');
      this.$emit('tags-changed', this.tagsCopy);
      this.$emit('add-tag', tag);
    },
    performEditTag (index) {
      if(!this.allowEditTags) return;
      if(!this._events['before-editing-tag']) this.editTag(index);
      this.$emit('before-editing-tag', {
        index,
        tag: this.tagsCopy[index],
        editTag: (goOn) => this.editTag(index, goOn)
      });
    },
    editTag (index, goOn) {
      if(!this.allowEditTags || goOn === false) return;
      this.toggleEdit(index);
      this.focus(index);
    },
    toggleEdit (index) {
      if(!this.allowEditTags || this.disabled) return;
      this.$set(this.tagsEditStatus, index, !this.tagsEditStatus[index]);
    },
    focus (index) {
      this.$nextTick(() => {
        const el = this.$refs.tagCenter[index].querySelector('input.tag-input');
        console.log(el)
        if(el) el.focus();
      });
    },
    createTagTexts (string) {
      const regex = new RegExp(this.separators.map(s => this.quote(s)).join('|'));
      return string.split(regex).map(text => {
        return { text };
      })
    },
    quote (regex) {
      return regex.replace(/([()[{*+.$^\\|?])/g, '\\$1');
    },
    hasForbiddingAddRule (tagClasses) {
      return tagClasses.some(type => {
        const rule = this.validation.find(rule => type === rule.type);
        return rule ? rule.disableAdd : false;
      });
    },
    updateNewTag () {

    },
    initTags () {
      this.tagsCopy = createTags(this.tags, this.validation);
      this.tagsEditStatus = this.clone(this.tags).map(() => false);
    },
    clone (items) {
      return JSON.parse(JSON.stringify(items));
    }
  },
  watch: {
    tags: {
      handler () {
        this.initTags();
      },
      deep: true
    }
  },
  created () {
    this.initTags();
  }

};
</script>
<style scoped lang="scss">
  @import './styles/colors';

  @font-face {
    font-family: 'icomoon';
    src:  url('../assets/fonts/custom/icomoon.eot?7grlse');
    src:  url('../assets/fonts/custom/icomoon.eot?7grlse#iefix') format('embedded-opentype'),
          url('../assets/fonts/custom/icomoon.ttf?7grlse') format('truetype'),
          url('../assets/fonts/custom/icomoon.woff?7grlse') format('woff');
    font-weight: normal;
    font-style: normal;
  }

  [class^="icon-"],[class*=" icon-"] {
    font-family: 'icomoon' !important;
    speak:none;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  .icon-check:before {
    content: "\e902";
  }
  .icon-close:before {
    content: "\e901";
  }
  .icon-undo:before {
    content: "\e900";
  }

  *, *:before, *:after {
    box-sizing: border-box;
  }

  input:focus {
    outline: none;
  }
  input[disabled] {
    background-color: transparent;
  }

  .input {
    border: 1px solid #ccc;
    padding: 4px;
    flex-wrap: wrap;
  }
  .tags {
    flex-wrap: wrap;
    width: 100%;
  }

  .tag {
    background-color: $primary;
    color: #fff;
    border-radius: 2px;
    padding: 3px 5px;
    margin: 2px;
    font-size: .85em;
    &:focus {
      outline: none;
    }
    .content {
      display: inline;
      align-items: center;
    }
    .tag-center {
      position: relative;
    }
    span.hidden {
      padding-left: 16px;
      visibility: hidden;
      height: 0px;
    }
    .actions {
      margin-left: 2px;
      align-items: center;
      font-size: 1.15em;
      i {
        cursor: pointer;
      }
    }
  }
</style>
