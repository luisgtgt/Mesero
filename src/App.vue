<template>
  <div class="caja">
    <div class="logo"><img :src="logo" alt="Logo" id="pri_logo"></div>
    <div class="menu-botones">
      <button @click="irASlide(0)">PRODUCTOS</button>
      <button @click="irASlide(1)">MESAS</button>
      <button @click="irASlide(2)">FACTURAS</button>
    </div>
  </div>

  <div class="main">
    <swiper :slides-per-view="1" :space-between="0" @swiper="onSwiper" class="my-swiper">
      
      <swiper-slide id="pro">
        <div class="productos">
          <div class="contenedor-busqueda">
            <input 
              v-model="buscarProducto" 
              type="text" 
              placeholder="🔍 Buscar producto por nombre o ID..." 
              class="input-busqueda"
            />
          </div>

          <div class="grid-items" v-if="productosFiltrados.length > 0">
            <div class="producto" v-for="item in productosFiltrados" :key="item.id">
              <div class="producto-info">
                <h3>ID: {{ item.id }}</h3>
                <h1>{{ item.nombre }}</h1>
                <p>Precio: {{ formatearMoneda(item.precio) }}</p>
                <p>Stock: {{ item.cantidad }} u.</p>
              </div>
              <div class="acciones">
                <button @click="editarProducto(item.id)" class="btn-editar">Editar</button>
                <button @click="eliminarProducto(item.id)" class="btn-eliminar">Eliminar</button>
              </div>
            </div>
          </div>
          <p v-else class="vacio">No se encontraron productos que coincidan.</p>

          <button class="add_btn_flotante" @click="abrirModalForm('producto')">+ PRODUCTO</button>
        </div>
      </swiper-slide>

      <swiper-slide>
        <div class="mesas-container">
          <div class="contenedor-busqueda">
            <input 
              v-model="buscarMesa" 
              type="text" 
              placeholder="🔍 Buscar mesa por número o estado (ej: disponible)..." 
              class="input-busqueda"
            />
          </div>

          <div class="grid-items" v-if="mesasFiltradas.length > 0">
            <div class="mesa-card" v-for="mesa in mesasFiltradas" :key="mesa.id" :class="mesa.estado">
              
              <div class="admin-mesa-acciones" v-if="mesa.estado === 'disponible'">
                <button @click="editarMesa(mesa.id)" class="btn-mini-accion" title="Editar Mesa">⚙️</button>
                <button @click="eliminarMesa(mesa.id)" class="btn-mini-accion btn-mini-eliminar" title="Eliminar Mesa">🗑️</button>
              </div>

              <div class="mesa-header-info">
                <h2>Mesa {{ mesa.numero }}</h2>
                <span class="badge">{{ mesa.estado.toUpperCase() }}</span>
              </div>
              
              <div class="pedido-detalle" v-if="mesa.pedidos.length > 0">
                <h4>Consumo:</h4>
                <ul>
                  <li v-for="p in mesa.pedidos" :key="p.id">
                    {{ p.nombre }} x{{ p.cantidadPedida }} ({{ formatearMoneda(p.precio * p.cantidadPedida) }})
                  </li>
                </ul>
                <p class="total-actual">Total: {{ formatearMoneda(calcularTotalMesa(mesa)) }}</p>
              </div>

              <div class="acciones-mesa">
                <button v-if="mesa.estado === 'disponible'" @click="abrirMesa(mesa.id)" class="btn-abrir">Abrir Mesa</button>
                <div v-else class="controles-activas">
                  <button @click="prepararAgregarPedido(mesa.id)" class="btn-pedir">+ Pedido</button>
                  <button @click="iniciarProcesoFacturacion(mesa.id)" class="btn-facturar">Facturar</button>
                  <button @click="cerrarmesa(mesa.id)" class="btn-cerrar-mesas">Cerrar mesa</button>
                </div>
              </div>
            </div>
          </div>
          <p v-else class="vacio">No se encontraron mesas que coincidan.</p>

          <button class="add_btn_flotante" @click="abrirModalForm('mesa')">+ MESA</button>
        </div>
      </swiper-slide>

      <swiper-slide>
        <div class="section-facturacion">
          <h2>Historial de Facturación</h2>
          
          <div class="contenedor-busqueda">
            <input 
              v-model="buscarFactura" 
              type="text" 
              placeholder="🔍 Buscar factura por ID, cliente o mesa..." 
              class="input-busqueda"
            />
          </div>

          <div class="tabla-facturas" v-if="facturasFiltradas.length > 0">
            <div class="factura-card-wrapper" v-for="factura in facturasFiltradas" :key="factura.id">
              
              <div :id="'factura-print-' + factura.id" class="factura-item">
                <div class="factura-header">
                  <strong>Factura #{{ factura.id }}</strong>
                  <span>{{ factura.fecha }}</span>
                </div>
                <p class="factura-origen"><strong>Mesa origen:</strong> {{ factura.numeroMesa }}</p>
                <p class="factura-cliente"><strong>Cliente:</strong> {{ factura.cliente.nombre }} {{ factura.cliente.documento ? `(CC/NIT: ${factura.cliente.documento})` : '' }}</p>
                <ul class="factura-items-list">
                  <li v-for="item in factura.items" :key="item.id">
                    {{ item.nombre }} x{{ item.cantidadPedida }} — {{ formatearMoneda(item.precio * item.cantidadPedida) }}
                  </li>
                </ul>
                <h3 class="factura-total">Total: {{ formatearMoneda(factura.total) }}</h3>
              </div>
              
              <div class="factura-acciones">
                <button @click="descargarPDF(factura)" class="btn-pdf">📥 Guardar PDF</button>
              </div>
            </div>
          </div>
          <p v-else class="vacio">No se encontraron facturas registradas o que coincidan.</p>
        </div>
      </swiper-slide>
    </swiper>
  </div>

  <div class="modal_productos" v-if="mostrarModal">
    <div class="contenedor">
      <h1>{{ tituloModal }}</h1>
      <button class="btn-cerrar" @click="cerrarModal">❌</button>
      
      <div class="error-banner" v-if="mensajeErrorModal">{{ mensajeErrorModal }}</div>
      
      <div v-if="tipoFormulario === 'producto'" class="form-group">
        <label>ID del Producto</label>
        <input v-model.number="formProducto.id" type="number" :disabled="idEditando !== -1">
        <label>Nombre</label>
        <input v-model="formProducto.nombre" type="text">
        <label>Precio ($)</label>
        <input v-model.number="formProducto.precio" type="number">
        <label>Cantidad en Stock</label>
        <input v-model.number="formProducto.cantidad" type="number">
      </div>

      <div v-if="tipoFormulario === 'mesa'" class="form-group">
        <label>Número o Nombre de la Mesa</label>
        <input v-model="formMesa.numero" type="text" placeholder="Ej: 1, VIP-1">
      </div>

      <button class="btn-guardar" @click="guardarFormulario">AGREGAR / GUARDAR</button>
    </div>
  </div>

  <div class="modal_productos" v-if="mostrarModalPedido">
    <div class="contenedor">
      <h1>Pedido - Mesa {{ mesaSeleccionada?.numero }}</h1>
      <button class="btn-cerrar" @click="mostrarModalPedido = false; mensajeErrorPedido = ''">❌</button>
      
      <div class="error-banner" v-if="mensajeErrorPedido">{{ mensajeErrorPedido }}</div>
      
      <label>Selecciona el Producto</label>
      <select v-model="pedidoTemporal.productoId">
        <option value="" disabled>-- Selecciona --</option>
        <option v-for="prod in listaproductos" :key="prod.id" :value="prod.id" :disabled="prod.cantidad <= 0">
          {{ prod.nombre }} ({{ formatearMoneda(prod.precio) }})
        </option>
      </select>

      <label>Cantidad</label>
      <input v-model.number="pedidoTemporal.cantidad" type="number" min="1">

      <button class="btn-guardar" @click="agregarProductoAMesa">Confirmar Pedido</button>
    </div>
  </div>

  <div class="modal_productos" v-if="mostrarModalTipoFactura">
    <div class="contenedor">
      <h1>Facturar Mesa {{ mesaAFacturar?.numero }}</h1>
      <button class="btn-cerrar" @click="mostrarModalTipoFactura = false">❌</button>
      <p style="font-size: 0.95rem; color: #555; text-align: center; margin: 10px 0;">¿Cómo deseas emitir esta factura?</p>
      
      <div class="opciones-factura">
        <button @click="facturarAnonimo" class="btn-opcion-anonimo">👤 Consumidor Final (Anónimo)</button>
        <button @click="irADatosCliente" class="btn-opcion-cliente">📝 Registrar Datos de Cliente</button>
      </div>
    </div>
  </div>

  <div class="modal_productos" v-if="mostrarModalDatosCliente">
    <div class="contenedor">
      <h1>Datos del Cliente</h1>
      <button class="btn-cerrar" @click="mostrarModalDatosCliente = false">❌</button>
      
      <div class="error-banner" v-if="mensajeErrorCliente">{{ mensajeErrorCliente }}</div>
      
      <div class="form-group">
        <label>Nombre Completo / Razón Social</label>
        <input v-model="formCliente.nombre" type="text" placeholder="Ej: Juan Pérez">
        
        <label>Cédula o NIT</label>
        <input v-model="formCliente.documento" type="text" placeholder="Ej: 1010XXXXXX">
      </div>

      <button class="btn-guardar" @click="facturarConDatos">Generar Factura</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import logo from './assets/logotra.png'
