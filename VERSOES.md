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
| v1.9.2 | **Erro de compilação relatado (13/11/2026)** | Scanner ICS em buffers de 4096 bytes, primeira passagem seletiva para UID/RECURRENCE-ID, medições `[PERF]` por passagem (tempo, SD, bytes, blocos, linhas), dois bancos de eventos com troca após sucesso, manutenção da AGENDA visível e indicador único `Atualização em andamento`. Mantém HOME, touch, retorno HOME, comandos, simulação e parser parcial. Verificação de sintaxe C++ com bibliotecas simuladas não detectou conflito envolvendo a função `dim()`. Arduino IDE ESP32 3.3.12 apresentou erro `'int dim(int, int)' was declared 'extern' and later 'static'`; versão substituída pela v1.9.2.1. |
| v1.9.2.1 | **Compilação/execução, desempenho ICS e preservação visual da AGENDA confirmados no CYD (13/11/2026)** | Corrigido conflito de `dim()` para `daysInMonth()`. Dois ciclos locais ICS: 12.454 e 12.698 s; ambos `result=OK`, 770 VEVENT, 53 séries, 187 exceções, seis reuniões, visão `+4 eventos hoje`. Logs de touch durante a releitura comprovam atendimento e retorno HOME. Indicador amarelo e preservação visual dos eventos **confirmados pelo usuário** durante releitura, porém mensagem do indicador apresentou renderização incorreta (`Atualizao...`). Download HTTPS e estabilidade prolongada pendentes. |

| v1.9.2.2 | **Ajuste de UI confirmado pelo usuário após entrega (14/11/2026)** | AGENDA: remove apenas o botão HOME superior e desativa sua área touch; mantém retorno HOME inferior. Mensagem amarela substituída por `Atualizando agenda`, evitando caracteres acentuados que não renderizaram corretamente no TFT e cabendo antes do botão inferior. Nenhuma mudança na leitura ICS, pinagem, Wi-Fi, memória ou algoritmo dos eventos. Revisão estática e C++17 com interfaces simuladas concluídas sem erros; o usuário confirmou que o ajuste funcionou. Essa confirmação não abrange uso prolongado. |

| v1.9.3 | **Erro de compilação reportado no Arduino ESP32 3.3.12 (14/11/2026)** | Inicia com `simulation=false`; espera NTP válido; carrega ICS local do dia; busca automaticamente ICS Outlook por HTTPS após leitura local; repete a cada 30 min e tenta novamente após 10 min em falha; na virada de dia reprocessa a seleção e não apresenta dados antigos como atuais. Mantém duplo buffer, processamento de ~12,5 s medido na versão anterior, touch e HOME inferior; valida início/fim do arquivo recebido e preserva/restaura `.bak` se o processamento do download falhar. Revisão C++17 com bibliotecas simuladas sem erros/avisos; sem compilação Arduino real ou validação da busca HTTPS nesta versão. TLS `setInsecure()` permanece risco conhecido. |

| v1.9.3.1 | **Compilação/execução no CYD confirmadas; ICS local OK; HTTPS falhou (14/11/2026)** | Corrige a definição de `daysInMonth()` mantendo nome e assinatura, mas eliminando a tabela local `static const int a[]` na função de uma única linha; substitui por `switch` de meses com retorno de 28/29/30/31 e isola protótipo em linha própria. Mantém toda funcionalidade de modo real, NTP, HTTPS automático de 30 min e retry de 10 min da v1.9.3. Verificação C++17 com cabeçalhos simulados sem erros/avisos; **ainda não era compilação real no Arduino no momento da entrega**. Compilação e execução posteriormente confirmadas pelo usuário com log de NTP/ICS local; HTTPS retornou `HTTP=-1`. A causa exata da geração de protótipos Arduino ainda não foi demonstrada por análise do `.ino.cpp` intermediário. |

| v1.9.3.2 | **Entregue para diagnóstico; compilação Arduino e teste CYD pendentes (14/11/2026)** | Após HTTP negativo, registra erro HTTP, diagnóstico TLS e testes DNS/TCP sem divulgar a URL Outlook. Se ICS local válido, exibe `Agenda livre` e `Outlook nao atualizado` em vez de erro genérico. Evita restaurar `.bak` antigo em falha antes da instalação de um novo ICS. Adiciona log do toque na antiga região HOME superior; preserva botão inferior, NTP e leitura ICS. Verificação C++17 com stubs aprovada, sem compilação real. |

