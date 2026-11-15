# Desk Buddy — Especificação funcional e técnica

**Base documental:** reorganização do documento mestre rev. 1.6, em 13/11/2026.
**Referência de interface mais recente confirmada pelo usuário:** v1.9.2.2 (indicador `Atualizando agenda`, HOME somente inferior na AGENDA e preservação da navegação). **v1.9.3.1:** execução real confirmada, ICS local válido e tentativa HTTPS falhando `HTTP=-1`. **v1.9.3.2:** diagnóstico executado em hardware: TLS falhou ao alocar memória. **v1.9.3.3:** redução de RAM via sprite 4 bits e correção de inversão touch, ainda pendente de compilação Arduino e teste no CYD.
**Estado:** especificação consolidada como referência de trabalho; revisão integral do conteúdo ainda pendente de aprovação.
**Escopo:** descrever **como o sistema deve funcionar**. Motivações ficam em [DECISOES.md](DECISOES.md); evidências de testes em [VERSOES.md](VERSOES.md).

## 1. Objetivo e limites do sistema

Desk Buddy é um painel de mesa com ESP32 CYD 2,8", display touchscreen, conectividade Wi-Fi e calendário Outlook ICS armazenado no microSD. A interface inicial é a HOME e a navegação ocorre por cartões.

### Estado funcional atual

- **HOME:** relógio grande, data, status Wi-Fi e quatro cartões em grade 2 × 2: AGENDA, CLIMA, + e +. O desenho atual está aprovado; não alterar sem novo requisito.
- **AGENDA:** calendário real Outlook, com visualização inteligente de até dois eventos e contador, sem paginação.
- **CLIMA:** apenas tela de placeholder “EM BREVE”, sem consultas ao serviço meteorológico na linha atual de firmware.
- **Cartões +:** reservados para módulos futuros; telas de placeholder.
- **Retorno à HOME:** na **AGENDA**, apenas o botão inferior (v1.9.2.2; novo ajuste de calibração Y candidato v1.9.3.3, a validar); nas telas placeholder, manter as áreas superior e inferior já existentes. Comandos Serial preservados para diagnóstico.
- **Expansão:** módulos podem ser desenvolvidos isoladamente e reunidos em uma etapa posterior de integração. O módulo de monitoramento OpenAI foi discutido como **proposta**, não como requisito aprovado ou implementado.

## 2. Hardware de referência — variante exata

| Componente | Configuração conhecida |
|---|---|
| Placa | ESP32-2432S028 / CYD 2,8" |
| TFT | ILI9341, 320 × 240, paisagem, rotação 1 |
| Biblioteca gráfica | TFT_eSPI com ILI9341_2_DRIVER, TFT_BGR, HSPI, 27 MHz |
| Renderização | v1.9.2.2/1.9.3.2: sprite 8 bits; v1.9.3.3: paleta 4 bits e fonte Noto diretamente no TFT para liberar RAM para TLS (a validar) |
| Backlight | GPIO 21 |
| microSD | VSPI: SCK 18, MISO 19, MOSI 23, CS 5, 4 MHz |
| Touch | XPT2046 bit-bang: CLK 25, MOSI 32, MISO 39, CS 33, IRQ 36 |
| GPIO36 | **INPUT**, não INPUT_PULLUP (pino somente entrada, sem pull-up interno) |
| Calibração touch conhecida | v1.9.3.2: `swapXY=true`, `invertX=false`, `invertY=true`; toques físicos no HOME inferior produziram y=10–24. v1.9.3.3 propõe `invertY=false` para alinhar ao display, ainda a validar. Limites 250/3850. |
| Arduino | Core ESP32 3.3.12 observado; placa “ESP32 Dev Module”; flash DIO 40 MHz; PSRAM off |
| Monitor Serial | 115200 |
| Fonte | `NotoSansBold15.h` local, sem distribuição pública |

A navegação por touch e o retorno à HOME foram confirmados na v1.9.1. **Calibração fina** e testes em toda a superfície continuam diferentes de uma confirmação de navegação básica.

### Requisitos de compatibilidade

- Verificar pinagem e driver contra a variante concreta do equipamento antes de qualquer mudança.
- Garantir que TFT, touch e microSD não sofram conflito de seleção SPI/pinos.
- Considerar o custo em RAM do sprite de tela inteira e das tarefas/buffers de leitura ICS.
- Impedir que operações longas de Wi-Fi, HTTPS e ICS bloqueiem o atendimento do touch/Serial.
- Redesenhar corretamente as telas ao navegar, atualizar relógio ou finalizar carregamento.

## 3. Comportamento da HOME

