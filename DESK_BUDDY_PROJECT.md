# DESK BUDDY — Documento mestre do projeto

**Revisão:** 1.4 — 13/11/2026  
**Status:** REFERÊNCIA INICIAL — PENDENTE DE VALIDAÇÃO DO CONTEÚDO  
**Último firmware com execução confirmada:** v1.8.1 (cinco testes aprovados pelo usuário)  
**Firmware mais recente testado em hardware:** v1.9.1 — exibição da AGENDA confirmada pelo usuário; demais testes da v1.9.1 pendentes

> Documento central de acordos, especificações, decisões, evidências e pendências. A aprovação de sua publicação não significa validação técnica de todas as informações.

## 1. Acordos obrigatórios

1. **Toda alteração que exija ajuste no código — inclusive correção de compilação, bug, mudança mínima ou adição de uma linha — deve ser feita pelo assistente e entregue como SKETCH INTEIRO, diretamente no chat, em um único bloco de código copiável**, desde o primeiro `#include` até o fechamento de `loop()`. **Nunca solicitar que o usuário localize, insira, substitua ou edite trechos manualmente; nunca entregar somente patch, fragmento, instrução de edição ou arquivo para download em substituição ao código integral.**
2. Identificar a versão no código e na resposta; explicar alterações e testes.
3. Manter `WIFI_SSID`, `WIFI_PASSWORD` e `ICS_URL` exclusivamente no `config_secrets.h` local. Nunca publicar segredos no chat nem no GitHub.
4. Preservar funcionalidades validadas, salvo decisão explícita. Funcionalidade adiada não é descartada.
5. Distinguir **proposto**, **entregue**, **compilado**, **testado** e **validado em hardware**. Não presumir validação.
6. Desenvolver por telas/módulos, mantendo cada entrega do `.ino` completa.
7. Consultar este documento e o último sketch relevante antes de mudanças importantes.
8. Comunicar-se em português brasileiro, com instruções práticas.

## 2. Objetivo e arquitetura

Painel de mesa baseado em ESP32 CYD 2,8", com HOME inicial e navegação por cartões.

- **HOME:** relógio grande, data, Wi-Fi e quatro cartões em grade 2 × 2; visual tecnológico escuro com contornos luminosos e cantos recortados, inspirado na referência visual do usuário.
- **AGENDA:** reuniões reais do Outlook ICS salvo no microSD.
- **CLIMA:** placeholder "EM BREVE"; remover integração Open-Meteo do firmware atual até retomada do módulo.
- **Cartões 3 e 4:** `+` reservados.
- **Navegação:** touchscreen e retorno à HOME; manter comandos Serial durante testes.

## 3. Hardware e ambiente

| Item | Configuração conhecida |
|---|---|
| Placa | ESP32-2432S028 / CYD 2,8" |
| Display | ILI9341, 320 × 240, rotação 1 |
| Gráficos | TFT_eSPI, TFT_eSprite 320 × 240, 8 bits |
| TFT_eSPI | ILI9341_2_DRIVER, TFT_BGR, HSPI, 27 MHz |
| Backlight | GPIO 21 |
| SD | ~2 GB, leitura e escrita testadas |
| SD SPI | VSPI: SCK 18, MISO 19, MOSI 23, CS 5, 4 MHz |
| Touch | XPT2046; pinos usados e funcionalmente testados para navegação: CLK 25, MOSI 32, MISO 39, CS 33, IRQ 36; calibração fina pendente |
| Arduino | ESP32 core 3.3.12 observado; ESP32 Dev Module; Flash DIO 40 MHz; PSRAM off |
| Serial | 115200 baud |
| Fonte | NotoSansBold15.h local |

A orientação e calibração do touchscreen ainda precisam ser confirmadas em hardware.

## 4. Arquivos e segurança

```text
Desk_Buddy/
├── Desk_Buddy.ino
├── config_secrets.h      # PRIVADO; nunca versionar
└── NotoSansBold15.h
```

No microSD, o calendário usa `/agenda.ics`, `/agenda.tmp` e `/agenda.bak`. O download temporário é verificado pelo início `BEGIN:VCALENDAR` antes da substituição. A URL publicada do Outlook é privada.

**GitHub:** `victorbeiral/Desk-Buddy`, branch `main`, repositório **privado**, confirmado pela integração em 13/11/2026. Não publicar `config_secrets.h`, arquivos ICS reais, tokens, credenciais ou logs privados. Este documento não substitui o backup do sketch validado.

## 5. Interface e comportamento

