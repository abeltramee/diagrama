### Arquitectura del Sistema
```mermaid
graph TD
  ... (graph TD
    %% Definición de estilos
    classDef analog fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef digital fill:#d4edda,stroke:#28a745,stroke-width:2px,color:black;
    classDef peripheral fill:#cce5ff,stroke:#007bff,stroke-width:2px,color:black;
    classDef software fill:#fff3cd,stroke:#ffc107,stroke-width:2px,stroke-dasharray: 5 5,color:black;

    %% --- MUNDO ANALÓGICO DE ENTRADA (Baseboard) ---
    subgraph "Baseboard de Acondicionamiento (Analógico)"
        Mic[Micrófono / Entrada Jack]:::analog --> PreAmp[Pre-amplificador y Suma de Offset DC]:::analog
        PreAmp --> AAF[Filtro Antialiasing Analógico (Pasa-bajos)]:::analog
    end

    %% --- NÚCLEO DIGITAL (STM32) ---
    subgraph "STM32 NUCLEO-F446RE (Dominio Digital)"
        AAF == Señal Analógica (0-3.3V) ==> ADC[ADC (Conversor A/D)]:::digital

        subgraph "Arquitectura de Software (FreeRTOS)"
            ADC -- DMA --> BuffRX[Buffer RX (buffer_RX[])]:::software
            
            BuffRX --> DSP_Task[Tarea DSP: Filtros Digitales FIR/IIR + AA Digital]:::software
            
            DSP_Task --> BuffTX[Buffer TX (buffer_TX[])]:::software
            
            BuffTX -- DMA --> DAC[DAC (Conversor D/A)]:::digital
            
            USB_Task[Tarea USB: Recepción de Parámetros]:::software -.->|Actualiza Coeficientes| DSP_Task
            GUI_Task[Tarea GUI: Gestión de Display]:::software
        end
    end

    %% --- PERIFÉRICOS E INTERFACES ---
    PC[PC Host (Control)]:::peripheral <==>|USB (Comunicaciones)| USB_Task
    
    GUI_Task ==>|SPI (Datos + Clock)| Display[Display TFT / Dot Matrix]:::peripheral

    %% --- MUNDO ANALÓGICO DE SALIDA ---
    subgraph "Etapa de Salida (Analógica)"
        DAC == Señal Escalada ==> ReconFilter[Filtro de Reconstrucción Analógico]:::analog
        ReconFilter --> AmpSalida[Adaptación de Impedancia / Salida Jack]:::analog
        AmpSalida --> Speaker[Altavoces / Auriculares]:::analog
    end

    %% Leyenda de conexiones
    %% ==> Flujo de Audio Principal
    %% --> Flujo de Datos Interno
    %% -.-> Flujo de Control/Parámetros
    %% <==> Comunicación Bidireccional)
