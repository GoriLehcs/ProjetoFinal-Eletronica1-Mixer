# Projeto Final — Eletrônica 1 (ES238, UFPE): Mixer

Mixer com MOSFET: somador inversor → MOSFET em fonte comum operando na lei quadrática → passa-baixa MFB (15,3 kHz) → passa-alta Sallen-Key (5,1 kHz). Com portadora de 10 kHz e modulante de 1 kHz, a saída é um sinal AM com bandas laterais em 9 e 11 kHz (índice de modulação ≈ 0,34 na simulação).

## Estrutura

| Pasta | Conteúdo |
| --- | --- |
| `KicadProjetoFinal/` | Projeto do KiCad: esquemático, configuração do simulador ngspice e placa de face única |
| `KicadProjetoFinal/Mixer.pretty/` | Biblioteca de footprints do projeto, com ilhas ampliadas para fresagem (registrada em `fp-lib-table`) |
| `KicadProjetoFinal/modelos/` | Modelos SPICE usados na simulação: IRF640N (Infineon) e TL084 (TI, cópia adaptada para o ngspice com invólucro quádruplo de 14 pinos) |
| `CompontentesExtrasSpice/` | Outros modelos e originais dos fabricantes: TL084 e LM324 (TI), OP07 (TI); `lm324_kicad.lib` é a cópia adaptada do LM324 |
| `Testes/` | Dados exportados do simulador (CSV com `;` como separador e `,` como decimal) |
| `docs/` | Enunciado do projeto |

## Abrir e simular

1. Abrir `KicadProjetoFinal/KicadProjetoFinal.kicad_pro` no KiCad 10 (o esquemático e a placa estão no formato do KiCad 10).
2. Os modelos usados ficam em `KicadProjetoFinal/modelos/` e são referenciados por caminho relativo à pasta do projeto (`modelos/...`), então o projeto funciona em qualquer computador.
3. Simulador: análise transitória de 5 ms com passo de 0,5 µs. Com o TL084, o ponto de operação DC converge e a simulação já começa em regime permanente.

## Valores atuais

Numeração após a reanotação de 23/11/2026. A configuração elétrica é a do teste 9; a única mudança é que R7 = 21,1 kΩ passou a ser 20 kΩ fixo + trimpot de 2 kΩ (RV2), ajustado na simulação para 1,1 kΩ.

| Componente | Valor | Função |
| --- | --- | --- |
| R1, R2 | 100 kΩ | Entradas do somador (portadora e voz) |
| R3 | 1,2 kΩ | Realimentação do somador (ganho −0,012) |
| R4 | 220 Ω | Isolamento da carga capacitiva da porta |
| C3 | 220 nF | Acoplamento da porta |
| R5, R6 + RV2 | 100 kΩ, 20 kΩ + 2 kΩ (≈ 1,1 kΩ) | Polarização da porta (VG ≈ −7,818 V, VOV ≈ 58 mV) |
| Q1 | IRF640N | Elemento não linear |
| R7 | 330 Ω | Dreno |
| R8, R9, R10, C6, C7 | 47 kΩ, 11 kΩ, 3 kΩ, 3,3 nF, 1 nF | Passa-baixa MFB |
| C9, C10, R11, R12 | 2,2 nF, 2,2 nF, 10 kΩ, 20 kΩ | Passa-alta Sallen-Key |
| U1 | TL084 | A: somador, B: passa-baixa, C: passa-alta, D: sem uso (seguidor) |
| RV1 | 100 kΩ | Nível da voz (divisor entre o microfone e R2) |
| U2, C5, C8 | L7805, 330 nF, 100 nF | 5 V para o módulo de microfone MAX4466 |
| C1, C2, C4 | 100 nF | Desacoplamento: +12 V e −12 V junto ao TL084; +12 V junto a R7 |
| J1 / J2 / J3 / J4 | KF301 2 vias / barra 1×3 / KF301 3 vias / KF301 2 vias | Portadora / microfone (VCC, GND, OUT) / alimentação (+12 V, GND, −12 V) / saída |
| Alimentação | ±12 V | |

## Placa (face única, fresagem CNC)

- Placa de 90 × 62 mm, cobre só na face inferior (B.Cu), com quatro furos M3 de 3,2 mm nos cantos.
- Regras de projeto: trilha de 0,8 mm (sinal) e 1,2 mm (alimentação), isolação de 0,5 mm, 1 mm de distância até a borda.
- Footprints da biblioteca `Mixer`: ilhas de 2,2 a 3,0 mm; nos componentes com passo de 2,54 mm as ilhas são ovais, alongadas na direção perpendicular à fileira.
- Roteamento feito com o Freerouting a partir de `KicadProjetoFinal_1face.dsn` (só a camada B.Cu; GND deixado para o plano de cobre). `KicadProjetoFinal.dsn` é a exportação original do KiCad.
- GND por plano de cobre na face inferior, com ligação sólida dos pads (sem alívio térmico, para não deixar pontes estreitas na fresagem).
- **Um jumper de fio** (trilha em F.Cu entre dois furos de GND, perto de J3): liga ao plano a ilha do pino de GND de J3, cercada pelas trilhas de +12 V e −12 V.
- DRC (`DRC.rpt`): nenhum erro, nenhum pad desconectado; restam só avisos de serigrafia, que a fresagem não usa.
- Na montagem, os bornes devem ficar com a entrada do fio voltada para a borda da placa.

## Registro dos testes

Os testes usam a numeração anterior à reanotação (por exemplo, R7 = 21,1 kΩ e R8 = 330 Ω no dreno).

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
