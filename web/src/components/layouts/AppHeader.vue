<template>
	<nav class="navbar navbar-expand-lg navbar-light bg-light" style="margin-bottom: 50px;">
		<div class="container">
			<router-link class="navbar-brand" to="/">Ticketing</router-link>
			
			<button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
				<span class="navbar-toggler-icon"></span>
			</button>

			<div class="collapse navbar-collapse" id="navbarSupportedContent">
				<ul class="navbar-nav me-auto mb-2 mb-lg-0">
					<li class="nav-item">
						<router-link :class="'nav-link ' + ($route.path == '/' ? 'active' : '')" to="/">Home</router-link>
					</li>
				</ul>

				<ul class="navbar-nav ms-auto mb-2 mb-lg-0">
					<template v-if="!user">
						<li class="nav-item">
							<router-link :class="'nav-link ' + ($route.path == '/login' ? 'active' : '')" to="/login">Login</router-link>
						</li>

						<li class="nav-item">
							<router-link :class="'nav-link ' + ($route.path == '/register' ? 'active' : '')" to="/register">Register</router-link>
						</li>
					</template>

					<template v-else>
						<li class="nav-item dropdown">
							<a class="nav-link dropdown-toggle" href="#" id="userDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
								<span>{{ user.name }}</span>&nbsp;
								<i class="fa fa-caret-down"></i>
							</a>

							<ul class="dropdown-menu" aria-labelledby="userDropdown">
								<li>
									<a class="dropdown-item" href="javascript:void(0)" @click="doLogout" :disabled="isLoggingOut">
										{{ isLoggingOut ? 'Logging out...' : 'Logout' }}
									</a>
								</li>
							</ul>
						</li>

						<li class="nav-item dropdown">
							<a class="nav-link dropdown-toggle" href="#" id="notificationsDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
								<i class="fa fa-bell"></i>
								<span v-if="unreadNotifications > 0" class="badge text-warning" style="margin-left: 5px;">({{ unreadNotifications }})</span>
							</a>

							<ul class="dropdown-menu" aria-labelledby="notificationsDropdown">
								<li v-for="notification in notifications" :key="notification._id">
									<router-link :class="'dropdown-item ' + (notification.isRead ? '' : 'unread-notification')" :to="'/notifications/fetch/' + notification._id">
										{{ notification.message }}
									</router-link>
								</li>

								<li><hr class="dropdown-divider" /></li>

								<li>
									<router-link class="dropdown-item" to="/notifications/all">
										All notifications
									</router-link>
								</li>
							</ul>
						</li>
					</template>
				</ul>
			</div>
		</div>
	</nav>
</template>

<script>
import "../../assets/css/bootstrap.css";
import "../../assets/fontawesome/css/all.css";

import axios from "axios";
import swal from "sweetalert2";
import store from "../../vuex/store";
import { io } from "socket.io-client";

export default {
	name: "AppHeader",

	computed: {
		user() {
			return store.getters.getUser;
		},
		unreadNotifications() {
			return store.getters.getUnreadNotifications;
		},
		notifications() {
			return store.getters.getNotifications;
		}
	},

	data() {
		return {
			isLoggingOut: false,
			socketIO: null
		};
	},

	methods: {
		async doLogout() {
			this.isLoggingOut = true;
			try {
				const response = await axios.post(this.$apiURL + "/logout", null, { headers: this.$headers });
				if (response.data.status === "success") {
					localStorage.removeItem(this.$accessToken);
					store.commit("setUser", null);
					this.$headers.Authorization = "Bearer";
					this.$router.push("/login");
				} else {
					swal.fire("Error", response.data.message, "error");
				}
			} catch (error) {
				console.error("Logout Error:", error);
				swal.fire("Error", "Logout failed.", "error");
			} finally {
				this.isLoggingOut = false;
			}
		},

		async getUser() {
			try {
				const response = await axios.post(this.$apiURL + "/getUser", null, { headers: this.$headers });
				if (response.data.status === "success") {
					const user = response.data.user;
					store.commit("setUser", user);
					store.commit("setUnreadNotifications", response.data.unreadNotifications);
					store.commit("setNotifications", response.data.notifications);

					// Initialize socket connection for notifications
					this.socketIO.emit("connected", user._id);
					this.socketIO.on("newNotification", data => {
						this.$notify(data, "success"); // Assuming $notify is defined in your Vue instance
					});

					this.socketIO.on("newReply", data => {
						const ticketId = store.getters.getTicketId;
						if (ticketId === data.ticketId) {
							store.commit("appendReply", data.reply);
						}
					});
				}
			} catch (error) {
				console.error("Fetch User Error:", error);
			}
		}
	},

	mounted() {
		this.socketIO = io(this.$apiURL);
		this.getUser();
	}
};
</script>

<style scoped>
.dropdown-toggle::after {
	display: none;
}
.unread-notification {
	background-color: #1a8754;
	color: white;
}
</style>
