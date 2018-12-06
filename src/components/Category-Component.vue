<template lang="pug">
  .nav-category-ele-container(@click="toggleCategory" :class="{ activeCategory: category_state }")
    .nav-category-title {{ category_title }}
    .nav-category-color-container
      .nav-category-color(:style="{ background: category_color }")
</template>

<script>


export default {
  name: 'category_components',
  props: {
    category_title: {
      type: String,
      required: true
    },
    category_color: {
      type: String,
      required: true
    },
    category_category: {
      type: String,
      required: true
    },
    category_state: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    toggleCategory() {
      this.$emit('toggleCategory', this.category_category);
    }
  }
};
</script>

<style scoped lang="sass">
.nav-category-ele-container
  width: calc(100% / 6)
  height: 60px
  cursor: pointer
  border-radius: 8px
  &:hover
    background: rgba(213, 222, 229, 0.2)
  &.activeCategory
    background: rgba(213, 222, 229, 0.5)
    border: 1px solid #ffffff
  .nav-category-title
    height: 18px
    white-space: nowrap
    overflow: hidden
    text-overflow: ellipsis
    font-size: 12px
    transform: scale(0.7)
  .nav-category-color-container
    width: 100%
    height: 42px
    display: flex
    justify-content: center
    align-items: center
    .nav-category-color
      width: 16px
      height: 16px
      background: pink
      border-radius: 50%
</style>