### HOME (v1.8.1, navegação validada)
- Resolução 320 × 240 paisagem; fundo azul-escuro; tipografia clara; destaques ciano, verde, amarelo e rosa.
- Hora grande, data, identificação do sistema e status real de Wi-Fi.
- Cartões AGENDA, CLIMA, +, +; toque abre a página correspondente.
- Fidelidade visual e áreas de toque dependem de foto/teste no dispositivo.

### AGENDA

**Nova especificação implementada na v1.9.1, exibição confirmada pelo usuário:** mostrar no máximo dois eventos, sem paginação. Se houver evento em andamento, destacá-lo e mostrar o próximo futuro; caso contrário, mostrar os dois próximos futuros. Abaixo, exibir `+N eventos hoje` para a quantidade de eventos restantes depois dos exibidos; não contar eventos encerrados nem os dois visíveis. Se houver somente um evento, mostrar um cartão; se não houver, indicar agenda livre. Recalcular com a passagem do tempo. Preservar HOME v1.8.1 sem ajustes e manter leitura ICS em segundo plano e retorno HOME por touch. Em caso de eventos simultâneos, a política de seleção ainda requer validação de uso.

- Na v1.8.1, quatro reuniões por página; substituição por visão de dois eventos aprovada para próxima implementação.
- Destacar reunião em andamento apenas se a hora simulada/real estiver dentro do intervalo.
- Truncar visualmente títulos longos sem alterar o ICS.
- Simulação inicial de testes: **segunda-feira, 16/11/2026 às 08:00**; permitir retorno ao relógio real.
- Não reintroduzir compromissos fictícios.

### CLIMA (adiado)
- Módulo anterior usava Open-Meteo para Paulínia-SP (−22.7542, −47.1488), condição, temperatura, próximas quatro horas e mínima/máxima móvel 24h.
- Na linha v1.8.x, **não realizar chamadas à Open-Meteo**; manter somente placeholder.

## 6. Rede, horário e ICS

- Wi-Fi por constantes do arquivo privado.
- NTP: `BRT3`, UTC−3 para São Paulo nas datas de teste.
- Outlook ICS por HTTPS, salvo no microSD. O código anterior usava `WiFiClientSecure.setInsecure()`, que **não valida certificado TLS**; melhoria futura.
- ICS de ~2 MB, contendo eventos simples, recorrências, exceções, exclusões e TZIDs Windows.
- Arduino ESP32 core 3.3.12 não ofereceu `timegm()`; solução anterior utilizou `daysFromCivil()` e `mktime()`.
- TZIDs observados: E. South America Standard Time, Central Standard Time, India Standard Time, GMT Standard Time, Eastern Standard Time, Pacific SA Standard Time, SA Western Standard Time.
- Parser **não é RFC 5545 completo**. Limitações em recorrências mensais/anuais, alguns BY*, RDATE, COUNT, EXDATE, exceções complexas, RANGE=THISANDFUTURE, VTIMEZONE e transições DST. Dias inteiros e fusos históricos requerem validação.
- Conferir a completude dos eventos com o Outlook; seis reuniões corretas não comprovam cobertura de todos os casos.

## 7. Histórico de versões e evidências

| Versão | Estado | Resultado |
|---|---|---|
| v1.7.2 | Diagnóstico validado | Download HTTP 200; 769 VEVENT pareados; incompatibilidade de TZIDs Windows identificada |
| v1.7.3 | Validada em hardware | ICS 2.024.858 bytes; 770 VEVENT; 484 simples; 471 datas convertidas; 11 não suportadas; 0 inconsistências estruturais; 56 linhas truncadas |
| v1.7.4 | **Última execução confirmada** | Simulação 16/11/2026 08:00; seis reuniões reais, cinco ocorrências geradas, três regras não suportadas; usuário confirmou horários corretos |
| v1.7.4 CLIMA | Problema observado | Requisições de clima retornaram HTTP -1 repetidamente; causa não confirmada |
| v1.8.0 | **Parcialmente validada em hardware (13/11/2026)** | HOME carregou; touch abriu AGENDA e CLIMA; retorno à HOME via Serial; SD e Wi-Fi OK; 770 VEVENT e seis reuniões corretas. Leitura ICS bloqueia touch/Serial por cerca de 2 min. Aviso GPIO36 INPUT_PULLUP. Sketch completo posteriormente entregue no chat. |
| v1.8.1 | **Validada nos cinco testes informados pelo usuário (13/11/2026)** | Inicialização OK; touch durante leitura ICS OK; botões HOME OK; resultado da agenda OK; releitura OK. ICS em tarefa FreeRTOS; GPIO36 configurado como INPUT. Não há log detalhado de tempo, heap ou testes prolongados; não presumir que todos os cenários estão cobertos. |
| v1.9.0 | **Erro de compilação relatado** | `AgendaView does not name a type`; versão substituída pela v1.9.1. |
| v1.9.1 | **Exibição validada pelo usuário (13/11/2026)** | Usuário informou "esta mostrando corretamente" após receber o sketch integral corrigido. Confirma exibição desejada da AGENDA; não implica testes independentes de touch, releitura, virada de dia ou estabilidade prolongada nesta versão. |

