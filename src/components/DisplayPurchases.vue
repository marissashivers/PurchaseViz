<template>
  <b-container fluid class="contain">
    <!-- User Interface controls -->
    <div class="ui">
      <!-- top row -->
      <b-row>
        <b-col>
          <!-- SORT BY CATEGORY -->
          <b-form-group
            label="Sort"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="sortBySelect"
            class="mb-0"
          >
            <b-input-group size="sm">
              <b-form-select
                v-model="sortBy"
                id="sortBySelect"
                :options="sortOptions"
                class="w-75"
              >
                <template v-slot:first>
                  <option value="">-- none --</option>
                </template>
              </b-form-select>
              <b-form-select
                v-model="sortDesc"
                size="sm"
                :disabled="!sortBy"
                class="w-25"
              >
                <option :value="false">Asc</option>
                <option :value="true">Desc</option>
              </b-form-select>
            </b-input-group>
          </b-form-group>
        </b-col>

        <b-col>
          <!-- SORTING DIRECTION -->
          <b-form-group
            label="Initial sort"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            label-for="initialSortSelect"
            class="mb-0"
          >
            <b-form-select
              v-model="sortDirection"
              id="initialSortSelect"
              size="sm"
              :options="['asc', 'desc', 'last']"
            ></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>

      <!-- bottom row -->
      <b-row>
        <b-col>
          <b-form-group
            label="Filter On"
            label-cols-sm="3"
            label-align-sm="right"
            label-size="sm"
            description="Leave all unchecked to filter on all data"
            class="mb-0"
          >
            <b-form-checkbox-group v-model="filterOn" class="mt-1">
              <b-form-checkbox value="createdAt">Date</b-form-checkbox>
              <b-form-checkbox value="purchaseLocation">Location</b-form-checkbox>
              <b-form-checkbox value="purchaseAmount">Amount</b-form-checkbox>
              <b-form-checkbox value="purchaseCategory">Category</b-form-checkbox>
            </b-form-checkbox-group>
          </b-form-group>
        </b-col>
      </b-row>

      <b-row>
        <b-col>
          <!-- search/filter -->
          <b-form-group
            label-size="sm"
            label-for="filterInput"
            class="mb-0"
          >
            <b-input-group size="sm">
              <b-form-input
                v-model="filter"
                type="search"
                id="filterInput"
                placeholder="Type to Search"
              ></b-form-input>
              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''"
                  >Clear</b-button
                >
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>
      </b-row>
    </div>

    <!-- Main table element -->
    <!-- TODO: fix date sorting -->
    <b-table
      striped hover
      show-empty
      small
      stacked="md"
      ref="table"
      :items="purchases"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :filterIncludedFields="filterOn"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      @filtered="onFiltered"
    >
      <template v-slot:cell(name)="row">
        {{ row.value.first }} {{ row.value.last }}
      </template>

      <template v-slot:cell(actions)="row">
        <b-button
          v-b-modal.info-modal
          size="sm"
          @click="info(row.item, row.index, $event.target)"
          class="mr-1"
        >
          Info modal
        </b-button>
        <b-button size="sm" @click="row.toggleDetails">
          {{ row.detailsShowing ? "Hide" : "Show" }} Details
        </b-button>
        <!-- added by me from old table -->
        <section
          class="btn-group align-self-center"
          role="group"
          aria-label="Purchase Options"
        >
          <button
            class="btn btn-sm btn-outline-secondary view"
            @click="editPurchase(row.item)"
          >
            <font-awesome-icon icon="pencil-alt" />
          </button>
          <button
            class="btn btn-sm btn-outline-secondary edit"
            @click="handleSavePurchase(item)"
          >
            <font-awesome-icon icon="save"></font-awesome-icon>
          </button>
          <button
            class="btn btn-sm btn-outline-secondary"
            @click="handleDeletePurchase(row.item)"
          >
            <font-awesome-icon icon="trash" />
          </button>
        </section>
      </template>

      <template v-slot:row-details="row">
        <b-card>
          <ul>
            <li v-for="(value, key) in row.item" :key="key">
              {{ key }}: {{ value }}
            </li>
          </ul>
        </b-card>
      </template>
    </b-table>

    

    <!-- Info modal -->
    <b-modal
      :id="infoModal.id"
      :title="infoModal.title"
      size="sm"
      ok-only
      @hide="resetInfoModal"
    >
      <b-form-group
        label="Date"
        label-for="date-input"
        invalid-feedback="Date is required"
      >
        <datepicker v-model="createdAtEdit" class="form-control">
        </datepicker>
      </b-form-group>
      <b-form-group
        label="Location"
        label-for="location-input"
        invalid-feedback="Location is required"
      >
        <b-form-input
          v-model="purchaseLocationEdit"
          required
        ></b-form-input>
      </b-form-group>
      <b-form-group
        label="Amount"
        label-for="amount-input"
        invalid-feedback="Amount is required"
      >
        <b-form-input
          v-model="purchaseAmountEdit"
          required
        ></b-form-input>
      </b-form-group>
      <b-form-group
        label="Category"
        label-for="category-input"
        invalid-feedback="Category is required"
      >
        <select
          class="form-control"
          v-model="purchaseCategoryEdit"
          aria-describedby="buttonAdd"
          ref="purchaseCategory"
        >
          <option value="null" disabled hidden>Category</option>
          <option v-for="item in categories" :key="item.id">
            {{ item.category }}
          </option>
        </select>
      </b-form-group>
    </b-modal>

    <!-- edit information modal -->
    <b-modal
      id="bv-edit-modal"
    >
      <template v-slot:modal-title>
        Edit Purchase
      </template>
      <b-form-group
        label="Date"
        label-for="date-input"
        invalid-feedback="Date is required"
      >
        <datepicker v-model="createdAtEdit" class="form-control">
        </datepicker>
      </b-form-group>
      <b-form-group
        label="Location"
        label-for="location-input"
        invalid-feedback="Location is required"
      >
        <b-form-input
          v-model="purchaseLocationEdit"
          required
        ></b-form-input>
      </b-form-group>
      <b-form-group
        label="Amount"
        label-for="amount-input"
        invalid-feedback="Amount is required"
      >
        <b-form-input
          v-model="purchaseAmountEdit"
          required
        ></b-form-input>
      </b-form-group>
      <b-form-group
        label="Category"
        label-for="category-input"
        invalid-feedback="Category is required"
      >
        <select
          class="form-control"
          v-model="purchaseCategoryEdit"
          aria-describedby="buttonAdd"
          ref="purchaseCategory"
        >
          <option value="null" disabled hidden>Category</option>
          <option v-for="item in categories" :key="item.id">
            {{ item.category }}
          </option>
        </select>
      </b-form-group>
      <!-- <b-button class="mt-3" block @click="$bvModal.hide('bv-edit-modal')">Cancel</b-button>
      <b-button class="mt-3" block @click="$bvModal.hide('bv-edit-modal')">Save</b-button> -->
    </b-modal>

    <!-- pagination -->
    <b-row>
      <b-col style="padding-top: 25px;">
        <b-pagination
          v-model="currentPage"
          :total-rows="totalRows"
          :per-page="perPage"
          align="fill"
          size="sm"
          class="my-0"
        ></b-pagination>
      </b-col>
      <b-col lg="3">
        <b-form-group
          label="per page"
          label-cols-sm="6"
          label-cols-md="4"
          label-cols-lg="3"
          label-align-sm="right"
          label-size="sm"
          label-for="perPageSelect"
          class="mb-0"
        >
          <b-form-select
            v-model="perPage"
            id="perPageSelect"
            size="sm"
            :options="pageOptions"
          ></b-form-select>
        </b-form-group>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import moment from "moment";
