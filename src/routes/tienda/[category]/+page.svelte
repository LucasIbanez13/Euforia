<script>
  import NavBar from '$lib/components/nav/navBar.svelte';
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { get } from 'svelte/store';
  import Papa from 'papaparse';

  let categoryProducts = [];
  let category = '';
  let loading = true; // Variable para controlar el estado de carga

  onMount(async () => {
    category = get(page).params.category;

    const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=0&single=true&output=csv');
    const data = await response.text();
    const products = Papa.parse(data, { header: true }).data;

    // Filtrar productos por categoría
    categoryProducts = products.filter(product => product.CATEGORY === category);
    loading = false; // Cambiar el estado de carga cuando se hayan cargado los productos
  });

  // Función para manejar el clic en un producto
  function handleProductClick(productId) {
    location.href = `/tienda/producto/${productId}`; // Redirige a la vista del producto con su ID usando location.href
  }
</script>

<main class="p-8 bg-gray-100 text-center">
  <NavBar />

  <header class="text-gray-600 body-font pt-2">
    <div class="container mx-auto flex flex-wrap p-5 flex-col md:flex-row items-center">
      <a role="button" on:click={() => history.back()} class="flex order-first lg:order-none lg:w-1/5 title-font font-medium items-center text-gray-900 lg:items-center lg:justify-center mb-4 md:mb-0">Volver</a>

    </div>
  </header>

  <section class="text-gray-600 body-font overflow-hidden">
    <div class="container px-5 py-12 mx-auto">
      <div class="flex flex-col text-center w-full mb-8">
        <h1 class="sm:text-4xl text-3xl font-medium title-font mb-2 text-gray-900">{category}</h1>
      </div>
    </div>
  </section>
  
  <section class="text-gray-600 body-font">
    <div class="container px-5 py-8 mx-auto">
      {#if loading}
        <p>Cargando productos...</p>
      {:else}
        {#if categoryProducts.length > 0}
          <div class="flex flex-wrap -m-4">
            {#each categoryProducts as product}
              <div class="lg:w-1/4 md:w-1/2 p-4 w-full" on:click={() => handleProductClick(product.ID)}>
                <a class="block relative h-48 rounded overflow-hidden">
                  <img alt="ecommerce" class="object-cover object-center w-full h-full block" src="{product.IMAGEN}">
                </a>
                <div class="mt-4">
                  <h3 class="text-gray-500 text-xs tracking-widest title-font mb-1">{product.CATEGORY}</h3>
                  <h2 class="text-gray-900 title-font text-lg font-medium">{product.PRODUCTO}</h2>
                  <p class="mt-1">{product.PRECIO}</p>
                </div>
              </div>
            {/each}
          </div>
        {:else}
          <p>No se encontraron productos para esta categoría.</p>
        {/if}
      {/if}
    </div>
  </section>
</main>
