<template>
    <Page backgroundColor="yellow">
        <ActionBar title="Выбрать" backgroundColor="black" />
        <StackLayout>
            <FlexboxLayout justifyContent="center">
                <Button text="Жанр" @tap="chooseGenre()" />
                <Button text="Cтрана" @tap="chooseCountry()" />
                <Button text="Тип" @tap="chooseType()" />
            </FlexboxLayout>
            <FlexboxLayout justifyContent="flex-start" flexDirection="column">
                <Label :text=" 'Жанр: ' + this.genre" />
                <Label :text=" 'Страна: ' +this.country" />
                <Label :text=" 'Тип: ' + this.type_for_modal_page" />
                <Label text="Рейтинг:" />
            </FlexboxLayout>
            <FlexboxLayout justifyContent="flex-start">
                <Button class="btn__rating" :text="min_rating" @tap="rating(1)" />
                <Button class="btn__rating" :text="max_rating" @tap="rating(2)" />
            </FlexboxLayout>
            <Button text="Выбрать" @tap="close()" />
        </StackLayout>
    </Page>
</template>
<script>
export default {
    props: {
        all_genres_id: Array,
        all_genres: Array,
        all_countries_id: Array,
        all_countries: Array,
    },
    data: function() {
        return{
            data: [],
            genres_id: [],
            genre: '',
            genre_id: 0,
            countries_id: [],
            country: '',
            country_id: 0,
            type: 'ALL',
            type_for_modal_page: 'Все',
            min_rating: 0,
            max_rating: 10,
        }
    },
    methods: {
        // Выбор жанра
        chooseGenre(){
            action("Выберите жанр", "Отмена", this.all_genres).then( result => {
                if (result != 'Отмена'){
                    this.genre = result
                    this.genre_id = this.genres_id[this.all_genres.indexOf(this.genre, 0)]
                }
            })
        },

        // Выбор страны
        chooseCountry(){
            action("Выберите страну", "Отмена", this.all_countries).then( result => {
                if (result != 'Отмена'){
                    this.country = result
                    this.country_id = this.countries_id[this.all_countries.indexOf(this.country, 0)]
                }
            })
        },

        //Выбор типа
        chooseType(){
            action("Выберите тип", "Отмена", ["Фильм", "Шоу", "Все"]).then( result => {
                if (result != 'Отмена'){
                    if(result == "Фильмы"){
                        this.type_for_modal_page = 'Фильмы'
                        this.type = 'FILM'
                    }
                    else if(result == "Шоу"){
                        this.type = 'TV_SHOW'
                        this.type_for_modal_page = 'Шоу'
                    }
                    else{
                        this.type = 'ALL'
                        this.type_for_modal_page = 'Все'
                    }
                }
                console.log(this.type)
            })
        },

        //Выбор рейтинга
        rating(minOrMaxRating){
            action("Выберите число", "Отмена", ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]).then(result => {
                if(result != 'Отмена'){
                    if(minOrMaxRating == 1)
                        this.min_rating = result
                    else
                        this.max_rating = result
                }
            })
        },
        //Закрыть модальное окно
        close() {
            this.data.push({
                genre_id: this.genre_id,
                country_id: this.country_id,
                type: this.type,
                min_rating: this.min_rating,
                max_rating: this.max_rating
            })
            this.$modal.close(this.data)
        }
    },

    mounted() {
        this.genres_id = this.all_genres_id  
        this.countries_id = this.all_countries_id
    }
}
</script>
<style scoped>

    Button {
        border-radius: 10;
        background-color: black ;
        color: white;
        font-size: 14;
        height: 50;
        width: 75;
    }

    .btn__rating {
        width: 50;
        height: 30;
    }

    Label {
        font-size: 12;
    }
</style>