import { Swiper, SwiperSlide } from 'swiper/vue';
import 'swiper/css';
import html2pdf from 'html2pdf.js'

// --- FORMATO DE MONEDA CON PUNTUACIÓN DE MILES ---
const formatearMoneda = (valor) => {
  if (valor === undefined || valor === null || isNaN(valor)) 
  return '$0'

  return new Intl.NumberFormat('es-CO', {
    style: 'currency',
    currency: 'COP',
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  }).format(valor)
}

// --- CONFIGURACIÓN E INICIALIZACIÓN DE LOCAL STORAGE ---
const listaproductos = ref(JSON.parse(localStorage.getItem('listaproductos')) || [
  { id: 1, nombre: 'Hamburguesa', precio: 15000, cantidad: 20 },
  { id: 2, nombre: 'Cerveza Club', precio: 5000, cantidad: 50 }
])

const listamesas = ref(JSON.parse(localStorage.getItem('listamesas')) || [
  { id: 1, numero: '1', estado: 'disponible', pedidos: [] },
  { id: 2, numero: '2', estado: 'disponible', pedidos: [] }
])

const listafacturas = ref(JSON.parse(localStorage.getItem('listafacturas')) || [])

const actualizarLocalStorage = () => {
  localStorage.setItem('listaproductos', JSON.stringify(listaproductos.value))
  localStorage.setItem('listamesas', JSON.stringify(listamesas.value))
  localStorage.setItem('listafacturas', JSON.stringify(listafacturas.value))
}

