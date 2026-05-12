<template>
  <ion-page>
    <ion-header class="ion-no-border" :translucent="true">
      <ion-toolbar class="transparent-toolbar">
        <ion-title class="main-title">AI Vision</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="toggleCamera">
            <ion-icon :icon="cameraReverseOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="no-scroll">
      <div class="camera-container">
        <!-- Video element for the camera feed -->
        <video 
          ref="videoElement" 
          class="camera-video" 
          autoplay 
          playsinline 
          muted>
        </video>
        
        <!-- Canvas element for bounding boxes -->
        <canvas 
          ref="canvasElement" 
          class="detection-canvas">
        </canvas>

        <!-- Loading overlay -->
        <div v-if="isLoading" class="loading-overlay">
          <ion-spinner name="crescent" color="primary"></ion-spinner>
          <p>Carregant model IA...</p>
        </div>
      </div>

      <!-- Bottom Sheet for Detections (Glassmorphism) -->
      <div class="detections-sheet">
        <div class="sheet-handle"></div>
        <h3 class="sheet-title">Deteccions Recents</h3>
        
        <div class="detections-list">
          <div v-if="latestDetections.length === 0" class="empty-state">
            No s'han detectat objectes.
          </div>
          <div 
            v-for="(detection, index) in latestDetections" 
            :key="index"
            class="detection-item"
          >
            <div class="detection-info">
              <span class="detection-class">{{ capitalize(detection.class) }}</span>
              <span class="detection-score">{{ Math.round(detection.score * 100) }}%</span>
            </div>
            <div class="progress-bar-container">
              <div class="progress-bar" :style="{ width: `${detection.score * 100}%`, backgroundColor: getColor(detection.score) }"></div>
            </div>
          </div>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { 
  IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonSpinner,
  IonButtons, IonButton, IonIcon 
} from '@ionic/vue';
import { cameraReverseOutline } from 'ionicons/icons';
import '@tensorflow/tfjs';
import * as cocoSsd from '@tensorflow-models/coco-ssd';
import { Haptics, ImpactStyle } from '@capacitor/haptics';

const videoElement = ref<HTMLVideoElement | null>(null);
const canvasElement = ref<HTMLCanvasElement | null>(null);
const isLoading = ref(true);
const latestDetections = ref<cocoSsd.DetectedObject[]>([]);
let model: cocoSsd.ObjectDetection | null = null;
let animationFrameId: number;
let stream: MediaStream | null = null;
let facingMode = 'environment';
let lastHapticTime = 0;

const loadModel = async () => {
  try {
    isLoading.value = true;
    model = await cocoSsd.load({ base: 'lite_mobilenet_v2' });
    console.log('Model carregat!');
    isLoading.value = false;
  } catch (error) {
    console.error('Error loading model:', error);
    isLoading.value = false;
  }
};

const setupCamera = async () => {
  if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
    try {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
      }
      stream = await navigator.mediaDevices.getUserMedia({
        audio: false,
        video: {
          facingMode: facingMode,
          width: { ideal: window.innerWidth },
          height: { ideal: window.innerHeight }
        }
      });
      if (videoElement.value) {
        videoElement.value.srcObject = stream;
        return new Promise<void>((resolve) => {
          videoElement.value!.onloadedmetadata = () => {
            resolve();
          };
        });
      }
    } catch (error) {
      console.error('Error accessing camera:', error);
    }
  }
};

const toggleCamera = async () => {
  facingMode = facingMode === 'environment' ? 'user' : 'environment';
  await setupCamera();
};

