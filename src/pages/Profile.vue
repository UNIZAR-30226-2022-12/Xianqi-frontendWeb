<template>
	<loader v-if="loading"/>
	<myProfile v-if="perfil!=null" :perfil="perfil" :stats="stats" :profileImage="profileImage" :myProfile="true"/>
	<myGames v-if="games!=null" :games="games" :gameArrayImages="gameArrayImages"/>
	<myStatics v-if="stats!=null" :perfil="perfil" :myProfile="true" :games="stats"/>
</template>

<script>
import loader from '../components/Loader.vue';
import myProfile from '../components/profile/ProfileData.vue';
import myGames from '../components/profile/ProfileGames.vue';
import myStatics from '../components/profile/ProfileStatics.vue';


export default {
	inject: ['$accounts'],
	data() {
		return {
			loading: true,
			perfil: null,
			games: null,
			gameArrayImages: [],
			columns: null,
			profileImage: '',
			stats: null,
			socket: null,
		}
	},
	methods:{
		equals(value){ 
			return value.id != 14; 
		}, 
	},
	components: {
		loader,
		myProfile,
		myGames,
		myStatics,
	},
	created() {
		this.$loggedStatus.logged = this.$accounts.isAuthenticated();
		this.$accounts.getProfile(sessionStorage.getItem('id')).then(response => {
			this.perfil = response.perfil;
			this.games = response.partidas;
			this.stats = response.estadisticas;

			if (this.perfil.hasImage) {
				// Pedir al back la foto
				this.$accounts.getProfileImage(sessionStorage.getItem('id')).then(data => {
					this.profileImage = data;
				});
			} else {
				this.profileImage = "images/profilePlaceholder.svg";
			}
			this.games = this.games.filter(this.equals)
			for (var i = 0; i < this.games.length; i++) {
				//llegan las fotos
				let id = this.games[i].id;
				if (this.games[i].hasImage && this.gameArrayImages[id] == null) {
					this.gameArrayImages[id] = '';
					this.$accounts.getProfileImage(id).then(data => {
						this.gameArrayImages[id] = data;
					});
				} else if (!this.games[i].hasImage && this.gameArrayImages[id] == null) {
					this.gameArrayImages[id] = "images/profilePlaceholder.svg";
				}
			}
			this.loading = false;
		});
	},
}
</script>

<style>
.section {
	border-radius: 1rem;
	border-style: solid;
	border-width: 1px;
	border-color: var(--surface-200);
}

/* Este estilo se activa cuando el tamaño de la ventana es >= 992px */
@media (min-width: 992px) {
	.profile-name {
		max-width: 30rem;
	}
}

/* Este estilo se activa cuando el tamaño de la ventana es <= 991px */
@media (max-width: 991px) {
	.profile-name {
		/*background-color: green !important;*/
	}
}
</style>