### Arquitectura del Sistema
```mermaid
graph TD
    %% --- MUNDO ANALÓGICO DE ENTRADA (Baseboard) ---
    subgraph "Baseboard de Acondicionamiento"
        Mic["Micrófono / Entrada Jack"] --> PreAmp["Pre-amplificador y Offset DC"]
    end

    %% --- NÚCLEO DIGITAL (STM32) ---
    subgraph "STM32 NUCLEO-F446RE"
        PreAmp == "Señal (0-3.3V)" ==> ADC[ADC]

        subgraph "Software (FreeRTOS)"
            ADC -- DMA --> BuffRX["Buffer RX"]
            
            subgraph "Procesamiento Digital"
                BuffRX --> AA_Digital["Antialiasing Digital"]
                AA_Digital --> DSP_Node["DSP (Filtros FIR/IIR)"]
            end
            
            DSP_Node --> BuffTX["Buffer TX"]
            BuffTX -- DMA --> DAC[DAC]
            
            USB_Node["USB"] -.->|Parámetros| DSP_Node
            GUI_Node["GUI"]
        end
    end

    %% --- PERIFÉRICOS ---
    PC["PC Host"] <==>|USB| USB_Node
    GUI_Node ==>|SPI| Display["Display"]

    %% --- SALIDA (Con Módulo PAM8403) ---
    subgraph "Etapa de Salida"
        DAC == "Audio" ==> ReconFilter["Filtro Reconstrucción"]
        ReconFilter --> PAM8403["Módulo Ampl. PAM8403"]
        PAM8403 --> Speaker["Speaker"]
    end

    %% Estilos con texto en negro
    class Mic,PreAmp,ReconFilter,PAM8403,Speaker analog;
    class ADC,DAC digital;
    class PC,Display peripheral;
    class BuffRX,AA_Digital,DSP_Node,BuffTX,USB_Node,GUI_Node software;

    classDef analog fill:#f9f,stroke:#333,stroke-width:2px,color:#000;
    classDef digital fill:#d4edda,stroke:#28a745,stroke-width:2px,color:#000;
    classDef peripheral fill:#cce5ff,stroke:#007bff,stroke-width:2px,color:#000;
    classDef software fill:#fff3cd,stroke:#ffc107,stroke-width:2px,stroke-dasharray: 5 5,color:#000;
