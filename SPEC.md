# Specification: AI Vision (ia-app)

Este documento detalla la especificación del proyecto siguiendo la metodología **Spec-Driven Development**.

## 1. Foundations

### Contexto
El proyecto nace de la necesidad de demostrar la integración de modelos de Inteligencia Artificial en dispositivos móviles sin dependencia de servidores externos, utilizando procesamiento "on-device".

### Objetivos
- Desarrollar una aplicación móvil híbrida capaz de realizar visión artificial en tiempo real.
- Optimizar el rendimiento del modelo para dispositivos de gama media/baja.
- Proporcionar una interfaz de usuario moderna e intuitiva (Glassmorphism).

### Alcance (Scope)
- **Incluido**: Detección de múltiples categorías de objetos, visualización de bounding boxes, feedback háptico, listado de detecciones recientes.
- **No incluido**: Entrenamiento de modelos personalizados (se usa COCO-SSD pre-entrenado), almacenamiento en la nube, reconocimiento facial.

---

## 2. Specify

### Descripción Funcional
La aplicación permite al usuario apuntar con la cámara a diferentes objetos y recibir una etiqueta visual de qué es el objeto y con qué probabilidad lo ha identificado la IA.

### Casos de Uso
1. **Detección en tiempo real**: Al abrir la app, la cámara se activa automáticamente e inicia el escaneo.
2. **Cambio de Cámara**: El usuario puede alternar entre la cámara trasera y delantera.
3. **Historial de Detecciones**: Un panel inferior muestra los 3 objetos detectados más recientemente con su porcentaje de confianza.

### Comportamiento Esperado
- El modelo debe cargar en menos de 5 segundos.
- Los cuadros delimitadores (bounding boxes) deben seguir al objeto suavemente.
- Si la confianza de detección es superior al 60%, se debe disparar una vibración corta (háptica).

### Requisitos Técnicos
- **Framework**: Ionic + Vue 3.
- **IA**: TensorFlow.js con modelo `lite_mobilenet_v2`.
- **Acceso Hardware**: Plugins de Capacitor para Cámara y Haptics.
