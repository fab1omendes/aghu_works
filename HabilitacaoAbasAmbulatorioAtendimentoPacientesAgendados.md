# Habilitação de Abas no Atendimento de Pacientes Agendados

Este documento descreve o processo de investigação e a solução aplicada para habilitar as abas **Anamnese, Evolução, CID/Procedimentos e Atestado** que apareciam inativas (disabled) na tela de atendimento.

## 1. Origem da Regra de Habilitação (Código Fonte)

A lógica que define se a aba está ativa ou inativa no Front-end (`.xhtml`) baseia-se em propriedades do Controller, como:
- `disabled="#{not atenderPacientesAgendadosController.habilitaAnamnese}"`

Essa propriedade é validada no back-end seguindo este fluxo:
1. **Controller:** `AtenderPacientesAgendadosController`
2. **Facade:** `IAmbulatorioFacade.validarProcessoConsultaAnamnese()`
3. **Objeto de Negócio (ON):** `AtendimentoPacientesAgendadosON.java`

No arquivo `AtendimentoPacientesAgendadosON.java`, o método `validarProcessoConsulta` verifica se o usuário logado possui permissão no módulo **CASCA** para um determinado "Processo".

## 2. Por que o código 46? (Parâmetros do Sistema)

O sistema não "chumba" o código 46 diretamente no código para Anamnese. Ele busca esse valor na tabela de parâmetros (`AGH_PARAMETROS`).

Os códigos identificados na base foram:
- **Anamnese:** parâmetro `P_SEQ_PROC_ANAMNESE` -> Valor **46**
- **Evolução:** parâmetro `P_SEQ_PROC_EVOLUCAO` -> Valor **11**
- **Atestado:** parâmetro `P_SEQ_PROC_ATESTADO` -> Valor **3**

## 3. Causa Raiz

As abas estavam inativas porque a tabela de permissões do CASCA (**`cse_perfil_processos`**) não possuía registros vinculando os perfis (ex: `MED01`) aos processos assistenciais acima (3, 11, 46). Sem esse vínculo, o método `cascaFacade.validarPermissaoConsultaPorServidorESeqProcesso` retorna sempre `false`.

## 4. Solução (SQLs de Habilitação)

Para habilitar as abas para os usuários que possuem o perfil **MED01** (médico), foram realizados os seguintes `INSERTs` (ou `UPDATEs` caso já existam):

```sql
-- Habilitar Atestado (Processo 3)
INSERT INTO agh.cse_perfil_processos 
(roc_seq, per_nome, ind_executa, ind_consulta, ind_assina, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ind_reg_medico_ant, ind_exige_em_atend, "version") 
VALUES (3, 'MED01', 'S', 'S', 'S', 'A', CURRENT_TIMESTAMP, 9999999, 955, 'N', 'N', 0);

-- Habilitar Evolução (Processo 11)
INSERT INTO agh.cse_perfil_processos 
(roc_seq, per_nome, ind_executa, ind_consulta, ind_assina, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ind_reg_medico_ant, ind_exige_em_atend, "version") 
VALUES (11, 'MED01', 'S', 'S', 'S', 'A', CURRENT_TIMESTAMP, 9999999, 955, 'N', 'N', 0);

-- Habilitar Anamnese (Processo 46)
INSERT INTO agh.cse_perfil_processos 
(roc_seq, per_nome, ind_executa, ind_consulta, ind_assina, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ind_reg_medico_ant, ind_exige_em_atend, "version") 
VALUES (46, 'MED01', 'S', 'S', 'S', 'A', CURRENT_TIMESTAMP, 9999999, 955, 'N', 'N', 0);

-- Habilitar CID/Procedimentos (Processos 10 e 19)
INSERT INTO agh.cse_perfil_processos 
(roc_seq, per_nome, ind_executa, ind_consulta, ind_assina, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ind_reg_medico_ant, ind_exige_em_atend, "version") 
VALUES (10, 'MED01', 'S', 'S', 'S', 'A', CURRENT_TIMESTAMP, 9999999, 955, 'N', 'N', 0);

INSERT INTO agh.cse_perfil_processos 
(roc_seq, per_nome, ind_executa, ind_consulta, ind_assina, ind_situacao, criado_em, ser_matricula, ser_vin_codigo, ind_reg_medico_ant, ind_exige_em_atend, "version") 
VALUES (19, 'MED01', 'S', 'S', 'S', 'A', CURRENT_TIMESTAMP, 9999999, 955, 'N', 'N', 0);
```

## 5. Habilitar Escolha de Procedimentos (Aba 4)

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

## 6. Fluxo para Habilitar Novos Procedimentos via Aplicação

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
