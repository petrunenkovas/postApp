<template>
    <div class="app">
        <h1>Страница с постами</h1>
        <div class="app__btns">
            <my-button @click="showDialog">Создать пост</my-button>
            <my-select v-model="selectedSort" :options="sortOptions"/>
        </div>
        <my-dialog v-model:show="dialogVisible">
            <post-form @create="createPost"/>
        </my-dialog>
        <post-list v-if="!isPostLoading" :posts="sortedPosts" @remove="removePost"/>
        <div v-else>Идет загрузка...</div>
    </div>
</template>

<script>
import PostForm from "@/components/PostForm.vue"
import PostList from "@/components/PostList.vue"
import axios from "axios";

    export default {
        components: {
            PostForm, PostList
        },
        data() {
            return {
                posts: [],
                dialogVisible: false,
                isPostLoading: true,
                selectedSort: '',
                sortOptions:[
                    {value:'title', name: 'По названию'},
                    {value:'body', name: 'По описанию'},

                ]
            }
        },
        methods: {
            createPost(post) {
                this.posts.push(post);
            },
            removePost(post) {
                this.posts = this.posts.filter(p => p.id !== post.id);
                this.dialogVisible = false;
            },
            showDialog() {
                this.dialogVisible = true;
            },
            async fetchPosts() {
                try {
                    this.isPostLoading = true;
                    const responce = await axios.get('https://jsonplaceholder.typicode.com/posts?_limit=10');
                    this.posts = responce.data;
                } catch (error) {
                    alert('Ошибка');
                } finally {
                    this.isPostLoading = false;
                }
            },
        },
        computed: {
            sortedPosts() {
                return [...this.posts].sort((post1, post2) => {
                    return post1[this.selectedSort]?.localeCompare(post2[this.selectedSort])
                });
            }
        },
        mounted() {
            this.fetchPosts();
        },
    }
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    margin-top: 15px;
}
.app {
    padding: 20px;
}
.app__btns {
    margin: 15px 0;
    display: flex;
    justify-content: space-between;
}
</style>