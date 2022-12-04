<template>
  <div>
    <b-row>
      <b-alert v-model="showSuccessAlert" variant="success" dismissible>
        {{ alertMessage }}
      </b-alert>
    </b-row>
    <b-row>
      <donor-overview
        :totalDonors="numberOfDonors"
        :activeDonors="activeDonors"
        @totalDonorsIsActive="setFilterTotalIsActive"
        @activeDonorIsActive="setFilterActiveIsActive"
      ></donor-overview>
    </b-row>
    <b-row class="mt-3">
      <b-card>
        <b-row align-h="between">
          <b-col cols="6"></b-col>
          <b-col cols="2">
            <b-row>
              <b-col>
                <b-button
                  variant="primary"
                  id="show-btn"
                  @click="showCreateModal"
                >
                  <span class="h6 text-white">Add item</span>
                </b-button>
              </b-col>
            </b-row>
          </b-col>
        </b-row>
        <b-row class="mt-3">
          <b-table
            striped
            hover
            :items="items"
            :fields="fields"
            class="text-center"
          >
            <template #cell(contact_name)="data">
              {{ `${data.item.firstname} ${data.item.lastname}` }}
            </template>
            <template #cell(donor_status)="data">
              <b-icon-bookmark-check-fill
                variant="success"
                v-if="data.item.donor_status === 'active'"
              ></b-icon-bookmark-check-fill>
              <b-icon-bookmark-x-fill
                variant="danger"
                v-else
              ></b-icon-bookmark-x-fill>
            </template>
            <!-- <template>
              <b-checkbox v-model="checkbox_1"> lalalala </b-checkbox>
              <code>{{ checkbox_1 }}</code>
              <hr />
            </template>-->
            <template #cell(actions)="data">
              <b-row>
                <b-col cols="7">
                  <b-icon-pencil-square
                    class="action-item"
                    variant="primary"
                    @click="getRowData(data.item.id)"
                  ></b-icon-pencil-square>
                </b-col>
                <b-col cols="1">
                  <b-icon-trash-fill
                    class="action-item"
                    variant="danger"
                    @click="showDeleteModal(data.item.id)"
                  ></b-icon-trash-fill>
                </b-col>
              </b-row>
            </template>
          </b-table>
        </b-row>
      </b-card>
    </b-row>

    <!-- Modal for adding new donors -->
    <b-modal ref="create-donor-modal" size="xl" hide-footer title="New Donor">
      <create-donor-form
        @closeCreateModal="closeCreateModal"
        @reloadDataTable="getDonorData"
        @showSuccessAlert="showAlertCreate"
      ></create-donor-form>
    </b-modal>

    <!-- Modal for updating donors -->
    <b-modal ref="edit-donor-modal" size="xl" hide-footer title="Edit Donor">
      <edit-donor-form
        @closeEditModal="closeEditModal"
        @reloadDataTable="getDonorData"
        @showSuccessAlert="showAlertUpdate"
        :donorId="donorId"
      ></edit-donor-form>
    </b-modal>

    <!-- Delete donor Modal -->
    <b-modal
      ref="delete-donor-modal"
      size="md"
      hide-footer
      title="Confirm Deletion"
    >
      <donor-delete
        @closeDeleteModal="closeDeleteModal"
        @reloadDataTable="getDonorData"
        @showDeleteAlert="showDeleteSuccessModal"
        :donorId="donorId"
      ></donor-delete>
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";
import DonorOverview from "@/components/DonorOverview.vue";
import CreateDonorForm from "@/components/CreateDonorForm.vue";
import EditDonorForm from "@/components/EditDonorForm.vue";
import DonorDelete from "@/components/DonorDelete.vue";

export default {
  components: {
    DonorOverview,
    CreateDonorForm,
    EditDonorForm,
    DonorDelete,
  },
  data() {
    return {
      //   checkbox_1: null,
      fields: [
        {
          key: "title",
          label: "Title",
          sortable: true,
        },
        {
          key: "body",
          label: "Body",
          sortable: false,
        },
        {
          key: "checkbox",
          label: "Check if yes",
          sortable: false,
        },
        "actions",
      ],
      items: [],
      numberOfDonors: 0,
      activeDonors: 0,
      activeDonorsData: [],
      donorId: 0,
      companySearchTerm: "",
      tableHeader: "",
      showSuccessAlert: false,
      alertMessage: "",
    };
  },
  mounted() {
    this.getDonorData();
  },
  methods: {
    showCreateModal() {
      this.$refs["create-donor-modal"].show();
    },
    closeCreateModal() {
      this.$refs["create-donor-modal"].hide();
    },
    getDonorData() {
      axios
        .get("http://localhost:3000/donors/")
        .then((response) => {
          this.items = response.data;
          this.numberOfDonors = response.data.length;
          this.activeDonorsData = response.data.filter(
            (item) => item.donor_status === "active"
          );
          this.activeDonors = this.activeDonorsData.length;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getRowData(id) {
      this.$refs["edit-donor-modal"].show();
      this.donorId = id;
    },
    closeEditModal() {
      this.$refs["edit-donor-modal"].hide();
    },
    setFilterTotalIsActive() {
      this.tableHeader = "Total donors";
      this.getDonorData();
    },
    showAlertCreate() {
      this.showSuccessAlert = true;
      this.alertMessage = "Donor was created successfully!";
    },
    showAlertUpdate() {
      this.showSuccessAlert = true;
      this.alertMessage = "Donor was updated successfully";
    },
    showDeleteModal(id) {
      this.$refs["delete-donor-modal"].show();
      this.donorId = id;
    },
    closeDeleteModal() {
      this.$refs["delete-donor-modal"].hide();
    },
    showDeleteSuccessModal() {
      this.showSuccessAlert = true;
      this.alertMessage = "Donor was deleted successfully!";
    },
  },
};
</script>

<style>
.action-item:hover {
  cursor: pointer;
}
</style>
