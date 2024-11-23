<template>
  <div class="card">
    <h2 class="font-semibold text-xl mb-4">Lista de Contactos</h2>
    <div class="flex justify-between">
      <!-- Botón para abrir el modal -->
      <Button
        icon="pi pi-plus"
        label="Agregar Contacto"
        class="mb-3 p-button-primary"
        @click="showModal()"
      />
      <IconField>
        <InputIcon>
          <i class="pi pi-search" />
        </InputIcon>
        <InputText v-model="searchQuery" placeholder="Buscar Contacto" />
      </IconField>
    </div>

    <!-- Tabla de contactos -->
    <DataTable
      :value="filteredContacts"
      :paginator="true"
      :rows="10"
      dataKey="id"
      :rowHover="true"
      showGridlines
      responsiveLayout="scroll"
      :emptyMessage="'No se encontraron coincidencias'"
    >
      <Column field="name" header="Nombre Completo" style="min-width: 12rem">
        <template #body="{ data }">
          {{ data.name }}
        </template>
      </Column>

      <Column field="phone" header="Teléfono" style="min-width: 12rem">
        <template #body="{ data }">
          {{ data.phone }}
        </template>
      </Column>

      <Column header="Acciones" style="min-width: 8rem">
        <template #body="{ data }">
          <Button
            icon="pi pi-pencil"
            label="Editar"
            class="p-button-text p-button-primary"
            @click="editContact(data)"
          />
          <Button
            icon="pi pi-trash"
            label="Eliminar"
            class="p-button-text p-button-danger"
            @click="confirmDeleteContact(data.id)"
          />
        </template>
      </Column>
    </DataTable>

    <Dialog
      :header="
        isEditing
          ? 'Editando un contacto existente'
          : 'Agregando un contacto nuevo'
      "
      v-model:visible="isModalVisible"
      :closable="false"
      :style="{ width: '30vw' }"
    >
      <form @submit.prevent="handleSave">
        <div class="field">
          <label for="name" class="block">Nombre:</label>
          <InputText
            id="name"
            v-model="currentContact.name"
            class="w-full"
            required
          />
        </div>

        <div class="field mt-3">
          <label for="phone" class="block">Teléfono:</label>
          <InputText
            id="phone"
            v-model="currentContact.phone"
            class="w-full"
            required
            type="tel"
          />
        </div>

        <div class="mt-4 flex justify-end gap-2">
          <Button
            label="Cancelar"
            class="p-button-secondary"
            @click="closeModal()"
          />
          <Button label="Guardar" type="submit" class="p-button-primary" />
        </div>
      </form>
    </Dialog>
  </div>
</template>

<script>
import Swal from "sweetalert2";

export default {
  data() {
    return {
      contacts: [], // Lista de contactos
      isModalVisible: false, // Control del modal
      currentContact: { id: null, name: "", phone: "" }, // Contacto actual (para agregar/editar)
      isEditing: false, // Bandera para saber si estamos editando
      searchQuery: "", // Query de búsqueda
    };
  },
  mounted() {
    // Cargar contactos desde LocalStorage al montar el componente
    const storedContacts = JSON.parse(localStorage.getItem("contacts")) || [];
    this.contacts = storedContacts;
  },
  computed: {
    filteredContacts() {
      // Si no hay búsqueda, mostramos todos los contactos
      if (this.searchQuery.trim() === "") {
        return this.contacts;
      }

      // Filtrar los contactos que coincidan con la búsqueda
      const filtered = this.contacts.filter((contact) => {
        const query = this.searchQuery.toLowerCase();
        return (
          contact.name.toLowerCase().includes(query) ||
          contact.phone.includes(query)
        );
      });

      // Si no hay resultados, mostrar un mensaje personalizado
      if (filtered.length === 0) {
        Swal.fire(
          "No hay resultados",
          "No se encontraron coincidencias con tu búsqueda.",
          "info"
        );
      }

      return filtered;
    },
  },

  methods: {
    saveToLocalStorage() {
      localStorage.setItem("contacts", JSON.stringify(this.contacts));
    },
    showModal() {
      this.resetForm();
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
    },
    resetForm() {
      this.currentContact = { id: null, name: "", phone: "" };
      this.isEditing = false;
    },
    editContact(contact) {
      this.currentContact = { ...contact };
      this.isEditing = true;
      this.isModalVisible = true;
    },
    confirmDeleteContact(id) {
      Swal.fire({
        title: "¿Estás seguro?",
        text: "Este contacto será eliminado permanentemente.",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "Sí, eliminar",
        cancelButtonText: "Cancelar",
      }).then((result) => {
        if (result.isConfirmed) {
          this.deleteContact(id);
          Swal.fire("¡Eliminado!", "El contacto ha sido eliminado.", "success");
        }
      });
    },
    deleteContact(id) {
      this.contacts = this.contacts.filter((contact) => contact.id !== id);
      this.saveToLocalStorage(); // Guardar cambios en LocalStorage
    },
    handleSave() {
      if (this.isEditing) {
        // Actualizar contacto existente
        const index = this.contacts.findIndex(
          (c) => c.id === this.currentContact.id
        );
        if (index !== -1) {
          this.contacts[index] = { ...this.currentContact };
          Swal.fire(
            "¡Actualizado!",
            "El contacto ha sido actualizado.",
            "success"
          );
        }
      } else {
        // Agregar nuevo contacto
        const newContact = { ...this.currentContact, id: Date.now() };
        this.contacts.push(newContact);
        Swal.fire("¡Guardado!", "El contacto ha sido agregado.", "success");
      }
      this.saveToLocalStorage(); // Guardar cambios en LocalStorage
      this.closeModal();
    },
  },
};
</script>

<style scoped>
.field label {
  font-weight: bold;
}
</style>
