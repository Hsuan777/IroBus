<template>
  <nav class="navbar navbar-expand-lg navbar-dark">
    <div v-show="$route.name !== 'index'" class="container-fluid" @click="hideCollapseNavbar">
      <NuxtLink class="navbar-brand" to="/">
        <img src="~/static/logo-2.png" alt="Logo">
      </NuxtLink>
      <button
        class="navbar-toggler"
        type="button"
        aria-controls="navbarNav"
        aria-expanded="false"
        aria-label="Toggle navigation"
        @click="toggleCollapseNavbar"
      >
        <span class="navbar-toggler-icon" />
      </button>
      <div id="navbarNav" ref="navbarNav" class="collapse navbar-collapse">
        <ul class="navbar-nav ms-auto">
          <li class="nav-item">
            <NuxtLink class="nav-link" to="/cityBus" @click="hideCollapseNavbar">
              市區公車
            </NuxtLink>
          </li>
          <!-- <li class="nav-item ">
            <NuxtLink class="nav-link text-dark" to="/">
              公路客運
            </NuxtLink>
          </li>
          <li class="nav-item">
            <NuxtLink class="nav-link text-dark" to="/">
              乘車規劃
            </NuxtLink>
          </li> -->
        </ul>
      </div>
    </div>
  </nav>
</template>
<script>
export default {
  created() {
    if (process.client) {
      this.collapse = require('bootstrap/js/dist/collapse');
    }
  },
  mounted() {
    const Collapse = this.collapse;
    this.collapseNavbar = new Collapse(this.$refs.navbarNav, {
      toggle: false,
    });
  },
  methods: {
    toggleCollapseNavbar() {
      this.collapseNavbar.toggle();
    },
    hideCollapseNavbar() {
      this.collapseNavbar.hide();
    },
  },
};
</script>
