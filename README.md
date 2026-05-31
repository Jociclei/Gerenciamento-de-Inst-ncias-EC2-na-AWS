# ☁️ Gerenciamento de Instâncias EC2 na AWS

> Repositório de documentação técnica desenvolvido como entregável do laboratório da DIO — **Gerenciamento de Instâncias EC2 na AWS**.

---

## 📋 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Conceitos Fundamentais](#-conceitos-fundamentais)
- [Tipos de Instâncias EC2](#-tipos-de-instâncias-ec2)
- [Ciclo de Vida de uma Instância](#-ciclo-de-vida-de-uma-instância)
- [Passo a Passo Prático](#-passo-a-passo-prático)
- [Opções de Armazenamento](#-opções-de-armazenamento)
- [Segurança e Redes](#-segurança-e-redes)
- [Monitoramento e Otimização de Custos](#-monitoramento-e-otimização-de-custos)
- [Boas Práticas](#-boas-práticas)
- [Insights e Aprendizados](#-insights-e-aprendizados)
- [Referências](#-referências)

---

## 📌 Sobre o Projeto

Este repositório documenta a jornada prática de aprendizado sobre o **Amazon EC2 (Elastic Compute Cloud)**, o serviço de computação em nuvem da AWS que permite criar e gerenciar servidores virtuais de forma escalável.

O objetivo é consolidar conhecimentos sobre como provisionar, configurar, monitorar e encerrar instâncias EC2, aplicando as melhores práticas do mercado.

---

## 📚 Conceitos Fundamentais

### O que é o Amazon EC2?

O **Amazon EC2** é um serviço web que fornece capacidade de computação segura e redimensionável na nuvem. Ele foi projetado para facilitar a computação em nuvem em escala web para os desenvolvedores.

| Conceito | Descrição |
|----------|-----------|
| **Instância** | Servidor virtual em execução na nuvem AWS |
| **AMI (Amazon Machine Image)** | Template com sistema operacional e configurações iniciais |
| **Par de Chaves** | Credenciais criptográficas para acesso seguro (SSH/RDP) |
| **Security Group** | Firewall virtual que controla o tráfego de entrada e saída |
| **VPC** | Rede virtual privada isolada dentro da AWS |
| **Região / AZ** | Localização geográfica / datacenter específico |

---

## 🖥️ Tipos de Instâncias EC2

As instâncias são organizadas em **famílias** otimizadas para diferentes casos de uso:

| Família | Caso de Uso | Exemplos |
|---------|-------------|----------|
| **Uso Geral** | Equilíbrio entre CPU, memória e rede | t3, t4g, m6i |
| **Otimizadas para Computação** | Alto desempenho de CPU | c6i, c7g |
| **Otimizadas para Memória** | Bancos de dados in-memory, caches | r6i, x2idn |
| **Otimizadas para Armazenamento** | Grandes volumes de I/O sequencial | i3, d3 |
| **Computação Acelerada** | Machine Learning, GPU | p4d, g5 |

> 💡 **Dica:** Para ambientes de estudo e desenvolvimento, as instâncias da família **t3.micro** ou **t3.small** geralmente são suficientes e elegíveis ao nível gratuito (Free Tier).

---

## 🔄 Ciclo de Vida de uma Instância

```
[Pending] ──► [Running] ──► [Stopping] ──► [Stopped]
                  │                              │
                  │                         [Terminated]
                  ▼
            [Shutting Down] ──► [Terminated]
```

| Estado | Descrição | Cobrança |
|--------|-----------|----------|
| **Pending** | Instância sendo inicializada | Não |
| **Running** | Em execução e disponível | ✅ Sim |
| **Stopping** | Sendo interrompida | Não |
| **Stopped** | Desligada (dados preservados) | Apenas armazenamento |
| **Terminated** | Encerrada permanentemente | Não |

> ⚠️ **Atenção:** Ao **terminar** uma instância, todos os dados no armazenamento efêmero (Instance Store) são perdidos permanentemente. Dados em volumes EBS podem ser preservados configurando a opção corretamente.

---

## 🛠️ Passo a Passo Prático

### 1. Acessar o Console da AWS

```
https://console.aws.amazon.com → EC2 → Instâncias
```

### 2. Criar uma Nova Instância

1. Clique em **"Executar instâncias"**
2. Defina um **nome** descritivo para a instância
3. Escolha uma **AMI** (ex: Amazon Linux 2023, Ubuntu 22.04)
4. Selecione o **tipo de instância** (ex: `t2.micro` para Free Tier)
5. Configure ou crie um **par de chaves** (`.pem`)
6. Configure as **configurações de rede** (VPC, sub-rede, IP público)
7. Defina as regras do **Security Group**
8. Configure o **armazenamento** (volume EBS)
9. Revise e clique em **"Executar instância"**

### 3. Conectar-se via SSH (Linux/Mac)

```bash
# Ajuste as permissões do arquivo de chave
chmod 400 minha-chave.pem

# Conecte-se à instância
ssh -i "minha-chave.pem" ec2-user@<IP-PÚBLICO-DA-INSTÂNCIA>
```

> Para instâncias Amazon Linux, use `ec2-user`. Para Ubuntu, use `ubuntu`.

### 4. Conectar-se via RDP (Windows)

1. No Console EC2, selecione a instância
2. Clique em **"Conectar" → "Cliente RDP"**
3. Faça download do arquivo `.rdp`
4. Descriptografe a senha do administrador usando o par de chaves
5. Abra o arquivo `.rdp` com as credenciais fornecidas

### 5. Gerenciar o Estado da Instância

```bash
# Listar instâncias via AWS CLI
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]' --output table

# Parar uma instância
aws ec2 stop-instances --instance-ids i-0123456789abcdef0

# Iniciar uma instância
aws ec2 start-instances --instance-ids i-0123456789abcdef0

# Terminar (encerrar) uma instância
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

---

## 💾 Opções de Armazenamento

| Tipo | Persistência | Melhor Para |
|------|--------------|-------------|
| **EBS (Elastic Block Store)** | Persistente | Sistema operacional, dados críticos |
| **Instance Store** | Temporário (perde com stop/terminate) | Cache, buffers temporários |
| **EFS (Elastic File System)** | Persistente, compartilhado | Múltiplas instâncias |
| **S3** | Persistente, objeto | Backups, arquivos estáticos |

---

## 🔒 Segurança e Redes

### Security Groups — Regras Importantes

```
# Exemplo: liberar acesso SSH apenas do seu IP
Tipo: SSH | Protocolo: TCP | Porta: 22 | Origem: MEU-IP/32

# Exemplo: liberar acesso HTTP público
Tipo: HTTP | Protocolo: TCP | Porta: 80 | Origem: 0.0.0.0/0

# Exemplo: liberar acesso HTTPS público
Tipo: HTTPS | Protocolo: TCP | Porta: 443 | Origem: 0.0.0.0/0
```

> ⚠️ **Nunca** abra a porta 22 para `0.0.0.0/0` (qualquer IP) em produção. Restrinja sempre ao seu IP ou use o AWS Systems Manager Session Manager para acesso sem SSH.

### IAM Roles para EC2

Sempre prefira usar **IAM Roles** (funções) em vez de credenciais de acesso hardcoded dentro das instâncias. Uma role permite que a instância acesse outros serviços AWS com segurança e de forma gerenciada.

---

## 📊 Monitoramento e Otimização de Custos

### CloudWatch — Métricas Básicas

| Métrica | O que Monitora |
|---------|----------------|
| `CPUUtilization` | % de uso de CPU |
| `NetworkIn / NetworkOut` | Tráfego de rede (bytes) |
| `DiskReadOps / DiskWriteOps` | Operações de disco |
| `StatusCheckFailed` | Falhas de integridade da instância |

### Dicas de Economia

- Use **instâncias Spot** para cargas de trabalho tolerantes a interrupções (até 90% mais barato)
- Use **Reserved Instances** para workloads previsíveis de 1 a 3 anos (até 72% de desconto)
- Configure **alarmes de billing** no CloudWatch para evitar surpresas na fatura
- Use o **AWS Cost Explorer** para analisar padrões de gastos
- Desligue instâncias fora do horário de uso com o **AWS Instance Scheduler**

---

## ✅ Boas Práticas

- [ ] Sempre usar **tags** para identificar e organizar recursos (ex: `Name`, `Environment`, `Project`)
- [ ] Nunca expor credenciais da AWS no código — use IAM Roles
- [ ] Habilitar **Multi-Factor Authentication (MFA)** na conta AWS
- [ ] Fazer backups regulares com **snapshots EBS** ou **AWS Backup**
- [ ] Usar instâncias em **múltiplas Zonas de Disponibilidade** para alta disponibilidade
- [ ] Revisar periodicamente as regras de Security Groups para remover acessos desnecessários
- [ ] Habilitar **VPC Flow Logs** para auditoria de tráfego de rede
- [ ] Utilizar o **princípio do menor privilégio** ao definir permissões IAM

---

## 💡 Insights e Aprendizados

Durante a prática com o EC2, os principais aprendizados foram:

**1. Elasticidade é o diferencial da nuvem**
A capacidade de aumentar ou diminuir recursos computacionais em minutos — sem precisar adquirir hardware físico — é o maior valor do EC2. Isso permite que times pequenos gerenciem infraestrutura antes restrita a grandes empresas.

**2. Segurança é responsabilidade compartilhada**
A AWS protege a infraestrutura, mas a configuração segura dos Security Groups, IAM e acessos à instância é responsabilidade do usuário. Configurações inadequadas são a principal causa de incidentes de segurança.

**3. Custo precisa de atenção constante**
Instâncias esquecidas em estado "Running" geram custos contínuos. Desenvolver o hábito de monitorar e desligar recursos não utilizados é essencial.

**4. AMIs aceleram a padronização**
Criar AMIs customizadas com as configurações e software já instalados elimina erros manuais e acelera o provisionamento de novos ambientes.

**5. CLI e IaC são o caminho para produção**
Embora o Console AWS seja ótimo para aprendizado, ambientes reais exigem automação via AWS CLI, CloudFormation ou Terraform para garantir reprodutibilidade e rastreabilidade.

---

## 📎 Referências

- [Documentação oficial do Amazon EC2](https://docs.aws.amazon.com/pt_br/ec2/index.html)
- [Gerenciando instâncias EC2 — AWS Toolkit for Visual Studio](https://docs.aws.amazon.com/pt_br/toolkit-for-visual-studio/latest/user-guide/tkv-ec2-ami.html)
- [Tipos de instâncias EC2](https://aws.amazon.com/pt/ec2/instance-types/)
- [Preços do EC2](https://aws.amazon.com/pt/ec2/pricing/)
- [AWS Free Tier](https://aws.amazon.com/pt/free/)
- [GitHub Quick Start — DIO](https://github.com/digitalinnovationone/github-quickstart)
- [Documentação do GitHub](https://docs.github.com/)

---

<div align="center">

Desenvolvido como parte do laboratório da **[DIO — Digital Innovation One](https://www.dio.me/)**

</div>
