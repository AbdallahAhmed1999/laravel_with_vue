<template>
    <div>
        <h2>Articles</h2>
        <form @submit.prevent="addArticle()"> 
            <div class="form-group">
                <input type="text" class="form-control" placeholder="Title" v-model="article.title">
            </div>
            <div class="form-group">
                <textarea class="form-control" rows="7" placeholder="Body" v-model="article.body"></textarea>
            </div>
            <button class="btn btn-primary float-right" type="submit">Save</button>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li class="page-item" v-bind:class="[{disabled: !pagination.prev_page_url}]"><a 
                 @click="fetchArticles(pagination.prev_page_url)" href="#" class="page-link">Previous</a>
                </li>

                <li class="page-item disabled">
                    <a href="#" class="page-link text-dark">Page {{pagination.current_page}} of {{pagination.last_page}}</a>
                </li>

                <li class="page-item" v-bind:class="[{disabled: !pagination.next_page_url}]">
                    <a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a>
                </li>
            </ul>
        </nav>
        <div class="card card-body my-2" v-for="article in articles" v-bind:key="article.id">
            <h3>{{ article.title }}</h3>
            <p>{{article.body}}</p>
            <hr>
            <button @click="editArticle(article)" class="btn btn-info mb-2">Edit</button>
            <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
        </div>

    </div>
</template>

<script>
import { METHODS } from 'http';
    export default {
        data(){
            return {
                articles: [],
                article : {
                    id : '',
                    title : '',
                    body : ''
                }, 
                pagination : {},
                edit : false
            }
        },
        created(){
            this.fetchArticles();
        },
        methods: {
            fetchArticles(page_url){
                let vm = this;
                page_url = page_url || 'api/articles'
                fetch(page_url)
                    .then(res => res.json())
                    .then(res => {
                        console.log(res);
                        this.articles = res.data;
                        vm.makePagination(res.meta,res.links);
                    })
                    .catch(err => console.log(err));
            },

            makePagination(meta, links){
                let pagination = {
                    current_page : meta.current_page,
                    last_page : meta.last_page,
                    next_page_url : links.next,
                    prev_page_url : links.prev
              }

              this.pagination = pagination;
            },

            deleteArticle(id){
                Swal.fire({
                    title: 'Are you sure?',
                    text: "Are you sure you want to delete this Article!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                    if (result.value) {
                        fetch(`api/articles/${id}`, {
                            method : 'delete',
                        })
                         .then(res => res.json())
                         .then(data => {
                             console.log(data);
                            Swal.fire(
                                'Deleted!',
                                'Article '+data.title+' Deleted Successfully!',
                                'success'
                            );
                            this.fetchArticles();
                         })
                         .catch(err => console.log(err));
                    }
                })
            },

            addArticle(){
                if(this.edit === false){
                    fetch('api/articles', {
                        method: 'post',
                        body: JSON.stringify(this.article),
                        headers:{
                            'content-type' :  'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        Swal.fire(
                            'Added!',
                            'Article '+this.article.title+' Saved Successfully!',
                            'success'
                        );
                        this.article.title = "";
                        this.article.body = "";
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
                }else{
                    fetch(`api/articles/${this.article.id}`, {
                        method: 'put',
                        body: JSON.stringify(this.article),
                        headers:{
                            'content-type' :  'application/json'
                        }
                    })
                    .then(res => res.json())
                    .then(data => {
                        Swal.fire(
                            'updated!',
                            'Article '+this.article.title+' Updated Successfully!',
                            'success'
                        );
                        this.article.title = "";
                        this.article.body = "";
                        this.edit = false;
                        this.fetchArticles();
                    })
                    .catch(err => console.log(err))
                }
            },
            editArticle(article){
                this.edit = true;
                this.article.id = article.id;
                this.article.title = article.title;
                this.article.body = article.body;
            }
        }
    }
</script>