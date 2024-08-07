<script>
  import { onDestroy, onMount } from 'svelte';
  import { cart } from '$lib/stores/cartStore';
  import { get } from 'svelte/store';

  export let showModal;
  export let toggleModal;

  const handleEsc = (event) => {
    if (event.key === 'Escape') {
      toggleModal();
    }
  };

  const handleClickOutside = (event) => {
    if (event.target.classList.contains('modal')) {
      toggleModal();
    }
  };

  const removeItem = (id) => {
    cart.update(items => items.filter(item => item.ID !== id));
  };

  const generateWhatsAppMessage = (cartItems) => {
    if (cartItems.length === 0) {
      return "Mi carrito está vacío.";
    }

    let message = "Hola, me gustaría comprar los siguientes productos:\n\n";
    cartItems.forEach(item => {
      message += `Producto: ${item.PRODUCTO}\nCantidad: ${item.quantity}\nPrecio: $${item.PRECIO}\n`;
      if (item.size) {
        message += `Talle: ${item.size}\n`;
      }
      if (item.color) {
        message += `Color: ${item.color}\n`;
      }
      message += `\n`;
    });
    const total = cartItems.reduce((acc, item) => acc + (item.quantity * item.PRECIO), 0);
    message += `Total: $${total}`;

    return message;
  };

  const sendWhatsAppMessage = () => {
    const cartItems = get(cart);
    const message = generateWhatsAppMessage(cartItems);
    const encodedMessage = encodeURIComponent(message);
    const phoneNumber = "3812018090";
    const whatsappURL = `https://wa.me/${phoneNumber}?text=${encodedMessage}`;
    window.open(whatsappURL, '_blank');
  };

  onMount(() => {
    if (typeof window !== 'undefined') {
      const storedCart = localStorage.getItem('cart');
      if (storedCart) {
        cart.set(JSON.parse(storedCart));
      }

      if ($showModal) {
        window.addEventListener('keydown', handleEsc);
      }

      onDestroy(() => {
        window.removeEventListener('keydown', handleEsc);
      });
    }
  });
</script>

<style>
  .modal {
    z-index: 50;
  }

  .cart-list {
    max-height: 300px; /* Altura máxima para el contenedor de la lista */
    overflow-y: auto; /* Barra de desplazamiento vertical si se supera la altura máxima */
  }

  .cart-item {
    min-height: 50px; /* Altura mínima de cada elemento del carrito */
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-bottom: 10px; /* Espacio entre los elementos del carrito */
  }
</style>

{#if $showModal}
  <div class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full modal" id="my-modal" on:click={handleClickOutside}>
    <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
      <div class="flex justify-between items-center mb-3">
        <button on:click={toggleModal} class="text-gray-500"><i class="fa-solid fa-x"></i></button>
      </div>
      <div class="mt-2 px-7 py-3">
        {#if $cart.length > 0}
          <div class="cart-list">
            {#each $cart as item}
              <div class="cart-item">
                <p class="text-sm text-gray-500">{item.PRODUCTO} - {item.quantity} x ${item.PRECIO}</p>
                {#if item.size}
                  <p class="text-sm text-gray-500">Talle: {item.size}</p>
                {/if}
                {#if item.color}
                  <p class="text-sm text-gray-500">Color: {item.color}</p>
                {/if}
                <button on:click={() => removeItem(item.ID)} class="text-red-500">Eliminar</button>
              </div>
            {/each}
          </div>
        {:else}
          <p class="text-sm text-gray-500">Tu carrito está vacío.</p> 
        {/if}
      </div>
      <div class="items-center px-4 py-3">
        <button 
          on:click={sendWhatsAppMessage}
          class="px-4 py-2 bg-indigo-500 text-white text-base font-medium rounded-md w-full shadow-sm hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-indigo-300"
          disabled={$cart.length === 0}>
          Comprar
        </button>
      </div>
    </div>
  </div>
{/if}