// --- ESTADOS DE BÚSQUEDA ---
const buscarProducto = ref('')
const buscarMesa = ref('')
const buscarFactura = ref('')

// --- PROPIEDADES COMPUTADAS PARA FILTRADO EN TIEMPO REAL ---
const productosFiltrados = computed(() => {
  if (!buscarProducto.value.trim()) return listaproductos.value
  const query = buscarProducto.value.toLowerCase().trim()
  return listaproductos.value.filter(p => 
    p.nombre.toLowerCase().includes(query) || 
    p.id.toString().includes(query)
  )
})

const mesasFiltradas = computed(() => {
  if (!buscarMesa.value.trim()) return listamesas.value
  const query = buscarMesa.value.toLowerCase().trim()
  return listamesas.value.filter(m => 
    m.numero.toLowerCase().includes(query) || 
    m.estado.toLowerCase().includes(query)
  )
})

const facturasFiltradas = computed(() => {
  if (!buscarFactura.value.trim()) return listafacturas.value
  const query = buscarFactura.value.toLowerCase().trim()
  return listafacturas.value.filter(f => 
    f.id.toString().includes(query) || 
    f.numeroMesa.toLowerCase().includes(query) || 
    f.cliente.nombre.toLowerCase().includes(query)
  )
})

// --- ESTADOS DE VALIDACIÓN ---
const mensajeErrorModal = ref('')
const mensajeErrorPedido = ref('')
const mensajeErrorCliente = ref('')

