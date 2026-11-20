# 🚀 Atividade 4 | Pipeline CI/CD com GitHub Actions e AWS CloudFormation

**Disciplina:** Desenvolvimento de Software e Operações de TI - DevOps

**Aluno:** André Moreira Araújo dos Santos

**Instituição:** Faculdade Unyleya

**Região AWS:** São Paulo (`sa-east-1`)

**Data da implementação:** 20/11/2026

## 1. Apresentação do projeto

Este projeto foi desenvolvido como parte da Atividade 4 da disciplina de Desenvolvimento de Software e Operações de TI (DevOps).

O objetivo é implementar um pipeline de Integração Contínua e Entrega Contínua (CI/CD) utilizando **GitHub Actions e AWS CloudFormation** para automatizar o provisionamento da infraestrutura criada na Atividade 3.

A solução permite validar o código de infraestrutura, autenticar de maneira segura na AWS e criar os recursos necessários para executar uma aplicação Python integrada a um banco de dados MySQL.

O projeto reutiliza o template CloudFormation da atividade anterior, demonstrando a evolução de uma implantação manual para um processo automatizado, versionado e reproduzível.

## 2. Tecnologias utilizadas

| Tecnologia | Finalidade |

|---|---|

| Git | Controle de versões |

| GitHub | Repositório do código-fonte |

| GitHub Actions | Automação do pipeline CI/CD |

| AWS CloudFormation | Infraestrutura como Código |

| AWS IAM | Gerenciamento de identidades e permissões |

| OpenID Connect (OIDC) | Autenticação federada entre GitHub e AWS |

| Amazon VPC | Isolamento e organização da rede |

| Amazon EC2 | Execução da aplicação Python |

| Application Load Balancer | Recebimento e encaminhamento das requisições HTTP |

| Amazon RDS | Hospedagem do banco MySQL |

| AWS Secrets Manager | Gerenciamento das credenciais do banco |

| Amazon Systems Manager | Suporte ao gerenciamento da instância EC2 |

## 3. Arquitetura da solução

A arquitetura possui dois fluxos principais: implantação da infraestrutura e funcionamento da aplicação.

### 3.1. Pipeline CI/CD

```text
GitHub Repository
       |
       v
 GitHub Actions
       |
       | Autenticacao OIDC
       v
     AWS IAM
       |
       v
 AWS CloudFormation
       |
       v
   Stack AWS
       |
       +--- Amazon VPC
       +--- Sub-redes publicas e privadas
       +--- Amazon EC2
       +--- Application Load Balancer
       +--- Amazon RDS MySQL
       +--- Security Groups
       +--- Funcoes IAM
```

O GitHub Actions utiliza o CloudFormation para provisionar ou atualizar a infraestrutura definida no arquivo `template.yaml`.

### 3.2. Fluxo da aplicação

```text
       Navegador
           |
           | HTTP
           v
 Application Load Balancer
           |
           | Porta 8000
           v
      Amazon EC2
    Aplicacao Python
           |
           | MySQL / TLS
           | Porta 3306
           v
     Amazon RDS
     Banco MySQL
```

A aplicação Python executada na EC2 recebe as requisições encaminhadas pelo Load Balancer.

O banco RDS está localizado em sub-redes privadas e aceita conexões da aplicação por meio de regras específicas de segurança.

A comunicação com o MySQL utiliza TLS com validação do certificado.

## 4. Estrutura do repositório

```text
devops-unyleya-atividade4/
|
|-- .github/
|   |-- workflows/
|       |-- deploy.yml
|
|-- .gitignore
|-- README.md
|-- template.yaml
```

### Descrição dos arquivos

**`template.yaml`**

Define os recursos de infraestrutura utilizando AWS CloudFormation.

Inclui a configuração da rede, grupos de segurança, EC2, Load Balancer, RDS e funções IAM.

O código da aplicação Python também está incorporado ao template e é instalado automaticamente na EC2 por meio do UserData.

**`.github/workflows/deploy.yml`**

Define o pipeline executado pelo GitHub Actions.

Contém as etapas de autenticação, validação do template, implantação e confirmação do resultado.

**`.gitignore`**

Evita o versionamento acidental de arquivos temporários e credenciais locais.

**`README.md`**

Apresenta a documentação técnica, os procedimentos de implantação e os resultados obtidos.

## 5. Funcionamento do pipeline

O pipeline possui dois modos de execução.

### 5.1. Validação automática

A cada `push` na branch `main`, o GitHub Actions executa:

1. Download do código do repositório.

2. Autenticação na AWS utilizando OIDC.

3. Confirmação da identidade AWS.

4. Validação do template CloudFormation.

