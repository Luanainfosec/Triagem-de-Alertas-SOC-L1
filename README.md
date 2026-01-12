<div align="center"> 
  <img alt="TryHackMe" src="https://repository-images.githubusercontent.com/518509014/f7450454-158c-45e0-8b38-0c0ae4d7394c" width="300px" /> <h1> 🛡️ Triagem de Alertas SOC L1</h1> Este projeto documenta a execução prática do laboratório de Triagem de Alertas SOC no TryHackMe, contendo exemplos de um Verdadeiro Positivo (TP) e um Falso Positivo (FP).
</div>

<br>

## 📑 Objetivos:
- Compreender a estrutura, o ciclo de vida e a importância dos alertas em um Centro de Operações de Segurança (SOC);
- Explorar campos de alerta, gerenciamento de status (Aberto, Em Andamento, Fechado) e metodologias de classificação.

<br>

## 🛠️ Ficha Técnica e Contexto Operacional:
- Plataforma: TryHackMe.
- Analista: [@Luanacyberdef](https://tryhackme.com/p/Luanacyberdef).
- Ambiente: Simulador de Dashboard SOC.
- Metodologia: Investigação baseada em evidências e análise de alertas.
- Sala no TryHackMe: [SOC L1 Alert Triage](https://tryhackme.com/room/socl1alerttriage).

<br>

## 🖥️ Visão Geral do Dashboard:
Ao iniciar a operação, foram identificados 5 alertas e 3 estavam pendentes na fila de triagem.
<img width="1242" height="456" alt="image" src="https://github.com/user-attachments/assets/8ab95f4e-c062-41f0-81e7-d04fde1dcf4f" />

> Simulador de Dashboard SOC.

<br>

### 📐 Propriedades de Alerta:

|       Propriedade       | Descrição                                                                                                             | Exemplo                                                                                                                             |
| :---------------------: | :-------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
|   **Tempo de Alerta**   | Horário de criação do alerta. Geralmente alguns minutos após o evento real.                                           | - Hora de alerta: 21 de março, 15:35H <br> - Horário do evento: 21 de março, 15:32H                                                 |
|    **Nome de Alerta**   | Resumo baseado na regra de detecção que gerou o alerta.                                                               | - Local de Login Incomum <br> - E-mail Marcado como Phishing <br> - Windows RDP Bruteforce <br> - Potencial de Exfiltração de Dados |
| **Gravidade do Alerta** | Define a urgência do alerta, inicialmente determinada pelos engenheiros de detecção. Pode ser ajustada pelo analista. | 🟢 Baixo / Informativo <br> 🟡 Médio / Moderado <br> 🟠 Alto / Severo <br> 🔴 Crítico / Urgente                                     |
|   **Status de Alerta**  | Indica se o alerta está sendo tratado ou se a triagem foi concluída.                                                  | 🆕 Novo / Não designado <br> 🔄 Em andamento / Pendente <br> ✅ Fechado / Resolvido                                                  |
|  **Veredito de Alerta** | Classificação indicando se o alerta é uma ameaça real ou não.                                                         | 🔴 Verdadeiro Positivo (TP) <br> 🟢 Falso Positivo (FP)                                                                             |
|  **Analista Designado** | Analista responsável pela revisão do alerta.                                                                          | - Também chamado de “dono do alerta”                                                                                                |

<br>

## 🚨 Alerta: Potencial de Exfiltração de Dados
### 1. Triagem dos Artefatos:
* **IP de Origem:** 192.168.45.66 (Localização: UK04 / Sala de Reuniões)
* **Destino:** .zoom.us
* **Volume de Dados:** 5,8 GB Enviados / 5,2 GB Recebidos.

### 2. Análise Técnica:
* **Análise de Destino:** O domínio pertence ao Zoom, um serviço homologado e amplamente utilizado para comunicação. Não há indícios de comunicação com IPs de C2 ou domínios maliciosos conhecidos.
* **Avaliação de Contexto:** O tráfego originou-se de uma Sala de Reuniões. Em um cenário de trabalho híbrido ou reuniões globais, o consumo de 5GB de dados é compatível com uma sessão de vídeo em HD de longa duração (Ex: uma manhã inteira de conferência).

### 3. Veredito: Falso Positivo(FP).
* O incidente ocorreu porque a atividade legítima de videoconferência ultrapassou o threshold de volume configurado no SIEM.

> **Obs:** Threshold = Limite.

## 🏷 Recomendação:
1. Implementar uma política de Whitelisting para domínios de colaboração conhecidos (Zoom, Teams, Meet);
2. Ajustar a regra de correlação para considerar o "Tipo de Ativo" (Ex: ignorar picos de tráfego de vídeo em dispositivos de salas de conferência durante o horário comercial).

### 📍 Notas de Análise:
1. Threshold muito baixo: Gera muitos Falsos Positivos (como o caso do Zoom), causando "fadiga de alertas".
2. Embora o alerta atual tenha sido um Falso Positivo devido ao tráfego legítimo do Zoom, é importante ressaltar que **os atacantes reais podem utilizar a técnica Low and Slow.** Nesses casos, **a exfiltração ocorre em volumes abaixo do threshold** de 5GB para evitar a detecção imediata, exigindo uma análise de comportamento de rede em períodos prolongados.

> A técnica Low and Slow (Baixo e Lento) é uma estratégia furtiva utilizada por atacantes para evitar a detecção por ferramentas de segurança.

## 🚨 Alerta: Criação de Arquivos com Dupla Extensão
### 1. Triagem dos Artefatos:
* **Host:** LPT-HR-009 (Provavelmente um laptop do setor de Recursos Humanos).
* **Usuário:** S.Conway
* **Processo Origem:** chrome.exe (Indica download via navegador).
* **Arquivo Alvo:** cats2025.mp4.exe (Disfarçado de vídeo para atrair o interesse do usuário).
* **URL de Origem (MotW):** `https://freecatvideoshd.monster/cats2025.mp4.exe` (Domínio altamente suspeito .monster).
* **Hash MD5:** 14d8486f3f63875ef93cfd240c5dc10b

### 2. Análise Técnica:
* **Técnica de Mascaramento:** O arquivo utiliza a extensão composta `.mp4.exe.` Em sistemas Windows com extensões ocultas, o usuário veria apenas `cats2025.mp4`, acreditando ser um vídeo inofensivo.
* **Análise do Mark of the Web (MotW):** A URL de origem utiliza um **domínio de baixa reputação** e nome apelativo ("freecatvideoshd"), características típicas de infraestrutura de Malware Delivery.
* **Vetor de Ataque:** O download foi realizado via Chrome, sugerindo que o usuário clicou em um link malicioso, possivelmente vindo de um e-mail ou anúncio (Phishing/Malvertising).

### 3. Veredito: Verdadeiro Positivo(TP)
* A combinação de uma dupla extensão executável disfarçada de mídia, baixada de um domínio não confiável para a pasta de Downloads, confirma a tentativa de infecção por malware.

> Dupla Extensão: cats2025.mp4.exe ---> Tanto o `.MP4` quanto o `.exe`. <br>
> Mark of the Web: O MotW é um recurso de segurança dos sistemas Windows que "carimba" arquivos baixados da internet ou de fontes externas não confiáveis. <br>
      ↪️ Exemplo: Se o usuário tentasse abrir esse arquivo, o Windows provavelmente mostraria aquela tela azul do SmartScreen dizendo "O Windows protegeu o seu computador".

## 📝 Plano de Resposta:
1. **Remediação:** Excluir o arquivo cats2025.mp4.exe e realizar um scan completo de EDR no host LPT-HR-009;
2. **Bloqueio:** Adicionar o domínio `freecatvideoshd.monster` e o `MD5` no Blacklist do Web Filter e do Antivírus corporativo;
3. **Educação:** Notificar o usuário S.Conway sobre os riscos de downloads em sites não oficiais.

---

## 🏛️ Créditos e Direitos Autorais:
> [!WARNING]
> **Nota:** Este projeto faz parte de estudos práticos na plataforma [TryHackMe](https://tryhackme.com/).
> Todos os direitos sobre laboratórios, marcas e infraestrutura pertencem à respectiva plataforma.
> A resultados reflete a metodologia analítica e os resultados obtidos durante a resolução do desafio. 

---

## 🔗 Compartilhe com a comunidade 🧡

Por favor, se esse conteúdo te ajudou, não esqueça de compartilhar 😁

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://twitter.com/share?url=https://github.com/Luanacyberdef/TrTTTTriagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://www.facebook.com/sharer/sharer.php?u=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://www.linkedin.com/shareArticle?url=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1)
