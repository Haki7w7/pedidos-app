<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
            <ion-back-button>Back</ion-back-button>
        </ion-buttons>
        <ion-title>Post</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
     
    <ion-card>
    <img alt="Silhouette of mountains" src="https://ionicframework.com/docs/img/demos/card-media.png" />
    <ion-card-header>
      <ion-card-title>{{ post?.title }}</ion-card-title>
      <ion-card-subtitle>Usuario ID: {{ post?.userId }}</ion-card-subtitle>
    </ion-card-header>

    <ion-card-content>
        <p>
            {{ post?.body }}
        </p>
    </ion-card-content>
  </ion-card>
     
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">

import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonCard, IonCardHeader, IonCardTitle, IonCardSubtitle, IonCardContent, IonButtons, IonBackButton } from '@ionic/vue';
import { ref } from 'vue';
import { Posts } from '@/interfaces/posts';

import { useRoute } from 'vue-router';

const route = useRoute();

const post = ref<Posts | null>(null);

function cargarPost(){

  fetch('https://jsonplaceholder.typicode.com/posts/' + route.params.id)
  .then(response => response.json())
  .then(json => {
    console.log(json);
    post.value = json;

  })

}
cargarPost();

</script>
