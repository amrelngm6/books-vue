<template>
    <div class="w-full">
        <div class=" bg-gray-600 pb-2 px-6 w-full flex ">
            <img src="./img/logo.png" class="p-4">

            <div class="w-full flex gap gap-4 pb-4 pt-6" > 
                <input v-model="searchValue" @keyup="search" placeholder="Looking for a book" type="text" class="float-right rounded-lg border-gray-400 bg-white p-2 mx-6" name="search">
            </div>
        </div> 

        <div class="p-6" v-if="isBookPage"> 
            <BookPage :key="activeBook.id" v-if="activeBook" :active-book="activeBook" />
        </div>

        <div v-if="!isBookPage">
            <div class="w-full flex  p-6">
                <h1 class="w-full text-2xl" v-text="title"></h1>
                <div class="w-full mt-4 flex gap gap-2 w-40">
                    <a @click="changeLayout('grid')" href="javascript:;" class="float-right"><img width="30" src="./img/grid.png" /></a>
                    <a @click="changeLayout('list')" href="javascript:;"><img width="30" src="./img/list.png" /></a>
                </div>
            </div>

            <div class="block p-6" v-if="booksList.length && !loading">

                <div v-if="layout && layout == 'grid' "  class="auto-cols-min grid grid-cols-4">

                    <div :key="index" v-for="(book, index) in booksList">
                        <div v-if="index < 20" class=" text-center h-full mb-6">
                            <a @click="bookPage(book.id)" href="javascript:;" class="h-full"><img :src="getImagePath(book)" class="cover mx-auto w-48 h-auto rounded-lg"></a>
                            <div class="w-full py-6 "> 
                                <a @click="bookPage(book.id)"  href="javascript:;" class="text-lg font-semibold" :title="book.title" v-text="book.title.substring(0, 100)"></a>
                            </div>
                        </div>
                    </div>
                </div>

                <div v-else class="flex gap gap-4 w-full  my-6 " :key="index" v-for="(book, index) in booksList">
                    <a @click="bookPage(book.id)"  href="javascript:;"><img :src="getImagePath(book)" class="w-48 h-auto rounded-lg cover"></a>
                    <div class="w-full py-6 "> 
                        <a @click="bookPage(book.id)"  href="javascript:;" class="text-2xl font-semibold" :title="book.title" v-text="book.title.substring(0, 350)"></a>
                        <p class="py-2 text-md gap gap-2 flex" :key="author_index" v-for="(author, author_index) in book.authors" >
                            <span v-text="author.name"></span>
                            <span class="font-semibold" v-text="author.death_year"></span>
                        </p>
                    </div>
                </div>

                <div class="py-10 text-center font-semibold mx-auto block">
                    <a href="javascript:;" class="mx-4 bg-gray-600 py-2 px-4 text-gray-100 hover:bg-red-600  rounded-lg " v-if="hasLess" @click="changePage('-')" >Previous page</a>
                    <a href="javascript:;" class="mx-4 bg-gray-600 py-2 px-4 text-gray-100 hover:bg-red-600 rounded-lg " v-if="hasMore" @click="changePage('+')" >Next page</a>
                </div>
            </div>
            <div class="block p-6" v-else-if="loading">
                Loading ... 
            </div>
            <div class="block p-6" v-else>
                No data available 
            </div>
        </div>
    </div>
</template>
<script>
const axios = require('axios').default;

import BookPage from './components/Book.vue'

export default {
    name: 'app',
    components: {
        BookPage
    },
    data() {
        return {
            title: 'Books list',
            apiURL: 'https://gutendex.com/books/',
            activeAPIPage: 1,
            layout: 'list',
            searchValue: null,
            hasMore: null,
            hasLess: null,
            loading: true,
            hasError: null,
            isBookPage: false,
            booksList: [],

            // Book Model
            activeBook: {},

            // Book page id
            activeBookId: null,
        };
    },
    mounted() {
        const t = this
        t.find()
        window.onpopstate = function() {
         t.find()   
       };
    },
    methods: {


        /**
         * Open single Book page
         */
        bookPage(id)
        {   
            this.activeBookId = id;
            var searchParams = new URLSearchParams();
            searchParams.set("id", this.activeBookId);
            history.pushState('', '', '?'+searchParams.toString());
            this.runApiBook(this.activeBookId)
            this.isBookPage = true
        },

        /**
         * Go to next/previous page
         */
        changePage(type = '+')
        {
            this.activeAPIPage = (type == '+') ? ++this.activeAPIPage : --this.activeAPIPage;
            var searchParams = new URLSearchParams(window.location.search);
            searchParams.set("page", this.activeAPIPage);
            history.pushState('', '', '?'+searchParams.toString());
            this.find()

        },

        /**
         * Change page view
         */
        changeLayout(layout = 'list')
        {
            this.layout = (layout == 'grid') ? 'grid' : 'list';
            var searchParams = new URLSearchParams(window.location.search);
            searchParams.set("layout", this.layout);
            history.pushState('', '', '?'+searchParams.toString());
            this.find()
        },


        /**
         * Filter book image format 
         */
        getImagePath(book) 
        {
            if (book && book.formats)
            {
                return book.formats['image/jpeg'];
            }
        },


        /**
         * Filter url params
         */
        getParms()
        {
            var search = location.search.substring(1);
            return search 
            ? JSON.parse('{"' + search.replace(/&/g, '","').replace(/=/g,'":"') + '"}', function(key, value) { return key===""?value:decodeURIComponent(value) })
            : {}
        }, 

        /**
         * Search for books 
         *
         */
        search()
        {

            var searchParams = new URLSearchParams(window.location.search);
            searchParams.set("search", this.searchValue);
            history.pushState('', '', '?'+searchParams.toString());
            this.find()
        }, 

        /**
         * Find books through the API
         *
         */
        find()
        {
            let url = new URLSearchParams(this.handleRouteUrl()).toString();
            history.pushState('', '', '?'+url);
            this.runApi('?'+url)
            this.isBookPage = false
        },

        /** 
         * Check the route params 
         */ 
        handleRouteUrl()
        {
            let params = this.getParms();
            this.layout = params.layout ? params.layout : this.layout;
            this.activeAPIPage = params.page ? params.page : this.activeAPIPage;
            this.searchValue = params.search ? params.search : this.searchValue;

            let url = ''; 
            url +=  '?page=' + this.activeAPIPage;
            url +=  this.searchValue ? '&search=' + this.searchValue : '';
            url +=  this.layout ? '&layout=' + this.layout : '';

            return url;
        },


        /**
         * Call the API and handle the response
         */
        runApi(params = '')
        {
            this.loading = true;
            this.handleRequest(params).then(response=>{
                this.loading = false;
                this.booksList = response.results ? response.results  : [];
                this.hasMore = response.next ? true : false;
                this.hasLess = response.previous ? true : false;

                if (response.detail)
                {
                    this.hasError = response.detail
                }
            })
        },

        /**
         * Call the API and handle the response
         */
        runApiBook(id)
        {
            this.loading = true;
            this.handleRequest(id).then(response=>{
                this.loading = false;
                this.activeBook = response
                if (response.detail)
                {
                    this.hasError = response.detail
                }
            })
        },

        async handleRequest(params) {


            return await axios.get(this.apiURL + params).then(response => {
                return response.data;
            });
        }
    }
}
</script>

<style lang="css">
    .cover
    {
        max-height: 300px;
    }
</style>