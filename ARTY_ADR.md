# Architecture Decision Records (ADR) - Plataforma Arty

**Proyecto**: Arty - Ecosistema Global del Arte Digital y Físico  
**Integrantes**: Angel Quishpe, Alan Rivera, Jorge Escobar  
**Fecha de creación**: 2025-10-14  
**Equipo**: Arquitectura de Software

---

## Índice de Decisiones

1. [ADR-001: Arquitectura Backend Monolítica](#adr-001-arquitectura-backend-monolítica)
2. [ADR-002: Base de Datos Única PostgreSQL](#adr-002-base-de-datos-única-postgresql)
3. [ADR-003: Aplicaciones Frontend Web y Móvil Separadas](#adr-003-aplicaciones-frontend-web-y-móvil-separadas)
4. [ADR-004: Almacenamiento Externo para Imágenes de Alta Resolución](#adr-004-almacenamiento-externo-para-imágenes-de-alta-resolución)
5. [ADR-005: Integración con Blockchain para Certificación](#adr-005-integración-con-blockchain-para-certificación)
6. [ADR-006: Sistema de Subastas en Tiempo Real](#adr-006-sistema-de-subastas-en-tiempo-real)

---

## ADR-001: Arquitectura Backend Monolítica

### Estado
**Aceptado** - 2025-10-14

### Contextoew
Para el lanzamiento inicial de Arty, necesitamos decidir la arquitectura del backend que soportará toda la plataforma.

**Funcionalidades a implementar:**
- Gestión de usuarios (artistas y compradores)
- Catálogo de obras de arte
- Sistema de subastas
- Transacciones y pagos
- Galerías virtuales personalizadas
- Sistema de valoración y recomendaciones

**Factores considerados:**
- Equipo de desarrollo pequeño (3 personas)
- MVP con lanzamiento rápido
- Presupuesto limitado inicial
- Necesidad de validar el modelo de negocio
- Sin experiencia previa del equipo en microservicios

### Decisión
Implementar una **arquitectura backend monolítica** que concentre toda la lógica de negocio en una única aplicación servidor.

**Estructura modular interna:**
- Módulo de Usuarios y Autenticación
- Módulo de Obras y Catálogo
- Módulo de Subastas
- Módulo de Transacciones
- Módulo de Galerías
- Módulo de Notificaciones

### Consecuencias

**Positivas:**
- Desarrollo más rápido (tiempo al mercado reducido)
- Deployment simplificado (un solo artefacto)
- Debugging más fácil (código centralizado)
- Testing más simple sin dependencias distribuidas
- Menor complejidad operacional
- Menor costo de infraestructura
- Transacciones ACID entre módulos sin complejidad adicional
- Ideal para validar el producto con usuarios reales

**Negativas:**
- Escalabilidad limitada (todo escala junto)
- Potencial acoplamiento entre módulos
- Actualizaciones requieren reiniciar toda la aplicación
- Riesgo de código espagueti si no se mantiene disciplina
- Más difícil migrar a microservicios después
- Punto único de falla

**Mitigaciones:**
- Mantener arquitectura modular con separación clara de responsabilidades
- Aplicar principios SOLID y patrones de diseño
- Code reviews obligatorios para mantener calidad
- Documentación de interfaces entre módulos
- Plan de migración a microservicios cuando se alcancen 10,000 usuarios activos
- Monitoreo de performance por módulo para identificar cuellos de botella

### Alternativas Rechazadas

**Opción A: Arquitectura de Microservicios**
- Rechazada por: complejidad excesiva para equipo pequeño, mayor tiempo de desarrollo (3-6 meses más), costos de infraestructura 3x superiores

**Opción B: Serverless (AWS Lambda/Firebase Functions)**
- Rechazada por: vendor lock-in, cold start latency afecta experiencia de usuario, costos impredecibles en fase inicial

---

## ADR-002: Base de Datos Única PostgreSQL

### Estado
**Aceptado** - 2025-10-14

### Contexto
Necesitamos decidir qué base de datos usar para almacenar toda la información de la plataforma.

**Datos a almacenar:**
- Perfiles de artistas y compradores
- Catálogo de obras (metadata, precios, descripción)
- Información de subastas (ofertas, pujas, historial)
- Transacciones y ventas
- Galerías personalizadas
- Sistema de likes y favoritos
- Certificados de autenticidad

**Requisitos:**
- Consistencia de datos (transacciones críticas)
- Relaciones complejas entre entidades
- Búsquedas y filtrados avanzados
- Escalabilidad razonable

### Decisión
Utilizar **PostgreSQL** como base de datos única para toda la plataforma.

**Esquema incluye:**
- Tabla de usuarios (artistas y compradores)
- Tabla de obras de arte
- Tabla de subastas
- Tabla de ofertas/pujas
- Tabla de transacciones
- Tabla de galerías
- Tabla de certificados
- Tabla de notificaciones

### Consecuencias

**Positivas:**
- Transacciones ACID garantizan consistencia
- JOINs nativos para consultas complejas (ej: obras por artista en subasta)
- Full-text search integrado para búsquedas de obras
- Soporte JSON para datos flexibles (metadata de obras)
- PostgreSQL es open source y gratuito
- Excelente rendimiento hasta millones de registros
- Comunidad grande y documentación exhaustiva
- Backup y recovery bien establecidos
- Simplicidad operacional (un solo servidor de BD)

**Negativas:**
- Escalabilidad vertical limitada (un solo servidor)
- No optimizado para casos específicos (ej: búsqueda semántica avanzada)
- Esquema rígido requiere migraciones planificadas
- Potencial cuello de botella en alto tráfico
- Punto único de falla

**Mitigaciones:**
- Índices en columnas críticas (búsquedas por artista, precio, categoría)
- Configurar réplicas de solo lectura para escalar consultas
- Implementar caché (Redis) para consultas frecuentes
- Monitoreo de queries lentas con herramientas como pg_stat_statements
- Particionado de tablas grandes si es necesario
- Backup automático diario con retención de 30 días

### Alternativas Rechazadas

**Opción A: MongoDB (NoSQL)**
- Rechazada por: las relaciones entre usuarios, obras, subastas y transacciones son inherentemente relacionales. SQL es más apropiado.

**Opción B: MySQL**
- Rechazada por: PostgreSQL ofrece mejor soporte para JSON, full-text search más avanzado y cumplimiento más estricto de estándares SQL

**Opción C: Base de datos especializada por dominio**
- Rechazada por: complejidad innecesaria, costos operacionales altos, requiere sincronización entre bases de datos

---

## ADR-003: Aplicaciones Frontend Web y Móvil Separadas

### Estado
**Aceptado** - 2025-10-14

### Contexto
Necesitamos decidir cómo los usuarios accederán a la plataforma Arty.

**Requisitos identificados:**
- Acceso desde computadoras (navegadores)
- Acceso desde smartphones (iOS y Android)
- Experiencia visual rica (galerías de arte)
- Notificaciones en tiempo real para subastas
- Funcionalidades similares en ambas plataformas

**Opciones consideradas:**
- Progressive Web App (PWA) única
- Aplicación web responsive + app nativa
- Solo aplicación web
- Solo aplicación móvil

### Decisión
Desarrollar **dos aplicaciones frontend separadas:**

1. **Aplicación Web (React)**: Para navegadores desktop y tablets
2. **Aplicación Móvil (React Native)**: Para iOS y Android

Ambas consumirán la misma API REST del servidor backend.

### Consecuencias

**Positivas:**
- Mejor experiencia de usuario por plataforma
- App móvil accede a funcionalidades nativas (cámara para subir fotos, notificaciones push)
- Presencia en App Store y Google Play mejora descubrimiento
- React Native permite compartir 70-80% del código entre iOS y Android
- Puede trabajar un desarrollador en web y otro en móvil en paralelo
- Optimizaciones específicas por plataforma
- Notificaciones push nativas para alertas de subastas

**Negativas:**
- Dos código bases diferentes para mantener
- Requiere habilidades de desarrollo móvil
- Proceso de aprobación en las tiendas de apps
- Testing en múltiples plataformas y dispositivos
- Actualizaciones de app móvil requieren aprobación (1-7 días)
- Mayor esfuerzo inicial de desarrollo

**Mitigaciones:**
- Usar React y React Native para compartir conocimiento del equipo
- Design system compartido para consistencia visual
- API única simplifica el desarrollo
- Priorizar lanzamiento web primero, móvil en fase 2
- CI/CD automatizado para ambas plataformas
- Beta testing con TestFlight (iOS) y Google Play Beta

### Alternativas Rechazadas

**Opción A: Solo PWA (Progressive Web App)**
- Rechazada por: limitaciones en notificaciones push en iOS, no aparece en tiendas de apps, experiencia de usuario inferior

**Opción B: Aplicación híbrida (Ionic/Cordova)**
- Rechazada por: performance inferior a React Native, experiencia menos nativa

**Opción C: Solo web sin móvil**
- Rechazada por: 60% del tráfico actual en plataformas de arte viene de móvil

---

## ADR-004: Almacenamiento Externo para Imágenes de Alta Resolución

### Estado
**Aceptado** - 2025-10-14

### Contexto
Las obras de arte requieren imágenes y videos de muy alta calidad para que los compradores puedan apreciar los detalles.

**Requisitos identificados:**
- Imágenes de alta resolución (4K, 8K para algunas obras)
- Múltiples versiones (thumbnail, preview, original)
- Videos de obras en 360° o time-lapses de creación
- Velocidad de carga rápida globalmente
- Escalabilidad para millones de obras

**Datos estimados:**
- 10,000 obras iniciales
- Promedio 5 imágenes por obra a 5MB c/u = 250GB
- Crecimiento proyectado: 1,000 obras/mes
- Videos: ~100MB por obra con video

### Decisión
Utilizar **almacenamiento en la nube externo** (AWS S3 o similar) para todas las imágenes y videos, separado de la base de datos.

**Estrategia:**
- Almacenar solo metadata y URLs en PostgreSQL
- Imágenes y videos en S3 con diferentes calidades
- CDN (CloudFront) para distribución global rápida
- Procesamiento automático de imágenes (redimensionado, optimización)

### Consecuencias

**Positivas:**
- Escalabilidad casi ilimitada (petabytes si es necesario)
- Durabilidad extrema (99.999999999% en S3)
- CDN global reduce latencia en todo el mundo
- Optimización automática de imágenes
- No sobrecarga la base de datos
- Costo por uso (pay-as-you-go)
- Backup automático incluido
- Procesamiento de imágenes con Lambda o servicios especializados

**Negativas:**
- Costo recurrente mensual (~$25-50/mes inicialmente)
- Dependencia de proveedor externo
- Complejidad adicional en la arquitectura
- Requiere gestión de URLs y expiración de enlaces
- Posible vendor lock-in

**Mitigaciones:**
- Usar abstracción de almacenamiento para facilitar cambio de proveedor
- Implementar caché de imágenes populares
- Lazy loading de imágenes en frontend
- Compresión automática sin pérdida de calidad
- Monitoreo de costos mensual con alertas
- Política de lifecycle para archivar imágenes no usadas

### Alternativas Rechazadas

**Opción A: Almacenar imágenes en PostgreSQL (BLOB)**
- Rechazada por: base de datos se volvería enorme, afecta performance, backups muy lentos, no escalable

**Opción B: Servidor de archivos propio**
- Rechazada por: requiere gestión de infraestructura, sin CDN global, sin redundancia, costos más altos a largo plazo

**Opción C: Almacenar solo thumbnails en servidor propio**
- Rechazada por: complejidad de gestionar dos sistemas, no resuelve problema de escalabilidad

---

## ADR-005: Integración con Blockchain para Certificación

### Estado
**Aceptado** - 2025-10-14

### Contexto
Uno de los valores diferenciales de Arty es garantizar la autenticidad y trazabilidad de las obras de arte, especialmente las digitales.

**Problemática identificada:**
- Falsificaciones de arte digital son comunes
- Compradores necesitan garantía de autenticidad
- Artistas quieren proteger su propiedad intelectual
- Necesidad de trazabilidad de la cadena de custodia

**Requisitos:**
- Registro inmutable de autoría
- Prueba de propiedad para compradores
- Trazabilidad de ventas y transferencias
- Validación independiente de autenticidad

### Decisión
Integrar con **plataforma blockchain** (inicialmente Polygon, capa 2 de Ethereum) para emitir certificados digitales de autenticidad y propiedad.

**Implementación:**
- Cada obra certificada recibe un NFT (token no fungible)
- Metadata almacenada en IPFS (descentralizada)
- Smart contract gestiona transferencias de propiedad
- Certificado digital descargable para compradores

### Consecuencias

**Positivas:**
- Garantía criptográfica de autenticidad
- Registro inmutable y verificable públicamente
- Diferenciador competitivo importante
- Aumenta confianza de compradores
- Permite reventa verificada en mercado secundario
- Trending topic: NFTs atraen atención mediática
- Internacionalmente reconocido
- Los artistas mantienen royalties en reventas

**Negativas:**
- Complejidad técnica adicional (Web3, wallets, smart contracts)
- Costos de gas por transacción (aunque Polygon es económico: ~$0.01)
- Curva de aprendizaje para usuarios (wallets, blockchain)
- Percepción negativa de NFTs por algunos sectores
- Requiere expertise en blockchain
- Dependencia de plataforma blockchain externa

**Mitigaciones:**
- Usar Polygon (Layer 2) por tarifas bajas y velocidad
- Abstraer complejidad: usuarios no necesitan conocer blockchain
- Arty maneja wallets en nombre de usuarios (custodial)
- Certificación blockchain es opcional (no obligatoria)
- Educación a usuarios sobre beneficios
- Plan B: sistema de certificación centralizado si blockchain no escala

### Alternativas Rechazadas

**Opción A: Sistema de certificación centralizado propio**
- Rechazada por: menos confianza (Arty podría falsificar), no verificable externamente, no permite reventa trazable

**Opción B: Ethereum mainnet**
- Rechazada por: costos de gas muy altos ($5-50 por transacción), lento (15 transacciones por segundo)

**Opción C: Blockchain privada**
- Rechazada por: no públicamente verificable, pierde beneficios de descentralización

---

## ADR-006: Sistema de Subastas en Tiempo Real

### Estado
**Aceptado** - 2025-10-14

### Contexto
Las subastas son una funcionalidad core de Arty para generar valor y emoción en la compra de obras.

**Requisitos identificados:**
- Múltiples usuarios pujando simultáneamente
- Actualizaciones en tiempo real de ofertas
- Contador regresivo visible para todos
- Prevenir fraude y pujas falsas
- Notificaciones instantáneas cuando alguien supera tu oferta

**Desafíos técnicos:**
- Sincronización entre múltiples usuarios
- Consistencia de datos (no permitir dos ganadores)
- Latencia mínima (<500ms para actualizar)
- Escalabilidad en subastas populares (cientos de usuarios)

### Decisión
Implementar sistema de subastas con **WebSockets** para comunicación bidireccional en tiempo real entre servidor y clientes.

**Arquitectura:**
- Servidor WebSocket maneja conexiones persistentes
- Broadcast de ofertas a todos los usuarios conectados
- Base de datos como fuente de verdad
- Sistema de heartbeat para detectar desconexiones
- Cola de mensajes para procesamiento asíncrono

### Consecuencias

**Positivas:**
- Experiencia de usuario fluida y en tiempo real
- Latencia mínima en actualización de ofertas
- Emoción y competitividad en subastas
- Notificaciones instantáneas
- Soporta cientos de usuarios concurrentes por subasta
- WebSocket es estándar bien soportado
- Reduce carga en servidor (vs polling continuo)

**Negativas:**
- Mayor complejidad técnica que REST simple
- Requiere infraestructura escalable para WebSockets
- Gestión de conexiones persistentes consume recursos
- Debugging más complejo
- Necesita manejo robusto de desconexiones/reconexiones
- Firewall/proxy corporativos pueden bloquear WebSockets

**Mitigaciones:**
- Fallback a long-polling si WebSocket falla
- Implementar reconexión automática en cliente
- Rate limiting por usuario para prevenir spam
- Validación de ofertas en backend antes de broadcast
- Logging exhaustivo de todas las pujas
- Testing de carga para identificar límites
- Escalado horizontal con balanceo de carga sticky sessions

### Alternativas Rechazadas

**Opción A: Polling (peticiones HTTP cada X segundos)**
- Rechazada por: alta latencia (5-10 segundos), carga excesiva en servidor, mala experiencia de usuario

**Opción B: Server-Sent Events (SSE)**
- Rechazada por: comunicación unidireccional (servidor→cliente), WebSocket es más flexible

**Opción C: Subastas por rondas (no tiempo real)**
- Rechazada por: pierde emoción y valor diferencial, experiencia de usuario inferior

---

## Registro de Cambios

| Fecha | ADR | Cambio | Autor |
|-------|-----|--------|-------|
| 2025-10-14 | Todos | Creación inicial de ADRs de Arty | Angel Quishpe, Alan Rivera, Jorge Escobar |

---

## Próximas Revisiones

- **ADR-001**: 2025-12-14 (evaluar si monolito es suficiente)
- **ADR-002**: 2026-01-14 (revisar performance de PostgreSQL)
- **ADR-003**: 2026-02-14 (evaluar adopción de app móvil)
- **ADR-004**: 2025-11-14 (revisar costos de almacenamiento)
- **ADR-005**: 2026-01-14 (evaluar uso de certificación blockchain)
- **ADR-006**: 2025-12-14 (medir performance de subastas en tiempo real)

---

## Decisiones Futuras a Considerar

- Motor de búsqueda especializado (Elasticsearch) para búsquedas semánticas avanzadas
- Sistema de recomendaciones con Machine Learning basado en gustos del usuario
- Integración con redes sociales para compartir obras
- Sistema de mensajería directa entre artistas y compradores
- Pasarela de pagos múltiple (criptomonedas, transferencias internacionales)
- Galería virtual en Realidad Virtual (VR) para experiencias inmersivas
- Programa de afiliados y comisiones para promotores
- API pública para integraciones de terceros

---

**Fin del documento**