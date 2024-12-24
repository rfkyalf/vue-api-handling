<template>
  <main class="min-h-screen bg-neutral-200 py-4">
    <section class="w-[1280px] mx-auto">
      <div class="flex gap-x-4 items-center">
        <input v-model="searchTerm" type="search" placeholder="search..." />
        <select v-model="category">
          <option value="" selected>Category:</option>
          <option value="men's clothing">Men's Clothing</option>
          <option value="women's clothing">Women's Clothing</option>
          <option value="jewelery">Jewelery</option>
          <option value="electronics">Electronics</option>
        </select>
        <select v-model="sortBy">
          <option value="" selected>Sort By:</option>
          <option value="title">Title</option>
          <option value="price">Price</option>
        </select>
        <select v-model="orderBy">
          <option value="" selected>Order by:</option>
          <option value="asc">Ascending</option>
          <option value="desc">Descending</option>
        </select>
        <select v-model="limit">
          <option value="" selected>Limit:</option>
          <option v-for="index in 20" :value="index">{{ index }}</option>
        </select>
      </div>
      <div class="flex gap-x-4 items-center mt-4">
        <button
          :disabled="page === 1"
          @click="page > 1 && page--"
          class="disabled:cursor-not-allowed"
        >
          Prev
        </button>
        <span>Page {{ page }} of {{ totalPage }}</span>
        <button
          :disabled="page === totalPage"
          @click="page < totalPage && page++"
          class="disabled:cursor-not-allowed"
        >
          Next
        </button>
      </div>
      <ul class="mt-1">
        <li v-for="product in limitedProducts" :key="product.id">
          {{ product.id }}. {{ product.title }} - ${{ product.price }} -
          {{ product.category }}
        </li>
      </ul>
      <p v-if="!loading && searchedProducts.length === 0">
        Nothing to see here
      </p>
      <p v-if="loading">Loading ...</p>
      <p v-if="error">{{ error }}</p>
    </section>
  </main>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue';

interface Product {
  id: number;
  title: string;
  price: number;
  description: string;
  category: string;
  image: string;
  rating: {
    rate: number;
    count: number;
  };
}

const BASE_URL = 'https://fakestoreapi.com/products';

const products = ref<Product[]>([]);
const loading = ref<boolean>(false);
const error = ref<string | null>(null);
const orderBy = ref<string>('');
const category = ref<string>('');
const sortBy = ref<string>('');
const searchTerm = ref<string>('');
const limit = ref<string>('');
const page = ref<number>(1);

const totalPage = computed(() => {
  const itemsPerPage = limit.value ? Number(limit.value) : 10;
  return Math.ceil(searchedProducts.value.length / itemsPerPage);
});

const fetchProducts = async () => {
  loading.value = true;
  try {
    const response = await fetch(BASE_URL);
    if (!response.ok) {
      throw new Error(`HTTP error! status ${response.status}`);
    }
    products.value = await response.json();
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unknown error occurred';
  } finally {
    loading.value = false;
  }
};

const orderByProducts = computed(() => {
  const sortedProducts = [...products.value];

  if (sortBy.value === 'title') {
    return orderBy.value === 'desc'
      ? sortedProducts.sort((a, b) => b.title.localeCompare(a.title))
      : sortedProducts.sort((a, b) => a.title.localeCompare(b.title));
  } else if (sortBy.value === 'price') {
    return orderBy.value === 'desc'
      ? sortedProducts.sort((a, b) => b.price - a.price)
      : sortedProducts.sort((a, b) => a.price - b.price);
  } else if (orderBy.value === 'desc') {
    return sortedProducts.sort((a, b) => b.id - a.id);
  }

  return sortedProducts;
});

const filteredProducts = computed(() => {
  if (category.value) {
    return orderByProducts.value.filter(
      (product) => product.category === category.value
    );
  }

  return orderByProducts.value;
});

const searchedProducts = computed(() => {
  if (searchTerm.value) {
    return filteredProducts.value.filter((product) =>
      product.title
        .toLocaleLowerCase()
        .includes(searchTerm.value.toLocaleLowerCase())
    );
  }

  return filteredProducts.value;
});

const limitedProducts = computed(() => {
  const itemsPerPage = limit.value ? Number(limit.value) : 10;
  return searchedProducts.value.slice(
    (page.value - 1) * itemsPerPage,
    page.value * itemsPerPage
  );
});

onMounted(() => fetchProducts());
</script>
