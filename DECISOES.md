# Desk Buddy — Decisões técnicas e acordos de desenvolvimento

**Base documental:** reorganização do documento mestre rev. 1.6 em 13/11/2026.
**Finalidade:** registrar **por que** soluções foram escolhidas, acordos obrigatórios, problemas recorrentes e protocolo de revisão.
**Estado:** registro das decisões conhecidas; revisão integral do conteúdo documental ainda pendente.

## 1. Acordos obrigatórios de entrega

1. **Qualquer mudança de código**, inclusive bug, correção de compilação ou uma linha, deve ser realizada pelo assistente e entregue como **sketch completo e versionado**, diretamente no chat, em **um único bloco de código copiável**, do primeiro `#include` até o fechamento de `loop()`.
2. **Nunca** pedir ao usuário para inserir, substituir, localizar ou editar trechos manualmente. Não enviar patch, fragmentos ou somente arquivo para download em substituição ao sketch integral.
3. A entrega deve indicar versão, alterações, limites e testes realmente executados.
4. Preservar funcionalidades comprovadas, salvo decisão explícita do usuário. Módulo temporariamente adiado não está descartado.
5. Distinguir rigorosamente **proposto**, **implementado/entregue**, **compilado**, **testado** e **validado em hardware**. Inspeção do código não equivale a compilação; compilação não equivale a teste no CYD.
6. Manter `WIFI_SSID`, `WIFI_PASSWORD` e `ICS_URL` apenas no `config_secrets.h` local. Não divulgar segredos, conteúdo de ICS privado, tokens ou arquivos de fonte.
7. Consultar a especificação, o histórico e o último sketch relevante antes de mudanças importantes; comunicar-se em português brasileiro com orientações práticas.
8. É permitido trabalhar com um **escopo modular temporário** — por exemplo HOME + AGENDA com CLIMA em placeholder — desde que o **sketch entregue esteja completo para esse escopo** e que a meta de integrar módulos validados seja preservada.

## 2. Padrão obrigatório de organização do sketch

Todo novo sketch gerado deverá ser organizado com **seções numeradas, identificáveis e nesta ordem**, adaptando as funções de cada seção sem perder a ordem principal:

```cpp
// 01 - Bibliotecas
// 02 - Configuração do hardware CYD
// 03 - Constantes e estruturas
// 04 - Variáveis de estado
// 05 - Declarações de funções
// 06 - Inicialização do display
// 07 - Controle do touch
// 08 - Gerenciamento das telas
// 09 - Funções da aplicação
// 10 - setup()
// 11 - loop()
```

- Na seção 05, declarar **explicitamente os protótipos de funções**, somente após os tipos/estruturas dos quais dependem.
- A seção 09 pode conter subseções `09.1 AGENDA`, `09.2 ICS`, `09.3 Wi-Fi/NTP`, etc.
- Os títulos de seção facilitam localização, manutenção e revisão; não autorizam entrega fragmentada.
- Motivo técnico adicional: evitar erros ligados à geração automática de protótipos da Arduino IDE, como `AgendaView does not name a type` na v1.9.0.

## 3. Decisões e justificativas

| ID | Decisão | Justificativa / consequência |
|---|---|---|
| D-001 | HOME em grade 2 × 2 e visual atual preservado | Layout aprovado pelo usuário; não introduzir mudanças visuais sem requisito. |
| D-002 | AGENDA prioriza dois eventos, sem paginação | Foco no compromisso atual/próximo e no seguinte; contador resume o restante do dia. |
| D-003 | Ler ICS em tarefa FreeRTOS separada | Na v1.8.0, a leitura síncrona bloqueava touch/Serial por cerca de dois minutos. A v1.8.1 corrigiu e foi aprovada em cinco verificações. |
| D-004 | GPIO36 do touch configurado como `INPUT` | É pino somente entrada sem pull-up interno; `INPUT_PULLUP` gerou aviso/incompatibilidade. |
| D-005 | Manter o calendário no microSD, com arquivo temporário/backup | Permite trabalhar com ICS local e proteger a cópia anterior durante atualizações. |
| D-006 | Segredos e fonte separados do `.ino` | Evitar exposição de credenciais/ICS e manter dependências locais. |
| D-007 | Entregar sempre sketch integral e versionado | Reduz erro humano na aplicação de alterações parciais. |
| D-008 | Pré-validação sistemática antes de cada entrega | Reduz ciclos de compilação, correção e regressão. |
| D-009 | Desenvolver por módulo e integrar posteriormente | Evita transportar código fora do foco (ex.: CLIMA completo durante fase HOME + AGENDA), mas preserva meta de integração. |
| D-010 | Organizar o sketch nas 11 seções padronizadas | Melhora leitura, manutenção, auditoria e previne dificuldades com protótipos Arduino. |
| D-011 | Documentar separadamente comportamento, decisões e versões | Elimina duplicações e diferencia requisito de justificativa e evidência. |
| D-012 | Preservar AGENDA visível durante a atualização ICS, com `Atualização em andamento` em amarelo no rodapé | Evita interromper a leitura da agenda. Exige buffer separado/snapshot consistente e troca controlada somente após sucesso; na ausência de dados iniciais mantém carregamento. **Solicitado em 13/11/2026, ainda não implementado na v1.9.1.** |

