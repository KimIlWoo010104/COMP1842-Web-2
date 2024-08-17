<template>
    <div>
        <h1>Words</h1>
        <div>
            <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="Search..." 
                class="search-input"
            />
        </div>
        <table id="words" class="ui celled compact table">
            <thead>
                <tr>
                    <th>English</th>
                    <th>German</th>
                    <th>French</th>
                    <th colspan="3"></th>
                </tr>
            </thead>
            <tr v-for="(word,i) in paginatedWords" :key="i">
                <td>{{ word.english }}</td>
                <td>{{ word.german }}</td>
                <td>{{ word.french }}</td>
                <td width="75" class="center aligned">
                    <router-link :to="{name: 'show', params: {id: word._id}}">Show</router-link></td>
                <td width="75" class="center aligned">
                    <router-link :to="{name: 'edit', params:{id: word._id}}">Edit</router-link></td>
                <td width="75" class="center aligned" @click.prevent="onDestroy(word._id)">
                    <a :href="`/words/${word._id}`">Delete</a></td>
            </tr>
        </table>
        <div class="pagination-controls">
            <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Previous</button>
            <span>Page {{ currentPage }} of {{ totalPages }}</span>
            <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">Next</button>
        </div>
    </div>
</template>

<script>
import {api} from '../helpers/helpers';

export default{
    name: 'words',
    data(){
        return {
            words:[],
            searchQuery:'',
            currentPage: 1,
            itemsPerPage:4,
        };
    },
    async mounted(){
        this.words = await api.getWords();
    },
    computed: {
        filteredWords() {
            const query = this.searchQuery.toLowerCase();
            return this.words.filter(word => {
                return (
                    word.english.toLowerCase().includes(query) ||
                    word.german.toLowerCase().includes(query) ||
                    word.french.toLowerCase().includes(query)
                );
            });
        },
        totalPages() {
            return Math.ceil(this.filteredWords.length / this.itemsPerPage);
        },
        paginatedWords() {
            const start = (this.currentPage - 1) * this.itemsPerPage;
            const end = start + this.itemsPerPage;
            return this.filteredWords.slice(start, end);
        }
    },
    watch: {
        searchQuery(newQuery) {
            this.currentPage = 1;
        }
    },
    methods: {
        async onDestroy(id) {
            const sure =window.confirm('Are you sure?');
            if(!sure) return;
            await api.deleteWord(id);
            this.flash('Word deleted successfully!', 'success');
            const newWords = this.words.filter(word => word._id !== id);
            this.words = newWords;
            if (this.currentPage > this.totalPages) {
                this.currentPage = this.totalPages;
            }
        },

    changePage(page) {
            if (page >= 1 && page <= this.totalPages) {
                this.currentPage = page;
            } 
        }
    }
};
</script>

<style scoped>
.pagination-controls {
    margin-top: 20px;
    text-align: center;
}

.pagination-controls button {
    margin: 0 5px;
}
</style>