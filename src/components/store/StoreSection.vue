<template>
    <div class="surface-section section mb-6 md:p-6 lg:p-6 mobileBorder">
        <pointsDialog :display="display" @ok-pressed="closeDialog"/>
        <loader v-if="this.loading"/>   
        <DataView v-else :value="set" :layout="layout" :paginator="true" :rows="6" :sortOrder="sortOrder" :sortField="sortField" style="border-color: var(--surface-border) !important">
            <template #header>
                <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                    <h5 class="md:m-0 text-center md:text-left"> Sets de {{ name }} </h5>
                    <div class="flex flex-wrap card-container">
                        <div class="flex align-items-center justify-content-center mr-2">
                            <Dropdown v-model="sortKey" :options="sortOptions" optionLabel="label" placeholder="Ordenar por..." @change="onSortChange($event)" style="border-radius: 1rem"/>
                        </div>
                        <div class="flex-grow-1 flex"></div>
                        <div class="flex align-items-center justify-content-center mr-0">
                            <DataViewLayoutOptions class="text-center" v-model="layout"/>
                        </div>
                    </div>
                </div>
            </template>
            <!-- Grid lista -->
            <template #list="slotProps">
                <div class="col-12">
                    <div class="product-list-item">
                        <img v-if="name == 'tableros'" class="store-item-list" :src="'images/themes/boards/' + slotProps.data.id + '.jpg'" :alt="slotProps.data.name">
                        <img v-else class="store-item-list store-item-grid-piece" :src="'images/themes/pieces/' + slotProps.data.id + '/canyonnegro.svg'">
                        <div class="product-list-detail">
                            <div class="product-name">{{slotProps.data.name}}</div>
                            <div class="product-description">{{slotProps.data.desc}}</div>
                            <i class="pi pi-tag product-category-icon"></i><span class="product-category">{{slotProps.data.category}}</span>
                        </div>
                        <div class="h-2"></div>
                        <div class="product-list-action">
                            <span v-if="slotProps.data.price > 0" class="product-price">
                                {{slotProps.data.price}}
                                <font-awesome-icon class="text-xl" icon="coins" />
                            </span>
                            <span v-else class="product-price">
                                ¡Gratis!
                            </span>
                            <Button icon="pi pi-shopping-cart" v-on:click="purchaseItem(slotProps.data.price, slotProps.data.id, slotProps.data.tipo)" label="Comprar" :disabled="slotProps.data.purchased"></Button>
                        </div>
                    </div>
                </div>
            </template>
            <!-- Grid cuadrada -->
            <template #grid="slotProps">
                <div class="col-12 md:col-4">
                    <div class="product-grid-item card">
                        <div class="product-grid-item-top">
                            <div>
                                <i class="pi pi-tag product-category-icon"></i>
                                <span class="product-category">{{slotProps.data.category}}</span>
                            </div>
                        </div>
                        <div class="product-grid-item-content">
                            <img v-if="name == 'tableros'" class="store-item-grid" :src="'images/themes/boards/' + slotProps.data.id + '.jpg'" :alt="slotProps.data.name">
                            <img v-else class="store-item-grid-piece store-item-grid" :src="'images/themes/pieces/' + slotProps.data.id + '/canyonnegro.svg'">
                            <div class="product-name">{{slotProps.data.name}}</div>
                            <div class="product-description">{{slotProps.data.desc}}</div>
                        </div>
                        <div class="h-2rem"></div>
                        <div class="product-grid-item-bottom">
                            <span v-if="slotProps.data.price > 0" class="product-price">
                                {{slotProps.data.price}}
                                <font-awesome-icon class="text-xl" icon="coins" />
                            </span>
                            <span v-else class="product-price">
                                ¡Gratis!
                            </span>
                            <Button icon="pi pi-shopping-cart" v-on:click="purchaseItem(slotProps.data.price, slotProps.data.id, slotProps.data.tipo, slotProps.data.name)" :disabled="slotProps.data.purchased"></Button>
                        </div>
                    </div>
                </div>
            </template>
        </DataView>
    </div>


</template>

<script>
import loader from '../LoaderSmall.vue';
import pointsDialog from './StorePoints.vue';

