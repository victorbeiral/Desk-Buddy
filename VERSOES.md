# Desk Buddy — Histórico de versões, falhas e testes

**Base documental:** histórico preservado do documento mestre rev. 1.6 (13/11/2026).
**Finalidade:** registrar **o que mudou**, defeitos conhecidos, correções e **exatamente quais testes foram aprovados**.
**Estado:** evidências relatadas pelo usuário e resultados documentados; sem presunção de ensaio adicional.

## 1. Convenção de status

- **Proposto:** requisito discutido, sem implementação entregue.
- **Entregue:** sketch fornecido, sem comprovação automática de compilação.
- **Erro confirmado:** erro de compilação ou comportamento observado e descrito.
- **Validado parcialmente em hardware:** algumas funções comprovadas, outras pendentes.
- **Validado nos testes especificados:** todos os testes explicitamente informados como OK; não significa cobertura total.
- **Pendente:** verificação ainda não executada ou não informada.

## 2. Linha do tempo do firmware

| Versão | Situação registrada | Mudanças, resultados e observações |
|---|---|---|
| v1.7.2 | Diagnóstico validado | Download HTTP 200; 769 VEVENT pareados; incompatibilidade de TZIDs Windows identificada. |
| v1.7.3 | Validada em hardware para diagnóstico ICS | Arquivo 2.024.858 bytes; 770 VEVENT; 484 simples; 471 datas convertidas; 11 não suportadas; 0 inconsistências estruturais; 56 linhas truncadas. |
| v1.7.4 AGENDA | Horários confirmados pelo usuário | Simulação segunda-feira 16/11/2026 08:00; seis reuniões reais; cinco ocorrências geradas e três regras não suportadas. |
| v1.7.4 CLIMA | Falha observada, causa não confirmada | Requisições do módulo clima retornavam HTTP −1 repetidamente. |
| v1.8.0 | Parcialmente validada em hardware (13/11/2026) | HOME renderizada; touch abria AGENDA e CLIMA; retorno à HOME via Serial; SD/Wi-Fi OK, 770 VEVENT e seis reuniões corretas. Leitura ICS síncrona bloqueava touch e Serial por cerca de dois minutos. Aviso de GPIO36 INPUT_PULLUP. |
| v1.8.1 | **Cinco testes funcionais aprovados pelo usuário (13/11/2026)** | ICS passou a tarefa FreeRTOS, interface não bloqueante, retorno HOME por touch, GPIO36 como INPUT. Inicialização OK; touch durante leitura OK; botões HOME OK; resultado da agenda OK; releitura OK. |
| v1.9.0 | **Erro de compilação confirmado** | Compilador apontou `AgendaView does not name a type`; tipo personalizado e geração automática de protótipos Arduino foram identificados como causa provável. Não considerar essa versão validada. |
| v1.9.1 | **Interface, Testes 01 e 02 validados parcialmente em hardware (13/11/2026)** | Substituição de `AgendaView` por `selectAgenda(time_t,int&,int&,int&,bool&)`; AGENDA com dois eventos, destaque de andamento e contador, sem paginação. Usuário confirmou “esta mostrando corretamente”, touch e botões HOME funcionando. **Teste 01: “tudo ok”** para releitura ICS local com comando `E`, touch durante a leitura, navegação HOME/CLIMA e agenda final. **Teste 02: “tudo ok”** para hora real, atualização por minuto, AGENDA real e retorno ao modo simulado. Download HTTPS, virada de dia, troca entre eventos e estabilidade prolongada ainda pendentes. Foi solicitada melhoria de UX: manter dados visíveis durante releitura ICS; ainda não implementada. |
| v1.9.2 | **Sketch preparado para entrega; compilação real e testes CYD pendentes (13/11/2026)** | Scanner ICS em buffers de 4096 bytes, primeira passagem seletiva para UID/RECURRENCE-ID, medições `[PERF]` por passagem (tempo, SD, bytes, blocos, linhas), dois bancos de eventos com troca após sucesso, manutenção da AGENDA visível e indicador único `Atualização em andamento`. Mantém HOME, touch, retorno HOME, comandos, simulação e parser parcial. Verificação de sintaxe C++ com bibliotecas simuladas aprovada sem avisos; **não equivale a compilação Arduino/ESP32 nem ensaio em hardware**. |

**Referência atual validada em hardware para a interface:** v1.9.1. **v1.9.2:** entregue para nova validação, sem confirmação de compilação ou hardware.
**Última versão com cinco testes funcionais aprovados em conjunto:** v1.8.1.
**Estado do `.ino` no GitHub:** o firmware mais recente foi entregue integralmente no chat; **não presumir arquivo de firmware commitado no repositório**.

## 3. Cenário de teste da AGENDA — 16/11/2026 às 08:00

Seis reuniões reais confirmadas historicamente no calendário de teste:

| Início | Fim | Título |
|---|---|---|
| 08:30 | 09:00 | STP - FIN AP Operate team daily touchpoint call |
| 09:30 | 10:00 | Cross-MAVEFSSRE-CTSSG DSU //Communication Subject to NDA |
| 10:00 | 10:30 | Daily - Data Model |
| 13:30 | 14:00 | Treasure Connect - daily |
| 14:00 | 15:00 | Handoff Demandas Luiz |
| 15:30 | 16:15 | Blackline import activities sync |