**Referência para processamento ICS com medição no CYD:** v1.9.2.1; **interface aprovada:** v1.9.2.2; **v1.9.3:** erro de compilação. **v1.9.3.1:** executou no CYD, leitura local aprovada, HTTPS falhou. **v1.9.3.2:** diagnóstico pendente. **Referência de interface visual aprovada anteriormente:** v1.9.1. **v1.9.2:** erro de compilação informado.
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
- [x] **Preservação visual v1.9.2.1:** usuário confirmou que eventos/contador permanecem visíveis com indicador amarelo durante releitura; observou erro de renderização da frase acentuada.
- [x] **Validar visual e touch v1.9.2.2:** o usuário respondeu “confirmado” ao teste da mensagem legível, HOME inferior e ajuste visual.
- [x] **v1.9.3.1 — execução em modo real:** Wi-Fi, 14/11/2026 21:16, ICS local em 12.641 ms, 770 VEVENT, 0 eventos no sábado e tentativa HTTPS comprovados.
- [ ] **v1.9.3.2 — diagnóstico:** investigar `HTTP=-1` com `[TLS]` e `[NET]`, verificar aviso `Outlook nao atualizado`, testar toque especificamente no HOME inferior e validar recuperação de arquivo.
- [ ] Ainda pendentes: download HTTPS bem-sucedido, atualização de 30 min, troca de dia e eventos reais, estabilidade.
- [ ] Testar especificamente atualização por HTTPS, falha do download e preservação do snapshot no CYD.
- [ ] Testar os dois cartões +, CLIMA placeholder e limpeza de tela sem artefatos.
- [ ] Testar estabilidade prolongada e coletar heap/reset quando possível.
- [ ] Corrigir/validar TLS com verificação de certificado antes de considerar o download seguro.
- [ ] Testar integração final dos módulos quando forem reunidos.

**Nota de rastreabilidade:** v1.8.1 teve cinco verificações funcionais aprovadas; v1.9.1 teve exibição e Testes 01/02 aprovados; v1.9.2.1 é a versão **mais recente executada no CYD com tempos ICS medidos**, mas falta confirmação visual da atualização e ensaios adicionais.

### Registro de teste — v1.9.1, Teste 01 (13/11/2026)

**Procedimento solicitado:** na AGENDA, executar `E` pelo Monitor Serial (releitura do ICS armazenado no microSD); durante `ATUALIZANDO`, navegar por touch para HOME, CLIMA e AGENDA; após conclusão, conferir dois eventos, contador `+4 eventos hoje` e saída de processamento no Serial.

**Resposta do usuário:** “teste 01 tudo ok”. **Conclusão:** aprovação conjunta dos itens solicitados no Teste 01: releitura ICS local, touch/navegação durante leitura e apresentação final da agenda. Não há logs anexados nem medições de duração/heap. Não extrapolar o resultado para o comando `I`, download HTTPS, virada de dia ou estabilidade prolongada.

### Registro de teste — v1.9.1, Teste 02 (13/11/2026)

**Procedimento solicitado:** acionar `R` no Serial; verificar HOME com horário real, virada de um minuto com atualização automática, apresentação da AGENDA do dia atual ou mensagem de agenda livre; retornar à simulação com `T` e confirmar os dois eventos e contador `+4 eventos hoje`.

**Resposta do usuário:** “teste 2 tudo ok”. **Conclusão:** os quatro critérios solicitados foram aprovados. Não foi feito teste dedicado de troca de eventos em seu horário exato, virada de dia, download HTTPS ou estabilidade prolongada.

### Nova melhoria solicitada após Teste 02 — ainda não implementada

**Comportamento atual da v1.9.1, por inspeção do sketch entregue:** durante a releitura ICS, a AGENDA exibe `ATUALIZANDO` no lugar dos eventos; o processo reescreve a lista utilizada na renderização. **Requisito novo:** manter últimos eventos e contador visíveis, usar apenas `Atualização em andamento` em amarelo no rodapé, manter dados estáveis até nova leitura terminar e trocar somente com sucesso. Classificação: **problema de UX/arquitetura confirmado por leitura de código**; **risco dependente de testes:** uso de RAM, sincronização FreeRTOS, troca de dia, falha de download. A v1.9.1 continua sem essa melhoria; próxima entrega deverá implementá-la em sketch integral e passar pela checklist. **Decisão posterior:** remover a indicação redundante `ICS...` e apresentar somente `Atualização em andamento` em amarelo no rodapé.