- Resolução 320 × 240, fundo azul-escuro, tipografia clara, destaques ciano, verde, amarelo e rosa.
- Relógio em destaque, data, identificação Desk Buddy e status real da conexão Wi-Fi.
- Quatro cartões: AGENDA e CLIMA na primeira linha, + e + na segunda.
- Toque no cartão abre a tela correspondente; na AGENDA, retorno à HOME exclusivamente pelo botão inferior a partir da v1.9.2.2 (pendente de teste no CYD). Nas demais telas internas, comportamento anterior preservado.
- Preservar o layout visual aprovado. Fidelidade de fontes, distribuição e áreas de toque pode exigir inspeção diretamente no equipamento.

## 4. Comportamento da AGENDA

A AGENDA deve priorizar **o que está acontecendo agora** ou **o próximo compromisso**, seguido da próxima ocorrência.

1. Exibir **no máximo dois eventos** e **não usar paginação**.
2. Havendo evento em andamento (`início <= agora < fim`), destacar esse evento com “EM ANDAMENTO” e exibir em segundo lugar o próximo evento futuro.
3. Sem evento em andamento, exibir os dois eventos futuros mais próximos, em ordem cronológica.
4. Exibir abaixo `+N eventos hoje` com a quantidade de eventos relevantes ainda não encerrados que **não** aparecem nos cartões.
5. Evento já encerrado não entra na seleção nem no contador.
6. Se houver um evento, exibir somente um cartão; se não houver eventos restantes, indicar “Agenda livre”.
7. Atualizar a seleção quando o relógio avança (modo real); a simulação de teste usa horário fixado até mudança explícita.
8. Mostrar horários e títulos reais; truncar somente **visualmente** os títulos longos.
9. Preservar retorno HOME e operação de touch enquanto o calendário é processado.
10. **Comportamento implementado e visualmente confirmado na v1.9.2.1:** durante releitura ou download ICS, **manter na AGENDA os últimos dois eventos e contador válidos já apresentados**; sinalizar a atualização com um único texto amarelo no rodapé; a v1.9.2.2 usa **`Atualizando agenda`** para evitar problemas de fonte/acentuação e largura, sem substituir a AGENDA inteira pela tela `ATUALIZANDO`.
11. Manter o conjunto de eventos exibido estável enquanto a nova lista é processada em segundo plano; somente substituir a agenda publicada quando o novo processamento concluir com sucesso. Em caso de falha, preservar os últimos dados válidos e sinalizar o erro sem apresentar informações parciais como se fossem atualizadas.
12. Se ainda não existir agenda válida após a inicialização, permitir mensagem de carregamento. Na virada de dia, evitar apresentar agenda do dia anterior como agenda do dia atual; esta situação exige verificação específica.
13. Não fabricar eventos ou alterar os dados da fonte ICS.

**Ponto ainda sujeito a validação de regra:** a política para múltiplos eventos simultâneos/em andamento. A implementação atual prioriza um evento em andamento; a melhor ordenação em sobreposições ainda não foi homologada.

### Cenário de referência de teste

- Data/hora simulada: segunda-feira, **16/11/2026 08:00**.
- Fonte real de teste: Outlook ICS no microSD.
- Resultado conhecido: seis reuniões do dia, verificadas em versões anteriores.
- Resultado esperado na tela às 08:00: primeira reunião às 08:30, segunda às 09:30 e **+4 eventos hoje**.
- Às 08:45, havendo evento de 08:30–09:00, ele deve aparecer como “EM ANDAMENTO”, seguido da reunião das 09:30 e contador **+4 eventos hoje**.
- A confirmação do uso real e o registro de quais verificações foram efetivamente feitas pertencem a [VERSOES.md](VERSOES.md).

## 5. Calendário Outlook, rede e horários

- Obter o arquivo ICS Outlook por HTTPS a partir de `ICS_URL` do arquivo local privado.
- Armazenar o calendário no microSD como `/agenda.ics`. Usar `/agenda.tmp` e `/agenda.bak` durante substituição e recuperação.
- Validar ao menos a abertura `BEGIN:VCALENDAR` antes de substituir o arquivo existente; preservar cópia anterior em caso de falha de substituição.
- Processar a leitura ICS em **tarefa FreeRTOS em segundo plano**, mantendo interface e comandos responsivos.
- **Modo de uso real (v1.9.3, pendente de testes):** iniciar com simulação **desligada**; sincronizar hora por NTP; ler primeiro a cópia ICS local para o dia atual (se existir), depois baixar a versão atualizada por HTTPS; atualizar novamente por HTTPS a cada **30 minutos** em uso real, com nova tentativa após **10 minutos** em caso de falha. Estes intervalos são parâmetros iniciais de operação sujeitos a revisão após testes.
- Na virada de dia, refazer a seleção do dia atual, não mostrar a agenda do dia anterior como se fosse atual, e buscar atualização HTTPS. Se não existir ICS local, baixar diretamente. Preservar os últimos dados válidos durante o processamento; evitar publicar lista incompleta após falhas.
- Comandos Serial `E` (somente releitura local), `I` (download forçado) e `T`/`R` (simulação/real) permanecem disponíveis; o modo `T` não dispara atualizações periódicas. O arquivo `config_secrets.h` continua como única origem de credenciais e URL privada. Se atualização HTTPS falhar, preservar ICS local válido e distinguir `Outlook nao atualizado` de erro de leitura local; diagnosticar DNS/TCP/TLS sem expor ICS_URL.
- Zona de horário principal de teste: São Paulo, `BRT3` (UTC−3 nas datas atuais); sincronização NTP com a rede.
- Tratar, na medida suportada pelo parser parcial, VEVENT simples, recorrências, exclusões, exceções e TZIDs Windows.
- O parser **não é implementação integral do RFC 5545**. Casos ainda limitados incluem recorrências mensais/anuais e regras `BY*`, `RDATE`, `COUNT`, `EXDATE`, `RANGE=THISANDFUTURE`, `VTIMEZONE`, DST histórico e eventos de dia inteiro. Não afirmar cobertura completa pelo fato de seis reuniões conhecidas aparecerem corretamente.
- Em core ESP32 3.3.12, soluções com `mktime()` e conversão civil foram utilizadas porque `timegm()` não estava disponível na configuração observada.