5. Registro do resultado da validação.

Esse processo não provisiona recursos.

O objetivo é verificar a integração entre GitHub e AWS e validar o template antes de uma implantação.

### 5.2. Implantação manual

A implantação é acionada pela opção **Run workflow**, disponível na aba Actions do GitHub.

Nesse modo, o pipeline executa as etapas de validação e também:

1. Recupera a variável `ACCESS_CIDR`.

2. Executa o comando `aws cloudformation deploy`.

3. Cria ou atualiza a stack `unyleya-lab4`.

4. Aguarda a conclusão do CloudFormation.

5. Consulta e apresenta o status final da stack.

Essa separação permite controlar o provisionamento e reduzir o risco de criação desnecessária de recursos cobrados.

## 6. Autenticação e segurança

A integração entre GitHub Actions e AWS utiliza **OpenID Connect (OIDC)**.

Esse mecanismo permite obter credenciais temporárias por meio do AWS Security Token Service, sem armazenar chaves de acesso permanentes no repositório.

A função utilizada pelo GitHub Actions é:

`GitHubActions-Unyleya-DeployRole`

A política de confiança restringe a autenticação ao repositório:

`euandremas/devops-unyleya-atividade4`

E à branch:

`main`

O CloudFormation utiliza uma função de execução específica:

`DevOps-Unyleya-CloudFormationRole`

### Controles adicionais

- Banco RDS sem acesso público.

- Grupos de segurança com comunicação restrita entre componentes.

- Aplicação EC2 sem necessidade de acesso SSH.

- Senha do MySQL gerenciada pelo AWS Secrets Manager.

- Comunicação TLS entre aplicação e banco.

- Volumes e armazenamento do banco criptografados.

- Metadados da EC2 configurados para exigir IMDSv2.

- Acesso HTTP ao Load Balancer restrito ao CIDR autorizado.

**Observação:** o acesso HTTP é utilizado exclusivamente para demonstração acadêmica. Uma implantação destinada à produção deverá utilizar HTTPS e controles adicionais de segurança.

## 7. Pré-requisitos

Para reproduzir o ambiente, são necessários:

- Conta AWS com acesso aos serviços utilizados.

- Repositório GitHub com GitHub Actions habilitado.

- Provedor OIDC do GitHub configurado no AWS IAM.

- Função IAM com política de confiança para o repositório e a branch autorizados.

- Permissões para executar o CloudFormation e provisionar os recursos definidos no template.

- Função de execução do CloudFormation configurada.

- Região AWS `sa-east-1`.

A variável de repositório `ACCESS_CIDR` deve conter o IP público autorizado no formato CIDR.

Exemplo de formato:

`203.0.113.10/32`

O endereço acima é apenas ilustrativo e deve ser substituído pelo IP público de quem acessará a aplicação.

A variável deve ser cadastrada em:

**Settings → Secrets and variables → Actions → Variables**

## 8. Como executar o projeto

### 8.1. Clonar o repositório

```bash
git clone https://github.com/euandremas/devops-unyleya-atividade4.git
cd devops-unyleya-atividade4
```

### 8.2. Validar automaticamente

Após configurar as permissões necessárias, uma alteração enviada à branch `main` acionará o workflow.

O resultado poderá ser acompanhado em:

**GitHub → Actions → Atividade 4 - DevOps CI/CD**

### 8.3. Executar o deploy

Para implantar a infraestrutura:

1. Acesse a aba Actions.

2. Selecione `Atividade 4 - DevOps CI/CD`.

3. Clique em `Run workflow`.

4. Selecione a branch `main`.

5. Inicie a execução.

6. Acompanhe os logs até a conclusão.

O workflow utiliza o comando `aws cloudformation deploy` para implantar a stack.

**Nome da stack:** `unyleya-lab4`

**Região:** `sa-east-1`

**Zonas de disponibilidade:** `sa-east-1a` e `sa-east-1b`

### 8.4. Verificar a infraestrutura

No AWS Console:

**CloudFormation → Pilhas → unyleya-lab4**

Verifique o status:

`CREATE_COMPLETE`

Na aba **Recursos**, é possível consultar os componentes provisionados.

Na aba **Saídas**, estão disponíveis o endereço da aplicação e outras informações úteis para os testes.

## 9. Testes realizados

A implantação foi executada em 20/11/2026, utilizando o GitHub Actions.

### 9.1. Validação do pipeline

**Execução:** GitHub Actions #3

**Resultado:** Sucesso

**Duração:** 11 minutos e 34 segundos.

O log apresentou:

```text
Successfully created/updated stack - unyleya-lab4
```

