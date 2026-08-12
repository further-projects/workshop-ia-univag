# Especificações do Mini Notion

Esta pasta contém as especificações funcionais do produto. Cada diretório representa uma feature
independentemente planejável e testável.

## Ordem recomendada

| Ordem | Feature | Estado | Próxima etapa |
| --- | --- | --- | --- |
| 001 | [Project Foundation](001-project-foundation/spec.md) | Draft | Revisar e clarificar |
| 002 | [Authentication](002-authentication/spec.md) | Draft | Revisar e clarificar |
| 003 | [Document Management](003-document-management/spec.md) | Draft | Revisar e clarificar |
| 004 | [Rich Text Editor](004-rich-text-editor/spec.md) | Draft | Revisar e clarificar |

## Estrutura por feature

```text
specs/NNN-feature/
├── spec.md
├── checklists/
│   └── requirements.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md
```

Somente `spec.md` e o checklist de requisitos são criados na fase atual. Os demais artefatos serão
gerados pelos comandos apropriados quando a feature estiver pronta.

## Fluxo

1. Criar ou atualizar a feature com `$speckit-specify`.
2. Resolver decisões pendentes com `$speckit-clarify`.
3. Produzir pesquisa e desenho técnico com `$speckit-plan`.
4. Gerar tarefas com `$speckit-tasks`.
5. Validar consistência com `$speckit-analyze`.
6. Implementar com `$speckit-implement`.

Uma feature não deve avançar enquanto seu checklist de requisitos contiver falhas relevantes.

## Responsabilidade dos documentos

- A [constituição](../.specify/memory/constitution.md) contém princípios permanentes.
- A [visão geral](../README.md) contém stack, arquitetura e decisões globais.
- Cada `spec.md` descreve necessidades, comportamento e sucesso sem definir implementação.
- Cada `plan.md` futuro conterá escolhas técnicas específicas da feature.
