# Lab: Criando Imagens Customizadas com Azure Compute Galleries

## 📝 1. Objetivo do Laboratório
O objetivo deste laboratório foi criar imagens de máquinas virtuais personalizadas (Golden Images) para Windows e Linux, utilizando o **Azure Compute Galleries**. 

O processo consistiu em:
* Provisionar as VMs de template.
* Realizar o *hardening* e a instalação de softwares necessários (IIS e Nginx).
* Preparar os sistemas operacionais para generalização.
* Capturar essas imagens para armazenamento e replicação centralizada, permitindo o provisionamento rápido de novas instâncias padronizadas.

## 📐 2. Arquitetura do Cenário

### Componentes Utilizados:
* **Resource Groups:**
  * `rg-labAzure-vms` (Recursos de Computação e origem das VMs)
  * `rg-labAzure-mgmt` (Onde o Azure Compute Gallery foi implantado)
* **Azure Compute Galleries:**
  * `acgubuntuweb` (Galeria Linux)
    * **Definição de Imagem Linux:** `img-ubuntu-server-web`
  * `acgws2022web` (Galeria Windows) 
    * **Definição de Imagem Windows:** `img-windows-server-2022-web`
* **Máquinas Virtuais de Origem (Templates):**
  * 1x Instância `Standard_D2s_v3` (`VM-Web-Template-Win`) rodando Windows Server 2022.
  * 1x Instância `Standard_D2s_v3` (`VM-Web-Template-Lin`) rodando Ubuntu Server.


## 🚀 3. Passo a Passo Resumido

### Fluxo Geral do Laboratório:
1. **Preparação do Ambiente:** Criação dos recursos **Azure Compute Gallery** (`acgubuntuweb` e `acgws2022web`) dentro do grupo de VMs (`rg-labAzure-vms`).
2. **Provisionamento e Customização:** Criação das VMs originais (Windows e Linux) e instalação dos pacotes e serviços necessários (como IIS no Windows e Nginx no Linux).
3. **Backup de Segurança:** Criação de um *Snapshot* dos discos das VMs no portal do Azure antes de realizar o processo destrutivo de generalização.
4. **Generalização do Sistema Operacional (Remoção de Identidade):**
   * **No Windows:** Execução do utilitário `Sysprep` para remover informações específicas do computador (SID, drivers, etc.).
   * **No Linux:** Execução do utilitário `waagent` para desprovisionar a máquina e limpar o histórico de comandos.
5. **Captura e Publicação:** Captura das máquinas virtualizadas através do portal do Azure, gerando as versões de imagem diretamente no **Azure Compute Gallery**.


## 💻 4. Scripts e Comandos Úteis

### 4.1. Ambiente Windows (IIS & Sysprep)
Abaixo estão os comandos utilizados para preparar o servidor web e, em seguida, generalizar o sistema operacional para a captura da imagem:

```powershell
# Passo 1 Instalação do papel de Servidor Web (IIS) com as ferramentas de gerenciamento
Install-WindowsFeature -name Web-Server -IncludeManagementTools

# Passo 2 Remoção do arquivo HTML padrão do IIS (se ele existir)
Remove-Item C:\inetpub\wwwroot\iisstart.htm -ErrorAction SilentlyContinue

# Passo 3 Criação de um novo arquivo limpo (Set-Content evita duplicação de texto)
Set-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value "Azure Expert VM is running $($env:computername)"

# Passo 4 Execução do utilitário Sysprep para remover informações específicas (SID, drivers) e desligar a VM
%windir%\System32\Sysprep\Sysprep.exe /generalize /oobe /shutdown
```

### 4.2. Ambiente Linux (Nginx & Waagent)

```bash
# Passo 1 Atualização do gerenciador de pacotes e instalação do Nginx
sudo apt update && sudo apt install nginx -y

# Passo 2 Sobrescreve a página padrão com o hostname dinâmico do Linux
echo "Azure Expert VM is running $(hostname)" | sudo tee /var/www/html/index.html

# Passo 3 Execução do waagent para desprovisionar a máquina (remover chaves SSH e dados temporários de usuário)
sudo waagent -deprovision+user -force

# Passo 4 Limpeza do histórico de comandos do terminal antes do encerramento
history -c && exit
```
