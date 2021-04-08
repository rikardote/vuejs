<template>
    <div class="container">
        <div class="row">
            <div class="col-md-12">
                    <div class="mt-5">
                        <button @click="initAddPost()" class="btn btn-primary btn-xs"> + Add New Post </button>
                    </div>

                        <table class="table table-bordered table-striped table-responsive mt-5" v-if="posts.length > 0">
                            <tbody>
                                <tr>
                                    <th>
                                        No.
                                    </th>
                                    <th>
                                        Title
                                    </th>
                                    <th>
                                        Description
                                    </th>
                                    <th>
                                        Action
                                    </th>
                                </tr>
                                <tr v-for="(posts, index) in posts" :key="posts.id">
                                    <td>{{ index + 1 }}</td>
                                    <td>
                                        {{ posts.title }}
                                    </td>
                                    <td>
                                        {{ posts.description }}
                                    </td>
                                    <td>
                                        <a href="#" @click="initUpdate(index)"><span class="fa fa-pencil-square-o" aria-hidden='true'></span></a>
                                        <a href="#" @click="deletePost(index)"><span class="fa fa-trash" aria-hidden="true"></span></a>
                                    </td>
                                </tr>
                            </tbody>
                        </table>

            </div>
        </div>

        <div class="modal fade" tabindex="-1" role="dialog" id="add_post_model">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title">Add New Post</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                                aria-hidden="true">X</span></button>

                    </div>
                    <div class="modal-body">

                        <div class="alert alert-danger" v-if="errors.length > 0">
                            <ul>
                                <li v-for="error in errors" :key="error.id">{{ error }}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label for="title">Title:</label>
                            <input type="text" name="title" id="title" placeholder="Title Name" class="form-control"
                                   v-model="posts.title">
                        </div>
                        <div class="form-group">
                            <label for="description">Description:</label>
                            <textarea name="description" id="description" cols="30" rows="5" class="form-control"
                                      placeholder="Post Description" v-model="posts.description"></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                        <button type="button" @click="createPost" class="btn btn-primary">Submit</button>
                    </div>
                </div><!-- /.modal-content -->
            </div><!-- /.modal-dialog -->
        </div><!-- /.modal -->

        <div class="modal fade" tabindex="-1" role="dialog" id="update_post_model">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title">Update Post</h4>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">X</span></button>
                    </div>
                    <div class="modal-body">

                        <div class="alert alert-danger" v-if="errors.length > 0">
                            <ul>
                                <li v-for="error in errors" :key="error.id">{{ error }}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label>Title:</label>
                            <input type="text" placeholder="Title" class="form-control"
                                   v-model="update_post.title">
                        </div>
                        <div class="form-group">
                            <label for="description">Description:</label>
                            <textarea cols="30" rows="5" class="form-control"
                                      placeholder="Task Description" v-model="update_post.description"></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                        <button type="button" @click="updatePost" class="btn btn-primary">Submit</button>
                    </div>
                </div><!-- /.modal-content -->
            </div><!-- /.modal-dialog -->
        </div><!-- /.modal -->

    </div>
</template>

<script>
import Swal from 'sweetalert2';
    export default {
        data(){
            return {
                status: '',
                message: '',
                posts: {
                    title: '',
                    description: ''
                },
                errors: [],
                posts: [],
                update_post: {}
            }
        },
        mounted()
        {
            this.readPosts();
        },
        methods: {
            showAlert(status, message){
                this.$swal(
                    'Exito!',
                    message,
                    status
                );
            },
            initAddPost()
            {
                this.errors = [];
                $("#add_post_model").modal("show");
            },
            createPost()
            {
                axios.post('/posts', {
                    title: this.posts.title,
                    description: this.posts.description,
                })
                    .then(response => {

                        this.reset();

                        $("#add_post_model").modal("hide");
                        this.readPosts();
                        this.showAlert('success','Registro Exitoso');

                    })
                    .catch(error => {
                        this.errors = [];
                        if (error.response.data.errors.title) {
                            this.errors.push(error.response.data.errors.title[0]);
                        }

                        if (error.response.data.errors.description) {
                            this.errors.push(error.response.data.errors.description[0]);
                        }
                    });
            },
            reset()
            {
                this.posts.title = '';
                this.posts.description = '';
            },
            readPosts()
            {
                axios.get('/posts')
                    .then(response => {

                        this.posts = response.data.posts;

                    });
            },
            initUpdate(index)
            {
                this.errors = [];
                $("#update_post_model").modal("show");
                this.update_post = this.posts[index];
            },
            updatePost()
            {
                axios.patch('/posts/' + this.update_post.id, {
                    title: this.update_post.title,
                    description: this.update_post.description,
                })
                    .then(response => {

                        $("#update_post_model").modal("hide");

                    })
                    .catch(error => {
                        this.errors = [];
                        if (error.response.data.errors.title) {
                            this.errors.push(error.response.data.errors.title[0]);
                        }

                        if (error.response.data.errors.description) {
                            this.errors.push(error.response.data.errors.description[0]);
                        }
                    });
            },
            deletePost(index)
            {
                this.$swal({
                title: 'Esta seguro de borrarlo?',
                text: "No lo podra recuperar",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Si, Borralo!'
                }).then((result) => {
                    if (result.isConfirmed) {
                        axios.delete('/posts/' + this.posts[index].id)
                            .then(response => {
                            this.posts.splice(index, 1);
                        });
                        Swal.fire(
                        'Borrado!',
                        'El registro se elimino exitosamente.',
                        'success'
                        );

                    }
                });
            }
        }
    }
</script>
