<template>
    <div>
        <form @submit.prevent="editarNota(nota)" v-if="editarActivo">
            <h3>Editar tareas</h3>
            <input type="text" class="form-control mb-2" placeholder="Nombre" v-model="nota.nombre">
            <input type="text" class="form-control mb-2" placeholder="Descripcion" v-model="nota.descripcion">
            <button type="submit" class="btn btn-warning">Editar</button>
            <button type="submit" @click="cancelarEdicion()" class="btn btn-danger">Cancelar Edición</button>
        </form>

        <form @submit.prevent="agregar" v-else>
            <h3>Agregar tareas</h3>
            <input type="text" class="form-control mb-2" placeholder="Nombre" v-model="nota.nombre">
            <input type="text" class="form-control mb-2" placeholder="Descripcion" v-model="nota.descripcion">
            <button type="submit" class="btn btn-primary">Agregar</button>
        </form>

        <hr class="mt-3">
        <h3>Listado de notas</h3>
        <ul class="list-group my-2">
            <li class="list-group-item" v-for="(item, index) in notas" :key="index">
                <span class="badge badge-primary float-right">
                    {{item.updated_at}}
                </span>
                <p>{{item.nombre}}</p>
                <p>{{item.descripcion}}</p>
                <button class="btn btn-danger btn-sm" @click="eliminarNota(item, index)">
                    Eliminar
                </button>
                <button class="btn btn-warning btn-sm" @click="editarFormulario(item)">
                    Editar
                </button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    data() {
        return {
            notas: [],
            nota: {nombre: '', descripcion: ''},
            editarActivo: false
        }
    },

    created() {
        axios.get('/notas')
        .then(res => {
            this.notas = res.data;
        })
    },

    methods: {
        editarFormulario(item){
            this.editarActivo = true
            this.nota.nombre = item.nombre
            this.nota.descripcion = item.descripcion
            this.nota.id = item.id
        },

        editarNota(item){
            const params = {
                nombre: item.nombre,
                descripcion: item.descripcion
            };
            axios.put(`/notas/${item.id}`, params)
            .then(res => {
                this.editarActivo = false;
                const index = this.notas.findIndex(
                    notaBuscar => item.id === res.data.id
                )
                this.notas[index] = res.data;
                this.nota = {nombre: '', descripcion: ''};

                axios.get('/notas')
                .then(res => {
                    this.notas = res.data;
                })
            })
        },

        cancelarEdicion(){
           this.editarActivo = false;
           this.nota = {nombre: '', descripcion: ''};
        },

        agregar(){
            if (this.nota.nombre.trim() === '' || this.nota.descripcion.trim() === '') {
                alert('Hay campos vacíos.');
                return;
            }

            console.log(this.nota.nombre, this.nota.descripcion);
            const params = {
                nombre: this.nota.nombre,
                descripcion: this.nota.descripcion
            }
            this.nota.nombre = '';
            this.nota.descripcion = '';
            axios.post('/notas', params)
            .then(res => {
                this.notas.push(res.data)
            })
        },

        eliminarNota(item, index){
            axios.delete(`/notas/${item.id}`)
            .then(()=>{
                this.notas.splice(index, 1)
            })
        }
    },
}
</script>
