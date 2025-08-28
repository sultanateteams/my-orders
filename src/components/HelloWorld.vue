<template>
  <div class="p-4">
    <!-- Xato -->
    <h1 v-if="errorText" class="text-danger">
      {{ errorText }}
    </h1>

    <!-- Loader -->
    <div
      v-if="loading"
      class="d-flex justify-content-center align-items-center"
    >
      <b-spinner label="Yuklanmoqda..."></b-spinner>
      <span class="ms-2">Ma'lumotlar yuklanmoqda...</span>
    </div>

    <!-- Jadval -->
    <b-table
      v-else
      :items="orders"
      :fields="fields"
      bordered
      striped
      hover
      small
      responsive="sm"
    >
      <!-- Rownum doim center -->
      <template #cell(rownum)="data">
        <div class="text-center">
          {{ data.item.rownum ?? "-" }}
        </div>
      </template>

      <!-- Modified_at maxsus -->
      <template #cell(modified_at)="data">
        <div :class="data.item.modified_at ? 'text-start' : 'text-center'">
          {{
            data.item.modified_at
              ? new Date(data.item.modified_at).toLocaleString()
              : "-"
          }}
        </div>
      </template>

      <!-- Qolgan barcha cellar -->
      <template #cell()="data">
        <div
          :class="
            data.value !== null && data.value !== ''
              ? 'text-start'
              : 'text-center'
          "
        >
          {{ data.value !== null && data.value !== "" ? data.value : "-" }}
        </div>
      </template>
    </b-table>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import supabase from "../config/supabazaClient";

const errorText = ref("");
const loading = ref(false);
const orders = ref([]);

const fields = [
  { key: "rownum", label: "#" },
  { key: "name1", label: "Status" },
  { key: "barcode", label: "Shtrix kod" },
  { key: "cost_price", label: "Narxi" },
  { key: "volume", label: "Hajmi" },
  { key: "weight", label: "Og‘irligi" },
  { key: "modified_at", label: "O‘zgargan sana" },
];

const queryParams = new URLSearchParams(window.location.search);
const org_id = queryParams.get("org_id");
const owner_id = queryParams.get("owner_id");

const fetchOrders = async () => {
  loading.value = true;
  try {
    if (!org_id || !owner_id) {
      errorText.value =
        "Textnik xato yuz berdi biz bilan bog'lanib bu haqida xabar berishingizni so'raymiz";
      return;
    }

    const { data, error } = await supabase.rpc("get_cargo_orders", {
      p_org_id: org_id,
      p_owner_id: owner_id,
    });

    if (error) {
      errorText.value = "Xatolik: " + error.message;
    } else {
      orders.value = data || [];
    }
  } catch (err) {
    errorText.value = "Ma'lumot olishda xatolik";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(fetchOrders);
</script>
