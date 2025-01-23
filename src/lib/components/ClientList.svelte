<!-- src/lib/components/ClientList.svelte -->
<script lang="ts">
  import { supabase } from '$lib/supabase';
  import { onMount, onDestroy } from 'svelte';

  export let clients = [];
  export let loading = true;
  export let error = null;

  let clientsSubscription = null;

  const fetchClients = async () => {
    try {
      const { data, error: err } = await supabase
        .from('clients')
        .select('*')
        .order('created_at', { ascending: false });

      if (err) throw err;
      clients = data;
    } catch (err) {
      error = err.message;
    } finally {
      loading = false;
    }
  };

  const deleteClient = async (id) => {
    if (confirm('¿Está seguro de eliminar este cliente?')) {
      try {
        const { error: err } = await supabase
          .from('clients')
          .delete()
          .match({ id });

        if (err) throw err;
      } catch (err) {
        error = err.message;
      }
    }
  };

  onMount(async () => {
    await fetchClients();

    clientsSubscription = supabase
      .channel('clients')
      .on(
        'postgres_changes', 
        { 
          event: '*', 
          schema: 'public', 
          table: 'clients' 
        },
        (payload) => {
          switch(payload.eventType) {
            case 'INSERT':
              clients = [payload.new, ...clients];
              break;
            case 'UPDATE':
              clients = clients.map(client => 
                client.id === payload.new.id ? payload.new : client
              );
              break;
            case 'DELETE':
              clients = clients.filter(client => 
                client.id !== payload.old.id
              );
              break;
          }
        }
      )
      .subscribe();
  });

  onDestroy(() => {
    if (clientsSubscription) {
      supabase.removeChannel(clientsSubscription);
    }
  });
</script>

{#if loading}
  <p>Cargando...</p>
{:else if error}
  <p class="error">Error: {error}</p>
{:else}
  <div class="bg-white shadow-md rounded-lg p-6">
    <h2 class="text-2xl font-bold mb-4">Lista de Clientes</h2>
    {#if clients.length === 0}
      <p class="text-gray-500">No hay clientes registrados</p>
    {:else}
      <ul class="divide-y divide-gray-200">
        {#each clients as client (client.id)}
          <li class="py-4 flex justify-between items-center">
            <div>
              <p class="text-lg font-semibold">{client.name}</p>
              <p class="text-gray-500">{client.email}</p>
            </div>
            {#if client.address}
              <span class="text-sm text-gray-400">{client.address}</span>
            {/if}
            <button
              class="text-red-500 hover:text-red-700 transition duration-300"
              on:click={() => deleteClient(client.id)}
            >
              Eliminar
            </button>
          </li>
        {/each}
      </ul>
    {/if}
  </div>
{/if}

