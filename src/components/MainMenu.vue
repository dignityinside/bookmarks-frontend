<template>
  <nav>
    <div class="nav">
      <template v-if="isLoggedIn">
        <div class="nav__item">
          <router-link :to="{name: 'home'}" class="btn btn-outline-success btn-sm">
            {{ $t('menu.boards') }}
          </router-link>
        </div>
        <div class="nav__item">
          <a class="btn btn-outline-danger btn-sm" href="#" @click.prevent="logout">
            {{ $t('menu.logout') }}
          </a>
        </div>
      </template>
      <template v-else>
        <div class="nav__item">
          <router-link :to="{name: 'login'}" class="btn btn-outline-success btn-sm">
            {{ $t('menu.login') }}
          </router-link>
        </div>
        <div class="nav__item">
          <router-link :to="{name: 'signup'}" class="btn btn-outline-primary btn-sm">
            {{ $t('menu.signup') }}
          </router-link>
        </div>
      </template>
      <div class="nav__item locale-changer">
        <select
          v-model="selectedLanguage"
          class="custom-select custom-select-sm"
          @change="changeLang($event.target.value)">
          <option v-for="(lang, i) in langs" :key="`Lang${i}`" :value="lang">
            {{ lang }}
          </option>
        </select>
      </div>
    </div>
  </nav>
</template>

<script>
import { loadLanguageAsync, localeParamName } from '@/lang/i18n-setup';

export default {
  data() {
    return {
      selectedLanguage: 'en',
      langs: ['en', 'ru'],
    };
  },
  computed: {
    isLoggedIn() {
      return this.$store.getters.isLoggedIn;
    },
  },
  methods: {
    logout() {
      this.$store.dispatch('logout')
        .then(() => {
          this.$router.push('/login');
        });
    },
    changeLang(lang) {
      loadLanguageAsync(lang).then();
    },
  },
  created() {
    const selectedLocale = localStorage.getItem(localeParamName);

    if (selectedLocale) {
      this.selectedLanguage = selectedLocale;
    }
  },
};
</script>

<style lang="scss" scoped>
.nav {
  display: flex;
  padding: 10px 0;
  justify-content: center;

  &__item {
    padding: 0 5px;
  }
}
</style>
