<template>
  <v-card elevation="5">
    <v-dialog v-model="deleteConfirmation" width="500" persistent>
      <v-card class="pa-10">
        <div align="center" class="text-h6">Confirmation</div>
        <div align="center" class="pa-10">
          Are you sure you want to close this ticket?
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

        <div align="center">Replies :</div>
     
        <div v-for="x in replyList" :key="x"> 
             <div>
         <b> {{x.account_type=='You' ? 'Client' : 'Admin'}}</b>
        </div>
            {{x.message}}
          
          <v-divider>
            
          </v-divider>
           
        </div>
        <div class="pt-10">
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
        <b>Submitted Reports</b>
      </v-col>
     
      <v-spacer></v-spacer>
      <v-col align-self="center">
          <JsonExcel :data="items_all">
                <div class="text-6 pl-5" style="cursor:pointer">
                  <b>Export to Excel</b>
                </div>
          </JsonExcel>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
         <div class="px-10">
        <v-text-field outlined v-model="search" placeholder="Search"></v-text-field>
      </div>
      </v-col>
      <v-col>
         <div class="px-10">
             <v-menu
          class="pa-0"
          ref="eventDate"
          v-model="eventDate"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          max-width="290px"
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
            hide-details=""
              v-model="date"
              outlined
              label="Date"
              persistent-hint
              v-bind="attrs"
              @blur="date = date"
              v-on="on"
              append-icon="mdi-close"
               @click:append="resetDate"

            ></v-text-field>
          </template>
          <v-date-picker
            @change="changeDate"
            v-model="date"
            no-title
            range
          ></v-date-picker>
        </v-menu>
      </div>
      </v-col>
        <v-col>
         <div class="px-10">
        <v-select @change="filterCategory" outlined v-model="category" placeholder="Select offices/department"
        :items="['All','College of Agriculture, Food, Environment and Natural Resources',
            'College of Arts and Sciences',
            'College of Criminal Justice',
            'College of Economics, Management and Development Studies',
            'College of Education',
            'College of Engineering and Information Technology',
            'College of Nursing',
            'College of Sports, Physical Education and Recreation',
            'College of Veterinary Medicine and Biomedical Sciences',
            'Office of the Student Affairs and Services',
            'University Infirmary',
            'University Library',
            'University Marketing Center',
            'University Registrar',
                    ]"
                  ></v-select>
                  
      </div>
      </v-col>
      <v-col>
        <v-select @change="filterCategory" outlined v-model="category" placeholder="Select Category"
        :items="['Closed ticket','All']"
                  ></v-select>
      </v-col>
    </v-row>
    <v-data-table
      :search="search"
      class="pa-5"
      :headers="headers"
      :items="items_all"
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
                <v-list-item-title>Close the ticket</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import JsonExcel from "vue-json-excel";
export default {
components:{
    JsonExcel
  },
  created() {
    this.loadData();
  },
  data() {
    return {
      category:'',
      replyList:[],
      items_all:[],
      buttonLoad: false,
      account_type: "",
      deleteConfirmation: false,
      selectedItem: [],
      events: [],
      date:[],
      selectedItem: {},
      isLoading: false,
      users: [],
      dialogAdd: false,
      isAdd: true,
      search:'',
      replyDialog: false,
      headers: [
        { text: "Date", value: "date" },
        { text: "Ticket Number", value: "id" },
        { text: "Email", value: "email" },
        { text: "Category", value: "category" },
        { text: "Title", value: "title" },
        { text: "Updated by", value: "updated_by" },
        { text: "Actions", value: "opt" },
        ,
      ],
    };
  },
  methods: {
    filterCategory(){
       this.items_all = []
      if(this.category=='All') {
        this.items_all = this.events
        return;
      }
      if(this.category=='Closed ticket') {
           this.items_all = this.events.filter(data=>data.title==this.category)
        return;
      }
       
        this.items_all = this.events.filter(data=>data.category==this.category)
    },
    resetDate(){
      this.items_all = this.events
      this.date=[]
    },
     changeDate(){
          this.items_all = []
           for(let key in this.events){
          if(new Date(this.date[0])<=new Date(this.events[key].date) && new Date(this.date[1])>=new Date(this.events[key].date)){
             this.items_all.push(this.events[key])
           
          }
        } 
      },
    async reply() { 
      this.loading = true;
      var params = {
        report_id: this.selectedItem.id,
        message: this.events.reply,
        is_respo: "yes",
        account_type: "Admin",
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
   async replyItem(item) {
      this.selectedItem = item;
      this.replyDialog = true;
     var res =  await this.$axios
        .get(`/respo_list/${this.selectedItem.id}/`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((res) => {
            this.replyList = res.data
        });
      
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
        .patch(`/report/${this.selectedItem.id}/`,{
          "title":"Closed ticket",
          "updated_by":localStorage.getItem('email'),
        }, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then(() => {
          this.deleteConfirmation = false;
          this.buttonLoad = false;
          alert("Successfully Closed");
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
          this.items_all = res.data
          this.isLoading = false;
        });
    },
  },
};
</script>

<style>
</style>