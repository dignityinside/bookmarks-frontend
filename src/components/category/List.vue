<template>
  <div class="row">
    <draggable
      v-model="categories"
      class="d-flex flex-wrap flex-auto"
      group="categories"
      handle=".category"
      @change="update()"
    >
      <div v-for="category in categories" :key="category.id" class="col-md-3">
        <div :style="{borderColor: '#' + category.color}" class="card mb-3">
          <div
            :style="{backgroundColor: '#' + category.color, opacity: 0.5}"
            :title="category.description"
            class="card-header d-flex justify-content-between align-items-center category"
          >
            <i v-show="category.icon" :class="category.icon"></i>
            {{ category.name }}
            <a class="btn btn-outline-light btn-sm" href="#" @click.prevent="edit(category.id)">
              <i class="fa fa-edit"></i>
            </a>
          </div>
          <link-list :items="category"></link-list>
        </div>
      </div>

      <div v-if="$store.getters.activeBoardId !== 0" class="col-md-3">
        <div class="card bg-white">
          <div class="card-header">
            <i class="fa fa-plus pr-1"></i>
            <a href="#" @click.prevent="$store.commit('toggle_category_modal', true)">
              {{ $t('category.add') }}
            </a>
          </div>
        </div>
      </div>
    </draggable>

    <modal v-if="$store.getters.showCategoryModal">
      <div slot="content">
        <category-form></category-form>
      </div>
    </modal>
  </div>
</template>

<script>
import _ from 'lodash';
import draggable from 'vuedraggable';
import CategoryForm from './Form.vue';
import LinkList from '../link/List.vue';
import CategoryService from '../../services/CategoryService';
import Modal from '../Modal.vue';

export default {
  name: 'CategoriesList',
  components: {
    CategoryForm,
    LinkList,
    Modal,
    draggable,
  },
  computed: {
    categories: {
      get() {
        return this.$store.getters.categories;
      },
      set(value) {
        this.$store.commit('update_categories', value);
      },
    },
  },
  methods: {
    edit(selectedId) {
      const selectedCategory = _.find(this.categories, { id: selectedId });
      this.$store.commit('toggle_category_modal', true);
      this.$nextTick(() => {
        CategoryService.edit(selectedCategory);
      });
    },
    update() {
      // eslint-disable-next-line array-callback-return
      this.categories.map((category, index) => {
        this.$store.dispatch('category_save', {
          url: `/categories/${category.id}`,
          method: 'put',
          // eslint-disable-next-line no-param-reassign
          sort: category.sort = index + 1,
        });
      });
    },
  },
  created() {
    CategoryService.$on('categoriesChanged', () => {
      this.categories = CategoryService.categories;
    });
  },
};
</script>

<style scoped>
.flex-auto {
  flex: auto;
}
</style>