As decisões D-009 a D-011 refletem a orientação do usuário adotada nesta reorganização. Não representam validação nova de hardware.

## 4. Revisão geral obrigatória de firmware

**Quando:** antes de entregar **qualquer** versão nova ou código alterado, mesmo que a mudança pareça pequena.

**Procedimento:**
1. Comparar com último sketch e requisitos válidos para identificar regressões.
2. Revisar sistematicamente os itens abaixo e corrigir problemas demonstráveis.
3. Separar no relato de entrega **(A) problemas confirmados corrigidos ou pendentes** e **(B) riscos que dependem de compilação/testes em hardware**.
4. Informar os testes que de fato foram realizados. **Nunca afirmar compilação, segurança ou estabilidade sem evidência.**
5. Entregar o **sketch inteiro em um único bloco**, com versão e comportamento esperado.
6. Quando surgirem novas falhas passíveis de recorrência, adicioná-las permanentemente a esta checklist e registrar a ocorrência em `VERSOES.md`.

### Checklist mínima de pré-validação

- [ ] **Pinos:** TFT/backlight/touch/SD, modos INPUT/OUTPUT, pinos só entrada, pinos de boot e conflitos de GPIO.
- [ ] **Variante exata CYD:** ESP32-2432S028, ILI9341, XPT2046, orientação e peculiaridades do equipamento confirmado.
- [ ] **Bibliotecas/core:** includes disponíveis, APIs compatíveis, configuração TFT_eSPI, core ESP32 3.3.12 e seleção de SD.
- [ ] **RAM e pilhas:** heap livre, sprite 320 × 240 × 8 bits, Strings, buffers ICS, stack FreeRTOS, risco de fragmentação.
- [ ] **Bloqueios:** HTTPS, Wi-Fi, leitura de SD/ICS, delays/loops, tempo de resposta touch e Serial.
- [ ] **Touch:** XPT2046, IRQ36 INPUT, calibração, debounce, transformação X/Y, zonas dos cartões e retorno HOME.
- [ ] **SPI:** barramentos e CS de TFT/SD/touch, clocks, inicialização e concorrência no acesso.
- [ ] **Display e fonte:** driver ILI9341, TFT_BGR, rotação 1, resolução, fontes, clipping e atualização de sprite.
- [ ] **Limpeza/redesenho:** resíduos da tela anterior, sobreposição de textos, limpeza completa e atualização periódica.
- [ ] **Agenda/relógio:** seleção e contador, limites do dia, evento em andamento, modo real/simulado e fuso.
- [ ] **Atualização ICS sem sumir com a AGENDA:** manter snapshot válido, mostrar `Atualização em andamento` em amarelo no rodapé, fazer troca atômica após sucesso, preservar dados se falhar e tratar mudança de data sem mostrar eventos antigos indevidamente.
- [ ] **ICS/SD:** integridade de arquivos, recorrência, fusos, backups, download, erros e reprocessamento.
- [ ] **Segurança:** segredos fora do código compartilhado; reconhecer TLS sem verificação como risco real da implementação conhecida.
- [ ] **Regressão:** preservar HOME, touch, Serial, botões HOME, eventos reais e funções já validadas.
- [ ] **Compilação e hardware:** executar quando o ambiente estiver disponível; caso contrário, marcar explicitamente como **não executados**.