// --- MODALES GENERALES ---
const mostrarModal = ref(false)
const tipoFormulario = ref('producto')
const idEditando = ref(-1)

const formProducto = ref({ id: '', nombre: '', precio: '', cantidad: '' })
const formMesa = ref({ numero: '' })

// --- NUEVOS ESTADOS PARA FACTURACIÓN EN DOS PASOS ---
const mostrarModalTipoFactura = ref(false)
const mostrarModalDatosCliente = ref(false)
const mesaAFacturar = ref(null)
const formCliente = ref({ nombre: '', documento: '' })

const tituloModal = computed(() => {
  if (tipoFormulario.value === 'mesa') {
    return idEditando.value === -1 ? "NUEVA MESA" : "EDITAR MESA"
  }
  return idEditando.value === -1 ? "AGREGAR PRODUCTO" : "EDITAR PRODUCTO"
})

const abrirModalForm = (tipo) => {
  tipoFormulario.value = tipo
  mensajeErrorModal.value = ''
  mostrarModal.value = true
}

const cerrarModal = () => {
  mostrarModal.value = false
  idEditando.value = -1
  mensajeErrorModal.value = ''
  formProducto.value = { id: '', nombre: '', precio: '', cantidad: '' }
  formMesa.value = { numero: '' }
}

const guardarFormulario = () => {
  mensajeErrorModal.value = ''
  
  if (tipoFormulario.value === 'producto') {
    if (!formProducto.value.id || !formProducto.value.nombre || !formProducto.value.precio || formProducto.value.cantidad === '') {
      mensajeErrorModal.value = 'Todos los campos son obligatorios.'
      return
    }
    if (formProducto.value.precio <= 0 || formProducto.value.cantidad < 0) {
      mensajeErrorModal.value = 'El precio debe ser mayor a 0 y el stock no puede ser negativo.'
      return
    }

    if (idEditando.value === -1) {
      const existeId = listaproductos.value.some(p => p.id === formProducto.value.id)
      if (existeId) {
        mensajeErrorModal.value = 'El ID ingresado ya pertenece a otro producto.'
        return
      }
      listaproductos.value.push({ ...formProducto.value })
    } else {
      listaproductos.value[idEditando.value] = { ...formProducto.value }
    }
    
  } else if (tipoFormulario.value === 'mesa') {
    if (!formMesa.value.numero || formMesa.value.numero.toString().trim() === '') {
      mensajeErrorModal.value = 'El nombre o número de la mesa es requerido.'
      return
    }

    if (idEditando.value === -1) {
      listamesas.value.push({
        id: Date.now(),
        numero: formMesa.value.numero,
        estado: 'disponible',
        pedidos: []
      })
    } else {
      listamesas.value[idEditando.value].numero = formMesa.value.numero
    }
  }
  
  actualizarLocalStorage()
  cerrarModal()
}

// --- PRODUCTOS ---
const editarProducto = (prodId) => {
  const index = listaproductos.value.findIndex(p => p.id === prodId)
  if (index !== -1) {
    formProducto.value = { ...listaproductos.value[index] }
    idEditando.value = index
    abrirModalForm('producto')
  }
}

const eliminarProducto = (prodId) => {
  listaproductos.value = listaproductos.value.filter(p => p.id !== prodId)
  actualizarLocalStorage()
}

// --- MESAS ACCIONES ---
const editarMesa = (mesaId) => {
  const index = listamesas.value.findIndex(m => m.id === mesaId)
  if (index !== -1) {
    formMesa.value = { numero: listamesas.value[index].numero }
    idEditando.value = index
    abrirModalForm('mesa')
  }
}

