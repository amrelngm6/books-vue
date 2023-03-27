<template>
    <div class="w-full">

        <div class="p-6" >
        
            <div v-if="activeBook.id" class="flex gap gap-4 w-full  my-6 " >
                <span><img :src="$parent.getImagePath(activeBook)" class="w-64 h-auto rounded-lg cover"></span>
                <div class="w-full py-6 "> 
                    <span class="text-2xl font-semibold" :title="activeBook.title" v-text="activeBook.title"></span>
                    <p class="pb-4 text-md gap gap-2 flex" :key="author_index" v-for="(author, author_index) in activeBook.authors" >
                    </p>
                    <hr />
                    <div class="py-4 flex w-full gap gap-2">
                        <span class="text-gray-400">Author(s): </span>
                        <span class="text-md gap gap-2 flex" :key="author_index" v-for="(author, author_index) in activeBook.authors" >
                            <span class="font-semibold" v-text="author.name"></span>
                            <span class="font-semibold" v-text="author.death_year"></span>
                        </span>
                    </div>

                    <hr />
                    <div class="py-4 flex w-full gap gap-2">
                        <span class="text-gray-400">Languages: </span>
                        <span class="text-md gap gap-2 flex" :key="lang_index" v-for="(lang, lang_index) in activeBook.languages" >
                            <span class="font-semibold  hover:text-red-600" v-text="lang"></span>
                        </span>
                    </div>

                    <hr />
                    <div class="py-4 flex w-full gap gap-6">
                        <span class="text-gray-400">Subjects: </span>
                        <span class="text-md gap gap-6 flex" :key="subject_index" v-for="(subject, subject_index) in activeBook.subjects" >
                            <span class="font-semibold hover:text-red-600" v-text="subject"></span>
                        </span>
                    </div>

                </div>
            </div>

            <div class="py-10 text-center font-semibold mx-auto block">
                <a href="javascript:;" class="mx-4 bg-gray-600 py-2 px-4 text-gray-100 hover:bg-red-600 rounded-lg " @click="back" >Back</a>
            </div>
        </div>

    </div>
</template>
<script>

export default {
    name: 'BookPage',
    components: {
    },
    data() {
        return {
        };
    },
    props: {
        activeBook:Object
    },
    mounted() {
        const t = this

        window.onpopstate = function() {
         t.$parent.find()   
       };
    },
    methods: {

        /**
         * Go back at browser
         */
        back()
        {
            window.history.go(-1); return false;
        } , 


    }
}
</script>
