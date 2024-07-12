<script>
  import NavBar from '$lib/components/nav/navBar.svelte';
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { get } from 'svelte/store';
  import Papa from 'papaparse';
  import { goto } from '$app/navigation';
  import { cart } from '$lib/stores/cartStore';
  import { writable } from 'svelte/store';

  let idProducts = [];
  let id = '';
  let loading = true;
  let selectedSize = '';
  let selectedColor = '';
  let quantity = 1;

  const toastMessage = writable('');

  const colors = [
    { name: 'ROJO', hex: 'red' },
    { name: 'BLANCO', hex: 'white' },
    { name: 'NEGRO', hex: 'black' },
    { name: 'VISON', hex: '#c8b9a6' },
    { name: 'AMARILLO', hex: 'yellow' },
    { name: 'VERDE', hex: 'green' },
    { name: 'ROSA', hex: 'pink' },
    { name: 'MORADO', hex: 'purple' },
    { name: 'GRIS', hex: 'gray' },
    { name: 'NARANJA', hex: 'orange' },
    { name: 'FUCSIA', hex: 'fuchsia' },
    { name: 'AZUL', hex: 'blue' },
    { name: 'DORADO', hex: 'gold' },
    { name: 'CELESTE', hex: 'lightblue' }
  ];

  onMount(async () => {
    id = get(page).params.id;

    try {
      const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=0&single=true&output=csv');
      const data = await response.text();
      const products = Papa.parse(data, { header: true }).data;

      idProducts = products.filter(product => product.ID === id);
    } catch (error) {
      console.error('Error fetching products:', error);
    } finally {
      loading = false;
    }
  });

  function handleProductClick(productId) {
    goto(`/tienda/${id}/${productId}`);
  }

  function addToCart(product) {
    if ((!selectedSize && product.TALLE && product.TALLE !== 'NO') || (!selectedColor && product.COLOR && product.COLOR !== 'NO')) {
      toastMessage.set('Por favor selecciona un tamaño y un color.');
      return;
    }

    cart.update(items => {
      if (!Array.isArray(items)) {
        items = [];
      }
      const itemIndex = items.findIndex(item => item.ID === product.ID && item.size === selectedSize && item.color === selectedColor);
      if (itemIndex === -1) {
        return [...items, { ...product, size: selectedSize, color: selectedColor, quantity }];
      } else {
        const updatedItems = [...items];
        updatedItems[itemIndex].quantity += quantity;
        return updatedItems;
      }
    });

    toastMessage.set('Producto añadido al carrito.');
    setTimeout(() => {
      toastMessage.set('');
    }, 5000);
  }

  function isColorAvailable(product, color) {
    return product.COLOR.includes(color);
  }

  function increaseQuantity() {
    quantity += 1;
  }

  function decreaseQuantity() {
    if (quantity > 1) {
      quantity -= 1;
    }
  }
</script>

<style>
  .toast {
    position: fixed;
    top: 40px;  /* Margen superior de 30px */
    right: 80px;
    background-color: #333;
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    opacity: 0.9;
  }

  .color-selector {
    display: flex;
    justify-content: center;
    margin-top: 8px;
  }

  .color-circle {
    display: inline-block;
    width: 30px;
    height: 30px;
    border-radius: 50%;
    margin: 0 8px;
    cursor: pointer;
    border: 2px solid transparent;
  }

  .color-circle.selected {
    border-color: #333;
  }

  .quantity-container {
    display: flex;
    align-items: center;
    margin-top: 8px;
  }

  .quantity-control {
    background-color: #eee;
    border: none;
    padding: 6px 10px;
    cursor: pointer;
  }

  .quantity-input {
    width: 50px;
    text-align: center;
    margin: 0 10px;
    padding: 6px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
</style>

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
                    <p class="mb-4">{product.PRECIO}</p>
                    {#if product.TALLE && product.TALLE !== 'NO'}
                      <div>
                        <p class="mb-2">Tamaño:</p>
                        <select class="w-full p-2 border rounded" bind:value={selectedSize}>
                          {#each product.TALLE.split('/') as size}
                            <option value={size.trim()}>{size.trim()}</option>
                          {/each}
                        </select>
                      </div>
                    {/if}
                    {#if product.COLOR && product.COLOR !== 'NO'}
                      <div class="mt-4">
                        <p class="mb-2">Color:</p>
                        <div class="color-selector">
                          {#each colors as color}
                            {#if isColorAvailable(product, color.name)}
                              <span class="color-circle {selectedColor === color.name ? 'selected' : ''}" style="background-color: {color.hex};" on:click={() => selectedColor = color.name}></span>
                            {/if}
                          {/each}
                        </div>
                      </div>
                    {/if}
                    <div class="quantity-container mt-4">
                      <button class="quantity-control" on:click={decreaseQuantity}>-</button>
                      <input type="number" min="1" value={quantity} class="quantity-input" on:input="{e => quantity = +e.target.value}">
                      <button class="quantity-control" on:click={increaseQuantity}>+</button>
                    </div>
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

  {#if $toastMessage}
    <div class="toast">{$toastMessage}</div>
  {/if}
</main>
