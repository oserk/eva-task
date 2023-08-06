<template>
  <div>
    <table>
      <tr>
        <th>SKU</th>
        <th>Product Name</th>
        <th>{{ selectedDates[0] }} Sales / Units Avg. Selling Price</th>
        <th v-if="selectedDates.length === 2">
          {{ selectedDates[1] }} Sales / Units Avg. Selling Price
        </th>
        <th>SKU Refund Rate</th>
      </tr>

      <tr
        v-for="(tableRow, tableRowIndex) in tableData.item.skuList"
        :key="tableRowIndex"
      >
        <td>{{ tableRow.sku }}</td>
        <td>{{ tableRow.productName }}</td>
        <td>
          {{ `${tableData.Currency}${tableRow.amount} / ${tableRow.qty}` }}
          <br />
          {{ tableData.Currency }}
          {{
            tableRow.qty > 0
              ? (Number(tableRow.amount) / Number(tableRow.qty)).toFixed(2)
              : '-'
          }}
        </td>
        <td v-if="selectedDates.length === 2">
          {{ `${tableData.Currency}${tableRow.amount2} / ${tableRow.qty2}` }}
          <br />
          {{ tableData.Currency }}
          {{
            tableRow.qty2 > 0
              ? (Number(tableRow.amount2) / Number(tableRow.qty2)).toFixed(2)
              : '-'
          }}
        </td>
        <td>
          {{ skuRefundRates[tableRowIndex] }}
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  props: {
    tableData: {
      type: Object,
    },
    selectedDates: {
      type: Array,
    },
    skuRefundRates: {
      type: Array,
    },
  },
};
</script>

<style scoped>
table {
  font-family: Arial, Helvetica, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

table td,
table th {
  border: 1px solid #ddd;
  padding: 8px;
}

table tr:nth-child(even) {
  background-color: #f2f2f2;
}

table tr:hover {
  background-color: #ddd;
}

table th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #3498db;
  color: white;
}
</style>
