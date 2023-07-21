<template>
  <div class="card">
    <!-- <div>columns: {{ columns }}</div>
    <br /> -->
    <div>groupRowsBy: {{ groupRowsBy }}</div>
    <br />
    <div>sortField: {{ sortField }}</div>
    <br />
    <div>multiSortMeta: {{ multiSortMeta }}</div>
    <br />
    groupRowsBy:
    <Dropdown
      v-model="groupRowsBy[0]"
      :options="columnsForDD"
      class="w-full md:w-14rem mb-3 mr-3"
      @change="groupRowsByChange($event)"
    />
    <!-- <MultiSelect 
      v-model="groupRowsBy"
      :options="columnsForDD"
      class="w-full md:w-14rem mb-3 mr-3"
      @change="groupRowsByChange($event)"
    /> -->
    sortField:
    <Dropdown
      v-model="sortField"
      :options="columnsForDD"
      :disabled="checked"
      class="w-full md:w-14rem mb-3 mr-3"
      @change="sortFieldChange($event)"
    />
    multiple:
    <InputSwitch
      v-model="checked"
      class="mr-3"
      @change="switchChange(checked)"
    />
    <Button
      label="reset multiSortMeta"
      @click="multiSortMeta = []"
      class="mb-3 mr-3"
    />
    <DataTable
      v-model:expandedRowGroups="expandedRowGroups"
      :value="customers"
      tableStyle="min-width: 50rem"
      :rowGroupMode="groupRowsBy ? 'subheader' : null"
      :groupRowsBy="groupRowsBy"
      :sortMode="checked ? 'multiple' : 'single'"
      :sortField="sortField"
      :multiSortMeta="multiSortMeta"
      @update:multiSortMeta="multiSortMetaUpdate($event)"
      @update:sortField="sortFieldUpdate($event)"
      :sortOrder="1"
    >
      <template #groupheader="slotProps">
        <span class="vertical-align-middle ml-2 font-bold line-height-3">
          {{ slotProps.data[groupRowsBy] }}
        </span>
      </template>

      <Column
        v-for="col of columns"
        :key="col"
        :field="col"
        :header="col"
        sortable
      ></Column>
    </DataTable>
    <Toast />
  </div>
</template>

<script>
import { CustomerService } from "@/service/CustomerService";

export default {
  data() {
    return {
      customers: null,
      columns: null,
      columnsForDD: null,
      expandedRowGroups: null,
      groupRowsBy: [],
      sortField: null,
      multiSortMeta: [],
      checked: false,
    };
  },

  mounted() {
    CustomerService.getCustomersMedium().then((data) => {
      if (data.length > 0) {
        this.columns = Object.keys(data[0]);
        this.columnsForDD = [null].concat(this.columns);
        this.multiSortMeta = [
          {
            field: this.columns[0],
            order: 1,
          },
        ];
      }
      this.customers = data;
    });
  },

  methods: {
    groupRowsByChange(event) {
      if (this.checked) {
        this.multiSortMeta = [
          {
            field: this.columns.filter(
              (column) => column === this.groupRowsBy[0]
            )[0],
            order: 1,
          },
        ];
      } else {
        this.sortField = event.value;
      }
      console.log("Group changed: ", event.value);
    },

    sortFieldChange(event) {
      console.log("Single sort changed: ", event.value);
    },

    switchChange(event) {
      console.log("Sort mode switched. multiple: ", event);
      if (event) {
        this.sortField = null;
        if (this.groupRowsBy.length > 0 && this.groupRowsBy[0]) {
          this.multiSortMeta = [
            {
              field: this.columns.filter(
                (column) => column === this.groupRowsBy[0]
              )[0],
              order: 1,
            },
          ];
        };
      } else {
        this.multiSortMeta = [];
      }
    },

    sortFieldUpdate(event) {
      console.log("Single sort by: ", event);
      this.sortField = event;
    },

    multiSortMetaUpdate(event) {
      console.log("Multi sort by: ", event);
      if (event.length === 1 && this.groupRowsBy.length > 0 && this.groupRowsBy[0]) {
        this.multiSortMeta = [
          {
            field: this.columns.filter(
              (column) => column === this.groupRowsBy[0]
            )[0],
            order: 1,
          },
        ];

        this.multiSortMeta = this.multiSortMeta.concat(event);
      } else {
        this.multiSortMeta = event;
      }
    },
  },
};
</script>
