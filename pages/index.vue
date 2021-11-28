<template>
  <div class="container">
    <div class="row align-items-center vh-80">
      <div class="col-md-6 mx-auto">
        <div class="d-flex justify-content-center mb-6">
          <img src="~/static/Frame-191.png" alt="IroBus">
        </div>
        <p class="text-white text-center mb-11">
          提供最即時的公車動態，讓您輕鬆掌握資訊，<br>現在就開始規劃您的路線吧！
        </p>
        <ul
          class="
            list-unstyled
            d-flex
            flex-column flex-lg-row
            justify-content-center
            align-items-center
          "
        >
          <li>
            <NuxtLink
              class="btn btn-focus d-flex align-items-center rounded-pill px-5 text-white"
              to="/cityBus"
            >
              <img src="~/assets/icon/bus.png" alt="bus" class="me-4">
              市區公車
            </NuxtLink>
          </li>
          <li class="mx-lg-4 my-4 my-lg-0">
            <button
              type="button"
              class="btn btn-focus d-flex align-items-center rounded-pill px-5 text-dark"
            >
              <img src="~/assets/icon/bus.png" alt="bus" class="me-4">
              公路客運
            </button>
          </li>
          <li>
            <button
              type="button"
              class="btn btn-focus d-flex align-items-center rounded-pill px-5 text-dark"
            >
              <img src="~/assets/icon/bus.png" alt="bus" class="me-4">
              乘車規劃
            </button>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {};
</script>