### Erro de compilação v1.9.2 e correção v1.9.2.1 (13/11/2026)

**Falha confirmada pelo usuário:** Arduino ESP32 core 3.3.12 relata `int dim(int,int) was declared extern and later static` na definição da função de dias do mês, apontando seu protótipo como declaração anterior. **Ação:** renomear a função e todas as suas referências para `daysInMonth()` na v1.9.2.1, evitando o identificador conflitante. O aviso de múltiplas bibliotecas `SD.h` não é falha; a IDE indicou como usada a versão do core ESP32. **Verificação realizada:** C++17 com cabeçalhos simulados, `-Wall -Wextra -Werror -fsyntax-only`, passou sem erros; **compilação real Arduino e hardware ainda pendentes**.

### Medições em hardware — v1.9.2.1 (13/11/2026)

**Fonte:** Monitor Serial fornecido pelo usuário após gravação da v1.9.2.1 no CYD ESP32-2432S028 (dois ciclos, inicialização e releitura ICS local). A execução no dispositivo comprova que o firmware compilou e carregou; não há registro completo do procedimento de compilação nem de ensaio prolongado.

| Métrica | Ciclo inicial | Releitura |
|---|---:|---:|
| Passagem 1 total | 5.888 ms | 5.886 ms |
| Passagem 1 SD | 4.798 ms | 4.814 ms |
| Passagem 2 total | 6.476 ms | 6.735 ms |
| Passagem 2 SD | 4.810 ms | 4.814 ms |
| Operação total | **12.454 ms** | **12.698 ms** |
| Resultado | OK | OK |
| Heap livre ao final | 76.820 bytes | 76.416 bytes |
| Heap mínimo registrado | 39.596 bytes | 39.460 bytes |

Nos dois ciclos: **2.024.858 bytes por passagem**, 495 blocos por passagem, 18.837 linhas por passagem, 56 linhas truncadas, 770 VEVENT, 53 séries, 187 exceções, 5 ocorrências geradas, 3 regras e 11 datas não suportadas, **6 reuniões armazenadas**. Visão da agenda: primeiro índice 0, segundo índice 1, `+4 eventos hoje`, sem alteração dos horários conhecidos.

**Touch:** foram registrados toques durante a segunda leitura, inclusive retorno HOME (toque na AGENDA seguido de registro na HOME). **Confirmação posterior do usuário:** eventos e contador permaneceram visíveis e o indicador amarelo apareceu conforme esperado; problema: a mensagem com caracteres acentuados apareceu truncada/corrompida (`Atualizao...`).

**Interpretação:** comparação aproximada com ~120 s anteriores aponta ~90% de redução, cerca de 9,5 vezes mais rápido; o tempo antigo era estimado, não medido com a instrumentação atual. Na releitura, SD responde por 9.628 ms dos 12.698 ms (~75,8%); próxima estratégia de desempenho mais promissora é evitar leituras repetidas via cache validado, sem aumentar velocidade SPI antes de testes.

**Pendências:** confirmar preservação visual dos eventos/contador e único indicador amarelo durante releitura; realizar download HTTPS; observar estabilidade, heap sob carga, travessia de dia e limites do parser.

### Confirmação visual v1.9.2.1 e correção v1.9.2.2 — 13/11/2026

**Relato do usuário:** durante o comando de releitura ICS, os eventos e o contador permaneceram visíveis e apareceu um único indicador amarelo no rodapé, como solicitado. O texto `Atualização em andamento` foi exibido com falha de caracteres (`Atualizao...`). Pediu remover o botão HOME superior, mantendo o inferior.

**Falha confirmada:** mensagem de estado exibida incorretamente; **causa técnica provável, não comprovada isoladamente:** suporte incompleto a UTF-8/acento na fonte e/ou largura disponível. **Correção implementada na v1.9.2.2:** indicador `Atualizando agenda` (frase correta, sem acentos e mais curta) usando fonte bitmap 2 com largura máxima disponível; botão HOME superior removido apenas da AGENDA e antiga área de toque desabilitada, preservando o inferior. Outras telas, ICS e HOME principal inalterados.