**Dados confidenciais de agenda:** repositório privado; não publicar estes títulos fora do contexto autorizado.

Na regra introduzida na v1.9.1, a seleção às 08:00 deve mostrar os dois eventos das 08:30 e 09:30 e o contador **+4 eventos hoje**. O usuário confirmou a exibição correta; não há log detalhado anexado sobre todos os casos de passagem do tempo.

## 4. Matriz de validação por versão

| Verificação | v1.8.1 | v1.9.1 |
|---|---|---|
| Inicialização | **OK informado** | Executada implicitamente para exibir interface; não registrada como ensaio separado |
| HOME e AGENDA na tela | **OK informado** | **OK: exibição correta informada** |
| Dois eventos e contador, sem paginação | Não se aplica (tela anterior) | **OK: exibição correta informada** |
| Touch durante leitura ICS | **OK informado** | **OK informado: Teste 01, releitura ICS local** |
| Navegação por touch | **OK informado** | **OK informado** |
| Botões de retorno HOME | **OK informado** | **OK informado** |
| Resultado da agenda | **OK informado** | **OK: exibição informada** |
| Releitura ICS | **OK informado** | **OK informado: Teste 01 com comando `E`** |
| Download HTTPS real | Não confirmado como ensaio independente | **Pendente** |
| Modo real, avanço por minuto e retorno à simulação | **Não registrado aqui** | **OK informado: Teste 02** |
| Virada de dia e transição entre eventos | **Pendente** | **Pendente (não cobertos pelo Teste 02)** |
| Calibração fina touch e placeholders + | **Pendente** | **Pendente** |
| Tempo de execução prolongado, heap e resets | **Pendente** | **Pendente** |
| Reintegração CLIMA completo | Adiada | **Adiada** |

Na v1.9.1, o usuário informou **Teste 01 integralmente OK**, incluindo touch durante a releitura **local** ICS; o ensaio não confirma download HTTPS real nem toda forma de concorrência.

## 5. Histórico de problemas e classificação

### Problemas confirmados

- **v1.8.0 — bloqueio de interface:** leitura ICS bloqueava touch/Serial por aproximadamente dois minutos. **Corrigido e aprovado em testes informados da v1.8.1**, ao passar a leitura para tarefa FreeRTOS.
- **v1.8.0 — configuração GPIO36:** uso de `INPUT_PULLUP` incompatível com IRQ36 na configuração observada. Ajustado para `INPUT` na v1.8.1 e navegação validada.
- **v1.9.0 — erro de compilação:** `AgendaView does not name a type`. Sketch v1.9.1 foi entregue com assinatura baseada em parâmetros de saída; a v1.9.1 executou no equipamento, segundo confirmação de exibição.
- **Linha de firmware conhecida — TLS sem validação:** uso de `setInsecure()` desativa verificação de certificado; **questão de segurança identificada**, ainda não corrigida na linha conhecida.

### Falhas observadas com causa não determinada

- **v1.7.4 CLIMA:** HTTP −1 nas consultas meteorológicas. A causa não foi estabelecida; CLIMA completo foi adiado.
- **ICS complexo:** 11 datas não suportadas, 56 linhas truncadas no diagnóstico v1.7.3 e três regras não suportadas no cenário v1.7.4; valores históricos, não extrapolar automaticamente para todos os calendários.

### Riscos que dependem de teste

- Uso prolongado de RAM/heap e estabilidade do ESP32.
- Conflitos ou contenção SPI/SD/touch em operações simultâneas ou após futura integração de módulos.
- Troca de dia, alterações de horário/fuso e eventos simultâneos.
- Download HTTPS real e preservação do ICS após falha.
- Calendários com recorrências/VTIMEZONE fora do subconjunto suportado.

A checklist preventiva e as razões para escolhas técnicas são mantidas em [DECISOES.md](DECISOES.md).

## 6. Pendências abertas e plano de próxima validação

- [x] Confirmar modo real, hora e data, mudança de minuto e retorno à simulação na v1.9.1 (**Teste 02, usuário: “tudo ok”**). Isso não comprova precisão prolongada de NTP nem virada de dia.
- [x] Revalidar explicitamente releitura ICS **local** na v1.9.1 (Teste 01, comando `E`; usuário: “tudo ok”).
- [x] Revalidar touch e navegação durante **releitura ICS local** na v1.9.1 (Teste 01, usuário: “tudo ok”).
- [ ] Revalidar especificamente responsividade Serial durante operação e touch/Serial durante **download HTTPS real** na v1.9.1.
- [ ] Testar **troca entre eventos** quando reunião começa/termina (não abrangida pela simples virada de minuto do Teste 02).
- [ ] Testar virada de dia, casos de agenda vazia, evento único e sobreposições.
- [ ] **Validar no CYD a implementação v1.9.2 de manutenção da AGENDA durante releitura/download ICS**: manter snapshot anterior, indicador `Atualização em andamento` em amarelo no rodapé, atualização atômica ao concluir e fallback sem perda do snapshot em caso de falha.
- [ ] Testar os dois cartões +, CLIMA placeholder e limpeza de tela sem artefatos.
- [ ] Testar estabilidade prolongada e coletar heap/reset quando possível.
- [ ] Corrigir/validar TLS com verificação de certificado antes de considerar o download seguro.
- [ ] Testar integração final dos módulos quando forem reunidos.

