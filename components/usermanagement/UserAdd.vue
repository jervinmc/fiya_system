<template>
<v-form ref="form">
  <v-dialog v-model="isOpen" width="1000" persistent>
    <v-card class="pa-10">
      <div align="center" class="text-h6">New User</div>
      <v-col cols="12" class="px-0">
        <div>Email</div>
        <div>
          <v-text-field outlined v-model="events.email"></v-text-field>
        </div>
      </v-col>
      <!-- <v-col cols="12" class="px-0">
        <div>Firstname</div>
        <div>
          <v-text-field outlined v-model="events.firstname"></v-text-field>
        </div>
      </v-col>
       <v-col cols="12" class="px-0">
        <div>Lastname</div>
        <div>
          <v-text-field outlined v-model="events.lastname"></v-text-field>
        </div>
      </v-col> -->
       <!-- <v-col cols="12" class="px-0">
        <div>Gcash Number</div>
        <div>
          <v-text-field outlined v-model="events.gcash"></v-text-field>
        </div>
      </v-col> -->
        <v-col cols="12" class="px-0">
        <div>Account Type</div>
        <div>
        <v-select outlined v-model="events.account_type" :items="['Admin','Super Admin']"></v-select>
        </div>
      </v-col>
       <v-col cols="12" class="px-0">
        <div>Password</div>
        <div>
        <v-text-field type="password" outlined v-model="events.password"></v-text-field>
        </div>
      </v-col>
      <v-col class="d-none">
        <input
          style="display: none"
          type="file"
          id="fileInput"
          ref="file"
          accept="image/png, image/jpeg"
          @change="onFileUpload"
        />
      </v-col>
      <v-card-actions>
        <v-row align="center">
          <v-col align="end">
            <v-btn color="red" text @click="cancel"> Cancel </v-btn>
          </v-col>
          <v-col>
            <v-btn
              color="success"
              text
              @click="addEvents"
              :loading="buttonLoad"
            >
              Save
            </v-btn>
          </v-col>
        </v-row>
      </v-card-actions>
    </v-card>
  </v-dialog>
  </v-form>
</template>

<script>
export default {
  props: ["isOpen", "items", "isAdd"],
  watch: {
    items() {
        this.events=this.items
        this.img_holder=this.items.image
    },
  },
  data() {
    return {
      room_list:['Standard','Deluxe','Suite'],
      events: [],
      category:[],
      buttonLoad: false,
      img_holder:'image_placeholder.png'
    };
  },
  methods: {
      async  categoryGetall(){
          this.isLoading = true;
      const res = await this.$axios
        .get(`/category/`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((res) => {
          console.log(res.data);
          this.category = res.data;
          this.isLoading = false;
        });
      },
    async addEvents() {
      this.buttonLoad = true;
      try {
        let form_data = new FormData();
        if (this.image != null && this.image != "") {
          form_data.append("image", this.image);
        }
        form_data.append("email", this.events.email);
        form_data.append("firstname", this.events.firstname);
        form_data.append("lastname", this.events.lastname);
        form_data.append("account_type", this.events.account_type);
        form_data.append("gcash", "09394337034");
       form_data.append("status", "Activated");
       form_data.append("password", this.events.password);
        if (this.isAdd) {
          const response = await this.$axios
            .post("/users/", form_data, {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            })
            .then(() => {
              this.$refs.form.reset()
              this.buttonLoad = false;
              this.$emit("cancel");
              this.$refs.form.reset()
              this.$emit("refresh");
            });
        } else {
          const response = await this.$axios
            .patch(`/users/${this.events.id}/`, form_data, {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            })
            .then(() => {
              this.buttonLoad = false;
              this.$emit("cancel");
              this.$refs.form.reset()
              this.$emit("refresh");
            });
        }
      } catch (error) {
        // alert(error);
        this.buttonLoad = false;
      }
    },
    onFileUpload(e) {
      this.image = e;
      e = e.target.files[0];
      if (e["name"].length > 100) {
        alert("255 characters exceeded filename.");
        return;
      }
      try {
        if (e.size > 16000000) {
          alert("Only 15mb file can be accepted.");
          return;
        }
      } catch (error) {
        alert(error);
        return;
      }
      this.image = e;
      if (e == null) {
      } else {
        this.url, (this.img_holder = URL.createObjectURL(e));
      }
    },
    cancel() {
      this.$emit("cancel");
    },
  },
  created(){
    this.categoryGetall()
  }
};
</script>

<style>
</style>