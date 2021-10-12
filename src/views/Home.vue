<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Onde andei?</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content>
      <div class="container">
        <h2>Coordenadas</h2>
        <p>{{ latitude }}, {{ longitude }}</p>
        <ion-button
         @click="saveCurrentCity"
        >
          Marcar Posição
        </ion-button>

        <ion-list v-if="visitedCities && visitedCities.length">
          <ion-card v-for="(city, index) in visitedCities" :key="index">
            <ion-card-header>
              <ion-card-subtitle>{{ city.visitedAt }}</ion-card-subtitle>
              <ion-card-title>{{ city.name }} - {{ city.regionCode }}</ion-card-title>
              <ion-card-subtitle>{{ city.latitude }}, {{ city.longitude }}</ion-card-subtitle>
            </ion-card-header>
          </ion-card>
        </ion-list>
      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonList, IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { Http } from '@capacitor-community/http';
import { Storage } from '@ionic/storage';

interface VisitedCity {
  latitude: number;
  longitude: number;
  name: string;
  regionCode: string;
  visitedAt: string;
}

export default defineComponent({
  name: 'Home',
  
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonList,
    IonCard,
    IonCardHeader, 
    IonCardTitle, 
    IonCardSubtitle
  },
  
  data() {
    return {
      localStorage: new Storage(),
      latitude: 0,
      longitude: 0,
      visitedCities: new Array<VisitedCity>()
    }
  },

  methods: {
    async setCurrentPosition() {
      const coordinates = await Geolocation.getCurrentPosition();
      this.latitude = coordinates.coords.latitude;
      this.longitude = coordinates.coords.longitude;
    },

    async saveCurrentCity() {
      const reverseLocation = await this.searchCity(this.latitude, this.longitude);

      const actualCity: VisitedCity = {
        latitude: this.latitude,
        longitude: this.longitude,  
        name: reverseLocation.locality,
        regionCode: reverseLocation.region_code,
        visitedAt: this.getNormalizedDate()
      };

      this.visitedCities.unshift(actualCity);
      await this.localStorage.set('cities', JSON.stringify(this.visitedCities));
    },

    async searchCity(latitude: number, longitude: number) {
      const response: any = await Http.get({
        url: `http://api.positionstack.com/v1/reverse?access_key=bf9a3b189486a642edc9e6d3cb9259ac&query=${latitude},${longitude}&limit=1`
      });

      return response.data.data[0];
    },

    getNormalizedDate() {
      const [actualDate] = new Date().toLocaleString().split(' ');
      return actualDate.replace(',', '');
    }
  },

  async created() {
    await this.localStorage.create();
  },

  async beforeMount() {
    const savedCities = await this.localStorage.get('cities');
    this.visitedCities = JSON.parse(savedCities) || new Array<VisitedCity>();
  },

  async ionViewWillEnter() {
    this.setCurrentPosition();
  },
});
</script>

<style scoped>
.container {
  text-align: center;
}
</style>