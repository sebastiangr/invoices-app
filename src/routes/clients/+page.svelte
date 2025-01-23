<script lang="ts">
  import ClientList from '$lib/components/ClientList.svelte';
  import ClientForm from '$lib/components/ClientForm.svelte';

  // Estados compartidos
  $: clients = [];
  $: loading = true;
  $: error = null;


  // Estados compartidos
  // let clients = [];
  // let loading = true;
  // let error = null;

</script>



  <!-- <div>
    <h2 class="text-2xl font-bold mb-6 text-center">Crear Factura</h2>
    <InvoiceForm />
  </div>
  <div>
    <h2 class="text-2xl font-bold mb-6 text-center">Listado de Facturas</h2>
    <InvoiceList />
  </div> -->


  <div>
    <ClientForm />
  </div>
  
  <div>
    <ClientList 
      bind:clients 
      bind:loading 
      bind:error 
    />
  </div>