A consulta posterior confirmou:

```text
unyleya-lab4
CREATE_COMPLETE
```

### 9.2. Teste da aplicação

Após o provisionamento, a aplicação Python foi acessada pelo endereço do Application Load Balancer.

A interface foi carregada corretamente, permitindo consultar o banco RDS MySQL.

### 9.3. Teste de conexão com MySQL

Resultados observados:

| Verificação | Resultado |

|---|---|

| Banco de dados | atividade3 |

| Versão MySQL | 8.4.11 |

| Consulta ao RDS | Sucesso |

| TLS | Ativo |

| Cifra TLS | TLS_AES_256_GCM_SHA384 |

### 9.4. Teste de persistência

Foi realizada a gravação de um registro fictício por meio da aplicação.

**Registro criado:**

| Campo | Valor |

|---|---|

| ID | 1 |

| Descrição | Registro fictício da demonstração de IaC |

| Data | 20/11/2026 |

Após a gravação, foi executada uma nova consulta.

O registro permaneceu disponível no banco.

**Resultado:** operações de leitura e gravação no Amazon RDS MySQL comprovadas.

## 10. Evidências da implementação

As evidências registradas durante a atividade incluem:

- Execução automática do GitHub Actions.

- Autenticação OIDC realizada com sucesso.

- Identidade AWS confirmada.

- Validação do template CloudFormation.

- Execução manual do pipeline.

- Provisionamento da stack `unyleya-lab4`.

- Status `CREATE_COMPLETE` no AWS CloudFormation.

- Criação dos recursos EC2, ALB e RDS.

- Aplicação Python acessível pelo Load Balancer.

- Consulta ao banco MySQL utilizando TLS.

- Gravação e persistência de um registro fictício.

Essas evidências demonstram a execução prática do pipeline e o funcionamento da infraestrutura provisionada.

## 11. Exclusão da infraestrutura

**Atenção: este laboratório utiliza recursos AWS que podem gerar custos.**

Após os testes e a coleta de evidências, os recursos devem ser removidos.

A exclusão pode ser realizada pelo AWS CloudFormation.

No console:

**CloudFormation → Pilhas → unyleya-lab4 → Excluir**

Alternativamente, por meio do AWS CLI:

```bash
aws cloudformation delete-stack \
  --stack-name unyleya-lab4 \
  --region sa-east-1
```

Para acompanhar a conclusão:

```bash
aws cloudformation wait stack-delete-complete \
  --stack-name unyleya-lab4 \
  --region sa-east-1
```

Após a exclusão, é necessário verificar se não permaneceram recursos cobrados, incluindo instâncias, volumes, snapshots, Load Balancers e outros componentes eventualmente criados fora da stack.

**Importante:** a exclusão da stack remove o banco de dados e os registros fictícios do laboratório, conforme as políticas de exclusão definidas no template.

A remoção da infraestrutura não é executada automaticamente pelo workflow atual.

## 12. Demonstração em vídeo

A apresentação em vídeo foi gravada e demonstra a estrutura do repositório, os trechos principais do `template.yaml` e do `.github/workflows/deploy.yml`, a autenticação OIDC, a validação automática, o deploy manual e o teste de persistência no RDS MySQL.

**Link do vídeo:** a inserir após a publicação da gravação em uma plataforma acessível ao avaliador.

### Evidências no GitHub Actions

- [Histórico de execuções do workflow](https://github.com/euandremas/devops-unyleya-atividade4/actions)
- **Execução #3:** implantação manual da stack `unyleya-lab4`, com confirmação `CREATE_COMPLETE`.
- **Execução #4:** validação automática após o commit da documentação, sem novo provisionamento.

## 13. Considerações finais

A Atividade 4 demonstrou a aplicação prática de conceitos de DevOps, automação e Infraestrutura como Código.

A solução implementada integrou GitHub Actions, autenticação OIDC e AWS CloudFormation para realizar o provisionamento controlado de recursos AWS.

Os testes comprovaram a criação da infraestrutura, o funcionamento da aplicação Python e a comunicação com o banco Amazon RDS MySQL.

O projeto também demonstrou a importância do versionamento, do controle de permissões e da validação automatizada no desenvolvimento de soluções em nuvem.

**Resultado final:** pipeline CI/CD implementado e testado com sucesso, realizando o provisionamento da infraestrutura originalmente desenvolvida na Atividade 3.

---

**Autor:** André Moreira Araújo dos Santos

**Curso:** Análise e Desenvolvimento de Sistemas

**Instituição:** Faculdade Unyleya

**Disciplina:** Desenvolvimento de Software e Operações de TI - DevOps

**Ano:** 2026
