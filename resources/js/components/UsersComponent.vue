<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users Table</h3>

                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal">Add new <i class="fas fa-user-plus fa-fw"></i></button>
                        </div>
                    </div>
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                                <tr>
                                    <th>ID</th>
                                    <th>Name</th>
                                    <th>Email</th>
                                    <th>Type</th>
                                    <th>Registered at</th>
                                    <th>Modify</th>
                                </tr>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at | myDate}}</td>
                                <td>
                                    <a href="#" @click="editModal(user)">
                                        <i class="fa fa-edit blue"></i>
                                    </a>
                                    /
                                    <a href="#" @click="deleteUser(user.id)">
                                        <i class="fa fa-trash red"></i>
                                    </a>
                                </td>
                            </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 v-show="!editMode" class="modal-title" id="addNewLabel">Add New</h5>
                        <h5 v-show="editMode" class="modal-title" id="addNewLabel">Update user</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser() : createUser()">
                    <div class="modal-body">
                        <div class="form-group">
                            <input v-model="form.name" type="text" name="name"
                            placeholder="Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.email" type="email" name="email"
                                   placeholder="Email address"
                                   class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                            <textarea v-model="form.bio" name="bio" id="bio"
                                   placeholder="Short bio for user (Optional)"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <select name="type" v-model="form.type" id="type" class="form-control" :class="{'is-invalid': form.errors.has('type')}">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standart User</option>
                                <option value="author">Author</option>
                            </select>
                            <has-error :form="form" field="type"></has-error>
                        </div>

                        <div class="form-group">
                            <input v-model="form.password" type="password" name="password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }"
                            >
                            <has-error :form="form" field="password"></has-error>
                        </div>

                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                        <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                        <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                    </div>
                    </form>
                </div>
            </div>
        </div>

    </div>



</template>

<script>
    export default {

        data() {
            return {
                editMode: false,
                users: {},
                form: new Form({
                    id: '',
                    name : '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: 'photo.jpg'
                })
            }
        },
        methods: {
            updateUser(){
                this.$Progress.start();
                this.form.put('api/user/' + this.form.id )
                    .then(() => {
                        $('#addNew').modal('hide');
                        swal.fire(
                            'Updated!',
                            'Information has been updated',
                            'success'
                        );
                        this.$Progress.finish();
                        Fire.$emit('AfterCreated');
                    })
                    .catch(() => {
                        swal.fire(
                            'Failed!',
                            'There was something wrong...',
                            'warning'
                        );
                        this.$Progress.fail();
                    });
            },
            editModal(user){
                this.editMode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user);

            },
            newModal(){
                this.editMode = false;
                this.form.reset();
                $('#addNew').modal('show');

            },
            deleteUser(id){
                 swal.fire({
                     title: 'Are you sure?',
                     text: "You won't be able to revert this!",
                     type: 'warning',
                     showCancelButton: true,
                     confirmButtonColor: '#3085d6',
                     cancelButtonColor: '#d33',
                     confirmButtonText: 'Yes, delete it!'
                 }).then((result) => {

                     // send request to the server
                    if (result.value) {
                        this.form.delete('api/user/' + id).then(() => {
                            swal.fire(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            );
                            Fire.$emit('AfterCreated');
                        }).catch(() => {
                            swal.fire(
                                'Failed!',
                                'There was something wrong...',
                                'warning'
                            );
                        });
                    }
                 })
            },
            loadUsers() {
                if(this.$gate.isAdminOrAuthor()) {
                    axios.get("api/user").then(({data}) => (this.users = data.data));
                }
            },
            async createUser() {
                this.$Progress.start();
                await this.form.post('api/user')
                    .then(()=>{

                        Fire.$emit('AfterCreated');
                        $('#addNew').modal('hide');

                        toast.fire({
                            type: 'success',
                            title: 'User created successfully',
                        });

                        this.$Progress.finish();

                    })
                    .catch(() => {

                    });
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreated', () => {
                this.loadUsers();
            });
            //setInterval(() => this.loadUsers(), 5000);

        }
    }
</script>
