<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Qual cidade estou?</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class='container'>
          <h2>Coordenadas</h2>
          <h3>{{  latitude  }}</h3>
          <h3>{{  longitude  }}</h3>
          <ion-button @click="buscaCidade()">Qual a cidade?</ion-button>
          <h3>{{  cidade  }}</h3>
      </div> 

      <ion-grid>
        <ion-row>
          <ion-col>
            <ul v-for="(cidade, index) in list" v-bind:key="index">
              <li>{{  cidade  }}</li>
              
            </ul>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonGrid, IonRow, IonCol } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { Http } from '@capacitor-community/http';
import { Storage } from '@ionic/storage'

export default defineComponent({
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonGrid, 
    IonRow,
    IonCol
  },
  data() {
    return {
      localStorage: new Storage(),
      latitude: 0,
      longitude: 0,
      cidade: '',
      list: []
    }
  },
  ionViewWillEnter(){
    this.printCurrentPosition();

  },
  created: async function () {
    await this.localStorage.create();

  },
  mounted: async function() {
    const list = await this.localStorage.get('list');
      if (JSON.parse(list) == null) {
        this.list = [];
      }
      else
        this.list = JSON.parse(list);
  },   

  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      console.log('Current position: ', coordinates);


        this.latitude = coordinates.coords.latitude;
        this.longitude = coordinates.coords.longitude; 
    },
    buscaCidade: async function() {
        const ACCESS_KEY = '9302a50c39e454eeaede83fe75e3f6a1';
        //let url = 'http://api.positionstack.com/v1/reverso?access_key=' + ACCESS_KEY + '&query=' + this.latitude + ',' + this.longitude;


        const options = {
          url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}&limit=1`
        };

        const response = await Http.get(options);
        console.log('Resposta recebida: ', response);


        this.cidade =`${response.data.data[0].locality},${response.data.data[0].county},${response.data.data[0].region_code}` 
        //this.cidade = response.data.data[0].locality + ', ' + response.data.data[0].region_code;

          this.list.push(this.cidade);
          await this.localStorage.set('list', JSON.stringify(this.list));
          console.log(this.list);

    },
    
  }
  
});
</script>

<style scoped>
.container {
  text-align: center;
}
</style>