const detectFrame = async () => {
  if (model && videoElement.value && canvasElement.value && videoElement.value.readyState === 4) {
    const video = videoElement.value;
    const canvas = canvasElement.value;
    const context = canvas.getContext('2d');

    if (context) {
      // Sync canvas dimensions
      if (canvas.width !== video.videoWidth || canvas.height !== video.videoHeight) {
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
      }

      const predictions = await model.detect(video);
      
      // Update UI
      latestDetections.value = predictions.slice(0, 3); // Keep top 3

      // Haptic feedback logic
      if (predictions.length > 0) {
        const now = Date.now();
        if (now - lastHapticTime > 2000) { // Limit haptic to every 2 seconds
          try {
            await Haptics.impact({ style: ImpactStyle.Light });
            lastHapticTime = now;
          } catch (e) {
            // Haptics might fail in browser, ignore
          }
        }
      }

      // Draw bounding boxes
      context.clearRect(0, 0, canvas.width, canvas.height);
      predictions.forEach(prediction => {
        const [x, y, width, height] = prediction.bbox;
        const color = getColor(prediction.score);
        
        // Draw box
        context.strokeStyle = color;
        context.lineWidth = 4;
        context.strokeRect(x, y, width, height);

        // Draw label background
        context.fillStyle = color;
        const text = `${prediction.class} (${Math.round(prediction.score * 100)}%)`;
        context.font = '16px Inter, Roboto, sans-serif';
        const textWidth = context.measureText(text).width;
        context.fillRect(x, y - 24, textWidth + 10, 24);

        // Draw label text
        context.fillStyle = '#FFFFFF';
        context.fillText(text, x + 5, y - 7);
      });
    }
  }
  animationFrameId = requestAnimationFrame(detectFrame);
};

const getColor = (score: number) => {
  if (score > 0.8) return '#10b981'; // Emerald
  if (score > 0.6) return '#f59e0b'; // Amber
  return '#ef4444'; // Red
};

const capitalize = (str: string) => {
  return str.charAt(0).toUpperCase() + str.slice(1);
};

onMounted(async () => {
  await setupCamera();
  await loadModel();
  if (videoElement.value) {
    videoElement.value.play();
    detectFrame();
  }
});

onUnmounted(() => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
  }
  if (stream) {
    stream.getTracks().forEach(track => track.stop());
  }
});
</script>

<style scoped>
.no-scroll {
  --overflow: hidden;
}

.transparent-toolbar {
  --background: rgba(0, 0, 0, 0.4);
  --color: white;
  backdrop-filter: blur(10px);
  position: absolute;
  top: 0;
  width: 100%;
  z-index: 10;
}

.main-title {
  font-weight: 700;
  letter-spacing: 1px;
}

.camera-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  background-color: #000;
  overflow: hidden;
}

.camera-video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.detection-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  pointer-events: none; /* Let touches pass through */
}

.loading-overlay {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(0, 0, 0, 0.7);
  padding: 24px;
  border-radius: 16px;
  backdrop-filter: blur(8px);
  color: white;
}

.loading-overlay p {
  margin-top: 16px;
  font-weight: 600;
  font-size: 16px;
}

.detections-sheet {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(30, 30, 30, 0.85);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-top-left-radius: 24px;
  border-top-right-radius: 24px;
  padding: 16px 24px 32px 24px;
  color: white;
  box-shadow: 0 -4px 24px rgba(0, 0, 0, 0.5);
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  transition: transform 0.3s ease;
}

.sheet-handle {
  width: 40px;
  height: 5px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
  margin: 0 auto 16px auto;
}

.sheet-title {
  font-size: 18px;
  font-weight: 700;
  margin: 0 0 16px 0;
  text-align: left;
}

.detections-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.empty-state {
  text-align: center;
  color: #a3a3a3;
  font-style: italic;
  padding: 12px 0;
}

.detection-item {
  background: rgba(255, 255, 255, 0.05);
  padding: 12px;
  border-radius: 12px;
}

.detection-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-weight: 600;
}

.detection-class {
  color: #f3f4f6;
}

.detection-score {
  color: #9ca3af;
}

.progress-bar-container {
  width: 100%;
  height: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  border-radius: 3px;
  transition: width 0.2s ease, background-color 0.2s ease;
}
</style>