### Observação de segurança

A implementação anterior de HTTPS utilizou `WiFiClientSecure.setInsecure()`, que **desabilita a verificação do certificado do servidor**. Isso é um problema conhecido de segurança a corrigir em trabalho futuro; não tratá-lo como resolvido.

## 6. Interfaces de diagnóstico e operação

Na linha atual, conservar comandos Serial de teste, quando presentes no sketch de referência:

| Comando | Função |
|---|---|
| `H` | HOME |
| `A` | AGENDA |
| `C` | CLIMA |
| `E` | Reler ICS local |
| `I` | Baixar ICS e ler |
| `T` | Modo de simulação |
| `R` | Modo relógio real |
| `X` | Diagnóstico touch |
| `S` | Diagnóstico de estado |

Não reintroduzir comandos de paginação na agenda de dois eventos.

## 7. Arquivos, confidencialidade e organização

O sketch da fase atual inclui somente os módulos necessários para **HOME + AGENDA**, preservando a navegação/placeholder CLIMA. O código integral de CLIMA permanece adiado, mas sua integração final segue como objetivo.

Arquivos esperados localmente na pasta Arduino do sketch:

```text
Desk_Buddy/
├── Desk_Buddy.ino
├── config_secrets.h        # Somente local. Nunca versionar nem publicar.
└── NotoSansBold15.h        # Dependência local.
```

Variáveis privadas mantidas fora do sketch público: `WIFI_SSID`, `WIFI_PASSWORD` e `ICS_URL`. Não versionar ICS real, tokens, credenciais nem logs privados. O repositório GitHub do projeto é **privado**.

**Estado do repositório:** o documento original e sua documentação já estão no GitHub; não presumir que o firmware v1.9.1 esteja versionado como `.ino` até um commit específico comprovar isso.

## 8. Critérios de aceitação funcional

- HOME inicial, relógio/data/Wi-Fi corretos, cartões sem regressão visual.
- Touch permite abrir AGENDA, CLIMA e os placeholders, além de voltar à HOME.
- Leitura/atualização ICS não bloqueia a navegação nem o monitor Serial.
- **v1.9.2.1 validada visualmente pelo usuário:** a releitura não oculta eventos nem contador; o indicador amarelo aparece no rodapé, porém seu texto com acentuação foi exibido incorretamente. **v1.9.2.2 (confirmada pelo usuário):** mensagem `Atualizando agenda` completa e botão HOME apenas inferior na AGENDA; **v1.9.3 pendente:** NTP, busca automática HTTPS e rotação diária.
- Seis compromissos conhecidos do cenário de teste preservados; tela mostra somente os dois prioritários e contador correto.
- Releitura/atualização de ICS, mudança de dia e modo real merecem revalidação por versão.
- Sem corrupção de microSD, travamento, artefatos gráficos persistentes ou reset inesperado em uso prolongado.
- Revisão geral obrigatória de pinagem, bibliotecas, RAM, SPI, touch, display, renderização, bloqueios e compatibilidade com a variante exata antes de cada entrega.
- **Não confundir exigência com resultado:** o estado real de cada teste é registrado em [VERSOES.md](VERSOES.md).

## 9. Módulos adiados e integração final

### CLIMA

A implementação anterior explorou Open-Meteo para Paulínia-SP (aprox. −22.7542, −47.1488), exibindo condição, temperatura, próximas quatro horas e mínima/máxima móvel de 24 h. Na linha v1.8.x/v1.9.x aqui documentada, **não fazer chamadas meteorológicas**: manter somente placeholder. A tentativa anterior retornou HTTP −1; causa não confirmada.

### Integração final

Os módulos aprovados separadamente deverão ser reunidos em uma versão integrada. Essa etapa exige **nova revisão completa e testes de regressão**, especialmente memória, SPI, paralelismo de tarefas, rede, atualização gráfica e touch. Módulo adiado não equivale a módulo descartado.
