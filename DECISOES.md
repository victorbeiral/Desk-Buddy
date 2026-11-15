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
| D-012 | Preservar AGENDA visível durante releitura ICS, com um único indicador amarelo | Evita interromper a consulta dos eventos. Implementado e confirmado pelo usuário na v1.9.2.1; exibição incorreta da frase acentuada motivou substituição por `Atualizando agenda` na v1.9.2.2 (a validar). |
| D-013 | Remover somente o botão HOME superior da AGENDA, preservando o inferior | Pedido do usuário após inspeção visual da v1.9.2.1; evita duplicidade visual. O toque na antiga área superior também deixa de navegar na AGENDA; outras telas mantêm o comportamento existente. **Confirmado pelo usuário na v1.9.2.2.** |
| D-014 | Ativar modo real automático de agenda na v1.9.3 | A v1.9.3.1 comprovou Wi-Fi, modo real, leitura local e tentativa automática por HTTPS no CYD. **HTTP=-1:** falha online a diagnosticar. |
| D-015 | Distinguir agenda local de falha online e preservar backup | Na v1.9.3.2, preservação do ICS local foi comprovada por logs durante falha HTTPS; diagnóstico determinou `SSL - Memory allocation failed`. |
| D-016 | Reduzir memória gráfica para tornar possível o handshake TLS no CYD sem PSRAM | v1.9.3.3 usa sprite indexado 4 bpp (38.400 B, em vez de 76.800 B), paleta de 9 cores e sobreposição de fonte Noto em RGB565 diretamente no TFT. Libera Strings dos UIDs de exceção ao fim do processamento. Novo código pendente de compilação real/validação visual e HTTPS. |
| D-017 | Corrigir inversão Y detectada após remoção do HOME superior | Log de toques físicos no HOME inferior trouxe y=10–24 (topo), indicativo de eixo Y invertido. v1.9.3.3 muda `touchInvertY=false` globalmente; requer teste dos quatro cartões HOME e botão inferior para evitar regressões. |

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
- [ ] **Bibliotecas/core:** includes disponíveis, APIs compatíveis, configuração TFT_eSPI, core ESP32 3.3.12 e seleção de SD. Conferir **argumentos padrão apenas uma vez**, preferencialmente nos protótipos; evitar nomes genéricos conflitantes e **vetores locais `static` em funções de uma única linha** (falhas de v1.9.2 e v1.9.3); quando possível, executar compilação real Arduino em vez de confiar somente em stubs.
- [ ] **RAM e pilhas:** heap livre, sprite (8 ou 4 bpp), Strings, buffers ICS, stack FreeRTOS, risco de fragmentação. Para TLS, medir **heap total e maior bloco contínuo de 8 bits**; `-32512` evidenciou alocação insuficiente na v1.9.3.2.
- [ ] **Bloqueios:** HTTPS, Wi-Fi, leitura de SD/ICS, delays/loops, tempo de resposta touch e Serial.
- [ ] **Touch:** XPT2046, IRQ36 INPUT, calibração, debounce, transformação X/Y, zonas dos cartões e retorno HOME. Toda remoção de botão deve remover também sua área de toque, sem afetar as outras telas.
- [ ] **SPI:** barramentos e CS de TFT/SD/touch, clocks, inicialização e concorrência no acesso.
- [ ] **Display e fonte:** driver ILI9341, TFT_BGR, rotação 1, resolução, fontes, clipping e atualização de sprite. Verificar suporte real a caracteres UTF-8/acento nas fontes selecionadas; conferir largura disponível, sem truncar indicador de estado.
- [ ] **Limpeza/redesenho:** resíduos da tela anterior, sobreposição de textos, limpeza completa e atualização periódica.
- [ ] **Agenda/relógio:** seleção e contador, limites do dia, evento em andamento, modo real/simulado e fuso. No modo de produção, **iniciar com simulação desativada**, aguardar NTP válido e refazer a leitura após virada do dia.
- [ ] **Atualização ICS sem sumir com a AGENDA:** manter snapshot válido, mostrar indicador amarelo único `Atualizando agenda` no rodapé da AGENDA, fazer troca controlada após sucesso, preservar dados se falhar e tratar mudança de data sem mostrar eventos antigos indevidamente.
- [ ] **ICS/SD:** integridade de arquivos, recorrência, fusos, backups, download, erros e reprocessamento. **Não restaurar backup de operação anterior se o download presente falhou antes de instalar novo ICS.** Verificar bootstrap local→HTTPS, atualização automática periódica, intervalo de retry e recuperação `.bak` caso o arquivo novo não seja parseável.
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
| R-009 | Pré-validação v1.9.2: argumento padrão declarado simultaneamente no protótipo e na definição de `touchRead` | Declarar valores padrão apenas no protótipo; revisão de sintaxe C++ com interfaces simuladas detectou o erro e a correção passou sem avisos. Compilação real Arduino ainda pendente. |
| R-010 | Compilação Arduino v1.9.2 e v1.9.3: funções `dim()` e `daysInMonth()` foram apontadas como `extern` e depois `static` | A mudança de nome não impediu a recorrência. Verificar também **definições compactas com vetor local `static`** e pré-processamento Arduino; v1.9.3.1 substituiu o vetor por `switch` e separou o protótipo. **Causa exata e correção real pendentes de confirmação na Arduino IDE**. |
| R-008 | v1.9.1: durante releitura ICS, `drawAgenda()` apresenta carregamento no lugar dos eventos e a tarefa reescreve a lista | A v1.9.2.1 implementou dois buffers de eventos, atualização somente ao completar; **preservação visual confirmada no CYD pelo usuário**. |