### Reuniões confirmadas em 16/11/2026

| Início | Fim | Título |
|---|---|---|
| 08:30 | 09:00 | STP - FIN AP Operate team daily touchpoint call |
| 09:30 | 10:00 | Cross-MAVEFSSRE-CTSSG DSU //Communication Subject to NDA |
| 10:00 | 10:30 | Daily - Data Model |
| 13:30 | 14:00 | Treasure Connect - daily |
| 14:00 | 15:00 | Handoff Demandas Luiz |
| 15:30 | 16:15 | Blackline import activities sync |

Os títulos são dados privados de agenda; repositório agora privado, mas evitar publicação ou compartilhamento indevido.

## 8. Validação v1.8.1 e próximos testes

- [x] HOME aparece ao iniciar.
- [ ] Hora/data e status Wi-Fi corretos.
- [x] AGENDA abre e mostra reuniões reais.
- [x] Simulação exibe as seis reuniões confirmadas.
- [ ] Paginação da agenda funciona.
- [x] CLIMA abre placeholder, sem Open-Meteo (navegação confirmada; ausência de chamadas pelo código).
- [ ] Ambos os cartões + abrem placeholders.
- [x] Retorno à HOME por touch funciona na v1.8.1.
- [x] Touch responde nos cartões AGENDA e CLIMA; calibração fina e HOME por touch pendentes.
- [ ] Wi-Fi, NTP, SD e download ICS continuam funcionais.
- [ ] Estabilidade prolongada sem resets ou travamentos (cinco testes funcionais OK, sem ensaio prolongado).

**Problemas da v1.8.0 corrigidos e testados na v1.8.1:** leitura ICS em tarefa FreeRTOS separada; touch e Serial continuam respondendo; indicador de atualização; retorno à HOME pelo toque superior e inferior; GPIO36 alterado para `INPUT`. O usuário marcou os cinco testes como OK. **Pendências:** calibração fina do touch, testes de estabilidade prolongada, paginação, cartões +, atualização HTTPS real e eventual concorrência entre download SD e outras tarefas.

Após validar: ajustar touch/layout; melhorar recorrências e fusos; reintegrar CLIMA isoladamente; desenvolver novos cartões.

## 9. Protocolo de aprovação e atualização

1. Aprovação explícita de uma decisão permite registrá-la como **aprovada**, mas não como **validada em hardware**.
2. Expressões como "funcionou" acompanhadas de evidência de teste podem registrar validação, indicando o que foi observado.
3. Propostas em discussão permanecem propostas; falhas permanecem pendentes.
4. Após cada aprovação inequívoca, atualizar este documento no GitHub **quando a integração e a permissão de escrita estiverem disponíveis** e confirmar o commit na conversa. Não afirmar atualização sem commit confirmado.
5. Registrar revisão, data, decisão, evidência e pendências; preservar acordos anteriores salvo substituição explícita.
6. Quando houver conflito, a decisão mais recente do usuário prevalece e o documento deve ser corrigido.
7. Mudanças sensíveis de hardware, segredos, arquitetura ou remoção de recursos validados exigem confirmação explícita.
8. A publicação inicial desta revisão foi autorizada pelo usuário em 13/11/2026, **como referência inicial pendente de validação de conteúdo**.

## 10. Itens ainda pendentes de aprovação técnica

- [x] HOME v1.8.1 aprovada visualmente sem mudanças.
- [x] Novo comportamento de AGENDA com dois eventos e contador aprovado como requisito; implementação concluída e exibição confirmada; testes adicionais pendentes.

- [ ] Validar o conteúdo integral desta documentação como especificação oficial.
- [ ] Confirmar a fidelidade da HOME com quatro cartões após teste visual.
- [ ] Confirmar se simulação 16/11/2026 08:00 permanece padrão durante testes.
- [ ] Confirmar método de retorno à HOME após teste do touch.

---

**Regra de ouro:** qualquer mudança de código, por menor que seja, é realizada pelo assistente e entregue como **sketch integral e versionado, diretamente no chat, em um único bloco**. Não passar trechos ou instruções para edição manual. Só o teste em hardware confirma funcionamento.
