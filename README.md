# Projeto Final — Eletrônica 1 (ES238, UFPE): Mixer

Mixer com MOSFET: somador inversor → MOSFET em fonte comum operando na lei quadrática → passa-baixa MFB (15,3 kHz) → passa-alta Sallen-Key (5,1 kHz). Com portadora de 10 kHz e modulante de 1 kHz, a saída é um sinal AM com bandas laterais em 9 e 11 kHz (índice de modulação ≈ 0,34 na simulação).

## Estrutura

| Pasta | Conteúdo |
| --- | --- |
| `KicadProjetoFinal/` | Projeto do KiCad (esquemático e configuração do simulador ngspice) |
| `CompontentesExtrasSpice/` | Modelos SPICE: IRF640N (Infineon), TL084 e LM324 (TI), OP07 (TI). Os arquivos `*_kicad.lib` são cópias adaptadas para o ngspice, com o invólucro quádruplo de 14 pinos |
| `Testes/` | Dados exportados do simulador (CSV com `;` como separador e `,` como decimal) |
| `docs/` | Enunciado do projeto |

## Abrir e simular

1. Abrir `KicadProjetoFinal/KicadProjetoFinal.kicad_pro` no KiCad 8 ou superior.
2. Os modelos são referenciados por caminho relativo (`${KIPRJMOD}/../CompontentesExtrasSpice/...`), então a pasta `CompontentesExtrasSpice` precisa ficar ao lado de `KicadProjetoFinal`.
3. Simulador: análise transitória de 5 ms com passo de 0,5 µs. Com o TL084, o ponto de operação DC converge e a simulação já começa em regime permanente.

## Valores atuais (configuração do teste 9)

| Componente | Valor | Função |
| --- | --- | --- |
| R1, R2 | 100 kΩ | Entradas do somador |
| R3 | 1,2 kΩ | Realimentação do somador (ganho −0,012) |
| R5 | 220 Ω | Isolamento da carga capacitiva da porta |
| C1 | 220 nF | Acoplamento da porta |
| R6, R7 | 100 kΩ, 21,1 kΩ | Polarização da porta (VG ≈ −7,818 V, VOV ≈ 58 mV) |
| Q1 | IRF640N | Elemento não linear |
| R8 | 330 Ω | Dreno |
| R9, R10, R11, C2, C3 | 47 kΩ, 11 kΩ, 3 kΩ, 3,3 nF, 1 nF | Passa-baixa MFB |
| C4, C5, R12, R13 | 2,2 nF, 2,2 nF, 10 kΩ, 20 kΩ | Passa-alta Sallen-Key |
| U1 | TL084 | A: somador, B: passa-baixa, C: passa-alta, D: sem uso (seguidor) |
| Alimentação | ±12 V | |

## Registro dos testes

| Arquivo | Configuração | Resultado principal |
| --- | --- | --- |
| `teste.csv` | LM324, polarização original, C1 = 2,2 µF | OP não convergiu; porta partiu de ≈ −4 V (Q1 em triodo) |
| `teste2.csv` | C1 = 220 nF, 10 ms | Estabilização com τ ≈ 4 ms; Q1 atravessando o limiar |
| `teste3.csv` | C1 = 220 nF, 30 ms | Regime permanente: Q1 cortado com o modelo da Infineon, sem mistura |
| `teste4.csv` | R3 = 1 kΩ, dreno = 330 Ω, R7 = 21,1 kΩ | Primeira mistura (9 e 11 kHz no dreno); efeito Miller identificado |
| `teste5.csv` | U1 trocado para TL084 | OP converge; terra virtual melhor; sem oscilação |
| `teste6.csv` | TL084, porta e dreno exportados | Perda de sinal na porta (Miller) confirmada em regime permanente |
| `teste7.csv` | R3 = 1,2 kΩ, saída exportada | AM completo na saída: 751 mV (10 kHz), 132 e 123 mV (9 e 11 kHz) |
| `teste7img.png` | — | Captura do simulador do teste 7 |
| `teste8.csv` | R7 = 21,2 kΩ (VOV ≈ 74 mV) | Bandas laterais caíram ≈ 30% pela realimentação via capacitância porta-dreno |
| `teste9.csv` | R4 (pull-down) retirado, R7 = 21,1 kΩ, V2 com polaridade corrigida | Configuração atual; resultado idêntico ao do teste 7 |

## Observações

- Os modelos SPICE são de seus fabricantes (Texas Instruments, Infineon) e seguem as licenças originais.
- A explicação detalhada do circuito e os relatórios de andamento estão em documentos separados do Claude.
