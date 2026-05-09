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

    <!-- PANEL PARA AGREGAR PRODUCTOS NUEVOS -->
    <section class="admin-panel">
      <h3>➕ Panel Administrativo: Agregar a la Carta</h3>
      <div class="form-nuevo-producto">
        <div class="campo">
          <label>Nombre del plato</label>
          <input v-model="nuevoProd.nombre" type="text" placeholder="Ej: Pizza Personal">
        </div>
        <div class="campo">
          <label>Precio ($)</label>
          <input v-model.number="nuevoProd.precio" type="number" placeholder="Ej: 15000">
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
          <label>Stock Inicial</label>
          <input v-model.number="nuevoProd.stock" type="number">
        </div>
        <div class="campo larga">
          <label>URL de la Imagen</label>
          <input v-model="nuevoProd.img" type="text" placeholder="https://imagen.com/foto.jpg">
        </div>
        <button v-on:click="crearProducto" class="btn-crear">Guardar Producto</button>
      </div>
    </section>

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
            <div v-else class="img-placeholder">Sin Imagen</div>
            <div :class="['badge-stock', p.stock < 5 ? 'bajo' : 'alto']">
              {{ p.stock }} disp.
            </div>
          </div>
          <div class="card-info">
            <h3 class="producto-nombre">{{ p.nombre }}</h3>
            <div class="stock-container">
              <div class="stock-bar-bg">
                <div class="stock-bar-fill"
                  :style="{ width: (p.stock * 100 / p.stockMax) + '%', backgroundColor: p.stock < 5 ? '#ff4d4d' : '#00ff88' }">
                </div>
              </div>
            </div>
            <div class="card-footer-action">
              <p class="precio">${{ p.precio.toLocaleString() }}</p>
              <button class="btn-add" v-on:click="agregarAlCarrito(p)" :disabled="p.stock === 0">
                {{ p.stock > 0 ? 'Añadir' : 'Agotado' }}
              </button>
            </div>
          </div>
        </div>
      </section>

      <aside class="carrito-sidebar">
        <h2>🛍️ Pedido</h2>
        <div v-if="carrito.length === 0" class="empty-state">No hay productos seleccionados.</div>
        <div v-else>
          <div v-for="item in carritoAgrupado" :key="item.id" class="item-carrito">
            <div class="item-info">
              <span class="item-cantidad">{{ item.cantidad }}x</span>
              <span class="item-nombre">{{ item.nombre }}</span>
            </div>
            <div class="item-controles">
              <strong>${{ (item.precio * item.cantidad).toLocaleString() }}</strong>
              <div class="botones-accion">
                <button class="btn-control add" v-on:click="agregarMas(item.id)"
                  :disabled="obtenerStockProducto(item.id) === 0">+</button>
                <button class="btn-control remove" v-on:click="eliminarDelCarrito(item.id)">×</button>
              </div>
            </div>
          </div>
          <div class="total-seccion">
            <hr>
            <h3>Total: ${{ totalFactura.toLocaleString() }}</h3>
            <button class="btn-pay" v-on:click="procesarPedidoCompleto">Pagar Ahora</button>
          </div>
        </div>
      </aside>
    </div>

    <!-- MODAL FACTURA -->
    <div v-if="mostrarFactura" class="modal-factura">
      <div class="ticket">
        <div class="ticket-header">
          <h2>RESTAURANTE EL CHEF</h2>
          <p>San Gil, Santander</p>
          <p>Fecha: 08/05/2026</p>
        </div>
        <div class="ticket-body">
          <div v-for="item in carritoAgrupado" :key="item.id" class="ticket-line-punteada">
            <span>{{ item.cantidad }}x {{ item.nombre }}</span>
            <span>${{ (item.precio * item.cantidad).toLocaleString() }}</span>
          </div>
        </div>
        <div class="total-line-final">
          <span>TOTAL:</span>
          <span>${{ totalFactura.toLocaleString() }}</span>
        </div>
        <button class="btn-download" v-on:click="descargarFactura">📥 Descargar Factura</button>
        <button class="btn-close" v-on:click="nuevaCompra">Nueva Compra</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// --- LISTA COMPLETA DE PRODUCTOS ---
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
  { id: 22, nombre: 'Malteada Vainilla', precio: 12500, cat: 'bebida', stock: 10, stockMax: 10, img: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTaKhwJqcZbxi9EYLu7t_d7Lsf6C8-dmT8VCg&s' },
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

const carrito = ref([]);
const categoriaActual = ref('todos');
const loading = ref(false);
const mostrarFactura = ref(false);

const nuevoProd = ref({
  nombre: '',
  precio: null,
  cat: 'comida',
  stock: 10,
  img: ''
});

const productosVisibles = computed(() => {
  if (categoriaActual.value === 'todos') return todosLosProductos.value;
  return todosLosProductos.value.filter(p => p.cat === categoriaActual.value);
});

const totalFactura = computed(() => carrito.value.reduce((acc, item) => acc + item.precio, 0));

const carritoAgrupado = computed(() => {
  const grupos = {};
  carrito.value.forEach(item => {
    if (!grupos[item.id]) grupos[item.id] = { ...item, cantidad: 0 };
    grupos[item.id].cantidad++;
  });
  return Object.values(grupos);
});

const filtrar = (cat) => categoriaActual.value = cat;

const agregarAlCarrito = (p) => {
  if (p.stock > 0) {
    p.stock--;
    carrito.value.push({ id: p.id, nombre: p.nombre, precio: p.precio });
  }
};

const agregarMas = (id) => {
  const p = todosLosProductos.value.find(prod => prod.id === id);
  if (p) agregarAlCarrito(p);
};

const eliminarDelCarrito = (id) => {
  const index = carrito.value.map(item => item.id).lastIndexOf(id);
  if (index !== -1) {
    carrito.value.splice(index, 1);
    const p = todosLosProductos.value.find(prod => prod.id === id);
    if (p) p.stock++;
  }
};

const obtenerStockProducto = (id) => todosLosProductos.value.find(p => p.id === id)?.stock || 0;

const crearProducto = () => {
  if (!nuevoProd.value.nombre || !nuevoProd.value.precio) {
    alert("Por favor llena el nombre y el precio");
    return;
  }

  todosLosProductos.value.push({
    id: Date.now(),
    nombre: nuevoProd.value.nombre,
    precio: nuevoProd.value.precio,
    cat: nuevoProd.value.cat,
    stock: nuevoProd.value.stock,
    stockMax: nuevoProd.value.stock,
    img: nuevoProd.value.img || 'https://via.placeholder.com/150'
  });

  nuevoProd.value = { nombre: '', precio: null, cat: 'comida', stock: 10, img: '' };
};

const procesarPedidoCompleto = () => {
  loading.value = true;
  setTimeout(() => {
    loading.value = false;
    mostrarFactura.value = true;
  }, 1500);
};

const nuevaCompra = () => {
  carrito.value = [];
  mostrarFactura.value = false;
};

const descargarFactura = () => {
  let txt = `RESTAURANTE EL CHEF\nTOTAL: $${totalFactura.value}`;
  const blob = new Blob([txt], { type: 'text/plain' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = 'factura.txt';
  a.click();
};
</script>

<style scoped>
/* --- CONFIGURACIÓN BASE --- */
.app-container {
  background: #000;
  color: #fff;
  min-height: 100vh;
  padding: 10px;
  /* Reducido para ganar espacio */
  font-family: 'Segoe UI', sans-serif;
}

.header-principal {
  text-align: center;
  color: #ff0000;
  margin-bottom: 20px;
}

/* --- PANEL ADMIN (Colapsable/Compacto en móvil) --- */
.admin-panel {
  background: #111;
  padding: 15px;
  border-radius: 12px;
  border: 1px dashed #ff0000;
  margin-bottom: 20px;
}

.form-nuevo-producto {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.campo input,
.campo select {
  background: #222;
  border: 1px solid #444;
  color: #fff;
  padding: 10px;
  border-radius: 8px;
  font-size: 16px;
}

/* --- FILTROS (Scroll suave) --- */
.filtros {
  display: flex;
  overflow-x: auto;
  gap: 8px;
  margin-bottom: 20px;
  padding-bottom: 5px;
}

.filtros button {
  background: #1a1a1a;
  color: #fff;
  border: 1px solid #ff0000;
  padding: 8px 15px;
  border-radius: 20px;
  white-space: nowrap;
  font-size: 0.9rem;
}

/* --- GRID DE PRODUCTOS (CORREGIDO) --- */
.main-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 100%;
}

.grid-productos {
  display: grid;
  /* Esto asegura que en pantallas pequeñas use todo el ancho con 2 columnas */
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 12px;
  width: 100%;
}

/* --- TARJETAS (Ajustadas para no verse estiradas) --- */
.card {
  background: #111;
  border: 1px solid #222;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.card-img-container {
  height: 120px;
  position: relative;
}

.card-img-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.badge-stock {
  position: absolute;
  top: 5px;
  right: 5px;
  font-size: 0.6rem;
  padding: 2px 6px;
  border-radius: 4px;
  background: #00ff88;
  color: #000;
  font-weight: bold;
}

.card-info {
  padding: 10px;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.producto-nombre {
  font-size: 0.85rem;
  margin-bottom: 8px;
  text-align: center;
  font-weight: bold;
}

.stock-bar-bg {
  background: #222;
  height: 4px;
  border-radius: 10px;
  margin-bottom: 10px;
}

.precio {
  color: #ff0000;
  font-weight: bold;
  font-size: 1rem;
  text-align: center;
  margin-bottom: 8px;
}

.btn-add {
  background: #ff0000;
  border: none;
  padding: 8px;
  border-radius: 6px;
  color: #fff;
  font-weight: bold;
  font-size: 0.9rem;
}

/* --- CARRITO --- */
.carrito-sidebar {
  background: #111;
  padding: 15px;
  border: 1px solid #ff0000;
  border-radius: 12px;
  width: 100%;
  /* Ocupa todo el ancho en móvil */
  box-sizing: border-box;
}

.item-controles {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn-control {
  width: 28px;
  height: 28px;
  border-radius: 6px;
  border: none;
  font-weight: bold;
}

.btn-control.add {
  background: #00ff88;
}

.btn-control.remove {
  background: #333;
  color: #ff4d4d;
}

/* --- MEDIA QUERIES PARA PC --- */
@media (min-width: 768px) {
  .main-content {
    flex-direction: row;
    align-items: flex-start;
  }

  .grid-productos {
    flex: 3;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  }

  .carrito-sidebar {
    flex: 1;
    position: sticky;
    top: 20px;
  }

  .form-nuevo-producto {
    flex-direction: row;
    flex-wrap: wrap;
  }

  .campo {
    flex: 1;
    min-width: 150px;
  }
}

/* SPINNER */
.overlay-pantalla-completa {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.spinner-imagen {
  width: 40px;
  height: 40px;
  border: 4px solid #222;
  border-top: 4px solid #ff0000;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
