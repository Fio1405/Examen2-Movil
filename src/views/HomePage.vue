<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Escáner de códigos QR</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="ion-padding">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Escáner de códigos QR</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-card>
        <ion-card-header>
          <ion-card-title>Escanear un código QR</ion-card-title>
        </ion-card-header>

        <ion-card-content>
          <p>Presiona el botón para escanear un código QR</p>
          <ion-button expand="block" @click="scan">Escanear QR</ion-button>
          <ion-button expand="block" color="danger" @click="clearHistory">Limpiar Historial</ion-button>
        </ion-card-content>
      </ion-card>

      <ion-card>
        <ion-card-header>
          <ion-card-title>Historial de Escaneos</ion-card-title>
        </ion-card-header>

        <ion-card-content>
          <ion-list>
            <ion-item v-for="(item, index) in historial" :key="index" @click="handleItemClick(item)">
              <ion-label>{{ item }}</ion-label>
            </ion-item>
          </ion-list>
          <ion-text color="medium" v-if="historial.length === 0">
            No hay escaneos aún
          </ion-text>
        </ion-card-content>
      </ion-card>

      <ion-alert
        :is-open="showAlert"
        header="Resultado del QR"
        :message="alertMessage"
        :buttons="['OK']"
        @didDismiss="showAlert = false"
      />
    </ion-content>
  </ion-page>
</template>


<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonList, IonItem, IonAlert } from '@ionic/vue';
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner';
import { ref } from 'vue';

const historial = ref<string[]>([]);
const showAlert = ref(false);
const alertMessage = ref('');


const scan = async () => {
  try {
    await CapacitorBarcodeScanner.scanBarcode({ hint: 17 });
    const result = await CapacitorBarcodeScanner.scanBarcode({hint: 17});
    if (result.ScanResult) {
      historial.value.push(result.ScanResult);
      handleScanResult(result.ScanResult);
    }
  } catch (error) {
    console.error('Error al escanear:', error);
  }
};

const handleScanResult = (content: string) => {
  if (content.startsWith('http://') || content.startsWith('https://')) {
    window.open(content, '_blank');
  } else if (content.includes('@') && content.includes('.')) {
    window.location.href = `mailto:${content}`;
  } else {
    alertMessage.value = ` Contenido del QR: ${content}`;
    showAlert.value = true;
    
  }
};

const handleItemClick = (item: string) => {
  handleScanResult(item);
};

const clearHistory = () => {
  historial.value = [];
};
</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 30%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#history {
  margin-top: 20px;
  padding: 0 16px;
}

#history h2 {
  font-size: 18px;
  margin-bottom: 10px;
}
</style>