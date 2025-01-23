<!-- src/lib/components/ClientForm.svelte -->
<script lang="ts">
  import { z } from 'zod';
  import { supabase } from '$lib/supabase';

  // Esquema de validación con Zod (igual que antes)
  const clientSchema = z.object({
    name: z.string().min(2, { message: "El nombre debe tener al menos 2 caracteres" }),
    email: z.string().email({ message: "Correo electrónico inválido" }),
    address: z.string().optional(),
    tax_id: z.string().optional()
  });

  // Estado del formulario
  let formData = {
    name: '',
    email: '',
    address: '',
    tax_id: ''
  };

  // Estados de manejo
  let errors = {
    name: '',
    email: '',
    address: '',
    tax_id: ''
  };
  let isSubmitting = false;
  let submitMessage = '';

  async function handleSubmit() {
    // Limpiar estados previos
    errors = {
      name: '',
      email: '',
      address: '',
      tax_id: ''
    };
    submitMessage = '';

    try {
      // Validar datos
      const validatedData = clientSchema.parse(formData);
      
      isSubmitting = true;

      // Insertar en Supabase
      const { data, error } = await supabase
        .from('clients')
        .insert(validatedData)
        .select();

      if (error) throw error;

      // Limpiar formulario
      formData = {
        name: '',
        email: '',
        address: '',
        tax_id: ''
      };

      submitMessage = 'Cliente agregado exitosamente';
    } catch (err) {
      if (err instanceof z.ZodError) {
        err.errors.forEach(error => {
          if (error.path[0]) {
            errors[error.path[0]] = error.message;
          }
        });
      } else {
        submitMessage = `Error: ${err.message}`;
      }
    } finally {
      isSubmitting = false;
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit} class="bg-white shadow-md rounded-lg p-6">
  <h2 class="text-2xl font-bold mb-6 text-center">Nuevo Cliente</h2>
  
  <!-- Campos del formulario (igual que en el ejemplo anterior) -->
  <div class="mb-4">
    <label for="name" class="block mb-2">Nombre</label>
    <input 
      type="text" 
      id="name"
      bind:value={formData.name}
      class="w-full px-3 py-2 border rounded-md {errors.name ? 'border-red-500' : 'border-gray-300'}"
    />
    {#if errors.name}
      <p class="text-red-500 text-sm mt-1">{errors.name}</p>
    {/if}
  </div>

  <div class="mb-4">
    <label for="email" class="block mb-2">Correo Electrónico</label>
    <input 
      type="email" 
      id="email"
      bind:value={formData.email}
      class="w-full px-3 py-2 border rounded-md {errors.email ? 'border-red-500' : 'border-gray-300'}"
    />
    {#if errors.email}
      <p class="text-red-500 text-sm mt-1">{errors.email}</p>
    {/if}
  </div>

  <!-- Resto de los campos igual que antes -->

  <button 
    type="submit" 
    disabled={isSubmitting}
    class="w-full bg-blue-500 text-white py-2 rounded-md hover:bg-blue-600 transition duration-300 {isSubmitting ? 'opacity-50 cursor-not-allowed' : ''}"
  >
    {isSubmitting ? 'Agregando...' : 'Agregar Cliente'}
  </button>

  {#if submitMessage}
    <p class="{submitMessage.includes('Error') ? 'text-red-500' : 'text-green-500'} text-center mt-4">
      {submitMessage}
    </p>
  {/if}
</form>