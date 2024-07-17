<script>
  import { page } from '$app/stores';
  import { get } from 'svelte/store';

  let category = '';
  let product = '';
  let breadcrumbUrl = '';

  $: {
    const $page = get(page);
    if ($page.params) {
      category = $page.params.category || '';
      product = $page.params.product || ''; // Asumiendo que el nombre del producto se obtiene de page.params.product
    }

    // Construir la URL de las migas de pan
    breadcrumbUrl = `/tienda/${category}`;
    if (product) {
      breadcrumbUrl += `/${product}`;
    }
  }
</script>

<nav class="text-gray-600 body-font pt-2">
  <div class="container mx-auto flex flex-wrap p-5 flex-col md:flex-row items-center">
    <a href="/tienda" class="flex order-first lg:order-none lg:w-1/5 title-font font-medium items-center text-gray-900 lg:items-center lg:justify-center mb-4 md:mb-0">Tienda</a>
    {#if category}
      <span>&gt;</span>
      <a href={breadcrumbUrl} class="flex order-first lg:order-none lg:w-1/5 title-font font-medium items-center text-gray-900 lg:items-center lg:justify-center mb-4 md:mb-0">{category}</a>
      {#if product}
        <span>&gt;</span>
        <span class="flex order-first lg:order-none lg:w-1/5 title-font font-medium items-center text-gray-900 lg:items-center lg:justify-center mb-4 md:mb-0">{product}</span>
      {/if}
    {/if}
  </div>
</nav>
