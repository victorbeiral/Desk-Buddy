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
| v1.9.1 | **Interface validada parcialmente em hardware (13/11/2026)** | Substituição de `AgendaView` por `selectAgenda(time_t,int&,int&,int&,bool&)`; AGENDA com dois eventos, destaque de andamento e contador, sem paginação. Usuário confirmou “esta mostrando corretamente” e depois confirmou que navegação touch e botões HOME “continuam” funcionando. Releitura ICS, rollover do dia e estabilidade prolongada ainda sem nova confirmação nessa versão. |

**Referência atual de interface:** v1.9.1.
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
| Touch durante leitura ICS | **OK informado** | **Não revalidado especificamente** |
| Navegação por touch | **OK informado** | **OK informado** |
| Botões de retorno HOME | **OK informado** | **OK informado** |
| Resultado da agenda | **OK informado** | **OK: exibição informada** |
| Releitura ICS | **OK informado** | **Pendente de reconfirmação** |
| Download HTTPS real | Não confirmado como ensaio independente | **Pendente** |
| Virada de dia/modo real | **Pendente** | **Pendente** |
| Calibração fina touch e placeholders + | **Pendente** | **Pendente** |
| Tempo de execução prolongado, heap e resets | **Pendente** | **Pendente** |
| Reintegração CLIMA completo | Adiada | **Adiada** |

Não confundir a confirmação de que o touch continua funcionando com teste específico **durante** nova leitura ICS na v1.9.1.

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

- [ ] Confirmar hora, data, modo real e Wi-Fi após inicialização.
- [ ] Revalidar explicitamente releitura ICS na v1.9.1.
- [ ] Revalidar touch e Serial durante leitura e download ICS na v1.9.1.
- [ ] Testar troca do evento atual/próximo à medida que o relógio avança.
- [ ] Testar virada de dia, casos de agenda vazia, evento único e sobreposições.
- [ ] Testar os dois cartões +, CLIMA placeholder e limpeza de tela sem artefatos.
- [ ] Testar estabilidade prolongada e coletar heap/reset quando possível.
- [ ] Corrigir/validar TLS com verificação de certificado antes de considerar o download seguro.
- [ ] Testar integração final dos módulos quando forem reunidos.

**Nota de rastreabilidade:** a v1.8.1 é referência para as cinco verificações funcionais informadas; a v1.9.1 é a referência **mais recente para a interface**, com escopo de validação explicitamente limitado.

## 7. Registro de documentação

| Data | Alteração documental |
|---|---|
| 13/11/2026 | Documento mestre do projeto rev. 1.6 incorporou revisão sistemática obrigatória antes de novos firmwares. |
| 13/11/2026 | Documentação separada em `ESPECIFICACAO.md`, `DECISOES.md`, `VERSOES.md`; documento mestre convertido em índice. Nenhuma alteração de firmware ou novo teste de hardware foi realizada nesta reorganização. |

A antiga versão integral de `DESK_BUDDY_PROJECT.md` permanece recuperável pelo **histórico de commits do GitHub**.