const eliminarMesa = (mesaId) => {
  listamesas.value = listamesas.value.filter(m => m.id !== mesaId)
  actualizarLocalStorage()
}

// --- MANEJO DE PEDIDOS Y APERTURA ---
const mostrarModalPedido = ref(false)
const mesaSeleccionada = ref(null)
const pedidoTemporal = ref({ productoId: '', cantidad: 1 })

const abrirMesa = (mesaId) => {
  const mesa = listamesas.value.find(m => m.id === mesaId)
  if (mesa) {
    mesa.estado = 'ocupada'
    actualizarLocalStorage()
  }
}

const prepararAgregarPedido = (mesaId) => {
  mesaSeleccionada.value = listamesas.value.find(m => m.id === mesaId)
  pedidoTemporal.value = { productoId: '', cantidad: 1 }
  mensajeErrorPedido.value = ''
  mostrarModalPedido.value = true
}

const agregarProductoAMesa = () => {
  mensajeErrorPedido.value = ''
  
  if (!pedidoTemporal.value.productoId) {
    mensajeErrorPedido.value = 'Debes seleccionar un producto del catálogo.'
    return
  }
  if (!pedidoTemporal.value.cantidad || pedidoTemporal.value.cantidad <= 0) {
    mensajeErrorPedido.value = 'La cantidad debe ser mayor o igual a 1.'
    return
  }

  const prod = listaproductos.value.find(p => p.id === pedidoTemporal.value.productoId)
  if (!prod) return

  if (prod.cantidad < pedidoTemporal.value.cantidad) {
    mensajeErrorPedido.value = `Unidades insuficientes. El stock disponible actual es de ${prod.cantidad} u.`
    return
  }

  prod.cantidad -= pedidoTemporal.value.cantidad

  const itemExistente = mesaSeleccionada.value.pedidos.find(p => p.id === prod.id)
  if (itemExistente) {
    itemExistente.cantidadPedida += pedidoTemporal.value.cantidad
  } else {
    mesaSeleccionada.value.pedidos.push({
      id: prod.id,
      nombre: prod.nombre,
      precio: prod.precio,
      cantidadPedida: pedidoTemporal.value.cantidad
    })
  }
  
  actualizarLocalStorage()
  mostrarModalPedido.value = false
}

const calcularTotalMesa = (mesa) => {
  return mesa.pedidos.reduce((acc, item) => acc + (item.precio * item.cantidadPedida), 0)
}

// --- LÓGICA DE FLUJO SECUENCIAL DE FACTURACIÓN ---
const iniciarProcesoFacturacion = (mesaId) => {
  const mesa = listamesas.value.find(m => m.id === mesaId)
  if (!mesa || mesa.pedidos.length === 0) return
  
  mesaAFacturar.value = mesa
  mostrarModalTipoFactura.value = true
}

const facturarAnonimo = () => {
  ejecutarFacturacion({ nombre: 'Anónimo / Consumidor Final', documento: '' })
  mostrarModalTipoFactura.value = false
}

const irADatosCliente = () => {
  mostrarModalTipoFactura.value = false
  formCliente.value = { nombre: '', documento: '' }
  mensajeErrorCliente.value = ''
  mostrarModalDatosCliente.value = true
}

const facturarConDatos = () => {
  if (!formCliente.value.nombre.trim()) {
    mensajeErrorCliente.value = 'El nombre o razón social es obligatorio.'
    return
  }
  
  ejecutarFacturacion({ 
    nombre: formCliente.value.nombre.trim(), 
    documento: formCliente.value.documento.trim() 
  })
  mostrarModalDatosCliente.value = false
}

