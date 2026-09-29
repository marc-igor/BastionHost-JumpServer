🔐 Bastion Host & PAM com JumpServer + Active Directory

Implementação de uma arquitetura de Privileged Access Management (PAM) utilizando JumpServer integrado ao Active Directory (AD) para centralizar, controlar, monitorar e auditar o acesso a servidores Linux e Windows.

O projeto foi desenvolvido com o objetivo de reduzir a exposição direta dos servidores, eliminando o acesso administrativo direto via SSH/RDP e estabelecendo o JumpServer como ponto central de acesso à infraestrutura.

Status: Implementado
Categoria: Cybersecurity / Infrastructure / PAM
Tecnologias: JumpServer · Active Directory · LDAP · Linux · Windows Server · SSH · RDP · MFA

🎯 Objetivos

A implementação teve como principais objetivos:

Centralizar o acesso administrativo à infraestrutura.

Eliminar o acesso direto dos usuários aos servidores críticos.

Integrar autenticação e identidade com o Active Directory.

Aplicar controle de acesso baseado em grupos e permissões.

Implementar MFA para os usuários.

Restringir a execução de comandos potencialmente destrutivos.

Registrar e auditar sessões administrativas.

Aumentar a rastreabilidade das ações realizadas nos servidores.

Reduzir a superfície de ataque da infraestrutura.

🏗️ Arquitetura

O JumpServer atua como Bastion Host / Gateway de acesso, intermediando as conexões entre os usuários e os ativos da infraestrutura.

                         ┌─────────────────────┐
                         │       Usuário       │
                         │                     │
                         │ Browser / Cliente   │
                         └──────────┬──────────┘
                                    │
                                    │ HTTPS
                                    ▼
                         ┌─────────────────────┐
                         │     JumpServer      │
                         │                     │
                         │  PAM / Bastion Host │
                         │  MFA                │
                         │  ACLs               │
                         │  Session Recording  │
                         └──────┬─────────┬────┘
                                │         │
                       LDAP/AD  │         │ SSH / RDP
                                │         │
                                ▼         ▼
                    ┌───────────────┐   ┌──────────────────┐
                    │ Active        │   │ Infraestrutura   │
                    │ Directory     │   │                  │
                    │               │   │ Linux            │
                    │ ads.local     │   │ Windows Server   │
                    └───────────────┘   │ AD / DHCP / etc. │
                                        └──────────────────┘


O fluxo básico de acesso é:

Usuário
   │
   ▼
Autenticação no JumpServer
   │
   ▼
Validação de identidade no AD
   │
   ▼
MFA
   │
   ▼
Validação de permissões / ACL
   │
   ▼
Acesso ao Asset
   │
   ▼
Sessão monitorada e registrada

🔑 Integração com Active Directory

O JumpServer foi integrado ao Active Directory através de LDAP, permitindo centralizar a identidade dos usuários.

Configurações utilizadas

Diretório: ads.local

Protocolo: LDAP

Autenticação: Active Directory

Sincronização: usuários e grupos

Controle de importação: baseado em associação aos grupos autorizados

Foi utilizado filtro baseado no atributo memberOf para limitar a sincronização aos usuários pertencentes aos grupos definidos para acesso à infraestrutura.

Exemplo conceitual:

(&(objectClass=user)
  (memberOf=CN=grpTI,OU=Groups,DC=ads,DC=local))


Dessa forma, usuários fora dos grupos autorizados não são automaticamente disponibilizados para acesso através do JumpServer.

🛡️ Hardening e Controles de Segurança

Foram implementadas políticas adicionais para aumentar a segurança da plataforma.

Política de senhas

Comprimento mínimo configurado entre 10 e 12 caracteres.

Requisitos de complexidade.

Rotação periódica.

Controle de tentativas de autenticação.

MFA

Foi habilitada autenticação multifator para os usuários utilizando mecanismos como:

Virtual MFA;

MFA via e-mail.

O objetivo é adicionar uma segunda camada de autenticação além da credencial tradicional.

Proteção contra força bruta

Foram aplicados mecanismos de proteção contra tentativas repetidas de autenticação, incluindo:

Bloqueio temporário de contas;

Bloqueio/restrição de IPs;

Limitação de tentativas;

Controle de origem das conexões.

Restrição por IP

O acesso à plataforma também pode ser limitado através de IP Whitelisting, permitindo que somente origens previamente autorizadas consigam alcançar o serviço.

🚦 Controle de Acesso

O controle de acesso foi estruturado utilizando uma combinação de:

Usuários;

Grupos;

Assets;

Account;

Permissões;

ACLs;

Políticas de acesso.

A ideia é aplicar o princípio de:

"Usuário somente acessa o recurso necessário para executar sua função."

Exemplo conceitual:

grpTI
 │
 ├── JumpServer
 │
 ├── Linux Servers
 │      ├── SSH
 │      └── Shell
 │
 └── Windows Servers
        └── RDP

⛔ Command Filter ACL

Um dos mecanismos utilizados foi o controle de comandos através de Command Filter ACLs.

O objetivo é impedir que determinados comandos ou ações administrativas sejam executados durante as sessões.

Exemplos de categorias que podem ser restringidas:

Linux
├── Comandos destrutivos
├── Alterações críticas de configuração
├── Manipulação indevida de usuários
└── Ações administrativas não autorizadas

Windows
├── Comandos destrutivos
├── Alterações críticas do sistema
├── Manipulação de serviços
└── Operações administrativas restritas


Esse mecanismo adiciona uma camada de controle entre o usuário autenticado e o sistema operacional.

🖥️ Gerenciamento de Assets

Os servidores da infraestrutura foram cadastrados no JumpServer para que o acesso administrativo pudesse ser realizado através da plataforma.

Exemplos de ativos utilizados no projeto:

