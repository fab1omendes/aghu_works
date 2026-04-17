# Configuração de Cabeçalho e Dados Hospitalares nas Impressões

Este documento descreve onde alterar as informações que aparecem no cabeçalho dos relatórios e documentos impressos do AGHU (Logos, Endereço, Nome do Hospital e CNES).

## 1. Dados de Texto (Nome, Endereço, Telefone)
A maioria das informações textuais do cabeçalho é controlada por **Parâmetros do Sistema**.

**Caminho no Sistema:** `Configuração > Administração > Parâmetros`

| Parâmetro | Descrição |
| :--- | :--- |
| **P_HOSPITAL_RAZAO_SOCIAL** | Nome do Hospital (Ex: Instituto Fernandes Figueira) |
| **P_HOSPITAL_END_LOGRADOURO** | Endereço (Rua, Número, Bairro) |
| **P_HOSPITAL_END_CIDADE** | Cidade do Hospital |
| **P_AGHU_UF_SEDE_HU** | UF (Estado) do Hospital |
| **P_HOSPITAL_END_FONE** | Telefone de contato |
| **P_HOSPITAL_END_CEP** | CEP do Hospital |

---

## 2. Imagens e Logotipos (EBSERH, SUS, Hospital)
As imagens podem ser configuradas via parâmetros (apontando para um caminho no servidor) ou substituindo os arquivos físicos no pacote da aplicação.

### Via Parâmetros (Caminho no Servidor)
| Parâmetro | Descrição |
| :--- | :--- |
| **P_AGHU_LOGO_HOSPITAL_RELATIVO_JEE7** | Caminho para o logotipo principal do Hospital | /opt/aghu/jboss/server/default/conf/logo-hospital.png
| **P_AGHU_LOGO_EE** | Caminho para o logotipo da EBSERH |
| **P_AGHU_LOGO_SUS** | Caminho para o logotipo do SUS |

### Via Arquivo Físico
As imagens padrão costumam residir no diretório de recursos do WAR:
`aghu.war/resources/img/`
*Arquivos comuns: `logo-ebserh.png`, `logoSus.jpg`, `logotipos-relatorios.png`.*

---

## 3. CNES e Definição de Instituição Local
Para que o CNES apareça corretamente e o sistema saiba qual instituição deve usar como cabeçalho principal, é necessário configurar a **Instituição Local**.

**Caminho no Sistema:** `Internação > Cadastros Básicos > Instituição Hospitalar`

**Passo a passo:**
1.  Pesquise pela instituição correspondente ao seu hospital.
2.  No formulário de edição, preencha o campo **CNES** (7 dígitos).
3.  **IMPORTANTE:** Marque o campo **Indicador Instituição Local**.
    *   *Sem esse check, o relatório pode não buscar os dados corretos para o cabeçalho.*
4.  Clique em **Gravar**.

> [!TIP]
> Algumas consultas de ambulatório e faturamento podem utilizar parâmetros específicos de CNES como `P_CNES_HCPA` ou `P_UBS_ESUS_CNES`, mas a configuração da **Instituição Local** é o padrão global para a maioria dos cabeçalhos de documentos médicos.
