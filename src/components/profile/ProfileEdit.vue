<template>
	<Button v-on:click="editDialog.display = true; getEditParameters()" class="p-button-raised" style="border-radius: 1rem" label="Editar perfil" icon="pi pi-user-edit" iconPos="left" />
    <Dialog v-model:visible="editDialog.display" :draggable="false" :modal="true" class="edit-dialog" :class="{ 'edit-dialog-sm-size': editDialog.isActive, 'edit-dialog-lg-size': !editDialog.isActive}">
        <template #header :class="colorHeader">
            <h3 class="text-center"> Editar perfil </h3> 
        </template>
        <!--VUELIDATE-->
        <div class="flex justify-content-center">
            <div class="card resize mt-5" style="width: 440px">
                <form @submit.prevent="handleSubmit(!vEdit$.$invalid)" class="p-fluid">
                    <!--NICKNAME-->
                    <div class="field">
                        <label for="nickname" :class="{'p-error':(vEdit$.edit.nickname.$invalid && edit.submitted) || (vEdit$.edit.nickname.$invalid && vEdit$.edit.nickname.$model != '')}">Nombre de usuario</label>
                        <div class="p-inputgroup">
                        <Button v-on:click="edit.nicknameDisable = !edit.nicknameDisable" icon="pi pi-pencil" />
                        <InputText id="nickname" :disabled="edit.nicknameDisable" placeholder="Nombre de usuario" v-model="edit.nickname" :class="{'p-invalid':(vEdit$.edit.nickname.$invalid && edit.submitted) || (vEdit$.edit.nickname.$invalid && vEdit$.edit.nickname.$model != '')}" />
                        <span class="p-inputgroup-addon">
                            <i class="pi pi-id-card"></i>
                        </span>
                        </div>
                        <small v-if="((vEdit$.edit.nickname.$invalid && vEdit$.edit.nickname.$model == '') || vEdit$.edit.nickname.$pending.$response) && edit.submitted" class="p-error">{{'Por favor, indique un nombre de usuario'}}</small>
                        <small v-else-if="(vEdit$.edit.nickname.$invalid && vEdit$.edit.nickname.$model != '')" class="p-error">{{'El nombre de usuario no puede tener más de 15 caracteres'}}</small>
                    </div>
                    <!--NAME-->
                    <div class="field">
                        <label for="name" :class="{'p-error':vEdit$.edit.name.$invalid && edit.submitted}">Nombre completo</label>
                        <div class="p-inputgroup">
                        <Button v-on:click="edit.nameDisable = !edit.nameDisable" icon="pi pi-pencil" />
                        <InputText id="name" :disabled="edit.nameDisable" placeholder="Nombre Apellido" v-model="edit.name" :class="{'p-invalid':vEdit$.edit.name.$invalid && edit.submitted}" />
                        <span class="p-inputgroup-addon">
                            <i class="pi pi-user"></i>
                        </span>
                        </div>
                        <small v-if="(vEdit$.edit.name.$invalid && edit.submitted) || vEdit$.edit.name.$pending.$response" class="p-error">{{'Por favor, indique su nombre'}}</small>
                    </div>
                    <!--BIRTHDAY-->
                    <div class="field">
                        <label for="date" :class="{'p-error':vEdit$.edit.date.$invalid && edit.submitted}">Fecha de nacimiento</label>
                        <div class="grid m-auto">
                            <Button class="col-fixed" style="border-top-right-radius: 0; border-bottom-right-radius: 0" v-on:click="edit.dateDisable = !edit.dateDisable" icon="pi pi-pencil" />
                            <Calendar contentStyle="border-radius: 0px !important" dateFormat="dd/mm/yy" id="date" :maxDate="new Date()" class="col p-0" :disabled="edit.dateDisable" placeholder="01/01/2000" v-model="edit.date" :showIcon="true" :class="{'p-invalid':vEdit$.edit.date.$invalid && edit.submitted}"/>
                        </div>
                        <small v-if="(vEdit$.edit.date.$invalid && edit.submitted) || vEdit$.edit.date.$pending.$response" class="p-error">{{'Por favor, indique su fecha de nacimiento'}}</small>
                    </div>
                    <!--COUNTRY-->
                    <div class="field">
                        <label for="country" :class="{'p-error':vEdit$.edit.country.$invalid && edit.submitted}">Seleccione su país de residencia</label>
                        <div class="p-inputgroup">
                        <Button class="col-fixed" style="border-top-right-radius: 0; border-bottom-right-radius: 0" v-on:click="edit.countryDisable = !edit.countryDisable" icon="pi pi-pencil" />
                        <Dropdown id="country" v-model="edit.country" :disabled="edit.countryDisable" :options="countries" optionLabel="name" :filter="true" placeholder="Seleccione su país" :showClear="true" :class="{'p-invalid':vEdit$.edit.country.$invalid && edit.submitted}">
                            <template #value="edit">
                            <div id="country-item" class="country-item country-item-value" v-if="edit.value">
                                <img src="images/flags/flag_placeholder.png" :class="'flag flag-' + edit.value.code.toLowerCase()" />
                                {{edit.value.name}}
                            </div>
                            <span v-else>
                                {{edit.placeholder}}
                            </span>
                            </template>
                            <template #option="slotProps">
                            <div class="country-item">
                                <img src="images/flags/flag_placeholder.png" :class="'flag flag-' + slotProps.option.code.toLowerCase()" style="height: auto !important"/>
                                {{slotProps.option.name}}
                            </div>
                            </template>
                        </Dropdown>
                        <span class="p-inputgroup-addon">
                            <i class="pi pi-globe"></i>
                        </span>
                        </div>
                        <small v-if="(vEdit$.edit.country.$invalid && edit.submitted) || vEdit$.edit.country.$pending.$response" class="p-error">{{'Por favor, indique su país de residencia'}}</small>
                    </div>
                    <!--PICTURE-->
                    <div class="field"> 
                        <label for="imagen">Foto de perfil</label>
                        <div class="p-inputgroup">
                        <FileUpload id="image"  class="resize" style="width : 440px;" @change="uploadFile" @click="Images=''" chooseLabel="Subir foto" ref="file" mode="basic" url="./upload" :maxFileSize="1000000" accept="image/*"/>
                        </div>
                    </div>
                    <!--PASSWORD-->
                    <div class="field"> 
                        <label for="password" :class="{'p-error':(vEdit$.edit.password.$invalid && edit.submitted && !edit.passwordDisable) || (vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '')}">Contraseña</label>
                        <div class="p-inputgroup">
                        <Button class="col-fixed" style="border-top-right-radius: 0; border-bottom-right-radius: 0" v-on:click="edit.passwordDisable = !edit.passwordDisable; editDialog.isActive = !editDialog.isActive; edit.password = ''" icon="pi pi-pencil" />
                        <Password id="password" v-model="edit.password" placeholder="Contraseña" :disabled="edit.passwordDisable" :class="{'p-invalid':(vEdit$.edit.password.$invalid && edit.submitted && !edit.passwordDisable) || (vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '')}" toggleMask>
                            <template #header>
                            <h6>Elija una contraseña</h6>
                            </template>
                            <template #footer="sp">
                            {{sp.level}}
                            <Divider />
                                <small v-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model == '' && edit.submitted && !edit.passwordDisable) || vEdit$.edit.password.$pending.$response" class="p-error">{{vEdit$.edit.password.required.$message.replace('Value is required', 'Por favor, especifique una contraseña')}}</small>
                                <small v-else-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '' && vEdit$.edit.password.$model.length < 8)" class="p-error"> {{'La contraseña debe tener al menos 8 caracteres'}} </small>
                                <small v-else-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '')" class="p-error"> {{vEdit$.edit.password.alpha.$message}} </small>
                                <p class="mt-2">Obligatorio</p>
                                <ul class="pl-2 ml-2 mt-0" style="line-height: 1.5">
                                <li>Debe contener 8 caracteres como mínimo</li>
                                <li>Debe contener mayúsculas</li>
                                <li>Debe contener minúsculas</li>
                                <li>Debe contener números</li>
                            </ul>
                            </template>
                        </Password>
                        <span class="p-inputgroup-addon">
                            <i class="pi pi-key"></i>
                        </span>
                        </div>
                        <small v-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model == '' && edit.submitted && !edit.passwordDisable) || vEdit$.edit.password.$pending.$response" class="p-error">{{vEdit$.edit.password.required.$message.replace('Value is required', 'Por favor, especifique una contraseña')}}</small>
                        <small v-else-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '' && vEdit$.edit.password.$model.length < 8)" class="p-error"> {{'La contraseña debe tener al menos 8 caracteres'}} </small>
                        <small v-else-if="(vEdit$.edit.password.$invalid && vEdit$.edit.password.$model != '')" class="p-error"> {{vEdit$.edit.password.alpha.$message}} </small>
                    </div>
                    <!--PASSWORD-->
                    <div v-if="!edit.passwordDisable" class="field">
                        <label for="confPassword" :class="{'p-error':(vEdit$.edit.confPassword.$invalid && edit.submitted) || (vEdit$.edit.password.$model != vEdit$.edit.confPassword.$model)}">Confirmar contraseña</label>
                        <div class="p-inputgroup">
                        <Password id="confPassword" v-model="edit.confPassword" placeholder="Confirmar contraseña" :feedback="false" :class="{'p-invalid':(vEdit$.edit.confPassword.$invalid && edit.submitted) || (vEdit$.edit.password.$model != vEdit$.edit.confPassword.$model)}" toggleMask></Password>
                        <span class="p-inputgroup-addon">
                            <i class="pi pi-key"></i>
                        </span>
                        </div>
                        <small v-if="(vEdit$.edit.confPassword.$invalid && edit.submitted) || vEdit$.edit.confPassword.$pending.$response" class="p-error">{{'Por favor, especifique una contraseña'}}</small>
                        <small v-else-if="(vEdit$.edit.password.$model != vEdit$.edit.confPassword.$model && edit.submitted)" class="p-error">{{'Las contraseñas no coinciden'}}</small>
                    </div>
                    <Button type="submit" :disabled="this.edit.editing || (this.vEdit$.edit.nickname.$invalid || this.vEdit$.edit.name.$invalid || this.vEdit$.edit.date.$invalid || this.vEdit$.edit.country.$invalid) || (!this.edit.passwordDisable && (this.vEdit$.edit.$invalid))" class="mt-2 mb-2 p-button-raised font-semibold h-3rem" style="border-radius: 1rem">
                        <div class="flex justify-content-center flex-wrap card-container w-full">
                            <div id="spinner" class="flex align-items-center justify-content-center mr-2">
                                <ProgressSpinner v-if="this.edit.editing" style="width:20px; height:20px" strokeWidth="8" fill="transparent" animationDuration="2s"/>
                            </div>                   
                            <div v-if="!this.edit.editing" class="flex align-items-center justify-content-center font-bold text-white">Guardar cambios</div>
                            <div v-else class="flex align-items-center justify-content-center font-bold text-white">Guardando cambios...</div>
                        </div>
                    </Button>

                    <Divider class="p-divider-center" layout="horizontal">
                        <b>O bien</b>
                    </Divider>
                </form>
                    <Button v-on:click="confirm()" type="submit" label="Eliminar cuenta" class="mt-2 w-full p-button-raised font-semibold h-3rem bg-pink-500 border-pink-500" style="border-radius: 1rem" />
            </div>
        </div>
        <template #footer></template>
    </Dialog>
