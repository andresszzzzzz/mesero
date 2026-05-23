<template>
  <!-- LOADING -->
  <div v-if="loading" class="overlay-pantalla-completa">
    <div class="contenedor-spinner">
      <div class="spinner-imagen"></div>
      <p class="texto-procesando">Procesando pedido...</p>
    </div>
  </div>

  <div class="app-container">

    <!-- HEADER -->
    <header class="header-principal">
      <h1>🍽 RESTAURANTE EL CHEF</h1>
      <p>San Gil, Santander</p>
    </header>

    <!-- BOTÓN ADMIN -->
    <button class="btn-toggle-admin" v-on:click="mostrarAdmin = !mostrarAdmin">
      ➕ Agregar Producto
    </button>

    <!-- PANEL ADMIN -->
    <section v-if="mostrarAdmin" class="admin-panel">
      <h3>Panel Administrativo</h3>
      <div class="form-nuevo-producto">
        <div class="campo">
          <label>Nombre</label>
          <input v-model="nuevoProd.nombre" type="text" placeholder="Pizza Personal">
        </div>
        <div class="campo">
          <label>Precio</label>
          <input v-model.number="nuevoProd.precio" type="number" placeholder="15000">
        </div>
        <div class="campo">
          <label>Categoría</label>
          <select v-model="nuevoProd.cat">
            <option value="comida">🍔 Comida</option>
            <option value="bebida">🥤 Bebida</option>
            <option value="almuerzo">🍲 Almuerzo</option>
          </select>
        </div>
        <div class="campo">
          <label>Stock</label>
          <input v-model.number="nuevoProd.stock" type="number">
        </div>
        <div class="campo">
          <label>URL Imagen</label>
          <input v-model="nuevoProd.img" type="text" placeholder="https://...">
        </div>
        <button class="btn-crear" v-on:click="crearProducto">
          Guardar Producto
        </button>
      </div>
    </section>

    <!-- FILTROS -->
    <nav class="filtros">
      <button v-on:click="filtrar('todos')">Todos</button>
      <button v-on:click="filtrar('comida')">🍔 Comida</button>
      <button v-on:click="filtrar('bebida')">🥤 Bebidas</button>
      <button v-on:click="filtrar('almuerzo')">🍲 Almuerzos</button>
    </nav>

    <!-- BOTÓN CARRITO MOBILE -->
    <button class="btn-carrito-mobile" v-on:click="mostrarCarrito = true">
      🛒 <span class="contador-carrito">{{ carrito.length }}</span>
    </button>

    <!-- OVERLAY -->
    <div v-if="mostrarCarrito" class="overlay-carrito" v-on:click="mostrarCarrito = false"></div>

    <!-- MAIN -->
    <div class="main-content">
      <!-- PRODUCTOS -->
      <section class="grid-productos">
        <div v-for="p in productosVisibles" :key="p.id" class="card">
          <div class="card-img-container">
            <img :src="p.img" alt="producto">
            <div class="badge-stock">{{ p.stock }} disp.</div>
          </div>
          <div class="card-info">
            <h3 class="producto-nombre">{{ p.nombre }}</h3>
            <p class="precio">${{ p.precio.toLocaleString() }}</p>
            <button class="btn-add" v-on:click="agregarAlCarrito(p)" :disabled="p.stock === 0">
              Añadir
            </button>
          </div>
        </div>
      </section>

      <!-- CARRITO (Ahora responde a la clase dinámica en móvil) -->
      <aside class="carrito-sidebar" :class="{ 'abierto-mobile': mostrarCarrito }">
        <div class="header-carrito-mobile">
          <h2>🛍 Pedido</h2>
          <button class="btn-cerrar-carrito" v-on:click="mostrarCarrito = false">✖</button>
        </div>
        <div v-if="carrito.length === 0" class="empty-state">
          No hay productos seleccionados.
        </div>
        <div v-if="carrito.length > 0">
          <div v-for="item in carritoAgrupado" :key="item.id" class="item-carrito">
            <div class="item-info">
              <span class="item-cantidad">{{ item.cantidad }}x</span>
              <span class="item-nombre">{{ item.nombre }}</span>
            </div>
            <div class="item-controles">
              <strong>${{ (item.precio * item.cantidad).toLocaleString() }}</strong>
              <div class="botones-accion">
                <button class="btn-control add" v-on:click="agregarMas(item.id)">+</button>
                <button class="btn-control remove" v-on:click="eliminarDelCarrito(item.id)">×</button>
              </div>
            </div>
          </div>
          <div class="total-seccion">
            <hr>
            <h3>Total: ${{ totalFactura.toLocaleString() }}</h3>
            <button class="btn-pay" v-on:click="procesarPedidoCompleto">💳 Pagar Ahora</button>
          </div>
        </div>
      </aside>
    </div>

    <!-- MODAL FACTURA -->
    <div v-if="mostrarFactura" class="modal-factura-overlay">
      <div class="modal-factura-contenedor">

        <button class="btn-cerrar-modal" v-on:click="nuevaCompra">✖</button>

        <div class="factura-real" id="facturaPDF">
          <!-- HEADER -->
          <div class="factura-top">
            <h2>🍽 RESTAURANTE EL CHEF</h2>
            <p>San Gil, Santander</p>
            <p>Fecha: 22/5/2026</p>
          </div>

          <!-- TOTAL -->
          <div class="factura-total-box">
            <span>TOTAL</span>
            <strong>${{ totalFactura.toLocaleString() }}</strong>
          </div>

          <!-- TABLA -->
          <div class="factura-tabla">
            <div class="tabla-header">
              <span>Producto</span>
              <span>Cant.</span>
              <span>Precio</span>
            </div>
            <div v-for="item in carritoAgrupado" :key="item.id" class="tabla-item">
              <span>{{ item.nombre }}</span>
              <span>{{ item.cantidad }}</span>
              <span>${{ (item.precio * item.cantidad).toLocaleString() }}</span>
            </div>
          </div>
        </div>

        <!-- ACCIONES -->
        <div class="modal-acciones">
          <button class="btn-descargar-factura" v-on:click="descargarFacturaPDF">
            📄 Descargar Factura PDF
          </button>
          <button class="btn-nueva-compra" v-on:click="nuevaCompra">
            Terminar y Limpiar
          </button>
        </div>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue';
