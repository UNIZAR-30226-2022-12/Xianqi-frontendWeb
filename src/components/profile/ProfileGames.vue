<template>
    <h2>Mis partidas</h2>
	<div class="surface-section section md:p-6 lg:p-6 mobileBorder">
        <DataTable :value="games" :paginator="true" :rows="10"
        :rowHover="true" v-model:selection="selectedRival" v-model:filters="filters" filterDisplay="menu" :loading="loading"
        paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown" :rowsPerPageOptions="[10,25,50]"
        currentPageReportTemplate="Mostrando del {first} al {last} de {totalRecords} entradas"
        :globalFilterFields="['nickname']" responsiveLayout="scroll" class="mobileFontSmall">
            <template #header>
                <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                    <h5 class="md:m-0 text-center md:text-left"> Partidas </h5>
                    <div class="flex flex-column flex-wrap card-container">
                        <div class="flex align-items-center justify-content-center">
                            <span class="p-input-icon-left">
                                <i class="pi pi-search" />
                                <InputText v-model="filters['global'].value" style="border-radius: 1rem" placeholder="Nombre de usuario" />
                            </span>
                        </div>
                    </div>
                </div>
            </template>

            <template #empty>
                No se han encontrado partidas
            </template>
            <template #loading>
                Cargando partidas. Por favor, espere.
            </template>
            <Column class="mobileFontSmall" field="nickname" header="Adversario" sortable>
                <template #body="{data}">
                    <div class="card">
                        <div class="card-container blue-container overflow-hidden">
                            <div class="flex">
                                <img :src="getImage(data.id)" class="foto-perfil-table" style="vertical-align: middle">
                                <Button v-on:click="otherProfile(data.id)" :label="data.nickname" class="text-left p-button-link" />
                            </div>
                        </div>
                    </div>
                </template>
            </Column>
            <Column field="flag" header="País" sortable>
                <template #body="{data}">
                    <img class="flag h-auto shadow-2 mr-2" :class="[data.flag]" src="images/flags/flag_placeholder.png">
                    <span class="mobileNoDisplay image-text">{{data.country}}</span>
                </template>
            </Column>
            <Column field="startDate" header="Inicio de juego" sortable></Column>
            <Column field="lastMovDate" header="Último movimiento" sortable></Column>
            <Column field="myTurn" header="Toca mover" sortable>
                <template #body="{data}">
                    <i v-if="data.myTurn" class="pi pi-check-circle text-green-600"></i>
                    <i v-else class="pi pi-times-circle text-pink-600"></i>
                </template>
            </Column>
            <Column header="Continuar">
                <template #body="{data}">
                    <Button v-if="data.estado == 0" :disabled="clicked" class="p-button-raised" v-on:click="loadGame(data.idSala)" style="border-radius: 1rem" label="Continuar"></Button>
                    <Button v-else disabled="true" class="p-button-raised" style="border-radius: 1rem" label="Terminada"></Button>
                </template>
            </Column>
        </DataTable>    
    </div>
</template>

<script>
import {FilterMatchMode,FilterOperator} from 'primevue/api';

export default {
    inject: ['$accounts'],
    // Para obtener el parametro columns
    props: {
        games: {
            type: Object,
            required: true
        },
        gameArrayImages: {
            type: Object,
            required: true
        }
    },
    data() {
        return {
            //columns: null,
            filters: {
                'global': {value: null, matchMode: FilterMatchMode.CONTAINS},
                'nickname': {operator: FilterOperator.AND, constraints: [{value: null, matchMode: FilterMatchMode.STARTS_WITH}]},
            },
            loading: true,
            selectedRival: null,
            clicked: false,
            /*
            games: [
                {id: '1', nickname: 'Pikanachi', flag: 'flag-es', country: 'Spain', startDate:'01/01/2020', lastMovDate:'01/01/2021', myTurn: false},
                {id: '5', nickname: 'John', flag: 'flag-fr', country: 'France', startDate:'01/01/2020', lastMovDate:'01/01/2021', myTurn: true},
                {id: '5', nickname: 'Juanksp', flag: 'flag-es', country: 'Spain', startDate:'01/01/2020', lastMovDate:'01/01/2021', myTurn: false},
                {id: '5', nickname: 'AlexZheng', flag: 'flag-cn', country: 'China', startDate:'01/02/2020', lastMovDate:'01/01/2021', myTurn: true},
                {id: '5', nickname: 'AlexZheng', flag: 'flag-cn', country: 'China', startDate:'01/01/2022', lastMovDate:'01/01/2021', myTurn: true},
                {id: '5', nickname: 'AlexZheng', flag: 'flag-cn', country: 'China', startDate:'01/01/2020', lastMovDate:'01/01/2021', myTurn: true},
            ]*/
        }
    },

    mounted() {
        this.loading = false;
    },
    methods: {
        formatDate(value) {
            return value.toLocaleDateString('en-US', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric',
            });
        },
        formatCurrency(value) {
            return value.toLocaleString('en-US', {style: 'currency', currency: 'USD'});
        },
        otherProfile(id){
            this.$router.push({name: 'OtherProfile', params: { id: id}});
        },
        loadGame(id){
            this.clicked = true;
            this.$accounts.loadGame(id).then(response =>{
                let color = null;
                //console.log(response.game[1])
                sessionStorage.setItem("movs", response.game[4])
                if(response.game[1] == sessionStorage.getItem('id')){ // Jugaba como rojo
                    color = "rojo"
                    this.$router.push(`/game/${response.game[2]}/${response.game[0]}/${color}`)
                } else {
                    color = "negro"
                    this.$router.push(`/game/${response.game[1]}/${response.game[0]}/${color}`)
                }
                
            })
        },
        getImage(id){
            return this.gameArrayImages[id]
        }
    }
}
</script>

<style lang="scss">

.foto-perfil-table {
    display: block; 
	/*vertical-align: middle;*/
	width: 2.5rem;
	height: 2.5rem;
    margin: 0.5rem;

    box-shadow: 1px 1px 6px rgba(0, 0, 0, 0.3), 0px 0px 2px rgba(0, 0, 0, 0.06), 0px 2px 3px rgba(0, 0, 0, 0.12) !important;
	border-radius: 50%;
	border-color: var(--surface-400);
	object-fit: cover;
}

::v-deep(.p-paginator) {
    .p-paginator-current {
        margin-left: auto;
    }
}

::v-deep(.p-progressbar) {
    height: .5rem;
    background-color: #D8DADC;

    .p-progressbar-value {
        background-color: #607D8B;
    }
}

::v-deep(.p-datepicker) {
    min-width: 25rem;

    td {
        font-weight: 400;
    }
}

::v-deep(.p-datatable.p-datatable-customers) {
    .p-datatable-header {
        padding: 1rem;
        text-align: left;
        font-size: 1.5rem;
    }

    .p-paginator {
        padding: 1rem;
    }

    .p-datatable-thead > tr > th {
        text-align: left;
    }

    .p-datatable-tbody > tr > td {
        cursor: auto;
    }

    .p-dropdown-label:not(.p-placeholder) {
        text-transform: uppercase;
    }
}
</style>