export default {
	inject: ['$accounts'],
    emits: ['purchase-item-pressed'],
    components: {
        loader,
		pointsDialog,
    },
    props: {
        set: {
            type: Object,
            required: true
        },
        name: {
            type: String,
            required: true
        },
        loading: {
            type: Boolean,
            required: true
        },
        points: {
            type: Number,
            required: true
        },
    },
    data() {
		return {
            layout: 'grid',
            sortKey: null,
            sortOrder: null,
            sortField: null,
            sortOptions: [
                {label: 'Precio de Mayor a Menor', value: '!price'},
                {label: 'Precio de Menor a Mayor', value: 'price'},
                {label: 'Comprados primero', value: '!purchased'},
                {label: 'No comprados primero', value: 'purchased'},
                {label: 'Por categoría', value: 'category'},
            ],
            display: false,
        }
    },
    methods: {
        comprado(index) {
            if(this.set[index].purchased) {
                return 'Comprado';
            } else {
                return 'Comprar';
            }
        },
        onSortChange(event){
            const value = event.value.value;
            const sortValue = event.value;

            if (value.indexOf('!') === 0) {
                this.sortOrder = -1;
                this.sortField = value.substring(1, value.length);
                this.sortKey = sortValue;
            }
            else {
                this.sortOrder = 1;
                this.sortField = value;
                this.sortKey = sortValue;
            }
        },
        purchaseItem(price, id, tipo, name) {
            var saldo = this.points - price;
            if (saldo >= 0) {
                // points suficientes
                this.$toast.add({severity:'success', summary:'Skin comprada', detail:'Has comprado la skin' + name + '. Tu saldo ahora es de ' + saldo + ' puntos.', life: 5000});
                this.$emit('purchase-item-pressed', price, id, tipo);
            } else {
                this.display = true;
            }
        },
        closeDialog() {
            this.display = false;
        }
    }
}

</script>

<style>

.p-dataview-header {
    border-top-left-radius: 1rem !important;
    border-top-right-radius: 1rem !important;
    border-top-style: none !important;
}

.p-paginator-bottom {
    border-bottom-left-radius: 1rem !important;
    border-bottom-right-radius: 1rem !important;
}

</style>

<style lang="scss" scoped>

.p-dataview {
    border-width: 1px !important;
    border-style: solid;
    border-radius: 1rem;
    border-bottom-style: none !important;
}

.store-item-list {
	vertical-align: middle;
	width: 5rem;
	height: 5rem;
	border-radius: 10%;
}

.store-item-grid {
	vertical-align: middle;
	width: 10rem;
	height: 10rem;
	border-radius: 10%;
}

.store-item-grid-piece {
    border-radius: 50%;
}

.board {
	vertical-align: middle;
	width: 8rem;
	height: 8rem;
	border-radius: 50%;
	object-fit: cover;
}

.card {
    padding: 2rem;
    /*box-shadow: 0 2px 1px -1px rgba(0,0,0,.2), 0 1px 1px 0 rgba(0,0,0,.14), 0 1px 3px 0 rgba(0,0,0,.12);*/
    border-radius: 1rem;
    margin-bottom: 2rem;
}
.p-dropdown {
    width: 14rem;
    font-weight: normal;
}

.product-name {
	font-size: 1.2rem;
	font-weight: 700;
}

.product-description {
	margin: 0 0 1rem 0;
}

.product-category-icon {
	vertical-align: middle;
	margin-right: .5rem;
}

.product-category {
	font-weight: 600;
	vertical-align: middle;
}

::v-deep(.product-list-item) {
	display: flex;
	align-items: center;
	padding: 1rem;
	width: 100%;

	img {
		box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
		margin-right: 2rem;
	}

	.product-list-detail {
		flex: 1 1 0;
	}

	.p-rating {
		margin: 0 0 .5rem 0;
	}

	.product-price {
		font-size: 1.5rem;
		font-weight: 600;
		margin-bottom: .5rem;
		align-self: flex-end;
	}

	.product-list-action {
		display: flex;
		flex-direction: column;
	}

	.p-button {
		margin-bottom: .5rem;
	}
}

::v-deep(.product-grid-item) {
	margin: .5rem;
	border: 1px solid var(--surface-border);
    min-height: auto;
    max-height: auto;

	.product-grid-item-top,
	.product-grid-item-bottom {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	img {
		box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
		margin: 2rem 0;
	}

	.product-grid-item-content {
		text-align: center;
	}

	.product-price {
		font-size: 1.5rem;
		font-weight: 600;
	}
}

@media screen and (max-width: 576px) {
	.product-list-item {
		flex-direction: column;
		align-items: center;

		img {
			margin: 2rem 0;
		}

		.product-list-detail {
			text-align: center;
		}

		.product-price {
			align-self: center;
		}

		.product-list-action {
			display: flex;
			flex-direction: column;
		}

		.product-list-action {
			margin-top: 2rem;
			flex-direction: row;
			justify-content: space-between;
			align-items: center;
			width: 100%;
		}
	}
}
</style>