const ejecutarFacturacion = (datosCliente) => {
  const mesa = mesaAFacturar.value
  if (!mesa) return

  listafacturas.value.push({
    id: listafacturas.value.length + 1,
    numeroMesa: mesa.numero,
    cliente: datosCliente,
    items: [...mesa.pedidos],
    total: calcularTotalMesa(mesa),
    fecha: new Date().toLocaleString()
  })
  
  mesa.estado = 'disponible'
  mesa.pedidos = []
  mesaAFacturar.value = null
  
  actualizarLocalStorage()
  irASlide(2) 
}

const cerrarmesa = (mesaId) => {
  const mesa = listamesas.value.find(m => m.id === mesaId)
  if (!mesa) return

  const seguro = confirm(`¿Estás seguro de cerrar la Mesa ${mesa.numero}? Se cancelarán los pedidos actuales y se devolverá el stock.`)
  if (!seguro) return

  mesa.pedidos.forEach(pedidoMesa => {
    const productoCatalogo = listaproductos.value.find(p => p.id === pedidoMesa.id)
    if (productoCatalogo) {
      productoCatalogo.cantidad += pedidoMesa.cantidadPedida
    }
  })

  mesa.estado = 'disponible'
  mesa.pedidos = []
  actualizarLocalStorage()
}

const descargarPDF = (factura) => {
  const elemento = document.getElementById(`factura-print-${factura.id}`);
  if (!elemento) return;
  const opciones = {
    margin: 10,
    filename: `Factura_${factura.id}_Mesa_${factura.numeroMesa}.pdf`,
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2, useCORS: true },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  };
  html2pdf().set(opciones).from(elemento).save();
}

// --- SWIPER ---
const swiperInstance = ref(null)
const onSwiper = (swiper) => { swiperInstance.value = swiper }
const irASlide = (index) => { if (swiperInstance.value) swiperInstance.value.slideTo(index) }
</script>

<style>
/* CONFIGURACIÓN Y RESETS GLOBALES RESPONSIVOS */
* { 
  box-sizing: border-box; 
  margin: 0; 
  padding: 0; 
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
}

#app { 
  display: flex; 
  flex-direction: column; 
  height: 100vh; 
  min-width: 300px;
  overflow-x: hidden;
}

/* ENCABEZADO / HEADER */
.caja { 
  display: flex; 
  flex-direction: column;
  align-items: center; 
  background: #1e1e1e; 
  padding: 12px; 
  gap: 10px;
  border-bottom: 3px solid #007bff;
}

#pri_logo { 
  width: 130px; 
  height: auto;
  object-fit: contain;
}

.menu-botones {
  display: flex;
  gap: 6px;
  width: 100%;
  justify-content: center;
}

.menu-botones button { 
  flex: 1;
  max-width: 110px;
  padding: 8px 4px; 
  background: #333; 
  border: 1px solid #444; 
  color: white; 
  cursor: pointer; 
  border-radius: 6px; 
  font-weight: bold; 
  font-size: 0.75rem;
  text-transform: uppercase;
  transition: all 0.2s ease;
}

.menu-botones button:hover { 
  background: #007bff; 
  border-color: #007bff;
}

/* ESTILOS DE LA BARRA DE BÚSQUEDA ADICIONADA */
.contenedor-busqueda {
  width: 100%;
  margin-bottom: 15px;
}

.input-busqueda {
  width: 100%;
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 0.95rem;
  background-color: #ffffff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.04);
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.input-busqueda:focus {
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.15);
}

/* CONTENEDOR PRINCIPAL */
.main { 
  flex: 1; 
  display: flex; 
  overflow: hidden; 
  background-color: #f4f6f9;
}
.my-swiper { 
  width: 100%; 
  height: 100%; 
}

/* DISEÑO DE CONTENEDORES INTERNOS */
.productos, .mesas-container, .section-facturacion { 
  padding: 15px; 
  overflow-y: auto; 
  height: 100%; 
  position: relative;
  padding-bottom: 80px; 
}

