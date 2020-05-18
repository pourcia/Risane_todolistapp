<template>
  <div v-if="!loading">
      <img class="rounder mx-auto d-block" :src="image" alt="loader" />
    </div>

    <div v-else>
      <button @click="createModal" class="btn btn-primary btn-block">Add New Task</button>
      <table class="table" v-if="tasks">
        <thead>
          <tr>
            <th>ID</th>
            <th>NAME</th>
            <th>BODY</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(task, index) in tasks">
            <td>{{ index + 1 }}</td>
            <td>{{ task.name }}</td>
            <td>{{ task.body }}</td>
            <td>
              <button @click="updateModal(index)" class="btn btn-outline-info">Edit</button>
            </td>
            <td>
              <button @click="deleteTask(index)" class="btn btn-outline-danger">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Modal -->
      <div
        class="modal fade"
        id="create-modal"
        tabindex="-1"
        role="dialog"
        aria-labelledby="exampleModalLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel">Create Modal</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <div class="alert alert-danger" v-if="errors.length > 0">
                <ul>
                  <li v-for="error in errors">{{ error }}</li>
                </ul>
              </div>
              <div class="form-group">
                <label for="name">Name</label>
                <input v-model="task.name" type="text" id="name" class="form-control" />
              </div>
              <div class="form-group">
                <label for="description">Description</label>
                <input v-model="task.body" type="text" id="description" class="form-control" />
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
              <button @click="createTask" type="button" class="btn btn-primary">Save changes</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Modal -->
      <div
        class="modal fade"
        id="update-modal"
        tabindex="-1"
        role="dialog"
        aria-labelledby="exampleModalLabel"
        aria-hidden="true"
      >
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel">Update Modal</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <div class="alert alert-danger" v-if="errors.length > 0">
                <ul>
                  <li v-for="error in errors">{{ error }}</li>
                </ul>
              </div>
              <div class="form-group">
                <label for="name">Name</label>
                <input v-model="new_update_task.name" type="text" id="name" class="form-control" />
              </div>
              <div class="form-group">
                <label for="description">Description</label>
                <input v-model="new_update_task.body" type="text" id="description" class="form-control" />
              </div>
            </div>
            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
              <button @click="updateTask" type="button" class="btn btn-primary">Save changes</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
    export default {
  data() {
    return {
      task: {
        name: "",
        body: ""
      },
      tasks: [],
      uri: "http://127.0.0.1:8000/tasks/",
      errors: [],
      new_update_task: [],
      image: "img/loader.gif",
      loading: false,
      toastr: (toastr.options = { positionClass: "toast-top-full-width" })
    };
  },

  methods: {
    createModal() {
      $("#create-modal").modal("show");
    },

    updateModal(index) {
      this.errors = [];
      $("#update-modal").modal("show");
      this.new_update_task = this.tasks[index];
    },

    loadTask() {
      axios.get(this.uri).then(response => {
        this.tasks = response.data.tasks;
        this.loading = true;
      });
    },

    createTask() {
      axios
        .post(this.uri, { name: this.task.name, body: this.task.body })
        .then(response => {
          this.resetData();
          this.tasks.push(response.data.task);
          $("#create-modal").modal("hide");
          toastr.success(response.data.message);
        })

        .catch(error => {
          this.errors = [];
          if (error.response.data.errors.name) {
            this.errors.push(error.response.data.errors.name[0]);
          }

          if (error.response.data.errors.body) {
            this.errors.push(error.response.data.errors.body[0]);
          }
        });
    },

    updateTask(){
        axios.patch(this.uri + this.new_update_task.id, {
            name: this.new_update_task.name,
            body: this.new_update_task.body,
        }).then(response => {
            $("#update-modal").modal("hide");
            toastr.success(response.data.message);
        }).catch(error=>{

            this.errors = [];
            if(error.response.data.errors.name){
                this.errors.push(error.response.errors.name[0]);
            }

            if(error.response.data.errors.body){
                this.errors.push(error.response.errors.body[0]);
            }
        })
    },

    deleteTask(index){

        let confirmBox = confirm("Do you really want to delete this?");

        if(confirmBox == true){
            axios.delete(this.uri + this.tasks[index].id)
            .then(response=>{
                this.$delete(this.tasks, index);
                toastr.success(response.data.message);
            }).catch(error =>{
                console.log("Could not delete for some reason")
            });
        }
    },

    resetData() {
      this.task.name = "",
      this.task.body = ""
    }
  },

  mounted() {
    this.loadTask();
  }
};
</script>
