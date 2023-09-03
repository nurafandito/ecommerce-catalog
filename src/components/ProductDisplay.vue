<script>
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const loading = ref(true);
    const currentProduct = ref({});
    let currentProductIndex = 0;

    const fetchProduct = async () => {
      try {
        loading.value = true; // Mengaktifkan spinner loading.
        const response = await fetch('https://fakestoreapi.com/products/');
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const data = await response.json();

        // Kode ini memastikan bahwa `currentProductIndex` akan kembali ke awal ketika mencapai produk terakhir.
        currentProductIndex = currentProductIndex % data.length;
        currentProduct.value = data[currentProductIndex];

        // Periksa kategori produk dan ubah menjadi "unavailable-product" jika tidak sesuai dengan kriteria.
        if (currentProduct.value.category !== "men's clothing" && currentProduct.value.category !== "women's clothing") {
          currentProduct.value.category = 'unavailable-product';
        }
      } catch (error) {
        console.error('Error fetching product:', error);
      } finally {
        loading.value = false; // Menonaktifkan spinner loading.
      }
    };

    const getNextProduct = () => {
      currentProductIndex++;
      fetchProduct();
    };

    const getProductCategoryClass = (category) => {
      if (category === "men's clothing") {
        return 'page-men';
      } else if (category === "women's clothing") {
        return 'page-women';
      } else {
        return 'unavailable-product';
      }
    };

    onMounted(() => {
      fetchProduct();
    });

    return {
      loading,
      currentProduct,
      getNextProduct,
      getProductCategoryClass,
    };
  },
};
</script>

<template>
  <div v-if="loading" class="loader-container">
    <div class="loader"></div>
  </div>
  <div v-else :class="['container', getProductCategoryClass(currentProduct.category)]">
    <div class="product-section">
      <div class="product-image" v-if="currentProduct.category !== 'unavailable-product'">
        <img :src="currentProduct.image" :alt="currentProduct.title" />
      </div>
      <div class="product-content" v-if="currentProduct.category !== 'unavailable-product'">
        <h1>{{ currentProduct.title }}</h1>
        <div class="product-meta">
          <p>{{ currentProduct.category }}</p>
          <div>
            <span>{{ currentProduct.rating.rate }}/5</span>
            <span class="rating">
              <template v-for="i in Math.round(currentProduct.rating.rate)"> ● </template>
              <template v-for="i in 5 - Math.round(currentProduct.rating.rate)"> ○ </template>
            </span>
          </div>
        </div>
        <p class="product-description">{{ currentProduct.description }}</p>
        <div class="product-pricing">${{ currentProduct.price }}</div>
        <div class="button">
          <button class="button-primary inactive">Buy now</button>
          <button class="button-secondary active" @click="getNextProduct">Next product</button>
        </div>
      </div>
      <div v-else class="page-unavailable-product">
        <div class="unavailable-product-content">
          <p>This product is unavailable to show</p>
          <button class="active" @click="getNextProduct">Next product</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.loader-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.loader {
  border: 15px solid #f3f3f3;
  border-radius: 50%;
  border-top: 15px solid #3498db;
  width: 100px;
  height: 100px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
