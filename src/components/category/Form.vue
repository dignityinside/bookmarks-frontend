<template>
    <div class="card bg-light">
        <div class="card-header" v-text="$t(isNewRecord ? 'form.addCategory' : 'form.updateCategory')"></div>
        <div class="card-body">
            <form method="post" @submit.prevent="submit">
                <div class="form-group">
                    <label for="category-name">{{ $t('form.name') }}</label>
                    <input id="category-name"
                           type="text"
                           autocomplete="off"
                           class="form-control"
                           :class="validationCssClass($v.name)"
                           v-model.trim="$v.name.$model"
                           @keydown="responseErrors = responseErrors.filter((item) => item.field !== 'name')"
                    >
                    <template v-for="(validator, validatorName) in $v.name.$params">
                        <div class="invalid-feedback"
                             v-if="!$v.name[validatorName]"
                             v-t="validator && validator.message ? validator.message : validator"
                        ></div>
                    </template>
                </div>
                <div class="form-group">
                    <label for="category-description">{{ $t('form.description') }}</label>
                    <textarea id="category-description"
                              rows="2"
                              type="text"
                              class="form-control"
                              :class="validationCssClass($v.description)"
                              v-model.trim="$v.description.$model"
                              @keydown="responseErrors = responseErrors.filter((item) => item.field !== 'description')"
                    ></textarea>
                    <template v-for="(validator, validatorName) in $v.description.$params">
                        <div class="invalid-feedback"
                             v-if="!$v.description[validatorName]"
                             v-t="validator && validator.message ? validator.message : validator"
                        ></div>
                    </template>
                </div>
                <div class="form-group">
                    <label for="category-color">{{ $t('form.color') }}</label>
                    <input id="category-color"
                           type="color"
                           autocomplete="off"
                           class="form-control"
                           :class="validationCssClass($v.color)"
                           v-model.trim="$v.color.$model"
                           @keydown="responseErrors = responseErrors.filter((item) => item.field !== 'color')"
                    >
                    <template v-for="(validator, validatorName) in $v.color.$params">
                        <div class="invalid-feedback"
                             v-if="!$v.color[validatorName]"
                             v-t="validator && validator.message ? validator.message : validator"
                        ></div>
                    </template>
                </div>
                <div class="form-group">
                    <label for="category-icon">{{ $t('form.icon') }}</label>
                    <input id="category-icon"
                           type="text"
                           class="form-control"
                           :class="validationCssClass($v.icon)"
                           v-model.trim="$v.icon.$model"
                           @keydown="responseErrors = responseErrors.filter((item) => item.field !== 'icon')"
                    >
                    <template v-for="(validator, validatorName) in $v.icon.$params">
                        <div class="invalid-feedback"
                             v-if="!$v.icon[validatorName]"
                             v-t="validator && validator.message ? validator.message : validator"
                        ></div>
                    </template>
                </div>
                <div class="form-group">
                    <button type="submit" class="btn btn-success mr-2" :disabled="isSaving">
                        <spinner :state="isSaving"><i :class="['fas', isNewRecord ? 'fa-plus-square' : 'fa-save']"></i>
                        </spinner>
                        {{ isNewRecord ? $t('form.add') : $t('form.save') }}
                    </button>
                    <button type="reset" class="btn btn-secondary mr-2" @click.prevent="close" :disabled="isSaving">
                      <i class="fas fa-times"></i>
                      {{ $t('form.close') }}
                    </button>
                  <button type="submit" class="btn btn-danger" @click.prevent="remove" :disabled="isRemoving" v-if="!isNewRecord">
                    <spinner :state="isRemoving">
                      <i class="fas fa-trash-alt"></i>
                    </spinner>
                    {{ $t('form.remove') }}
                  </button>
                </div>
            </form>
        </div>
    </div>
</template>

<script>
    import CategoryService from "../../services/CategoryService";
    import BoardService from "../../services/BoardService";
    import Spinner from "../misc/Spinner";
    import {required, minLength, maxLength, helpers} from 'vuelidate/lib/validators';
    import {serverError} from "@/validators/validators";

    export default {
        name: "CategoryForm",
        components: {Spinner},
        data() {
            return {
                // form data
                id: null,
                name: '',
                description: '',
                color: '',
                icon: '',
                // states
                isSaving: false,
                isRemoving: false,
                // server errors
                responseErrors: [],
            };
        },
        validations: {
            name: {
                required: helpers.withParams({message: 'error.required'}, required),
                minLength: helpers.withParams({message: {path: 'error.tooShort', args: {min: 2}}}, minLength(2)),
                maxLength: helpers.withParams({message: {path: 'error.tooLong', args: {max: 255}}}, maxLength(255)),
                serverError: serverError('name'),
            },
            description: {
                maxLength: helpers.withParams({message: {path: 'error.tooLong', args: {max: 255}}}, maxLength(255)),
                serverError: serverError('description'),
            },
            color: {
                minLength: helpers.withParams({message: {path: 'error.tooShort', args: {min: 6}}}, minLength(6)),
                maxLength: helpers.withParams({message: {path: 'error.tooLong', args: {max: 7}}}, maxLength(7)),
                serverError: serverError('color'),
            },
            icon: {
                minLength: helpers.withParams({message: {path: 'error.tooShort', args: {min: 2}}}, minLength(2)),
                maxLength: helpers.withParams({message: {path: 'error.tooLong', args: {max: 255}}}, maxLength(255)),
                serverError: serverError('icon'),
            },
        },
        computed: {
            isNewRecord() {
                return this.id === null;
            },
        },
        methods: {
          submit() {
            this.$v.$touch();

            if (this.$v.$invalid) {
              return;
            }

            this.isSaving = true;

            this.$store.dispatch('category_save', {
              url: this.isNewRecord ? '/categories' : '/categories/' + this.id,
              method: this.isNewRecord ? 'post' : 'put',
              board_id: this.$store.getters.activeBoardId,
              name: this.name,
              description: this.description,
              color: this.color,
              icon: this.icon,
            })
              .then(resp => {
                BoardService.fetchBoards();
                this.$store.commit('toggle_category_modal', false);
                this.reset();
              })
              .catch(err => {
                if (err.response.status === 422) {
                  this.responseErrors = err.response.data;
                  // this.$v.$touch();
                }
              }).finally(() => {
                this.isSaving = false;
              });

            },
            reset() {
                this.id = null;
                this.name = '';
                this.description = '';
                this.color = '';
                this.icon = '';
            },
            close() {
              this.reset();
              this.$store.commit('toggle_category_modal', false);
            },
            remove() {
              this.isRemoving = true;

              this.$store.dispatch('category_remove', {
                url: '/categories/' + this.id,
                method: 'delete',
              })
                .then(resp => {
                  BoardService.fetchBoards();
                  this.$store.commit('toggle_category_modal', false);
                  this.reset();
                })
                .catch(err => {
                  if (err.response.status === 422) {
                    this.responseErrors = err.response.data;
                  }
                }).finally(() => {
                  this.isRemoving = false;
              });
            },
            validationCssClass(validation) {
                return {
                    'is-valid': !validation.$error && validation.$dirty,
                    'is-invalid': validation.$error,
                }
            },
        },
        created() {
            // Reset validation
            this.$v.$reset();

            CategoryService.$on('edit', (item) => {
                this.id = item.id;
                this.name = item.name;
                this.description = item.description;
                this.color = item.color;
                this.icon = item.icon;
            });

            CategoryService.$on('reset', () => {
                reset();
            });
        },
    }
</script>