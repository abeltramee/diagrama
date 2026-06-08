### Arquitectura del Sistema
```mermaid
graph TD
    %% --- MUNDO ANALÓGICO DE ENTRADA (Baseboard) ---
    subgraph "Baseboard de Acondicionamiento"
        Mic["Micrófono / Entrada Jack"] --> PreAmp["Pre-amplificador y Offset DC"]
        PreAmp --> AA_Analog["Filtro Antialiasing (Pasa-bajos)"]
    end

    %% --- NÚCLEO DIGITAL (STM32) ---
    subgraph "STM32 NUCLEO-F446RE"
        AA_Analog == "Señal (0-3.3V)" ==> ADC[ADC]

        subgraph "Software (FreeRTOS)"
            ADC -- DMA --> BuffRX["Buffer RX"]
            
            subgraph "Procesamiento Digital"
                BuffRX --> DSP_Node["DSP (Filtros FIR/IIR)"]
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

    %% --- SALIDA SIMPLIFICADA ---
    subgraph "Etapa de Salida"
        DAC == "Audio" ==> CapAcople["Capacitor de Acople"]
        CapAcople --> Amp["Amplificador"]
        Amp --> Speaker["Speaker"]
    end

    %% Estilos con texto en negro
    class Mic,PreAmp,AA_Analog,CapAcople,Amp,Speaker analog;
    class ADC,DAC digital;
    class PC,Display peripheral;
    class BuffRX,DSP_Node,BuffTX,USB_Node,GUI_Node software;

    classDef analog fill:#f9f,stroke:#333,stroke-width:2px,color:#000;
    classDef digital fill:#d4edda,stroke:#28a745,stroke-width:2px,color:#000;
    classDef peripheral fill:#cce5ff,stroke:#007bff,stroke-width:2px,color:#000;
    classDef software fill:#fff3cd,stroke:#ffc107,stroke-width:2px,stroke-dasharray: 5 5,color:#000;