| R-011 | v1.9.2.1: texto amarelo de atualização com caracteres ausentes/corrompidos (`Atualizao...`) na tela | Verificar caracteres suportados pela fonte TFT_eSPI e largura real do rodapé; usar expressão legível e adequada (`Atualizando agenda`), sem UTF-8 não suportado; confirmar no CYD. A v1.9.2.2 implementa o ajuste, teste pendente. |

| R-012 | Risco de produção: firmware inicia no horário simulado ou lê apenas ICS local, impedindo agenda real atualizada | Verificar `simulation=false` no boot, sincronização NTP, carregamento local inicial, download HTTPS automático, atualização periódica e virada de dia. Aplicado em v1.9.3, testes reais pendentes. |

| R-013 | v1.9.3.1: HTTP=-1 ao acessar ICS Outlook | No core ESP32 3.3.12, -1 significa falha de conexão; usar erro HTTP, TLS, DNS e TCP, sem divulgar URL. v1.9.3.2 acrescenta diagnósticos, ainda sem confirmação de causa. |
| R-014 | v1.9.3.1: restauração indiscriminada de `.bak` após falha de HTTPS | Restrição na v1.9.3.2: restaurar somente arquivo substituído na mesma tentativa de atualização; validar em hardware. |

| R-015 | v1.9.3.2: `HTTP=-1`, TLS `-32512 SSL - Memory allocation failed`; DNS e TCP 443 OK | `HTTP=-1` é efeito de falha TLS por falta/fragmentação de memória, comprovado pelo erro de alocação. v1.9.3.3 reduz sprite de 8 para 4 bits, descarrega UIDs temporários de exceções e mede o maior bloco contíguo. HTTPS ainda pendente de sucesso em hardware. |
| R-016 | v1.9.3.2: tentativa de usar HOME inferior gera toques em x~260, y~15 | `touchInvertY=true` converte o toque inferior para faixa superior, que agora é desativada. Ajuste proposto: `touchInvertY=false`, testar regressão de HOME/AGENDA/CLIMA/+ no CYD. |

| R-017 | v1.9.3.3: erro de compilação `expected ';' before ')'` em `tzOffset()` | Revisão humana falhou ao deixar parêntese excedente `off=(v>=a+3600&&v<b)?60:0);`. Corrigir para `off=(v>=a+3600&&v<b)?60:0;` na v1.9.3.4; exigir checagem sintática do **artefato exato entregue**, não apenas de arquivo anterior ou diferente. Compilação Arduino pendente. |

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

- **Foco:** HOME + AGENDA, linha v1.9.2.2 validada visualmente; v1.9.3 entregue para operação real automática e avaliação prática a partir de 16/11/2026.
- **CLIMA:** manter cartão/navegação e placeholder, sem trazer implementação meteorológica ao sketch enquanto não trabalharmos nesse módulo.
- **Integração futura:** reunir módulos parcialmente validados e fazer nova revisão sistemática + regressão cruzada.
- **Próxima entrega de firmware:** aplicar o padrão de 11 seções e a checklist; **não alterar o sketch atual apenas por reorganização documental**.
