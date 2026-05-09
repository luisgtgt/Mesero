<template>
  <div class="caja">
    <div class="logo"><img :src="logo" alt="" id="pri_logo"></div>
    <button>PRODUCTOS</button>
    <button>MESAS</button>
    <button>FACTURAS</button>
  </div>

  <div class="main">
    <swiper :slides-per-view="1" :space-between="0" @swiper="onSwiper" class="my-swiper">
      <swiper-slide id="pro">
        <div class="productos">
          <div class="producto" v-for="item in listaproductos">
            <h3>id</h3>
            <h1>{{ item.id }}</h1>
            <h3>nombre</h3>
            <h1>{{ item.nombre }}</h1>
            <h3>cantidad</h3>
            <h1>{{ item.cantidad }}</h1>
            <h3>precio</h3>
            <h1>{{ item.precio }}</h1>
            <button @click="editar(item.id)">editar</button>
            <button @click="eliminar(item.id)">eliminar</button>
          </div>
          <button class="add_producto" @click="abrirmodal">AGREGAR PRODUCTO</button>
        </div>
      </swiper-slide>

      <swiper-slide>
        <div class="section mesas">
          <h2>Control de Mesas</h2>
        </div>
      </swiper-slide>

      <swiper-slide>
        <div class="section facturacion">
          <h2>Facturación</h2>
        </div>
      </swiper-slide>
    </swiper>
  </div>

  <div class="modal_productos" v-if="mostrar">
    <div class="contenedor">
      <h1>{{ titulo1 }}</h1>
      <button @click="cerrarmodal">❌</button>
      <label for="">ID</label>
      <input v-model.number="id" type="number">
      <label for="">nombre</label>
      <input v-model="nombre" type="text">
      <label for="">precio</label>
      <input v-model.number="precio" type="number">
      <label for="">cantidad</label>
      <input v-model.number="cantidad" type="number">
      <button @click="addboton()">AGREGAR</button>
    </div>
  </div>

</template>

<script setup>
import { ref } from 'vue'
import logo from './assets/logotra.png'
let mostrar = ref(false)
let listaproductos = ref([])
let titulo1 = ref("AGREGAR PRODUCTO")
let addboton = ref(() => addProducto(-1))


//variables de produtos
let id = ref("")
let nombre = ref("")
let precio = ref("")
let cantidad = ref("")

function addProducto(x = -1) {
  if (x == -1) {
    const nuevoproducto = {
      id: id.value,
      nombre: nombre.value,
      precio: precio.value,
      cantidad: cantidad.value
    }

    listaproductos.value.push(nuevoproducto)

    id.value = ""
    nombre.value = ""
    precio.value = ""
    cantidad.value = ""

    cerrarmodal()
  }
  else{
    console.log("hola")
  }


}

function eliminar(x) {
  let posi = listaproductos.value.findIndex((item) => item.id == x)
  console.log("posi")
  listaproductos.value.splice(posi, 1)
}

function editar(x){
  let posi = listaproductos.value.findIndex((item) => item.id == x)
  id.value = listaproductos.value[posi].id
  nombre.value = listaproductos.value[posi].nombre
  precio.value = listaproductos.value[posi].precio
  cantidad.value = listaproductos.value[posi].cantidad
  titulo1 = `EDITAR ARTICULO ${listaproductos.value[posi].nombre}`
  abrirmodal()
  
  addboton.value = () => addProducto(posi) 
}



const abrirmodal = () => {
  mostrar.value = true;
  console.log("se abrio")
}
const cerrarmodal = () => {
  mostrar.value = false;
  titulo1 = "AGREGAR PRODUCTO"
  id.value = ""
  nombre.value = ""
  precio.value = ""
  cantidad.value = ""
  addboton.value = () => addProducto()
}



//importacion de swiper para el movimiento de el menu de los productos, mesas y facturas
import { Swiper, SwiperSlide } from 'swiper/vue';
import 'swiper/css';

const onSwiper = (swiper) => {
  console.log('Swiper inicializado');
};
//fin de la importacion de swiper


</script>

<style>
* {
  margin: 0px;
  padding: 0px
}

#app {
  display: flex;
  flex-direction: column;
  height: 100vh
}

#pri_logo {
  width: 200px;
}

.productos {
  overflow-y: auto;
  flex: 1;
}

/*estilos para el swiper*/
.my-swiper {
  width: 100%;
  height: 100%;
  /* Ocupa toda la pantalla */
  flex-grow: 1;
  overflow-y: auto;
}

.section {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  padding: 20px;
}

.add_producto {
  position: fixed;
  bottom: 10px;
}

.main {
  overflow: hidden;
  display: flex;
  flex: 1;
}

/* Colores para diferenciar mientras pruebas */
.productos {
  background-color: #f8f9fa;
  display: flex;

  .producto {
    display: flex;
    place-items: center;
    flex-direction: column;
    padding: 10px;
    margin: 8px;
    border: 1px solid black;
    border-radius: 10px
  }
}

.mesas {
  background-color: #e9ecef;
}

.facturacion {
  background-color: #dee2e6;
}

/*fin de los estilos de swiper*/

.modal_productos {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  height: 100vh;
  position: absolute;
  display: flex;
  place-items: center;
  place-content: center;
  top: 0;
  left: -50%;
  right: -50%;
  z-index: 10;

  .contenedor {
    justify-content: center;
    display: flex;
    flex-direction: column;
    width: 400px;
  }
}
</style>