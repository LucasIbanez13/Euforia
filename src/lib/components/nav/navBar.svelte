<script>
  import { writable } from 'svelte/store';
  import { cart } from '$lib/stores/cartStore';
  import Modal from '$lib/components/modal/modalCard.svelte';

  let showModal = writable(false);
  let cartItemCount = writable(0);

  function toggleModal() {
    showModal.update(n => !n);
  }

  $: {
    cart.subscribe(items => {
      cartItemCount.set(items.length);
    });
  }
</script>

<nav>
  <div class="container mx-auto flex items-center justify-between h-16">
    <a href="/tienda" class="flex items-center text-white text-xl font-bold">
      <img src="/logoEuf.jpeg" alt="Logo de la tienda" class="h-16 mr-4 ml-4">
    </a>
    <div class="relative">
      <button on:click={toggleModal} class="text-black px-4 py-2 rounded-md">
        <i class="fa-solid fa-bag-shopping fa-2x"></i>
        {#if $cartItemCount > 0}
          <span class="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-red-100 bg-red-600 rounded-full">{$cartItemCount}</span>
        {/if}
      </button>
    </div>
  </div>

  <!-- Usar el componente Modal -->
  <Modal {showModal} {toggleModal} />
</nav>

<style>
  nav {
    width: 100%;
  }
</style>