### Regras para classificar os achados

**A — Problema confirmado:** incompatibilidade objetivamente demonstrável, erro de lógica confirmado, erro de compilação observado ou falha reproduzida. Corrigir antes da entrega sempre que possível; relatar qualquer pendência real.

**B — Risco dependente de teste:** preocupação plausível sem reprodução/evidência definitiva, que depende do compilador, da variante física, de SPI real, rede, quantidade de memória ou uso prolongado. Associar teste específico, sem classificá-lo falsamente como defeito confirmado.

## 5. Checklist evolutiva — problemas recorrentes já identificados

| ID | Ocorrência | Verificação incorporada |
|---|---|---|
| R-001 | v1.8.0: ICS síncrono bloqueava interface por ~2 min | Conferir se tarefas de leitura/HTTPS não bloqueiam touch, Serial e desenho; verificar operação em paralelo. |
| R-002 | v1.8.0: `INPUT_PULLUP` no GPIO36 | Conferir GPIO somente entrada e configuração `INPUT`; observar mensagens do core. |
| R-003 | v1.9.0: `AgendaView` não reconhecido na compilação | Estruturas antes de protótipos; declarar funções explicitamente; considerar pré-processador Arduino. |
| R-004 | v1.7.x: TZIDs Windows, exceções e regras ICS não suportadas | Revisar parser parcial, fusos, recorrências, contagem e resultado real do Outlook. |
| R-005 | v1.7.4 CLIMA: HTTP −1 em chamadas meteorológicas | Antes de reintegrar CLIMA, separar erro de rede/TLS/endpoint de falha do display/agenda; testar retorno HTTP. |
| R-006 | Interface CYD compartilha recursos limitados | Rever RAM, CS/SPI e renderização sempre que novos módulos forem integrados. |
| R-007 | HTTPS conhecido com `setInsecure()` | Não confundir download funcional com conexão autenticada por certificado TLS. |
| R-008 | v1.9.1: durante releitura ICS, `drawAgenda()` apresenta carregamento no lugar dos eventos e a tarefa reescreve a lista | Não remover apenas a tela de carregamento; proteger o conjunto publicado por snapshot separado para que touch/renderização não consumam lista em atualização. Solicitação de melhoria pendente de implementação. |

**Evolução:** sempre que um novo problema for identificado **e considerado passível de voltar a ocorrer**, acrescentar nova linha nesta tabela e, quando pertinente, novo item à checklist mínima. Não apagar o histórico de erros.

## 6. Protocolo de aprovação e manutenção

1. Aprovação de uma **decisão** autoriza registrá-la como aprovada; **não** comprova execução em hardware.
2. Confirmações do usuário como “funcionou” devem ser descritas com seu **escopo exato**, sem inferir testes não realizados.
3. Propostas continuam como propostas até aprovação; riscos e falhas ficam registrados até resolução.
4. Depois de decisão explícita, atualizar no GitHub os documentos aplicáveis, se conexão e permissão estiverem disponíveis, e informar o commit confirmado.
5. Anotar data, versão, evidência, pendências e distinção entre problema confirmado e risco.
6. Decisão mais recente do usuário prevalece em conflitos, preservando registro histórico da anterior em `VERSOES.md` ou neste registro.
7. Alterações sensíveis de hardware, segredos, arquitetura ou retirada de comportamento validado exigem confirmação explícita.
8. A publicação inicial do documento mestre em 13/11/2026 foi autorizada como **referência inicial pendente de validação integral de conteúdo**. A presente migração não converte automaticamente todos os detalhes em especificação aprovada.

## 7. Aplicação prática ao ciclo atual

- **Foco:** HOME + AGENDA da linha v1.9.1.
- **CLIMA:** manter cartão/navegação e placeholder, sem trazer implementação meteorológica ao sketch enquanto não trabalharmos nesse módulo.
- **Integração futura:** reunir módulos parcialmente validados e fazer nova revisão sistemática + regressão cruzada.
- **Próxima entrega de firmware:** aplicar o padrão de 11 seções e a checklist; **não alterar o sketch atual apenas por reorganização documental**.
