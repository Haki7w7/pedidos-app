<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Crear Post</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <div class="ion-padding-horizontal ion-padding-top">
        <ion-button expand="block" color="secondary" type="button" @click="escanearQR">
          <ion-icon slot="start" :icon="qrCodeOutline" />
          Escanear QR
        </ion-button>
      </div>

      <form @submit.prevent="crearPost">
        <ion-list>
          <ion-item v-if="codigoQR">
            <ion-label>
              <h2>Código QR escaneado</h2>
              <p>{{ codigoQR }}</p>
            </ion-label>
          </ion-item>

          <ion-item>
            <ion-input
              label="Title"
              label-placement="stacked"
              name="title"
              v-model="title"
              placeholder="Título del post"
              required
            ></ion-input>
          </ion-item>

          <ion-item>
            <ion-textarea
              label="Body"
              label-placement="stacked"
              name="body"
              v-model="body"
              placeholder="Contenido del post"
              required
            ></ion-textarea>
          </ion-item>

          <ion-item>
            <ion-select
              label="Usuario"
              label-placement="stacked"
              name="userId"
              v-model="userId"
              placeholder="Seleccione un usuario"
            >
              <ion-select-option
                v-for="usuario in usuarios"
                :key="usuario.id"
                :value="usuario.id"
              >
                {{ usuario.name }}
              </ion-select-option>
            </ion-select>
          </ion-item>
        </ion-list>

        <div class="ion-padding">
          <ion-button expand="block" type="submit" :disabled="enviando">
            Crear Post
          </ion-button>
        </div>
      </form>

      <ion-list v-if="postCreado">
        <ion-list-header>
          <ion-label>Datos del Create (Respuesta)</ion-label>
        </ion-list-header>
        <ion-item>
          <ion-label>
            <h2>Title</h2>
            <p>{{ postCreado.title }}</p>
          </ion-label>
        </ion-item>
        <ion-item>
          <ion-label>
            <h2>Body</h2>
            <p>{{ postCreado.body }}</p>
          </ion-label>
        </ion-item>
        <ion-item>
          <ion-label>
            <h2>User ID</h2>
            <p>{{ postCreado.userId }}</p>
          </ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonList,
  IonListHeader,
  IonItem,
  IonLabel,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption,
  IonButton,
  IonIcon,
} from '@ionic/vue';
import { qrCodeOutline } from 'ionicons/icons';
import {
  CapacitorBarcodeScanner,
  CapacitorBarcodeScannerTypeHint,
} from '@capacitor/barcode-scanner';
import { Usuarios } from '@/interfaces/usuarios';
import { Posts } from '@/interfaces/posts';

const usuarios = ref<Usuarios[]>([]);
const title = ref('foo');
const body = ref('bar');
const userId = ref(1);
const codigoQR = ref('');
const postCreado = ref<Posts | null>(null);
const enviando = ref(false);

async function escanearQR() {
  const result = await CapacitorBarcodeScanner.scanBarcode({
    hint: CapacitorBarcodeScannerTypeHint.ALL,
  });

  if (result.ScanResult) {
    const texto = result.ScanResult.trim();
    codigoQR.value = texto;
    try {
      const data = JSON.parse(texto);
      if (data && typeof data === 'object') {
        if ((data.title ?? data.Title) !== undefined) {
          title.value = String(data.title ?? data.Title);
        }
        if ((data.body ?? data.Body) !== undefined) {
          body.value = String(data.body ?? data.Body);
        }
        if ((data.userId ?? data.user ?? data.User) !== undefined) {
          userId.value = Number(data.userId ?? data.user ?? data.User);
        }
        return;
      }
    } catch {
      // No es JSON válido, intentamos extraer formato clave: valor (title:, Body:, User:)
    }

    const titleMatch = texto.match(/title\s*:\s*([^\n\r]+?)(?=\s+(?:body|user(?:id)?)\s*:|$)/i);
    const bodyMatch = texto.match(/body\s*:\s*([^\n\r]+?)(?=\s+(?:title|user(?:id)?)\s*:|$)/i);
    const userMatch = texto.match(/user(?:id)?\s*:\s*(\d+)/i);

    if (titleMatch || bodyMatch || userMatch) {
      if (titleMatch) title.value = titleMatch[1].trim();
      if (bodyMatch) body.value = bodyMatch[1].trim();
      if (userMatch) userId.value = Number(userMatch[1]);
      return;
    }

    const partes = texto.split('|');
    if (partes.length >= 2) {
      title.value = partes[0].trim();
      body.value = partes[1].trim();
      if (partes[2] && !isNaN(Number(partes[2]))) {
        userId.value = Number(partes[2].trim());
      }
    } else {
      body.value = texto;
    }
  }
}

function cargarUsuarios() {
  fetch('https://jsonplaceholder.typicode.com/users')
    .then(response => response.json())
    .then(json => {
      console.log(json);
      usuarios.value = json;
    });
}

function crearPost() {
  enviando.value = true;
  fetch('https://jsonplaceholder.typicode.com/posts', {
    method: 'POST',
    body: JSON.stringify({
      title: title.value,
      body: body.value,
      userId: userId.value,
    }),
    headers: {
      'Content-type': 'application/json; charset=UTF-8',
    },
  })
    .then(response => response.json())
    .then(json => {
      console.log(json);
      postCreado.value = json;
    })
    .finally(() => {
      enviando.value = false;
    });
}

cargarUsuarios();
crearPost();
</script>
