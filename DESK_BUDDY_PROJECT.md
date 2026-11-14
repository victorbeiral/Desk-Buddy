# Desk Buddy — Índice do projeto

**Revisão documental:** 1.7 — 13/11/2026
**Status:** REFERÊNCIA DE TRABALHO — PENDENTE DE VALIDAÇÃO INTEGRAL DO CONTEÚDO
**Firmware de interface mais recente confirmado pelo usuário:** v1.9.1 (AGENDA, touch e botões HOME). **Novo sketch:** v1.9.2.1, correção do erro de compilação v1.9.2, ainda não validado na Arduino IDE/CYD.
**Último conjunto completo de cinco testes funcionais informado:** v1.8.1.

> Este arquivo é agora um **índice**, não mais o repositório de todas as informações. A divisão documental não altera firmware, hardware nem validações previamente obtidas.

## Documentação oficial de trabalho

| Documento | Finalidade |
|---|---|
| [ESPECIFICACAO.md](ESPECIFICACAO.md) | **Como o sistema deve funcionar:** hardware, pinagem, telas, HOME, AGENDA, módulos, rede, ICS e critérios funcionais. |
| [DECISOES.md](DECISOES.md) | **Por que as soluções foram escolhidas:** acordos, desenvolvimento modular, 11 seções do sketch, segurança e checklist evolutiva de pré-validação. |
| [VERSOES.md](VERSOES.md) | **O que mudou e quais testes foram aprovados:** versões, falhas, correções, evidências e pendências por versão. |

## Estado atual do desenvolvimento

- **Foco do ciclo:** HOME + AGENDA; v1.9.1 validada nos Testes 01 e 02, v1.9.2 apresentou conflito de compilação em `dim()`; v1.9.2.1 corrige o identificador, aguardando compilação para avaliar desempenho ICS, atualização não bloqueante e indicador único. HOME sem mudanças visuais; agenda com dois eventos e contador.
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

**Próxima ação técnica:** continuar a validação do firmware v1.9.1 sem reescrever ou reorganizar o código apenas por causa desta migração documental.
