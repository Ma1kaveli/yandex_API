<template>
    <Page backgroundColor="yellow" >
        <ActionBar title="Яндекс.Расписание" backgroundColor="black" />
        <ScrollView orientation="vertical">
            <StackLayout>
                    <ActivityIndicator v-if="!this.ready" busy="true" color="black" />
                    <Label class="load" v-if="!this.ready" text="Подождите идет загрузка..." />
                    <Label class="data" v-if="this.ready" text="Напишите число отправки этого месяца:" />
                    <TextField v-if="this.ready" v-model="except_day" />
                    <Button v-if="this.ready" text="Выберите город отправки: " @tap="chooseCity(1)" />
                    <Label class="data" v-if="this.ready" v-model="place_of_dispatch" />
                    <Button v-if="this.ready" text="Выберите город прибытия: " @tap="chooseCity(2)" />
                    <Label class="data" v-if="this.ready" v-model="place_of_arrival" />
                    <Button v-if="this.ready" text="Поиск" @tap="validaty()" />
                    <Label v-model="total" />
                    <ListView height="300" for="el in information">
                        <v-template>
                            <GridLayout columns="100%, 100%, 100%, 100%">
                                <Label col="0">{{el.date}}</Label>
                                <Label col="1">{{el.full_title}}</Label>
                                <Label col="2">{{el.from_tittle}}</Label>
                                <Label col="3">{{el.from_transport_type}}</Label>
                            </GridLayout>
                        </v-template>
                    </ListView>
            </StackLayout>
        </ScrollView>
    </Page>
</template>

<script>
import { Http } from '@nativescript/core'
export default {
    data: function() {
        return {
            token: '977939bf-e119-40e4-941c-b92eb7fb6a43',
            ready: false,
            data: null,
            citys_name: [],
            citys_yandex_code: [],
            now: '',
            except_day: '',
            place_of_dispatch: '-',
            place_of_arrival: '-',
            city_dispatch_code: '',
            city_arrival_code: '',
            total: '',
            information: []
        }
    },

    methods: {
        chooseCity(direction){
            if(this.citys_name.length != 0){
                action("Выберите город", "Отмена", this.citys_name).then( result => {
                    if (result != 'Отмена'){
                        if(direction == 1)
                            this.place_of_dispatch = result
                        else
                            this.place_of_arrival = result
                    }
                })
            }
        },
        

        getInformation() {
            this.information = []
            Http.request({
                url: 'https://api.rasp.yandex.net/v3.0/search/?apikey=' + this.token + '&format=json&from=' + this.city_dispatch_code +'&to=' + this.city_arrival_code + '&lang=ru_RU&page=1&date=' + this.now,
                method: 'GET',
            }).then(
                (responce) => {
                    this.data = responce.content.toJSON();
                    if (this.data.pagination.total != 0){
                        this.total = 'Найдено: ' + this.data.pagination.total
                        this.data.segments.forEach(el => {
                            if(el.from.transport_type == 'bus'){
                                el.from.transport_type = 'автобус'
                            } else if (el.from.transport_type == 'plane'){
                                el.from.transport_type = 'самолет'
                            } else if (el.from.transport_type == 'water'){
                                el.from.transport_type = 'лодка'
                            } else if (el.from.transport_type == 'train'){
                                el.from.transport_type = 'поезд'
                            }
                            this.information.push({
                                date: el.arrival,
                                full_title: el.thread.title,
                                from_tittle: el.from.title,
                                from_transport_type: el.from.transport_type
                            })
                            console.log(this.information)
                        });
                    } else {
                        this.total = 'По запросу ничего не найдено'
                    }
                }
            ).catch( e => {
                console.log(e)
                this.total = 'По запросу ничего не найдено'
            })
        },
        
        

        validaty() {
            this.city_dispatch_code = this.citys_yandex_code[this.citys_name.indexOf(this.place_of_dispatch, 0)]
            this.city_arrival_code = this.citys_yandex_code[this.citys_name.indexOf(this.place_of_arrival, 0)]
            if (this.except_day != '' && this.place_of_dispatch != '-' && this.place_of_arrival != '-'){
                var nowDate = new Date()
                if (this.except_day >= nowDate.getDate() && this.except_day <= 31) {
                        this.now = nowDate.getFullYear() + '-05-' + this.except_day
                        this.getInformation()
                } else {
                    alert({
                        title: "Ошибка",
                        message: "Данные введены не коректно, попробуйте еще раз"
                    })
                }
            } else {
                alert({
                    title: "Ошибка",
                    message: "Проверьте, что вы ввели все данные"
                })
            }
        }
    },

    mounted() {
        Http.request({
                url: 'https://api.rasp.yandex.net/v3.0/stations_list/?apikey=' + this.token + '&lang=ru_RU&format=json',
                method: 'GET',
        }).then(
            (responce) => {
                this.data = responce.content.toJSON();
                this.data = this.data.countries
                this.data.forEach(el => {
                    if(el.title == 'Россия'){
                        this.data = el
                        el.regions.forEach(elem => {
                            if(elem.title == 'Ханты-Мансийский автономный округ'){
                                this.data = elem
                                elem.settlements.forEach(element => {
                                    if(element.title != null && element.codes.yandex_code != undefined){
                                        this.citys_name.push(element.title);
                                        this.citys_yandex_code.push(element.codes.yandex_code)
                                    }
                                });
                            }
                        });
                    }
                });
                console.log('Good')
                this.ready = true
            }
        ).catch( e => {
            console.log(e)
        })
    }
}
</script>
<style scoped>
    Page {
        color:aliceblue;
    }

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

    .load {
        text-align: center;
    }

    .data {
        margin-left: 25;
    }

    Button {
        border-radius: 10;
        background-color: black ;
        color: white;
        font-size: 16;
    }

    ListView Label {
        font-size: 12;
    }
</style>