**Pré-validação:** comparação do diff com v1.9.2.1 e verificação C++17 com cabeçalhos simulados (`-Wall -Wextra -Werror -fsyntax-only`) concluídas sem erros; verificações de 11 seções, protótipos, pinagem touch, SD, buffers, seleção de tela e botão inferior. **Compilação real no Arduino ESP32 3.3.12 e teste de display/touch ainda pendentes**.

### Erro de compilação na v1.9.3 e correção candidata v1.9.3.1 — 14/11/2026

**Log reportado pelo usuário:** na v1.9.3, o compilador Arduino ESP32 core 3.3.12 apontou `'int daysInMonth(int,int)' was declared 'extern' and later 'static'`, com referência à declaração explícita e à definição em linha única contendo vetor local `static const int a[]`. A v1.9.2 apresentou mensagem semelhante para o nome anterior `dim()`; portanto o **identificador não era explicação suficiente**. **Hipótese técnica provável, ainda não verificada no código intermediário da IDE:** interferência do pré-processamento Arduino no trecho compacto com armazenamento estático local.

**Ação na v1.9.3.1:** remover por completo o vetor local `static`; usar `switch` para quantidade de dias do mês e declarar a assinatura em linha própria; a semântica 28/29/30/31 dias permanece igual para datas válidas. Verificação estática e compilação de sintaxe C++17 com stubs, `-Wall -Wextra -Werror -fsyntax-only`, **sem erros**. A compilação pela Arduino IDE real e o teste no CYD **ainda não foram executados**; não declarar correção comprovada até confirmação do usuário. Aviso “Multiple libraries for SD.h” é informativo quando a biblioteca do core ESP32 está sendo usada.

### Relato de execução v1.9.3.1 e preparação v1.9.3.2 (14/11/2026)

O usuário enviou log do CYD: Wi-Fi conectado, hora real 14/11/2026 21:16, leitura local **12.641 ms** e `result=OK`, 770 VEVENT, 187 exceções, 0 reuniões do sábado. O Outlook retornou `HTTP=-1` em 42 ms e houve nova tentativa agendada para 10 minutos. **A causa da conexão recusada/falha de conexão ainda não é conhecida.** `Agenda livre` é coerente com a leitura local; a mensagem `Falha na atualizacao` refletia falha de HTTPS, não falha da agenda local.

Toques registrados na AGENDA: x entre 252 e 262, y entre 23 e 27, correspondendo **à antiga região do HOME superior removido**; não houve registro na faixa do HOME inferior (y entre 211 e 238), portanto falha do botão inferior ainda não foi demonstrada.

**Risco lógico detectado na revisão:** após falha de download anterior à instalação de novo ICS, código v1.9.3.1 poderia restaurar backup velho caso `.bak` existisse. A v1.9.3.2 condiciona recuperação à substituição executada no download atual. Também acrescenta `errorToString`, erro TLS e testes de DNS/TCP (sem exibir URL privada), diferencia falha online da agenda local e registra a área de toque superior desativada. **Teste realizado:** sintaxe C++17 com interfaces simuladas, sem erros/avisos. **Não realizado:** compilação Arduino ESP32 3.3.12 e hardware v1.9.3.2. A versão não comprova solução do `HTTP=-1`. `setInsecure()` segue risco conhecido.

### v1.9.3 — preparação para operação real (14/11/2026)

**Objetivo aprovado pelo usuário:** deixar a AGENDA funcional, buscando eventos reais do dia para testar na prática a partir de segunda-feira, **16/11/2026**. A v1.9.2.2 teve o ajuste visual confirmado.

**Implementação v1.9.3, ainda não testada no CYD:** simulação desligada na partida, sincronização NTP, leitura ICS local inicial, download Outlook em FreeRTOS, atualização automática a cada 30 min no modo real, nova tentativa em 10 min após falha, passagem de dia, backup do ICS até confirmação de parse, mensagem `Atualizando agenda`, comando `T` preservado só para testes. **Revisão pré-entrega:** verificação de sintaxe C++17 (`-Wall -Wextra -Werror`) usando cabeçalhos simulados; não equivale a compilação com core ESP32 real. **Riscos:** consumo de heap durante TLS+SD, confiabilidade da URL publicada Outlook, parser RFC5545 parcial, certificado TLS não verificado, recuperação após falhas de energia/rede e operação prolongada.

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
