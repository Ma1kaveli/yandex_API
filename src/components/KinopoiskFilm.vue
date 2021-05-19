<!-- Возможно получиться переделать чтобы была другая страница 
   -
   - ПОКА ЧТО НЕ РАБОТАЕТ
   - МОЖНО НЕ СМОТРЕТЬ!!!!!!
-->

<template>
    <FlexboxLayout>
        <FlexboxLayout>
            <Button @tap="$navigateBack" />
            <Label v-model="this.name" />
        </FlexboxLayout>
        <FlexboxLayout>
            <FlexboxLayout>
                <Label v-model="this.year" />
                <!--<Label v-for="" :text="'Страна: ' + country" />
                <Label :text="'Жанр: ' + genre" />-->
                <Label v-model="this.film_length" />
                <Label v-model="this.rating" />
                <Label v-model="this.rating_age_limit" />
            </FlexboxLayout>
        </FlexboxLayout>
        <FlexboxLayout>
                <Label v-model="this.description" />
        </FlexboxLayout>
    </FlexboxLayout>
</template>
<script>
import { Http } from '@nativescript/core';
export default {
    props: {
        id: Number
    },
    data: function() {
        return {
            token: "06e18309-a747-43c7-9ad4-f9db545a52af",
            domen: "https://kinopoiskapiunofficial.tech/",
            data: null,
            name: '',
            image: '',
            year: 'Неизвестно',
            film_length: 'Неизвестно',
            description: 'Нету',
            country: [],
            genre: [],
            rating: '-',
            rating_age_limit: '-',
        }
    },
    methods: {

    },
    mounted() {
        Http.request({
                url: this.domen + 'api/v2.1/films/' + this.id,
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

    }
}
</script>
<style scoped>
    Page {
        background-color: yellow;
        color: black;
    }
</style>