Asset	Sistema	Função
ADDS03	Windows Server	Active Directory / Domain Controller
DHCP02	Windows Server	DHCP
UbuntuS01	Ubuntu Linux	Docker / Serviços
Outros servidores	Linux / Windows	Infraestrutura
Monitoramento

Durante as sessões, o ambiente permite acompanhar informações relacionadas ao recurso utilizado, como:

CPU;

Memória;

Disco;

Status da sessão;

Usuário conectado;

Asset acessado;

Horário da conexão.

🎥 Auditoria e Gravação de Sessões

Um dos principais objetivos da arquitetura é proporcionar rastreabilidade das atividades administrativas.

As sessões realizadas através do JumpServer podem ser registradas e auditadas, permitindo identificar:

QUEM
 │
 ├── Usuário autenticado
 │
 ▼
ACESSOU O QUÊ
 │
 ├── Servidor
 ├── Protocolo
 └── Conta utilizada
 │
 ▼
QUANDO
 │
 ├── Data
 └── Horário
 │
 ▼
O QUE FOI FEITO
 │
 ├── Comandos
 └── Atividades da sessão


Essa visibilidade facilita processos de:

Auditoria;

Investigação de incidentes;

Troubleshooting;

Compliance;

Análise de atividades administrativas.

🔒 Modelo de Acesso

Antes da implementação:

Usuário ───────► SSH ───────► Servidor
Usuário ───────► RDP ───────► Servidor


Após a implementação:

Usuário
   │
   ▼
JumpServer
   │
   ├──► Autenticação
   ├──► MFA
   ├──► ACL
   ├──► Command Filter
   ├──► Auditoria
   └──► Session Recording
          │
          ▼
       Servidor


A mudança permite centralizar as decisões de acesso e reduzir a quantidade de pontos de entrada administrativos expostos diretamente na infraestrutura.

🧩 Tecnologias
Tecnologia	Utilização
JumpServer	PAM / Bastion Host
Active Directory	Gestão de identidade
LDAP	Integração entre JumpServer e AD
Linux	Servidores e workloads
Windows Server	Servidores corporativos
SSH	Acesso administrativo Linux
RDP	Acesso administrativo Windows
Docker	Execução de workloads
MFA	Segundo fator de autenticação
📋 Controles Implementados

 Bastion Host

 Integração LDAP/Active Directory

 Sincronização de usuários e grupos

 Filtro por memberOf

 MFA

 Política de senhas

 Proteção contra brute force

 Restrição por IP

 Controle de acesso baseado em grupos

 Command Filter ACL

 Centralização de Assets

 Monitoramento de sessões

 Gravação/auditoria de sessões

 Controle de acesso SSH

 Controle de acesso RDP

📈 Resultados

A arquitetura implementada proporciona:

Centralização: um único ponto para gerenciamento dos acessos administrativos.

Rastreabilidade: identificação de usuários, ativos e sessões.

Controle: aplicação de políticas e restrições antes e durante o acesso.

Auditoria: registro das atividades realizadas.

Redução de exposição: diminuição da necessidade de acesso direto aos servidores.

Governança: maior controle sobre contas privilegiadas e recursos críticos.

🛡️ Segurança e Compliance

A arquitetura foi projetada considerando princípios de segurança como:

Least Privilege (Privilégio Mínimo);

Defense in Depth (Defesa em Profundidade);

Accountability;

Segregação de acesso;

Auditoria e rastreabilidade;

MFA;

Controle de sessões administrativas.

Esses controles podem contribuir para processos internos de governança e requisitos relacionados à LGPD e a frameworks de segurança como a ISO/IEC 27001, dependendo do contexto organizacional e dos demais controles implementados no ambiente.

📁 Estrutura do Projeto
.
├── README.md
├── docs/
│   ├── architecture/
│   ├── screenshots/
│   └── diagrams/
├── configs/
│   ├── ldap/
│   ├── acl/
│   └── policies/
└── assets/
    └── documentation/


Nota: arquivos contendo credenciais, tokens, senhas, chaves privadas ou informações reais de infraestrutura não devem ser versionados neste repositório.

⚠️ Observações de Segurança

Este projeto envolve conceitos de infraestrutura e administração privilegiada. Por isso:

Não armazenar credenciais reais no Git.

Não publicar chaves privadas.

Não expor endereços IP públicos desnecessariamente.

Não publicar informações sensíveis do Active Directory.

Utilizar dados fictícios ou sanitizados nos exemplos.

Aplicar as configurações em ambientes controlados antes de utilizá-las em produção.

📚 Conceitos Demonstrados

Este projeto demonstra conhecimentos práticos em:

Cybersecurity

PAM

Bastion Host

MFA

Hardening

Access Control

Session Auditing

Least Privilege

Infrastructure

Linux

Windows Server

Active Directory

LDAP

SSH

RDP

Docker

Governance

Gestão de identidades

Controle de privilégios

Auditoria

Rastreabilidade

Políticas de acesso

Governança de infraestrutura

🚀 Conclusão

A implementação do JumpServer como camada intermediária entre os usuários e a infraestrutura permite transformar o acesso administrativo tradicional em um processo centralizado, controlado e auditável.

A integração com o Active Directory fornece uma camada centralizada de identidade, enquanto MFA, ACLs, filtros de comandos e gravação de sessões adicionam controles complementares para proteção dos ativos críticos.

O projeto demonstra, na prática, a aplicação de conceitos de PAM, IAM, hardening, controle de acesso, auditoria e governança de infraestrutura em um ambiente híbrido Linux/Windows.

🔗 Projeto

Bastion Host & PAM com JumpServer + Active Directory

Projeto desenvolvido para fins de estudo, laboratório e demonstração de conceitos de segurança e governança de acessos privilegiados.
