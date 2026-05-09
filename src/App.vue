<template>
  <div v-if="loading" class="overlay-pantalla-completa">
    <div class="contenedor-spinner">
      <div class="spinner-imagen"></div>
      <p class="texto-procesando">Procesando pedido...</p>
    </div>
  </div>

  <div class="app-container">
    <header class="header-principal">
      <h1>RESTAURANTE EL CHEF</h1>
      <p>San Gil, Santander</p>
    </header>

    <nav class="filtros">
      <button v-on:click="filtrar('todos')" :class="{ activo: categoriaActual === 'todos' }">Todos</button>
      <button v-on:click="filtrar('comida')" :class="{ activo: categoriaActual === 'comida' }">🍔 Comida</button>
      <button v-on:click="filtrar('bebida')" :class="{ activo: categoriaActual === 'bebida' }">🥤 Bebidas</button>
      <button v-on:click="filtrar('almuerzo')" :class="{ activo: categoriaActual === 'almuerzo' }">🍲 Almuerzos</button>
    </nav>

    <div class="main-content">
      <section class="grid-productos">
        <div v-for="p in productosVisibles" :key="p.id" class="card">
          <div class="card-img-container">
            <img v-if="p.img" :src="p.img" alt="producto">
            <div v-else class="img-placeholder">Imagen no disponible</div>
          </div>
          <div class="card-info">
            <h3 class="producto-nombre">{{ p.nombre }}</h3>
            <div class="card-footer-action">
              <p class="precio">${{ p.precio.toLocaleString() }}</p>
              <button class="btn-add" v-on:click="agregarAlCarrito(p)">Añadir</button>
            </div>
          </div>
        </div>
      </section>

      <aside class="carrito-sidebar">
        <h2>🛍️ Pedido Actual</h2>
        <div v-if="carrito.length === 0" class="empty-state">No hay productos.</div>
        <div v-else>
          <div v-for="(item, index) in carrito" :key="index" class="item-carrito">
            <span>{{ item.nombre }}</span>
            <strong>${{ item.precio.toLocaleString() }}</strong>
          </div>
          <div class="total-seccion">
            <hr>
            <h3>Total: ${{ totalFactura.toLocaleString() }}</h3>
            <button class="btn-pay" v-on:click="procesarPedidoCompleto">Pagar Ahora</button>
          </div>
        </div>
      </aside>
    </div>

    <div v-if="mostrarFactura" class="modal-factura">
      <div class="ticket">
        <div class="ticket-header">
          <h2>RESTAURANTE EL CHEF</h2>
          <p>San Gil, Santander</p>
          <p>Fecha: 24/04/2026</p>
        </div>

        <div class="ticket-line-punteada encabezado-tabla">
          <span>PRODUCTO</span>
          <span class="puntos-relleno"></span>
          <span>PRECIO</span>
        </div>

        <div class="ticket-body">
          <div v-for="item in carrito" :key="item.id" class="ticket-line-punteada">
            <span class="line-nombre">1x {{ item.nombre }}</span>
            <span class="puntos-relleno"></span>
            <span class="line-precio">${{ item.precio.toLocaleString() }}</span>
          </div>
        </div>

        <div class="ticket-footer">
          <div class="total-line-final">
            <span>TOTAL:</span>
            <span>${{ totalFactura.toLocaleString() }}</span>
          </div>
          <p class="gracias">¡Gracias por su visita!</p>
          <button class="btn-close" v-on:click="nuevaCompra">Cerrar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const todosLosProductos = ref([
  { id: 1, nombre: 'Hamburguesa Especial', precio: 18000, cat: 'comida', img: 'https://s3yuumiproduction.s3.us-east-2.amazonaws.com/9a8c18d2-a458-477d-adab-a5f00495d7e2_eea23bb78e.webp' },
  { id: 2, nombre: 'Perro Caliente Suizo', precio: 14000, cat: 'comida', img: 'https://adrianagibbs.com/wp-content/uploads/2017/09/LaCasaBistro.jpg' },
  { id: 3, nombre: 'Salchipapa Mediana', precio: 15000, cat: 'comida', img: 'https://upload.wikimedia.org/wikipedia/commons/thumb/8/82/Lima_salchipapas.jpg/1280px-Lima_salchipapas.jpg' },
  { id: 4, nombre: 'Pizza Hawaiana (Porción)', precio: 50000, cat: 'comida', img: 'https://irecetasfaciles.com/wp-content/uploads/2020/03/pizza-hawaiana.jpg' },
  { id: 5, nombre: 'Sandwich de Pollo', precio: 12000, cat: 'comida', img: 'https://www.recetasnestlecam.com/sites/default/files/srh_recipes/c5ad0cfe9d4beb9d633c9709113a1452.jpg' },
  { id: 6, nombre: 'Burrito de Carne', precio: 19000, cat: 'comida', img: 'https://cloudfront-us-east-1.images.arcpublishing.com/infobae/LL7KGOM7VZFPZP4I6NCWUYZPUQ.jpg' },
  { id: 7, nombre: 'Tacos al Pastor', precio: 22000, cat: 'comida', img: 'https://cdn.colombia.com/gastronomia/2011/09/29/tacos-al-pastor-3634.jpg' },
  { id: 8, nombre: 'Nuggets x10', precio: 13500, cat: 'comida', img: 'https://imag.bonviveur.com/nuggets-de-pollo-caseros.jpg' },
  { id: 9, nombre: 'Papas Francesas', precio: 9000, cat: 'comida', img: 'https://cocina-casera.com/wp-content/uploads/2023/01/patatas-fritas-crujientes-francesa-1.jpg' },
  { id: 10, nombre: 'Arepa con Todo', precio: 11000, cat: 'comida', img: 'https://productostipicolatino.com/wp-content/uploads/2024/05/arepa-con-todo2.jpg' },
  { id: 11, nombre: 'Empanadas x3', precio: 6000, cat: 'comida', img: 'https://salsasaderezos.com/cdn/shop/articles/Receta-Empanadas-Bogotanas-Aderezos-S_1200x1200.jpg?v=1728566820' },
  { id: 12, nombre: 'Chorizo con Arepa', precio: 8500, cat: 'comida', img: 'https://arepasmania.com/cdn/shop/products/ColombianChorizoArepa_872x.jpg?v=1626442750' },
  { id: 13, nombre: 'Nachos con Queso', precio: 16000, cat: 'comida', img: 'https://www.divinacocina.es/wp-content/uploads/nachos-con-salsa-queso.jpg' },
  { id: 14, nombre: 'Wrap de Pollo', precio: 15500, cat: 'comida', img: 'https://resuelveconbimbo-com-v2-assets.s3.amazonaws.com/s3fs-public/2024-01/Banner%20Desktop_Wrap%20de%20Pollo.webp?VersionId=Gi5bWLFSfFnCTn80o5DrZEklPfhs8l3q' },
  { id: 15, nombre: 'Coca-Cola 350ml', precio: 4500, cat: 'bebida', img: 'https://locatelcolombia.vtexassets.com/arquivos/ids/194239/7702535005354.png?v=636153524310130000' },
  { id: 16, nombre: 'Jugo de Mora', precio: 7500, cat: 'bebida', img: 'https://puntacamaron.com.co/107/jugo-de-mora.jpg' },
  { id: 17, nombre: 'Limonada Cerezada', precio: 9000, cat: 'bebida', img: 'https://www.frutisandy.com/wp-content/uploads/2021/03/Limonada-Cerezada.jpg' },
  { id: 18, nombre: 'Agua Mineral', precio: 3000, cat: 'bebida', img: 'https://pizzeriacarpaneto.com/wp-content/uploads/2020/07/bebmansingas1.jpg' },
  { id: 19, nombre: 'Cerveza Águila', precio: 5500, cat: 'bebida', img: 'https://drinkcentral.co/wp-content/uploads/2023/03/CERVEZA-AGUILA-LATA-330ml.webp' },
  { id: 20, nombre: 'Cerveza Corona', precio: 9500, cat: 'bebida', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSSUvV1MOQYN3QFuHnmTtdi2vJdqhrW8b0hYQ&s' },
  { id: 21, nombre: 'Té Helado', precio: 8000, cat: 'bebida', img: 'https://cdn7.kiwilimon.com/recetaimagen/3613/640x640/18285.jpg.jpg' },
  { id: 22, nombre: 'Malteada Vainilla', precio: 12500, cat: 'bebida', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTaKhwJqcZbxi9EYLu7t_d7Lsf6C8-dmT8VCg&s' },
  { id: 23, nombre: 'Café Americano', precio: 4000, cat: 'bebida', img: 'https://imag.bonviveur.com/cafe-americano-en-la-taza.jpg' },
  { id: 24, nombre: 'Capuccino', precio: 6500, cat: 'bebida', img: 'https://www.allrecipes.com/thmb/chsZz0jqIHWYz39ViZR-9k_BkkE=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/8624835-how-to-make-a-cappuccino-beauty-4x3-0301-13d55eaad60b42058f24369c292d4ccb.jpg' },
  { id: 25, nombre: 'Soda Saborizada', precio: 8500, cat: 'bebida', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQoCqrFaXU3lMe7Mgt-8hEoSt97Y0OLr6d33Q&s' },
  { id: 26, nombre: 'Batido Fresa', precio: 11000, cat: 'bebida', img: 'https://www.finedininglovers.es/sites/default/files/recipe_content_images/Batido%20de%20fresa.jpg' },
  { id: 27, nombre: 'Milo Frío', precio: 9000, cat: 'bebida', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQJJWJJjstV-hd-LLahUPK1_uDYwHYmzyULZw&s' },
  { id: 28, nombre: 'Bandeja Paisa', precio: 28000, cat: 'almuerzo', img: 'https://recetasdecocina.elmundo.es/wp-content/uploads/2025/05/bandeja-paisa-1024x683.jpg' },
  { id: 29, nombre: 'Corriente de Pollo', precio: 15000, cat: 'almuerzo', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT4Fab4viJZBbAzabu462hLD7LmtgX8m4QCyA&s' },
  { id: 30, nombre: 'Trucha al Ajillo', precio: 24000, cat: 'almuerzo', img: 'https://puntacamaron.com.co/91/trucha-al-ajillo.jpg' },
  { id: 31, nombre: 'Lomo de Cerdo', precio: 22000, cat: 'almuerzo', img: 'https://i.blogs.es/c6ca3c/chatgpt-image-17-dic-2025-10_24_27-a.m./450_1000.png' },
  { id: 32, nombre: 'Sobrebarriga en salsa criolla', precio: 26000, cat: 'almuerzo', img: 'https://www.recetasnestle.com.co/sites/default/files/srh_recipes/2cffdfca583775cd22a461944ad45eb6.jpg' },
  { id: 33, nombre: 'Sopa Mondongo', precio: 18500, cat: 'almuerzo', img: 'https://www.elespectador.com/resizer/v2/LCDPGSB5GNH5BHH6PCTQFY7M34.jpg?auth=33d410abca65c04e17687f0de7d8e1b641356c857e9a258cee2ea943f3f759de&width=920&height=613&smart=true&quality=60' },
  { id: 34, nombre: 'Arroz con Pollo', precio: 17000, cat: 'almuerzo', img: 'https://i0.wp.com/surtidoradeaves.com/wp-content/uploads/2017/08/arroz-con-pollo.png?fit=750%2C500&ssl=1' },
  { id: 35, nombre: 'Mojarra Frita', precio: 25000, cat: 'almuerzo', img: 'https://recetas.encolombia.com/wp-content/uploads/2013/03/mojarra-frita.webp' },
  { id: 36, nombre: 'Pasta Bolognesa', precio: 19500, cat: 'almuerzo', img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSMCP1c7HyNLwExPpBnunHTXb1TpD0ikBtjpw&s' },
  { id: 37, nombre: 'Ensalada César', precio: 16500, cat: 'almuerzo', img: 'https://assets.tmecosys.com/image/upload/t_web_rdp_recipe_584x480/img/recipe/ras/Assets/b876d8ea-fc9b-4b04-9958-9c70fe1c74e0/Derivates/fb3399fa-df15-4d0d-9beb-83a79a37a16e.jpg' },
  { id: 38, nombre: 'Asado de Res', precio: 23000, cat: 'almuerzo', img: 'https://mojo.generalmills.com/api/public/content/oKDBX8mQfkSqraA1_GfVGw_gmi_hi_res_jpeg.jpeg?v=6cac91ad&t=16e3ce250f244648bef28c5949fb99ff' },
  { id: 39, nombre: 'Ajiaco Bogotano', precio: 21000, cat: 'almuerzo', img: 'https://mojo.generalmills.com/api/public/content/lWuocevZmkK6Iq3JG_OZUw_gmi_hi_res_jpeg.jpeg?v=c23ac952&t=16e3ce250f244648bef28c5949fb99ff' },
  { id: 40, nombre: 'Cazuela Mariscos', precio: 35000, cat: 'almuerzo', img: 'https://elrinconcolombiano.com/wp-content/uploads/2023/06/Cazuela-de-Mariscos-receta-colombiana.jpg' }
]);

const productosVisibles = ref([...todosLosProductos.value]);
const carrito = ref([]);
const totalFactura = ref(0);
const categoriaActual = ref('todos');
const loading = ref(false);
const mostrarFactura = ref(false);

const filtrar = (cat) => {
  categoriaActual.value = cat;
  productosVisibles.value = cat === 'todos' ? [...todosLosProductos.value] : todosLosProductos.value.filter(p => p.cat === cat);
};

const agregarAlCarrito = (p) => {
  carrito.value.push(p);
  totalFactura.value = carrito.value.reduce((acc, item) => acc + item.precio, 0);
};

const procesarPedidoCompleto = () => {
  loading.value = true;
  setTimeout(() => {
    loading.value = false;
    mostrarFactura.value = true;
  }, 2000);
};

const nuevaCompra = () => {
  carrito.value = [];
  totalFactura.value = 0;
  mostrarFactura.value = false;
};
</script>

<style scoped>
/* --- ELIMINAR LÍNEA BLANCA SUPERIOR (image_4f14f2.png) --- */
:deep(body), :deep(html), :deep(#app) {
  margin: 0 !important;
  padding: 0 !important;
  background-color: #000 !important; /* Forza el fondo negro desde la raíz */
}

/* --- ESTILO GENERAL --- */
.app-container {
  background: #000;
  color: #fff;
  min-height: 100vh;
  padding: 20px;
  margin: 0; /* Asegura que no haya margen que cause la línea blanca */
  font-family: sans-serif;
}

.header-principal h1 {
  color: #ff0000;
  text-align: center;
  margin: 0 0 5px 0; /* Eliminado el margin-top para que pegue arriba */
}

.header-principal p {
  text-align: center;
  color: #aaa;
  margin-bottom: 20px;
}

.filtros {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 25px;
  position: sticky;
  top: 0;
  background: #000;
  padding: 15px;
  z-index: 100;
}

.filtros button {
  background: #1a1a1a;
  color: #fff;
  border: 1px solid #ff0000;
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 4px;
}

.filtros button.activo {
  background: #ff0000;
  box-shadow: 0 0 10px #ff0000;
}

.main-content {
  display: flex;
  gap: 20px;
}

/* --- GRID CORREGIDO PARA ALINEACIÓN (image_5c2c37.jpg, image_4fe2f1.jpg) --- */
.grid-productos {
  flex: 3;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 15px;
}

.card {
  background: #111;
  border: 1px solid #333;
  border-radius: 10px;
  overflow: hidden;
  display: flex;
  flex-direction: column; /* Alineación vertical */
  height: 100%; /* Estira las tarjetas uniformemente */
}

.card-img-container {
  height: 150px; /* ALTURA FIJA PARA TODAS LAS IMÁGENES */
  width: 100%;
  background: #222;
  display: flex;
  justify-content: center;
  align-items: center;
}

.card-img-container img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* Ajusta la imagen sin deformarla */
}

.img-placeholder {
  color: #555;
  font-size: 0.8rem;
}

.card-info {
  padding: 15px;
  flex-grow: 1; /* Empuja el botón hacia abajo */
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  text-align: center;
}

.producto-nombre {
  font-size: 1rem;
  margin-bottom: 10px;
  min-height: 2.5em; /* RESERVA ESPACIO PARA DOS LÍNEAS DE TEXTO */
  display: flex;
  align-items: center;
  justify-content: center;
}

.precio {
  color: #ff0000;
  font-weight: bold;
  font-size: 1.2rem;
  margin-bottom: 10px;
}

.btn-add {
  background: #ff0000;
  color: #fff;
  border: none;
  padding: 10px;
  cursor: pointer;
  font-weight: bold;
  width: 100%;
  border-radius: 3px;
}

/* --- SIDEBAR CARRITO --- */
.carrito-sidebar {
  flex: 1;
  background: #111;
  border: 2px solid #ff0000;
  padding: 15px;
  border-radius: 10px;
  height: fit-content;
  position: sticky;
  top: 100px;
}

.item-carrito {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 0.9rem;
}

.total-seccion {
  margin-top: 20px;
}

.btn-pay {
  background: #fff;
  color: #ff0000;
  width: 100%;
  padding: 15px;
  font-weight: bold;
  border: none;
  margin-top: 15px;
  cursor: pointer;
  border-radius: 5px;
}

/* --- SPINNER (image_5c34ad.png) --- */
.overlay-pantalla-completa {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.95);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  flex-direction: column;
}

.spinner-imagen {
  width: 60px;
  height: 60px;
  border: 7px solid #333;
  border-top: 7px solid #ff0000;
  border-left: 7px solid #ff0000;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.texto-procesando {
  color: #fff;
  margin-top: 15px;
  font-size: 1.2rem;
}

/* --- FACTURA CON LÍNEA DE PUNTOS (image_5bc75d.png) --- */
.modal-factura {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 5000;
}

.ticket {
  background: #fff;
  color: #000;
  padding: 25px;
  width: 330px;
  font-family: 'Courier New', Courier, monospace;
}

.ticket-header {
  text-align: center;
  margin-bottom: 15px;
}

.ticket-line-punteada {
  display: flex;
  align-items: baseline;
  margin-bottom: 6px;
}

.encabezado-tabla {
  font-weight: bold;
  margin-bottom: 10px;
}

.puntos-relleno {
  flex-grow: 1;
  border-bottom: 2px dotted #000;
  margin: 0 5px;
  height: 10px;
}

.line-nombre { flex-shrink: 0; }
.line-precio { flex-shrink: 0; font-weight: bold; }

.total-line-final {
  display: flex;
  justify-content: space-between;
  font-weight: bold;
  font-size: 1.3rem;
  margin-top: 15px;
  border-top: 1px solid #000;
  padding-top: 10px;
}

.gracias {
  text-align: center;
  margin: 15px 0;
  font-style: italic;
}

.btn-close {
  background: #ff0000;
  color: #fff;
  border: none;
  padding: 10px;
  cursor: pointer;
  width: 100%;
  margin-top: 10px;
  border-radius: 3px;
}
</style>
