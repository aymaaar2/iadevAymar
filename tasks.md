# Tasks: AI Vision (ia-app)

Planificación y seguimiento de tareas siguiendo la metodología **Spec-Driven Development**.

## 3. Planning

### Decisiones Técnicas
- **Frontend**: Vue 3 con Composition API para una mejor gestión del estado del modelo de IA.
- **Modelo de IA**: `lite_mobilenet_v2` por su equilibrio entre precisión y velocidad en navegadores móviles.
- **Rendering**: Uso de `<canvas>` sobrepuesto al elemento `<video>` para dibujar las detecciones sin afectar el rendimiento del stream de video.

### Lista de Tareas

#### Fase 1: Configuración del Entorno [x]
- [x] Inicializar proyecto Ionic con Vue.
- [x] Instalar dependencias de TensorFlow.js (`@tensorflow/tfjs`, `@tensorflow-models/coco-ssd`).
- [x] Configurar Capacitor para soporte nativo Android.

#### Fase 2: Implementación de Cámara [x]
- [x] Crear componente de video a pantalla completa.
- [x] Implementar lógica de permisos y acceso a `getUserMedia`.
- [x] Añadir funcionalidad de alternar entre cámaras.

#### Fase 3: Integración de IA [x]
- [x] Carga asíncrona del modelo COCO-SSD.
- [x] Lógica de detección frame-by-frame usando `requestAnimationFrame`.
- [x] Implementación de bounding boxes en Canvas.

#### Fase 4: Interfaz y UX [x]
- [x] Diseño de panel inferior (Bottom Sheet) con efecto blur.
- [x] Implementación de feedback háptico con Capacitor Haptics.
- [x] Visualización de scores de confianza con barras de progreso.

#### Fase 5: Entrega y Documentación [/]
- [x] Crear README.md con instrucciones.
- [x] Generar SPEC.md (Foundations & Specify).
- [x] Generar tasks.md (Planning).
- [ ] Generar PDF final para entrega.
