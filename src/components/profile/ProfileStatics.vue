<template>
    <h2 v-if="myProfile"> Mis estadísticas</h2>
    <h2 v-else>Estadísticas de {{perfil.name}}</h2>
	<div class="surface-section section mb-6 md:p-6 lg:p-6 mobileBorder mobileBackground">
        <h4>Resumen total</h4>
        <div class="surface-section section mt-4 p-6">
            <div class="grid m-auto">
                <div class="col-fixed pr-4" style="padding: 0">
                    <h5 class="mobileFont">Partidas ganadas: {{wonGames}}</h5>
                </div>
                <div class="col-fixed" style="padding-top:0; padding-bottom:0; padding-left: 0">
                    <h5 class="mobileFont">Partidas jugadas: {{playedGames}}</h5>
                </div>
            </div>
            <ProgressBar :value="progressBar" :showValue="false"/>
        </div>
    </div>
    <div class="surface-section section md:p-6 lg:p-6 mobileBorder mobileBackground">
        <h4>Resumen semanal</h4>
        <div class="p-chart surface-section section mt-4 md:p-6 lg:p-6 p-3 text-center">
            <Chart type="bar" :data="lastWeek" :options="basicOptions"/>
        </div>
    </div>   
</template>

<script>
export default {
    props: {
		myProfile: {
			type: Boolean,
			required: true
		},
        perfil: {
            type: Object,
            required: false
        },
        games: {
            type: Object,
            required: false
        },
    },
    data() {
        return {
            diasMes: [31,28,31,30,31,30,31,31,30,31,30,31],
            wonGames: 10,
            playedGames: 40,
            progressBar: '',
            historic: {
                labels: ['Jugadas', 'Ganadas'],
                datasets: [
                    {label: 'Total histórico', backgroundColor: '#42A5F5', color: '#ff0000', data: [70, 30] },
                ]
            },
            lastWeek: {
                labels: [],
                datasets: [
                    { label: 'Partidas jugadas', backgroundColor: '#42A5F5', color: '#ff0000', data: [65, 59, 80, 81, 56, 55, 40] },
                    { label: 'Partidas ganadas',backgroundColor: '#FFA726', color: '#ff0000', data: [28, 48, 40, 19, 86, 27, 90]}
                ]
            },
            basicOptions: {
                plugins: {
                    legend: { labels: { color: '#9e9e9e' } }
                },
                scales: {
                    x: {
                        ticks: {
                            color: '#9e9e9e'
                        },
                        grid: {
                            color: '#e6e6e6'
                        }
                    },
                    y: {
                        ticks: {
                            color: '#9e9e9e'
                        },
                        grid: {
                            color: '#e6e6e6'
                        }
                    }
                }
            },
        }
    },
    created(){
        /*
         *   LAST WEEK
         */
        //Cargar los datos de la tabla
        //console.log(this.games)
        this.lastWeek.datasets[0].data = this.games.diaJugadas
        this.lastWeek.datasets[1].data = this.games.diaGanadas
        console.log("last:", this.lastWeek.datasets)
        //Ponerle las labels a la tabla horizontal
        let today = new Date()
        this.lastWeek.labels[0] = 'Hoy'
        for(let i = 5; i >= 0;i--){
            let dia = today.getDate();
            let mes = today.getMonth();
            let anyo = today.getFullYear();
            if(dia == 1){
                //console.log("MES ANTERIOR")
                if(mes == 0){
                    //console.log("AÑO ANATERIOR")
                    mes = 12;
                    anyo = anyo - 1;
                } else{
                    mes = mes - 1;
                }
                dia = this.diasMes[mes];
            } else {
                dia = dia - 1;
            }
            let dd = String(dia);
            let mm = String(mes + 1); //empieza en 0
            let yyyy = String(anyo);
            this.lastWeek.labels[5 - i + 1] = dd + '/' + mm + '/' + yyyy
            today = new Date(mm + '/' + dd + '/' + yyyy);
        }
        /*
         *   TOTAL
         */
        this.wonGames = this.games.totalGanadas
        this.playedGames = this.games.totalJugadas
        this.progressBar = this.wonGames/this.playedGames * 100
    },
}
</script>

<style>
.p-chart{
    color: var(--text-color);
}
</style>