<template>
    <div class="form-container">


        <!-- Logo -->
        <div>
            <b-img class="pb-3 logo" src="wwflixlogo.svg" left fluid alt="NETFLIX"></b-img>
        </div>


        <!-- Form -->
        <b-form id="form1" @submit="onSubmit" @reset="onReset" v-if="showForm">
            <b-form-group id="input-group-1" label="" label-for="input-1" description="">
                <b-input-group>
                    <b-input-group-prepend>
                        <span class="input-group-text"><img src="search_icon.svg"><i class="fa fa-user fa-lg"></i></span>
                    </b-input-group-prepend>
                    <b-form-input id="input-1" type="text" required placeholder="Search movie title" v-model="$v.form_search.title.$model" :state="$v.form_search.title.$dirty ? !$v.form_search.title.$error : null" aria-describedby="input-1-live-feedback" autocomplete="off" class="LoginInput" size="lg">
                    </b-form-input>
                </b-input-group>
            </b-form-group>
            <!-- Form Button -->
            <!--        <b-button size="lg" pill :disabled="$v.form_search.$invalid" type="submit" variant="primary">Search</b-button> -->
        </b-form>


        <div v-if="this.movie_data != null">
            <!-- <b-button v-b-toggle.sidebar-1>Toggle Sidebar</b-button> -->
            <!-- Sidebar -->
            <b-sidebar id="sidebar-1" :title="this.movie_data.Title" backdrop-variant="dark" backdrop shadow @shown="sidebarShown" @hidden="sidebarHidden">
                <template #footer="{ hide }">
                    <div class="d-flex bg-dark text-light align-items-center px-3 py-2">
                        <strong class="mr-auto"></strong>
                        <b-button size="sm" @click="hide">Close</b-button>
                    </div>
                </template>
                <div class="px-3 py-2">
                    <p>
                        {{this.movie_data.Plot}}
                    </p>
                    <b-img :src="this.movie_data.Poster" fluid></b-img>
                    <small class="p-0 m-0 nolines">
                        <p class="mt-2">Actors: {{this.movie_data.Actors}}</p>
                        <p>Awards: {{this.movie_data.Awards}}</p>
                        <p>BoxOffice: {{this.movie_data.BoxOffice}}</p>
                        <p>Country: {{this.movie_data.Country}}</p>
                        <p>DVD: {{this.movie_data.DVD}}</p>
                        <p>Director: {{this.movie_data.Director}}</p>
                        <p>Genre: {{this.movie_data.Genre}}</p>
                        <p>Language: {{this.movie_data.Language}}</p>
                        <p>Metascore: {{this.movie_data.Metascore}}</p>
                        <p>Production: {{this.movie_data.Production}}</p>
                        <p>Rated: {{this.movie_data.Rated}}</p>
                        <p>Ratings: {{this.movie_data.Ratings}}</p>
                        <p>Released: {{this.movie_data.Released}}</p>
                        <p>Runtime: {{this.movie_data.Runtime}}</p>
                        <p>Title: {{this.movie_data.Title}}</p>
                        <p>Type: {{this.movie_data.Type}}</p>
                        <p>Website: {{this.movie_data.Website}}</p>
                        <p>Writer: {{this.movie_data.Writer}}</p>
                        <p>Year: {{this.movie_data.Year}}</p>
                        <p>imdbRating: {{this.movie_data.imdbRating}}</p>
                    </small>
                </div>
            </b-sidebar>
        </div>


        <!-- Result and Page Number -->
        <div v-if="this.result_data != null || this.totalResults != undefined">
            Total: {{this.totalResults}}, Page: {{this.form_search.page}}
            <!-- Filters and Pagination Buttons -->
            <b-button pill size="sm" class="m-1" v-on:click="toggleSortAZ">{{this.sorting.AZ[this.sorting.AZ_Key]}}</b-button>
            <b-button pill size="sm" v-on:click="toggleSortDate">{{this.sorting.Date[this.sorting.Date_Key]}}</b-button>
            <b-button pill size="sm" class="m-1" v-on:click="prevPage">Prev</b-button>
            <b-button pill size="sm" v-on:click="nextPage">Next</b-button>
        </div>
        

        <!-- Horizontal List -->
        <section>
            <div v-if="this.result_data != null || this.totalResults != undefined">
                <vue-horizontal-list :items="this.result_data.Search" :options="{responsive: [{end: 576, size: 1}, {start: 576, end: 768, size: 3},{size: 4}]}">
                    <template v-slot:default="{item}">
                        <div>
                            <div class="aspect r-5-3 border-2 overflow-hidden">
                                <!-- <img :src="item.Poster" /> -->
                                <small>{{item.Year}}
                                    <!-- (imdb: <b-link :href="'https://www.imdb.com/title/'+item.imdbID" target="_blank">{{item.imdbID}}</b-link> -->
                                </small>
                                <b-img thumbnail :src="item.Poster" fluid alt="Fluid image" @error="imageLoadError"></b-img>
                            </div>
                            <h6 class="pt-2 pb-0">{{item.Title}}</h6>
                            <b-button v-b-toggle.sidebar-1 @click="sendInfo(item.imdbID)">Full Plot</b-button>
                        </div>
                    </template>
                </vue-horizontal-list>
            </div>
        </section>


        <!-- End of main container -->
    </div>
</template>




<script>
/* eslint-disable */
const axios = require('axios');
import VueHorizontalList from "vue-horizontal-list";
import { validationMixin } from 'vuelidate'
import { required, minLength, helpers, alphaNum } from 'vuelidate/lib/validators'

export default {

    name: 'Omdb',
    props: {
        msg: String,
        hash: null
    },
    components: { VueHorizontalList },

    metaInfo: {
        meta: [
            { charset: 'utf-8' },
            { name: 'apple-itunes-app', content: 'app-id=myAppStoreID, affiliate-data=myAffiliateData, app-argument=myURL' }
        ],
        title: 'OMDB API',
        titleTemplate: '%s ',
        htmlAttrs: {
            lang: 'en',
            amp: true
        }
    },
    mixins: [validationMixin],

    data() {
        return {
            apiurl: this.$root.$store.state.apiURL,
            token: {},
            errors: [],
            result_data: null,
            totalResults: null,

            movie_data: {
                Plot: "",
                Poster: ""
            },

            form_search: {
                title: null,
                page: 1
            },

            showForm: true,
            omdbapi_url: "https://www.omdbapi.com",
            omdbapi_key: "16b4dde3",
            sorting: {
                AZ: ["A-Z", "Z-A"],
                AZ_Key: 0,
                Date: ["Newest First", "Oldest First"],
                Date_Key: 0
            }
        }
    },
    validations: {
        form_search: {
            title: {
                required,
                minLength: minLength(1)
            }
        }
    },

    mounted() {
        console.log(' - mounted')
        this.getOMDB();
    },

    created() {
        console.log(' - created')
    },

    methods: {
        onSubmit(evt) {
            evt.preventDefault()
            this.getOMDB();
        },

        onReset(evt) {
            evt.preventDefault()
            this.showForm = false
            this.$nextTick(() => {
                this.showForm = true
            })
        },

        sendInfo(ID) {
            console.log('ID:', ID)
            this.getPLOT(ID)
        },


        getOMDB() {
            //let apit = new Date().getTime();
            let title = this.form_search.title;
            let page = this.form_search.page;

            if (!title) {
                title = "Welcome"
                this.form_search.title = title;
                let page = "1";
            }

            axios.get(this.omdbapi_url + '?s=' + title + '&page=' + page + '&apikey=' + this.omdbapi_key, {
                    headers: {
                        /*Authorization: 'Bearer ' + this.token.access_token*/
                    }
                })
                .then(response => {
                    //console.log("response",response)
                    this.result_data = response.data;
                    this.totalResults = this.result_data.totalResults;
                    console.log("this.result_data", this.result_data)
                    console.log("Total Results:", this.result_data.totalResults)
                    console.log("Response:", this.result_data.Response)
                })
                .catch(error => {
                    console.log(error.response)
                    this.makeToast(true, error, 'danger', 'b-toaster-bottom-full', 'Could not retrieve data...')
                });
        },


        getPLOT(ID) {

            console.log("getPLOT");

            axios.get(this.omdbapi_url + '?i=' + ID + '&plot=full&apikey=' + this.omdbapi_key, {
                    headers: {
                        /*Authorization: 'Bearer ' + this.token.access_token*/
                    }
                })
                .then(response => {
                    console.log("response", response)
                    this.movie_data = response.data;
                })
                .catch(error => {
                    console.log(error.response)
                    this.makeToast(true, error, 'danger', 'b-toaster-bottom-full', 'Could not retrieve data...')
                });
        },

        imageLoadError(event) {
            console.log('Image failed to load');
            event.target.src = "poster-notfound.jpg"
        },

        sidebarShown(event) {
            console.log('Sidebar shown');
            //this.getPLOT()
        },

        sidebarHidden(event) {
            console.log('Sidebar hidden');
            this.cleanSidebar();
        },

        cleanSidebar() {
            this.movie_data.Plot = "";
            this.movie_data.Poster = "";
        },

        /* Filters */
        toggleSortAZ() {
            let list = this.result_data.Search
            if (this.sorting.AZ_Key == 0) {
                this.sorting.AZ_Key = 1;
                list.sort((a, b) => (a.Title > b.Title) ? -1 : 1)
            } else {
                this.sorting.AZ_Key = 0;
                list.sort((a, b) => (a.Title > b.Title) ? 1 : -1)
            }
        },
        toggleSortDate() {
            let list = this.result_data.Search
            if (this.sorting.Date_Key == 0) {
                this.sorting.Date_Key = 1;
                list.sort((a, b) => (a.Year > b.Year) ? -1 : 1)
            } else {
                this.sorting.Date_Key = 0;
                list.sort((a, b) => (a.Year > b.Year) ? 1 : -1)
            }
        },
        nextPage() {
            if (this.form_search.page <= this.result_data.totalResults / 10) {
                this.form_search.page += 1;
            }
            this.getOMDB()
        },
        prevPage() {
            if (this.form_search.page > 1) {
                this.form_search.page -= 1;
            }
            this.getOMDB()
        },

        makeToast(append = false, msg, variant, toaster) {
            this.toastCount++
            this.$bvToast.toast('Msg: ' + msg + ` ${this.toastCount}`, {
                title: 'Network says...',
                variant: variant,
                autoHideDelay: 5000,
                toaster: toaster,
                appendToast: append
            })
        }


    }
}
</script>
<style scoped>
.section div {
    margin-top: 24px;
    margin-bottom: 24px;
}

.background-image {
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
}

img {
    object-fit: cover;
    width: 100%;
    height: 100%;
}

.ImageBoxed {
    flex: 0 0 25%;
    max-width: 25%;
}

.logo {
    max-width: 33%;
    max-height: 100px;
    min-height: 50px;
}

.input-group-text {
    width: 48px;
    border-right: none;
    background-color: #ffffff;
}

.LoginInput {
    border-left: none;
}

.nolines {
    padding: 0;
    margin: 0;
}
</style>