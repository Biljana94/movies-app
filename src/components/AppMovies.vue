<template>
    <div class="default">
        <!--ako ima filmova u nizu (v-if="filteredMovies.length"), 
        filteredMovies je computed properti koji pozivamo kao svaki drugi properti ako treba da ga pozovemo --> 
        <template v-if="filteredMovies.length">
            <!--uzimamo filmo iz filteredMovies-->
            <h4 class="selected">Selected Movies: {{selectedMovies}}</h4>
            <div class="de-selected-all">
                <button class="btn btn-outline-dark">Select All</button>
                <button class="btn btn-outline-dark">Deselect All</button>
            </div>
            
            <div class="" v-for="(movie, index) in filteredMovies" :key="index">
                <!--ispisujemo filmove ciji je ispis u MovieRow.vue komponenti, 
                zato moramo bind-ovati movie *:movie(ono sto smo prosledili kroz props)="movie(movie iz v-for petlje)"*;
                @selected="selectedMovie(movie) na selected se okida metoda iz child komponente jer smo tamo napisali button a poziva
                se metoda iz ove komponente koja samo broji selektovane filmove-->
                <movie-row :movie="movie" @selected="selectedMovie()"></movie-row>
            </div>
        </template>

        <!--ako nema filmova-->
        <template v-else>
            <div class="jumbotron jumbotron-fluid">
                <h1 class="display-4">Notice</h1>
                <p class="lead">The movie you are searching for does not exist in our database.</p>
            </div>
        </template>
    </div>
</template>


<script>

import { moviesService } from '../services/MoviesService.js';
import MovieRow from './MovieRow.vue';

export default {
    components: {
        MovieRow,
    },

    data () {
        return {
            movies: [],
            term: '',
            selected: false, //u childu(MovieRow.vue) menjamo selected na true
            selectedMovies: 0,
            // take: 10,
            // skip: 5,
            // // total: 0,
            // page: 1,

        }
    },

    //hook - Filmovi treba da se dobavljaju u `beforeRouteEnter`
    //`beforeRouteEnter` nema 'this' pa zato u next() pisemo 'vm' - to nam je kao this
    beforeRouteEnter(to, from, next) {
        moviesService.getAll()
            .then(response => {
                next(vm => {
                    // console.log(response)
                    vm.movies = response;
                    // vm.total = response.total;
                })
            });
    },

    //
    created() {

        //nad window pozivamo EventHub koji osluskuje input polje iz MovieSearch.vue
        window.EventHub.$on('search', (term) => { //search-ujemo(parametar) termin(term)
            moviesService.getAll(term)
                .then(response => {
                    this.term = term; //term koji korisnik unese stavljamo u nas this.term iz data(){return{term: ''}} da mozemo da izbacimo rezultate
                    this.movies = response;
                })
        })
    },

    computed: {
        //filtrirali smo niz 'movies' i svaki film koji korisnik unese u search polje uziamo pomocu this.term
        filteredMovies() {
            return this.movies.filter(movie =>  movie.title.toLowerCase().includes(this.term.toLowerCase()));
        }
    },

    methods: {
        selectedMovie() {
            // this.selected = true; //kad se klikne na dugme selected se menja u true
            // this.movie.selected = true; 
            this.selectedMovies++;
        },

        // selectAll() {
            
        // }
    },
}
</script>

<style>
.default {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: flex-start;
    margin: 2rem;
}

.jumbotron {
    width: 80%;
    display: block;
    margin: 0 auto;
    border: none;
}

.selected {
    display: block;
    margin: 0 auto;
    width: 80%;
    margin-bottom: 1rem;
}

.de-selected-all {
    display: block;
    margin: 0 auto;
    width: 80%;
}

.de-selected-all button {
    margin: 0 1rem;    
}
</style>
