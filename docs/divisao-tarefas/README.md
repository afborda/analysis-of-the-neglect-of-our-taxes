# Divisão de Tarefas — Abner + Miguel

> Cada um tem uma frente clara de trabalho. O objetivo é que ninguém fique bloqueado
> esperando o outro e que ambos aprendam as duas dimensões do projeto: infraestrutura
> e análise.

---

## Visão Geral

```mermaid
flowchart TD
    subgraph ABNER["👨‍💻 Abner — Infraestrutura + Dados Complexos"]
        A1["Task 1\nBootstrap do projeto"]
        A2["Task 2\nGoogle Cloud + BD+ setup"]
        A8["Task 8\nDocker + Playwright"]
        A7["NB05 — Salário Prefeito\nAPI Portal da Transparência"]
        A9["NB06 — Ficha Limpa\nPlaywright + TSE"]
        A10["NB07 — IVM Ranking\nÍndice final combinado"]
    end

    subgraph MIGUEL["📊 Miguel — Análise + Visualização"]
        M3["NB01 — Perfil dos Municípios\nIBGE + IDH + mapas"]
        M4["NB02 — Eleições Históricas\nTSE + timeline de partidos"]
        M5["NB03 — Bolsa Família + Emprego\nMDS + RAIS + correlações"]
        M6["NB04 — Finanças das Prefeituras\nSICONFI + Lei de Responsabilidade"]
    end

    subgraph SYNC["🤝 Pontos de Sincronização"]
        S1["Sync 1\nBD+ funcionando para os dois"]
        S2["Sync 2\nNB01–04 prontos, revisar juntos"]
        S3["Sync 3\nDocker testado + NB05/06 prontos"]
        S4["Sync 4\nNB07 — construir IVM juntos"]
    end

    A2 --> S1
    S1 --> M3
    M6 --> S2
    A8 --> S3
    S2 --> S3
    S3 --> S4
    A10 --> S4

    style ABNER fill:#E3F2FD,color:#000
    style MIGUEL fill:#E8F5E9,color:#000
    style SYNC fill:#FFF8E1,color:#000
```

---

## Regras de Colaboração

1. **Commits frequentes** — commitar ao final de cada notebook ou task, não acumular
2. **Nunca commitar na branch do outro** sem combinar
3. **Cache obrigatório** — todo Playwright usa `data/raw/` com cache antes de rodar
4. **Nos Syncs** — um apresenta para o outro, explicando o raciocínio da análise
5. **Dúvidas técnicas** — resolver junto, não deixar acumular

---

## Arquivos de cada um

- [Tarefas do Abner](abner.md)
- [Tarefas do Miguel](miguel.md)
