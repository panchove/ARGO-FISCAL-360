# ARGO FISCAL PRINTER 360 – Arquitectura del Sistema

**Código:** ARGO-FISCAL-PRINTER-360  
**Documento:** Arquitectura del Sistema  
**Versión:** 1.0  
**Estado:** Borrador  

---

## 1. Propósito

Definir la arquitectura de ARGO FISCAL 360, describiendo su estructura modular, capas, responsabilidades y flujo de ejecución para garantizar cumplimiento fiscal, trazabilidad y extensibilidad.

---

## 2. Principios de Arquitectura

- Separación de responsabilidades
- Aislamiento por capas
- Independencia de fabricantes
- Trazabilidad total
- Robustez ante fallos
- Compatibilidad estricta con ICG

---

## 3. Vista General

```plantuml
@startuml
title Arquitectura General - ARGO FISCAL 360

rectangle "ICG POS" as POS

rectangle "ARGO FISCAL 360" {
  component "ICG Bridge"
  component "ICG Compliance Layer"
  component "Fiscal Core"
  component "Journal Manager"
  component "Recovery Module"
  component "Drivers"
  component "Protocol Layer"
  component "Transport Layer"
}

database "BD ICG" as DB
database "SQLite Journal" as SQLITE
folder "File System" as FS
node "Impresora Fiscal" as PRN

POS --> "ICG Bridge"
"ICG Bridge" --> "ICG Compliance Layer"
"ICG Compliance Layer" --> "Fiscal Core"

"Fiscal Core" --> DB
"Fiscal Core" --> SQLITE
"Fiscal Core" --> FS

"Fiscal Core" --> "Drivers"
"Drivers" --> "Protocol Layer"
"Protocol Layer" --> "Transport Layer"
"Transport Layer" --> PRN

"Recovery Module" --> SQLITE
"Recovery Module" --> DB

@enduml
```

---

## 4. Capas del Sistema

### 4.1 ICG Bridge

Responsable de:

- Exponer funciones DLL esperadas por ICG
- Recibir XML
- Manejar contrato de entrada/salida

---

### 4.2 ICG Compliance Layer

- Normalización Retail/Rest/Hotel/Manager
- Validación estricta de XML
- Adaptación de flujos

---

### 4.3 Fiscal Core

- Orquestación de la operación
- Validación de negocio
- Generación de comandos fiscales
- Coordinación con BD ICG
- Coordinación con drivers

Es el **corazón del sistema**.

---

### 4.4 Journal Manager

- Persistencia en SQLite
- Manejo de estados de transacción
- Generación de hash
- Registro de eventos

---

### 4.5 Recovery Module

- Detección de inconsistencias
- Reconstrucción de datos fiscales
- Reaplicación a BD ICG

---

### 4.6 Drivers

- Implementación por fabricante
- Conversión de documento → comandos

Ejemplos:

- Driver.HKA
- Driver.PNP
- Driver.VMAX
- Driver.ISC

---

### 4.7 Protocol Layer

- Construcción de tramas
- Codificación de comandos
- Manejo de checksum/LRC

---

### 4.8 Transport Layer

- Serial (COM)
- USB (COM virtual)
- TCP/IP

---

## 5. Flujo de Ejecución

```plantuml
@startuml
title Flujo de Ejecución

start

:ICG llama DLL;
:Recibir XML;

:Validar (Compliance Layer);
:Crear transacción SQLite;

:Consultar BD ICG;
:Generar modelo interno;

:Generar comandos;

if (Estado impresora OK?) then (Sí)
  :Enviar comandos;
  :Recibir respuesta;
  :Obtener resultado fiscal;
  :Actualizar BD ICG;
  :Guardar expediente;
  :Estado = Printed;
else (No)
  :Registrar error;
  :Estado = Failed;
endif

stop
@enduml
```

---

## 6. Arquitectura de Drivers

```plantuml
@startuml
title Arquitectura de Drivers

interface IFiscalPrinterDriver

class HkaDriver
class PnpDriver
class VmaxDriver
class IscDriver

IFiscalPrinterDriver <|-- HkaDriver
IFiscalPrinterDriver <|-- PnpDriver
IFiscalPrinterDriver <|-- VmaxDriver
IFiscalPrinterDriver <|-- IscDriver

@enduml
```

---

## 7. Arquitectura de Protocolo

```plantuml
@startuml
title Protocolo

class CommandBuilder
class FrameEncoder
class Transport

CommandBuilder --> FrameEncoder
FrameEncoder --> Transport

@enduml
```

---

## 8. Gestión de Estado

Cada transacción pasa por:

```text
Created → Validated → CommandsGenerated → Printing → Printed
                               ↓
                            Failed
                               ↓
                        RecoveryRequired
                               ↓
                            Recovered
```

---

## 9. Estrategia de Integración

- Fase 1: Compatibilidad total con DLL existente
- Fase 2: Implementación de drivers directos
- Fase 3: Eliminación de dependencias externas

---

## 10. Escalabilidad

- 1 instancia por POS
- Aislamiento total por carpeta
- No compartición de impresoras

---

## 11. Estado del documento

Borrador inicial – sujeto a validación