</template>

<script>
import { useVuelidate } from "@vuelidate/core";
import { required, minLength, maxLength, helpers } from "@vuelidate/validators";
const alpha = helpers.regex(/^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])([a-zA-Z0-9]+)$/);

export default {
    inject: ['$accounts'],
    props: ['perfil'],
    setup () {
        return {
        vEdit$: useVuelidate(),
        }
    },
    data() {
        return {
            editDialog: {
                display: false,
                isActive: true,
            },
            edit: {
                nickname: '',
                nicknameDisable: true,
                name:'',
                nameDisable: true,
                date: '',
                maxDate: '',
                dateDisable: true,
                country: '',
                countryDisable: true,
                password: "",
                confPassword: '',
                passwordDisable: true,
                submitted: false,
                editing: false,
            },
            Images: '',
            countries: '',
        }
    },
    created() {
        this.$accounts.getCountries().then(data => {
            this.countries = data;
        });
    },
    methods: {
        handleSubmit() {
            this.edit.submitted = true;
            this.edit.editing = true;

            //No hemos cambiado el password passwordDisable = true
            if (this.edit.passwordDisable) {
                //console.log('NO ME CAMBIAS EL PASS');
                if (this.vEdit$.edit.nickname.$invalid || this.vEdit$.edit.name.$invalid || this.vEdit$.edit.date.$invalid || this.vEdit$.edit.country.$invalid ) {
                    //console.log('NO VALE Y NO ME CAMBIAS EL PASS');
                    this.edit.editing = false;
                    return;
                }
            } else if (this.vEdit$.edit.nickname.$invalid || this.vEdit$.edit.name.$invalid || this.vEdit$.edit.date.$invalid || this.vEdit$.edit.country.$invalid  || this.vEdit$.edit.password.$invalid || this.vEdit$.edit.confPassword.$invalid) {
                //console.log('NO VALE Y ME CAMBIAS EL PASS');
                this.edit.editing = false;
                return;
            }
            // La form ha sido validada correctamente en front
            //console.log('Formulario validado correctamente');

            //Sumarle 1 a la fecha xD
            this.edit.date.setDate(this.edit.date.getDate() + 1);
            this.$accounts.changeProfile(this.edit.nickname, this.edit.name, this.edit.date.toISOString().split('T')[0], this.edit.country.name, this.edit.password,this.Images).then(data => {
                if(data){
                    this.edit.editing = false;
                    this.editDialog.display = false;
                    this.$router.go()
                }
            });
        },
        getEditParameters() {
            this.edit.nickname = this.perfil.nickname;
            this.edit.name = this.perfil.name;

            let auxDate = new Date(this.perfil.birthday);
            let dd = String(auxDate.getDate());
            let mm = String(auxDate.getMonth() + 1); //empieza en 0
            let yyyy = String(auxDate.getFullYear());

            this.edit.date = new Date(dd + '/' + mm + '/' + yyyy);

            this.edit.country = this.perfil.country;
            this.edit.password = '';

            //Volver a desabilitar todos los botones (clicks consecutivos)
            this.$nextTick(() => { this.vEdit$.$reset() });
            this.editDialog.isActive = true;
            this.edit.nicknameDisable = true;
            this.edit.nameDisable = true;
            this.edit.dateDisable = true;
            this.edit.countryDisable = true;
            this.edit.passwordDisable = true;
        },
        uploadFile() {
            this.Images = ''
            let file = this.$refs.file.files[0];
            //console.log(file)
            const reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = () => {
                this.Images = reader.result.split(',')[1];
            };
        },
        confirm() {
            this.$confirm.require({
                message: 'Se borrararán todos sus datos y no podrá recuperarlos.',
                header: '¿Seguro que desea eliminar su cuenta?',
                icon: 'pi pi-exclamation-triangle',
                rejectLabel: 'Si',
				acceptLabel: 'No',
                accept: () => {
					this.$toast.add({severity:'info', summary:'Cuenta no eliminada', detail:'Su cuenta no ha sido eliminada', life: 3000});
                },
                reject: () => {
                    //Borrarle la cuenta
                    this.$accounts.deleteAccount().then(data => {
                        if(data){
                            this.$accounts.logout()
                            this.$router.push("/")
                        }
                    });
                    //this.$toast.add({severity:'error', summary:'Sesión activa', detail:'Sesión no cerrada', life: 3000});
                }
            });
		},
    },
    validations() {
        return {
            //para que autotrackee el estado $dirty
            "$autoDirty": true,
            //que no autovalide hasta que le meta algo osea hasta que $dirty, lo quito porque da problemas (deja enviar forms vacias)
            //"$lazy": true,
            edit: {
                nickname: { required, max: maxLength(15) },
                name: { required },
                date: { required },
                password: { required, min: minLength(8), alpha: helpers.withMessage('Debe contener al menos mayusculas, minusculas y números', alpha) },
                confPassword: { required },
                country: { required },
            },
        }
    },
}
</script>

<style>
.edit-dialog {
    background-color: var(--surface-a); /* Get el current background del tema */
    border-radius: 15px;
    animation-duration: 0.6s;
    animation-name: lineIns derted;
    transition: height 0.6s, width 0.6s;
}

.edit-dialog-sm-size {
    height: 55rem;
    width: 40rem;
}

.edit-dialog-lg-size {
    height: 62rem;
    width: 40rem;
}

/* Color del spinner */
@keyframes p-progress-spinner-color {
    100% {
        stroke: white;
    }
    0% {
        stroke: white;
    }
    40% {
        stroke: white;
    }
    66% {
        stroke: white;
    }
    80% {
        stroke: white;
    }
    90% {
        stroke: white;
    }
}
</style>
