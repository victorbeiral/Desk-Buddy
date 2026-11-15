# Desk Buddy — Índice do projeto

**Revisão documental:** 2.1 — 14/11/2026
**Status:** REFERÊNCIA DE TRABALHO — PENDENTE DE VALIDAÇÃO INTEGRAL DO CONTEÚDO
**Firmware mais recente executado no CYD:** v1.9.2.1 — dois ciclos de ICS local em 12,454 e 12,698 s, seis reuniões e touch durante releitura comprovados por logs. **v1.9.2.2 teve ajuste visual confirmado. v1.9.3 apresentou erro de compilação no Arduino; v1.9.3.1 corrige a definição de `daysInMonth` e aguarda compilação real/CYD.**
**Último conjunto completo de cinco testes funcionais informado:** v1.8.1.

> Este arquivo é agora um **índice**, não mais o repositório de todas as informações. A divisão documental não altera firmware, hardware nem validações previamente obtidas.

## Documentação oficial de trabalho

| Documento | Finalidade |
|---|---|
| [ESPECIFICACAO.md](ESPECIFICACAO.md) | **Como o sistema deve funcionar:** hardware, pinagem, telas, HOME, AGENDA, módulos, rede, ICS e critérios funcionais. |
| [DECISOES.md](DECISOES.md) | **Por que as soluções foram escolhidas:** acordos, desenvolvimento modular, 11 seções do sketch, segurança e checklist evolutiva de pré-validação. |
| [VERSOES.md](VERSOES.md) | **O que mudou e quais testes foram aprovados:** versões, falhas, correções, evidências e pendências por versão. |

## Estado atual do desenvolvimento

- **Foco do ciclo:** HOME + AGENDA; v1.9.2.2 teve ajuste visual confirmado. v1.9.3.1 preparada para iniciar no relógio real, ler o ICS local e baixar Outlook automaticamente a cada 30 min (retry após 10 min), visando teste cotidiano a partir de segunda-feira 16/11/2026. Validação de download HTTPS, heap e virada do dia ainda pendente. HOME sem mudanças visuais; agenda com dois eventos e contador.
- **CLIMA:** placeholder, implementação completa adiada, não descartada.
- **Objetivo posterior:** integrar todos os módulos parcialmente validados e realizar nova revisão de regressão, RAM, SPI e interface.
- **Novo padrão de código:** seções numeradas 01–11 e protótipos explícitos; aplicados na preparação da v1.9.2.
- **Entrega obrigatória:** qualquer ajuste de código deve ser feito pelo assistente e entregue como sketch **integral e versionado**, em **um único bloco copiável no chat**; nunca apenas trechos.
- **Pré-validação obrigatória:** revisão sistemática do código, distinguindo **problemas confirmados** de **riscos dependentes de teste**. Checklist evolutiva em [DECISOES.md](DECISOES.md).
- **GitHub:** `victorbeiral/Desk-Buddy`, branch `main`, repositório privado. Nunca publicar `config_secrets.h`, ICS reais, credenciais, tokens ou arquivos de fonte.
- **Atenção:** o firmware v1.9.1 foi entregue no chat; não há registro aqui de commit do sketch `.ino` no repositório.

## Regra para evitar informações contraditórias

- **Requisito vigente ou descrição técnica atual:** atualizar `ESPECIFICACAO.md`.
- **Nova regra, escolha de engenharia ou justificativa:** atualizar `DECISOES.md`.
- **Nova versão, falha observada, teste executado ou validação:** atualizar `VERSOES.md`.
- **Links e resumo curto:** atualizar somente este índice.
- Decisões mais recentes do usuário prevalecem. O histórico não deve ser apagado, mas registros antigos não devem continuar apresentados como estado atual.

## Histórico anterior

A versão completa deste documento até a **revisão 1.6 (13/11/2026)** segue disponível no **histórico Git**. Sua informação relevante foi distribuída entre os três arquivos acima. A autorização para reorganizar os arquivos **não equivale** a validar integralmente a exatidão técnica de cada registro.

**Próxima ação técnica:** compilar a v1.9.3.1 no Arduino ESP32 3.3.12; se compilar, carregar no CYD e verificar Wi-Fi/NTP, primeira consulta HTTPS automática, agenda real e navegação na segunda-feira.