**Nota de rastreabilidade:** a v1.8.1 é referência para as cinco verificações funcionais informadas; a v1.9.1 é a referência **mais recente para a interface**, com escopo de validação explicitamente limitado.

### Registro de teste — v1.9.1, Teste 01 (13/11/2026)

**Procedimento solicitado:** na AGENDA, executar `E` pelo Monitor Serial (releitura do ICS armazenado no microSD); durante `ATUALIZANDO`, navegar por touch para HOME, CLIMA e AGENDA; após conclusão, conferir dois eventos, contador `+4 eventos hoje` e saída de processamento no Serial.

**Resposta do usuário:** “teste 01 tudo ok”. **Conclusão:** aprovação conjunta dos itens solicitados no Teste 01: releitura ICS local, touch/navegação durante leitura e apresentação final da agenda. Não há logs anexados nem medições de duração/heap. Não extrapolar o resultado para o comando `I`, download HTTPS, virada de dia ou estabilidade prolongada.

### Registro de teste — v1.9.1, Teste 02 (13/11/2026)

**Procedimento solicitado:** acionar `R` no Serial; verificar HOME com horário real, virada de um minuto com atualização automática, apresentação da AGENDA do dia atual ou mensagem de agenda livre; retornar à simulação com `T` e confirmar os dois eventos e contador `+4 eventos hoje`.

**Resposta do usuário:** “teste 2 tudo ok”. **Conclusão:** os quatro critérios solicitados foram aprovados. Não foi feito teste dedicado de troca de eventos em seu horário exato, virada de dia, download HTTPS ou estabilidade prolongada.

### Nova melhoria solicitada após Teste 02 — ainda não implementada

**Comportamento atual da v1.9.1, por inspeção do sketch entregue:** durante a releitura ICS, a AGENDA exibe `ATUALIZANDO` no lugar dos eventos; o processo reescreve a lista utilizada na renderização. **Requisito novo:** manter últimos eventos e contador visíveis, usar apenas `Atualização em andamento` em amarelo no rodapé, manter dados estáveis até nova leitura terminar e trocar somente com sucesso. Classificação: **problema de UX/arquitetura confirmado por leitura de código**; **risco dependente de testes:** uso de RAM, sincronização FreeRTOS, troca de dia, falha de download. A v1.9.1 continua sem essa melhoria; próxima entrega deverá implementá-la em sketch integral e passar pela checklist. **Decisão posterior:** remover a indicação redundante `ICS...` e apresentar somente `Atualização em andamento` em amarelo no rodapé.

### Preparação da v1.9.2 — revisão pré-entrega (13/11/2026)

**O que foi alterado no sketch:** leitura SD em blocos de 4 KB; primeira passagem apenas para exceções de recorrência; medidas de tempo total e tempo de leitura em cada passagem; duas listas independentes de eventos (exibida e em processamento), publicadas por troca após sucesso; indicador único `Atualização em andamento` no rodapé da AGENDA; preservação da cópia anterior em caso de falha; seções numeradas 01–11 e protótipos explícitos.

**Problema confirmado detectado e corrigido na revisão:** o valor padrão do parâmetro `debug` de `touchRead` havia sido repetido no protótipo e na implementação, gerando erro de sintaxe C++; removido da implementação. **Verificação executada:** compilação somente de sintaxe C++ com cabeçalhos simulados, concluída sem erros/avisos; **não foi executada compilação no Arduino ESP32 3.3.12**.

**Riscos ainda dependentes de teste:** desempenho real no microSD, RAM/heap em execução e tarefas simultâneas, fidelidade dos seis eventos e contador, caracteres acentuados no TFT, renderização durante download, SD/HTTPS real, estabilidade e troca de dia. Problema de segurança TLS com `setInsecure()` segue conhecido e não corrigido nesta otimização.

**Medições necessárias:** coletar linhas `[PERF]` das passagens 1/2 e operação total, comparar com cerca de 120 segundos da v1.9.1, registrar contagem de seis reuniões e testar touch HOME durante a releitura.

## 7. Registro de documentação

| Data | Alteração documental |
|---|---|
| 13/11/2026 | Documento mestre do projeto rev. 1.6 incorporou revisão sistemática obrigatória antes de novos firmwares. |
| 13/11/2026 | Documentação separada em `ESPECIFICACAO.md`, `DECISOES.md`, `VERSOES.md`; documento mestre convertido em índice. Nenhuma alteração de firmware ou novo teste de hardware foi realizada nesta reorganização. |

A antiga versão integral de `DESK_BUDDY_PROJECT.md` permanece recuperável pelo **histórico de commits do GitHub**.
