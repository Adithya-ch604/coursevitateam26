<template>
  <div id="app">
    <div class="main-content">
      <!-- Your login or other page content goes here -->
    </div>
    <!-- Footer -->
    <footer class="footer text-center text-lg-start bg-light text-muted">
      <!-- Copyright -->
      <div class="text-center p-4" style="background-color: rgba(0, 0, 0, 0.05);">
        © <span v-text="year"></span> Copyright:
        <a class="text-reset fw-bold">COURSEVITA_TEAM 26</a>
      </div>
      <!-- Copyright -->
    </footer>
    <!-- Footer -->
  </div>
</template>


<script>

	import jQuery from "jquery"
	import "../../assets/js/bootstrap.js"
	import "../../assets/js/notify.js"

	export default {
		name: "AppFooter",

		data() {
			return {
				year: new Date().getFullYear()
			}
		}
	}
</script>

<style scoped>
html, body {
  height: 100%;
  margin: 0;
}

#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.footer {
  margin-top: auto;
}
</style>
