<template>
  <v-card elevation="5">
      <user-add  :isOpen="dialogAdd" @cancel="dialogAdd=false" @refresh="loadData" :items="selectedItem" :isAdd="isAdd" />
     <v-dialog v-model="deleteConfirmation" width="500" persistent>
    <v-card class="pa-10">
    <div align="center" class="text-h6">Confirmation</div>
    <div align="center" class="pa-10">
        Are you sure you want to delete this item?
    </div>
      <v-card-actions>
        <v-row align="center">
            <v-col align="end">
                <v-btn color="red" text @click="isCategory=false"> Cancel </v-btn>
            </v-col>
            <v-col>
                <v-btn color="success" text :loading="buttonLoad" @click="submitCategory"> Confirm </v-btn>
            </v-col>
        </v-row>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <v-dialog v-model="isCategory" width="500" persistent>
    <v-card class="pa-10">
    <div align="center" class="text-h6">Category</div>
    <div align="center" class="pa-10">
        Please select category.
    </div>
    <div>
        <v-select outlined :items="['Category1','Category2','Category3']"  v-model="category" ></v-select>
    </div>
      <v-card-actions>
        <v-row align="center">
            <v-col align="end">
                <v-btn color="red" text @click="isCategory=false"> Cancel </v-btn>
            </v-col>
            <v-col>
                <v-btn color="success" text :loading="buttonLoad" @click="submitCategory"> Confirm </v-btn>
            </v-col>
        </v-row>
      </v-card-actions>
    </v-card>
  </v-dialog>
    <!-- <beneficiaries-add :isOpen="dialogAdd" @cancel="dialogAdd=false" @refresh="loadData" :items="selectedItem" :isAdd="isAdd" /> -->
    <v-row>
      <v-col align="start" class="pa-10 text-h5" >
        <b>Users Management</b>
      </v-col>

      <v-col align-self="center" align="end" class="pr-10" v-if="account_type=='Super Admin'">
        <v-btn
          class="rnd-btn"
          rounded
          large
          color="black"
          depressed
          dark
          width="190"
          @click="addItem"
        >
          <span class="text-none">Add User</span>
        </v-btn>
      </v-col>
    </v-row>
          <v-col align-self="center" class="pa-10 ">
        <v-text-field placeholder="search" outlined v-model="search"></v-text-field>
      </v-col>
    <v-data-table
      class="pa-5"
      :search="search"
      :headers="headers"
      :items="events"
      :loading="isLoading"
    >
     <!-- <template v-slot:[`item.is_active`]="{ item }">
        <div>
          <v-chip align="center" :style="getColorStatus(item.is_active)"
            ><span> 'Activated' : 'Deactivated' }} </span></v-chip
          >
        </div>
      </template> -->
     <template #[`item.price`]="{ item }">
          <div>
            {{formatPrice(item.price)}}
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

      <template #[`item.opt`]="{ item }">
        <v-menu offset-y z-index="1">
          <template v-slot:activator="{ attrs, on }">
            <v-btn icon v-bind="attrs" v-on="on">
              <v-icon>mdi-dots-horizontal</v-icon>
            </v-btn>
          </template>
          <v-list dense>
              <v-list-item @click.stop="editItem(item)">
              <v-list-item-content>
                <v-list-item-title>Edit</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-list-item @click.stop="status(item,'Activated')">
              <v-list-item-content>
                <v-list-item-title>Activate</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-list-item @click.stop="status(item,'Deactivated')">
              <v-list-item-content>
                <v-list-item-title>Deactivate</v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import UserAdd from "./UserAdd.vue";



export default {
  components: { UserAdd },
  created() {
    this.loadData();
    this.user_type = localStorage.getItem('account_type')
  },
  data() {
    return {
      user_type:'',
      search:'',
    category:'',
      buttonLoad:false,
      account_type:'',
      deleteConfirmation:false,
      selectedItem:[],
        events:[],
      selectedItem:{},
      isLoading: false,
      users: [],
      dialogAdd:false,
      isCategory:false,
      isAdd:true,
      headers: [
        { text: "ID", value: "id" },
        // { text: "Firstname", value: "firstname" },
        { text: "Account Type", value: "account_type" },
        { text: "Email", value: "email" },
        { text: "Status", value: "status" },
        { text: "Action", value: "opt" },
        ,
      ],
    };
  },
  methods: {
      status(item,status){
    this.buttonLoad=true
      this.$axios.patch(`/users/${item.id}/`,{
          status: status
      },{
        headers:{
          Authorization:`Bearer ${localStorage.getItem('token')}`
        }
      })
      .then(()=>{
          this.isCategory=false
          this.buttonLoad=false
        //   alert('Successfully Deleted!')
          this.loadData()
      })
      },
      approve(item){

        this.buttonLoad=true
      this.$axios.patch(`/cases/${this.selectedItem.id}/`,{
          category:this.category
      },{
        headers:{
          Authorization:`Bearer ${localStorage.getItem('token')}`
        }
      })
      .then(()=>{
          this.isCategory=false
          this.buttonLoad=false
        //   alert('Successfully Deleted!')
          this.loadData()
      })
      },
    async  submitCategory() {
        this.buttonLoad=true
      this.$axios.patch(`/cases/${this.selectedItem.id}/`,{
          category:this.category
      },{
        headers:{
          Authorization:`Bearer ${localStorage.getItem('token')}`
        }
      })
      .then(()=>{
          this.isCategory=false
          this.buttonLoad=false
        //   alert('Successfully Deleted!')
          this.loadData()
      })
      },
        setCategory(item){
            this.isCategory =true
            this.selectedItem = item
      },
    getColorStatus(item) {
      if (item) {
        return "background-color:green;border-radius:15px;padding:7px; width:150px; color:white;";
      } else  { 
        return "background-color:red;border-radius:15px;padding:7px; width:150px; color: white;";
      } 
    
    },
    async deleteValue(){
     this.buttonLoad=true
      this.$axios.delete(`/beneficiaries/${this.selectedItem.id}/`,{
        headers:{
          Authorization:`Bearer ${localStorage.getItem('token')}`
        }
      })
      .then(()=>{
          this.deleteConfirmation=false
          this.buttonLoad=false
          alert('Successfully Deleted!')
          this.loadData()
      })
    },
     deleteItem(val){
      this.selectedItem = val
      this.deleteConfirmation=true
    },

     formatPrice(value) {
      let val = (value / 1).toFixed(2).replace(",", ".");
      return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    editItem(val){
      this.selectedItem=val
      this.dialogAdd=true
      this.isAdd=false
    },
    addItem(){
      this.isAdd=true
      this.dialogAdd=true
    },
    // async status(data, status) {
    //   this.isLoading = true;
    //   const res = await this.$axios
    //     .patch(
    //       `/announcement/${data.id}/`,
    //       {
    //         is_active: status == "Deactivate" ? false : true,
    //       },
    //       {
    //         headers: {
    //           Authorization: `Bearer ${localStorage.getItem("token")}`,
    //         },
    //       }
    //     )
    //     .then((res) => {
    //       this.loadData();
    //     });
    // },
    loadData() {
      this.account_type=localStorage.getItem('account_type')
      this.eventsGetall();
    },
    async eventsGetall() {
      this.isLoading = true;
      const res = await this.$axios
        .get(`/users/`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((res) => {
          console.log(res.data);

          this.events = res.data.filter(data=>data.account_type !='Owner');
          this.isLoading = false;
        });
    },
  },
};
</script>

<style>

</style>