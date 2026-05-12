# AI Vision (ia-app)

Una aplicación móvil moderna construida con **Ionic**, **Vue 3** y **TensorFlow.js** para el reconocimiento de objetos en tiempo real.

Este proyecto sigue la metodología **Spec-Driven Development (SDD)** para garantizar una arquitectura sólida y una documentación clara.

## 📄 Documentación del Proceso (SDD)
- **[Especificaciones (Foundations & Specify)](SPEC.md)**: Objetivos, alcance y descripción funcional.
- **[Planificación (Planning)](tasks.md)**: Decisiones técnicas y seguimiento de tareas.

## 🚀 Características

- **Detección en tiempo real**: Utiliza la cámara del dispositivo para identificar objetos instantáneamente.
- **Inteligencia Artificial**: Implementa el modelo `COCO-SSD` de TensorFlow.js (versión lite para rendimiento móvil).
- **Interfaz Premium**: Diseño elegante con efectos de glassmorphism, animaciones suaves y una experiencia de usuario fluida.
- **Feedback Háptico**: Vibraciones sutiles al detectar nuevos objetos para una experiencia inmersiva.
- **Multi-plataforma**: Preparada para funcionar en la web y como aplicación nativa en Android mediante Capacitor.

## 🛠️ Stack Tecnológico

- **Framework**: [Ionic Framework](https://ionicframework.com/) con **Vue 3** (Composition API).
- **Build Tool**: [Vite](https://vitejs.dev/).
- **IA**: [@tensorflow/tfjs](https://www.tensorflow.org/js) y [@tensorflow-models/coco-ssd](https://github.com/tensorflow/tfjs-models/tree/master/coco-ssd).
- **Mobile**: [Capacitor](https://capacitorjs.com/) para el acceso a hardware nativo (Cámara, Hápticos).
- **Lenguaje**: TypeScript.

## 📦 Instalación

1. Clona el repositorio:
   ```bash
   git clone https://github.com/aymaaar2/iadevAymar.git
   cd iadevAymar
   ```

2. Instala las dependencias:
   ```bash
   npm install
   ```

## 💻 Desarrollo

Para ejecutar la aplicación en modo desarrollo en tu navegador:

```bash
npm run dev
```

Para construir la aplicación para producción:

```bash
npm run build
```

## 📱 Despliegue en Android

La aplicación ya cuenta con la configuración de Android. Para abrir el proyecto en Android Studio:

```bash
npx cap sync
npx cap open android
```

## 📄 Licencia

Este proyecto está bajo la Licencia MIT.
