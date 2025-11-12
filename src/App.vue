<template>
  <div id="app">
    <header class="header">
      <h1>Carrito de Compras</h1>
      <p>Total de artículos: {{ totalItems }}</p>
    </header>

    <main class="main-content">
      <div class="products-section">
        <h2>Productos Disponibles</h2>
        <div class="products-grid">
          <div v-for="product in products" :key="product.id" class="product-card">
            <img :src="product.image" :alt="product.name" class="product-image" />
            <h3>{{ product.name }}</h3>
            <p class="price">${{ product.price }}</p>
            <q-btn
              color="primary"
              label="Agregar al Carrito"
              @click="() => {
                const existingItem = carrito.find(item => item.id === product.id);
                if (existingItem) {
                  existingItem.quantity++;
                } else {
                  carrito.push({ ...product, quantity: 1 });
                }
              }"
            />
          </div>
        </div>
      </div>

      <div class="cart-section">
        <h2>Tu Carrito</h2>
        <div v-if="carrito.length === 0" class="empty-cart">
          <p>Tu carrito está vacío</p>
        </div>
        <div v-else class="cart-items">
          <div v-for="(item, index) in carrito" :key="index" class="cart-item">
            <img :src="item.image" :alt="item.name" class="cart-item-image" />
            <div class="cart-item-details">
              <h4>{{ item.name }}</h4>
              <p class="price">${{ item.price }}</p>
              <div class="quantity-controls">
                <q-btn
                  flat
                  round
                  icon="remove"
                  @click="() => { if (carrito[index].quantity > 1) carrito[index].quantity--; }"
                  :disable="item.quantity <= 1"
                />
                <span>{{ item.quantity }}</span>
                <q-btn
                  flat
                  round
                  icon="add"
                  @click="() => { carrito[index].quantity++; }"
                />
              </div>
            </div>
            <q-btn
              flat
              round
              icon="delete"
              color="negative"
              @click="() => { carrito.splice(index, 1); }"
            />
          </div>
          <div class="cart-total">
            <div class="cart-summary">
              <div class="summary-row">
                <span class="summary-label">Total de artículos:</span>
                <span class="summary-value">{{ totalItems }}</span>
              </div>
              <div class="summary-row">
                <span class="summary-label">Subtotal:</span>
                <span class="summary-value">${{ subtotal.toFixed(2) }}</span>
              </div>
              <div class="summary-row">
                <span class="summary-label">Impuesto (19%):</span>
                <span class="summary-value">${{ impuesto.toFixed(2) }}</span>
              </div>
              <div class="summary-row total-row">
                <span class="summary-label">Total Final:</span>
                <span class="summary-value">${{ totalPrice.toFixed(2) }}</span>
              </div>
            </div>
            <q-btn color="positive" label="Comprar" size="lg" class="buy-button" />
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

// --- Función de Ayuda para Inicializar el Carrito ---
const getInitialCart = () => {
  const savedCart = localStorage.getItem('carrito');
  if (savedCart) {
    try {
      return JSON.parse(savedCart);
    } catch (e) {
      console.error('Error loading cart:', e);
      return [];
    }
  }
  return [];
};

// Estado Reactivo (ref)
const carrito = ref(getInitialCart());

// Productos de ejemplo
const products = ref([
  { id: 1, name: 'Producto 1', price: 10, image: '/vite.svg' },
  { id: 2, name: 'Producto 2', price: 20, image: '/vite.svg' },
  { id: 3, name: 'Producto 3', price: 30, image: '/vite.svg' },
  { id: 4, name: 'Producto 4', price: 40, image: '/vite.svg' },
  { id: 5, name: 'Producto 5', price: 50, image: '/vite.svg' },
  { id: 6, name: 'Producto 6', price: 60, image: '/vite.svg' },
]);


// Calcula total de items en el carrito
const totalItems = computed(() => {
  return carrito.value.reduce((total, item) => total + item.quantity, 0);
});

// Calcula el subtotal del carrito (sin impuestos)
const subtotal = computed(() => {
  return carrito.value.reduce((total, item) => total + item.price * item.quantity, 0);
});

// Calcula el impuesto (19% del subtotal)
const impuesto = computed(() => {
  return subtotal.value * 0.19;
});

// Calcula el precio total del carrito (subtotal + impuesto)
const totalPrice = computed(() => {
  return subtotal.value + impuesto.value;
});

// Watch para guardar en localStorage cuando cambia el carrito
watch(carrito, (newCart) => {
  // Guardar en localStorage
  localStorage.setItem('carrito', JSON.stringify(newCart));

  // Mostrar en consola cuando el carrito cambia
  console.log('Carrito actualizado:', newCart);
}, { deep: true });

// Watch para mostrar alerta cuando el total supera $1000
watch(totalPrice, (newTotal) => {
  if (newTotal > 1000) {
    console.warn(`⚠️ ALERTA: El total del carrito ($${newTotal.toFixed(2)}) supera los $1000`);
    // Alerta visual para el usuario
    alert(`¡Atención! El total de tu carrito es $${newTotal.toFixed(2)}, que supera los $1000`);
  }
});
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  text-align: center;
  margin-bottom: 40px;
}

.header h1 {
  color: #42b883;
  margin-bottom: 10px;
}

.main-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 40px;
}

.products-section h2,
.cart-section h2 {
  color: #42b883;
  margin-bottom: 20px;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.product-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  text-align: center;
  transition: box-shadow 0.3s;
}

.product-card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.product-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  margin-bottom: 10px;
}

.price {
  font-weight: bold;
  color: #42b883;
  margin: 10px 0;
}

.cart-section {
  border-left: 1px solid #ddd;
  padding-left: 20px;
}

.empty-cart {
  text-align: center;
  color: #666;
  font-style: italic;
}

.cart-items {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.cart-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.cart-item-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
}

.cart-item-details {
  flex: 1;
}

.cart-item-details h4 {
  margin: 0 0 5px 0;
}

.quantity-controls {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 10px;
}

.cart-total {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 2px solid #ddd;
}

.cart-summary {
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.summary-row:last-child {
  border-bottom: none;
}

.summary-label {
  font-size: 16px;
  color: #555;
}

.summary-value {
  font-size: 16px;
  font-weight: 600;
  color: #2c3e50;
}

.total-row {
  margin-top: 10px;
  padding-top: 15px;
  border-top: 2px solid #42b883 !important;
}

.total-row .summary-label {
  font-size: 18px;
  font-weight: bold;
  color: #2c3e50;
}

.total-row .summary-value {
  font-size: 20px;
  font-weight: bold;
  color: #42b883;
}

.buy-button {
  width: 100%;
  padding: 12px;
  font-size: 16px;
}

@media (max-width: 768px) {
  .main-content {
    grid-template-columns: 1fr;
  }

  .cart-section {
    border-left: none;
    border-top: 1px solid #ddd;
    padding-left: 0;
    padding-top: 20px;
    margin-top: 20px;
  }
}
</style>

