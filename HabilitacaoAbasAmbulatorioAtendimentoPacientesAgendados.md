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

> **Nota:** É necessário que o usuário logado possua o perfil **MED01** associado ao seu login na tabela `cse_usuario_perfis` e realize um novo Login após a execução desses SQLs.
