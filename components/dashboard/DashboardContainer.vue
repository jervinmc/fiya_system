<template>
  <v-card elevation="5">
    <v-dialog v-model="deleteConfirmation" width="500" persistent>
      <v-card class="pa-10">
        <div align="center" class="text-h6">Confirmation</div>
        <div align="center" class="pa-10">
          Are you sure you want to delete this item?
        </div>
        <v-card-actions>
          <v-row align="center">
            <v-col align="end">
              <v-btn color="red" text @click="deleteConfirmation = false">
                Cancel
              </v-btn>
            </v-col>
            <v-col>
              <v-btn
                color="success"
                text
                :loading="buttonLoad"
                @click="deleteValue"
              >
                Confirm
              </v-btn>
            </v-col>
          </v-row>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="replyDialog" width="500" persistent>
      <v-card class="pa-10">
          <div align="center">Message</div>
        <div>
          <v-row>
            <div>Concerns</div>
            <v-col cols="12">
              <v-textarea
              readonly
                outlined
                v-model="selectedItem.message"
                placeholder="message here."
              ></v-textarea>
            </v-col>
          </v-row>
        </div>
        <div align="center">Reply</div>
        <div>
          <v-row>
            <div>Message</div>
            <v-col cols="12">
              <v-textarea
                outlined
                v-model="events.reply"
                placeholder="message here."
              ></v-textarea>
            </v-col>
          </v-row>
        </div>
        <v-card-actions>
          <v-row align="center">
            <v-col align="end">
              <v-btn color="red" text @click="replyDialog = false">
                Cancel
              </v-btn>
            </v-col>
            <v-col>
              <v-btn color="success" text :loading="buttonLoad" @click="reply">
                Confirm
              </v-btn>
            </v-col>
          </v-row>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-row>
      <v-col align="start" class="pa-10 text-h5" cols="auto">
        <b>Inventory Report</b>
      </v-col>
      <v-spacer></v-spacer>
    </v-row>
    <v-data-table
      class="pa-5"
      :headers="headers"
      :items="events"
      :loading="isLoading"
    >
      <template v-slot:[`item.status`]="{ item }">
        <span>{{ item.status }} </span>
      </template>
      <template #[`item.price`]="{ item }">
        <div>
          {{ formatPrice(item.price) }}
        </div>
      </template>
      <template #[`item.stocks`]="{ item }">
        <div>
          {{ item.status == "Add" ? `+${item.stocks}` : `-${item.stocks}` }}
        </div>
      </template>
      <template v-slot:loading>
        <v-skeleton-loader
          v-for="n in 5"
          :key="n"
          type="list-item-avatar-two-line"
          class="my-2"
        ></v-skeleton-loader>
      </template>
      <template #[`item.image`]="{ item }">
        <v-img :src="item.image" height="150" width="150"></v-img>
      </template>
      <template #[`item.opt`]="{ item }">
        <v-menu offset-y z-index="1">
          <template v-slot:activator="{ attrs, on }">
            <v-btn icon v-bind="attrs" v-on="on">
              <v-icon>mdi-dots-horizontal</v-icon>
            </v-btn>
          </template>
          <v-list dense>
            <v-list-item @click.stop="replyItem(item)">
              <v-list-item-content>
                <v-list-item-title>Reply</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-list-item @click.stop="deleteItem(item)">
              <v-list-item-content>
                <v-list-item-title>Delete</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
export default {
  created() {
    this.loadData();
  },
  data() {
    return {
      buttonLoad: false,
      account_type: "",
      deleteConfirmation: false,
      selectedItem: [],
      events: [],
      selectedItem: {},
      isLoading: false,
      users: [],
      dialogAdd: false,
      isAdd: true,
      replyDialog: false,
      headers: [
        { text: "ID", value: "id" },
        { text: "Message", value: "message" },
        { text: "Status", value: "status" },
        { text: "Is Viewed?", value: "is_viewed" },
        { text: "Is Replied?", value: "is_replied" },
        { text: "Actions", value: "opt" },
        ,
      ],
    };
  },
  methods: {
    async reply() {
      this.loading = true;
      var params = {
        report_id: this.selectedItem.id,
        message: this.events.reply,
        is_respo: "yes",
        status: "",
      };
      var params1 = {
        is_viewed: "no",
        is_replied: "yes",
      };
      this.$axios.post("/report/", params);
      this.$axios.patch(`/report/${this.selectedItem.id}/`, params1);
      this.loading = false;
      this.replyDialog = false;
      this.loadData();
    },
    replyItem(item) {
      this.selectedItem = item;
      this.replyDialog = true;
    },
    getColorStatus(item) {
      if (item == "Pending") {
        return "background-color:#FFF5CC;border-radius:15px;padding:7px; width:150px; color: #344557;";
      } else if (item == "Approved") {
        return "background-color:green;border-radius:15px;padding:7px; width:150px; color:white;";
      } else {
        return "background-color:red;border-radius:15px;padding:7px; width:150px; color: white;";
      }
    },
    async deleteValue() {
      this.buttonLoad = true;
      this.$axios
        .delete(`/report/${this.selectedItem.id}/`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then(() => {
          this.deleteConfirmation = false;
          this.buttonLoad = false;
          alert("Successfully Deleted!");
          this.loadData();
        });
    },
    deleteItem(val) {
      this.selectedItem = val;
      this.deleteConfirmation = true;
    },

    formatPrice(value) {
      let val = (value / 1).toFixed(2).replace(",", ".");
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    editItem(val) {
      this.selectedItem = val;
      this.dialogAdd = true;
      this.isAdd = false;
    },
    addItem() {
      this.isAdd = true;
      this.dialogAdd = true;
    },
    async status(data, status) {
      this.isLoading = true;
      const res = await this.$axios
        .patch(
          `/announcement/${data.id}/`,
          {
            is_active: status == "Deactivate" ? false : true,
          },
          {
            headers: {
              Authorization: `Bearer ${localStorage.getItem("token")}`,
            },
          }
        )
        .then((res) => {
          this.loadData();
        });
    },
    loadData() {
      this.account_type = localStorage.getItem("account_type");
      this.eventsGetall();
    },
    async eventsGetall() {
      this.isLoading = true;
      const res = await this.$axios
        .get(`/report/`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((res) => {
          console.log(res.data);
          this.events = res.data;
          this.isLoading = false;
        });
    },
  },
};
</script>

<style>
</style>