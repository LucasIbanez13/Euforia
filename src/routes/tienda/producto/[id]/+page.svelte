<script>
  import NavBar from '$lib/components/nav/navBar.svelte';
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { get } from 'svelte/store';
  import Papa from 'papaparse';
  import { goto } from '$app/navigation';
  import { cart } from '$lib/stores/cartStore';

  let idProducts = [];
  let id = '';
  let loading = true; // Variable para controlar el estado de carga

  onMount(async () => {
    id = get(page).params.id;

    const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=0&single=true&output=csv');
    const data = await response.text();
    const products = Papa.parse(data, { header: true }).data;

    idProducts = products.filter(product => product.ID === id);
    loading = false; // Cambiar el estado de carga cuando se hayan cargado los productos
  });

  // Función para manejar el clic en un producto
  function handleProductClick(productId) {
    goto(`/tienda/${id}/${productId}`); // Redirige a la vista del producto con su ID
  }

  // Función para añadir el producto al carrito
  function addToCart(product) {
    cart.update(items => {
      const itemIndex = items.findIndex(item => item.ID === product.ID);
      if (itemIndex === -1) {
        return [...items, { ...product, quantity: 1 }];
      } else {
        const updatedItems = [...items];
        updatedItems[itemIndex].quantity += 1;
        return updatedItems;
      }
    });
  }
</script>

<main class="p-8 bg-gray-100 text-center">
  <NavBar />

  <header class="text-gray-600 body-font pt-2">
    <div class="container mx-auto flex flex-wrap p-5 flex-col md:flex-row items-center">
      <a role="button" on:click={() => history.back()} class="flex order-first lg:order-none lg:w-1/5 title-font font-medium items-center text-gray-900 lg:items-center lg:justify-center mb-4 md:mb-0">Volver</a>
    </div>
  </header>

  {#if loading}
    <p>Cargando productos...</p>
  {:else}
    {#if idProducts.length > 0}
      <div class="flex justify-center">
        {#each idProducts as product}
          <section class="text-gray-600 body-font w-full md:w-3/4 lg:w-2/3 xl:w-1/2 my-8">
            <div class="container mx-auto flex flex-col md:flex-row items-center">
              <div class="lg:max-w-lg lg:w-full md:w-1/2 w-5/6 mb-10 md:mb-0">
                <img class="object-cover object-center rounded" alt="hero" src="{product.IMAGEN}">
              </div>
              <div class="lg:flex-grow md:w-1/2 lg:pl-24 md:pl-16 flex flex-col md:items-start md:text-left items-center text-center">
                <h1 class="title-font sm:text-4xl text-3xl mb-4 font-medium text-gray-900">{product.PRODUCTO}</h1>
                <p class="mb-8 leading-relaxed">{product.DESCRIPCION}</p>
                <div class="flex w-full md:justify-start justify-center items-end">
                  <div class="relative mr-4 lg:w-full xl:w-1/2 w-2/4">
                    <p class="mb-8 leading-relaxed">{product.PRECIO}</p>
                  </div>
                  <button class="inline-flex text-white bg-indigo-500 border-0 py-2 px-6 focus:outline-none hover:bg-indigo-600 rounded text-lg" on:click={() => addToCart(product)}>Comprar</button>
                </div>
              </div>
            </div>
          </section>
        {/each}
      </div>
    {:else}
      <p class="text-gray-600">No se encontraron productos para este ID: {id}</p>
    {/if}
  {/if}
</main>
