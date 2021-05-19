<template >
    <StackLayout>
        <Button text = "Where do you live?" @tap='chose()'/>
        <StackLayout class='qwerty' orientation="vertical">
            <label class="text">City:</label>
            <Label class='text1'  :text='listOfItems[this.selectedItem]'/>
        
            <label class='text'> Season:</label>
            <Label class='text1'  :text='weather.fact.season'/>
            <label class='text'> Temp:</label>
            <Label class='text1'  :text='weather.fact.temp'/>
            <label class='text'> Wind:</label>
            <Label class='text1'  :text='weather.fact.wind_speed'/>
                <HtmlView html="<hr>" />
        </StackLayout>
    </StackLayout>
</template>

<script >
import { Http } from '@nativescript/core'
import * as ApplicationSettings from "@nativescript/core/application-settings";
  export default {
    data() {
      return {
        listOfItems: [
          { title: "Khanty-Mansiysk",
            toString: () => {
              return 'Khanty-Mansiysk';
            },
            latitude: 61.004,
            longitude: 69.001
          }, 
          { title: "Omsk",
            toString: () => {
              return 'Omsk';
            },
            latitude: 54.992,
            longitude: 73.368
          },
          { title: "Тобольск",
            toString: () => {
              return 'Тобольск';
            },
            latitude: 58.198,
            longitude: 68.256
          },
        ],
        selectedItem: 0,
        weather: {
            fact: {
                temp : 0,
                wind_speed : 0,
                season: 'summer,'
            }
        },
        imagePath: '',
        cities:['Khanty-Mansiysk', 'Omsk', 'Тобольск']
      }
    },
    mounted(){
      if(ApplicationSettings.getString('weather')){
        this.weather.fact=JSON.parse(ApplicationSettings.getString('weather'));
       
      }
      if(ApplicationSettings.getString('chose')){
            this.selectedItem=JSON.parse(ApplicationSettings.getString('chose'));
        }
        else{
            this.chose()
        }
     
    },
        methods:{
            check(){
                Http.request({
                    url: 'https://api.weather.yandex.ru/v2/forecast?limit=1'+'&lat=' + 
                    String(this.listOfItems[this.selectedItem].latitude) + '&lon=' +
                    String(this.listOfItems[this.selectedItem].longitude),
                    method: "GET",
                    headers: {"X-Yandex-API-Key": "2774d3ce-4acb-4d49-8852-4495f84d391b"},
                    })
                    .then(
                    (response) => {
                    this.weather = response.content.toJSON();
                });
            },
            chose(){
                action("Ur choice", "exit", this.cities)
                .then(result => {
                this.selectedItem = this.cities.indexOf(result);
                ApplicationSettings.setString('chose', JSON.stringify(this.selectedItem));        
                this.check();    
            });
        }
    }
}
</script>

<style>
    .city{
        font-size: 30;
        text-align: center;
        width: 100%;
        background-color: #ffffff;
    }
    .pogoda{
        background-color: #ffffff;
    }

    .text{
        padding: 20px;
        color : #7297d3;
    }

    .text1{
        margin: 50px auto;
    
        font-weight: 900;
        padding: 40px;
    }
    .qwerty{
        font-size: 22;
        padding: 20px;
        padding-bottom: 270px;
        background-color: #ffffff;
        border-radius: 10%;
    }
</style>