.productos { background-color: #f8f9fa; }
.mesas-container { background-color: #e9ecef; }
.section-facturacion { background-color: #dee2e6; }

.section-facturacion h2 {
  font-size: 1.3rem;
  margin-bottom: 15px;
  color: #333;
}

/* REJILLA (GRID) INTELIGENTE PARA TARJETAS */
.grid-items { 
  display: grid; 
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr)); 
  gap: 15px; 
  width: 100%;
}

/* TARJETAS DE PRODUCTO Y MESA */
.producto, .mesa-card { 
  background: white; 
  padding: 15px; 
  border-radius: 10px; 
  box-shadow: 0 3px 6px rgba(0,0,0,0.08); 
  border: 1px solid #e0e0e0; 
  display: flex; 
  flex-direction: column; 
  justify-content: space-between;
  gap: 12px;
  position: relative;
}

.producto-info h1 { font-size: 1.25rem; color: #222; margin: 4px 0; }
.producto-info h3 { font-size: 0.75rem; color: #888; }
.producto-info p { font-size: 0.9rem; color: #555; }

/* CONTROLES DE MESAS */
.mesa-card { border-top: 5px solid #6c757d; }
.mesa-card.disponible { border-top-color: #28a745; }
.mesa-card.ocupada { border-top-color: #dc3545; }

/* ACCIONES DE ADMINISTRACIÓN EN LA TARJETA DE LA MESA */
.admin-mesa-acciones {
  position: absolute;
  top: -18px;
  right: 10px;
  display: flex;
  gap: 4px;
  z-index: 5;
}
.btn-mini-accion {
  background: #fff;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 2px 6px;
  font-size: 0.75rem;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: transform 0.1s ease;
}
.btn-mini-accion:hover {
  transform: scale(1.1);
  background: #f0f0f0;
}
.btn-mini-eliminar:hover {
  background: #ffebee;
  border-color: #f5c6cb;
}

.mesa-header-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 5px;
}
.mesa-header-info h2 { font-size: 1.2rem; }

.badge { 
  padding: 4px 8px; 
  font-size: 0.7rem; 
  border-radius: 4px; 
  font-weight: bold; 
  color: white; 
}
.disponible .badge { background: #28a745; }
.ocupada .badge { background: #dc3545; }

.pedido-detalle { 
  background: #f8f9fa; 
  padding: 10px; 
  border-radius: 6px; 
  font-size: 0.85rem; 
}
.pedido-detalle ul { list-style: none; margin-top: 5px; }
.pedido-detalle li { margin-bottom: 3px; color: #555; }
.total-actual { 
  font-weight: bold; 
  border-top: 1px dashed #ccc; 
  margin-top: 8px; 
  padding-top: 6px; 
  color: #000; 
}

/* BOTONES INTERNOS DE ACCIÓN */
.acciones, .acciones-mesa { display: flex; gap: 6px; }
.acciones button, .acciones-mesa button { 
  flex: 1; 
  padding: 8px; 
  font-size: 0.8rem; 
  font-weight: bold;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.btn-editar { background: #e3f2fd; color: #0d6efd; }
.btn-eliminar { background: #ffebee; color: #dc3545; }
.btn-abrir { background: #28a745; color: white; }
.controles-activas { display: flex; gap: 6px; width: 100%; }
.btn-pedir { background: #ffc107; color: #212529; flex: 1; }
.btn-facturar { background: #17a2b8; color: white; flex: 1; }
.btn-cerrar-mesas {
  background: #dc3545;
  color: white;
  flex: 0.5;
}
.btn-cerrar-mesas:hover{
  background: #bd2130;
}

/* HISTORIAL DE FACTURAS */
.tabla-facturas { 
  display: flex; 
  flex-direction: column; 
  gap: 15px; 
}
.factura-card-wrapper { 
  background: white; 
  border-radius: 10px; 
  padding: 15px; 
  box-shadow: 0 2px 5px rgba(0,0,0,0.05);
}
.factura-item { background: white; }
.factura-header { 
  display: flex; 
  justify-content: space-between; 
  font-size: 0.8rem; 
  color: #666; 
  border-bottom: 1px dashed #ddd;
  padding-bottom: 6px;
  margin-bottom: 10px;
}
.factura-origen { font-size: 0.9rem; margin-bottom: 3px; }
.factura-cliente { font-size: 0.9rem; margin-bottom: 8px; color: #0d6efd; }
.factura-items-list { padding-left: 15px; font-size: 0.85rem; margin-bottom: 10px; color: #444; }
.factura-total { font-size: 1.1rem; border-top: 2px solid #333; padding-top: 5px; display: inline-block; }
.factura-acciones { display: flex; justify-content: flex-end; margin-top: 10px; border-top: 1px solid #eee; padding-top: 8px; }
.btn-pdf { background: #007bff; color: white; border: none; padding: 6px 12px; border-radius: 4px; font-weight: bold; cursor: pointer; font-size: 0.8rem; }

/* BOTÓN FLOTANTE */
.add_btn_flotante { 
  position: fixed; 
  bottom: 20px; 
  right: 20px; 
  padding: 12px 20px; 
  background: #007bff; 
  color: white; 
  border: none; 
  border-radius: 50px; 
  font-weight: bold; 
  cursor: pointer; 
  z-index: 99; 
  box-shadow: 0 4px 10px rgba(0,0,0,0.25);
  font-size: 0.85rem;
}

/* MODALES RESPONSIVOS Y ELEMENTOS DE ERROR */
.modal_productos { 
  background: rgba(0, 0, 0, 0.6); 
  backdrop-filter: blur(4px); 
  position: fixed; 
  top: 0; left: 0; right: 0; bottom: 0; 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  z-index: 2000; 
  padding: 15px;
}
.contenedor { 
  background: white; 
  padding: 20px; 
  border-radius: 12px; 
  width: 100%;
  max-width: 380px; 
  display: flex; 
  flex-direction: column; 
  gap: 10px; 
  position: relative; 
}

/* DISEÑO DE BOTONES DE SELECCIÓN DE FACTURA */
.opciones-factura {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 5px;
}
.opciones-factura button {
  padding: 12px;
  font-size: 0.95rem;
  font-weight: bold;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  text-align: left;
  transition: background 0.2s ease;
}
.btn-opcion-anonimo { background: #e2e3e5; color: #383d41; }
.btn-opcion-anonimo:hover { background: #d6d8db; }
.btn-opcion-cliente { background: #cce5ff; color: #004085; }
.btn-opcion-cliente:hover { background: #b8daff; }

/* BANNER DE NOTIFICACIONES DE ERROR */
.error-banner {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  padding: 10px;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
  text-align: center;
}

.contenedor h1 { font-size: 1.2rem; color: #333; margin-bottom: 5px; }
.btn-cerrar { position: absolute; top: 15px; right: 15px; background: none; border: none; font-size: 1.1rem; cursor: pointer; }
.form-group { display: flex; flex-direction: column; gap: 6px; }
.contenedor label { font-size: 0.85rem; font-weight: bold; color: #555; }
.contenedor input, .contenedor select { padding: 8px; border: 1px solid #ccc; border-radius: 6px; font-size: 0.95rem; width: 100%; }
.btn-guardar { padding: 10px; background: #28a745; color: white; border: none; border-radius: 6px; font-weight: bold; cursor: pointer; margin-top: 5px; }

.vacio { text-align: center; color: #777; margin-top: 30px; font-size: 0.9rem; }

/* MEDIA QUERIES PARA PANTALLAS */
@media (min-width: 480px) {
  .caja {
    flex-direction: row;
    justify-content: space-between;
    padding: 10px 20px;
  }
  .menu-botones {
    width: auto;
    justify-content: flex-end;
  }
  .menu-botones button {
    font-size: 0.85rem;
    padding: 8px 16px;
    max-width: none;
  }
}

@media (min-width: 768px) {
  .productos, .mesas-container, .section-facturacion {
    padding: 25px;
  }
  .grid-items {
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 20px;
  }
  .section-facturacion h2 {
    font-size: 1.6rem;
  }
}

@media (max-width: 340px) {
  .grid-items {
    grid-template-columns: 1fr;
  }
  .menu-botones button {
    font-size: 0.7rem;
    padding: 6px 2px;
  }
}
</style>