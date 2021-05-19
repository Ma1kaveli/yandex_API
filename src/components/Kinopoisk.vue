<template>
    <ScrollView orientation="vertical">
        <StackLayout v-if="this.film_active == false">
            <FlexboxLayout justifyContent="center">
                <Button class="param__btn" text="Параметры" @tap="settings()" />
                <Button class="param__btn" text="Поиск" @tap="searchFilms(0)" />
            </FlexboxLayout>
            <FlexboxLayout class="films" flexDirection="column" v-for="(el, k) in films" v-bind:key="k">
                <Image :src="el.image" />
                <Button class="btn__films" :text="el.name" @tap="filmInformation(el.id)" />
            </FlexboxLayout>
            <FlexboxLayout justifyContent="center">
                <Button class="navigation__btn" v-if="this.pages != 1" text="back" @tap="pagination(1)" />
                <Button class="navigation__btn" v-if="this.pages <= this.pagesCount" text="next" @tap="pagination(2)" />
            </FlexboxLayout>
        </StackLayout>
        <StackLayout v-if="this.film_active == true">
            <FlexboxLayout justifyContent="flex-start">
                <Button text="Назад" @tap="closeInformationFilm()" />
                <Label v-model="this.name" />
            </FlexboxLayout>
            <FlexboxLayout justifyContent="flex-start">
                <Image :src="this.image"  /> 
                <Label v-model="this.year" />
                <Label v-model="this.film_length" />
                <Label v-model="this.rating" />
                <Label v-model="this.rating_age_limit" />
            </FlexboxLayout>
            <FlexboxLayout>
                    <Label editable="false" v-model="this.description" />
            </FlexboxLayout>
            <Button text="Трейлер" @tap="filmTrailer()" />
        </StackLayout>
    </ScrollView>
</template>
<script>
import { action, Http } from '@nativescript/core';
import KinopoiskParam from './KinopoiskChooseParam.vue';
//import Embed from 'v-video-embed';
export default {
    data: function() {
        return{
            token: "06e18309-a747-43c7-9ad4-f9db545a52af",
            domen: "https://kinopoiskapiunofficial.tech/",
            data: null,
            genres: [],
            genres_id: [],
            genre: '',
            genre_id: 0,
            countries_id: [],
            countries: [],
            country: '',
            country_id: 0,
            type: '',
            min_rating: 0,
            max_rating: 10,
            films: [],
            pagesCount: null,
            pages: 1,
            film_active: false,

            id: 0,
            name: '',
            image: '',
            year: '-',
            film_length: '-',
            description: '-',
            film_country: [],
            film_genre: [],
            rating: '-',
            rating_age_limit: '-',

            film_trailer: null,
            trailer: [],
        }
    },
    methods: {
        settings() {
            this.$showModal(KinopoiskParam, { props: {
                all_genres_id: this.genres_id,
                all_genres: this.genres,
                all_countries_id: this.countries_id,
                all_countries: this.countries
            }}).then(data => {
                data.forEach(el => {
                    this.genre_id = el.genre_id
                    this.country_id = el.country_id
                    this.type = el.type
                    this.min_rating = el.min_rating
                    this.max_rating = el.max_rating
                });
            })
        },

        // Найтми фильмы
        searchFilms(nextPage){
            if(nextPage != 1)
                this.pages = 1
            this.films = []
            Http.request({
                url: this.domen + 'api/v2.1/films/search-by-filters?country=' + this.country_id 
                                        + '&genre=' + this.genre_id 
                                        + '&type=' + this.type 
                                        + '&ratingFrom=' + this.min_rating 
                                        + '&ratingTo=' + this.max_rating
                                        + '&page=' + this.pages,
                method: 'GET',
                headers: {"X-API-KEY": this.token}
            }).then(
                (responce) => {
                    this.data = responce.content.toJSON();
                    this.pagesCount = this.data.pagesCount
                    this.data.films.forEach(el => {
                        this.films.push({
                            id: el.filmId,
                            image: el.posterUrl,
                            name: el.nameRu
                        })
                    });
                }
            ).catch(e => {
                console.log(e)
            })
        },

        //Для счетчика страниц, чтобы передавать в request
        pagination(nextOrBackPage) {
            if (nextOrBackPage == 1 && this.pages >= 1){
                this.pages -= 1
            } else {
                this.pages += 1
            }
            this.searchFilms(1)
        },

        //Вывод информации о фильме
        filmInformation(film_id){
            this.film_active = true
            this.id = film_id
            Http.request({
                url: this.domen + 'api/v2.1/films/' + film_id,
                method: 'GET',
                headers: {"X-API-KEY": this.token}
                
            }).then(
                (responce) => {
                    console.log(responce)
                    this.data = responce.content.toJSON();
                    this.name = this.data.data.nameRu
                    this.image = this.data.data.posterUrl
                    this.year = this.data.data.year
                    this.film_length = this.data.data.filmLength
                    this.description = this.data.data.description
                    this.data.data.countries.forEach(el => {
                        this.country.push(el.country)
                    });
                    this.data.data.genres.forEach(el => {
                        this.genre.push(el.genre)
                    });
                    this.rating = this.data.data.rating.rating
                    this.rating_age_limit = this.data.data.ratingAgeLimits
                    this.country = this.data.data.countries
                }
            )
        },

        // Назад
        closeInformationFilm(){
            this.film_active = false
        },

        // Трейлер фильма
        filmTrailer(){
            Http.request({
                url: this.domen + 'api/v2.1/films/' + this.id + '/videos',
                method: 'GET',
                headers: {"X-API-KEY": this.token}
            }).then(
                (responce) => {
                    this.film_trailer = responce.content.toJSON();
                    this.film_trailer.trailers.forEach(el => {
                        this.trailer.push(el.url)
                    });

                    console.log(this.trailer)

                }
            ).catch( e => {
                //action сделать чтобы говорил что нету трейлеров
            })
        }
    },
    mounted() {
        Http.request({
            url: this.domen + 'api/v2.1/films/filters' ,
            method: 'GET',
            headers: {"X-API-KEY": this.token}
        }).then(
            (responce) => {
                this.data = responce.content.toJSON();
                console.log(this.data)
                this.data.genres.forEach(el => {
                    this.genres_id.push(el.id)
                    this.genres.push(el.genre)
                });
                this.data.countries.forEach(el => {
                    this.countries_id.push(el.id)
                    this.countries.push(el.country)
                });
            }
        ).catch( e => {
            console.log(e)
        })
    }
}
</script>
<style scoped>

    TextField {
        color: white;
        font-size: 16;
        background-color: black;
        border-radius: 10;
        padding-left: 10;
        outline: none;
    }

    Label {
        color:black;
        font-size: 16;   
    }

    Button {
        border-radius: 10;
        background-color: black ;
        color: white;
        font-size: 16;
    }

    .param__btn {
        width: 100;
        height: 50;
    }

    .btn__films {
        background-color: black;
        color: yellow;
        border: none;
    }

    .navigation__btn{
        width: 100;
        height: 50;
    }

    Image {
        width: 200;
        height: 150;
    }
</style>