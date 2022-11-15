<template>
    <div class="app">
        <h1>Страница с постами</h1>
        <my-input v-model="searchQuery" placeholder="Поиск..."></my-input>
        <div class="app__btns">
            <my-button @click="showDialog">Создать пост</my-button>
            <my-select v-model="selectedSort" :options="sortOptions"/>
        </div>
        <my-dialog v-model:show="dialogVisible">
            <post-form @create="createPost"/>
        </my-dialog>
        <post-list v-if="!isPostLoading" :posts="sortedAndSearchedPosts" @remove="removePost"/>
        <div v-else>Идет загрузка...</div>
        <div ref = "observer" class="observer"></div>
        <!-- <div class="page__wrapper">
            <div v-for="pageNum in totalPages" 
            :key="pageNum" 
            class="page" 
            :class="{
                'current-page':pageNum === page
                }"
                @click="changePage(pageNum)"
                >
                {{pageNum}}
            </div>
        </div> -->
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
                page: 1,
                limit: 10,
                totalPages: 0,
                sortOptions:[
                    {value:'title', name: 'По названию'},
                    {value:'body', name: 'По описанию'},

                ],
                searchQuery:'',
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
            // changePage(pageNum) {
            //     this.page = pageNum;
            // },
            async fetchPosts() {
                try {
                    this.isPostLoading = true;
                    const responce = await axios.get('https://jsonplaceholder.typicode.com/posts',{
                        params: {
                            _page: this.page,
                            _limit: this.limit,
                        }
                    });
                    this.totalPages = Math.ceil(responce.headers['x-total-count']/ this.limit);
                    this.posts = responce.data;
                } catch (error) {
                    alert('Ошибка');
                } finally {
                    this.isPostLoading = false;
                }
            },
            async loadMorePosts() {
                try {
                    this.page += 1;
                    const responce = await axios.get('https://jsonplaceholder.typicode.com/posts',{
                        params: {
                            _page: this.page,
                            _limit: this.limit,
                        }
                    });
                    this.totalPages = Math.ceil(responce.headers['x-total-count']/ this.limit);
                    this.posts = [...this.posts, ...responce.data];
                } catch (error) {
                    alert('Ошибка');
                }
            },
        },
        computed: {
            sortedPosts() {
                return [...this.posts].sort((post1, post2) => {
                    return post1[this.selectedSort]?.localeCompare(post2[this.selectedSort])
                });
            },
            sortedAndSearchedPosts() {
                return this.sortedPosts.filter(post => post.title.toLowerCase().includes(this.searchQuery.toLowerCase()))
            },
        },
        watch: {
            // page() {
            //     this.fetchPosts();
            // }
        },
        mounted() {
            this.fetchPosts();
            const options = {
                rootMargin: '0px',
                threshold: 1.0
            };
            const callback = (entries, observer) => {
                if (entries[0].isIntersecting && this.page < this.totalPages) {
                    this.loadMorePosts();
                }
            };

            const observer = new IntersectionObserver(callback, options);
            observer.observe(this.$refs.observer);
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
.page__wrapper {
    display: flex;
    margin-top: 15px;
}
.page {
    border: 1px solid black;
    padding: 10px;
}
.current-page {
    border: 2px solid teal;
}
.observer {
    height: 1px;
    background: none;
}
</style>