import { jsPDF } from 'jspdf';

const carrito = ref([]);
const categoriaActual = ref('todos');
const loading = ref(false);
const mostrarAdmin = ref(false);
const mostrarCarrito = ref(false);
const mostrarFactura = ref(false);
const productosVisibles = ref([]);
const carritoAgrupado = ref([]);
const totalFactura = ref(0);

const nuevoProd = ref({
  nombre: '',
  precio: null,
  cat: 'comida',
  stock: 10,
  img: ''
});

const todosLosProductos = ref([
  { id: 1, nombre: 'Hamburguesa Especial', precio: 18000, cat: 'comida', stock: 10, stockMax: 10, img: 'https://s3yuumiproduction.s3.us-east-2.amazonaws.com/9a8c18d2-a458-477d-adab-a5f00495d7e2_eea23bb78e.webp' },
  { id: 2, nombre: 'Perro Caliente Suizo', precio: 14000, cat: 'comida', stock: 8, stockMax: 8, img: 'https://adrianagibbs.com/wp-content/uploads/2017/09/LaCasaBistro.jpg' },
  { id: 3, nombre: 'Salchipapa Mediana', precio: 15000, cat: 'comida', stock: 12, stockMax: 12, img: 'https://upload.wikimedia.org/wikipedia/commons/thumb/8/82/Lima_salchipapas.jpg/1280px-Lima_salchipapas.jpg' },
  { id: 4, nombre: 'Pizza Hawaiana (Porción)', precio: 7000, cat: 'comida', stock: 20, stockMax: 20, img: 'https://irecetasfaciles.com/wp-content/uploads/2020/03/pizza-hawaiana.jpg' },
  { id: 5, nombre: 'Sandwich de Pollo', precio: 12000, cat: 'comida', stock: 5, stockMax: 5, img: 'https://www.recetasnestlecam.com/sites/default/files/srh_recipes/c5ad0cfe9d4beb9d633c9709113a1452.jpg' },
  { id: 6, nombre: 'Burrito de Carne', precio: 19000, cat: 'comida', stock: 7, stockMax: 7, img: 'https://cloudfront-us-east-1.images.arcpublishing.com/infobae/LL7KGOM7VZFPZP4I6NCWUYZPUQ.jpg' },
  { id: 7, nombre: 'Tacos al Pastor', precio: 22000, cat: 'comida', stock: 15, stockMax: 15, img: 'https://cdn.colombia.com/gastronomia/2011/09/29/tacos-al-pastor-3634.jpg' },
  { id: 8, nombre: 'Nuggets x10', precio: 13500, cat: 'comida', stock: 10, stockMax: 10, img: 'https://imag.bonviveur.com/nuggets-de-pollo-caseros.jpg' },
  { id: 9, nombre: 'Papas Francesas', precio: 9000, cat: 'comida', stock: 25, stockMax: 25, img: 'https://cocina-casera.com/wp-content/uploads/2023/01/patatas-fritas-crujientes-francesa-1.jpg' },
  { id: 10, nombre: 'Arepa con Todo', precio: 11000, cat: 'comida', stock: 14, stockMax: 14, img: 'https://productostipicolatino.com/wp-content/uploads/2024/05/arepa-con-todo2.jpg' },
  { id: 11, nombre: 'Empanadas x3', precio: 6000, cat: 'comida', stock: 30, stockMax: 30, img: 'https://salsasaderezos.com/cdn/shop/articles/Receta-Empanadas-Bogotanas-Aderezos-S_1200x1200.jpg?v=1728566820' },
  { id: 12, nombre: 'Chorizo con Arepa', precio: 8500, cat: 'comida', stock: 12, stockMax: 12, img: 'https://arepasmania.com/cdn/shop/products/ColombianChorizoArepa_872x.jpg?v=1626442750' },
  { id: 13, nombre: 'Nachos con Queso', precio: 16000, cat: 'comida', stock: 9, stockMax: 9, img: 'https://www.divinacocina.es/wp-content/uploads/nachos-con-salsa-queso.jpg' },
  { id: 14, nombre: 'Wrap de Pollo', precio: 15500, cat: 'comida', stock: 6, stockMax: 6, img: 'https://resuelveconbimbo-com-v2-assets.s3.amazonaws.com/s3fs-public/2024-01/Banner%20Desktop_Wrap%20de%20Pollo.webp?VersionId=Gi5bWLFSfFnCTn80o5DrZEklPfhs8l3q' },
  { id: 15, nombre: 'Coca-Cola 350ml', precio: 4500, cat: 'bebida', stock: 50, stockMax: 50, img: 'https://locatelcolombia.vtexassets.com/arquivos/ids/194239/7702535005354.png?v=636153524310130000' },
  { id: 16, nombre: 'Jugo de Mora', precio: 7500, cat: 'bebida', stock: 20, stockMax: 20, img: 'https://puntacamaron.com.co/107/jugo-de-mora.jpg' },
  { id: 17, nombre: 'Limonada Cerezada', precio: 9000, cat: 'bebida', stock: 15, stockMax: 15, img: 'https://www.frutisandy.com/wp-content/uploads/2021/03/Limonada-Cerezada.jpg' },
  { id: 18, nombre: 'Agua Mineral', precio: 3000, cat: 'bebida', stock: 40, stockMax: 40, img: 'https://pizzeriacarpaneto.com/wp-content/uploads/2020/07/bebmansingas1.jpg' },
  { id: 19, nombre: 'Cerveza Águila', precio: 5500, cat: 'bebida', stock: 24, stockMax: 24, img: 'https://drinkcentral.co/wp-content/uploads/2023/03/CERVEZA-AGUILA-LATA-330ml.webp' },
  { id: 20, nombre: 'Cerveza Corona', precio: 9500, cat: 'bebida', stock: 18, stockMax: 18, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSSUvV1MOQYN3QFuHnmTtdi2vJdqhrW8b0hYQ&s' },
  { id: 21, nombre: 'Té Helado', precio: 8000, cat: 'bebida', stock: 15, stockMax: 15, img: 'https://cdn7.kiwilimon.com/recetaimagen/3613/640x640/18285.jpg.jpg' },
  { id: 22, module: 'Malteada Vainilla', nombre: 'Malteada Vainilla', precio: 12500, cat: 'bebida', stock: 10, stockMax: 10, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTaKhwJqcZbxi9EYLu7t_d7Lsf6C8-dmT8VCg&s' },
  { id: 23, nombre: 'Café Americano', precio: 4000, cat: 'bebida', stock: 30, stockMax: 30, img: 'https://imag.bonviveur.com/cafe-americano-en-la-taza.jpg' },
  { id: 24, nombre: 'Capuccino', precio: 6500, cat: 'bebida', stock: 20, stockMax: 20, img: 'https://www.allrecipes.com/thmb/chsZz0jqIHWYz39ViZR-9k_BkkE=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/8624835-how-to-make-a-cappuccino-beauty-4x3-0301-13d55eaad60b42058f24369c292d4ccb.jpg' },
  { id: 25, nombre: 'Soda Saborizada', precio: 8500, cat: 'bebida', stock: 22, stockMax: 22, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQoCqrFaXU3lMe7Mgt-8hEoSt97Y0OLr6d33Q&s' },
  { id: 26, nombre: 'Batido Fresa', precio: 11000, cat: 'bebida', stock: 12, stockMax: 12, img: 'https://www.finedininglovers.es/sites/default/files/recipe_content_images/Batido%20de%20fresa.jpg' },
  { id: 27, nombre: 'Milo Frío', precio: 9000, cat: 'bebida', stock: 15, stockMax: 15, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQJJWJJjstV-hd-LLahUPK1_uDYwHYmzyULZw&s' },
  { id: 28, nombre: 'Bandeja Paisa', precio: 28000, cat: 'almuerzo', stock: 10, stockMax: 10, img: 'https://recetasdecocina.elmundo.es/wp-content/uploads/2025/05/bandeja-paisa-1024x683.jpg' },
  { id: 29, nombre: 'Corriente de Pollo', precio: 15000, cat: 'almuerzo', stock: 15, stockMax: 15, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT4Fab4viJZBbAzabu462hLD7LmtgX8m4QCyA&s' },
  { id: 30, nombre: 'Trucha al Ajillo', precio: 24000, cat: 'almuerzo', stock: 8, stockMax: 8, img: 'https://puntacamaron.com.co/91/trucha-al-ajillo.jpg' },
  { id: 31, nombre: 'Lomo de Cerdo', precio: 22000, cat: 'almuerzo', stock: 10, stockMax: 10, img: 'https://i.blogs.es/c6ca3c/chatgpt-image-17-dic-2025-10_24_27-a.m./450_1000.png' },
  { id: 32, nombre: 'Sobrebarriga en salsa criolla', precio: 26000, cat: 'almuerzo', stock: 6, stockMax: 6, img: 'https://www.recetasnestle.com.co/sites/default/files/srh_recipes/2cffdfca583775cd22a461944ad45eb6.jpg' },
  { id: 33, nombre: 'Sopa Mondongo', precio: 18500, cat: 'almuerzo', stock: 12, stockMax: 12, img: 'https://www.elespectador.com/resizer/v2/LCDPGSB5GNH5BHH6PCTQFY7M34.jpg?auth=33d410abca65c04e17687f0de7d8e1b641356c857e9a258cee2ea943f3f759de&width=920&height=613&smart=true&quality=60' },
  { id: 34, nombre: 'Arroz con Pollo', precio: 17000, cat: 'almuerzo', stock: 20, stockMax: 20, img: 'https://i0.wp.com/surtidoradeaves.com/wp-content/uploads/2017/08/arroz-con-pollo.png?fit=750%2C500&ssl=1' },
  { id: 35, nombre: 'Mojarra Frita', precio: 25000, cat: 'almuerzo', stock: 9, stockMax: 9, img: 'https://recetas.encolombia.com/wp-content/uploads/2013/03/mojarra-frita.webp' },
  { id: 36, nombre: 'Pasta Bolognesa', precio: 19500, cat: 'almuerzo', stock: 14, stockMax: 14, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSMCP1c7HyNLwExPpBnunHTXb1TpD0ikBtjpw&s' },
  { id: 37, nombre: 'Ensalada César', precio: 16500, cat: 'almuerzo', stock: 10, stockMax: 10, img: 'https://assets.tmecosys.com/image/upload/t_web_rdp_recipe_584x480/img/recipe/ras/Assets/b876d8ea-fc9b-4b04-9958-9c70fe1c74e0/Derivates/fb3399fa-df15-4d0d-9beb-83a79a37a16e.jpg' },
  { id: 38, nombre: 'Asado de Res', precio: 23000, cat: 'almuerzo', stock: 11, stockMax: 11, img: 'https://mojo.generalmills.com/api/public/content/oKDBX8mQfkSqraA1_GfVGw_gmi_hi_res_jpeg.jpeg?v=6cac91ad&t=16e3ce250f244648bef28c5949fb99ff' },
  { id: 39, nombre: 'Ajiaco Bogotano', precio: 21000, cat: 'almuerzo', stock: 15, stockMax: 15, img: 'https://mojo.generalmills.com/api/public/content/lWuocevZmkK6Iq3JG_OZUw_gmi_hi_res_jpeg.jpeg?v=c23ac952&t=16e3ce250f244648bef28c5949fb99ff' },
  { id: 40, nombre: 'Cazuela Mariscos', precio: 35000, cat: 'almuerzo', stock: 5, stockMax: 5, img: 'https://elrinconcolombiano.com/wp-content/uploads/2023/06/Cazuela-de-Mariscos-receta-colombiana.jpg' }
]);

const actualizarProductosVisibles = () => {
  if (categoriaActual.value === 'todos') {
    productosVisibles.value = todosLosProductos.value;
  } else {
    productosVisibles.value = todosLosProductos.value.filter(p => p.cat === categoriaActual.value);
  }
};

const actualizarCarritoYTotal = () => {
  totalFactura.value = carrito.value.reduce((acc, item) => acc + item.precio, 0);

  const grupos = {};
  carrito.value.forEach(item => {
    if (!grupos[item.id]) {
      grupos[item.id] = { ...item, cantidad: 0 };
    }
    grupos[item.id].cantidad++;
  });
  carritoAgrupado.value = Object.values(grupos);
};

actualizarProductosVisibles();

const filtrar = (cat) => {
  categoriaActual.value = cat;
  actualizarProductosVisibles();
};

const agregarAlCarrito = (p) => {
  if (p.stock > 0) {
    p.stock--;
    carrito.value.push({
      id: p.id,
      nombre: p.nombre,
      precio: p.precio
    });
    actualizarCarritoYTotal();
  }
};

const agregarMas = (id) => {
  const p = todosLosProductos.value.find(prod => prod.id === id);
  if (p) {
    agregarAlCarrito(p);
  }
};

const eliminarDelCarrito = (id) => {
  const index = carrito.value.map(item => item.id).lastIndexOf(id);
  if (index !== -1) {
    carrito.value.splice(index, 1);
    const p = todosLosProductos.value.find(prod => prod.id === id);
    if (p) {
      p.stock++;
    }
    actualizarCarritoYTotal();
  }
};

const crearProducto = () => {
  if (!nuevoProd.value.nombre || !nuevoProd.value.precio) {
    alert('Llena nombre y precio');
    return;
  }

  todosLosProductos.value.push({
    id: Date.now(),
    nombre: nuevoProd.value.nombre,
    precio: nuevoProd.value.precio,
    cat: nuevoProd.value.cat,
    stock: nuevoProd.value.stock,
    stockMax: nuevoProd.value.stock,
    img: nuevoProd.value.img || 'https://via.placeholder.com/300'
  });

  nuevoProd.value = { nombre: '', precio: null, cat: 'comida', stock: 10, img: '' };
  mostrarAdmin.value = false;
  actualizarProductosVisibles();
};

const procesarPedidoCompleto = () => {
  if (carrito.value.length === 0) {
    alert('No hay productos en el carrito');
    return;
  }
  loading.value = true;
  setTimeout(() => {
    loading.value = false;
    mostrarFactura.value = true;
    mostrarCarrito.value = false;
  }, 1500);
};

const nuevaCompra = () => {
  carrito.value = [];
  actualizarCarritoYTotal();
  mostrarFactura.value = false;
};

const descargarFacturaPDF = () => {
  const elemento = document.getElementById('facturaPDF');
  if (!elemento) return;

  const doc = new jsPDF({
    orientation: 'portrait',
    unit: 'mm',
    format: 'a4'
  });

  doc.html(elemento, {
    callback: function (docOutput) {
      docOutput.save('Factura_Chef.pdf');
    },
    x: 10,
    y: 10,
    width: 190,
    windowWidth: 380
  });
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #000;
}

.app-container {
  background: #000;
  color: #fff;
  min-height: 100vh;
  padding: 15px;
  font-family: 'Segoe UI', sans-serif;
}

.header-principal {
  text-align: center;
  margin-bottom: 25px;
}

.header-principal h1 {
  color: #ff0000;
  font-size: 2rem;
  margin-bottom: 5px;
}

.header-principal p {
  color: #aaa;
}

.btn-toggle-admin {
  width: 100%;
  background: linear-gradient(135deg, #ff0000, #b30000);
  color: white;
  border: none;
  padding: 15px;
  border-radius: 15px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  margin-bottom: 20px;
}

.admin-panel {
  background: #111;
  border: 1px solid #ff0000;
  border-radius: 18px;
  padding: 22px;
  margin-bottom: 25px;
}

.form-nuevo-producto {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 18px;
}

.campo {
  display: flex;
  flex-direction: column;
}

.campo input,
.campo select {
  background: #1c1c1c;
  border: 1px solid #333;
  color: white;
  padding: 14px;
  border-radius: 12px;
}

.btn-crear {
  background: linear-gradient(135deg, #ff0000, #b30000);
  border: none;
  color: white;
  padding: 14px;
  border-radius: 12px;
  font-weight: bold;
  cursor: pointer;
}

.filtros {
  display: flex;
  gap: 10px;
  margin-bottom: 25px;
}

.filtros button {
  background: #1a1a1a;
  color: white;
  border: 1px solid #ff0000;
  padding: 10px 18px;
  border-radius: 25px;
  cursor: pointer;
}

.main-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.grid-productos {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 18px;
}

.card {
  background: #0f0f0f;
  border: 1px solid #222;
  border-radius: 18px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  min-height: 380px;
}

.card-img-container {
  width: 100%;
  height: 180px;
  position: relative;
}

.card-img-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.badge-stock {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #00ff88;
  color: #000;
  padding: 6px 10px;
  border-radius: 12px;
  font-weight: bold;
  font-size: 0.8rem;
}

.card-info {
  padding: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  flex: 1;
}

.producto-nombre {
  font-size: 1.05rem;
  text-align: center;
  margin-bottom: 10px;
}

.precio {
  color: #ff0000;
  font-size: 1.4rem;
  font-weight: bold;
  text-align: center;
  margin-bottom: 15px;
}

.btn-add {
  width: 100%;
  background: linear-gradient(135deg, #ff0000, #c00000);
  border: none;
  padding: 13px;
  border-radius: 14px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.carrito-sidebar {
  background: #111;
  padding: 20px;
  border: 1px solid #ff0000;
  border-radius: 18px;
}

.item-carrito {
  margin-bottom: 15px;
  border-bottom: 1px solid #222;
  padding-bottom: 10px;
}

.item-info {
  margin-bottom: 5px;
}

.item-cantidad {
  color: #00ff88;
  font-weight: bold;
  margin-right: 5px;
}

.item-controles {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.botones-accion {
  display: flex;
  gap: 5px;
}

.btn-control {
  width: 28px;
  height: 28px;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
}

.btn-control.add {
  background: #00ff88;
}

.btn-control.remove {
  background: #333;
  color: #ff4d4d;
}

.total-seccion h3 {
  margin: 15px 0;
}

.btn-pay {
  width: 100%;
  background: linear-gradient(135deg, #00c853, #009624);
  border: none;
  padding: 14px;
  border-radius: 14px;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.modal-factura-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  padding: 20px;
}

.modal-factura-contenedor {
  background: #111;
  border: 2px solid #ff0000;
  border-radius: 20px;
  width: 100%;
  max-width: 380px;
  padding: 20px;
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.btn-cerrar-modal {
  position: absolute;
  top: 10px;
  right: 15px;
  background: transparent;
  border: none;
  color: #888;
  font-size: 1.2rem;
  cursor: pointer;
}

.btn-cerrar-modal:hover {
  color: #ff0000;
}

.factura-real {
  background: #fff;
  color: #000;
  border-radius: 10px;
  padding: 15px;
  font-family: monospace;
}

.factura-top {
  text-align: center;
  margin-bottom: 10px;
  border-bottom: 1px dashed #333;
  padding-bottom: 10px;
}

.factura-top h2 {
  font-size: 1.1rem;
}

.factura-top p {
  font-size: 0.8rem;
}

.factura-total-box {
  background: #000;
  color: #fff;
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-radius: 5px;
  font-weight: bold;
  margin-bottom: 10px;
}

.factura-tabla {
  font-size: 0.85rem;
}

.tabla-header,
.tabla-item {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
  text-align: right;
  margin-bottom: 5px;
}

.tabla-header {
  font-weight: bold;
  border-bottom: 1px solid #000;
}

.tabla-header span:first-child,
.tabla-item span:first-child {
  text-align: left;
}

.modal-acciones {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.btn-descargar-factura {
  background: #ff0000;
  color: #fff;
  border: none;
  padding: 12px;
  border-radius: 10px;
  font-weight: bold;
  cursor: pointer;
}

.btn-nueva-compra {
  background: #222;
  color: #ccc;
  border: 1px solid #444;
  padding: 10px;
  border-radius: 10px;
  cursor: pointer;
}

.overlay-pantalla-completa {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, .9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.spinner-imagen {
  width: 55px;
  height: 55px;
  border: 5px solid #222;
  border-top: 5px solid #ff0000;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* =======================================================
   NUEVOS AJUSTES RESPONSIVE (Modificados y añadidos)
   ======================================================= */

/* Estilos base del botón flotante y overlay del carrito (ocultos por defecto) */
.btn-carrito-mobile {
  display: none;
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: linear-gradient(135deg, #00c853, #009624);
  color: white;
  border: none;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  font-size: 1.5rem;
  z-index: 999;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
  align-items: center;
  justify-content: center;
}

.contador-carrito {
  position: absolute;
  top: -2px;
  right: -2px;
  background: #ff0000;
  color: white;
  font-size: 0.8rem;
  font-weight: bold;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #000;
}

.overlay-carrito {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.7);
  z-index: 998;
}

.header-carrito-mobile {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.btn-cerrar-carrito {
  background: transparent;
  border: none;
  color: #ff4d4d;
  font-size: 1.2rem;
  cursor: pointer;
  display: none; /* Solo se verá en móvil */
}

/* DE 800PX HACIA ABAJO: Activamos el Carrito Lateral Desplegable */
@media(max-width:800px) {
  .grid-productos {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }

  .card {
    min-height: 350px;
  }

  .card-img-container {
    height: 130px;
  }

  /* Mostramos el botón flotante del carrito */
  .btn-carrito-mobile {
    display: flex;
  }

  .btn-cerrar-carrito {
    display: block;
  }

  /* Transformamos el sidebar en un panel lateral oculto */
  .carrito-sidebar {
    position: fixed;
    top: 0;
    right: -100%; /* Totalmente oculto a la derecha */
    width: 85%;
    max-width: 340px;
    height: 100vh;
    z-index: 999;
    border-radius: 20px 0 0 20px;
    border-y: none;
    border-right: none;
    transition: right 0.3s ease-in-out;
    overflow-y: auto;
    box-shadow: -5px 0 25px rgba(0,0,0,0.8);
  }

  /* Clase que activa Vue para mostrarlo */
  .carrito-sidebar.abierto-mobile {
    right: 0;
  }
}

/* DE 400PX HACIA ABAJO: Categorías en 2 y 2 sin scroll */
@media(max-width:400px) {
  .filtros {
    display: grid;
    grid-template-columns: repeat(2, 1fr); /* Fuerza cuadrícula de 2x2 */
    gap: 10px;
  }

  .filtros button {
    width: 100%;
    text-align: center;
    padding: 10px 5px;
    font-size: 0.9rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}

@media(min-width:801px) {
  .main-content {
    flex-direction: row;
  }

  .grid-productos {
    flex: 3;
  }

  .carrito-sidebar {
    flex: 1;
    position: sticky;
    top: 20px;
  }
}
</style>