import Datepicker from "vuejs-datepicker";
export default {
  name: "DisplayPurchases",
  components: {
    Datepicker,
  },
  computed: {
    purchases() {
      return this.$store.getters.purchases;
    },
    categories() {
      return this.$store.getters.categories;
    },
    sortOptions() {
    // Create an options list from our fields
    return this.fields
      .filter((f) => f.sortable)
      .map((f) => {
        return { text: f.label, value: f.key };
      });
    },
  },
  watch: {
    purchases: function() {
      console.log("purchases changed");
    }
  },
  data() {
    return {
      items: this.$store.getters.purchases,
      fields: [
        {
          key: "createdAt",
          label: "Date",
          sortable: true,
          sortDirection: "desc",
          formatter: (value) => {
            return this.formatDate(value.toDate());
          },
        },
        {
          key: "purchaseLocation",
          label: "Location",
          sortable: true,
          class: "text-center",
        },
        {
          key: "purchaseAmount",
          label: "Amount",
          sortable: true,
          sortByFormatted: true,
          filterByFormatted: true,
          formatter: (value) => {
            return "$" + Number(value).toFixed(2)
          }
        },
        {
          key: "purchaseCategory",
          label: "Category",
          sortable: true,
          sortDirection: "desc",
        },
        { key: "actions", label: "Actions" },
      ],
      totalRows: 1,
      currentPage: 1,
      perPage: 10,
      pageOptions: [5, 10, 15],
      sortBy: "",
      sortDesc: false,
      sortDirection: "desc",
      filter: null,
      filterOn: [],
      // info modal:
      infoModal: {
        id: "info-modal",
        title: "",
        content: "",
      },
      // info modal edit information:
      createdAtEdit: "",
      purchaseLocationEdit: "",
      purchaseAmountEdit: "",
      purchaseCategoryEdit: "",
    };
  },
  mounted() {
    //Set the initial number of items
    this.totalRows = this.$store.getters.purchases.length;
  },
  methods: {
    // @click="info(row.item, row.index, $event.target)"
    // TODO: make it so we can actually edit a purchase
    info(item, index, button) {
      // set fields before edit
      this.purchaseLocationEdit = item.purchaseLocation;
      this.createdAtEdit = item.createdAt.toDate();
      this.purchaseAmountEdit = item.purchaseAmount;
      this.purchaseCategoryEdit = item.purchaseCategory;
      this.infoModal.title = "Edit Purchase";
      //this.infoModal.content = JSON.stringify(item, null, 2);
      this.$root.$emit("bv::show::modal", this.infoModal.id, button);
    },
    resetInfoModal() {
      this.infoModal.title = "";
      this.infoModal.content = "";
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },

    // added by me
    formatDate(date) {
      return moment(date).format("MM-DD-YYYY");
    },
    handleDeletePurchase(item) {
      console.log(item);
      this.$store.dispatch("deletePurchase", item);
    },
    // purchase edits
    editPurchase(item) {
      this.purchaseLocationEdit = item.purchaseLocation;
      this.createdAtEdit = item.createdAt.toDate();
      this.purchaseAmountEdit = item.purchaseAmount;
      this.purchaseCategoryEdit = item.purchaseCategory;
      // show modal
      this.$bvModal.show('bv-edit-modal');
    },
  },
};
</script>

<style scoped>
  .contain {
    padding-left: 100px;
    padding-right: 100px;
  }
  .ui {
    padding-left: 150px;
    padding-right: 150px;
  }
</style>