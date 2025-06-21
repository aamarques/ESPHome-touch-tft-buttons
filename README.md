# 📱 Monitor Garagem/Portão ESP32 - Documentação Técnica

**Versão Atual:** v3.4 (Dezembro 2024)  
**Status:** ✅ Funcionando Perfeitamente  
**Hardware:** NodeMCU-32S + TFT ILI9341 2.8" + XPT2046 Touch

---

## 📋 Índice

1. [Visão Geral](#visão-geral)
2. [Hardware e Componentes](#hardware-e-componentes)
3. [Ligações e Pinout](#ligações-e-pinout)
4. [Configurações de Software](#configurações-de-software)
5. [Coordenadas Mapeadas](#coordenadas-mapeadas)
6. [Estados e Comandos](#estados-e-comandos)
7. [Interface Visual](#interface-visual)
8. [Proteções e Segurança](#proteções-e-segurança)
9. [Troubleshooting](#troubleshooting)
10. [Histórico de Versões](#histórico-de-versões)

---

## 🎯 Visão Geral

Sistema de monitoramento e controle para Garagem e Portão baseado em ESP32 com interface touchscreen. Integrado ao Home Assistant via WiFi, permite visualizar estados dos sensores e acionar dispositivos remotamente.

### Funcionalidades Principais

- ✅ **Display Landscape 2.8"** com interface touchscreen
- ✅ **Monitoramento em tempo real** de 2 sensores binários
- ✅ **Controle por toque** de Garagem e Portão
- ✅ **Integração Home Assistant** via API
- ✅ **Status WiFi, Data/Hora e Uptime**
- ✅ **Proteções anti-loop** e debounce
- ✅ **IP estático configurado**

---

## 🔧 Hardware e Componentes

### Componentes Utilizados

| Componente | Modelo | Especificações |
|------------|--------|----------------|
| **Microcontrolador** | NodeMCU-32S | ESP32-WROOM-32, 520KB RAM, SEM PSRAM |
| **Display** | TFT ILI9341 2.8" | 240x320 pixels, SPI, 3.3V |
| **Touchscreen** | XPT2046 | Resistivo, SPI compartilhado |
| **Alimentação** | USB 5V | Via NodeMCU ou fonte externa |

### Especificações Técnicas

**ESP32 NodeMCU-32S:**
- Chip: ESP32-WROOM-32 (SEM PSRAM)
- CPU: Dual-core Xtensa LX6 @ 240MHz
- RAM: 520KB interna (sem PSRAM externa)
- Flash: 4MB
- WiFi: 802.11 b/g/n
- Bluetooth: v4.2 BR/EDR + BLE

**Display ILI9341:**
- Resolução: 240x320 pixels (nativo portrait)
- Interface: SPI 4-wire
- Tensão: 3.3V
- Taxa de dados: 20MHz (configurado)
- Paleta de cores: 8-bit (otimizado para RAM)

**Touchscreen XPT2046:**
- Tipo: Resistivo
- Interface: SPI (compartilhado com display)
- Tensão: 3.3V
- Threshold: 600 (configurado)

---

## 🔌 Ligações e Pinout

### Tabela de Conexões

| Função | GPIO ESP32 | Pino Display/Touch | Descrição |
|--------|------------|-------------------|-----------|
| **SPI CLK** | GPIO18 | SCK | Clock compartilhado |
| **SPI MOSI** | GPIO23 | SDI/MOSI | Data OUT compartilhado |
| **SPI MISO** | GPIO19 | SDO/MISO | Data IN (usado pelo touch) |
| **Display DC** | GPIO2 | DC/RS | Data/Command select |
| **Display CS** | GPIO15 | CS | Chip Select display |
| **Display RST** | GPIO4 | RST | Reset display |
| **Touch CS** | GPIO16 | T_CS | Chip Select touch |
| **Touch IRQ** | GPIO17 | T_IRQ | Touch interrupt |
| **Backlight** | GPIO5 | LED/BL | Controle backlight |
| **Alimentação** | 3.3V | VCC | Alimentação 3.3V |
| **Terra** | GND | GND | Terra comum |

### Diagrama de Ligações

```
NodeMCU-32S          TFT ILI9341 2.8"
┌─────────────┐      ┌──────────────┐
│ GPIO18 (SCK)├──────┤ SCK          │
│GPIO23 (MOSI)├──────┤ SDI/MOSI     │
│GPIO19 (MISO)├──────┤ SDO/MISO     │
│  GPIO2 (DC) ├──────┤ DC/RS        │
│ GPIO15 (CS) ├──────┤ CS           │
│ GPIO4 (RST) ├──────┤ RST          │
│GPIO16(T_CS) ├──────┤ T_CS         │
│GPIO17(T_IRQ)├──────┤ T_IRQ        │
│ GPIO5 (BL)  ├──────┤ LED/BL       │
│        3.3V ├──────┤ VCC          │
│         GND ├──────┤ GND          │
└─────────────┘      └──────────────┘
```

### Notas Importantes de Hardware

⚠️ **CRÍTICO:** NodeMCU-32S NÃO possui PSRAM - usar apenas configurações 8-bit  
⚠️ **SPI Compartilhado:** Display e touchscreen usam o mesmo barramento SPI  
⚠️ **Tensão:** Todos os componentes operam em 3.3V  
⚠️ **Velocidade SPI:** Máximo 20MHz para evitar erros em modo full-duplex

---

## ⚙️ Configurações de Software

### Configurações ESPHome Principais

```yaml
# Framework e Board
esp32:
  board: nodemcu-32s
  framework:
    type: esp-idf

# Display - Configuração Aprovada
display:
  - platform: ili9xxx
    model: ili9341
    rotation: 90                    # Landscape
    dimensions:
      height: 320                   # Dimensão física original
      width: 240                    # Dimensão física original
    invert_colors: false            # Cores corretas
    color_order: BGR                # Padrão ILI9341
    data_rate: 20MHz               # Velocidade segura
    color_palette: 8BIT            # Economiza RAM

# Touchscreen - Configuração Otimizada
touchscreen:
  platform: xpt2046
  update_interval: 100ms          # Anti-spam
  threshold: 600                  # Anti-ruído
  calibration:
    x_min: 280
    x_max: 3860
    y_min: 340
    y_max: 3860
```

### Configurações de Rede

```yaml
# WiFi com IP Estático
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.3.9
    gateway: 192.168.3.1
    subnet: 255.255.255.0
    dns1: 192.168.3.1
    dns2: 8.8.8.8
```

### Configurações de Proteção

```yaml
# Variáveis Anti-Loop
globals:
  - id: system_ready
    type: bool
    initial_value: 'false'         # Sistema inicia protegido
  - id: last_touch_time
    type: unsigned long
    initial_value: '0'             # Controle de debounce

# Script de Inicialização
script:
  - id: system_startup
    then:
      - delay: 5s                  # Aguarda 5s após boot
      - lambda: id(system_ready) = true;  # Habilita touchscreen
```

---

## 📍 Coordenadas Mapeadas

### Processo de Mapeamento

As coordenadas foram mapeadas através de **testes reais** tocando em pontos específicos da tela e registrando as coordenadas exatas retornadas pelo touchscreen.

### Coordenadas dos Botões (Testadas)

#### Botão Garagem (Área Verde)
```yaml
Coordenadas Testadas:
- Canto Superior Esquerdo: X=120, Y=233
- Canto Superior Direito:  X=120, Y=130  
- Centro:                  X=99,  Y=185
- Canto Inferior Esquerdo: X=80,  Y=233
- Canto Inferior Direito:  X=80,  Y=130

Área Final Configurada:
X: 80 ≤ touch_x ≤ 120
Y: 130 ≤ touch_y ≤ 233
```

#### Botão Portão (Área Vermelha)
```yaml
Coordenadas Testadas:
- Canto Superior Esquerdo: X=120, Y=117
- Canto Superior Direito:  X=120, Y=7
- Centro:                  X=99,  Y=71
- Canto Inferior Esquerdo: X=78,  Y=117
- Canto Inferior Direito:  X=78,  Y=7

Área Final Configurada:
X: 78 ≤ touch_x ≤ 120  
Y: 7 ≤ touch_y ≤ 117
```

### Linha Divisória (Referência)
```yaml
Linha Amarela Central: Y=121
- Exatamente na linha: X=151, Y=121
- Pouco à esquerda:    X=144, Y=136
- Pouco à direita:     X=144, Y=115
```

### Mapa Visual das Coordenadas

```
Tela Física (320x240 landscape):
┌─────────────────────────────────────┐ Y=0
│           ÁREA PORTÃO                │
│      X=78-120, Y=7-117              │ Y=117
├─────────────────┬───────────────────┤ Y=121 (Linha Divisória)
│   ÁREA GARAGEM  │                   │
│ X=80-120,Y=130- │                   │ Y=130
│           233   │                   │
│                 │                   │ Y=233
└─────────────────┴───────────────────┘ Y=240
X=0              X=160               X=320
```

---

## 🎮 Estados e Comandos

### Sensores Home Assistant

#### Sensor Garagem
```yaml
Entidade HA: binary_sensor.shellyplus1_441793d027f0_switch_100_input
ID Local: garage_door_sensor
Estados:
  - true:  "ABERTA" (texto verde, círculo verde)
  - false: "FECHADA" (texto vermelho, círculo vermelho)
```

#### Sensor Portão  
```yaml
Entidade HA: binary_sensor.reed_switch_input
ID Local: gate_door_sensor
Estados:
  - true:  "ABERTO" (texto verde, círculo verde)
  - false: "FECHADO" (texto vermelho, círculo vermelho)
```

### Comandos de Controle

#### Comando Garagem
```yaml
Entidade HA: switch.portao_da_garagem_switch_0
Serviço: switch.toggle
Ação: Aciona Shelly 1 Plus para abrir/fechar garagem
```

#### Comando Portão
```yaml
Entidade HA: switch.portao_de_correr_switch_0  
Serviço: switch.toggle
Ação: Aciona addon conectado ao reed switch
```

### Estados do Sistema

#### Status WiFi
```yaml
- Conectado: "WiFi:OK" (verde)
- Desconectado: "WiFi:OFF" (vermelho)
```

#### Status Touchscreen
```yaml
- Habilitado: "Touch:ON" (verde) - após 5s do boot
- Desabilitado: "Touch:OFF" (vermelho) - primeiros 5s
```

#### Data e Hora
```yaml
Formato: DD/MM HH:MM
Exemplo: "19/06 14:30"
Fonte: SNTP (Europe/Lisbon)
```

#### Uptime
```yaml
Formato: Up:XdHH:MM
Exemplo: "Up:2d05:15" (2 dias, 5 horas, 15 minutos)
```

---

## 🖥️ Interface Visual

### Layout da Tela (320x240 landscape)

```
┌─────────────────────────────────────┐
│    GARAGEM      │      PORTÃO       │ ← Títulos (linha 15)
│    FECHADA      │      FECHADO      │ ← Estados (linha 40)
│                 │                   │
│       ●         │        ●          │ ← Indicadores (Y=95)
│   [ ACIONAR ]   │   [ ACIONAR ]     │ ← Botões (Y=140-180)
├─────────────────┴───────────────────┤
│WiFi:OK│Touch:ON│19/06 14:30│Up:2d5h│ ← Rodapé (linha 215)
└─────────────────────────────────────┘
```

### Elementos Visuais

#### Títulos
- **Posição:** "GARAGEM" (50,15), "PORTÃO" (210,15)
- **Fonte:** font_title (16px)
- **Cor:** Branco (255,255,255)

#### Estados
- **Posição:** Centralizados abaixo dos títulos
- **Fonte:** font_normal (12px)
- **Cores:** Verde (ABERTO/ABERTA), Vermelho (FECHADO/FECHADA)

#### Indicadores Circulares
- **Posição:** (80,95) Garagem, (240,95) Portão
- **Raio:** 20 pixels
- **Cores:** Verde (ABERTO), Vermelho (FECHADO)
- **Borda:** Branca (255,255,255)

#### Botões
- **Garagem:** Retângulo (5,140,150,40) - Azul (0,120,200)
- **Portão:** Retângulo (165,140,150,40) - Laranja (200,120,0)
- **Efeito Pressionado:** Cores invertidas por 500ms
- **Texto:** "ACIONAR" centralizado

#### Divisória
- **Linha Vertical:** X=160, Y=10-200, Cinza (100,100,100)
- **Linha Horizontal:** Y=200, X=10-310, Cinza (100,100,100)

### Esquema de Cores

| Elemento | Estado | RGB | Hexadecimal |
|----------|--------|-----|-------------|
| **Fundo** | - | (0,0,0) | #000000 |
| **Texto Branco** | - | (255,255,255) | #FFFFFF |
| **Aberto/Aberta** | Ativo | (0,255,0) | #00FF00 |
| **Fechado/Fechada** | Inativo | (255,0,0) | #FF0000 |
| **Botão Garagem** | Normal | (0,120,200) | #0078C8 |
| **Botão Portão** | Normal | (200,120,0) | #C87800 |
| **WiFi OK** | Conectado | (0,255,0) | #00FF00 |
| **Data/Hora** | - | (255,255,0) | #FFFF00 |
| **Uptime** | - | (0,255,255) | #00FFFF |
| **Linhas** | - | (100,100,100) | #646464 |

---

## 🛡️ Proteções e Segurança

### Sistema Anti-Loop

#### Problema Original
O sistema entrava em loop infinito enviando comandos continuamente para o switch da garagem, causando abertura/fechamento constante.

#### Soluções Implementadas

**1. Inicialização Segura (5 segundos)**
```yaml
on_boot:
  priority: 600
  then:
    - script.execute: system_startup

script:
  - id: system_startup
    then:
      - delay: 5s
      - lambda: id(system_ready) = true;
```
- **Resultado:** Touchscreen desabilitado por 5s após boot
- **Indicador:** "Touch:OFF" (vermelho) → "Touch:ON" (verde)

**2. Debounce de Toque (1 segundo)**
```yaml
if (current_time - id(last_touch_time) < 1000) {
  ESP_LOGI("touch", "Debounce ativo - ignorando touch");
  return;
}
```
- **Resultado:** Máximo 1 toque por segundo
- **Previne:** Toques múltiplos acidentais

**3. Threshold Aumentado**
```yaml
threshold: 600  # (anterior: 400)
```
- **Resultado:** Menos sensível a ruído elétrico
- **Previne:** Toques fantasma

**4. Update Interval Otimizado**
```yaml
update_interval: 100ms  # (anterior: 50ms)
```
- **Resultado:** Menos verificações de toque
- **Previne:** Spam de eventos

**5. Filtros de Coordenadas**
```yaml
// Ignora coordenadas claramente inválidas
if ((touch_x == 0 && touch_y == 0) || 
    (touch_x == 319 && touch_y == 239) ||
    touch_x < 0 || touch_x > 319 ||
    touch_y < 0 || touch_y > 239) {
  return;
}
```

### Logs de Debug

O sistema possui logs detalhados para debug:
```
[touch] Sistema ainda não está pronto, ignorando touch
[touch] Debounce ativo - ignorando touch  
[touch] Coordenadas inválidas X:0 Y:0 - ignorando
[touch] Touch VÁLIDO detectado em X:95 Y:185
[touch] ACIONANDO GARAGEM - X:95 Y:185
```

---

## 🔧 Troubleshooting

### Problemas Comuns e Soluções

#### 1. Erro PSRAM na Inicialização
```
E (514) quad_psram: PSRAM ID read error: 0xffffffff
E (518) esp_psram: PSRAM enabled but initialization failed
```
**Causa:** NodeMCU-32S não possui PSRAM  
**Solução:** Remover configuração `psram:` do código  
**Status:** ✅ Corrigido na v2.3

#### 2. Erro SPI Full-Duplex
```
E (697) spi_hal: When work in full-duplex mode at frequency > 26.7MHz
E (737) spi_master: assigned clock speed not supported
```
**Causa:** `data_rate: 40MHz` muito alto para modo compartilhado  
**Solução:** Reduzir para `data_rate: 20MHz`  
**Status:** ✅ Corrigido na v2.2

#### 3. Display com Linhas/Piscar
```
Sintomas: Linhas na tela, piscar, preenchimento incompleto
```
**Causa:** Dimensões incorretas (`width: 320, height: 240`)  
**Solução:** Usar dimensões físicas (`height: 320, width: 240`)  
**Status:** ✅ Corrigido na v2.4

#### 4. Touch Não Detectado
```
Sintomas: Toques não registrados ou em área errada
```
**Causa:** Coordenadas mapeadas incorretamente  
**Solução:** Usar coordenadas reais testadas (X=78-120, Y=7-233)  
**Status:** ✅ Corrigido na v3.3

#### 5. Loop de Comandos
```
Sintomas: Garagem abrindo/fechando continuamente
```
**Causa:** Toques fantasma durante inicialização  
**Solução:** Sistema anti-loop completo (debounce, delay, filtros)  
**Status:** ✅ Corrigido na v3.4

### Verificações de Diagnóstico

#### Verificar Hardware
```bash
# Conexões SPI
- GPIO18 → SCK (Clock)
- GPIO23 → MOSI (Data OUT)  
- GPIO19 → MISO (Data IN)
- Tensão: 3.3V em todos os componentes
```

#### Verificar Software
```yaml
# ESPHome Logger
logger:
  level: INFO  # ou DEBUG para mais detalhes

# Verificar logs específicos
[touch] Touch VÁLIDO detectado em X:XX Y:XX
[startup] Sistema PRONTO - touchscreen HABILITADO
```

#### Verificar Rede
```bash
# Ping para o IP estático
ping 192.168.3.9

# Verificar no Home Assistant
Developer Tools → States → search "esp32-monitor-garagem"
```

---

## 📚 Histórico de Versões

### v3.4 (Atual) - Proteção Anti-Loop
**Data:** Dezembro 2024  
**Status:** ✅ Funcionando Perfeitamente

**Correções Críticas:**
- 🛡️ Sistema anti-loop completo implementado
- ⏱️ Debounce de 1 segundo entre toques
- 🚀 Inicialização segura com 5s de delay
- 🔧 Threshold aumentado para 600
- 📊 Indicador "Touch:ON/OFF" no rodapé

**Configurações:**
- Coordenadas: Garagem X=80-120,Y=130-233 | Portão X=78-120,Y=7-117
- Proteções: system_ready, debounce, filtros rigorosos
- Interface: Status completo no rodapé

### v3.3 - Coordenadas Mapeadas
**Data:** Dezembro 2024

**Correções:**
- 📍 Coordenadas reais mapeadas via testes
- 🎯 Touch por retângulos exatos (X AND Y)
- 🌐 IP estático configurado (192.168.3.9)
- 🎨 Círculos visuais reposicionados

### v3.2 - Touch Corrigido
**Data:** Dezembro 2024

**Correções:**
- 🖱️ Áreas de toque corrigidas
- ✨ Efeito visual de botão pressionado
- 🎨 Layout vertical otimizado

### v3.1 - Layout Melhorado
**Data:** Dezembro 2024

**Correções:**
- 📱 WiFi movido para rodapé
- 🎨 Estados centralizados
- 🔴🟢 Cores corrigidas (Verde=ABERTO, Vermelho=FECHADO)

### v3.0 - Interface Completa
**Data:** Dezembro 2024

**Novidades:**
- 🖥️ Interface vertical implementada
- 🏠 Integração Home Assistant
- 🖱️ Controle via touchscreen
- ⏰ Data/hora e uptime

### v2.4 - Dimensões Corrigidas
**Data:** Dezembro 2024

**Correções:**
- 📐 Dimensões físicas corretas (height:320, width:240)
- 🖥️ Display landscape funcionando
- 🎨 Cores e coordenadas corretas

### v2.3 - Sem PSRAM
**Data:** Dezembro 2024

**Correções:**
- 🧠 PSRAM removido (NodeMCU-32S não possui)
- 🎨 Paleta 8-bit implementada
- 💾 Otimizações de memória

### v2.2 - SPI Corrigido  
**Data:** Dezembro 2024

**Correções:**
- 📡 Velocidade SPI reduzida para 20MHz
- 🔧 Erros full-duplex resolvidos

### v2.1 - Cores Corrigidas
**Data:** Dezembro 2024

**Correções:**
- 🎨 Configuração de cores corrigida
- 📺 Display funcionando com cores adequadas

---

## 📞 Suporte e Manutenção

### Informações de Contato
- **Projeto:** Monitor Garagem/Portão ESP32
- **Versão:** v3.4
- **Data:** Dezembro 2024

### Backup e Configuração
- **Código fonte:** Salvo como artifact ESPHome
- **Configuração WiFi:** secrets.yaml
- **IP estático:** 192.168.3.9

### Próximas Melhorias Sugeridas
- 📊 Gráficos de histórico de acionamentos
- 🔔 Notificações push
- 🎵 Feedback sonoro
- 📱 Interface web própria
- 🌡️ Sensores de temperatura ambiente

---

**Documentação atualizada em:** Dezembro 2024  
**Próxima revisão:** Quando houver nova versão do código

---

*Esta documentação é atualizada automaticamente a cada nova versão do código ESPHome.*