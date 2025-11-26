<template>
  <div
    class="max-w-md mx-auto mt-8 p-6 border border-gray-200 rounded-lg shadow-sm bg-white space-y-4"
  >
    <h2 class="text-xl font-semibold text-gray-800 text-center">
      Gerar Recibo de Pagamento (PDF)
    </h2>

    <!-- Nome do Vendedor -->
    <div class="flex flex-col space-y-1">
      <label class="text-sm font-medium text-gray-700">Nome do Vendedor</label>
      <input
        v-model="sellerName"
        type="text"
        class="px-3 py-2 rounded-md border border-gray-300 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
      />
    </div>

    <!-- Nome do Comprador -->
    <div class="flex flex-col space-y-1">
      <label class="text-sm font-medium text-gray-700">Nome do Comprador</label>
      <input
        v-model="buyerName"
        type="text"
        class="px-3 py-2 rounded-md border border-gray-300 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
      />
    </div>

    <!-- Valor -->
    <div class="flex flex-col space-y-1">
      <label class="text-sm font-medium text-gray-700"
        >Valor do Pagamento (R$)</label
      >
      <input
        v-model.number="amount"
        type="number"
        step="0.01"
        class="px-3 py-2 rounded-md border border-gray-300 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
      />
    </div>

    <!-- Itens -->
    <div class="flex flex-col space-y-2">
      <label class="text-sm font-medium text-gray-700">Itens Comprados</label>

      <div
        v-for="(item, index) in items"
        :key="index"
        class="flex items-center space-x-2"
      >
        <input
          v-model="items[index]"
          type="text"
          class="flex-1 px-3 py-2 rounded-md border border-gray-300 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
        />

        <button
          type="button"
          @click="removeItem(index)"
          v-if="items.length > 1"
          class="px-3 py-2 text-xs bg-red-100 text-red-700 font-medium rounded-md hover:bg-red-200"
        >
          Remover
        </button>
      </div>

      <button
        type="button"
        @click="addItem"
        class="self-start px-3 py-2 text-xs bg-gray-100 text-gray-800 rounded-md hover:bg-gray-200"
      >
        Adicionar Item
      </button>
    </div>

    <!-- Data -->
    <div class="flex flex-col space-y-1">
      <label class="text-sm font-medium text-gray-700">Data</label>
      <input
        v-model="date"
        type="date"
        class="px-3 py-2 rounded-md border border-gray-300 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
      />
    </div>

    <!-- Botão -->
    <button
      type="button"
      @click="generatePDF"
      class="w-full mt-2 px-4 py-2 bg-blue-600 text-white text-sm font-semibold rounded-md hover:bg-blue-700 focus:ring-2 focus:ring-blue-500 transition"
    >
      Gerar PDF do Recibo
    </button>
  </div>
</template>

<script setup>
import { ref } from "vue";
import jsPDF from "jspdf";

// Data de hoje formatada para input fecha YYYY-MM-DD
const today = new Date();
const yyyy = today.getFullYear();
const mm = String(today.getMonth() + 1).padStart(2, "0");
const dd = String(today.getDate()).padStart(2, "0");
const todayFormatted = `${yyyy}-${mm}-${dd}`;

// Campos reativos
const sellerName = ref("Padaria do Zé");
const buyerName = ref("Cliente Exemplo");
const amount = ref(50.0);
const items = ref(["pão", "queijo", "presunto", "mortadela"]);
const date = ref(todayFormatted);

// Funções
const addItem = () => {
  items.value.push("");
};

const removeItem = (index) => {
  items.value.splice(index, 1);
};

const generatePDF = () => {
  const doc = new jsPDF();

  const left = 20;
  let y = 20;

  doc.setFontSize(18);
  doc.text("RECIBO DE PAGAMENTO", left, y);

  y += 10;
  doc.line(left, y, 190, y);
  y += 10;

  doc.setFontSize(12);
  doc.text(`Vendedor: ${sellerName.value}`, left, y);
  y += 7;

  doc.text(`Comprador: ${buyerName.value}`, left, y);
  y += 7;

  const valorFormatado = amount.value
    ? amount.value.toLocaleString("pt-BR", {
        style: "currency",
        currency: "BRL",
      })
    : "R$ 0,00";

  doc.text(`Valor do pagamento: ${valorFormatado}`, left, y);
  y += 10;

  // converter data para DD/MM/YYYY
  let d = date.value;
  const [year, month, day] = d.split("-");
  const dataFormatadaPDF = `${day}/${month}/${year}`;

  doc.text(`Data: ${dataFormatadaPDF}`, left, y);
  y += 10;

  doc.line(left, y, 190, y);
  y += 10;

  doc.setFontSize(14);
  doc.text("Itens Comprados:", left, y);
  y += 8;

  doc.setFontSize(12);
  items.value.forEach((item) => {
    doc.text(`- ${item || "(vazio)"}`, left, y);
    y += 7;
  });

  y += 10;
  doc.setFontSize(10);
  doc.text("Assinatura do Vendedor: _______________________", left, y);
  y += 10;
  doc.text("Assinatura do Comprador: ______________________", left, y);

  doc.save(`recibo_${buyerName.value}.pdf`);
};
</script>
