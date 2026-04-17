# Habilitação de Abas no Atendimento de Pacientes Agendados

Este documento descreve o processo de investigação e a solução aplicada para habilitar a escolha de procedimentos na aba **CID/Procedimentos** que aparecia desativado (disabled) na tela de atendimento e não deixava o atendimento finalizar por estar selecionado a `flag` Procedimentos na criação da Grade de Atendimento.

## 1. Habilitar Escolha de Procedimentos (Aba 4)

Além das permissões de acesso, a lista de procedimentos na Aba 4 exige uma configuração específica para que o atendimento possa ser finalizado.

### O Procedimento "Nenhum Procedimento Realizado"
O AGHU exige que ao menos um procedimento seja marcado. Quando não há intervenções, usa-se o "Nenhum Procedimento Realizado".

1.  **Parâmetro do Sistema:** Verifique o parâmetro **`P_SEQ_PROC_NENHUM`**.
2.  **Vínculo com Banco:** O valor numérico desse parâmetro deve ser IGUAL ao `seq` do registro na tabela `agh.mam_procedimentos`.
3.  **Validação de CBO:** O sistema ignora a validação de competência (CBO) apenas para o ID definido neste parâmetro. Se o ID for diferente, o erro *"O CBO do profissional não é compatível"* impedirá a gravação.
4.  **Adicionando NENHUM:** Para adicionar a opção nenhum procedimento execute o sql:
INSERT INTO agh.mam_procedimentos
(seq, descricao, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ped_seq, ind_generico, "version", phi_seq)
VALUES(27, 'NENHUM PROCEDIMENTO REALIZADO', 'A', '2011-10-13 11:26:06.445', 9999999, 955, NULL, 'S', 0, NULL);
5. Obs **`P_SEQ_PROC_NENHUM`** defina para 27 pi deixe 1 na tabela e 1 no parâmetro

---

## 2. Fluxo para Habilitar Novos Procedimentos via Aplicação

Para adicionar novos procedimentos à lista de seleção do médico sem usar SQL, siga esta sequência no menu do sistema:

### Passo 1: Cadastro de Procedimento Especiais via Prescrição
Antes de tudo, o procedimento deve existir na base da preescrição que adicionará automaticamento a tabela do faturamento Procedimento Hospitalar Interno (PHI).
*   **Menu:** `Prescrição > Cadastros > Procedimentos Especiais`
*   **Ação:** Criar um novo procedimento especial e vincular ao código SIGTAP (SUS) correspondente.
    **Nota:** Ticar Imp.Sum.Alta e Perm. Prescrição e CCIH

### Passo 2: Cadastro do Procedimento Hospitalar Interno (PHI)
Ao realizar o Passo 1, o sistema já cria o PHI, com as flags Ativo e Origem Prescrição ticadas.
*   **Menu:** `Financeiro › Faturamento › Cadastros › Procedimento Hospitalar Interno`
*   **Ação:** Não é necessário criar o procedimento aqui, pois ele já foi criado no Passo 1.

### Passo 3: Relacionar Procedimento SUS à Procedimento Hospitalar Interno
*   **Menu:** `Financeiro › Faturamento › Cadastros › Tabelas › Relacionar procedimento SUS ao procedimento interno`
*   **Ação:** Selecionar o **Procedimento Hospitalar Interno** (ex: Consulta em Ortopedia) e clicar em **Pesquisar**, depois incluir o procedimento SUS correspondente e **Selecionar** as `flags` |Paga| e |Plano Ambulatório|. e por fim o botão **Gravar**

### Passo 4: Vincular Procedimento e Especialidade
*   **Menu:** `Ambulatório › Cadastros › Procedimento e Especialidade`
*   **Ação:** Selecionar a **Especialidade/Agenda** (ex: Ortopedia Ambulatorial) e clicar em **Pesquisar**, depois incluir o procedimento correspondente e **Gravar**, as `flags` |Consulta| e |Plano Ambulatório|. e por fim o botão **Gravar**

### Passo 5: Vincular Procedimento e Especialidade
*   **Menu:** `Ambulatório › Cadastros › Procedimento e Especialidade`
*   **Ação:** Selecionar a **Especialidade/Agenda** (ex: Ortopedia Ambulatorial) e clicar em **Pesquisar**, depois incluir o procedimento `Proced. Hosp. Especialidades` correspondente e **GRAVAR**
*   **Nota:** Quando o checkbox “Consulta” estiver marcado, estes "procedimentos" ficarão ocultos na visualização do médico. O sistema subentende que tal “procedimento” trata-se de uma Consulta Especializada (conforme especialidade), sem necessidade de estar visível ao médico. Logo, quando o AGHU possuir o módulo "Faturamento", será possível identificar que todas as agendas possuem no mínimo um “procedimento” para cobrança, conforme a tabela SIGTAP
(DATASUS).

---
