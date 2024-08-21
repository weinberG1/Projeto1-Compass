# Atividade 1 - Linux 

Este repositório contém a atividade desenvolvida durante a Sprint 4 do Programa de Bolsas DevSecOps - JUN 2024. O projeto tem como objetivo a criação de um ambiente Linux na AWS, utilizando os serviços EC2, Elastic IP, Servidor Web Apache e automação com scripts.

## Passo a Passo na AWS

### Etapa 1: Gerando Chaves de Acesso

1. **Acesse o serviço EC2:** No console da AWS, navegue até `Serviços` > `Computação` > `EC2`.
2. **Gerencie Pares de Chaves:** Na barra lateral, clique em `Rede e Segurança` > `Pares de chaves`.
3. **Crie um Novo Par de Chaves:**
   - Clique em **`Criar par de chaves`**.
   - **Nome:** `"sua-chave"`
   - **Tipo:** `RSA`
   - **Formato:** `.pem`
   - **Adicione as seguintes tags:**
     - **CostCenter:** `C092000024`
     - **Project:** `PB - JUN 2024`

### Etapa 2: Criando a Instância EC2

1. **Inicie uma Nova Instância:** No serviço EC2, clique em `Instâncias` > `Executar instâncias`.
2. **Configure a Instância:**
   - **Tags:**
     - **CostCenter:** `C092000024`
     - **Project:** `PB - JUN 2024`
     - **Tipos de recursos:** `Instâncias e Volumes`
   - **Imagem:** `Amazon Linux 2`
   - **Tipo:** `t3.small`
   - **Par de chaves:** `"sua-chave"`
   - **Rede:**
     - **VPC:** Selecione uma VPC com acesso à Internet.
     - **Sub-rede:** Escolha uma sub-rede com acesso à Internet.
     - **IP Público:** `Desabilitar`
     - **Firewall:** `Criar grupo de segurança`
       - **Nome:** `"qualquer"`
       - **Descrição:** `Acesso público: (22/TCP, 111/TCP e UDP, 2049/TCP/UDP, 80/TCP, 443/TCP)`
   - **Armazenamento:**
     - **Tamanho:** `16GB`
     - **Tipo:** `gp2`

### Etapa 3: Alocando um IP Elástico

1. **Acesse IPs Elásticos:** No serviço EC2, navegue até `Rede e Segurança` > `IPs elásticos`.
2. **Aloque um Endereço IP:** Clique em **`Alocar endereço IP elástico`**.
   - **Conjunto de endereços:** `Conjunto de endereços IPv4 da Amazon`
   - **Grupo de borda de rede:** `us-east-1`
   - **Tags:**
     - **CostCenter:** `C092000024`
     - **Project:** `PB - JUN 2024`
3. **Associe o IP à Instância:**
   - Selecione o IP elástico alocado.
   - Clique em `Ações` > **`Associar endereço IP elástico`**.
   - **Tipo de recurso:** `Instância`
   - **Instância:** Selecione a instância criada.
   - **Endereço IP privado:** Escolha o IP privado da instância.

### Etapa 4: Criando o Sistema de Arquivos EFS

1. **Acesse o serviço EFS:** No console da AWS, vá para `Serviços` > `Armazenamento` > `EFS`.
2. **Crie um Sistema de Arquivos:**
   - **Nome:** `"nome-do-projeto"`
   - **VPC:** Selecione a mesma VPC da instância EC2.
3. **Configure a Rede do EFS:**
   - Selecione o EFS criado.
   - Clique em **`Visualizar detalhes`**.
   - Navegue até a seção `Rede` > **`Gerenciar`**.
   - Defina o **"Grupo de segurança"** como `"nome-do-projeto"`.
4. **Obtenha as Instruções de Montagem:**
   - Selecione o EFS.
   - Clique em `Anexar` > **`Montar via IP`**.
   - Anote o código fornecido (será usado na própria instância EC2).

## Passo a Passo no Linux

### Etapa 5: Conectando-se à Instância

1. **Acesse a Instância EC2:** No serviço EC2, selecione a instância e clique em **`Conectar`**.
2. **Utilize o EC2 Instance Connect** (recomendado) ou SSH.

### Etapa 6: Configurando o Ambiente Linux

1. **Atualize o Sistema:**
   ```bash
   sudo yum update && sudo yum upgrade -y
   ```
2. **Instale o NFS:**
   ```bash
   sudo yum install nfs-utils -y
   ```
3. **Crie o Diretório de Montagem:**
   ```bash
   sudo mkdir /efs
   ```
4. **Monte o EFS:**
   - Utilize o código obtido na **Etapa 4** (substitua os valores entre colchetes):
     ```bash
     sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [IP do EFS]:/ [Diretório de montagem]
     ```
5. **Verifique a Montagem:**
   ```bash
   df -h
   ```
6. **Crie um Diretório no EFS:**
   ```bash
   sudo mkdir /efs/seu_nome
   ```
7. **Instale o Apache:**
   ```bash
   sudo yum install httpd -y
   ```
8. **Inicie o Apache:**
   ```bash
   sudo systemctl start httpd
   ```

### Etapa 7: Criando o Script de Validação

1. **Crie o Arquivo do Script:**
   ```bash
   sudo nano check_apache.sh
   ```
2. **Insira o Código do Script:**
   ```bash
   #!/bin/bash
   STATUS=$(systemctl is-active httpd)
   TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
   if [ "$STATUS" = "active" ]; then
       echo "$TIMESTAMP Apache ONLINE - Tudo Certo" >> /home/ec2-user/efs/seu_nome/apache_online.log
   else
       echo "$TIMESTAMP Apache OFFLINE - Verifique o servidor" >> /home/ec2-user/efs/seu_nome/apache_offline.log
   fi
   ```
3. **Defina as Permissões do Script:**
   ```bash
   sudo chmod 755 check_apache.sh
   ```

### Etapa 8: Automatizando a Execução do Script com o Crontab

1. **Edite o Crontab:**
   ```bash
   sudo crontab -e
   ```
2. **Adicione a Tarefa Cron:**
   ```
   */5 * * * * /home/ec2-user/check_apache.sh
   ```
   - Esta configuração executará o script a cada 5 minutos.
