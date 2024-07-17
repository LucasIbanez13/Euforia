<script>
  import NavBar from '$lib/components/nav/navBar.svelte';
  import { onMount } from 'svelte';
  import Papa from 'papaparse';

  // Datos de productos
  async function getProducto() {
    const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=0&single=true&output=csv');
    const data = await response.text();
    const parsedData = Papa.parse(data, { header: true }).data;
    return parsedData;
  }

  let products = [];
  onMount(async () => {
    products = await getProducto();
  });

  // Datos de ofertas
  async function getOferta() {
    const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=1591664686&single=true&output=csv');
    const data = await response.text();
    const parsedData = Papa.parse(data, { header: true }).data;
    return parsedData;
  }

  let ofertas = [];
  onMount(async () => {
    ofertas = await getOferta();
  });

  // Datos de categorías
  async function getCategory() {
    const response = await fetch('https://docs.google.com/spreadsheets/d/e/2PACX-1vTlLjf4fDHfmV_IbT957SY6Lq-o4xInxsRelXCI2HxChD19INdmlWGFQWndW77oiwkZL6Cu4WGP4585/pub?gid=861181266&single=true&output=csv');
    const data = await response.text();
    const parsedData = Papa.parse(data, { header: true }).data;
    return parsedData;
  }

  let categories = [];
  onMount(async () => {
    categories = await getCategory();
  });
</script>

<main class="p-8 bg-gray-100 text-center">
  <NavBar />

  <section class="text-gray-600 body-font">
    <div class="container mx-auto">
      <div class="flex flex-wrap justify-center">
        {#each categories as category}
        <div class="w-full sm:w-1/2 lg:w-1/3 p-4">
          <a href={`/tienda/${category.CATEGORY}`}>
            <div class="relative mx-auto" style="max-width: 450px; height: 150px;">
              <div>
                <img alt="gallery" class="absolute inset-0 w-full h-full object-cover" src="{category.IMG}">
              </div>
              <div class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50 hover:bg-opacity-0 transition duration-300">
                <h2 class="text-white text-center text-sm uppercase">{category.CATEGORY}</h2>
              </div>
            </div>
          </a>
        </div>
        {/each}
      </div>
    </div>
  </section>

  <section class="text-gray-600 body-font">
    <div class="container px-5 py-24 mx-auto">
      <div class="flex flex-wrap -m-4">
        {#each ofertas as product}
          <a href={`/tienda/producto/${product.ID}`} class="lg:w-1/4 md:w-1/2 p-4 w-full">
            <div class="block relative h-48 rounded overflow-hidden">
              <img alt="{product.PRODUCTO}" class="object-cover object-center w-full h-full block" src="{product.IMAGEN}">
            </div>
            <div class="mt-4">
              <h2 class="text-gray-900 title-font text-lg font-medium">{product.PRODUCTO}</h2>
              <p class="mt-1">${product.PRECIO}</p>
            </div>
            <div>
              <button type="button" class="btn variant-filled bg-pink-500 text-white">Comprar</button>
            </div>
          </a>
        {/each}
      </div>
    </div>
  </section>
</main>
