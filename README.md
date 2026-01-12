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

## 🚦 Visão Geral do Dashboard:
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

## 🚨 Alerta 1: Potencial de Exfiltração de Dados

### Triagem dos Artefatos:
- **IP de Origem:** 192.168.45.66 (Localização: UK04 / Sala de Reuniões)  
- **Destino:** .zoom.us  
- **Volume de Dados:** 5,8 GB Enviados / 5,2 GB Recebidos  

### Análise Técnica:
- **Destino:** Domínio Zoom, serviço autorizado e amplamente utilizado. Sem indícios de comunicação com IPs C2 ou domínios maliciosos.  
- **Contexto:** Tráfego originado de sala de reuniões. Volume compatível com videoconferência em HD de longa duração.  

### Veredito:
**Falso Positivo (FP)** — o tráfego legítimo de videoconferência ultrapassou o threshold configurado no SIEM.


> **Obs:** Threshold = Limite.

### Recomendações:
- Implementar whitelist para domínios de colaboração (Zoom, Teams, Meet).  
- Ajustar regras de correlação considerando tipo de ativo (ex.: salas de conferência).  

### Notas de Análise:
- Threshold baixo gera muitos Falsos Positivos, aumentando fadiga de alertas.  
- Técnicas de exfiltração “Low and Slow” podem operar abaixo do threshold, exigindo análise de comportamento em períodos prolongados.  
- **Low and Slow:** Técnica furtiva usada por atacantes para evitar detecção, transmitindo dados lentamente.

<br>

## 🚨 Alerta 2: Criação de Arquivos com Dupla Extensão

### Triagem dos Artefatos:
- **Host:** LPT-HR-009 (Laptop do setor de RH)  
- **Usuário:** S.Conway  
- **Processo Origem:** chrome.exe  
- **Arquivo:** cats2025.mp4.exe  
- **URL de Origem (MotW):** `https://freecatvideoshd.monster/cats2025.mp4.exe`  
- **Hash MD5:** 14d8486f3f63875ef93cfd240c5dc10b  

### Análise Técnica:
- **Mascaramento:** Extensão `.mp4.exe` disfarça arquivo executável como vídeo.  
- **Mark of the Web (MotW):** Indica download de fonte externa, sinalizando risco.  
- **Vetor de Ataque:** Download via navegador, possivelmente por phishing ou malvertising.  

### Veredito:
**Verdadeiro Positivo:** Arquivo malicioso confirmado por extensão dupla e domínio suspeito

### Notas de Análise:
- Dupla Extensão: cats2025.mp4.exe ---> Tanto o `.MP4` quanto o `.exe`.
- Mark of the Web: O MotW é um recurso de segurança dos sistemas Windows que "carimba" arquivos baixados da internet ou de fontes externas não confiáveis. <br>
      ↪️ Exemplo: Se o usuário tentasse abrir esse arquivo, o Windows provavelmente mostraria aquela tela azul do SmartScreen dizendo "O Windows protegeu o seu computador".

### 📝 Plano de Resposta:
1. **Remediação:** Excluir o arquivo cats2025.mp4.exe e realizar um scan completo de EDR no host;
2. **Bloqueio:** Adicionar o domínio `freecatvideoshd.monster` e o `MD5` na Blacklist do Web Filter e do Antivírus corporativo;
3. **Educação:** Alertar o usuário S.Conway sobre os riscos de downloads em sites não oficiais.

---

## 🏛️ Créditos e Direitos Autorais:
> [!IMPORTANT]
> **Nota:** Este projeto faz parte de estudos práticos na plataforma [TryHackMe](https://tryhackme.com/).
> Todos os direitos sobre laboratórios, marcas e infraestrutura pertencem à respectiva plataforma.
> A documentação reflete a metodologia analítica e os resultados obtidos durante a resolução do desafio. 

---

## 🔗 Compartilhe com a comunidade 🧡

Por favor, se esse conteúdo te ajudou, não esqueça de compartilhar 😁

[![GitHub Repo stars](https://img.shields.io/badge/share%20on-twitter-03A9F4?logo=twitter)](https://twitter.com/share?url=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-facebook-1976D2?logo=facebook)](https://www.facebook.com/sharer/sharer.php?u=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1) [![GitHub Repo stars](https://img.shields.io/badge/share%20on-linkedin-3949AB?logo=linkedin)](https://www.linkedin.com/shareArticle?url=https://github.com/Luanacyberdef/Triagem-de-Alertas-SOC-L1)
