<template>
	<div class="layout-menu-container">
		<AppSubmenu :onlineFriends="onlineFriends" :primera="true" :items="menu" @friends-notify-pressed="clearFriendReq" :root="true" :numFriendReq="this.numFriendReq" class="layout-menu" @menuitem-click="onMenuItemClick" @keydown="onKeyDown" />
        <GameReq v-if="showInvitaion" :nickname="nickname" :id="id" :idSala="idSala" @close-game-request="closeGameRequest"/>
        <RejectedInvitation/>
    </div>
</template>

<script>
import GameReq from './components/GameRequest.vue';
import RejectedInvitation from './components/RejectedInvitation.vue';
import AppSubmenu from './AppSubmenu';
export default {
    emits: ['menu-toggle', 'menuitem-click'],
    inject: ['$accounts', '$game'],
    components: {
		'AppSubmenu': AppSubmenu,
        GameReq,
        RejectedInvitation,
	},
	data() {
		return {
            showInvitaion: false,
            nickname: '',
            showRejectedInvitation: false,
            socket: null,
            numFriendReq: 0,
            onlineFriends: [],
            id: null,
            idSala: 1,

            menu: [
                {
                    label: 'Menú',
                    items: [
                        {label: 'Perfil', icon: 'pi pi-fw pi-home', to: '/Profile'},
                        {label: 'Amigos', icon: 'pi pi-fw pi-users', to: '/Friends'},
                        {label: 'Ranking', icon: 'pi pi-fw pi-list', to: '/Ranking'},
                        {label: 'Tienda', icon: 'pi pi-fw pi-shopping-cart', to: '/Store'},
                        {label: 'Historial', icon: 'pi pi-fw pi-history', to: '/Historial'},
                    ]
                },
            ]
		}
	},
    created() {
        this.$game.socket.emit('enter',{'id': sessionStorage.getItem('id')})
        this.$game.socket.on("friendRequest",(data)=>{
            console.log("FriendRequest de", data);
            this.$toast.add({severity:'info', summary:'Solicitud de amistad', detail:'Has recibido una solicituh de amistad.', life: 5000});
            this.numFriendReq++;
            //console.log("numFriendReq", this.numFriendReq);
        })
        this.$game.socket.on("onlineFriends",(data)=>{
            this.onlineFriends = []
            data.forEach(friend => {
                this.onlineFriends.push({"id": friend['id'], "name": friend['nickname']});
            });
        })
        this.$game.socket.on("gameRequest",(data)=>{
            this.id = data["id"]
            this.idSala = data["idSala"]
            this.$accounts.getNickname(data["id"]).then(data => { //this.createAc.image
                //console.log(data);
                this.nickname = data;
                this.showInvitaion = true;
                //console.log(this.showInvitaion, this.idSala)
            });
        })
        this.$game.socket.on("rejectReq",()=>{
            this.emitter.emit("open-rejected-invitation", true);

        })
        this.$game.socket.on("noGame",()=>{
            console.log("noGame")
            this.emitter.emit("open-rejected-invitation", true);

        })
		this.$game.socket.emit('getOnlineFriends',{'id': sessionStorage.getItem('id')})
    },
    methods: {
        clearFriendReq() {
			//Ya hemos atendido la notificacion
            this.numFriendReq = 0;
            // Ir a la pagina de amigos y que muestre solo las solicitudes
			this.$router.push('/FriendRequest');
		},
        onMenuToggle(event) {
            this.$emit('menu-toggle', event);
        },
        onMenuItemClick(event) {
            this.$emit('menuitem-click', event);
        },
		onKeyDown(event) {
			const nodeElement = event.target;
			if (event.code === 'Enter' || event.code === 'Space') {
				nodeElement.click();
				event.preventDefault();
			}
		},
        closeGameRequest(){
            this.showInvitaion = false;
        },
    },
	computed: {
		darkTheme() {
			return this.$appState.darkTheme;
		}
	}
}
</script>

<style>

.button {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    position: relative;
    color: var(--surface);
    border: none;
    border-radius: 50%;
    width: 3rem;
    height: 3rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

.button:hover {
    background-color: var(--surface-hover);
}

.imagen {
    height: 35px;
    margin-right: 7px;
}

</style>